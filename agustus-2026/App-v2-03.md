# 📝 Daily Work Report - Idham (2026-08-03)

---

## 📌 Informasi Issue
- **Nomor Issue**: #153
- **Judul Issue**: Prospect Management — Ganti Pendaftaran Publik dengan "Jadikan Prospek" dari Registrasi, Integrasi Funnel Dokumen di Detail Prospek, & Work Order dari Prospek

## 📅 Laporan Harian - 3 Agustus 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Belum ada commit hari ini. Bekerja di branch **`issue-153`** (Prospect), di atas basis yang sudah memuat merge #154 (Work Order) oleh Dedy (1 Agustus). Working tree: **17 berkas ter-stage** (±528 baris ditambah / ±487 dihapus) — pengurangan besar karena halaman pendaftaran prospek publik dihapus.

Tiga arah pekerjaan hari ini: **(1)** mengganti alur **pendaftaran prospek publik** (link referal) dengan aksi **"Jadikan Prospek"** dari halaman registrasi pelanggan, **(2)** mengintegrasikan **funnel dokumen penuh** (Quotation → PO → SO → Work Order) langsung ke halaman detail prospek, dan **(3)** memungkinkan **Work Order dibuat dari SO milik prospek** (bukan hanya customer).

---

### 🧩 Area 1 — Hapus Pendaftaran Prospek Publik (Link Referal)

Alur lama (calon pelanggan mendaftar sendiri lewat link referal marketing) dibuang, digantikan alur berbasis registrasi yang sudah ada.

- [frontend/src/app/pages/public/prospectRegistration.jsx](frontend/src/app/pages/public/prospectRegistration.jsx) **[DELETED]** *(−298 baris)*
  - **Perubahan**: Halaman form pendaftaran prospek publik dihapus seluruhnya.
- [frontend/src/app/router/public.jsx](frontend/src/app/router/public.jsx)
  - **Perubahan**: Menghapus route publik `register/:referal?` beserta lazy import halaman tersebut.
- [backend/src/controllers/prospect.controller.js](backend/src/controllers/prospect.controller.js) *(−30 baris)*
  - **Perubahan**: Menghapus handler `registerProspect` (validasi referal → admin, upload foto, buat prospek dari data publik).
- [backend/src/routes/prospect.route.js](backend/src/routes/prospect.route.js)
  - **Perubahan**: Menghapus route publik `POST /prospect/register` dan importnya.
- [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json) · [backend/src/locales/en/translation.json](backend/src/locales/en/translation.json) · [frontend i18n](frontend/src/i18n/locales/id/translations.json)
  - **Perubahan**: Menghapus string terkait (mis. `registrationLink` "Kirim Pendaftaran Prospek").
  - **Fungsi (Area 1)**: Menyederhanakan intake prospek — tidak lagi memelihara endpoint & halaman publik tanpa login.

### 🧩 Area 2 — Alur Baru "Jadikan Prospek" dari Halaman Registrasi

Prospek kini dibuat oleh admin dari data **registrasi pelanggan** yang sudah masuk, lewat drawer prospek yang ter-prefill.

- [frontend/src/app/pages/users/registration/detail.jsx](frontend/src/app/pages/users/registration/detail.jsx) *(+78 baris)*
  - **Perubahan**: Menambah aksi menu **"Jadikan Prospek"** (privilege `prospect.create`) yang: mengambil foto registrasi via `/file/registration-photo/${id}`, lalu membuka `ProspectCreateDrawer` dengan `initialData` (data registrasi) & `initialPhoto`.
  - **Fungsi**: Admin mengonversi registrasi menjadi prospek dalam satu klik, data & foto langsung terisi.
- [frontend/src/app/pages/services/prospect/create.jsx](frontend/src/app/pages/services/prospect/create.jsx) *(+46 baris)*
  - **Perubahan**: `ProspectCreateDrawer` menerima prop `initialData`/`initialPhoto`; saat dibuka dengan data awal, form di-`reset` (nama diambil dari `pic_name` registrasi karena registrasi hanya punya satu field nama) dan `pic_photo` di-set dari foto registrasi.
  - **Fungsi**: Drawer prospek dapat dipakai baik untuk pembuatan kosong maupun ter-prefill dari registrasi.
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) · [en](frontend/src/i18n/locales/en/translations.json)
  - **Perubahan**: Menambah `menu.createProspect` = "Jadikan Prospek".

### 🧩 Area 3 — Integrasi Funnel Dokumen di Detail Prospek

