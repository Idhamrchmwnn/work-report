# 📝 Daily Work Report - Idham (2026-10-27)

---

## 📅 Laporan Harian - 27 Oktober 2026

---

## 🌿 Branch: `issue-490` — Berita Acara Perubahan Layanan (BAPL)

### 📌 Informasi Issue

- **Nomor Issue**: #490
- **Judul Issue**: Berita Acara Perubahan Layanan (BAPL) untuk pelanggan reguler yang berganti paket, dengan tanda tangan pelanggan lewat link publik
- **Status Branch**: `Belum di-merge`. Masih tahap `save`, sudah di-rebase ke `master` terbaru (konflik diselesaikan), dan sudah ada di `origin/issue-490`.

### 📅 Rincian Commit

#### [6339b320] - save #490 - 27 Oktober 2026, 13:50

- **Ringkasan**:
  - Fitur baru **Berita Acara Perubahan Layanan (BAPL)**: dokumen pelengkap saat pelanggan broadband reguler berganti paket. Dokumen ini ditandatangani pelanggan lewat link publik.
  - Dokumen bisa dibuat dari tiga sumber:
    - **Permintaan web** yang sudah dikonfirmasi dan tertaut ke langganan.
    - **Permintaan aplikasi mobile** yang sudah ditinjau.
    - **Manual** dari menu **Dokumen > Perubahan Layanan**.
  - BAPL **tidak pernah** mengubah langganan, status permintaan, atau data pelanggan. Penggantian paket di Radius tetap dilakukan manual.
  - Isi dokumen disusun server (data langganan, paket lama, paket baru), bukan dari kiriman klien.
  - Link tanda tangan bisa disalin atau dikirim lewat WhatsApp milik PIC sendiri. Server tidak mengirim pesan apa pun.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Model & Service**

  - `backend/src/models/serviceChangeDocument.model.js` [NEW]
    - Koleksi baru `service_change_documents`.
    - Isi dokumen: sumber (`web`/`mobile`/`manual`) beserta rujukannya, salinan data pelanggan dan langganan, paket lama dan baru, tanggal berlaku, status (`pending`/`signed`), tanda tangan dan nama penanda tangan, `share_token`, dan nomor dokumen.
    - Indeks unik **parsial** (`deleted: false`) menjamin satu dokumen aktif per permintaan. Ini mencegah dokumen ganda walau dua PIC membuat bersamaan, tanpa cek-lalu-simpan.
    - Indeks `customer + createdAt` untuk tabel.
    - Soft delete. Dokumen yang dihapus membebaskan permintaan untuk dibuatkan dokumen baru.
  - `backend/src/services/serviceChangeDocument.service.js` [NEW]
    - `resolveSource`: menyusun isi dokumen di server dari sumbernya.
      - **Web:** permintaan harus berstatus `confirmed` dan sudah tertaut ke pelanggan dan langganan.
      - **Mobile:** permintaan harus sudah ditinjau (`change_review`); paket baru diambil dari `req_change`.
      - **Manual:** langganan harus milik pelanggan yang dipilih.
      - Hanya pelanggan reguler internal, dan paket baru tidak boleh sama dengan paket sekarang.
    - `createServiceChangeDocument`: nomor dokumen dari penghitung; tabrakan nomor diulang otomatis, dan tabrakan "satu dokumen per permintaan" dijawab jelas.
    - `updateServiceChangeDocument`: mengubah tanggal berlaku dengan pengecekan `__v` (409 bila basi). Dokumen yang sudah ditandatangani terkunci (422).
    - `deleteServiceChangeDocument`: soft delete bersyarat status `pending`, sehingga tanda tangan yang masuk bersamaan tidak ikut terhapus.
    - `findServiceChangeDocumentsForTable`, `findServiceChangeDocumentById`, dan `findServiceChangeDocumentBySource` untuk tabel, detail, dan panel di drawer permintaan.
    - **Sisi publik:**
      - `findPublicServiceChangeDocument` membaca lewat `share_token`, dan `toPublicDocument` membuang id internal dan token dari respons.
      - `validateSignInput` hanya menerima data URL PNG (maks 512 KB) dan nama penanda tangan (maks 100 karakter).
      - `signServiceChangeDocument` menandatangani secara atomik dengan syarat `status: 'pending'`. Tanda tangan kedua dijawab 409 dan tidak menimpa.
    - `generateShareToken` menghasilkan token acak 192 bit.

  **Backend — Controller & Route**

  - `backend/src/controllers/serviceChangeDocument.controller.js` [NEW]
    - Handler admin (daftar, detail, cari per sumber, buat, ubah, hapus) dan handler publik (baca dan tanda tangan). Status error dari service diterapkan ke `res` sebelum dilempar ulang.
  - `backend/src/routes/serviceChangeDocument.route.js` [NEW]
    - Endpoint admin baru, masing-masing dengan privilege dan Swagger:
      - `POST /service-change-document/list`
      - `GET /service-change-document/read/:document_id`
      - `GET /service-change-document/source/:source/:ref`
      - `POST /service-change-document/create`
      - `PATCH /service-change-document/update/:document_id`
      - `DELETE /service-change-document/delete/:document_id`
  - `backend/src/routes/public.route.js`
    - Endpoint publik baru:
      - `GET /public-docs/service-change/:token`, dengan batas laju yang sama seperti faktur publik.
      - `POST /public-docs/service-change/sign`, dengan `publicSignLimiter`.
  - `backend/src/app.js`
    - Mendaftarkan route BAPL.
  - `backend/src/middlewares/logger.middleware.js`
    - Endpoint tanda tangan publik ditambahkan ke `BODY_REDACTED_PREFIXES`, karena body-nya berisi gambar tanda tangan (base64, ratusan KB) dan nama pelanggan.

  **Backend — Privilege, i18n & Data**

  - `backend/src/config/privilege.json`
    - Grup privilege baru `serviceChangeDocument` (`list`, `read`, `create`, `update`, `delete`).
  - `backend/src/config/privilegeDictionary.json`
    - Entri kamus untuk kelima privilege baru.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Pesan BAPL: tidak ditemukan, sudah ditandatangani, terkunci, sumber belum siap, paket sama, data basi, dll.
  - `backend/src/data/aiNavigationRoutes.json`
    - Peta menu AI Agent ikut diperbarui sesuai hak akses menu Dokumen yang baru.

  **Backend — Test**

  - `backend/test/integration/serviceChangeDocument.test.js` [NEW]
    - **Dari permintaan web:**
      - Data disalin dari server, bukan dari klien.
      - Ditolak 422 bila belum `confirmed` atau belum tertaut ke langganan; ditolak bila paket baru sama.
      - Satu dokumen aktif per permintaan, juga saat dibuat bersamaan.
      - Status permintaan dan paket langganan tidak berubah.
    - **Mobile dan manual:** permintaan mobile wajib sudah ditinjau; manual memastikan kepemilikan langganan dan menolak pelanggan mitra.
    - **Tanda tangan publik:**
      - Halaman publik tanpa id internal maupun token; token tidak dikenal dijawab 404.
      - Tanda tangan sekali saja (yang kedua 409); tanda tangan bersamaan menghasilkan tepat satu yang berhasil.
      - Input yang bukan PNG atau nama kosong ditolak.
    - **Ubah dan hapus:** tanggal berlaku dengan `__v` (basi 409); dokumen bertanda tangan terkunci (422); setelah dihapus, permintaan bisa dibuatkan dokumen lagi.

  **Frontend — Menu Dokumen > Perubahan Layanan**

  - `frontend/src/app/pages/users/document/serviceChange/index.jsx` [NEW]
    - Tab **Perubahan Layanan** di halaman Dokumen: tabel BAPL beserta tombol buat dokumen.
  - `frontend/src/app/pages/users/document/serviceChange/schema/columns.jsx` [NEW]
    - Kolom tabel: tanggal dibuat, nomor, pelanggan, username PPPoE, paket lama dan baru, tanggal berlaku, status, PIC.
    - Aksi baris: pratinjau, salin link, kirim WhatsApp, dan hapus. Hapus disembunyikan untuk dokumen yang sudah ditandatangani.
  - `frontend/src/app/pages/users/document/serviceChange/CreateDrawer.jsx` [NEW]
    - Drawer buat dokumen manual (pilih pelanggan, layanan/akun PPPoE, paket baru, tanggal berlaku) maupun ubah tanggal berlaku.
  - `frontend/src/app/pages/users/document/serviceChange/schema/createSchema.js` [NEW]
    - Validasi Yup form pembuatan.
  - `frontend/src/app/pages/users/document/serviceChange/ServiceChangeDocumentView.jsx` [NEW]
    - Tampilan dokumen BAPL yang dipakai di pratinjau admin maupun halaman publik.
  - `frontend/src/app/pages/users/document/serviceChange/ServiceChangeDocumentPanel.jsx` [NEW]
    - Panel BAPL di drawer permintaan: menampilkan dokumen yang sudah ada atau tombol buat, beserta petunjuk bila permintaan belum siap.
  - `frontend/src/app/pages/users/document/serviceChange/useShareDocument.js` [NEW]
    - Aksi salin link tanda tangan dan buka WhatsApp berisi link ke nomor pelanggan, lewat WhatsApp milik PIC sendiri.
  - `frontend/src/app/pages/users/document/serviceChange/utils.js` [NEW], `frontend/src/app/pages/users/document/serviceChange/utils.test.js` [NEW]
    - Penyusun link publik dan URL WhatsApp, beserta test-nya.
  - `frontend/src/app/pages/users/document/registry.js`
    - Mendaftarkan tab Perubahan Layanan (privilege `serviceChangeDocument.list`).
  - `frontend/src/app/navigation/users.js`, `frontend/src/app/router/users/documentRoute.jsx`
    - Menu dan route Dokumen kini juga terbuka untuk pemegang `serviceChangeDocument.list`.

  **Frontend — Integrasi & Halaman Publik**

  - `frontend/src/app/pages/mobileApp/serviceChange/components/WebRequestDrawer.jsx`
    - Panel BAPL di drawer permintaan web. Tombol buat aktif bila permintaan berstatus Dikonfirmasi dan sudah tertaut ke pelanggan dan layanan.
  - `frontend/src/app/pages/mobileApp/serviceChange/components/ReviewDrawer.jsx`
    - Panel BAPL di drawer tinjau permintaan aplikasi mobile. Tombol buat aktif setelah permintaan ditinjau.
  - `frontend/src/app/pages/public/PublicServiceChangeDocument.jsx` [NEW]
    - Halaman publik `/p/service-change/:token`: pelanggan membaca dokumen lalu menandatanganinya. Dokumen yang sudah ditandatangani tampil apa adanya.
  - `frontend/src/app/router/public.jsx`
    - Route publik baru, dimuat lazy.
  - `frontend/src/components/shared/DocumentPreviewModal.jsx`
    - Mendukung jenis `service-change`: detail diambil ulang dari server (baris tabel tidak memuat tanda tangan), dan link berbagi ditampilkan.
  - `frontend/src/components/shared/table/rows.jsx`, `frontend/src/components/shared/table/status.js`
    - Sel dan opsi status BAPL ("Menunggu Tanda Tangan" / "Ditandatangani") untuk kolom dan filter tabel.
  - `frontend/src/constants/privilegeDescriptions.id.json`, `frontend/src/constants/privilegeDescriptions.en.json`
    - Tooltip kelima privilege baru.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Seluruh teks tab, form, panel, dokumen, dan halaman tanda tangan publik.

  **Dokumentasi**

  - `AGENTS.md`
    - Aturan baru §2.A nomor 32 "Berita Acara Perubahan Layanan":
      - Dokumen pelengkap yang tidak mengubah langganan.
      - Tiga sumber, dengan isi disusun server.
      - Satu dokumen aktif per permintaan lewat indeks parsial.
      - Tanda tangan publik atomik dan dokumen bertanda tangan terkunci.
      - Token 192 bit, dan kirim WhatsApp dari nomor PIC sendiri.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #490 | Berita Acara Perubahan Layanan (BAPL) | Perubahan paket pelanggan reguler kini punya dokumen resmi yang ditandatangani pelanggan secara online, dari permintaan web, mobile, maupun manual |

