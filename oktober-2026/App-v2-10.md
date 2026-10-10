# 📝 Daily Work Report - Idham (2026-10-10)

---

## 📅 Laporan Harian - 10 Oktober 2026

---

## 🌿 Branch: `issue-394` — Toggle Portal untuk Pelanggan (Visibilitas di Portal Mitra / p-api)

### 📌 Informasi Issue

- **Nomor Issue**: #394
- **Judul Issue**: Toggle "Portal" untuk Pelanggan (visibilitas pelanggan di portal mitra / p-api)
- **Status Branch**: `Belum di-merge` (masih tahap `save`)

### 📅 Rincian Commit

#### [e1754eab] - save #394 - 10 Oktober 2026, 20:34

- **Ringkasan**: Pelanggan reguler (`pid: 'master'`) kini punya toggle `show_in_portal`, seperti Mitra Bisnis. Pelanggan yang dimatikan tidak terlihat dan tidak bisa diubah oleh sesi admin di aplikasi mitra (p-api), termasuk akun radius miliknya. Bawaannya `false` (tersembunyi). Pelanggan milik mitra dan aplikasi mobile pelanggan tidak terpengaruh.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Model, Service, Controller, Route**

  - `backend/src/models/customer.model.js`
    - Menambah field `show_in_portal` (Boolean, bawaan `false`, ber-index). Dokumen pelanggan lama yang belum punya field ini dianggap tersembunyi.
  - `backend/src/services/customer.service.js`
    - `show_in_portal` ditambahkan ke daftar field non-sensitif, jadi ikut terkirim ke datatable Pelanggan.
    - Konstanta baru `CUSTOMER_PORTAL_VISIBLE_FILTER`: filter Mongo yang membuang pelanggan reguler dengan `show_in_portal` bukan `true`. Filter ini memakai `$nor` supaya tidak bentrok dengan `$or` lain milik pemanggil. Pelanggan milik mitra tidak ikut tersaring.
    - Fungsi baru `toggleCustomerPortalVisibility(customerId)`: membalik nilai `show_in_portal` secara atomik dalam satu `findOneAndUpdate` (pipeline update), jadi dua klik bersamaan tidak saling menimpa. Hanya berlaku untuk pelanggan `pid: 'master'`. Kegagalan dicatat dengan `logger.error`.
    - Fungsi baru `buildPortalVisibleAuthenticationFilter(hiddenPartnerIds)`: menyusun filter akun radius yang membuang akun milik pelanggan tersembunyi, baik pelanggan reguler yang dimatikan maupun pelanggan milik mitra tersembunyi. Agar daftar ID tidak membengkak, fungsi ini memilih sisi yang lebih kecil: mengecualikan pelanggan tersembunyi, atau membatasi ke pelanggan yang tampil.
  - `backend/src/controllers/customer.controller.js`
    - Handler baru `changePortalVisibilityCustomer`: membalas 400 bila `id` kosong atau bukan string, 404 bila pelanggan tidak ditemukan atau milik mitra. Bila berhasil, membalas `{ status, message }`.
    - `createCustomer` kini membuang `show_in_portal` dari body. Visibilitas hanya bisa diubah lewat endpoint khusus yang memerlukan privilege.
  - `backend/src/routes/customer.route.js`
    - Route baru `PATCH /api/v1/customer/change-portal-visibility` dengan `protectedAdmin` dan `checkPrivilege('customer.changeSensitive')`, lengkap dengan dokumentasi Swagger (body, respons, kode 400/401/403/404).

  **Backend — Penegakan di Partner API (p-api)**

  - `backend/src/controllers/partnerApiCustomer.controller.js`
    - Helper `withPortalVisibility` kini juga menggabungkan `CUSTOMER_PORTAL_VISIBLE_FILTER`. Dampaknya: daftar, statistik status, dan detail pelanggan untuk admin otomatis menyembunyikan pelanggan reguler yang dimatikan.
    - Helper baru `buildCustomerScopeFilter(req)` menggantikan filter yang sebelumnya disalin di lima endpoint tulis (ubah, hapus, ganti status, set dokumen, hapus avatar). Mitra tetap dibatasi ke pelanggannya sendiri. Admin kini dibatasi ke pelanggan yang tampil di portal, jadi pelanggan tersembunyi dijawab 404. Ini juga menutup celah lama: mitra tersembunyi sebelumnya masih bisa diubah lewat endpoint tulis.
    - `createPartnerAppCustomer`: nilai `show_in_portal` dari body dibuang. Pelanggan reguler yang dibuat admin lewat p-api otomatis diberi `show_in_portal: true`, kalau tidak pelanggan itu langsung hilang setelah dibuat.
  - `backend/src/controllers/partnerApiRadius.controller.js`
    - `getPartnerAuthenticationFilter` (cabang admin) kini memakai `buildPortalVisibleAuthenticationFilter`, menggantikan penyaringan manual yang sebelumnya hanya menangani pelanggan milik mitra tersembunyi.
    - `resolveOwnedRef`: admin ditolak 400 ("pelanggan tidak ditemukan") saat menautkan akun radius ke pelanggan reguler yang tersembunyi.
  - `backend/src/routes/partnerApi.route.js`
    - Hanya dokumentasi Swagger: setiap endpoint pelanggan dan radius p-api diberi catatan bahwa pelanggan tersembunyi dikecualikan untuk `role: admin` (dijawab 404, atau 400 saat penautan radius).

  **Backend — Privilege & i18n**

  - `backend/src/config/privilege.json`
    - Key baru `customer.changeSensitive` (hasil generator `npm run gp`).
  - `backend/src/config/privilegeDictionary.json`
    - Entri kamus untuk `customer.changeSensitive`: label, deskripsi, lokasi UI (`/users/customer`), endpoint, dependensi `customer.read` + `customer.list`, dan risiko `critical`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `customer.portalVisibilityChange` ("Visibilitas pelanggan di portal mitra berhasil diubah").

  **Backend — Test**

  - `backend/test/integration/customerPortalVisibility.test.js` [NEW]
    - Toggle membalik nilai, dan dokumen lama tanpa field menjadi `true` pada toggle pertama.
    - Menolak 404 untuk pelanggan milik mitra atau yang tidak ada, dan 400 untuk `id` kosong atau bukan string.
    - Di p-api, list/list-status/read menyembunyikan pelanggan tersembunyi, dan jalur tulis menjawab 404.
    - Pelanggan buatan admin lewat p-api langsung tampil.
    - Akun radius pelanggan tersembunyi tidak tampil dan tidak bisa ditautkan.
    - Uji balapan: N toggle bersamaan menghasilkan tepat N kali balik.
    - Uji kedua mode `buildPortalVisibleAuthenticationFilter` memberi hasil yang sama.
  - `backend/test/integration/partnerApiCustomer.{changeStatus,delete,documents,list,listStatus,read,update}.test.js`
    - Data uji pelanggan reguler ditambah `show_in_portal: true`, karena bawaan baru `false` membuat pelanggan itu tersembunyi dan test lama menjadi merah.

  **Frontend**

  - `frontend/src/app/pages/users/customer/schema/columns.jsx`
    - Kolom baru **Portal** setelah kolom Status. Kolom ini memakai `StatusCell` yang memanggil `/customer/change-portal-visibility/` dan hanya aktif untuk pemegang `customer.changeSensitive`. Kolom bisa difilter aktif/nonaktif.
  - `frontend/src/app/pages/users/customer/profile.jsx`
    - Field `show_in_portal` di halaman profil pelanggan ditampilkan dengan `PortalBadge`, sama seperti profil mitra.
  - `frontend/src/constants/privilegeDescriptions.id.json`, `frontend/src/constants/privilegeDescriptions.en.json`
    - Tooltip checkbox untuk privilege `customer.changeSensitive` di halaman manajemen hak akses.

