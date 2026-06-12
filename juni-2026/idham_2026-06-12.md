# 📝 Daily Work Report - Idham (2026-06-12)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Vendor Purchase Order (PO) Management — Pengembangan lanjutan fitur Purchase Order: CRUD lengkap PO, upload lampiran, approval workflow, preview dokumen, dan notifikasi Telegram

---

## 📅 Laporan Harian - 12 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Branch: `issue-104` — semua perubahan masih WIP, belum di-commit hari ini

**Backend:**

- [backend/src/models/vendorPO.model.js](backend/src/models/vendorPO.model.js) [NEW]
  - **Deskripsi**: Model Mongoose lengkap untuk entitas `VendorPO`. Mencakup sub-schema `LineItemSchema` (item, tipe: service/goods/contractor, qty, unit_price, subtotal) dan `AttachmentSchema` (file lampiran PO). Field utama meliputi: `po_number` (unique), `status` (draft/pending/approved/rejected), `po_date`, `delivery_date`, `payment_method`, `currency`, `quotation_number`, `tax_type`, `tax_amount`, `grand_total`, `line_items`, `attachments`, serta field audit (submitted_by, approved_by, rejected_by + timestamp masing-masing).

- [backend/src/services/vendor.service.js](backend/src/services/vendor.service.js)
  - **Deskripsi**: Penambahan 8 fungsi service baru untuk Vendor PO: `findVendorPOById`, `findPOsByVendor`, `createNewVendorPO`, `updateVendorPOData`, `updateVendorPOStatus`, `deleteVendorPOById`, `addAttachmentToPO`, `removeAttachmentFromPO`.

- [backend/src/controllers/vendor.controller.js](backend/src/controllers/vendor.controller.js)
  - **Deskripsi**: Penambahan controller lengkap untuk Vendor PO: `createPO` (generate `po_number` otomatis format `PO-YYYYMM-XXXXXX`), `listPO`, `readPO`, `submitPO` (ubah status ke pending + trigger Telegram), `approvePO` (ubah status ke approved + trigger Telegram), `rejectPO` (ubah status ke rejected + simpan alasan + trigger Telegram), `deletePO`, `updatePO`, `uploadPOAttachment` (upload file ke MinIO bucket `appFiles`), `deletePOAttachment` (hapus dari MinIO + hapus dari array attachments PO).

- [backend/src/routes/vendor.route.js](backend/src/routes/vendor.route.js)
  - **Deskripsi**: Registrasi 10 route baru untuk Vendor PO:
    - `POST /vendor-po/create`
    - `GET /vendor-po/list/:vendor_id`
    - `GET /vendor-po/view/:id`
    - `PATCH /vendor-po/submit/:id`
    - `PATCH /vendor-po/approve/:id` (privilege: `vendor.changeStatus`)
    - `PATCH /vendor-po/reject/:id` (privilege: `vendor.changeStatus`)
    - `DELETE /vendor-po/delete/:id`
    - `PATCH /vendor-po/update/:id`
    - `POST /vendor-po/attachment/:id`
    - `DELETE /vendor-po/attachment/:id/:filename`

- [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js)
  - **Deskripsi**: Penambahan 2 fungsi baru: `removeAppFile` (helper untuk menghapus file dari MinIO bucket `appFiles` secara silent) dan `getVendorPOFile` (endpoint streaming file lampiran PO dari MinIO dengan support inline/download mode via query `?download`).

- [backend/src/routes/files.route.js](backend/src/routes/files.route.js)
  - **Deskripsi**: Penambahan route `GET /file/vendor-po/:name` untuk mengakses file lampiran PO, dilindungi privilege `vendor.read`.

- [backend/src/utils/minio.js](backend/src/utils/minio.js)
  - **Deskripsi**: Perbaikan error handling di fungsi `minioUpload`: catch block sekarang menangkap error object dan mencetaknya ke console (`[minioUpload]` + detail error) sebelum melempar error, sehingga debugging upload gagal menjadi lebih mudah.

