# 📝 Daily Work Report - Idham (2026-06-21)

---

## 📌 Informasi Issue
- **Nomor Issue**: #118
- **Judul Issue**: Vendor & Sales Order (SO) Management — Penyelesaian modul SO backend+frontend, refactor alur persetujuan SO agar sesuai dengan PO, perbaikan tampilan preview dokumen SO, dan penyelesaian registrasi modul vendor base

---

## 📅 Laporan Harian - 21 Juni 2026

### ✅ Commit Hari Ini

#### `a4ec411` — save #118 (22:43 WIB)
> 26 files changed, 5306 insertions(+)

Commit ini berisi penyelesaian modul SO secara lengkap (backend + frontend) serta refactor alur persetujuan SO agar struktur datanya sama dengan PO.

---

**Backend:**

- [backend/src/models/vendorSO.model.js](backend/src/models/vendorSO.model.js) [NEW]
  - Model `VendorSO` (koleksi `vendor_so`). Field utama: `vendor`, `so_number` (unique), `so_date`, `document_path`, `document_name`, `notes`, `approval` (ObjectId ref Admin — admin yang menyetujui), `approved_at`, `complete`, `submitted_at`, `created_by`, `pid`, `created_at`.
  - Menggunakan `approval` (bukan `signed`/`signed_by`) agar struktur persetujuan konsisten dengan model VendorPO.

- [backend/src/services/vendor.service.js](backend/src/services/vendor.service.js) [+589 lines total]
  - Seluruh SO service: `findVendorSOById`, `findAllSOsForTable`, `findSOsByVendor`, `createNewVendorSO`, `approveVendorSO(id, adminId, newDocumentPath)`, `submitVendorSO`, `deleteVendorSOById`. Field projection `VENDOR_SO_FIELDS` mencakup `approval`, `approved_at`, `complete`.

- [backend/src/controllers/vendor.controller.js](backend/src/controllers/vendor.controller.js) [+664 lines total]
  - Seluruh SO controllers: `createSO`, `listSO`, `listAllSO`, `readSO`, `submitSO`, `approveSO`, `rejectSO`, `deleteSO`.
  - `approveSO`: Menerima opsional `signature` + `position` di body. Jika ada, signature admin di-embed ke dokumen SO (PDF via `pdf-lib`, gambar via `sharp`) sebelum SO ditandai approved. File baru yang sudah ada tanda tangan disimpan ke MinIO.

- [backend/src/utils/sign-document.js](backend/src/utils/sign-document.js) [NEW, +72 lines]
  - `embedSignatureIntoPDF(pdfBuffer, signatureDataUrl, position)` — embed PNG tanda tangan ke halaman PDF pada posisi persentase (koordinat browser dikonversi ke koordinat PDF origin kiri-bawah).
  - `embedSignatureIntoImage(imageBuffer, signatureDataUrl, position)` — composite PNG tanda tangan di atas gambar via `sharp`.

- [backend/src/utils/telegram.js](backend/src/utils/telegram.js) [+41 lines]
  - `TelegramNotifSOSubmit(so, user)` — Notifikasi Telegram ke grup approval saat SO di-submit: nomor SO, nama vendor, tanggal, nama pengaju, dan link langsung ke detail SO.

- [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js) [partial]
  - `getVendorSOFile` (endpoint `GET /file/so-document/:id`) — Stream file dokumen SO dari MinIO berdasarkan ID SO. Mendukung `?download=1` untuk force download, default inline.
  - `downloadAppFile`, `uploadAppBase64` — helper internal untuk read/write file MinIO as Buffer/base64 (digunakan oleh `approveSO` untuk signature embedding).

- [backend/src/routes/vendor.route.js](backend/src/routes/vendor.route.js) [+91 lines total]
  - 8 route SO: `POST /vendor-so/create`, `POST /vendor-so/list-all`, `GET /vendor-so/list/:vendor_id`, `GET /vendor-so/view/:id`, `PATCH /vendor-so/submit/:id`, `PATCH /vendor-so/approve/:id`, `PATCH /vendor-so/reject/:id`, `DELETE /vendor-so/delete/:id`.

