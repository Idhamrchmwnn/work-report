# 📝 Daily Work Report - Idham (2026-07-28)

---

## 📌 Informasi Issue
- **Nomor Issue**: #153 (pecahan dari #123)
- **Judul Issue**: Prospect Management — Finalisasi Modul Funnel Pra-Deal & Penyelarasan Integrasi Lintas-Modul (Import Path + Prop Drawer)

## 📅 Laporan Harian - 28 Juli 2026

Hari ini terdiri atas dua bagian yang saling melengkapi: **(1)** menuntaskan dan **meng-commit** modul **Prospect Management** secara utuh di branch `issue-153`, lalu **(2)** memulai serangkaian **perbaikan integrasi lintas-modul** (masih WIP) agar halaman detail prospek dapat memanggil komponen milik modul Customer Management yang belakangan direlokasi. Branch aktif: `issue-153`, bertengger di atas pekerjaan tim terbaru (`resolve #161`, Dedy).

Dengan commit hari ini, ketiga branch fitur hasil pemecahan dari WIP monolitik #123 kini lengkap dan berdiri sendiri: **#153 Prospect** (hari ini), **#123 Customer Management** (21–22 Juli), dan **#154 Work Order** (27 Juli). Sisa pekerjaan WIP berfokus pada "lem" integrasi antar-modul tersebut — bagian yang wajar muncul ketika tiga modul yang semula satu kesatuan dipisah lalu perlu saling memanggil kembali.

---

### 🏗️ Konteks & Latar Belakang

Modul **Prospect Management** memperkenalkan entitas *pra-deal* ke dalam ekosistem Customer Management. Sebelum modul ini, sistem langsung mencatat entitas sebagai Partner (customer resmi) yang mensyaratkan data lengkap. Dengan Prospect, tim marketing dapat mencatat lead/prospek secara ringan lebih dahulu — termasuk dari **pendaftaran publik via link referal** — lalu mengonversikannya menjadi customer resmi setelah deal terjadi. Modul ini juga membawa mekanisme **penguncian bertahap dokumen (*phase-lock*, Model A)** yang menjaga agar dokumen funnel (`Quotation → PO → SO → Work Order`) hanya dapat disunting pada fase terdepan.

Karena halaman **detail prospek** berperan sebagai pusat kendali seluruh funnel, ia perlu memanggil banyak komponen milik modul lain: drawer pembuatan/edit Quotation, drawer review Customer PO & SO, serta drawer & pratinjau Work Order. Ketika modul Customer Management direlokasi ke direktori `users/` (hasil audit sebelumnya) dan Work Order berdiri di `services/workOrder/`, jalur import lama pada detail prospek menjadi tidak valid. Pekerjaan WIP hari ini menutup celah tersebut sekaligus menyelaraskan kontrak antar-komponen (penamaan prop drawer).

---

### 📅 Rincian Commit

#### [`f93d335`] - resolve #153 - fitur Prospect (#153 - Prospect Management)

