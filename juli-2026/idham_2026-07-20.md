# 📝 Daily Work Report - Idham (2026-07-20)

---

## 📌 Informasi Issue
- **Nomor Issue**: #153 (pecahan dari #123)
- **Judul Issue**: Prospect Management — Integrasi Fitur ke Branch `issue-153`, Pemecahan Branch dari #123, & Resolusi Konflik Merge

## 📅 Laporan Harian - 20 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Belum ada commit baru dari Idham hari ini — **sebuah operasi `git merge` sedang berlangsung dan belum di-commit** (masih ada konflik yang harus diselesaikan). Branch aktif: `issue-153` (tracking `origin/issue-153`).
>
> Status merge saat ini: **30 berkas sudah ter-resolve & staged** (±5.194 baris ditambah), menyisakan **2 berkas konflik** yang belum diselesaikan pada berkas terjemahan i18n frontend (`en`: 12 penanda konflik, `id`: 18 penanda konflik).

Hari ini fokusnya bukan menulis fitur baru, melainkan **merapikan dan mengintegrasikan** hasil pekerjaan WIP besar yang selama ini menumpuk di branch `issue-123`. WIP monolitik itu — yang sebelumnya mencampur tiga modul sekaligus (Prospect, Customer Management, Work Order) dalam satu working tree tak-ter-commit — telah dipecah menjadi tiga branch fitur yang bersih dan berdiri sendiri, lalu fitur **Prospect** di-merge ke branch integrasi `issue-153` di atas pekerjaan rekan tim (Dedy) yang sudah lebih dulu masuk (#140, #144, #147, #148).

---

### 🏗️ Konteks & Latar Belakang

Laporan 15–17 Juli mencatat seluruh pekerjaan sebagai WIP tunggal di branch `issue-123` (`save #123`), di mana modul Prospect, Customer Management (Quotation/PO/SO), dan Work Order tercampur dalam satu perubahan tak-ter-commit. Kondisi ini menyulitkan review dan integrasi. Langkah hari ini adalah **restrukturisasi** menjadi tiga branch fitur terpisah:

- `issue-123-new` → **Customer Management** (Quotation / PO / SO) — commit `f5ccbb9`.
- `issue-153-new` → **Prospect** — commit `db8549a` ("resolve #153 - fitur Prospect").
- `issue-154-new` → **Work Order** — commit `35668c9` ("resolve #154 - fitur Work Order").

Dengan pemisahan ini, tiap fitur bisa di-review, di-merge, dan dirilis secara independen mengikuti nomor issue-nya masing-masing. Pekerjaan konkret 20 Juli adalah menuntaskan tahap pertama integrasi: **me-merge `issue-153-new` (Prospect) ke branch `issue-153`**. Karena branch tujuan sudah memuat perubahan Dedy yang menyentuh berkas bersama (konfigurasi aplikasi, privilege, dan berkas terjemahan), muncul konflik yang harus diselesaikan secara manual.

---

### 🔀 Rincian Operasi Merge

**Merge**: `issue-153-new` → `issue-153` (`Merge branch 'issue-153-new' into issue-153`)

- **Konflik yang SUDAH diselesaikan (staged)**:
  - [backend/src/app.js](backend/src/app.js) — penggabungan registrasi modul/route aplikasi.
  - [backend/src/config/privilege.json](backend/src/config/privilege.json) — penyatuan definisi privilege `prospect` dengan blok privilege lain yang ditambahkan Dedy.
- **Konflik yang MASIH terbuka (belum di-`git add`)**:
  - [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) — 18 penanda konflik.
  - [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) — 12 penanda konflik.
  - **Penyebab**: kedua sisi merge sama-sama menambahkan blok kunci terjemahan yang besar. Sisi `HEAD` (branch integrasi) membawa blok seperti `notifications` (judul, empty-state per kategori, tipe), sedangkan sisi Prospect membawa blok `quotation`/Customer Management dan kunci-kunci Prospect. Keduanya perlu digabung — bukan salah satu dibuang — agar seluruh string tetap tersedia di dua bahasa.

### 🗂️ Berkas Fitur Prospect yang Masuk lewat Merge (staged, sebagian berkas baru)

Backend:
- [backend/src/controllers/prospect.controller.js](backend/src/controllers/prospect.controller.js) · [backend/src/models/prospect.model.js](backend/src/models/prospect.model.js) · [backend/src/routes/prospect.route.js](backend/src/routes/prospect.route.js) · [backend/src/services/prospect.service.js](backend/src/services/prospect.service.js) · [backend/src/services/prospectPhase.service.js](backend/src/services/prospectPhase.service.js)
  - **Deskripsi**: Seluruh backend modul Prospect — CRUD prospek, pendaftaran publik, foto survei, serta logika penguncian bertahap (*phase-lock*) — kini masuk sebagai berkas resmi (bukan lagi WIP), termasuk penyesuaian [files.controller.js](backend/src/controllers/files.controller.js), [files.route.js](backend/src/routes/files.route.js), dan berkas terjemahan backend.

Frontend (banyak berkas baru, termasuk yang belum pernah muncul di laporan 15–17):
- [frontend/src/app/pages/services/prospect/convert.jsx](frontend/src/app/pages/services/prospect/convert.jsx) **[NEW]** — halaman konversi prospek menjadi customer resmi.
- [frontend/src/app/pages/services/prospect/report.jsx](frontend/src/app/pages/services/prospect/report.jsx) **[NEW]** — halaman laporan/rekap prospek.
- [frontend/src/app/pages/services/prospect/schema/statusOptions.js](frontend/src/app/pages/services/prospect/schema/statusOptions.js) **[NEW]** — opsi status prospek terpusat.
- [frontend/src/app/router/services/prospectRoute.jsx](frontend/src/app/router/services/prospectRoute.jsx) **[NEW]** — definisi route modul Prospect (terproteksi).
- [frontend/src/app/pages/public/prospectRegistration.jsx](frontend/src/app/pages/public/prospectRegistration.jsx) · [create.jsx](frontend/src/app/pages/services/prospect/create.jsx) · [detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx) · [edit.jsx](frontend/src/app/pages/services/prospect/edit.jsx) · [index.jsx](frontend/src/app/pages/services/prospect/index.jsx) · [phaseLock.js](frontend/src/app/pages/services/prospect/phaseLock.js) · [schema/columns.jsx](frontend/src/app/pages/services/prospect/schema/columns.jsx) · [schema/prospectSchema.js](frontend/src/app/pages/services/prospect/schema/prospectSchema.js)
  - **Deskripsi**: Seluruh halaman & schema frontend Prospect.
- [frontend/src/app/navigation/services.js](frontend/src/app/navigation/services.js) · [frontend/src/app/router/protected.jsx](frontend/src/app/router/protected.jsx) · [frontend/src/app/router/public.jsx](frontend/src/app/router/public.jsx)
  - **Deskripsi**: Pendaftaran menu navigasi & route (terproteksi dan publik) untuk modul Prospect.
- [frontend/src/components/shared/table/DocumentActionsMenu.jsx](frontend/src/components/shared/table/DocumentActionsMenu.jsx) **[NEW]** — menu aksi dokumen bersama.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengembang/Tim**: Kode fitur Prospect, Customer Management, dan Work Order kini terpisah rapi ke tiga branch berbasis issue (#153, #123-new, #154-new), sehingga masing-masing dapat di-review dan di-merge secara mandiri — memangkas risiko satu perubahan besar yang sulit ditinjau.
- **Bug Fix / Solusi Masalah**: Menyelesaikan konflik integrasi pada `app.js` dan `privilege.json` agar registrasi modul & hak akses `prospect` menyatu dengan pekerjaan rekan tim tanpa saling menimpa. Sisa konflik terjemahan i18n sedang dikerjakan dengan pendekatan gabung-kedua-sisi.
- **Menu/Tombol Baru**: Setelah merge tuntas, modul **Prospect** hadir sebagai menu navigasi resmi (services), lengkap dengan halaman daftar, detail, buat/edit, konversi ke customer, laporan, dan form pendaftaran publik.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Hari ini adalah tahap *integrasi*, bukan penambahan fitur baru. Modul Prospect yang sebelumnya berupa WIP menumpuk diangkat menjadi commit resmi di branch `issue-153-new`, lalu di-merge ke branch integrasi `issue-153` bersama pekerjaan tim lain. Konflik muncul pada berkas bersama (konfigurasi, privilege, terjemahan) dan diselesaikan dengan menggabungkan kedua sisi.
- **Langkah Penyelesaian (untuk melanjutkan pekerjaan)**:
  1. Buka kedua berkas i18n yang konflik ([id](frontend/src/i18n/locales/id/translations.json), [en](frontend/src/i18n/locales/en/translations.json)).
  2. Untuk setiap penanda `<<<<<<< / ======= / >>>>>>>`, **gabungkan** blok kunci dari kedua sisi (mis. `notifications` dari HEAD **dan** `quotation`/kunci Prospect dari sisi merge) — jangan buang salah satu.
  3. Pastikan struktur JSON tetap valid (koma, kurung) dan kunci `en` sepadan dengan `id`.
  4. `git add` kedua berkas, lalu `git commit` untuk menuntaskan merge; verifikasi aplikasi berjalan dan terjemahan tampil benar di kedua bahasa.
