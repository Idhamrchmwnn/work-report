# 📝 Daily Work Report - Idham (2026-06-16)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Vendor Purchase Order (PO) Management — Finalisasi: integrasi tabel PO di halaman Activation, penambahan locale backend, dan penyesuaian konfigurasi environment development

---

## 📅 Laporan Harian - 16 Juni 2026

### 📅 Rincian Commit

#### [b03dc5a] - resolve #104 (#104 - Vendor PO Management)
*16 Juni 2026, 22:22 WIB*

**42 file berubah, +5.947 / -37 baris**

---

**Backend:**

- [backend/src/models/vendor.model.js](backend/src/models/vendor.model.js) [NEW, +63]
  - **Deskripsi**: Model Mongoose Vendor: nama, kode unik, status, alamat, deskripsi, kontak AM dan NOC.

- [backend/src/models/vendorService.model.js](backend/src/models/vendorService.model.js) [NEW, +92]
  - **Deskripsi**: Model VendorItem/Service dengan tipe: `service`, `goods`, `contractor`.

- [backend/src/models/vendorPO.model.js](backend/src/models/vendorPO.model.js) [NEW, +204]
  - **Deskripsi**: Model VendorPO lengkap: sub-schema `LineItemSchema` dan `AttachmentSchema`, field status (draft/pending/approved/rejected), `share_token`, `signature`, `signed_at`, `stamp`, audit trail (submitted/approved/rejected by + timestamp).

- [backend/src/models/document.model.js](backend/src/models/document.model.js) [+4]
  - **Deskripsi**: Penyesuaian model dokumen untuk mendukung lampiran PO.

- [backend/src/services/vendor.service.js](backend/src/services/vendor.service.js) [NEW, +455]
  - **Deskripsi**: 20+ fungsi service: CRUD Vendor, VendorItem, VendorPO; `addAttachmentToPO`, `removeAttachmentFromPO`, `findVendorPOByToken`, `signVendorPO`.

- [backend/src/controllers/vendor.controller.js](backend/src/controllers/vendor.controller.js) [NEW, +499]
  - **Deskripsi**: Handler CRUD Vendor, VendorItem, dan VendorPO (create, list, read, submit, approve, reject, delete, update, uploadAttachment, deleteAttachment, updatePO dengan stamp upload).

- [backend/src/controllers/publicPO.controller.js](backend/src/controllers/publicPO.controller.js) [NEW, +35]
  - **Deskripsi**: Endpoint publik tanpa auth: `getPOByToken` (fetch PO via share token) dan `signPOByToken` (simpan tanda tangan digital).

- [backend/src/routes/vendor.route.js](backend/src/routes/vendor.route.js) [NEW, +68]
  - **Deskripsi**: 20 route endpoint untuk Vendor, VendorItem, VendorPO dengan privilege middleware.

- [backend/src/routes/files.route.js](backend/src/routes/files.route.js) [+3]
  - **Deskripsi**: Route `GET /file/vendor-po/:name` untuk streaming file lampiran PO.

- [backend/src/routes/public.route.js](backend/src/routes/public.route.js) [+4]
  - **Deskripsi**: 2 route publik: `GET /public-docs/po/:token` dan `POST /public-docs/po/sign`.

- [backend/src/utils/telegram.js](backend/src/utils/telegram.js) [+44]
  - **Deskripsi**: 3 fungsi notifikasi Telegram: `TelegramNotifPOSubmit`, `TelegramNotifPOApproved`, `TelegramNotifPORejected` — dikirim ke chat ID `telegram_approval` dengan link langsung ke dokumen PO publik.

- [backend/src/config/privilege.json](backend/src/config/privilege.json) [+8]
  - **Deskripsi**: Privilege baru untuk modul vendor: `vendor.list`, `vendor.read`, `vendor.create`, `vendor.update`, `vendor.delete`, `vendor.changeStatus`.

- [backend/src/app.js](backend/src/app.js) [+2]
  - **Deskripsi**: Registrasi route vendor ke aplikasi Express.

- [backend/src/locales/en/translation.json](backend/src/locales/en/translation.json) [+25]
- [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json) [+25]
  - **Deskripsi**: Penambahan pesan error/validasi backend untuk modul Vendor dan PO: `vendor.notFound`, `vendor.po.cannotSubmit`, `vendor.po.cannotApprove`, `vendor.po.cannotReject`, `vendor.po.cannotDelete`, `vendor.po.emptyItems`, dsb. (EN & ID).

**Frontend:**

- [frontend/src/app/pages/services/activation/index.jsx](frontend/src/app/pages/services/activation/index.jsx) [+87] ⭐ **BARU DI COMMIT INI**
  - **Deskripsi**: Refaktorisasi halaman Activation dengan menambahkan **sistem tab**: tab **BAA** (dokumen aktivasi layanan seperti sebelumnya) dan tab **PO** (tabel semua Purchase Order vendor). Tab PO menampilkan daftar PO dari endpoint `/vendor-po/list-all` menggunakan kolom yang baru dibuat.

- [frontend/src/app/pages/services/activation/schema/poColumns.jsx](frontend/src/app/pages/services/activation/schema/poColumns.jsx) [NEW, +100] ⭐ **BARU DI COMMIT INI**
  - **Deskripsi**: Definisi kolom tabel untuk daftar PO di halaman Activation: `created_at` (date range filter), `po_number` (link ke detail PO), `vendor` (link ke detail vendor), `grand_total` (format mata uang), `submitted_by`, `status` (badge berwarna dengan filter select: draft/pending/approved/rejected).