- **Ringkasan**: 27 berkas berubah, ±4.850 baris ditambah / ±34 dihapus. Modul Prospect end-to-end, dari backend hingga frontend, resmi menjadi commit bersih di branch `issue-153`.
- **Komponen yang Berubah — Backend**:
  - [backend/src/controllers/prospect.controller.js](backend/src/controllers/prospect.controller.js) `[NEW]` · [backend/src/services/prospect.service.js](backend/src/services/prospect.service.js) `[NEW]` · [backend/src/routes/prospect.route.js](backend/src/routes/prospect.route.js) `[NEW]` · [backend/src/models/prospect.model.js](backend/src/models/prospect.model.js)
    - **Deskripsi**: CRUD prospek, pendaftaran publik (`registerProspect` via link referal marketing), unggah foto survei (KTP PIC & lokasi), serta field survei (koordinat, jenis & nomor KTP, foto). Pesan error bisnis dilempar sebagai kode konstanta yang diterjemahkan di frontend.
  - [backend/src/services/prospectPhase.service.js](backend/src/services/prospectPhase.service.js) `[NEW]`
    - **Deskripsi**: Inti logika penguncian bertahap — `computeProspectPhase` (fungsi murni), `loadProspectDocuments`, dan `getProspectPhase(prospectId)` sebagai entry point guard di controller dokumen.
  - [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js) · [backend/src/routes/files.route.js](backend/src/routes/files.route.js)
    - **Deskripsi**: Penyajian foto prospek — foto KTP PIC dengan watermark nama admin (data sensitif) & foto lokasi tanpa watermark; route `GET /file/prospect-photo/:id` & `/file/prospect-site/:id` (privilege `prospect.read`).
  - [backend/src/app.js](backend/src/app.js) · [backend/src/config/privilege.json](backend/src/config/privilege.json)
    - **Deskripsi**: Registrasi route & definisi privilege modul `prospect` (read/create/list/update/delete/convert).
- **Komponen yang Berubah — Frontend**:
  - **Halaman inti**: [index.jsx](frontend/src/app/pages/services/prospect/index.jsx) `[NEW]`, [create.jsx](frontend/src/app/pages/services/prospect/create.jsx) `[NEW]`, [edit.jsx](frontend/src/app/pages/services/prospect/edit.jsx) `[NEW]`, [detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx) `[NEW]`, [convert.jsx](frontend/src/app/pages/services/prospect/convert.jsx) `[NEW]`, [report.jsx](frontend/src/app/pages/services/prospect/report.jsx) `[NEW]`
    - **Deskripsi**: Daftar, buat, edit, detail (pusat kendali funnel), konversi prospek → customer, dan laporan/rekap prospek.
  - **Logika & schema**: [phaseLock.js](frontend/src/app/pages/services/prospect/phaseLock.js) `[NEW]`, [schema/columns.jsx](frontend/src/app/pages/services/prospect/schema/columns.jsx) `[NEW]`, [schema/prospectSchema.js](frontend/src/app/pages/services/prospect/schema/prospectSchema.js) `[NEW]`, [schema/statusOptions.js](frontend/src/app/pages/services/prospect/schema/statusOptions.js) `[NEW]`
    - **Deskripsi**: Cermin frontend dari phase-lock backend, definisi kolom tabel, validasi form, dan opsi status prospek terpusat.
  - **Publik, route & navigasi**: [prospectRegistration.jsx](frontend/src/app/pages/public/prospectRegistration.jsx) `[NEW]`, [router/services/prospectRoute.jsx](frontend/src/app/router/services/prospectRoute.jsx) `[NEW]`, [router/protected.jsx](frontend/src/app/router/protected.jsx), [router/public.jsx](frontend/src/app/router/public.jsx), [navigation/services.js](frontend/src/app/navigation/services.js), terjemahan [en](frontend/src/i18n/locales/en/translations.json)/[id](frontend/src/i18n/locales/id/translations.json)
    - **Deskripsi**: Form pendaftaran prospek publik (`register/:referal`), pendaftaran route (terproteksi & publik), menu navigasi, dan string dua bahasa.
- **Deskripsi Perubahan & Fungsi**: Menghadirkan funnel pra-deal lengkap — dari pencatatan lead (termasuk pendaftaran mandiri oleh calon pelanggan), pengelolaan dokumen berfase dengan penguncian bertahap, hingga konversi menjadi customer resmi — beserta laporan prospek dan penyajian foto survei yang aman.

---

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Setelah commit di atas, dimulai **perbaikan integrasi lintas-modul** pada halaman detail prospek. Masih **WIP dan belum ter-stage** — **2 berkas** (±22 baris ditambah / ±17 dihapus).

Perubahan ini kecil dari sisi jumlah baris namun penting secara fungsional: tanpa penyesuaian ini, halaman detail prospek gagal mengimpor komponen dokumen milik modul Customer Management/Work Order yang lokasinya sudah berpindah, dan drawer detail Quotation tidak menerima data dengan benar karena beda penamaan prop.

