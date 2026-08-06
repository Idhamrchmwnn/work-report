# 📝 Daily Work Report - Idham (2026-08-06)

---

## 📌 Informasi Issue
- **Nomor Issue**: — (fitur baru; branch `master`)
- **Judul Issue**: Customer PKS — Modul Dokumen Perjanjian Kerja Sama (PKS) untuk Mitra: Buat/Kelola, Persetujuan Internal, Tanda Tangan Digital Publik, & Unggah PKS Bertanda Tangan

## 📅 Laporan Harian - 6 Agustus 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Belum ada commit hari ini. Bekerja di branch **`master`**. Working tree: **15 berkas diubah + 8 berkas/direktori baru** — ±620 baris pada berkas termodifikasi, dan ±3.370 baris pada berkas baru (total ± 4.000 baris). Merupakan **modul fitur baru penuh**, belum di-stage.

Hari ini membangun modul **Customer PKS (Perjanjian Kerja Sama)** — dokumen kerja sama antara perusahaan dan **Mitra (Partner)**. Polanya mengikuti modul dokumen penjualan yang sudah ada (Quotation/PO/SO): sisi **internal** (admin membuat/mengelola/menyetujui) dan sisi **publik** (mitra meninjau & menandatangani lewat tautan ber-token). Siklus status: **`draft → sent → signed`**, dengan langkah **persetujuan internal** dan opsi **unggah berkas PKS bertanda tangan**.

---

### 🧩 Backend — Berkas Baru

- [backend/src/models/customerPKS.model.js](backend/src/models/customerPKS.model.js) **[NEW]** *(121 baris)*
  - **Perubahan**: Skema PKS milik **Partner** (`partner` ref, wajib). Field utama: `pks_number` (auto-urut via counter `pid`/`seq`), `title`, `status` (`draft`/`sent`/`signed`), `pks_date`/`effective_date`/`expiry_date` (masa berlaku), `terms`, `notes`; alur persetujuan `approval` (Admin) + `approved_at` + `complete`; tanda tangan digital `signature`/`pic_signature`/`signed_at`/`signer_name`/`signed_by`; `share_token` (tautan publik); serta opsi unggah `signed_file`/`signed_file_uploaded`.
  - **Fungsi**: Sumber data tunggal dokumen PKS beserta status, masa berlaku, persetujuan, dan tanda tangan.
- [backend/src/services/customerPKS.service.js](backend/src/services/customerPKS.service.js) **[NEW]** *(225 baris)*
  - **Perubahan**: Lapisan data PKS — pembuatan (dengan penomoran urut), pencarian by id/token/partner, pembaruan, penghapusan, dan pemuatan dengan populate partner/approval.
  - **Fungsi**: Query & mutasi PKS yang dipakai controller.
- [backend/src/controllers/customerPKS.controller.js](backend/src/controllers/customerPKS.controller.js) **[NEW]** *(349 baris)*
  - **Perubahan**: Handler internal — `createPKS`, `listPKS`/`listAllPKS`, `readPKS`, `updatePKS`, `deletePKS`, `submitPKS` (ajukan persetujuan), `approvePKS`/`rejectPKS` (persetujuan internal), `sendPKS` (kirim ke mitra → status `sent` + `share_token`), `requestPKSPreview`, dan `uploadSignedPKS` (unggah PKS bertanda tangan).
  - **Fungsi**: Seluruh alur kerja PKS dari sisi admin.
- [backend/src/controllers/publicCustomerPKS.controller.js](backend/src/controllers/publicCustomerPKS.controller.js) **[NEW]** *(85 baris)*
  - **Perubahan**: `getPKSByToken` (mitra membuka dokumen via token) & `signPKSByToken` (mitra menandatangani digital → status `signed`).
  - **Fungsi**: Sisi publik tanpa login bagi mitra untuk meninjau & menandatangani.
- [backend/src/routes/customerPKS.route.js](backend/src/routes/customerPKS.route.js) **[NEW]** *(407 baris)*
  - **Perubahan**: Route internal PKS dengan privilege `customerPKS.*` (create/read/update/delete/changeStatus) beserta dokumentasi Swagger.
  - **Fungsi**: Menegakkan hak akses & mendokumentasikan endpoint.

### 🧩 Backend — Berkas yang Diubah (wiring)

- [backend/src/app.js](backend/src/app.js) *(+2)* — mendaftarkan `CustomerPKSRoute` di `/api/v1`.
- [backend/src/routes/public.route.js](backend/src/routes/public.route.js) *(+12)* — route publik ber-token: `GET /public-docs/customer-pks/:token` & `POST /public-docs/customer-pks/sign`.
- [backend/src/config/privilege.json](backend/src/config/privilege.json) *(+7)* — blok privilege `customerPKS` (`CUSTOMERPKS_CREATE/READ/CHANGESTATUS/UPDATE/DELETE`).
- [backend/src/utils/telegram.js](backend/src/utils/telegram.js) *(+51)* — `TelegramNotifCustomerPKSSubmit`: notifikasi **"PKS — MENUNGGU PERSETUJUAN"** (nomor, judul, tanggal) + tautan **"BERIKAN PERSETUJUAN"** ke halaman review.
- [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json) · [en](backend/src/locales/en/translation.json) *(+21 masing-masing)* — string backend PKS.

### 🧩 Frontend — Berkas Baru

