# 📝 Daily Work Report - Idham (2026-10-26)

---

## 📅 Laporan Harian - 25–26 Oktober 2026

> Mencakup pekerjaan 25 Oktober hingga 26 Oktober dini hari (commit `resolve #480` pukul 00:51). Pekerjaan `save #480` tanggal 24 Oktober sudah dilaporkan di [App-v2-24.md](App-v2-24.md); laporan ini hanya berisi lanjutannya.

---

## 🌿 Branch: `issue-480` — Pemisahan Endpoint Pelanggan Reguler & Pelanggan Mitra di Partner API (Penyelesaian & Audit Keamanan)

### 📌 Informasi Issue

- **Nomor Issue**: #480
- **Judul Issue**: Partner API: endpoint pelanggan reguler (`/customers`) dan pelanggan mitra (`/customer-partners`) dipisah, beserta pengerasan keamanan
- **Status Branch**: `Belum di-merge`. Seluruh pekerjaan sudah di-rebase ke `master` terbaru, di-squash menjadi satu commit `resolve #480`, dan sudah ada di `origin/issue-480`. Siap PR.

### 📅 Rincian Commit

#### [dc9b0abe] - resolve #480 - 26 Oktober 2026, 00:51

- **Ringkasan**:
  - Melanjutkan `save #480` dengan tiga putaran audit keamanan. Setiap celah yang ditemukan diperbaiki dan dikunci test.
  - Arah fitur dipastikan bersama manajemen: endpoint pelanggan reguler dan pelanggan mitra **tetap dipisah**, dan `/customers` khusus pelanggan reguler.
  - Cakupan dibatasi **hanya Partner API**. Perubahan pada perbaikan data `setCustomerPID` (endpoint developer aplikasi utama) yang sempat masuk `save #480` dikembalikan seperti semula.
  - Hasil akhir di-rebase ke `master` terbaru dan di-squash menjadi satu commit.

