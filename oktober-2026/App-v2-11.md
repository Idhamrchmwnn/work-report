# 📝 Daily Work Report - Idham (2026-10-11)

---

## 📅 Laporan Harian - 11 Oktober 2026

---

## 🌿 Branch: `issue-394` — Toggle Portal untuk Pelanggan & Pelanggan Bisnis (Visibilitas di Portal Mitra / p-api)

### 📌 Informasi Issue

- **Nomor Issue**: #394
- **Judul Issue**: Toggle "Portal" untuk Pelanggan dan Pelanggan Bisnis (visibilitas di portal mitra / p-api)
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-394`, siap PR)

### 📅 Rincian Commit

#### [34aa1f8e] - save #394 - 11 Oktober 2026, 16:48

- **Ringkasan**: Melanjutkan pekerjaan kemarin (toggle portal untuk Pelanggan reguler). Toggle `show_in_portal` kini juga tersedia untuk **Pelanggan Bisnis yang tidak punya reseller pemilik**. Bisnis yang dimatikan tidak terlihat dan tidak bisa diubah oleh sesi admin di aplikasi mitra (p-api), termasuk fakturnya. Bisnis milik reseller tidak punya toggle sendiri dan mengikuti visibilitas reseller pemiliknya. Commit ini juga menutup dua celah keamanan di endpoint p-api dan endpoint toggle mitra.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Service, Controller, Route**

  - `backend/src/services/business.service.js`
    - `show_in_portal` ditambahkan ke daftar field yang dikirim ke datatable Bisnis.
    - Konstanta baru `BUSINESS_PORTAL_VISIBLE_FILTER`: filter Mongo yang membuang bisnis tanpa reseller pemilik (`partner: null`) yang `show_in_portal`-nya bukan `true`. Filter ini memakai `$nor` supaya tidak bentrok dengan `$or` pemanggil. Bisnis milik reseller tidak ikut tersaring.
    - Fungsi baru `toggleBusinessPortalVisibility(partnerId)`: membalik `show_in_portal` secara atomik dalam satu `findOneAndUpdate` (pipeline update). Fungsi ini hanya berlaku untuk dokumen bisnis (`reseller: false`) tanpa pemilik. Dokumen lama tanpa field ini menjadi tampil pada toggle pertama.
    - Fungsi baru `getHiddenPortalBusinessIds()`: mengambil daftar `_id` bisnis tersembunyi, termasuk yang sudah soft delete, untuk menyaring faktur.
  - `backend/src/controllers/business.controller.js`
    - Handler baru `changePortalVisibilityBusiness`: membalas 400 bila `id` kosong atau bukan string, dan 404 bila bisnis tidak ditemukan atau milik reseller. Bila berhasil, membalas `{ status, message }`.
    - `createBusiness` kini membuang `show_in_portal` dari body. Visibilitas hanya bisa diubah lewat endpoint khusus yang memerlukan privilege.
  - `backend/src/routes/business.route.js`
    - Route baru `PATCH /api/v1/business/change-portal-visibility` dengan `protectedAdmin` dan `checkPrivilege('business.changeSensitive')`, lengkap dengan dokumentasi Swagger.
  - `backend/src/services/partner.service.js`
    - Fungsi lama `setPartnerPortalVisibility` yang sudah tidak dipakai dihapus.
    - **Perbaikan keamanan:** toggle visibilitas mitra kini hanya mencari dokumen `reseller: true`. Pelanggan bisnis juga disimpan sebagai dokumen Partner, jadi sebelumnya endpoint mitra bisa dipakai membalik visibilitas bisnis tanpa privilege `business.changeSensitive`.
  - `backend/src/routes/partner.route.js`
    - Hanya Swagger: respons 404 endpoint toggle mitra kini menjelaskan bahwa pelanggan bisnis juga dijawab 404, karena bisnis punya endpoint sendiri.

  **Backend — Penegakan di Partner API (p-api)**

  - `backend/src/controllers/partnerApiBusiness.controller.js`
    - Helper `withPortalVisibility` kini juga menggabungkan `BUSINESS_PORTAL_VISIBLE_FILTER`. Dampaknya: daftar, statistik, dan detail bisnis untuk admin menyembunyikan bisnis tanpa pemilik yang dimatikan.
    - Helper baru `buildBusinessScopeFilter(req)` dipakai di endpoint ubah, hapus, dan ganti status. Mitra tetap dibatasi ke bisnisnya sendiri. Admin kini dibatasi ke bisnis yang tampil di portal, jadi bisnis tersembunyi dijawab 404.
    - `createPartnerAppBusiness`: nilai `show_in_portal` dari body dibuang. Bisnis tanpa pemilik yang dibuat admin lewat p-api otomatis diberi `show_in_portal: true`, kalau tidak bisnis itu langsung hilang setelah dibuat.
    - **Perbaikan keamanan di `updatePartnerAppBusiness`:** field `show_in_portal`, `reseller`, `wallet`, `wallet_history`, dan `hsa_profit` kini dibuang dari body untuk semua role. Sebelumnya field itu hanya dibuang untuk role partner, padahal sesi admin di p-api tidak diperiksa hak aksesnya. Akibatnya admin p-api bisa mengubah saldo atau menjadikan pelanggan bisnis sebagai reseller lewat body update.
  - `backend/src/controllers/partnerApiInvoice.controller.js`
    - Filter faktur admin kini juga mengecualikan faktur milik bisnis tersembunyi (dari `getHiddenPortalBusinessIds`), selain faktur mitra tersembunyi. Faktur `type: 'partner'` juga dipakai pelanggan bisnis.
  - `backend/src/routes/partnerApi.route.js`
    - Hanya Swagger: endpoint bisnis dan faktur p-api diberi catatan bahwa bisnis tersembunyi dikecualikan untuk `role: admin` (404), dan bahwa field sensitif dari body diabaikan.

  **Backend — Privilege & i18n**

  - `backend/src/config/privilege.json`
    - Key baru `business.changeSensitive` (hasil generator `npm run gp`).
  - `backend/src/config/privilegeDictionary.json`
    - Entri kamus `business.changeSensitive`: label, deskripsi, lokasi UI (`/users/business`), endpoint, dependensi `business.read` + `business.list`, dan risiko `critical`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `business.portalVisibilityChange` ("Visibilitas pelanggan bisnis di portal mitra berhasil diubah").

  **Backend — Test**

  - `backend/test/integration/businessPortalVisibility.test.js` [NEW]
    - Toggle membalik nilai, dan dokumen lama tanpa field menjadi tampil pada toggle pertama.
    - Menolak 404 untuk bisnis milik reseller, reseller, dan id yang tidak ada. Menolak 400 untuk id kosong atau bukan string.
    - Uji balapan: N toggle bersamaan menghasilkan tepat N kali balik.
    - Endpoint toggle mitra tidak bisa dipakai membalik visibilitas bisnis.
    - Di p-api, list/list-status/read menyembunyikan bisnis tersembunyi, dan bisnis milik reseller mengikuti resellernya.
    - Jalur tulis (ubah, status, hapus) menjawab 404 untuk bisnis tersembunyi.
    - Update p-api mengabaikan `show_in_portal`, `reseller`, dan `wallet` dari body.
    - Bisnis buatan admin lewat p-api langsung tampil.
    - Detail faktur milik bisnis tersembunyi dijawab 404.

  **Frontend**

  - `frontend/src/app/pages/users/business/schema/columns.jsx`
    - Kolom baru **Portal** dengan `StatusCell` yang memanggil `/business/change-portal-visibility/`. Kolom ini hanya aktif untuk pemegang `business.changeSensitive` dan bisa difilter aktif/nonaktif. Baris bisnis milik reseller dikosongkan karena tidak punya toggle sendiri.
  - `frontend/src/app/pages/users/business/profile.jsx`
    - Halaman profil bisnis menampilkan `show_in_portal` dengan `PortalBadge`. Field ini disembunyikan untuk bisnis milik reseller.
  - `frontend/src/constants/privilegeDescriptions.id.json`, `frontend/src/constants/privilegeDescriptions.en.json`
    - Tooltip checkbox untuk privilege `business.changeSensitive`.

  **Dokumentasi**

  - `AGENTS.md`
    - Pola baru di Bab 9 tentang visibilitas di portal mitra (`show_in_portal`). Isinya: tiga entitas yang punya toggle (mitra reseller, pelanggan reguler, bisnis tanpa pemilik), filter `$nor`, toggle wajib atomik, `show_in_portal` wajib dibuang dari body, bisnis tanpa pemilik dikenali dari `partner: null` (bukan `pid`), dan cara menyaring relasi akun radius. Tanggal pembaruan menjadi 2026-10-11.

#### [5b9206a6] - resolve #394 - 11 Oktober 2026, 16:50

- **Komponen yang Berubah**: gabungan seluruh perubahan di atas dan commit `save #394` kemarin (toggle portal Pelanggan reguler, lihat laporan [App-v2-10.md](App-v2-10.md)). Totalnya 34 berkas.
- **Deskripsi Perubahan & Fungsi**:
  - Squash kedua commit `save #394` menjadi satu commit `resolve #394` di atas `master` terbaru, lalu di-push ke `origin/issue-394`. Salinan sebelum squash disimpan di branch lokal `backup/issue-394-before-squash`.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #394 | Toggle Portal Pelanggan & Bisnis | Admin bisa mengatur pelanggan reguler dan pelanggan bisnis (tanpa reseller) mana yang terlihat di aplikasi mitra; dua celah keamanan p-api ditutup |

