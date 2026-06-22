# 📝 Daily Work Report - Idham (2026-06-22)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Vendor PO Management — Manajemen Vendor, Purchase Order, dan Sales Order

## 📅 Laporan Harian - 22 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Tidak ada pekerjaan WIP — seluruh perubahan sudah ter-commit dalam `fdad266 save #104`.

---

### 📅 Rincian Commit

#### [fdad266] - save #104 (#104 - Vendor PO Management)

- **Komponen yang Berubah**:

  **Backend — Core**:
  - `backend/src/app.js` — Registrasi VendorRoute ke aplikasi Express
  - `backend/src/config/privilege.json` — Tambah privilege modul vendor (`vendor.*`)
  - `backend/src/controllers/files.controller.js` — Tambah handler file untuk vendor (download/upload dokumen vendor)
  - `backend/src/controllers/vendor.controller.js` [NEW] — Controller lengkap modul vendor: CRUD vendor, layanan vendor, PO, SO, approval PO/SO, tanda tangan digital
  - `backend/src/controllers/publicPO.controller.js` [NEW] — Controller dokumen PO publik (akses tanpa login)
  - `backend/src/services/vendor.service.js` [NEW] — Service layer seluruh operasi vendor, PO, dan SO (query, create, update, approval)
  - `backend/src/models/vendor.model.js` [NEW] — Model data master Vendor (nama, alamat, kontak, NPWP)
  - `backend/src/models/vendorPO.model.js` [NEW] — Model Purchase Order (nomor PO, item, nominal, status approval, dokumen, tanda tangan)
  - `backend/src/models/vendorService.model.js` [NEW] — Model layanan/jasa yang terikat ke vendor
  - `backend/src/models/document.model.js` — Minor update model dokumen
  - `backend/src/routes/vendor.route.js` [NEW] — Seluruh route API vendor, PO, SO (CRUD + approval + dokumen)
  - `backend/src/routes/files.route.js` — Tambah route file untuk akses dokumen vendor
  - `backend/src/routes/public.route.js` — Tambah route publik untuk preview dokumen PO
  - `backend/src/utils/telegram.js` — Tambah notifikasi Telegram untuk event PO/SO (dibuat, disetujui, ditolak)
  - `backend/src/locales/en/translation.json` — Tambah terjemahan modul vendor (EN)
  - `backend/src/locales/id/translation.json` — Tambah terjemahan modul vendor (ID)

  **Frontend — Modul Vendor**:
  - `frontend/src/app/pages/services/vendor/index.jsx` [NEW] — Halaman daftar vendor dengan tabel data
  - `frontend/src/app/pages/services/vendor/create.jsx` [NEW] — Form pembuatan vendor baru
  - `frontend/src/app/pages/services/vendor/edit.jsx` [NEW] — Form edit data vendor
  - `frontend/src/app/pages/services/vendor/detail.jsx` [NEW] — Halaman detail vendor (info, layanan, PO, SO)
  - `frontend/src/app/pages/services/vendor/vendorServiceDetail.jsx` [NEW] — Halaman detail layanan vendor
  - `frontend/src/app/pages/services/vendor/CreatePODrawer.jsx` [NEW] — Drawer buat PO baru dari halaman vendor
  - `frontend/src/app/pages/services/vendor/PODocumentPreview.jsx` [NEW] — Komponen preview dokumen PO inline (PDF/image via iframe)
  - `frontend/src/app/pages/services/vendor/schema/columns.jsx` [NEW] — Definisi kolom tabel vendor
  - `frontend/src/app/pages/services/vendor/schema/vendorSchema.js` [NEW] — Validasi form vendor (Yup/Zod)
  - `frontend/src/app/pages/services/vendor/schema/vendorPOSchema.js` [NEW] — Validasi form Purchase Order
  - `frontend/src/app/pages/services/vendor/schema/vendorItemSchema.js` [NEW] — Validasi item/line PO

  **Frontend — Sub-modul Purchase Order (PO)**:
  - `frontend/src/app/pages/services/vendor/po/create.jsx` [NEW] — Form pembuatan PO lengkap dengan line item
  - `frontend/src/app/pages/services/vendor/po/detail.jsx` [NEW] — Halaman detail PO: preview dokumen inline, status persetujuan, history
  - `frontend/src/app/pages/services/vendor/po/EditPODrawer.jsx` [NEW] — Drawer edit data PO

  **Frontend — Halaman Publik**:
  - `frontend/src/app/pages/public/PublicPODocument.jsx` [NEW] — Halaman publik viewer dokumen PO (dapat diakses tanpa login via link)

  **Frontend — Halaman Aktivasi**:
  - `frontend/src/app/pages/services/activation/index.jsx` — Tambah tab PO di halaman Aktivasi; state dan handler PO approval
  - `frontend/src/app/pages/services/activation/components/POReviewDrawer.jsx` [NEW] — Drawer review PO: Approve/Reject dengan konfirmasi
  - `frontend/src/app/pages/services/activation/schema/poColumns.jsx` [NEW] — Definisi kolom tabel PO di halaman Aktivasi (`POApprovalStatusCell`)

  **Frontend — Komponen Shared**:
  - `frontend/src/components/ui/Modal.jsx` [NEW] — Komponen modal generik (Headless UI Dialog + Transition, animasi fade/scale)
  - `frontend/src/components/ui/index.js` — Export `Modal`
  - `frontend/src/components/shared/ConfirmModal.jsx` — Upgrade ke dual API (mendukung props lama + API baru `isOpen`/`onConfirm`/`title`/`description`)
  - `frontend/src/components/shared/DocumentPreviewModal.jsx` — Update untuk mendukung preview dokumen vendor
  - `frontend/src/components/shared/table/rows.jsx` — Update row component tabel

  **Frontend — Router & Navigasi**:
  - `frontend/src/app/navigation/services.js` — Tambah item menu "Vendor" di sidebar
  - `frontend/src/app/router/protected.jsx` — Registrasi vendorRoute
  - `frontend/src/app/router/public.jsx` — Tambah route publik untuk dokumen PO
  - `frontend/src/app/router/services/vendorRoute.jsx` [NEW] — Definisi routing modul vendor (index, create, edit, detail, PO, SO)
  - `frontend/src/middleware/AuthGuard.jsx` — Update middleware AuthGuard

  **Frontend — Terjemahan**:
  - `frontend/src/i18n/locales/en/translations.json` (+236 baris) — Terjemahan lengkap modul vendor, PO, SO (EN)
  - `frontend/src/i18n/locales/id/translations.json` (+249 baris) — Terjemahan lengkap modul vendor, PO, SO (ID)

  **Frontend — Minor Fix**:
  - `frontend/.env.development` — Bersihkan variabel env yang tidak diperlukan
  - `frontend/jsconfig.json` — Perbaiki path alias
  - `frontend/src/app/pages/network/ipv4Management/create.jsx` — Minor fix
  - `frontend/src/app/pages/network/ipv4Management/edit.jsx` — Minor fix
  - `frontend/src/app/pages/services/hotspotVoucher/create.jsx` — Minor fix
  - `frontend/src/app/pages/services/hotspotVoucher/edit.jsx` — Minor fix
  - `frontend/src/app/pages/services/hotspotVoucher/editBatch.jsx` — Minor fix

  **Telegram API**:
  - `telegram-api/src/bots.json` — Update konfigurasi bot Telegram untuk notifikasi vendor

