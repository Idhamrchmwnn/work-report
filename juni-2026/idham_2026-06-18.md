# 📝 Daily Work Report - Idham (2026-06-18)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Vendor Purchase Order (PO) Management — Pengembangan modul Sales Order (SO) vendor: halaman create & detail SO, integrasi di Activation dan Detail Vendor, serta perbaikan bug kamera di Telegram Mini Apps

---

## 📅 Laporan Harian - 18 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Branch: `issue-104` — tidak ada commit baru hari ini

---

#### File Baru (Untracked)

- [frontend/src/app/pages/services/vendor/so/create.jsx](frontend/src/app/pages/services/vendor/so/create.jsx) [NEW, 179 baris]
  - **Deskripsi**: Halaman Create Sales Order (SO) untuk vendor. Form input: nomor SO, tanggal SO, catatan, dan upload dokumen SO (PDF/gambar via FilePond). Data vendor di-fetch otomatis dari URL param `vendorId`. Submit mengirim data ke endpoint `/vendor-so/create` menggunakan `FormData`. Menggunakan Yup schema `vendorSOSchema` untuk validasi.

- [frontend/src/app/pages/services/vendor/so/detail.jsx](frontend/src/app/pages/services/vendor/so/detail.jsx) [NEW, 342 baris]
  - **Deskripsi**: Halaman Detail SO vendor — menampilkan informasi lengkap SO (nomor, vendor, tanggal, total, status tanda tangan). Fitur: tombol **Submit** (kirim notifikasi ke tim internal via `/vendor-so/submit/:id`), tombol **Sign** (tandatangani SO via `/vendor-so/sign/:id`, privilege: `vendor.changeStatus`), tombol **Reject** (tolak SO), tombol **Delete** (hapus SO, hanya jika belum ditandatangani). Menampilkan dokumen SO yang diupload vendor (inline via `SERVER_URL`), riwayat status, dan breadcrumb navigasi ke detail vendor.

- [frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx](frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx) [NEW, 173 baris]
  - **Deskripsi**: Drawer review Sales Order di halaman Activation — menampilkan ringkasan SO (nomor, vendor, total, tanggal, waktu tanda tangan jika sudah signed), link langsung ke detail SO, tombol **Sign** dan **Reject** dengan dialog konfirmasi via `ConfirmModal`. Struktur serupa dengan `POReviewDrawer`.

- [frontend/src/app/pages/services/activation/schema/soColumns.jsx](frontend/src/app/pages/services/activation/schema/soColumns.jsx) [NEW, 128 baris]
  - **Deskripsi**: Definisi kolom tabel SO untuk halaman Activation. Kolom: `created_at` (date filter), `so_number` (link ke detail), `vendor` (link ke detail vendor), `grand_total` (format mata uang), `sign_status` (via komponen `SOSignStatusCell`), kolom Aksi (`RowActions` — link view, delete jika belum signed). Komponen `SOSignStatusCell`: jika sudah signed tampilkan icon centang hijau; jika pending dan punya privilege tampilkan tombol **Review**; jika tidak punya privilege tampilkan icon jam pending.

---

#### File Dimodifikasi

- [frontend/src/app/pages/services/activation/index.jsx](frontend/src/app/pages/services/activation/index.jsx) [+68]
  - **Deskripsi**: Penambahan **tab SO** (Sales Order) di halaman Activation — sekarang ada 3 tab: BAA, PO, dan SO. Tab SO menampilkan tabel SO dari endpoint `/vendor-so/list-all`. Integrasi penuh `SOReviewDrawer` dengan handler `handleSOReview`, `handleCloseSODrawer`, `handleSOSign`, `handleSOReject`. State baru: `isSOReviewOpen`, `selectedSO`, `isSOProcessing`.

- [frontend/src/app/pages/services/vendor/detail.jsx](frontend/src/app/pages/services/vendor/detail.jsx) [+162]
  - **Deskripsi**: Integrasi modul SO di halaman Detail Vendor — import `SOSignStatusCell` dan `SOReviewDrawer`. State baru: `soList`, `loadingSO`, `isSOReviewOpen`, `selectedSOReview`, `isSOProcessing`, `deleteSOTarget`, `isDeletingSO`. Handler lengkap: `handleSOReview`, `handleCloseSODrawer`, `handleSOSign`, `handleSOReject`, `handleDeleteSO`, `fetchSOList`. Sehingga admin dapat melakukan sign/reject/delete SO langsung dari halaman detail vendor tanpa berpindah halaman.

- [frontend/src/app/pages/services/vendor/schema/createSchema.js](frontend/src/app/pages/services/vendor/schema/createSchema.js) [+11]
  - **Deskripsi**: Penambahan Yup validation schema `vendorSOSchema` — field: `vendor_id` (required), `so_number` (required), `so_date` (optional date), `notes` (optional string).

