# 📝 Daily Work Report - Idham (2026-06-11)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Vendor Purchase Order (PO) Management — Fitur pembuatan, pengelolaan, dan persetujuan Purchase Order dari vendor

---

## 📅 Laporan Harian - 11 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Branch: `issue-104` — 21 file berubah (+1.754 / -654 baris), belum di-commit

**Backend:**

- [backend/src/controllers/vendor.controller.js](backend/src/controllers/vendor.controller.js)
  - **Deskripsi**: Penambahan handler baru untuk CRUD Vendor PO (`findVendorPOById`, `findPOsByVendor`, `createNewVendorPO`, `updateVendorPOStatus`, `deleteVendorPOById`). Juga menambahkan import fungsi notifikasi Telegram untuk trigger saat PO disubmit, disetujui, atau ditolak.

- [backend/src/services/vendor.service.js](backend/src/services/vendor.service.js)
  - **Deskripsi**: Penambahan 5 fungsi service baru untuk pengelolaan VendorPO: find by ID, find by vendor, create, update status, dan delete. Setiap fungsi menggunakan model `VendorPO` dengan field selector yang sudah didefinisikan.

- [backend/src/models/vendorPO.model.js](backend/src/models/vendorPO.model.js) [NEW]
  - **Deskripsi**: Model Mongoose baru untuk entitas Purchase Order vendor, mencakup field seperti `po_number`, `po_date`, `line_items`, `grand_total`, status approval, dan relasi ke model Vendor.

- [backend/src/utils/telegram.js](backend/src/utils/telegram.js)
  - **Deskripsi**: Penambahan 3 fungsi notifikasi Telegram baru: `TelegramNotifPOSubmit`, `TelegramNotifPOApproved`, `TelegramNotifPORejected`. Notifikasi dikirim ke chat ID `telegram_approval` dari App Settings, berisi informasi lengkap PO (nomor, vendor, total, link dokumen).

- [backend/src/routes/vendor.route.js](backend/src/routes/vendor.route.js)
  - **Deskripsi**: Refaktorisasi dan penambahan route endpoint untuk Vendor PO (list, detail, create, update status, delete).

- [backend/src/models/vendorService.model.js](backend/src/models/vendorService.model.js)
  - **Deskripsi**: Penyesuaian model VendorService/Item mengikuti perubahan terminologi dari "service" menjadi "item".

- [backend/src/models/document.model.js](backend/src/models/document.model.js)
  - **Deskripsi**: Pembaruan model dokumen, kemungkinan untuk mendukung lampiran dokumen PO.

- [backend/src/config/privilege.json](backend/src/config/privilege.json)
  - **Deskripsi**: Penambahan privilege baru untuk aksi-aksi terkait Vendor PO (submit, approve, reject, delete).

- [backend/src/app.js](backend/src/app.js)
  - **Deskripsi**: Registrasi route baru Vendor PO ke aplikasi Express.

**Frontend:**

- [frontend/src/app/pages/services/vendor/CreatePODrawer.jsx](frontend/src/app/pages/services/vendor/CreatePODrawer.jsx) [NEW]
  - **Deskripsi**: Komponen Drawer baru untuk membuat Purchase Order. Berisi form input data PO termasuk pemilihan item, jumlah, harga satuan, dan kalkulasi total otomatis.

- [frontend/src/app/pages/services/vendor/PODocumentPreview.jsx](frontend/src/app/pages/services/vendor/PODocumentPreview.jsx) [NEW]
  - **Deskripsi**: Komponen preview dokumen PO dalam format yang siap cetak/PDF. Menampilkan header perusahaan, detail vendor, tabel line item, dan total keseluruhan (+301 baris).

- [frontend/src/app/pages/services/vendor/vendorServiceDetail.jsx](frontend/src/app/pages/services/vendor/vendorServiceDetail.jsx) [NEW]
  - **Deskripsi**: Komponen detail terpisah untuk Vendor Item/Service (+183 baris), dipisahkan dari halaman `detail.jsx` utama agar lebih modular.

- [frontend/src/app/pages/services/vendor/po/create.jsx](frontend/src/app/pages/services/vendor/po/create.jsx) [NEW]
  - **Deskripsi**: Halaman Create Purchase Order baru di sub-direktori `po/`.

- [frontend/src/app/pages/services/vendor/po/detail.jsx](frontend/src/app/pages/services/vendor/po/detail.jsx) [NEW]
  - **Deskripsi**: Halaman Detail Purchase Order dengan tampilan status approval dan aksi approve/reject.

- [frontend/src/components/ui/Modal.jsx](frontend/src/components/ui/Modal.jsx) [NEW]
  - **Deskripsi**: Komponen Modal reusable baru yang digunakan untuk preview dokumen PO dan konfirmasi aksi.