- [backend/src/utils/telegram.js](backend/src/utils/telegram.js)
  - **Deskripsi**: Penambahan 3 fungsi notifikasi Telegram untuk workflow PO: `TelegramNotifPOSubmit` (PO menunggu persetujuan), `TelegramNotifPOApproved` (PO disetujui), `TelegramNotifPORejected` (PO ditolak + alasan). Notifikasi dikirim ke chat ID `telegram_approval` dari App Settings, berformat HTML dengan link langsung ke dokumen PO.

- [backend/src/config/privilege.json](backend/src/config/privilege.json)
  - **Deskripsi**: Penambahan privilege baru `vendor.changeStatus` untuk membatasi aksi approve/reject PO hanya pada user dengan wewenang tertentu.

- [backend/src/app.js](backend/src/app.js)
  - **Deskripsi**: Registrasi route vendor ke aplikasi Express.

- [backend/src/models/document.model.js](backend/src/models/document.model.js)
  - **Deskripsi**: Penyesuaian model dokumen untuk mendukung attachment pada PO.

- [backend/src/models/vendorService.model.js](backend/src/models/vendorService.model.js)
  - **Deskripsi**: Penyesuaian field model VendorService mengikuti perubahan terminologi item.

**Frontend:**

- [frontend/src/app/pages/services/vendor/po/create.jsx](frontend/src/app/pages/services/vendor/po/create.jsx) [NEW, 532 baris]
  - **Deskripsi**: Halaman Create Purchase Order lengkap: form input data PO (vendor, nomor PO, tanggal, delivery date, payment method, quotation number), manajemen line item (tambah/hapus/edit item dengan kalkulasi subtotal otomatis), pengaturan pajak (none/percentage/fixed), field terms & notes, serta kalkulasi grand total real-time.

- [frontend/src/app/pages/services/vendor/po/detail.jsx](frontend/src/app/pages/services/vendor/po/detail.jsx) [NEW, 419 baris]
  - **Deskripsi**: Halaman Detail PO menampilkan informasi lengkap PO, status badge (draft/pending/approved/rejected), tabel line item, lampiran dokumen (upload & delete), preview dokumen PO, serta tombol aksi: Submit, Approve, Reject (dengan dialog alasan), Edit, Delete.

- [frontend/src/app/pages/services/vendor/po/EditPODrawer.jsx](frontend/src/app/pages/services/vendor/po/EditPODrawer.jsx) [NEW, 335 baris]
  - **Deskripsi**: Drawer form untuk mengedit data PO yang sudah ada (status masih draft), dengan field dan kalkulasi yang sama seperti form create.

- [frontend/src/app/pages/services/vendor/CreatePODrawer.jsx](frontend/src/app/pages/services/vendor/CreatePODrawer.jsx) [NEW, 216 baris]
  - **Deskripsi**: Komponen Drawer untuk membuat PO langsung dari halaman detail vendor, memudahkan pembuatan PO tanpa pindah halaman.

- [frontend/src/app/pages/services/vendor/PODocumentPreview.jsx](frontend/src/app/pages/services/vendor/PODocumentPreview.jsx) [NEW, 311 baris]
  - **Deskripsi**: Komponen preview dokumen PO bergaya formal/cetak: header perusahaan, info vendor, tabel line item dengan subtotal per baris, ringkasan pajak, grand total, dan seksi lampiran. Mendukung tampilan kontak AM/NOC kondisional berdasarkan toggle di data PO.

- [frontend/src/app/pages/services/vendor/vendorServiceDetail.jsx](frontend/src/app/pages/services/vendor/vendorServiceDetail.jsx) [NEW, 183 baris]
  - **Deskripsi**: Halaman detail terpisah untuk satu Vendor Item/Service, dipisahkan dari `detail.jsx` vendor utama agar lebih modular dan fokus.

- [frontend/src/app/pages/services/vendor/detail.jsx](frontend/src/app/pages/services/vendor/detail.jsx)
  - **Deskripsi**: Refaktorisasi halaman detail vendor: terminologi "service" diubah menjadi "item", opsi tipe diperbarui (service/goods/contractor), ditambahkan tombol navigasi ke halaman PO dan integrasi komponen baru (`Modal`, `CreatePODrawer`).

- [frontend/src/app/pages/services/vendor/schema/createSchema.js](frontend/src/app/pages/services/vendor/schema/createSchema.js)
  - **Deskripsi**: Pembaruan Yup validation schema untuk form vendor item.