- **Deskripsi Perubahan & Fungsi**:
  - Implementasi penuh modul Vendor Management: dari pendaftaran vendor, pengelolaan layanan, pembuatan Purchase Order, upload dokumen, hingga alur persetujuan digital oleh admin.
  - Backend dibangun dengan arsitektur Model-Service-Controller yang bersih: service layer menangani semua logika bisnis, controller hanya sebagai adapter HTTP.
  - Sistem approval menggunakan field `approval` (ObjectId ref Admin) agar data approver bisa di-populate dengan nama dan avatar.
  - Dokumen PO dapat diakses secara publik melalui link tanpa login, memudahkan sharing ke vendor.
  - Notifikasi Telegram otomatis terkirim saat PO/SO dibuat, disetujui, atau ditolak.
  - Komponen `Modal.jsx` dibuat sebagai fondasi modal generik yang dapat digunakan di seluruh aplikasi.
  - `ConfirmModal.jsx` diperbarui agar kompatibel mundur (props lama tetap berfungsi) sekaligus mendukung API baru yang lebih ringkas.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**: Admin dapat mengakses modul Vendor baru melalui sidebar — mendaftarkan vendor, membuat Purchase Order dengan line item, mengunggah dokumen PO, dan memproses persetujuan dari halaman Aktivasi tab PO. Admin dengan privilege `vendor.changeStatus` dapat menyetujui atau menolak PO.
- **Bug Fix / Solusi Masalah**: Minor fix pada halaman ipv4Management (create/edit) dan hotspotVoucher (create/edit/editBatch) — kemungkinan penyesuaian import atau kompatibilitas komponen setelah penambahan path alias baru di `jsconfig.json`.
- **Menu/Tombol Baru**: Menu "Vendor" baru di sidebar navigasi. Halaman Aktivasi kini memiliki tab "PO" di samping tab "BAA". Tombol Approve/Reject di POReviewDrawer. Halaman detail PO memiliki tombol Cetak dan preview dokumen inline.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Penjelasan Fitur**: Modul Vendor Management memungkinkan pengelolaan hubungan dengan pemasok secara terpusat. Admin mendaftarkan vendor beserta detail kontak dan NPWP, mengaitkan layanan/jasa yang mereka sediakan, lalu membuat Purchase Order (PO) sebagai dokumen resmi pembelian. Dokumen PO dapat diunggah dan ditampilkan sebagai preview inline di halaman detail. Alur persetujuan melibatkan admin ber-privilege: PO menunggu review di halaman Aktivasi → admin Approve/Reject → status terupdate dan notifikasi Telegram terkirim. Link publik PO juga tersedia untuk dibagikan ke vendor tanpa perlu login.
- **Langkah Penggunaan (Tutorial)**:
  1. Masuk ke sidebar → **Layanan** → **Vendor** → klik **Tambah Vendor**.
  2. Isi data vendor (nama, alamat, kontak, NPWP) → Simpan.
  3. Buka halaman detail vendor → tambahkan layanan di tab "Layanan".
  4. Klik **Buat PO** → isi nomor PO, pilih layanan, tambah line item dan nominal → unggah dokumen PO → Simpan.
  5. PO berstatus *Pending Approval* muncul di halaman **Aktivasi** tab **PO**.
  6. Admin ber-privilege buka PO → klik **Review** → pilih **Approve** atau **Reject** dengan konfirmasi.
  7. Status PO berubah menjadi *Approved* disertai avatar admin yang menyetujui dan timestamp persetujuan.
  8. Di halaman detail PO, klik **Cetak** untuk mencetak dokumen langsung dari browser.
