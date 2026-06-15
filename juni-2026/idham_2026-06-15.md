# 📝 Daily Work Report - Idham (2026-06-15)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Vendor Purchase Order (PO) Management — Fitur lanjutan: Public PO Document (akses via link publik + tanda tangan digital), stamp upload, share token, dan penyempurnaan detail PO

---

## 📅 Laporan Harian - 15 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Branch: `issue-104` — 42 file berubah (+5.848 / -28 baris), belum di-commit

---

#### Penambahan Baru Hari Ini (sejak laporan 12 Juni 2026)

**Backend:**

- [backend/src/controllers/publicPO.controller.js](backend/src/controllers/publicPO.controller.js) [NEW]
  - **Deskripsi**: Controller publik (tanpa autentikasi) untuk dua endpoint dokumen PO: `getPOByToken` (mengambil data PO berdasarkan `share_token`) dan `signPOByToken` (menyimpan tanda tangan digital ke PO via token). Validasi mencegah dokumen yang sudah ditandatangani di-sign ulang.

- [backend/src/routes/public.route.js](backend/src/routes/public.route.js)
  - **Deskripsi**: Penambahan 2 route publik tanpa middleware auth: `GET /public-docs/po/:token` dan `POST /public-docs/po/sign`, sehingga vendor eksternal dapat mengakses dan menandatangani dokumen PO tanpa login ke sistem.

- [backend/src/services/vendor.service.js](backend/src/services/vendor.service.js)
  - **Deskripsi**: Penambahan 2 fungsi service baru: `findVendorPOByToken` (cari PO berdasarkan `share_token`) dan `signVendorPO` (simpan signature + timestamp `signed_at` ke PO).

- [backend/src/controllers/vendor.controller.js](backend/src/controllers/vendor.controller.js)
  - **Deskripsi**: Pembaruan `submitPO`: saat PO disubmit, sistem otomatis generate `share_token` unik dan menyertakannya dalam notifikasi Telegram sehingga penerima notifikasi langsung bisa mengakses dokumen PO via link publik. Pembaruan `updatePO`: menambahkan fitur upload/hapus gambar **stempel** (stamp) ke MinIO bucket `appFiles` — stempel lama otomatis dihapus saat diganti.

- [backend/src/models/vendorPO.model.js](backend/src/models/vendorPO.model.js)
  - **Deskripsi**: Penambahan 3 field baru ke skema VendorPO: `share_token` (string unik untuk akses publik), `signature` (data URL base64 tanda tangan digital), dan `signed_at` (timestamp penandatanganan).

**Frontend:**

- [frontend/src/app/pages/public/PublicPODocument.jsx](frontend/src/app/pages/public/PublicPODocument.jsx) [NEW, 359 baris]
  - **Deskripsi**: Halaman publik dokumen PO yang dapat diakses siapa saja via link tanpa login (`/p/po/:token`). Fitur lengkap: fetch data PO via token, tampilan dokumen PO siap cetak menggunakan `PODocumentPreview`, tombol **Print** (react-to-print), komponen `DrawerSign` untuk menggambar tanda tangan digital, upload gambar tanda tangan dari file, dan submit tanda tangan ke backend. Menampilkan preview tanda tangan yang sudah tersimpan jika dokumen sudah pernah ditandatangani.

- [frontend/src/app/router/public.jsx](frontend/src/app/router/public.jsx)
  - **Deskripsi**: Penambahan route publik `p/po/:token` yang me-lazy-load halaman `PublicPODocument` tanpa autentikasi.

- [frontend/src/app/pages/services/vendor/po/detail.jsx](frontend/src/app/pages/services/vendor/po/detail.jsx)
  - **Deskripsi**: Penambahan fitur **Salin Link Publik** — tombol dengan ikon `LinkIcon` yang meng-copy URL publik dokumen PO (`/p/po/:share_token`) ke clipboard. Tombol hanya muncul jika PO sudah memiliki `share_token` (setelah disubmit). Juga tersedia link preview dokumen publik di section aksi.