- [frontend/src/app/router/services/vendorRoute.jsx](frontend/src/app/router/services/vendorRoute.jsx)
  - **Deskripsi**: Penambahan 3 route baru: `vendor/item/:id` (detail item), `vendor/view/:vendorId/po/create` (create PO), `vendor/po/:id` (detail PO), masing-masing dengan privilege guard.

- [frontend/src/components/ui/Modal.jsx](frontend/src/components/ui/Modal.jsx) [NEW, 40 baris]
  - **Deskripsi**: Komponen Modal reusable berbasis Headless UI `Dialog`, digunakan untuk preview dokumen PO dan dialog konfirmasi. Diekspor melalui barrel `components/ui/index.js`.

- [frontend/src/components/ui/index.js](frontend/src/components/ui/index.js)
  - **Deskripsi**: Penambahan ekspor komponen `Modal` baru.

- [frontend/src/components/shared/ConfirmModal.jsx](frontend/src/components/shared/ConfirmModal.jsx)
  - **Deskripsi**: Peningkatan komponen ConfirmModal dengan dukungan input teks tambahan (digunakan untuk field alasan penolakan PO).

- [frontend/src/middleware/AuthGuard.jsx](frontend/src/middleware/AuthGuard.jsx)
  - **Deskripsi**: Perbaikan race condition di AuthGuard: ditambahkan state `privilegeReady` agar komponen menunggu fetch privilege selesai sebelum mengecek hak akses, mencegah redirect palsu ke halaman unauthorized saat privilege belum ter-load.

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) & [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: Penambahan translation key untuk seluruh fitur Vendor PO (label form, status, pesan sukses/error, label tabel, dsb.) dalam Bahasa Inggris dan Indonesia.

**Telegram API:**

- [telegram-api/src/bots.json](telegram-api/src/bots.json)
  - **Deskripsi**: Pembaruan konfigurasi bot Telegram untuk environment development/testing.

---

### 📅 Rincian Commit

> Tidak ada commit baru hari ini (12 Juni 2026) — seluruh pekerjaan masih dalam status Work in Progress.

Commit terakhir terkait issue ini:

#### [b6feb54] - save #104 (#104 - Vendor PO Management)
*10 Juni 2026*

- Implementasi awal modul Vendor lengkap: model Vendor & VendorService, CRUD backend, halaman list/create/detail/edit frontend, navigasi, privilege (+2.546 baris, 21 file).

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**:
  - Admin dapat membuat Purchase Order (PO) untuk vendor dengan auto-generate nomor PO (`PO-YYYYMM-XXXXXX`), mengisi line item (service/goods/contractor), mengatur pajak, dan melihat grand total secara real-time.
  - Admin dapat mengupload dan menghapus lampiran dokumen (PDF, gambar, dsb.) pada setiap PO.
  - Admin dapat melihat preview dokumen PO bergaya formal langsung di browser sebelum mencetak atau mengirim.
  - Admin dapat mengubah status PO melalui alur: Draft → Submit → Approve/Reject.
  - Admin approver dapat menyetujui atau menolak PO (dengan wajib mengisi alasan penolakan) menggunakan privilege `vendor.changeStatus`.
  - Admin dapat mengakses file lampiran PO secara inline di browser maupun mengunduhnya.

- **Bug Fix / Solusi Masalah**:
  - **AuthGuard race condition**: Halaman yang membutuhkan privilege tidak lagi salah redirect ke unauthorized saat privilege belum selesai di-fetch saat pertama kali load.
  - **MinIO upload debugging**: Error upload ke MinIO kini mencetak detail error ke console sehingga memudahkan diagnosa masalah upload di production.

- **Menu/Tombol Baru**:
  - Tombol **Buat PO** tersedia di halaman Detail Vendor untuk memulai Purchase Order langsung dari konteks vendor.
  - Halaman **Detail PO** (`/services/vendor/po/:id`) dengan tombol aksi Submit, Approve, Reject, Edit, Delete, dan Upload Lampiran.
  - Notifikasi Telegram otomatis dikirim ke grup approval saat PO disubmit, disetujui, atau ditolak — dilengkapi link langsung ke dokumen PO di sistem.