---

## 🌿 Branch: `issue-384` — Arsip Surat Keluar

### 📌 Informasi Issue

- **Nomor Issue**: #384
- **Judul Issue**: Menu Arsip > Surat Keluar
- **Status Branch**: `Belum di-merge` (sudah `resolve`, siap PR)

### 📅 Rincian Commit

#### [cef5b5f8] - resolve #384 - 10 Oktober 2026, 19:56

- **Ringkasan**: Menambah menu **Arsip > Surat Keluar** yang fungsinya sama dengan Arsip > Administrasi (arsip PDF unggahan dengan penomoran otomatis), tetapi koleksi, nomor urut, dan hak aksesnya terpisah. Supaya kodenya tidak disalin dua kali, seluruh logika Dokumen Administrasi diekstrak menjadi kode bersama (pabrik model, service, controller, penyaji berkas, dan halaman frontend) yang kini dipakai kedua modul.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Model**

  - `backend/src/models/shared/archiveDocument.model.js` [NEW]
    - Pabrik `createArchiveDocumentModel({ modelName, collection })` berisi schema dokumen arsip: judul, nomor dokumen, tanggal, kategori, keterangan, berkas PDF, pembuat/pengubah, dan `pid`.
    - Membawa index `pid + created_at`, index unik `document_number` yang hanya berlaku untuk dokumen belum terhapus (nomor dokumen terhapus bisa dipakai lagi), plugin soft delete, dan nomor urut otomatis (`seq`).
    - Mengekspor `ARCHIVE_DOCUMENT_TEXT_LIMITS` (batas panjang teks) yang dipakai bersama oleh schema dan validasi controller.
  - `backend/src/models/administrationDocument.model.js`
    - Schema yang sebelumnya ditulis lengkap (~85 baris) diganti satu panggilan `createArchiveDocumentModel` dengan koleksi tetap `administration_document`, jadi data lama tidak berubah.
  - `backend/src/models/outgoingLetter.model.js` [NEW]
    - Model `OutgoingLetter` di koleksi baru `outgoing_letter`, dibuat dari pabrik yang sama.

  **Backend — Service**

  - `backend/src/services/archiveDocument.service.js` [NEW]
    - Pabrik `createArchiveDocumentService` berisi `findById`, `findAllForTable` (datatable), `create`, `update`, `deleteById` (soft delete), `nextCount` (nomor urut berikutnya), dan `findCategorySelect` (autocomplete kategori dari nilai yang pernah dipakai).
    - Error dipetakan dengan rapi: nomor dokumen duplikat menjadi 409, pelanggaran schema menjadi 400, dan error lain menjadi 500 setelah dicatat ke log.
  - `backend/src/services/administrationDocument.service.js`
    - Isinya (~240 baris) diganti pengikatan ke `createArchiveDocumentService`. Nama fungsi yang diekspor tetap sama, jadi pemanggil lama tidak berubah.
  - `backend/src/services/outgoingLetter.service.js` [NEW]
    - Service Surat Keluar, berupa pengikatan model `OutgoingLetter` dan teks i18n `outgoingLetter.*` ke pabrik yang sama.

  **Backend — Controller & Route**

  - `backend/src/controllers/archiveDocument.controller.js` [NEW]
    - Pabrik `createArchiveDocumentController({ service, i18nPrefix, filePrefix, context })` berisi handler `create`, `listAll`, `read`, `update`, `remove`, `previewNumber`, dan `categorySelect`.
    - Validasi unggahan: wajib PDF, maksimal 15 MB, dan harus bisa dibaca jumlah halamannya. Nama berkas disimpan sebagai `{filePrefix}-{waktu}_{acak}.pdf`.
    - Penomoran otomatis `{urutan}/{kode}/{bulanRomawi}/{tahun}`. Nomor urut diambil saat menyimpan, bukan dari pratinjau, supaya dua pengunggah bersamaan tidak mendapat nomor yang sama. Nomor manual tetap didukung.
    - Validasi kode dokumen, panjang teks, dan tanggal dijalankan sebelum berkas diunggah, supaya tidak ada berkas yatim di storage.
    - Berkas baru dihapus bila penyimpanan gagal, dan berkas lama dihapus setelah diganti. Penghapusan dokumen bersifat soft delete, dan PDF-nya sengaja tetap disimpan agar arsip bisa dipulihkan.
  - `backend/src/controllers/administrationDocument.controller.js`
    - Isinya (~420 baris) diganti pembungkus tipis yang mengikat service, prefix i18n `administrationDocument`, dan prefix berkas `administrasi` ke pabrik controller. Nama handler yang diekspor tetap sama.
  - `backend/src/controllers/outgoingLetter.controller.js` [NEW]
    - Controller Surat Keluar dengan prefix i18n `outgoingLetter` dan prefix berkas `surat-keluar`.
  - `backend/src/routes/outgoingLetter.route.js` [NEW]
    - Tujuh endpoint, masing-masing dengan privilege dan Swagger: `POST /outgoing-letter/create`, `GET /next-number`, `POST /category-select`, `POST /list-all`, `GET /view/:document_id`, `PATCH /update/:document_id`, `DELETE /delete/:document_id`.
  - `backend/src/app.js`
    - Mendaftarkan `OutgoingLetterRoute` di `/api/v1`.

  **Backend — Penyaji Berkas**

  - `backend/src/controllers/files.controller.js`
    - Pabrik baru `createArchiveFileHandler({ filePrefix, notFoundKey, context })` yang mengalirkan PDF dari MinIO dan membatasi nama berkas ke pola milik modulnya. Dengan begitu privilege `.read` satu modul tidak bisa dipakai membaca berkas modul lain.
    - `getAdministrationDocumentFile` kini dibuat dari pabrik ini, dan ditambah `getOutgoingLetterFile` (pola `surat-keluar-*`).
  - `backend/src/routes/files.route.js`
    - Route baru `GET /file/outgoing-letter/:name` dengan `checkPrivilege('outgoingLetter.read')`.

  **Backend — Privilege, i18n & Test**

  - `backend/src/config/privilege.json`
    - Grup privilege baru `outgoingLetter`: `list`, `read`, `create`, `update`, `delete`.
  - `backend/src/config/privilegeDictionary.json`
    - Entri kamus untuk kelima privilege `outgoingLetter.*`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Pesan validasi yang dipakai bersama (kode dokumen, tanggal, tanpa berkas, bukan PDF, terlalu besar) dipindah dari `administrationDocument.*` ke grup baru `archiveDocument.*`.
    - Grup baru `outgoingLetter.*` berisi pesan khusus Surat Keluar (tidak ditemukan, berhasil diunggah/diperbarui, nomor duplikat, dsb).
  - `backend/test/integration/outgoingLetter.test.js` [NEW]
    - Unggahan masuk ke koleksi sendiri dengan nama berkas `surat-keluar-*`.
    - Nomor urut terpisah dari Dokumen Administrasi, dan nomor yang sama boleh dipakai di kedua modul.
    - Nomor duplikat sesama Surat Keluar ditolak 409.
    - Dokumen Administrasi tidak bisa dibaca, diubah, atau dihapus lewat endpoint Surat Keluar (404).
    - Penyaji berkas Surat Keluar menolak berkas Administrasi dan melayani berkas `surat-keluar-*`.

  **Frontend — Komponen Bersama (`archive/uploadedDocument/`)**

  - `frontend/src/app/pages/archive/uploadedDocument/configs.js` [NEW]
    - Konfigurasi per jenis dokumen (`ADMINISTRATION_DOCUMENT_CONFIG`, `OUTGOING_LETTER_CONFIG`): `id`, URL API, URL berkas, key privilege (identik dengan backend), dan key label i18n. Semua komponen di folder ini hanya mengambil nilai khusus modul dari sini.
  - `frontend/src/app/pages/archive/uploadedDocument/UploadedDocumentPage.jsx` [NEW]
    - Halaman generik berisi tabel, tombol unggah, drawer, dan pratinjau, digerakkan oleh `config`. Tombol dan aksi dicek dengan `useHasPrivilege` sesuai privilege di `config`.
  - `frontend/src/app/pages/archive/uploadedDocument/UploadedDocumentDrawer.jsx` (rename dari `administration/AdministrationDocumentDrawer.jsx`)
    - Drawer unggah dan ubah kini menerima `config`. URL API, id form, autocomplete kategori, dan teks toast diambil dari konfigurasi.
  - `frontend/src/app/pages/archive/uploadedDocument/UploadedDocumentPreviewModal.jsx` (rename dari `administration/AdministrationDocumentPreviewModal.jsx`)
    - Modal pratinjau PDF (semua halaman, unduh, cetak) kini memakai `config.fileBase` dan label dari konfigurasi.
  - `frontend/src/app/pages/archive/uploadedDocument/downloadUploadedDocument.js` (rename dari `administration/downloadAdministrationFile.js`)
    - Fungsi unduh kini menerima `fileBase`, jadi bisa dipakai kedua modul.
  - `frontend/src/app/pages/archive/uploadedDocument/getUploadedDocumentEditDrawer.jsx` [NEW]
    - Membuat komponen pembungkus drawer Ubah per `config.id` dan menyimpannya di cache. `RowActions` hanya meneruskan props standar, dan identitas komponen harus stabil antar-render supaya drawer tidak remount.
  - `frontend/src/app/pages/archive/uploadedDocument/schema/columns.jsx` (rename dari `administration/schema/columns.jsx`)
    - Kolom tabel kini menerima `config`. Privilege aksi Lihat/Ubah/Hapus dan URL hapus diambil dari konfigurasi, tidak lagi tertulis `administrationDocument.*`.
  - `frontend/src/app/pages/archive/uploadedDocument/schema/uploadedDocumentSchema.js` (rename dari `administration/schema/administrationSchema.js`)
    - Schema Yup diganti nama menjadi `uploadedDocumentSchema`, dan pesan validasinya memakai key bersama `archiveDocument.*`.

  **Frontend — Halaman, Navigasi & Router**

  - `frontend/src/app/pages/archive/administration/index.jsx`
    - Halaman Administrasi (~90 baris) kini cukup `<UploadedDocumentPage config={ADMINISTRATION_DOCUMENT_CONFIG} />`.
  - `frontend/src/app/pages/archive/outgoingLetter/index.jsx` [NEW]
    - Halaman Surat Keluar: `<UploadedDocumentPage config={OUTGOING_LETTER_CONFIG} />`.
  - `frontend/src/app/navigation/archive.js`
    - Item menu baru **Surat Keluar** (`/archive/outgoing-letter`, ikon `PaperAirplaneIcon`) yang hanya tampil untuk pemegang `outgoingLetter.list`.
  - `frontend/src/app/router/protected.jsx`
    - Route lazy baru `archive/outgoing-letter`.
  - `frontend/src/app/pages/archive/ArchiveIndexRedirect.jsx`
    - Saat membuka `/archive`, user yang hanya memegang `outgoingLetter.list` kini diarahkan ke halaman Surat Keluar.
  - `frontend/src/components/shared/table/rows.jsx`
    - `AdministrationDocumentNumberCell` diganti nama menjadi `UploadedDocumentNumberCell` dan menerima `readPrivilege`. Klik nomor dokumen membuka pratinjau bila user punya hak baca modul tersebut.

  **Frontend — Privilege & i18n**

  - `frontend/src/constants/privilegeDescriptions.id.json`, `frontend/src/constants/privilegeDescriptions.en.json`
    - Tooltip untuk kelima privilege `outgoingLetter.*`.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Key menu `nav.archive.outgoingLetter`.
    - Teks form yang dipakai bersama (kode dokumen, petunjuk penomoran, unggah berkas, dsb) dipindah ke grup baru `archiveDocument.*`. Grup `administrationDocument.*` tinggal berisi teks khusus Administrasi.
    - Grup baru `outgoingLetter.*` (daftar, unggah, ubah, berhasil, pratinjau).

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #394 | Toggle Portal untuk Pelanggan | Admin bisa mengatur pelanggan reguler mana yang terlihat di aplikasi mitra (p-api) |
| #384 | Arsip Surat Keluar | Menu baru Arsip > Surat Keluar + refactor modul arsip dokumen unggahan menjadi komponen bersama |

