# 📝 Daily Work Report - Idham (2026-06-04)

---

## 📌 Informasi Issue
- **Nomor Issue**: #90
- **Judul Issue**: Implementasi Modul Tiket Backbone

## 📅 Laporan Harian - 4 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Tidak ada pekerjaan yang masih dalam proses. Semua perubahan telah di-commit.

---

### 📅 Rincian Commit

#### [[b2192ae]](b2192ae) - resolve #90 *(Commit awal — struktur modul backbone)*

- **Komponen yang Berubah**:
  - `backend/src/config/privilege.json`
  - `backend/src/controllers/ticket.controller.js`
  - `backend/src/routes/ticket.route.js`
  - `backend/src/services/ticket.service.js`
  - `backend/src/services/warehouseItem.service.js`
  - `backend/src/locales/en/translation.json`
  - `backend/src/locales/id/translation.json`
  - `frontend/src/app/pages/tickets/backbone/index.jsx` [NEW]
  - `frontend/src/app/pages/tickets/backbone/create.jsx` [NEW]
  - `frontend/src/app/pages/tickets/backbone/detail.jsx` [NEW]
  - `frontend/src/app/pages/tickets/backbone/edit.jsx` [NEW]
  - `frontend/src/app/pages/tickets/backbone/close.jsx` [NEW]
  - `frontend/src/app/pages/tickets/backbone/BackboneReport.jsx` [NEW]
  - `frontend/src/app/pages/tickets/backbone/schema/columns.jsx` [NEW]
  - `frontend/src/app/pages/tickets/backbone/schema/createSchema.js` [NEW]
  - `frontend/src/app/pages/tickets/backbone/schema/closeSchema.js` [NEW]
  - `frontend/src/app/router/tickets/backboneRoute.jsx` [NEW]

- **Deskripsi Perubahan & Fungsi**:
  - `privilege.json` — Penambahan definisi privilege baru untuk modul Tiket Backbone: `ticketBackbone.list`, `ticketBackbone.read`, `ticketBackbone.create`, `ticketBackbone.update`, dan `ticketBackbone.delete` ke dalam konfigurasi sistem hak akses.
  - `ticket.controller.js` — Penambahan controller handler untuk seluruh operasi CRUD Tiket Backbone: list dengan filter status, detail, create, update, close (mengubah status tiket menjadi selesai), dan delete. Termasuk handler khusus `list-status` untuk menghitung jumlah tiket per status (new, incomplete, complete, expired) yang ditampilkan sebagai ringkasan di halaman list.
  - `ticket.route.js` — Registrasi seluruh endpoint API Tiket Backbone ke Express router dengan middleware `protectedAdmin` dan `checkPrivilege` sesuai aksi masing-masing.
  - `ticket.service.js`, `warehouseItem.service.js` — Penambahan fungsi service untuk query data tiket backbone dari MongoDB dan kalkulasi pengembalian item warehouse saat tiket ditutup.
  - `locales/en` & `locales/id` — Penambahan kunci terjemahan untuk seluruh label, pesan error, dan notifikasi modul Tiket Backbone di sisi backend.
  - `backbone/index.jsx` — Halaman list Tiket Backbone dengan summary card 4 status (new, incomplete, complete, expired) menggunakan ikon HeroIcons, diikuti komponen `Datatables` dengan kolom dari `getColumns`. Tombol buat tiket baru dikontrol privilege `ticketBackbone.create`.
  - `backbone/create.jsx` — Form pembuatan tiket backbone menggunakan `react-hook-form` dengan `useFieldArray` untuk mengelola daftar item yang digunakan secara dinamis. Validasi menggunakan skema Yup dari `createSchema`.
  - `backbone/detail.jsx` — Halaman detail tiket backbone yang menampilkan informasi lengkap tiket, status, daftar item, dan riwayat update. Dilengkapi tombol aksi (edit, close) yang dikontrol privilege.
  - `backbone/edit.jsx` — Form edit tiket backbone dengan pre-fill data existing dan dukungan update item dinamis via `useFieldArray`.
  - `backbone/close.jsx` — Halaman penutupan tiket backbone dengan form konfirmasi yang menampilkan ringkasan item yang digunakan dan meminta catatan penyelesaian sebelum mengubah status tiket menjadi selesai.
  - `backbone/BackboneReport.jsx` — Komponen cetak laporan tiket backbone dalam format yang siap dicetak/di-export, menampilkan informasi tiket, penanggung jawab, item, dan catatan penyelesaian.
  - `schema/columns.jsx` — Definisi kolom DataTable tiket backbone menggunakan `@tanstack/react-table`, mencakup kolom nomor tiket, judul, status, penanggung jawab, tanggal buat, dan aksi (view, edit, delete).
  - `schema/createSchema.js` — Skema validasi Yup untuk form create dan edit tiket backbone, termasuk validasi array item dengan field nama dan jumlah.
  - `schema/closeSchema.js` — Skema validasi Yup untuk form penutupan tiket, memvalidasi field catatan penyelesaian yang wajib diisi.
  - `backboneRoute.jsx` — Mendefinisikan 5 route lazy-loaded untuk modul Tiket Backbone: list (`/backbone`), create (`/backbone/create`), detail (`/backbone/view/:ticketId`), edit (`/backbone/edit/:ticketId`), dan close (`/backbone/close/:ticketId`), masing-masing dengan privilege handle yang sesuai.

