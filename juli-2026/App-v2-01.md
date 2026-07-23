# 📝 Daily Work Report - Idham (2026-07-01)

---

## 📌 Informasi Issue
- **Nomor Issue**: #118
- **Judul Issue**: Vendor Sales Order (SO) — Implementasi Penuh Modul SO Sebagai Mirror Purchase Order

## 📅 Laporan Harian - 1 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Tidak ada pekerjaan WIP — seluruh perubahan hari ini sudah ter-commit.

---

### 📅 Rincian Commit

#### [b7497a95] - TEMP: backup 104+SO+cleanup (14:32 WIB)

Commit backup sementara yang berisi akumulasi seluruh WIP dari 30 Juni 2026 (restrukturisasi folder vendor/PO/SO + cleanup). **Tidak dipush ke remote** — dibuat sebagai titik aman sebelum proses merge.

---

#### [2a405b35] - Merge remote-tracking branch 'origin/issue-104' into issue-118 (14:35 WIB)

Merge commit: branch `issue-104` (Vendor PO Management yang telah di-resolve Dedy) digabungkan ke `issue-118`. Konflik yang diselesaikan secara manual:
- `backend/src/app.js`
- `backend/src/controllers/files.controller.js`
- `backend/src/controllers/vendor.controller.js`
- `backend/src/locales/en/translation.json` & `id/translation.json`
- `backend/src/models/vendor.model.js`, `vendorPO.model.js`
- `backend/src/routes/files.route.js`, `vendor.route.js`
- `backend/src/services/vendor.service.js`
- `backend/src/utils/telegram.js`
- `frontend/.env.example`

---

#### [9e360091] - save #118 (#118 - Vendor Sales Order) (14:38 WIB)

- **Komponen yang Berubah**:

  **Backend — Modul Sales Order Lengkap**:
  - `backend/src/app.js` — Registrasi `vendorSORoute` ke aplikasi Express
  - `backend/src/controllers/files.controller.js` (+19) — Tambah handler file SO: download dari MinIO dan upload dokumen SO
  - `backend/src/controllers/vendorSO.controller.js` [NEW] (+426 baris) — Controller lengkap CRUD SO, approval admin, tanda tangan digital (embedding signature ke PDF/image via `sign-document.js`), kirim notifikasi Telegram
  - `backend/src/controllers/publicSO.controller.js` [NEW] (+72 baris) — Controller akses dokumen SO publik tanpa autentikasi
  - `backend/src/services/vendorSO.service.js` [NEW] (+260 baris) — Service layer SO: `findVendorSOById`, `createVendorSO`, `updateVendorSO`, `approveVendorSO`, populate relasi vendor dan admin
  - `backend/src/models/vendorSO.model.js` (+154 baris, diperbarui) — Model SO dilengkapi: field `approval` (ObjectId ref Admin), `approved_at`, `complete`, lampiran, nomor SO auto-increment — selaras penuh dengan `vendorPO.model.js`
  - `backend/src/routes/vendorSO.route.js` [NEW] (+287 baris) — Seluruh route API SO: CRUD, approval, upload dokumen, akses file
  - `backend/src/routes/files.route.js` (+8) — Tambah route akses file dokumen SO
  - `backend/src/routes/public.route.js` (+8) — Tambah route publik untuk preview dokumen SO
  - `backend/src/utils/telegram.js` (+50) — Notifikasi Telegram untuk event SO: dibuat, disetujui, ditolak
  - `backend/src/locales/en/translation.json` (+23) — Terjemahan SO backend (EN)
  - `backend/src/locales/id/translation.json` (+22) — Terjemahan SO backend (ID)

  **Frontend — Modul Sales Order**:
  - `frontend/src/app/pages/services/salesOrder/create.jsx` [NEW] (+697 baris) — Form pembuatan SO lengkap: pilih vendor, tambah line item, upload dokumen SO, kalkulasi total
  - `frontend/src/app/pages/services/salesOrder/edit.jsx` [NEW] (+781 baris) — Form edit SO dengan kemampuan update dokumen dan line item
  - `frontend/src/app/pages/services/salesOrder/SODocumentPreview.jsx` [NEW] (+350 baris) — Komponen preview dokumen SO inline (PDF/image via iframe) yang digunakan di dalam drawer dan halaman detail
  - `frontend/src/app/pages/services/vendorManagement/schema/vendorSOSchema.js` [NEW] (+48 baris) — Validasi form SO dengan Yup
  - `frontend/src/app/pages/services/vendorManagement/detail.jsx` (+261 baris) — Halaman detail vendor diperbarui: tambah tab **Sales Order** dengan tabel daftar SO, tombol buat SO baru, dan aksi per baris SO (lihat detail, edit, hapus)

  **Frontend — Halaman Publik SO**:
  - `frontend/src/app/pages/public/PublicSODocument.jsx` [NEW] (+431 baris) — Viewer dokumen SO publik: menampilkan detail SO, line item, dan dokumen; dapat diakses via link unik tanpa login
  - `frontend/src/app/pages/public/ReviewSOPage.jsx` [NEW] (+367 baris) — Halaman review SO publik untuk alur persetujuan/konfirmasi oleh pihak eksternal (vendor)

  **Frontend — Halaman Aktivasi (tab SO)**:
  - `frontend/src/app/pages/services/activation/index.jsx` (+93 baris) — Tambah tab **SO** di halaman Aktivasi: state `isSOReviewOpen`, `selectedSO`, `isSOProcessing`, `isSOPreviewOpen`; handler `handleSOReview`, `handleCloseSODrawer`, `handleSOViewDocument`, `handleCloseSOPreview`, `handleSOApprove`, `handleSOReject`
  - `frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx` (+399 baris, penulisan ulang besar) — Drawer review SO yang kini lengkap: tampilkan detail SO + line item + dokumen preview, pilih admin yang akan di-assign (dengan `Combobox`), checkbox "gunakan admin default", tombol kirim preview, Approve/Reject dengan ConfirmModal; menggunakan `useHasPrivilege` untuk kontrol akses
  - `frontend/src/app/pages/services/activation/schema/soColumns.jsx` (+145 baris, diperbarui) — Kolom tabel SO di halaman Aktivasi diperbarui sesuai perubahan model dan status baru

  **Frontend — Router & Terjemahan**:
  - `frontend/src/app/router/protected.jsx` (+7) — Daftarkan route SO create/edit/detail
  - `frontend/src/app/router/public.jsx` (+6) — Daftarkan route publik SO
  - `frontend/src/app/router/services/vendorRoute.jsx` (+10) — Tambah path SO ke vendor route
  - `frontend/src/i18n/locales/en/translations.json` (+114 baris) — Terjemahan SO lengkap: form, status approval, konfirmasi, pesan sukses/error, halaman publik (EN)
  - `frontend/src/i18n/locales/id/translations.json` (+169 baris) — Terjemahan SO lengkap (ID)

  **Frontend — Komponen Shared**:
  - `frontend/src/components/shared/DocumentPreviewModal.jsx` (+36) — Update mendukung preview dokumen bertipe `so` selain `po` dan dokumen standar
  - `frontend/src/components/shared/table/rows.jsx` (+101) — Tambah komponen sel kolom untuk status SO: approval status + tanda tangan vendor