### Kemampuan Baru Pengguna/Admin

- Membuat BAPL langsung dari drawer permintaan perubahan layanan (web/mobile), atau manual dari menu Dokumen.
- Mengirim link tanda tangan ke pelanggan lewat salin link atau WhatsApp.
- Melihat status dokumen (Menunggu Tanda Tangan / Ditandatangani) dan pratinjau dokumen beserta tanda tangannya.
- Pelanggan menandatangani dokumen dari HP lewat link publik, tanpa login.

### Bug Fix / Solusi Masalah

- Tidak ada (fitur baru).

### Menu/Fitur Baru

- Tab **Perubahan Layanan** di **Pengguna > Dokumen**.
- Panel **Berita Acara** di drawer permintaan perubahan layanan web dan mobile.
- Halaman publik tanda tangan `/p/service-change/:token`.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: BAPL adalah berita acara resmi saat pelanggan reguler berganti paket. Isinya disusun otomatis dari data langganan dan permintaan. Dokumen hanya pelengkap, jadi penggantian paket di sistem Radius tetap dilakukan seperti biasa. Satu permintaan hanya punya satu dokumen aktif, dan dokumen yang sudah ditandatangani tidak bisa diubah atau dihapus.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka permintaan perubahan layanan: permintaan web yang sudah **Dikonfirmasi** dan tertaut ke pelanggan, atau permintaan mobile yang sudah ditinjau. Bisa juga dari **Pengguna > Dokumen > Perubahan Layanan** untuk pembuatan manual.
  2. Klik buat **Berita Acara**, periksa tanggal berlaku, lalu simpan.
  3. Kirim link tanda tangan ke pelanggan lewat **Salin link** atau **Kirim WhatsApp**.
  4. Pelanggan membuka link, membaca dokumen, mengisi nama, lalu menandatangani.
  5. Status dokumen berubah menjadi **Ditandatangani** dan dokumen terkunci.