Halaman detail prospek menjadi pusat kendali seluruh dokumen funnel, langsung dari prospek (belum jadi customer).

- [frontend/src/app/pages/services/prospect/detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx) *(+327 / −94 baris — perubahan terbesar)*
  - **Perubahan**: Mengimpor & menyambungkan drawer/komponen dokumen: Quotation (create/detail/preview/edit), Customer PO & SO (review + create), Edit SO, dan **Edit Work Order**; menambah state untuk review Quotation, pembuatan PO/SO dari Quotation, serta pratinjau/edit/hapus Work Order. Menampilkan **foto PIC & foto site** prospek (fetch `/file/prospect-photo/…` & `/file/prospect-site/…`).
  - **Fungsi**: Dari satu halaman prospek, admin dapat menerbitkan & mengelola Quotation → PO → SO → WO tanpa berpindah modul, serta melihat dokumen identitas/lokasi.
- [frontend/src/i18n/locales/*/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Perubahan**: Menambah label pendukung funnel: `createPO` ("Buat Order Pembelian"), `createSOWithoutPO` ("Lanjut ke Order Penjualan tanpa PO"), `customerSignature` ("Tanda Tangan Pelanggan").

### 🧩 Area 4 — Work Order dari SO Milik Prospek

- [frontend/src/app/pages/services/workOrder/create.jsx](frontend/src/app/pages/services/workOrder/create.jsx) *(+27 baris)*
  - **Perubahan**: `CreateWorkOrderDrawer` kini menerima `prospectId` selain `customerId`. Query daftar SO & WO memakai `parentQuery` (`prospect=…` atau `customer=…`), sehingga WO bisa dibuat dari SO milik **prospek** maupun customer. Dependency `useEffect` menyertakan `prospectId`.
  - **Fungsi**: Melengkapi funnel prospek hingga tahap Work Order tanpa harus lebih dulu menjadi customer resmi.

### 🧩 Penyesuaian Pendukung

- [frontend/src/app/pages/services/prospect/index.jsx](frontend/src/app/pages/services/prospect/index.jsx) *(±25 baris)* · [edit.jsx](frontend/src/app/pages/services/prospect/edit.jsx) *(±6)* · [schema/columns.jsx](frontend/src/app/pages/services/prospect/schema/columns.jsx) *(±4)*
  - **Perubahan**: Penyesuaian daftar, form edit, dan kolom tabel prospek mengikuti perubahan alur (mis. sumber data & aksi).
- [backend/package-lock.json](backend/package-lock.json) · [frontend/package-lock.json](frontend/package-lock.json)
  - **Perubahan**: Pembaruan lock dependency.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Admin dapat **"Jadikan Prospek"** langsung dari halaman registrasi pelanggan; data & foto ter-prefill otomatis.
  - Dari detail prospek, admin mengelola seluruh dokumen funnel (Quotation → PO → SO → Work Order) dan melihat foto PIC/site.
  - **Work Order** dapat dibuat dari SO milik prospek, bukan hanya customer resmi.
- **Bug Fix / Solusi Masalah**: Menghapus permukaan publik (halaman + endpoint pendaftaran prospek) yang memperbesar area serang & beban perawatan; alur intake kini terkontrol admin.
- **Menu/Tombol Baru**: Aksi **"Jadikan Prospek"** pada detail registrasi; aksi buat PO/lanjut SO tanpa PO serta pratinjau/edit/hapus Work Order pada detail prospek.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Intake prospek berpindah dari pendaftaran mandiri (publik) ke konversi dari registrasi oleh admin. Setelah menjadi prospek, seluruh dokumen penjualan dapat diterbitkan dan dieksekusi (hingga Work Order) langsung dari halaman detail prospek.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka detail **registrasi** pelanggan → menu **"Jadikan Prospek"** → drawer prospek terbuka dengan data & foto ter-prefill → simpan.
  2. Buka **detail prospek** → terbitkan **Quotation**, lanjut ke **PO** (atau "Lanjut ke SO tanpa PO"), lalu **SO**.
  3. Dari SO prospek yang sudah ditandatangani, buat **Work Order** langsung di detail prospek.

> **Catatan**: Perubahan mencakup penghapusan endpoint/halaman publik & string i18n; pastikan build frontend & backend disegarkan. Nama berkas laporan mengikuti pola `App-v2-<tanggal>` (hari ke-3 Agustus) — sesuaikan bila konvensi penomoran berbeda untuk bulan baru.
