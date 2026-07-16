# 📝 Daily Work Report - Idham (2026-07-16)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Prospect Management — Modul Funnel Pra-Deal (Fase Dokumen: Penguncian Bertahap Dokumen Prospek / *Phase-Lock* Model A)

## 📅 Laporan Harian - 16 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Tidak ada commit baru hari ini (commit terakhir tetap `7aa4cee save #123` tertanggal 2026-07-13). **Seluruh pekerjaan 16 Juli 2026 masih berstatus WIP** di branch `issue-123`.
>
> **Fokus hari ini** adalah menambahkan **penguncian bertahap dokumen prospek (*phase-lock*, Model A)** di atas fondasi kemarin. Delta terhadap laporan 15 Juli: 2 berkas baru (`prospectPhase.service.js`, `phaseLock.js`), `customerDocument.controller.js` kini ikut berubah, guard fase ditanam pada controller Quotation/PO/SO/WO, dan integrasi UI pada detail prospek. Total working tree sekarang 43 berkas berubah + 9 berkas baru (±2.699 baris tambah / ±1.269 hapus).

---

### 🏗️ Konteks & Arsitektur — Phase-Lock (Model A)

Dokumen prospek mengalir melalui **4 fase berurutan**: `1 Quotation → 2 PO → 3 SO → 4 Work Order`. Aturan bisnis (keputusan terkunci):

- **Satu dokumen "aktif" per fase** (keputusan #3).
- Sebuah fase menjadi **read-only** bila: dokumen aktifnya sudah **FINAL**, **atau** fase berikutnya sudah mulai (sudah ada dokumen aktif). Hanya **fase terdepan** yang boleh diedit.
- Definisi FINAL per fase: Quotation FINAL saat `approval` terisi (keputusan #2); PO FINAL saat `approval`/`complete`; SO FINAL saat `approval`/`complete`/`status = signed`; WO FINAL saat `status = done`.
- Definisi "mati" (tidak dihitung aktif): Quotation `rejected`/`expired`/`lost`; WO `cancelled`. Reject PO/SO = soft-delete sehingga yang tersisa di list pasti masih hidup.

Logika inti murni tanpa I/O (`computeProspectPhase`) sehingga **dapat dipakai ulang & mudah diuji**, dan sengaja **dicerminkan di backend dan frontend** agar penegakan aturan (guard API) dan tampilan (sembunyikan tombol) konsisten.

---

### 🗂️ Backend — Berkas Baru

- [backend/src/services/prospectPhase.service.js](backend/src/services/prospectPhase.service.js) **[NEW]**
  - **Deskripsi**: Inti fitur phase-lock. Ekspor `PROSPECT_PHASE`, `computeProspectPhase` (fungsi murni → `started`/`final`/`nextStarted`/`locked`/`canCreate` + dokumen aktif per fase), `loadProspectDocuments` (memuat quotation/PO/SO/WO paralel via `Promise.all`), dan `getProspectPhase(prospectId)` sebagai entry point controller.

### 🗂️ Backend — Berkas yang Berubah (integrasi guard fase)

- [backend/src/controllers/customerQuotation.controller.js](backend/src/controllers/customerQuotation.controller.js)
  - **Deskripsi**: Memanggil `getProspectPhase` saat create/edit/delete quotation untuk menolak aksi bila fase Quotation terkunci (`locked`/`canCreate`).
- [backend/src/controllers/customerDocument.controller.js](backend/src/controllers/customerDocument.controller.js) *(mulai berubah hari ini)*
  - **Deskripsi**: Menanam guard fase pada alur dokumen PO & SO (create/edit/reject) — mencegah perubahan dokumen pada fase yang sudah read-only.
- [backend/src/controllers/workOrder.controller.js](backend/src/controllers/workOrder.controller.js)
  - **Deskripsi**: Guard fase pada pembuatan Work Order (fase 4) — WO hanya boleh dibuat bila fase sebelumnya sudah final & fase WO belum terkunci.
- [backend/src/locales/en/translation.json](backend/src/locales/en/translation.json) · [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json)
  - **Deskripsi**: Pesan error penolakan aksi karena fase terkunci.

### 🖥️ Frontend — Berkas Baru

- [frontend/src/app/pages/services/prospect/phaseLock.js](frontend/src/app/pages/services/prospect/phaseLock.js) **[NEW]**
  - **Deskripsi**: Cermin frontend dari `prospectPhase.service.js`. Ekspor `PROSPECT_PHASE` & `computeProspectPhase` dengan aturan `locked`/`canCreate` identik, dipakai halaman detail prospek untuk menyembunyikan tombol Buat/Edit/Hapus dan menandai fase yang terkunci.

### 🖥️ Frontend — Berkas yang Berubah

- [frontend/src/app/pages/services/prospect/detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx)
  - **Deskripsi**: Menghitung `phase = computeProspectPhase({ quotations, poList, soList, woList })` dari dokumen yang dimuat, lalu memakai `phase.locked`/`phase.canCreate` untuk mengondisikan tombol aksi dokumen dan badge status "Terkunci" per fase.
- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) · [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: String i18n baru untuk phase-lock (mis. `locked` = "Terkunci") — masing-masing tumbuh ±108 baris.

> **Catatan**: Sisa berkas WIP (pendaftaran prospek publik, foto survei, PIC penawaran, dokumen & pratinjau Work Order, `DocumentActionsMenu`, dll.) tetap sama seperti dilaporkan pada [15 Juli 2026](idham_2026-07-15.md) dan masih menunggu commit.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**: Alur dokumen prospek kini ditegakkan secara berurutan — sistem otomatis mengunci fase yang sudah selesai/final sehingga admin tidak bisa lagi mengubah dokumen di fase lama setelah maju ke fase berikutnya. Tombol Buat/Edit/Hapus hilang otomatis pada fase yang terkunci.
- **Bug Fix / Solusi Masalah**: Menutup celah inkonsistensi data di mana dokumen fase awal (mis. Quotation) masih bisa diubah setelah PO/SO/WO dibuat. Penegakan ganda (guard API + UI) memastikan pengguna tidak bisa melewati aturan lewat request langsung.
- **Menu/Tombol Baru**: Badge/indikator status **"Terkunci"** per fase pada halaman detail prospek; tombol aksi dokumen kini muncul kondisional sesuai fase.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: *Phase-lock* mengatur agar funnel dokumen prospek (Quotation → PO → SO → Work Order) hanya bisa disunting pada fase terdepan. Logika dihitung dari status dokumen yang ada (aktif vs mati, final vs belum) dan diterapkan sama persis di backend (menolak API) maupun frontend (menyembunyikan tombol).
- **Langkah Penggunaan (Tutorial)**:
  1. Buka detail sebuah prospek yang memiliki dokumen.
  2. Selama Quotation belum final dan belum ada PO, tombol Edit/Hapus Quotation tersedia.
  3. Setelah Quotation di-approve (final) atau setelah PO dibuat, fase Quotation otomatis menjadi read-only dan ditandai "Terkunci".
  4. Coba edit dokumen di fase terkunci lewat API → ditolak dengan pesan error fase terkunci (guard `getProspectPhase`).
  5. Buat dokumen fase berikutnya hanya ketika fase saat ini sudah final dan fase tujuan belum dimulai (`canCreate`).