- [frontend/src/app/pages/users/customerPKS/create.jsx](frontend/src/app/pages/users/customerPKS/create.jsx) **[NEW]** *(218)* · [edit.jsx](frontend/src/app/pages/users/customerPKS/edit.jsx) **[NEW]** *(273)* · [schema/customerPKSSchema.js](frontend/src/app/pages/users/customerPKS/schema/customerPKSSchema.js) **[NEW]** *(20)*
  - **Perubahan**: Drawer buat/ubah PKS (nomor, judul, tanggal & masa berlaku, terms, catatan) + validasi schema.
  - **Fungsi**: Admin membuat/menyunting PKS untuk mitra.
- [CustomerPKSDocumentPreview.jsx](frontend/src/app/pages/users/customerPKS/CustomerPKSDocumentPreview.jsx) **[NEW]** *(240)* · [CustomerPKSReviewDrawer.jsx](frontend/src/app/pages/users/customerPKS/CustomerPKSReviewDrawer.jsx) **[NEW]** *(549)*
  - **Perubahan**: Lembar dokumen PKS untuk pratinjau, dan drawer peninjauan (approve/reject/kirim, tanda tangan/riwayat).
  - **Fungsi**: Pratinjau & alur persetujuan/pengiriman dari sisi internal.
- [frontend/src/app/pages/public/PublicCustomerPKSDocument.jsx](frontend/src/app/pages/public/PublicCustomerPKSDocument.jsx) **[NEW]** *(519)* · [ReviewCustomerPKSPage.jsx](frontend/src/app/pages/public/ReviewCustomerPKSPage.jsx) **[NEW]** *(364)*
  - **Perubahan**: Halaman publik dokumen PKS & halaman review-tanda-tangan bagi mitra (via token).
  - **Fungsi**: Mitra meninjau & menandatangani PKS tanpa login.

### 🧩 Frontend — Berkas yang Diubah (wiring)

- [frontend/src/app/pages/users/partner/profile.jsx](frontend/src/app/pages/users/partner/profile.jsx) *(+224)*
  - **Perubahan**: Menambah **tab/section PKS** pada profil Mitra — daftar PKS, buat (`CreateCustomerPKSDrawer`), edit, pratinjau, sel status (`CustomerPKSStatusCell`), dengan state pemuatan tersendiri.
  - **Fungsi**: Titik masuk utama pengelolaan PKS per mitra.
- [frontend/src/components/shared/table/rows.jsx](frontend/src/components/shared/table/rows.jsx) *(+96)*
  - **Perubahan**: `CustomerPKSStatusCell` — sel status PKS (draft/sent/signed) mengikuti lifecycle.
  - **Fungsi**: Menampilkan & (bila berwenang) menggerakkan status PKS di tabel.
- [frontend/src/components/shared/DocumentPreviewModal.jsx](frontend/src/components/shared/DocumentPreviewModal.jsx) *(+33)*
  - **Perubahan**: Dukungan tipe dokumen `customer-pks` pada modal pratinjau bersama.
  - **Fungsi**: Pratinjau PKS memakai komponen modal yang sama dengan dokumen lain.
- [frontend/src/app/router/protected.jsx](frontend/src/app/router/protected.jsx) *(+7)* · [frontend/src/app/router/public.jsx](frontend/src/app/router/public.jsx) *(+8)*
  - **Perubahan**: Route internal & publik (review PKS) didaftarkan.
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) · [en](frontend/src/i18n/locales/en/translations.json) *(+65 masing-masing)* · [privilegeDescriptions.id.json](frontend/src/constants/privilegeDescriptions.id.json) · [en](frontend/src/constants/privilegeDescriptions.en.json) *(+5)*
  - **Perubahan**: String UI PKS dua bahasa & deskripsi privilege `customerPKS`.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Membuat & mengelola dokumen **PKS (Perjanjian Kerja Sama)** per Mitra (nomor otomatis, judul, masa berlaku, terms).
  - Alur **persetujuan internal** (submit → approve/reject) dengan notifikasi Telegram ke penyetuju.
  - **Kirim ke mitra**; mitra meninjau & **menandatangani digital** lewat tautan publik ber-token — atau admin **mengunggah PKS bertanda tangan**.
  - Pratinjau dokumen PKS lewat modal bersama; pantau status (draft/sent/signed) dari profil mitra.
- **Bug Fix / Solusi Masalah**: — (fitur baru; belum ada perbaikan bug).
- **Menu/Tombol Baru**: **Tab/section PKS** pada profil Mitra (buat/edit/pratinjau/kirim/setujui), halaman **review PKS publik** untuk mitra.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: PKS adalah dokumen kerja sama dengan Mitra yang mengalir dari draft → dikirim → ditandatangani. Terdapat langkah persetujuan internal sebelum dikirim, dan mitra dapat menandatangani secara digital via tautan publik (atau berkas bertanda tangan diunggah admin).
- **Langkah Penggunaan (Tutorial)**:
  1. **Buat PKS**: buka profil Mitra → tab PKS → "Buat PKS" (isi judul, tanggal, masa berlaku, terms).
  2. **Ajukan & setujui**: submit untuk persetujuan → penyetuju menerima notifikasi Telegram → approve/reject.
  3. **Kirim ke mitra**: kirim PKS (status `sent`); sistem menyediakan tautan publik.
  4. **Tanda tangan**: mitra membuka tautan review → menandatangani digital (status `signed`); atau admin mengunggah PKS yang sudah ditandatangani.
  5. **Pantau**: status PKS (draft/sent/signed) terlihat di tab PKS profil mitra; dokumen bisa dipratinjau lewat modal.

> **Catatan teknis**: Modul menambah model, route, privilege, dan notifikasi baru — **restart backend** diperlukan; pastikan role admin diberi privilege `CUSTOMERPKS_*`. Seluruh perubahan masih WIP (belum di-stage/commit). Nama berkas laporan mengikuti pola `App-v2-<tanggal>` (hari ke-6 Agustus).