- **Deskripsi Perubahan & Fungsi**:
  - Hari ini menyelesaikan modul Sales Order secara penuh sebagai cerminan (mirror) dari Purchase Order — arsitektur backend (controller/service/route), model, public viewer, halaman Aktivasi, hingga terjemahan semuanya sejajar dengan PO.
  - Proses dimulai dengan membuat backup sementara (`b7497a95`), kemudian merge `origin/issue-104` ke `issue-118` dengan resolusi konflik manual di 11 file, lalu diakhiri commit `save #118` berisi seluruh kode SO yang bersih.
  - `SOReviewDrawer` ditulis ulang secara signifikan: kini mendukung pilihan admin dinamis via Combobox, preview dokumen SO langsung di dalam drawer, dan kontrol akses berbasis privilege.
  - Model `vendorSO.model.js` diperbarui agar konsisten penuh dengan `vendorPO.model.js`: pola `approval` (ObjectId ref Admin) + `approved_at` + `complete` menggantikan pendekatan boolean lama.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**: Admin kini dapat membuat, mengedit, dan mengelola Sales Order dari halaman detail vendor. SO dapat disetujui atau ditolak dari halaman **Aktivasi** tab **SO**. Link publik SO dapat dibagikan ke vendor untuk review dan tanda tangan. Notifikasi Telegram terkirim otomatis di setiap tahap alur SO.
- **Bug Fix / Solusi Masalah**: Konflik merge antara `issue-104` dan `issue-118` diselesaikan — kedua branch kini terintegrasi penuh tanpa tumpang tindih pada file backend inti (`app.js`, `vendor.controller.js`, `vendor.service.js`, terjemahan, Telegram utils, dll.).
- **Menu/Tombol Baru**: Tab **SO** baru di halaman Aktivasi. Tab **Sales Order** di halaman detail vendor. Halaman create/edit SO baru. Tombol "Preview Dokumen" di `SOReviewDrawer`.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Penjelasan Fitur**: Modul Sales Order (SO) melengkapi modul Purchase Order yang sudah ada. PO digunakan untuk pembelian dari vendor (ISP ke vendor), sedangkan SO digunakan untuk penjualan/penawaran dari vendor ke ISP. Keduanya kini memiliki alur yang identik: buat dokumen → upload → approval admin → link publik → tanda tangan digital. Perbedaan utama: SO memiliki `vendorSOSchema.js` tersendiri, halaman create/edit di `salesOrder/`, dan reviewer di `SOReviewDrawer` dapat memilih admin yang di-assign secara dinamis.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka halaman detail vendor → tab **Sales Order** → klik **Buat SO**.
  2. Isi nomor SO, tambah line item, masukkan harga, upload dokumen SO → Simpan.
  3. SO berstatus *Pending Approval* tampil di **Aktivasi** → tab **SO**.
  4. Admin ber-privilege klik **Review** → `SOReviewDrawer` terbuka: pilih admin penyetuju, preview dokumen → klik **Approve** atau **Reject**.
  5. Klik **Salin Link** untuk mendapat URL publik SO — bagikan ke vendor.
  6. Vendor buka link publik → lihat detail SO → tanda tangan digital → status SO berubah menjadi *Signed*.