- [frontend/src/app/router/services/vendorRoute.jsx](frontend/src/app/router/services/vendorRoute.jsx) [+18]
  - **Deskripsi**: Penambahan 2 route baru untuk modul SO:
    - `vendor/view/:vendorId/so/create` (lazy load `so/create`, privilege: `vendor.create`)
    - `vendor/so/:id` (lazy load `so/detail`, privilege: `vendor.read`)

- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) [+62]
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) [+60]
  - **Deskripsi**: Penambahan seluruh translation key untuk modul SO dalam EN & ID:
    - `vendor.so.*` — name, create, list, number, date, total, subtotal, grandTotal, detail, items, lineItems, addItem, qty, summary, notes, currency, tax, taxNone, taxFixed, docSettings, viewDetail, signedAt, signedBy, sentByVendor, pendingSign, signed, sign, reject, submit, notFound, created, submitted, signedSuccess, confirmDelete, uploadDoc, uploadDocDesc, document, noDocument, history.*
    - `activation.listSO`, `activation.confirmSignSOTitle/Desc`, `activation.confirmRejectSOTitle/Desc`, `activation.signAction`, `activation.rejectAction`
    - `form.soDate`, `form.soNumber`

- [frontend/.env.development](frontend/.env.development)
  - **Deskripsi**: `VITE_API_URL` diubah ke `http://127.0.0.1:3000` untuk testing backend lokal. (Tidak perlu di-commit.)

---

### 🔍 Analisis & Bug Fix (Non-commit)

**Bug: Kamera belakang terbuka saat presensi di Telegram Mini Apps**
- **File**: [telegram-apps/src/components/CameraOverlay.jsx](telegram-apps/src/components/CameraOverlay.jsx)
- **Penyebab 1 (Permission popup ganda)**: `enumerateDevices()` dipanggil 3x sebelum/saat permission — bisa memicu multiple dialog izin di Telegram WebApp. Ditambah fallback `getUserMedia()` ke-2 jika constraints deviceId tidak valid.
- **Penyebab 2 (Kamera belakang)**: Default `facingMode: 'environment'` (kamera belakang) sebagai fallback, dan `currentDeviceIndex: 0` yang di Android sering mengarah ke kamera belakang.
- **Solusi yang dianalisis**: Hapus enumerasi device sebelum permission, gunakan `facingMode: 'user'` langsung sebagai default, enumerate device hanya setelah permission diberikan, switch kamera pakai toggle `facingMode` bukan index device.

---

### 📅 Rincian Commit

> Tidak ada commit baru hari ini (18 Juni 2026). Seluruh pekerjaan masih WIP.

Commit terakhir terkait issue ini: `78f2532` (17 Juni 2026, 15:41 WIB).

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**:
  - Admin dapat membuat **Sales Order (SO)** untuk vendor — input nomor SO, tanggal, catatan, dan upload dokumen SO dari vendor (PDF/gambar).
  - Admin dapat melihat detail SO, mengunduh/melihat dokumen SO yang diupload, dan mengikuti riwayat status (created → submitted → signed/rejected).
  - Admin dapat mengirim notifikasi internal saat SO siap ditandatangani (tombol Submit).
  - Admin approver (privilege `vendor.changeStatus`) dapat **menandatangani** atau **menolak** SO langsung dari halaman detail SO maupun dari drawer review di halaman Activation dan Detail Vendor.
  - Halaman **Activation** kini memiliki 3 tab: **BAA**, **PO**, dan **SO** — semua dokumen vendor terpusat di satu halaman.
  - Tabel SO menampilkan status tanda tangan secara visual: icon centang hijau jika sudah signed, tombol Review jika menunggu, atau icon pending jika tidak punya hak akses.

- **Bug Fix / Solusi Masalah**:
  - **Kamera presensi Telegram Mini Apps**: Dianalisis penyebab popup izin ganda dan kamera belakang yang terbuka — solusi mengganti pendekatan enumerasi device dari *sebelum* ke *setelah* permission, dan default ke kamera depan (`facingMode: 'user'`).

- **Menu/Tombol Baru**:
  - Tab **SO** (Sales Order) baru di halaman **Activation** (`/services/activation`).
  - Tombol **Buat SO** di halaman Detail Vendor — membuka form create SO untuk vendor tersebut.
  - Tombol **Submit**, **Sign**, **Reject**, **Delete** di halaman Detail SO.
  - Drawer **Review SO** dapat dibuka dari tabel SO di Activation maupun dari tabel SO di Detail Vendor.