- [frontend/src/app/pages/services/prospect/detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx)
  - **Deskripsi**:
    - **Perbaikan jalur import**: mengganti impor relatif lama (mis. `../quotation/...`, `../activation/components/CustomerPOReviewDrawer`, `../customerPurchaseOrder/...`) menjadi jalur absolut sesuai lokasi baru pasca-relokasi — `app/pages/users/quotation/*`, `app/pages/users/customerPurchaseOrder/*`, `app/pages/users/customerSalesOrder/*`, dan `app/pages/services/workOrder/*`. Ini mengembalikan kemampuan detail prospek memanggil drawer Create/Edit Quotation, review Customer PO/SO, serta drawer & pratinjau Work Order.
    - **Penyelarasan prop drawer**: pemanggilan `QuotationDetailDrawer` diperbaiki dari `open`/`cellData` menjadi `isOpen`/`data` agar sesuai kontrak komponen tujuan.
- [frontend/src/app/pages/users/quotation/QuotationDetailDrawer.jsx](frontend/src/app/pages/users/quotation/QuotationDetailDrawer.jsx)
  - **Deskripsi**: Membuat komponen **kompatibel dua arah** terhadap penamaan prop — menerima `isOpen` maupun `open` (`isOpen ?? open`) dan `data` maupun `cellData` (`data ?? cellData`). Ini menjembatani perbedaan konvensi antar-pemanggil (detail prospek vs. halaman lain) sehingga keduanya tetap bekerja tanpa memaksa perubahan serentak di semua tempat.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Tim marketing dapat mencatat & mengelola prospek (lead) secara ringan, menerima pendaftaran mandiri calon pelanggan lewat link referal publik, mengelola dokumen funnel dengan penguncian bertahap, dan mengonversi prospek menjadi customer resmi.
  - Tersedia halaman laporan/rekap prospek serta penyajian foto survei yang aman (KTP ber-watermark).
- **Bug Fix / Solusi Masalah**:
  - (WIP) Memperbaiki jalur import yang rusak pada halaman detail prospek akibat relokasi modul Customer Management ke `users/` — mengembalikan fungsi pembuatan/pratinjau dokumen lintas-fase dari satu layar.
  - (WIP) Menyelaraskan penamaan prop `QuotationDetailDrawer` dan membuatnya kompatibel dua arah, mencegah drawer gagal menerima data saat dipanggil dari konteks berbeda.
- **Menu/Tombol Baru**: Menu navigasi & route **Prospect** (daftar, detail, buat/edit, konversi, laporan) serta form **pendaftaran prospek publik** (`/register/:referal`).

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Prospect Management adalah funnel pra-deal yang menghubungkan lead hingga menjadi customer. Halaman detail prospek menjadi pusat kendali yang menampilkan seluruh dokumen funnel dan menegakkan aturan penguncian bertahap. Karena memanggil komponen milik modul lain, integrasi jalur import & kontrak prop antar-komponen menjadi krusial agar seluruh aksi dokumen berjalan dari satu tempat.
- **Langkah Penggunaan (Tutorial)**:
  1. **Catat/daftarkan prospek**: buat prospek dari menu Prospect, atau calon pelanggan mendaftar sendiri via `/register/<admin_id_marketing>` (lengkap dengan foto KTP PIC, foto lokasi, koordinat).
  2. **Kelola funnel dari detail**: buka detail prospek untuk menerbitkan/mengedit/mempratinjau Quotation, Customer PO, Customer SO, dan Work Order — tombol muncul kondisional mengikuti fase (yang terkunci ditandai "Terkunci").
  3. **Pantau & laporkan**: gunakan halaman laporan prospek untuk rekap.
  4. **Konversi**: setelah deal, jalankan konversi prospek → customer resmi.