### Kemampuan Baru Pengguna/Admin

- Admin dengan hak `customer.changeSensitive` dapat menyalakan/mematikan visibilitas pelanggan di portal mitra langsung dari tabel Pelanggan.
- Admin dengan hak `outgoingLetter.*` dapat mengunggah, melihat, mengubah, menghapus, dan mengunduh arsip PDF surat keluar dengan penomoran otomatis.

### Bug Fix / Solusi Masalah

- Menutup celah di Partner API: data mitra/pelanggan yang disembunyikan dari portal sebelumnya masih bisa diubah, dihapus, atau diganti statusnya lewat endpoint tulis.
- Toggle visibilitas pelanggan dibuat atomik (satu query), menghindari race condition yang ada pada pola toggle mitra.

### Menu/Fitur Baru

- **Arsip > Surat Keluar** (`/archive/outgoing-letter`).
- Kolom **Portal** di tabel **Pengguna > Pelanggan**.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur (Toggle Portal Pelanggan)**: Setiap pelanggan reguler kini punya status "Portal". Bila mati, pelanggan tersebut (beserta akun radiusnya) tidak terlihat oleh admin yang memakai aplikasi mitra. Bawaannya **mati**, jadi setelah rilis semua pelanggan reguler lama tersembunyi dari aplikasi mitra sampai dinyalakan. Akses aplikasi mobile pelanggan tidak berubah, dan pelanggan milik mitra tidak terpengaruh.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Pengguna > Pelanggan**.
  2. Pada kolom **Portal**, klik toggle di baris pelanggan yang ingin ditampilkan di aplikasi mitra.
  3. Status juga terlihat di halaman profil pelanggan.

- **Penjelasan Fitur (Surat Keluar)**: Tempat menyimpan arsip PDF surat keluar perusahaan, terpisah dari Dokumen Administrasi.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Arsip > Surat Keluar**.
  2. Klik tombol tambah, isi data surat (nomor terisi otomatis, kategori), lalu unggah PDF-nya.
  3. Gunakan aksi baris untuk melihat pratinjau, mengunduh, mengubah, atau menghapus arsip.