- **Celah keamanan yang ditutup** (semuanya di endpoint pelanggan Partner API):

  | Celah | Dampak sebelum diperbaiki | Perbaikan |
  | ----- | ------------------------- | --------- |
  | Pindah pelanggan ke mitra lain saat masih punya akun radius | Mitra lama tetap bisa melihat, mengubah, dan memutus akun pelanggan yang sudah pindah | Ditolak 422 sampai akun dipindah atau dihapus |
  | Pelanggan milik mitra terhapus atau mitra lama tanpa pengaturan portal | Masih terbaca dan bisa diubah admin p-api | Cakupan memakai daftar-putih mitra yang aktif dan tampil di portal |
  | Pelanggan tanpa mitra dengan `pid` bukan `master` | Lolos syarat Pelaporan, sehingga tampil tanpa layanan terlaporkan | Cakupan reguler mensyaratkan `pid: 'master'` |
  | Create menerima field apa pun dari body | `documents` berbentuk JSON bisa mencatat nama berkas pelanggan lain lalu memicu penghapusannya; `auth_token` dan `created_at` bisa disusupkan | Create memakai daftar-putih field |
  | ID berbentuk objek atau array di update, change-status, dan teknisi/penagih | "Ubah pelanggan X" bisa berubah menjadi "ubah pelanggan mana pun yang cocok" | Hanya string/angka yang diterima (400) |
  | Unggahan `documents` di create/update tanpa validasi | Berkas jenis apa pun dan jumlah tak terbatas bisa disimpan | Hanya JPG/PNG/PDF, maksimal 10 berkas |
  | Foto profil diproses setelah pelanggan tersimpan | Gambar rusak menghasilkan 500 padahal pelanggan sudah terbuat, sehingga aplikasi mengulang dan pelanggan menjadi ganda | Foto diproses sebelum menyimpan |
  | MinIO gagal setelah data tersimpan | 500 padahal data sudah masuk; dokumen tercatat tanpa berkasnya | Dokumen diunggah sebelum menyimpan dengan rollback; kegagalan foto setelah tersimpan hanya dicatat |
  | Jawaban "sudah dipakai" untuk KTP/NPWP/HP/email | Bisa dipakai massal untuk menebak apakah seseorang terdaftar sebagai pelanggan pihak lain | Batas laju 60 permintaan per 5 menit per sesi (429) |
  | `'remove-data'` pada `username`/`password` | Login pelanggan bisa dikosongkan dan berakhir error 500 | Ditolak 400 |
  | Field teks `image` di update | Aplikasi luar bisa menyimpan URL sembarang di data yang dipakai bersama V1 | Tidak lagi diterima |
  | Respons read | Mengirim `admin_id` dan flag `super` admin internal ke aplikasi mitra | Flag `super` dibuang; sesi mitra hanya menerima nama |

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Partner API**

  - `backend/src/controllers/partnerApiCustomer.controller.js`
    - **Cakupan per jenis (`buildOwnerFilter`):**
      - Reguler mensyaratkan `partner` kosong **dan** `pid: 'master'`.
      - Pelanggan mitra untuk admin memakai daftar-putih `partner: { $in: <mitra reseller yang tampil di portal> }`, menggantikan daftar-hitam mitra tersembunyi.
    - **Create:**
      - Memakai daftar-putih field sesuai Swagger (`pick`); `documents` hanya dari unggahan berkas.
      - Urutannya kini: validasi dan proses unggahan, unggah dokumen (rollback bila gagal), baru menyimpan pelanggan.
    - **Update:**
      - Pindah mitra ditolak 422 (`moveHasAuthentication`) bila pelanggan masih punya akun radius.
      - `image` dibuang dari daftar field, dan `'remove-data'` pada `username`/`password` ditolak 400.
      - `tech_support`/`pay_support` harus string.
      - Dokumen dicatat lewat `pushObj` dalam update yang sama dengan perubahan data.
    - **Change-status:** ID harus string/angka.
    - **Read:** `tech_support`/`pay_support` disaring lewat `toAdminRef` (tanpa `super`; `admin_id` hanya untuk sesi admin).
    - **Hapus avatar:** kegagalan MinIO kini 500, bukan 404.
    - **Helper baru:** `isPlainId`, `validateDocumentUploads`, `prepareAvatar`, `uploadPreparedAvatar`, `discardPreparedAvatar`, `uploadDocumentsOrRollback`, `removeUploadedDocuments`, `toAdminRef`.
    - Log yang sebelumnya tanpa prefiks konteks dirapikan.
  - `backend/src/middlewares/partnerApiCustomerLimiter.middleware.js` [NEW]
    - `partnerApiCustomerWriteLimiter`: 60 permintaan per 5 menit per sesi pengguna (kunci peran + `_id`, bukan IP). Kelebihannya dijawab 429.
  - `backend/src/routes/partnerApi.route.js`
    - Limiter dipasang di create dan update kedua kelompok endpoint.
    - **Swagger:**
      - Body create dan update dijadikan komponen bersama (`PartnerApiCustomerCreateFields`, `PartnerApiCustomerUpdateFields`); `/customer-partners/*` hanya menambahkan `partner` lewat `allOf`, sehingga dokumentasi tidak dobel.
      - Ditambah respons 422 untuk pindah mitra, 429 untuk batas laju, aturan unggahan, aturan ID, dan bentuk `tech_support`/`pay_support`.

  **Backend — Service & i18n**

  - `backend/src/services/partner.service.js`
    - Fungsi baca baru `getVisiblePortalPartnerIds` (daftar-putih mitra reseller yang tampil di portal), tanpa cache. Hanya dipakai Partner API.
  - `backend/src/services/recovery.service.js`
    - Dikembalikan ke versi `master`. Perubahan `setCustomerPID` dari `save #480` dibatalkan karena endpoint ini milik aplikasi utama (di luar p-api), dan p-api sudah menyembunyikan data `pid` yang tidak rapi dengan sendirinya.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru: `customer.moveHasAuthentication`, `customer.tooManyDocuments`, `customer.invalidImage`, `customer.rateLimited`, `customer.loginFieldRequired`.

  **Backend — Test**

  - `backend/test/integration/partnerApiCustomer.hardening.test.js` [NEW]
    - 26 test: 24 dijalankan untuk **kedua** kelompok endpoint lewat `describe.each` (12 kasus × 2), ditambah 2 test respons read:
      - Operator Mongo di change-status dan teknisi/penagih.
      - Validasi tipe dan jumlah dokumen.
      - Foto rusak dan foto valid.
      - Kegagalan MinIO pada create dan update.
      - `'remove-data'` pada kredensial, dan field `image`.
      - Penyaringan data admin di respons read.
  - `backend/test/integration/partnerApiCustomerLimiter.test.js` [NEW]
    - 429 setelah 60 permintaan; sesi lain tidak ikut terkena.
  - `backend/test/integration/partnerApiCustomer.read.test.js`
    - Daftar-putih mitra (mitra terhapus, mitra lama, data bisnis tidak terbaca) dan pelanggan dengan `pid` bukan `master`.
  - `backend/test/integration/partnerApiCustomer.update.test.js`
    - Pindah mitra dengan akun radius ditolak 422, dan ID boolean/array ditolak 400.
  - `backend/test/integration/partnerApiCustomer.create.test.js`
    - `documents` berbentuk JSON, `auth_token`, dan `created_at` diabaikan.
  - `backend/test/integration/partnerApiCustomer.documents.test.js`
    - Kegagalan MinIO saat menghapus avatar kini dikunci sebagai 500.
  - `backend/test/integration/recoveryCustomerPid.test.js`
    - Dihapus bersama pembatalan perubahan `setCustomerPID`.
  - Test baru diverifikasi gagal pada kode sebelum perbaikan, jadi benar-benar menangkap celahnya. 298 test terkait lulus setelah rebase ke `master` terbaru.

  **Dokumentasi**

  - `AGENTS.md`
    - Aturan 11 pola Visibilitas Partner API diperbarui:
      - Daftar-putih mitra dan syarat `pid: 'master'`.
      - Daftar-putih field create, aturan ID, aturan unggahan dan foto profil.
      - Batas laju, larangan mengosongkan kredensial, dan penolakan pindah mitra.
      - Catatan pengecualian penamaan p-api (bentuk jamak dan `:id`).

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #480 | Pemisahan endpoint pelanggan Partner API (penyelesaian) | Endpoint pelanggan reguler dan mitra terpisah sesuai keputusan manajemen; 12 celah keamanan ditutup; perubahan murni di Partner API tanpa perubahan database |