- [frontend/src/app/pages/services/vendor/detail.jsx](frontend/src/app/pages/services/vendor/detail.jsx)
  - **Deskripsi**: Refaktorisasi halaman detail vendor: terminologi "service" diubah menjadi "item", opsi tipe diperbarui (`service`, `goods`, `contractor`), ditambahkan tombol aksi PO dan integrasi komponen baru.

- [frontend/src/app/router/services/vendorRoute.jsx](frontend/src/app/router/services/vendorRoute.jsx)
  - **Deskripsi**: Penambahan route baru untuk halaman PO (`/services/vendor/po/:id`, `/services/vendor/po/create`).

- [frontend/src/components/shared/ConfirmModal.jsx](frontend/src/components/shared/ConfirmModal.jsx)
  - **Deskripsi**: Peningkatan komponen ConfirmModal agar mendukung custom pesan dan tambahan input (misal alasan penolakan PO).

- [frontend/src/components/ui/index.js](frontend/src/components/ui/index.js)
  - **Deskripsi**: Ekspor komponen `Modal` baru ke barrel export UI.

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) & [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: Penambahan ~109 key terjemahan baru untuk fitur Vendor PO (en & id).

- [frontend/src/middleware/AuthGuard.jsx](frontend/src/middleware/AuthGuard.jsx)
  - **Deskripsi**: Penyesuaian guard autentikasi untuk route PO baru.

---

### 📅 Rincian Commit

#### [b6feb54] - save #104 (#104 - Vendor Purchase Order Management)
*Tanggal: 10 Juni 2026*

- **Komponen yang Berubah**:
  - `backend/src/app.js`
  - `backend/src/config/privilege.json` [+7 baris]
  - `backend/src/controllers/vendor.controller.js` [NEW, +217]
  - `backend/src/locales/en/translation.json` [+16]
  - `backend/src/locales/id/translation.json` [+16]
  - `backend/src/models/vendor.model.js` [NEW, +63]
  - `backend/src/models/vendorService.model.js` [NEW, +81]
  - `backend/src/routes/vendor.route.js` [NEW, +210]
  - `backend/src/services/vendor.service.js` [NEW, +244]
  - `frontend/src/app/navigation/services.js` [+10]
  - `frontend/src/app/pages/services/vendor/create.jsx` [NEW, +226]
  - `frontend/src/app/pages/services/vendor/detail.jsx` [NEW, +722]
  - `frontend/src/app/pages/services/vendor/edit.jsx` [NEW, +266]
  - `frontend/src/app/pages/services/vendor/index.jsx` [NEW, +51]
  - `frontend/src/app/pages/services/vendor/schema/columns.jsx` [NEW, +208]
  - `frontend/src/app/pages/services/vendor/schema/createSchema.js` [NEW, +56]
  - `frontend/src/app/router/protected.jsx`
  - `frontend/src/app/router/services/vendorRoute.jsx` [NEW, +40]
  - `frontend/src/i18n/locales/en/translations.json` [+60]
  - `frontend/src/i18n/locales/id/translations.json` [+54]
- **Deskripsi Perubahan & Fungsi**:
  - Implementasi awal modul Vendor: model data Vendor & VendorService, CRUD lengkap di backend (controller, service, route), serta halaman frontend (list, create, detail, edit).
  - Navigasi side menu "Vendor" ditambahkan di bagian Services.
  - Konfigurasi privilege untuk akses CRUD vendor.

#### [7da8042] - resolve #107 (#107 - Fix Combobox Component)
*Tanggal: 10 Juni 2026*

- **Komponen yang Berubah**:
  - `frontend/src/components/shared/form/Combobox.jsx` [1 baris]
- **Deskripsi Perubahan & Fungsi**:
  - Perbaikan bug pada komponen Combobox yang digunakan di berbagai form di seluruh aplikasi.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**:
  - Admin dapat membuat, melihat, mengedit, dan menghapus data Vendor beserta item/layanan yang dimilikinya.
  - Admin dapat membuat Purchase Order (PO) untuk vendor, dengan input line item, jumlah, harga satuan, dan kalkulasi grand total otomatis.
  - Admin dapat melihat preview dokumen PO dalam format terstruktur siap cetak sebelum disubmit.
  - Admin approver dapat menyetujui atau menolak PO yang diajukan beserta alasan penolakan.

- **Bug Fix / Solusi Masalah**:
  - Komponen Combobox (#107) diperbaiki sehingga tidak lagi mengalami bug saat digunakan pada form create/edit vendor maupun form lainnya.

- **Menu/Tombol Baru**:
  - Menu **Vendor** baru tersedia di navigasi bagian **Services**.
  - Tombol **Buat PO** tersedia di halaman detail vendor untuk memulai pembuatan Purchase Order.
  - Notifikasi Telegram otomatis dikirim ke grup approval saat PO disubmit, disetujui, atau ditolak — memudahkan proses approval tanpa harus login ke sistem.
