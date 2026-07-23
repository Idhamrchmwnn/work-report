# 📝 Daily Work Report - Idham (2026-06-19)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Vendor Purchase Order (PO) Management — Pengembangan backend modul Sales Order (SO): model, service, controller, routes, file serving, digital signature embedding, notifikasi Telegram, dan dependensi baru

---

## 📅 Laporan Harian - 19 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Branch: `issue-104` — tidak ada commit baru hari ini

---

#### File Baru (Untracked)

- [backend/src/models/vendorSO.model.js](backend/src/models/vendorSO.model.js) [NEW]
  - **Deskripsi**: Mongoose model `VendorSO` — koleksi MongoDB `vendor_so`. Field: `vendor` (ref Vendor, required), `so_number` (String, required, unique), `so_date` (Date), `document_path` (path file di MinIO), `document_name` (nama asli file), `notes`, `signed` (Boolean, default false), `signed_by` (ref Admin), `signed_at`, `submitted_at`, `created_by` (ref Admin, required), `pid` (default `'master'`), `created_at`.

- [backend/src/utils/sign-document.js](backend/src/utils/sign-document.js) [NEW]
  - **Deskripsi**: Utility untuk embed tanda tangan digital ke dalam dokumen SO. Dua fungsi:
    - `embedSignatureIntoPDF(pdfBuffer, signatureDataUrl, position)` — menggunakan `pdf-lib` untuk embed gambar PNG tanda tangan ke halaman PDF pada posisi persentase (x, y, width, height). Mendukung multi-halaman; koordinat dikonversi dari sistem origin kiri-atas (browser canvas) ke kiri-bawah (PDF).
    - `embedSignatureIntoImage(imageBuffer, signatureDataUrl, position)` — menggunakan `sharp` untuk composite gambar tanda tangan PNG di atas foto/gambar pada posisi persentase.

---

#### File Dimodifikasi

**Backend:**

- [backend/src/controllers/vendor.controller.js](backend/src/controllers/vendor.controller.js) [+162]
  - **Deskripsi**: Penambahan seluruh SO controllers:
    - `createSO` — validasi vendor, cek ukuran & tipe file dokumen SO (max 20 MB, PDF/gambar), upload ke MinIO bucket `appFiles`, simpan ke DB via `createNewVendorSO`.
    - `listSO` — daftar SO per vendor.
    - `listAllSO` — daftar semua SO (untuk tabel Activation), support DataTable params.
    - `readSO` — detail satu SO by ID.
    - `submitSO` — update `submitted_at`, kirim notifikasi Telegram via `TelegramNotifSOSubmit`.
    - `signSO` — sign SO dengan optional embed tanda tangan digital ke dokumen: download file asli dari MinIO, panggil `embedSignatureIntoPDF` atau `embedSignatureIntoImage` sesuai ekstensi, upload file baru yang sudah ditandatangani, update DB dengan `signed: true`, `signed_by`, `signed_at`, dan path dokumen baru.
    - `rejectSO` — tolak (hapus) SO yang belum ditandatangani.
    - `deleteSO` — hapus SO beserta file dokumennya dari MinIO.

- [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js) [+35]
  - **Deskripsi**: Penambahan 3 item:
    - `downloadAppFile(filename)` — stream file dari MinIO bucket `appFiles` (helper untuk sign-document).
    - `uploadAppBase64(filename, base64)` — upload file dalam format base64 ke MinIO bucket `appFiles` (digunakan untuk menyimpan dokumen SO yang sudah di-embed tanda tangan).
    - `getVendorSOFile` (endpoint handler) — stream file dokumen SO dari MinIO berdasarkan `id` SO. Mendukung query `?download=1` untuk force download, default inline (untuk preview di browser). Resolve content-type otomatis dari ekstensi file.

- [backend/src/routes/vendor.route.js](backend/src/routes/vendor.route.js) [+17]
  - **Deskripsi**: Penambahan 8 route SO:
    - `POST /vendor-so/create` (privilege: `vendor.create`)
    - `POST /vendor-so/list-all` (privilege: `vendor.read`)
    - `GET /vendor-so/list/:vendor_id` (privilege: `vendor.read`)
    - `GET /vendor-so/view/:id` (privilege: `vendor.read`)
    - `PATCH /vendor-so/submit/:id` (privilege: `vendor.update`)
    - `PATCH /vendor-so/sign/:id` (privilege: `vendor.changeStatus`)
    - `PATCH /vendor-so/reject/:id` (privilege: `vendor.changeStatus`)
    - `DELETE /vendor-so/delete/:id` (privilege: `vendor.delete`)

- [backend/src/routes/files.route.js](backend/src/routes/files.route.js) [+2]
  - **Deskripsi**: Penambahan route `GET /file/so-document/:id` → `getVendorSOFile`. Berbeda dengan PO yang menggunakan nama file, SO menggunakan ID dokumen sehingga lebih aman (tidak mengekspos path file langsung).

- [backend/src/services/vendor.service.js](backend/src/services/vendor.service.js) [+132]
  - **Deskripsi**: Penambahan seluruh SO service functions dengan constant `VENDOR_SO_FIELDS` untuk field projection:
    - `findVendorSOById(id)` — find SO by ID dengan populate vendor, created_by, signed_by.
    - `findAllSOsForTable(params)` — DataTable query dengan populate vendor, created_by, signed_by; filter `pid: 'master'`.
    - `findSOsByVendor(vendorId)` — list SO per vendor, sort by created_at desc.
    - `createNewVendorSO(data)` — insert SO baru dengan error handling `extractError`.
    - `signVendorSO(id, adminId, newDocumentPath)` — update signed fields, opsional update document_path jika ada dokumen yang di-embed tanda tangan.
    - `submitVendorSO(id)` — update `submitted_at`.
    - `deleteVendorSOById(id)` — hapus SO dari DB.