---

#### Pekerjaan WIP Keseluruhan (Carry-over dari Sebelumnya)

**Backend:**

- [backend/src/models/vendor.model.js](backend/src/models/vendor.model.js) [NEW]
  - **Deskripsi**: Model Mongoose untuk entitas Vendor: nama, kode unik, status aktif, alamat, deskripsi, kontak AM (Account Manager) dan NOC (email & telepon).

- [backend/src/models/vendorService.model.js](backend/src/models/vendorService.model.js) [NEW]
  - **Deskripsi**: Model Mongoose untuk entitas VendorItem/Service dengan tipe: `service`, `goods`, `contractor`. Field mencakup nama, kode, tipe, harga, satuan, deskripsi, dan status aktif.

- [backend/src/services/vendor.service.js](backend/src/services/vendor.service.js) [NEW]
  - **Deskripsi**: 20+ fungsi service untuk Vendor, VendorItem, dan VendorPO: CRUD lengkap masing-masing entitas + fungsi khusus (`findVendorPOByToken`, `signVendorPO`, `addAttachmentToPO`, `removeAttachmentFromPO`, dsb.).

- [backend/src/controllers/vendor.controller.js](backend/src/controllers/vendor.controller.js) [NEW]
  - **Deskripsi**: Handler CRUD lengkap: Vendor (list, select, read, create, update, delete), VendorItem (list, read, create, update, delete), VendorPO (create, list, read, submit, approve, reject, delete, update, uploadAttachment, deleteAttachment).

- [backend/src/routes/vendor.route.js](backend/src/routes/vendor.route.js) [NEW]
  - **Deskripsi**: 20 route endpoint untuk Vendor, VendorItem, dan VendorPO dengan middleware privilege masing-masing.

- [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js)
  - **Deskripsi**: Tambahan `removeAppFile` (helper hapus file MinIO) dan `getVendorPOFile` (streaming file lampiran PO dengan mode inline/download).

- [backend/src/routes/files.route.js](backend/src/routes/files.route.js)
  - **Deskripsi**: Route `GET /file/vendor-po/:name` untuk akses file lampiran PO (dilindungi privilege `vendor.read`).

- [backend/src/utils/telegram.js](backend/src/utils/telegram.js)
  - **Deskripsi**: Fungsi notifikasi Telegram: `TelegramNotifPOSubmit` (include link publik dokumen), `TelegramNotifPOApproved`, `TelegramNotifPORejected`.

- [backend/src/config/privilege.json](backend/src/config/privilege.json)
  - **Deskripsi**: Privilege baru: `vendor.list`, `vendor.read`, `vendor.create`, `vendor.update`, `vendor.delete`, `vendor.changeStatus`.

- [backend/src/utils/minio.js](backend/src/utils/minio.js)
  - **Deskripsi**: Perbaikan error logging pada `minioUpload`.

**Frontend:**

- [frontend/src/app/pages/services/vendor/](frontend/src/app/pages/services/vendor/) [NEW]
  - `index.jsx` (51 baris) — Halaman list vendor dengan tabel dan navigasi ke detail.
  - `create.jsx` (226 baris) — Form create vendor baru.
  - `edit.jsx` (266 baris) — Form edit data vendor.
  - `detail.jsx` (551 baris) — Detail vendor: info vendor, daftar item, tombol Buat PO.
  - `CreatePODrawer.jsx` (216 baris) — Drawer buat PO dari halaman detail vendor.
  - `PODocumentPreview.jsx` (313 baris) — Komponen preview dokumen PO bergaya formal/cetak.
  - `vendorServiceDetail.jsx` (183 baris) — Halaman detail satu VendorItem.
  - `schema/columns.jsx` (208 baris) — Definisi kolom tabel list vendor.
  - `schema/createSchema.js` (108 baris) — Yup validation schema untuk form vendor & vendor item.