- [backend/src/routes/files.route.js](backend/src/routes/files.route.js) [+5 lines]
  - Route `GET /file/so-document/:id` → `getVendorSOFile`.

- [backend/src/locales/en/translation.json](backend/src/locales/en/translation.json) [+35 lines]
- [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json) [+35 lines]
  - Key SO backend: `failedGetList`, `notFound`, `deleteFailed`, `editFailed`, `alreadyApproved`, `cannotReject`, `fileTooLarge`, `invalidFileType`.

- [backend/package.json](backend/package.json) [+1]
  - Dependensi baru: `pdf-lib: ^1.17.1`.

---

**Frontend:**

- [frontend/src/app/pages/services/activation/schema/soColumns.jsx](frontend/src/app/pages/services/activation/schema/soColumns.jsx) [NEW, +144 lines]
  - `SOApprovalStatusCell` — Status persetujuan SO di tabel, **sama persis dengan `POApprovalStatusCell`**: jika `approval` set → centang hijau + teks "Approved" + Avatar admin approver; jika belum disetujui & punya privilege → tombol Review; jika tidak punya privilege → icon pending.
  - `SOSignStatusCell` — alias backward-compat.
  - `getSOColumns(t, onReview)` — Kolom tabel: `created_at`, `so_number`, `vendor`, `grand_total`, `approval_status`, `actions`.

- [frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx](frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx) [NEW, +173 lines]
  - Drawer review SO di halaman Activation — **sama persis dengan `POReviewDrawer`**: menampilkan ringkasan SO, link ke detail, tombol **Approve** + **Reject** dengan ConfirmModal konfirmasi.

- [frontend/src/app/pages/services/activation/index.jsx](frontend/src/app/pages/services/activation/index.jsx) [+153 lines]
  - Tab SO di halaman Activation (tab ketiga). Handler: `handleSOReview`, `handleCloseSODrawer`, `handleSOApprove` (endpoint `/vendor-so/approve`), `handleSOReject`.

- [frontend/src/app/pages/services/vendor/detail.jsx](frontend/src/app/pages/services/vendor/detail.jsx) [NEW, +805 lines]
  - Halaman detail vendor yang sudah terintegrasi penuh dengan modul SO: list SO per vendor, `SOApprovalStatusCell` di tabel, `SOReviewDrawer` untuk approve/reject dari halaman vendor, handler `handleSOApprove`, `handleSOReject`, `handleDeleteSO`, `fetchSOList`.

- [frontend/src/app/pages/services/vendor/so/components/SOSignModal.jsx](frontend/src/app/pages/services/vendor/so/components/SOSignModal.jsx) [NEW, +434 lines]
  - Modal penandatanganan digital SO (2 langkah): **Step 1** — tampilkan dokumen SO (PDF render via `pdfjs-dist`, atau gambar), admin klik/seret untuk menentukan posisi tanda tangan; **Step 2** — gambar tanda tangan di canvas (`react-signature-canvas`). Submit ke endpoint `/vendor-so/approve/:id` dengan `signature` (PNG data URL) dan `position` (persentase koordinat).

- [frontend/src/app/pages/services/vendor/so/create.jsx](frontend/src/app/pages/services/vendor/so/create.jsx) [NEW, +179 lines]
  - Halaman Create SO: form input nomor SO, tanggal, catatan, upload dokumen SO (PDF/gambar via FilePond). Submit ke `/vendor-so/create`.

- [frontend/src/app/pages/services/vendor/so/detail.jsx](frontend/src/app/pages/services/vendor/so/detail.jsx) [NEW, +353 lines]
  - Halaman detail SO. Preview dokumen SO selalu tampil inline (iframe 700px, sama seperti PO). Tombol **Cetak** di header menggunakan `iframe.contentWindow.print()`. Tombol **Setujui** membuka `SOSignModal` (admin gambar tanda tangan). Tombol **Ajukan**, **Tolak**, **Hapus**. Info: `approvedBy`, `approvedAt` (konsisten dengan PO yang menampilkan `approvedBy`).