- [backend/src/utils/telegram.js](backend/src/utils/telegram.js) [+41]
  - **Deskripsi**: Penambahan fungsi `TelegramNotifSOSubmit(so, user, targetChatId)` — mengirim notifikasi Telegram ke grup approval (`telegram_approval` dari AppSettings) saat SO di-submit. Format pesan: nomor SO, nama vendor, tanggal SO, nama yang mengajukan, waktu pengajuan, dan link langsung ke halaman detail SO di web (`WEB_URL/services/vendor/so/:id`).

- [backend/src/locales/en/translation.json](backend/src/locales/en/translation.json) [+10]
- [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json) [+10]
  - **Deskripsi**: Penambahan key backend locale untuk `vendor.so.*`: `failedGetList`, `notFound`, `deleteFailed`, `editFailed`, `alreadySigned`, `cannotReject`, `fileTooLarge`, `invalidFileType`.

- [backend/package.json](backend/package.json) [+1]
  - **Deskripsi**: Penambahan dependensi `pdf-lib: ^1.17.1` untuk manipulasi/embed tanda tangan ke dokumen PDF.

- [frontend/package.json](frontend/package.json) [+1]
  - **Deskripsi**: Penambahan dependensi `pdfjs-dist: ^3.11.174` untuk rendering/preview dokumen PDF di frontend.

**Frontend (lanjutan dari kemarin):**

- [frontend/src/app/pages/services/vendor/detail.jsx](frontend/src/app/pages/services/vendor/detail.jsx) [+162]
  - **Deskripsi**: Integrasi SO di halaman Detail Vendor: state SO management, handler `handleSOReview`, `handleCloseSODrawer`, `handleSOSign`, `handleSOReject`, `handleDeleteSO`, `fetchSOList`, import `SOSignStatusCell` dan `SOReviewDrawer`.

- [frontend/src/app/pages/services/activation/index.jsx](frontend/src/app/pages/services/activation/index.jsx) [+68]
  - **Deskripsi**: Tab SO di halaman Activation, integrasi `SOReviewDrawer` dengan handler sign/reject.

- [frontend/src/app/pages/services/vendor/schema/createSchema.js](frontend/src/app/pages/services/vendor/schema/createSchema.js) [+11]
  - **Deskripsi**: Yup schema `vendorSOSchema` untuk form create SO.

- [frontend/src/app/router/services/vendorRoute.jsx](frontend/src/app/router/services/vendorRoute.jsx) [+18]
  - **Deskripsi**: Route `so/create` dan `so/:id` untuk modul SO.

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) [+62]
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) [+60]
  - **Deskripsi**: Seluruh translation key `vendor.so.*`, `activation.*SO*`, `form.soDate`, `form.soNumber`.

- [frontend/.env.development](frontend/.env.development)
  - **Deskripsi**: `VITE_API_URL` diubah ke `http://127.0.0.1:3000` untuk testing lokal. (Tidak perlu di-commit.)

**Untracked (frontend — lanjutan):**

- [frontend/src/app/pages/services/vendor/so/create.jsx](frontend/src/app/pages/services/vendor/so/create.jsx) [NEW, 179 baris] — Form create SO.
- [frontend/src/app/pages/services/vendor/so/detail.jsx](frontend/src/app/pages/services/vendor/so/detail.jsx) [NEW, 342 baris] — Halaman detail SO.
- [frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx](frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx) [NEW, 173 baris] — Drawer review SO.
- [frontend/src/app/pages/services/activation/schema/soColumns.jsx](frontend/src/app/pages/services/activation/schema/soColumns.jsx) [NEW, 128 baris] — Kolom tabel SO + `SOSignStatusCell`.

---

### 📅 Rincian Commit

> Tidak ada commit baru hari ini (19 Juni 2026). Seluruh pekerjaan masih WIP.

Commit terakhir terkait issue ini: `78f2532` (17 Juni 2026, 15:41 WIB).

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**:
  - Backend modul SO sekarang lengkap — semua CRUD SO (create, read, list, submit, sign, reject, delete) sudah tersedia sebagai REST API dengan proteksi autentikasi dan privilege.
  - Admin dapat **menandatangani dokumen SO secara digital**: saat sign SO, sistem otomatis embed gambar tanda tangan ke file PDF atau foto SO yang sudah diupload vendor, lalu menyimpan versi dokumen baru yang sudah ditandatangani di MinIO.
  - Sistem mengirim **notifikasi Telegram otomatis** ke grup approval ketika SO di-submit, lengkap dengan nomor SO, nama vendor, dan link langsung ke halaman detail.
  - File dokumen SO dapat dilihat secara inline di browser (PDF/gambar) via endpoint `/file/so-document/:id`, atau diunduh dengan menambahkan `?download=1`.

- **Arsitektur & Teknis**:
  - Model `VendorSO` dengan field `signed`, `signed_by`, `signed_at`, `submitted_at` — alur status: created → submitted → signed.
  - Utility `sign-document.js` reusable untuk PDF (via `pdf-lib`) dan gambar (via `sharp`) — mendukung posisi tanda tangan berbasis persentase (koordinat relatif terhadap ukuran halaman/gambar).
  - `uploadAppBase64` helper baru di `files.controller.js` untuk upload hasil signed document yang sudah ada di memori sebagai Buffer/base64.
  - Dependensi baru: `pdf-lib` (backend) untuk manipulasi PDF, `pdfjs-dist` (frontend) untuk render PDF.