---

#### [[2f859ad]](2f859ad) - resolve #90 *(Commit revisi — penyempurnaan dan integrasi)*

- **Komponen yang Berubah**:
  - `backend/nodemon.json`
  - `frontend/src/app/navigation/tickets.js`
  - `frontend/src/app/pages/tickets/MessageUpdate.jsx`
  - `frontend/src/app/pages/tickets/backbone/close.jsx`
  - `frontend/src/app/pages/tickets/backbone/BackboneReport.jsx`
  - `frontend/src/app/router/protected.jsx`
  - `frontend/src/components/shared/MapRouteDrawer.jsx`
  - `frontend/src/i18n/locales/en/translations.json`
  - `frontend/src/i18n/locales/id/translations.json`
  - `frontend/.env.development`
  - `telegram-apps/.gitignore`
  - `telegram-apps/src/components/shared/AsyncSelect.jsx`

- **Deskripsi Perubahan & Fungsi**:
  - `navigation/tickets.js` — Penambahan entri navigasi sidebar untuk modul Tiket Backbone dengan ikon, path `/tickets/backbone`, dan privilege guard `ticketBackbone.list`, sehingga menu backbone muncul di grup navigasi Tiket.
  - `router/protected.jsx` — Import `ticketBackboneRoute` dan penyebaran routenya ke dalam array `protectedRoutes` di grup Tickets, mendaftarkan seluruh route backbone ke sistem routing aplikasi.
  - `backbone/close.jsx`, `backbone/BackboneReport.jsx` — Penyempurnaan tampilan dan logika halaman close tiket serta komponen laporan berdasarkan hasil review, termasuk perbaikan kalkulasi item dan format tampilan laporan cetak.
  - `MessageUpdate.jsx` — Penambahan minor pada komponen pesan update tiket, kemungkinan dukungan tipe tiket backbone pada sistem notifikasi update.
  - `MapRouteDrawer.jsx` — Penyesuaian komponen drawer peta rute, sinkronisasi dengan kebutuhan detail tiket backbone.
  - `translations.json` (EN & ID) — Penambahan kunci terjemahan frontend untuk seluruh label UI modul Tiket Backbone: judul halaman, label form, status tiket, tombol aksi, dan pesan notifikasi.
  - `frontend/.env.development` — Penyesuaian konfigurasi environment development, kemungkinan penambahan variabel terkait integrasi modul baru.
  - `backend/nodemon.json` — Penyesuaian konfigurasi nodemon untuk kebutuhan development backend.