- [frontend/src/app/pages/services/vendor/schema/createSchema.js](frontend/src/app/pages/services/vendor/schema/createSchema.js) [+118 lines total]
  - Schema Yup `vendorSOSchema` untuk form create SO.

- [frontend/src/app/router/services/vendorRoute.jsx](frontend/src/app/router/services/vendorRoute.jsx) [+85 lines total]
  - Route `so/create` dan `so/:id` untuk modul SO, ditambahkan ke route vendor.

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) [+216 lines total]
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) [+221 lines total]
  - Seluruh key SO: `vendor.so.*` (approve, approved, approvedAt, approvedBy, pendingApproval, approvedSuccess, dll), `activation.confirmApproveSOTitle/Desc`, `activation.confirmRejectSOTitle/Desc`.

- [frontend/package.json](frontend/package.json) [+1]
  - Dependensi baru: `pdfjs-dist: ^3.11.174` untuk render PDF di SOSignModal.

---

### 🛠️ Pekerjaan Belum Di-commit / WIP

#### Staged (siap commit)

- [backend/src/app.js](backend/src/app.js) [+2]
  - Registrasi `VendorRoute` ke Express app — import dan `app.use('/api/v1', VendorRoute)`.

- [backend/src/config/privilege.json](backend/src/config/privilege.json) [+8]
  - Definisi privilege modul vendor: `VENDOR_LIST`, `VENDOR_READ`, `VENDOR_CREATE`, `VENDOR_UPDATE`, `VENDOR_DELETE`, `VENDOR_CHANGESTATUS`.

- [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js) [+21]
  - `removeAppFile(filename)` — helper hapus file dari MinIO bucket `appFiles`.
  - `getVendorPOFile` (endpoint `GET /file/vendor-po/:name`) — Stream file lampiran PO dari MinIO.

- [backend/src/models/vendor.model.js](backend/src/models/vendor.model.js) [NEW]
  - Mongoose model `Vendor` — field: `name`, `code`, `address`, `phone`, `email`, `pic_name`, `pic_phone`, `notes`, `pid`, `created_by`, `created_at`.

- [backend/src/models/vendorPO.model.js](backend/src/models/vendorPO.model.js) [NEW]
  - Mongoose model `VendorPO` dengan embedded `LineItemSchema` dan `AttachmentSchema`. Field approval: `approval` (ref Admin), `complete`, `share_token`, `signature`, `signed_at`.

- [backend/src/models/vendorService.model.js](backend/src/models/vendorService.model.js) [NEW]
  - Mongoose model `VendorService` — katalog item/layanan vendor dengan field: `name`, `code`, `type`, `unit`, `price`, `vendor`.

- [frontend/src/app/navigation/services.js](frontend/src/app/navigation/services.js) [+10]
  - Tambah menu navigasi **Vendor** (`nav.services.vendor`, role: `vendor.list`, icon: `MdOutlineStore`) ke sidebar modul Services.

- [frontend/src/app/pages/services/vendor/index.jsx](frontend/src/app/pages/services/vendor/index.jsx) [NEW, +51 lines]
  - Halaman daftar vendor — tabel via Datatables dengan kolom dari `getVendorColumns`.

- [frontend/src/app/pages/services/vendor/create.jsx](frontend/src/app/pages/services/vendor/create.jsx) [NEW, +226 lines]
  - Form create vendor: nama, kode, alamat, telepon, email, PIC, catatan.

- [frontend/src/app/pages/services/vendor/edit.jsx](frontend/src/app/pages/services/vendor/edit.jsx) [NEW, +266 lines]
  - Form edit vendor — pre-fill data dari API, submit PATCH.

- [frontend/src/app/pages/services/vendor/vendorServiceDetail.jsx](frontend/src/app/pages/services/vendor/vendorServiceDetail.jsx) [NEW, +183 lines]
  - Sub-halaman daftar item/layanan vendor (VendorService CRUD) di dalam halaman detail vendor.