### Kemampuan Baru Pengguna/Admin

- Aplikasi eksternal mendapat jawaban yang jelas untuk kasus yang sebelumnya membingungkan: 422 saat pindah mitra masih punya akun layanan, 429 saat terlalu banyak simpan data, dan 400 untuk berkas atau foto yang tidak valid.

### Bug Fix / Solusi Masalah

- Data pelanggan milik mitra terhapus, mitra lama, atau mitra tersembunyi tidak lagi bisa dibuka admin p-api.
- Akun layanan tidak lagi tertinggal di mitra lama saat pelanggan dipindah.
- Pelanggan tidak lagi tergandakan karena gangguan pemrosesan foto atau MinIO.
- Login pelanggan tidak bisa lagi dikosongkan lewat API.
- Data admin internal tidak lagi terkirim ke aplikasi mitra.

### Menu/Fitur Baru

- Tidak ada menu baru (perubahan khusus Partner API).

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Partner API memisahkan pelanggan reguler (`/p-api/v1/customers/*`) dan pelanggan mitra (`/p-api/v1/customer-partners/*`). Sesi mitra hanya mengelola pelanggannya sendiri di `/customer-partners/*`. Sesi admin memakai keduanya. Semua aturan keamanan (cakupan data, jenis pelanggan yang tidak bisa ditukar, validasi input dan unggahan, batas laju) ditegakkan di server, sehingga aplikasi tidak bisa melewatinya.
- **Langkah Penggunaan (untuk pengembang aplikasi eksternal)**:
  1. Login lewat Partner API sebagai admin atau mitra.
  2. Pelanggan reguler: gunakan `/p-api/v1/customers/*` (khusus admin).
  3. Pelanggan mitra: gunakan `/p-api/v1/customer-partners/*`. Admin wajib menyebut `partner` saat membuat pelanggan.
  4. Untuk memindah pelanggan ke mitra lain, pindahkan atau hapus dulu akun layanannya; bila tidak, server menjawab 422.
  5. Tangani 429 dengan memberi jeda sebelum mencoba lagi.