### Kemampuan Baru Pengguna/Admin

- Admin dengan hak `business.changeSensitive` dapat menyalakan/mematikan visibilitas pelanggan bisnis (tanpa reseller pemilik) di portal mitra langsung dari tabel Bisnis.
- Admin dengan hak `customer.changeSensitive` dapat melakukan hal yang sama untuk pelanggan reguler (dikerjakan kemarin, kini sudah `resolve`).

### Bug Fix / Solusi Masalah

- Endpoint toggle visibilitas **mitra** tidak bisa lagi dipakai membalik visibilitas **pelanggan bisnis**. Sebelumnya ini bisa dilakukan tanpa privilege yang sesuai karena keduanya disimpan di koleksi yang sama.
- Sesi admin di p-api tidak bisa lagi mengubah `reseller`, saldo (`wallet`, `wallet_history`, `hsa_profit`), atau `show_in_portal` pelanggan bisnis lewat body update.
- Bisnis tersembunyi dan fakturnya tidak bisa lagi dibaca, diubah, atau dihapus lewat p-api oleh sesi admin.

### Menu/Fitur Baru

- Kolom **Portal** di tabel **Pengguna > Bisnis**.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur (Toggle Portal Pelanggan Bisnis)**: Pelanggan bisnis yang tidak punya reseller pemilik kini punya status "Portal". Bila mati, bisnis tersebut beserta fakturnya tidak terlihat oleh admin yang memakai aplikasi mitra. Bawaannya **mati**, jadi setelah rilis semua bisnis tanpa pemilik tersembunyi dari aplikasi mitra sampai dinyalakan. Bisnis milik reseller tidak punya toggle; visibilitasnya mengikuti reseller pemiliknya.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Pengguna > Bisnis**.
  2. Pada kolom **Portal**, klik toggle di baris bisnis yang ingin ditampilkan di aplikasi mitra. Baris bisnis milik reseller tidak menampilkan toggle.
  3. Status juga terlihat di halaman profil bisnis.