- [frontend/src/app/pages/services/vendor/schema/columns.jsx](frontend/src/app/pages/services/vendor/schema/columns.jsx) [NEW, +208 lines]
  - Definisi kolom tabel vendor untuk Datatables.

- [frontend/src/app/pages/services/vendor/CreatePODrawer.jsx](frontend/src/app/pages/services/vendor/CreatePODrawer.jsx) [NEW, +216 lines]
  - Drawer create PO di dalam halaman detail vendor — form dengan line items dan upload lampiran.

- [frontend/src/app/pages/services/vendor/PODocumentPreview.jsx](frontend/src/app/pages/services/vendor/PODocumentPreview.jsx) [NEW, +306 lines]
  - Komponen preview dokumen PO yang bisa di-print (digunakan oleh `useReactToPrint` di PO detail).

- [frontend/src/app/pages/services/vendor/po/create.jsx](frontend/src/app/pages/services/vendor/po/create.jsx) [NEW, +527 lines]
  - Halaman Create PO: form lengkap dengan line items, tax, mata uang, tanggal pengiriman, lampiran.

- [frontend/src/app/pages/services/vendor/po/detail.jsx](frontend/src/app/pages/services/vendor/po/detail.jsx) [NEW, +412 lines]
  - Halaman Detail PO: info PO, line items table, preview dokumen PO yang bisa di-print. Tombol: Edit, Ajukan, Copy Link, Cetak, Setujui/Tolak, Hapus.

- [frontend/src/app/pages/services/vendor/po/EditPODrawer.jsx](frontend/src/app/pages/services/vendor/po/EditPODrawer.jsx) [NEW, +266 lines]
  - Drawer edit PO — update line items, tax, catatan.

- [frontend/src/app/router/protected.jsx](frontend/src/app/router/protected.jsx) [+2]
  - Import dan registrasi `vendorRoute` ke protected router.

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) [+3]
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) [+3]
  - Tambah key `nav.services.vendor`.

#### Untracked

- [frontend/src/app/pages/services/activation/components/POReviewDrawer.jsx](frontend/src/app/pages/services/activation/components/POReviewDrawer.jsx) [NEW]
  - Drawer review PO di halaman Activation: ringkasan PO, fitur kirim preview request ke admin lain (Combobox pilih admin → POST `/vendor-po/request-preview`), tombol Approve/Reject.

- [frontend/src/app/pages/services/activation/schema/poColumns.jsx](frontend/src/app/pages/services/activation/schema/poColumns.jsx) [NEW]
  - `POApprovalStatusCell` dan `getPOColumns` untuk tabel PO di Activation.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **Alur persetujuan SO konsisten dengan PO**: Field `approval` (ObjectId ref Admin) menggantikan `signed`/`signed_by` — admin yang menyetujui SO tercatat sebagai objek dengan avatar tampil di tabel status, sama persis dengan tampilan approval PO.
- **Tanda tangan digital tetap ada**: Saat admin klik Setujui di halaman detail SO, modal tanda tangan 2-langkah terbuka — admin memposisikan tanda tangan di dokumen lalu menggambar, hasilnya di-embed ke file PDF/gambar SO sebelum disimpan.
- **Preview dokumen SO**: Selalu tampil inline (iframe) tanpa perlu klik toggle. Tombol Cetak di header memanggil `iframe.contentWindow.print()`.
- **Menu Vendor di sidebar**: Modul vendor sekarang terdaftar di navigasi dan router — admin dapat mengakses daftar vendor, buat/edit vendor, kelola item/layanan, buat & lihat PO dan SO langsung dari sidebar.
- **Privilege vendor**: `VENDOR_LIST`, `VENDOR_READ`, `VENDOR_CREATE`, `VENDOR_UPDATE`, `VENDOR_DELETE`, `VENDOR_CHANGESTATUS` sudah terdaftar di `privilege.json` dan siap dikonfigurasi per role.
