# 📝 Daily Work Report - Idham (2026-06-29)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Quotation Management — Modul Penawaran Harga kepada Customer

## 📅 Laporan Harian - 29 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

**Backend — Staged (sudah di `git add`)**:
- `backend/src/app.js`
  - **Deskripsi**: Registrasi route `customerQuotation` ke aplikasi Express.

- `backend/src/config/privilege.json`
  - **Deskripsi**: Tambah privilege modul quotation (`quotation.*`: list, read, create, update, changeStatus, delete).

- `backend/src/controllers/files.controller.js`
  - **Deskripsi**: Tambah handler file untuk dokumen quotation (download dari MinIO dan upload file quotation).

- `backend/src/locales/en/translation.json` & `id/translation.json`
  - **Deskripsi**: Tambah terjemahan backend untuk pesan error/sukses modul quotation dan customer (EN & ID).

- `backend/src/routes/files.route.js`
  - **Deskripsi**: Tambah route akses file dokumen quotation.

- `backend/src/routes/public.route.js`
  - **Deskripsi**: Tambah route publik untuk preview dokumen quotation tanpa autentikasi.

- `backend/src/utils/telegram.js`
  - **Deskripsi**: Tambah notifikasi Telegram untuk event quotation (+59 baris): dibuat, disetujui admin, ditolak, ditandatangani customer.

**Backend — Untracked (belum di-stage)**:
- `backend/src/models/customerQuotation.model.js` [NEW] (+188 baris)
  - **Deskripsi**: Model data Quotation dengan sub-schema: `AttachmentSchema` (nama file, path MinIO, ukuran, uploader) dan `LineItemSchema` (produk ref, nama, tipe: service/goods/internet, satuan, harga tipe OTC/MRC, qty, harga satuan). Model utama menyimpan nomor quotation (auto-increment), customer, tanggal, daftar item, total, status approval admin (`approval` ref Admin), dan tanda tangan customer.

- `backend/src/controllers/customerQuotation.controller.js` [NEW] (+414 baris)
  - **Deskripsi**: Controller lengkap: CRUD quotation, approval admin, tanda tangan customer (customer signature embedding), dan akses dokumen publik.

- `backend/src/controllers/publicQuotation.controller.js` [NEW]
  - **Deskripsi**: Controller untuk akses dokumen quotation publik tanpa login.

- `backend/src/services/customerQuotation.service.js` [NEW] (+369 baris)
  - **Deskripsi**: Service layer: query, create, update quotation, approve, customer sign, populate relasi customer dan admin.

- `backend/src/routes/customerQuotation.route.js` [NEW] (+137 baris)
  - **Deskripsi**: Seluruh route API quotation: CRUD + approval + customer signature + file upload.

- `backend/src/utils/is-object-id.js` [NEW]
  - **Deskripsi**: Utility helper untuk validasi apakah suatu nilai adalah MongoDB ObjectId yang valid.

**Frontend — Staged**:
- `frontend/.env.development`
  - **Deskripsi**: Update variabel environment.

- `frontend/src/app/navigation/services.js`
  - **Deskripsi**: Tambah item menu "Customer Management" di sidebar navigasi dengan icon `MdOutlineGroups`, terikat ke privilege `quotation.list`.

- `frontend/src/app/router/protected.jsx`
  - **Deskripsi**: Registrasi `customerRoute` di protected router.

- `frontend/src/app/router/public.jsx`
  - **Deskripsi**: Tambah route publik untuk halaman preview dokumen quotation.

- `frontend/src/components/shared/table/rows.jsx`
  - **Deskripsi**: Tambah komponen sel kolom untuk status quotation (+88 baris): status approval admin dan status tanda tangan customer.

- `frontend/src/constants/app.constant.js`
  - **Deskripsi**: Tambah konstanta aplikasi baru terkait modul quotation.

- `frontend/src/i18n/locales/en/translations.json` & `id/translations.json`
  - **Deskripsi**: Tambah 126 baris terjemahan modul quotation dan customer management (EN & ID): label form, status, konfirmasi, pesan sukses/error, navigasi.

**Frontend — Untracked (belum di-stage)**:
- `frontend/src/app/pages/services/customerManagement/index.jsx` [NEW] (+39 baris)
  - **Deskripsi**: Halaman daftar customer dengan tabel data.

- `frontend/src/app/pages/services/customerManagement/detail.jsx` [NEW] (+266 baris)
  - **Deskripsi**: Halaman detail customer: informasi customer, riwayat quotation yang pernah dibuat untuk customer tersebut.

- `frontend/src/app/pages/services/customerManagement/schema/columns.jsx` [NEW] (+118 baris)
  - **Deskripsi**: Definisi kolom tabel daftar customer.