- [frontend/src/app/pages/public/PublicPODocument.jsx](frontend/src/app/pages/public/PublicPODocument.jsx) [NEW, +359]
  - **Deskripsi**: Halaman publik dokumen PO (akses via token tanpa login): tampilan dokumen siap cetak, fitur print, menggambar/upload tanda tangan digital, submit tanda tangan ke backend.

- [frontend/src/app/pages/services/vendor/](frontend/src/app/pages/services/vendor/) [NEW]
  - `index.jsx` (+51) — List vendor.
  - `create.jsx` (+226) — Form create vendor.
  - `edit.jsx` (+266) — Form edit vendor.
  - `detail.jsx` (+551) — Detail vendor + manajemen item.
  - `CreatePODrawer.jsx` (+216) — Drawer buat PO dari detail vendor.
  - `PODocumentPreview.jsx` (+313) — Preview dokumen PO bergaya formal/cetak.
  - `vendorServiceDetail.jsx` (+183) — Detail satu VendorItem.
  - `schema/columns.jsx` (+208) — Kolom tabel list vendor.
  - `schema/createSchema.js` (+108) — Yup validation schema vendor & item.

- [frontend/src/app/pages/services/vendor/po/](frontend/src/app/pages/services/vendor/po/) [NEW]
  - `create.jsx` (+532) — Form Create PO: line item, kalkulasi grand total, pajak, terms.
  - `detail.jsx` (+449) — Detail PO: status, lampiran, aksi submit/approve/reject/edit/delete, copy link publik.
  - `EditPODrawer.jsx` (+269) — Drawer edit PO (status draft).

- [frontend/src/components/ui/Modal.jsx](frontend/src/components/ui/Modal.jsx) [NEW, +40]
  - **Deskripsi**: Komponen Modal reusable (Headless UI Dialog).

- [frontend/src/components/shared/ConfirmModal.jsx](frontend/src/components/shared/ConfirmModal.jsx) [+22]
  - **Deskripsi**: Tambahan dukungan API alternatif (`isOpen`, `onConfirm`, `isLoading`, `title`, `description`) backward-compatible dengan API lama.

- [frontend/src/middleware/AuthGuard.jsx](frontend/src/middleware/AuthGuard.jsx) [+14]
  - **Deskripsi**: Fix race condition: state `privilegeReady` mencegah false-redirect sebelum privilege selesai di-fetch.

- [frontend/src/app/router/protected.jsx](frontend/src/app/router/protected.jsx) [+2]
  - **Deskripsi**: Registrasi `vendorRoute` ke protected router.

- [frontend/src/app/router/public.jsx](frontend/src/app/router/public.jsx) [+6]
  - **Deskripsi**: Route publik `/p/po/:token` untuk halaman dokumen PO tanpa login.

- [frontend/src/app/router/services/vendorRoute.jsx](frontend/src/app/router/services/vendorRoute.jsx) [NEW, +67]
  - **Deskripsi**: Route lengkap modul vendor: list, create, edit, detail, item detail, PO create, PO detail.

- [frontend/src/app/navigation/services.js](frontend/src/app/navigation/services.js) [+10]
  - **Deskripsi**: Menu **Vendor** di navigasi sidebar Services.

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) [+166]
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) [+180]
  - **Deskripsi**: Translation key lengkap untuk fitur Vendor & PO (EN & ID).

- [frontend/src/components/ui/index.js](frontend/src/components/ui/index.js) [+1]
  - **Deskripsi**: Ekspor komponen Modal baru.

- [frontend/jsconfig.json](frontend/jsconfig.json) [-1/+1]
  - **Deskripsi**: Cleanup format JSON (hapus komentar tidak valid).

**Telegram API:**

- [telegram-api/src/bots.json](telegram-api/src/bots.json) [+25]
  - **Deskripsi**: Pembaruan konfigurasi bot Telegram untuk environment development.

---

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

- [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js)
  - **Deskripsi**: Perubahan nama MinIO bucket untuk environment development: dari `'app-development'` menjadi `'dekasimal-dev'` pada semua 9 bucket (avatarAdmin, avatarCustomer, avatarPartner, adminSign, appFiles, adminPresence, adminPermission, documentCustomer, documentPartner). Menyesuaikan dengan nama bucket aktual di server MinIO dev.

- [frontend/.env.development](frontend/.env.development)
  - **Deskripsi**: Perubahan `VITE_API_URL` dari `https://server-dev.dekadata.net` ke `http://127.0.0.1:3000` untuk testing di server lokal.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**:
  - Halaman **Activation** kini memiliki dua tab: **BAA** (dokumen aktivasi layanan existing) dan **PO** (daftar semua Purchase Order dari semua vendor) — admin bisa memantau status seluruh PO dari satu halaman terpusat.
  - Tabel PO di tab Activation mendukung filter status (draft/pending/approved/rejected) dan date range, serta link langsung ke detail PO maupun detail vendor.
  - Seluruh modul Vendor (manajemen vendor, item, PO, dokumen publik, tanda tangan digital, notifikasi Telegram) kini telah di-commit dan siap digunakan.
  - Pesan error dari backend (validasi PO, status tidak valid, dll.) kini ditampilkan dalam Bahasa Indonesia dan Inggris.

- **Bug Fix / Solusi Masalah**:
  - **MinIO bucket name** (WIP): Nama bucket di-update ke `dekasimal-dev` agar sesuai dengan konfigurasi MinIO server development yang aktual — mencegah error upload file saat testing lokal.
  - **API URL** (WIP): URL API dikembalikan ke localhost untuk keperluan testing backend lokal.

- **Menu/Tombol Baru**:
  - Tab **PO** baru di halaman **Activation** (`/services/activation`) — menampilkan daftar lengkap Purchase Order seluruh vendor dengan status badge berwarna.