- [frontend/src/app/pages/services/vendor/po/](frontend/src/app/pages/services/vendor/po/) [NEW]
  - `create.jsx` (532 baris) — Form create PO: line item, kalkulasi subtotal/grand total, pajak, terms & notes.
  - `detail.jsx` (449 baris) — Detail PO: status badge, aksi submit/approve/reject/edit/delete, lampiran, preview dokumen, copy link publik.
  - `EditPODrawer.jsx` (335 baris) — Drawer edit data PO (hanya saat status draft).

- [frontend/src/components/ui/Modal.jsx](frontend/src/components/ui/Modal.jsx) [NEW, 40 baris]
  - **Deskripsi**: Komponen Modal reusable (Headless UI `Dialog`).

- [frontend/src/middleware/AuthGuard.jsx](frontend/src/middleware/AuthGuard.jsx)
  - **Deskripsi**: Fix race condition: tambah state `privilegeReady` agar tidak false-redirect ke unauthorized sebelum privilege ter-fetch.

- [frontend/src/app/router/services/vendorRoute.jsx](frontend/src/app/router/services/vendorRoute.jsx) [NEW]
  - **Deskripsi**: Route untuk semua halaman vendor: list, create, edit, detail, item detail, PO create, PO detail — masing-masing dengan privilege guard.

- [frontend/src/app/navigation/services.js](frontend/src/app/navigation/services.js)
  - **Deskripsi**: Penambahan menu **Vendor** di navigasi sidebar bagian Services.

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) & [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: Penambahan 166 (en) dan 186 (id) translation key baru untuk seluruh fitur Vendor & PO.

- [frontend/jsconfig.json](frontend/jsconfig.json)
  - **Deskripsi**: Cleanup minor: hapus komentar invalid dalam JSON, perbaiki format file.

---

### 📅 Rincian Commit

> Tidak ada commit baru hari ini (15 Juni 2026).

Commit terakhir terkait issue ini:

#### [b6feb54] - save #104 (#104 - Vendor PO Management)
*10 Juni 2026* — Implementasi awal modul Vendor lengkap: model, CRUD backend, halaman frontend list/create/detail/edit, navigasi, privilege (+2.546 baris, 21 file).

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**:
  - Admin dapat membagikan dokumen PO kepada vendor eksternal via **link publik** (tanpa perlu login ke sistem) — link di-generate otomatis saat PO disubmit dan langsung disertakan di notifikasi Telegram.
  - Vendor/pihak eksternal dapat membuka halaman dokumen PO di browser, melihat detail PO, mencetak dokumen, serta **menandatangani secara digital** (menggambar tanda tangan atau upload gambar tanda tangan).
  - Admin dapat mengupload **gambar stempel** (stamp) perusahaan ke dalam data PO untuk ditampilkan di dokumen cetak, dan menghapus/mengganti stempel kapan saja.
  - Admin dapat menyalin link publik dokumen PO langsung dari halaman detail PO dengan satu klik tombol **Salin Link**.

- **Bug Fix / Solusi Masalah**:
  - **AuthGuard race condition**: Halaman yang membutuhkan privilege tidak lagi salah redirect ke halaman unauthorized saat pertama kali load (sebelum data privilege selesai di-fetch).
  - **MinIO upload error**: Error saat upload file ke MinIO kini tercatat detail di console untuk memudahkan debugging di production.

- **Menu/Tombol Baru**:
  - Halaman publik dokumen PO (`/p/po/:token`) dapat diakses tanpa login — cocok dibagikan ke vendor/pihak ketiga.
  - Tombol **Salin Link Publik** (`LinkIcon`) di halaman detail PO — hanya muncul setelah PO disubmit.
  - Tombol **Tanda Tangan Digital** di halaman publik PO — vendor dapat menandatangani langsung di browser (drawing pad atau upload file).
  - Fitur upload/hapus **Stempel** di form edit PO.