- `frontend/src/app/pages/services/quotation/create.jsx` [NEW] (+635 baris)
  - **Deskripsi**: Form pembuatan quotation lengkap: pilih customer, tambah line item dinamis (tipe service/goods/internet, OTC/MRC, qty, harga), upload lampiran, kalkulasi total otomatis.

- `frontend/src/app/pages/services/quotation/edit.jsx` [NEW] (+236 baris)
  - **Deskripsi**: Form edit quotation.

- `frontend/src/app/pages/services/quotation/detail.jsx` [NEW] (+11 baris)
  - **Deskripsi**: Shell halaman detail quotation.

- `frontend/src/app/pages/services/quotation/QuotationDetailDrawer.jsx` [NEW] (+699 baris)
  - **Deskripsi**: Drawer detail quotation lengkap: info quotation, line item dengan total, preview dokumen, status approval admin (avatar + timestamp), status tanda tangan customer, tombol Approve/Reject/Edit/Hapus/Kirim Link. Mengikuti pola `VendorPODetailDrawer`.

- `frontend/src/app/pages/services/quotation/QuotationDocumentPreview.jsx` [NEW] (+371 baris)
  - **Deskripsi**: Komponen preview dokumen quotation inline (PDF/image via iframe), digunakan di dalam `QuotationDetailDrawer`.

- `frontend/src/app/pages/services/quotation/EditQuotationDrawerCell.jsx` [NEW] (+38 baris)
  - **Deskripsi**: Adapter komponen `EditQuotationDrawer` untuk digunakan sebagai aksi baris tabel (RowActions convention) — fetch data quotation lengkap via nomor quotation sebelum membuka drawer.

- `frontend/src/app/pages/services/quotation/schema/quotationColumns.jsx` [NEW]
  - **Deskripsi**: Definisi kolom tabel daftar quotation.

- `frontend/src/app/pages/public/PublicQuotationDocument.jsx` [NEW] (+486 baris)
  - **Deskripsi**: Halaman publik viewer dokumen quotation — dapat diakses tanpa login via link unik, menampilkan detail quotation dan memungkinkan customer menandatangani dokumen secara digital.

- `frontend/src/app/router/services/customerRoute.jsx` [NEW] (+32 baris)
  - **Deskripsi**: Definisi routing modul customer dan quotation: `/customer`, `/customer/view/:id`, `/quotation/create`, `/quotation/:id`.

- `frontend/src/constants/quotation.constant.js` [NEW] (+12 baris)
  - **Deskripsi**: Konstanta modul quotation (status, tipe item, tipe harga, dll.).

- `frontend/src/utils/fetchCompanyInfo.js` [NEW] (+31 baris)
  - **Deskripsi**: Utility function untuk mengambil informasi perusahaan dari API — digunakan pada dokumen publik quotation untuk menampilkan logo dan info pengirim.

---

### 📅 Rincian Commit

Tidak ada commit baru hari ini — seluruh pekerjaan masih berstatus WIP (sebagian staged, sebagian untracked).

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**: Admin dapat membuat penawaran harga (quotation) kepada customer: memilih customer, menambahkan item layanan/barang dengan harga OTC atau MRC, mengunggah dokumen quotation, lalu mengirim link ke customer. Customer dapat membuka link tersebut tanpa login dan menandatangani quotation secara digital. Admin dengan privilege dapat menyetujui atau menolak quotation dari dalam sistem.
- **Bug Fix / Solusi Masalah**: Tidak ada bug fix — ini adalah modul baru.
- **Menu/Tombol Baru**: Menu "Customer Management" baru di sidebar navigasi.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Penjelasan Fitur**: Modul Customer Quotation Management memungkinkan pembuatan dokumen penawaran harga resmi kepada customer ISP. Quotation berisi daftar layanan/item dengan tipe harga OTC (one-time charge) atau MRC (monthly recurring charge), lampiran dokumen, dan kalkulasi total otomatis. Alur kerja: admin buat quotation → setujui → kirim link publik ke customer → customer buka link dan tanda tangan digital → quotation selesai. Notifikasi Telegram terkirim di setiap tahap.
- **Langkah Penggunaan (Tutorial)**:
  1. Sidebar → **Customer Management** → pilih customer → klik **Buat Quotation**.
  2. Isi nomor quotation, tanggal, tambah line item (pilih produk/layanan, tentukan tipe OTC/MRC, qty, harga).
  3. Upload dokumen quotation (opsional) → Simpan.
  4. Admin ber-privilege **Approve** quotation dari drawer detail.
  5. Klik **Salin Link** → bagikan URL publik ke customer.
  6. Customer buka link → tanda tangan digital → status quotation berubah menjadi *Signed*.
