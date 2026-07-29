# 📝 Daily Work Report - Idham (2026-07-29)

---

## 📌 Informasi Issue
- **Nomor Issue**: #154
- **Judul Issue**: Work Order — Generalisasi Jenis WO, Penyatuan Pratinjau Dokumen, Tautan Koordinat, Penanggung Jawab (Sinkron Tiket ↔ WO) & Dokumen WO Full i18n

## 📅 Laporan Harian - 29 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Belum ada commit hari ini. Seluruh pekerjaan masih **WIP** di branch `issue-154` (di atas basis `resolve #156`, Dedy): **29 berkas tersentuh** (staged + unstaged + 2 berkas baru), dengan 2 berkas dokumen WO **dihapus** sebagai bagian refactor. Total gabungan ±875 baris ditambah / ±735 dihapus.

Pekerjaan hari ini menyempurnakan modul Work Order dengan lima arah: **(1)** menjadikan WO dapat mewakili berbagai jenis pekerjaan (bukan lagi khusus instalasi), **(2)** menyatukan pratinjau/cetak dokumen WO ke komponen modal bersama, **(3)** menambahkan komponen tautan koordinat (Google Maps), **(4)** memperkenalkan konsep **Penanggung Jawab** tunggal yang tersinkron antara tiket dan WO, serta **(5)** membuat dokumen WO sepenuhnya mengikuti bahasa aktif (i18n).

---

### 🧩 Area 1 — Generalisasi Jenis Work Order (selaras dengan Ticket.type)

Sebelumnya Work Order secara implisit hanya untuk instalasi. Kini WO memiliki field **`type`** yang nilainya disamakan dengan enum `Ticket.type`, sehingga satu mekanisme WO dapat dipakai lintas jenis pekerjaan lapangan.

- [backend/src/models/workOrder.model.js](backend/src/models/workOrder.model.js)
  - **Deskripsi**: Menambah konstanta `WORK_ORDER_TYPES` dan field `type` (default `installation`, tervalidasi enum dengan pesan `INVALID_TYPE`).
- [frontend/src/app/pages/services/workOrder/schema/typeOptions.js](frontend/src/app/pages/services/workOrder/schema/typeOptions.js) `[NEW]`
  - **Deskripsi**: Daftar jenis WO frontend — `survey`, `installation`, `customer`, `partner`, `dismantle`, `backbone`, `backhaul`, `payment`, `other`, `noc` — dengan label yang di-reuse dari namespace `ticket.type.*`.
- [backend/src/controllers/workOrder.controller.js](backend/src/controllers/workOrder.controller.js) · [backend/src/services/workOrder.service.js](backend/src/services/workOrder.service.js) · [backend/src/routes/workOrder.route.js](backend/src/routes/workOrder.route.js)
  - **Deskripsi**: Pembuatan WO menerima `type`; `createTicketFromWorkOrder` menghasilkan tiket yang tipenya mengikuti `wo.type`, dengan `wo.topic` dipetakan ke field tiket yang sesuai (`installation_type` / `survey_type` / `maintenance_topic`) via tabel `TICKET_TOPIC_FIELD_BY_TYPE`.
- [backend/src/config/privilege.json](backend/src/config/privilege.json)
  - **Deskripsi**: Penyesuaian privilege terkait aksi WO.

### 🧩 Area 2 — Penyatuan Pratinjau/Cetak Dokumen WO ke Modal Bersama

Modal pratinjau dan halaman cetak khusus Work Order dihapus, digantikan pemakaian komponen pratinjau bersama `DocumentPreviewModal`, agar konsisten dengan dokumen lain dan mengurangi duplikasi.

- [WorkOrderPreviewModal.jsx](frontend/src/app/pages/services/workOrder/WorkOrderPreviewModal.jsx) **[DELETED]** · [print.jsx](frontend/src/app/pages/services/workOrder/print.jsx) **[DELETED]**
  - **Deskripsi**: Menghapus modal pratinjau & halaman cetak khusus WO; route `work-order/print/:id` juga dihapus dari [workOrderRoute.jsx](frontend/src/app/router/services/workOrderRoute.jsx).
- [detail.jsx](frontend/src/app/pages/services/workOrder/detail.jsx) · [DocumentPreviewModal.jsx](frontend/src/components/shared/DocumentPreviewModal.jsx) · [WorkOrderDocument.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx) · [WorkOrderDetailDrawer.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx)
  - **Deskripsi**: Detail WO beralih memakai `DocumentPreviewModal` bersama (state `showPreview`); modal bersama menerima dokumen WO; komponen dokumen & drawer disesuaikan.
- [create.jsx](frontend/src/app/pages/services/workOrder/create.jsx) · [edit.jsx](frontend/src/app/pages/services/workOrder/edit.jsx) · [index.jsx](frontend/src/app/pages/services/workOrder/index.jsx) · [schema/columns.jsx](frontend/src/app/pages/services/workOrder/schema/columns.jsx)
  - **Deskripsi**: Form buat/edit memuat pilihan `type`; daftar & kolom menampilkan jenis WO.

### 🧩 Area 3 — Komponen Tautan Koordinat

- [frontend/src/components/shared/CoordinateLink.jsx](frontend/src/components/shared/CoordinateLink.jsx) `[NEW]`
  - **Deskripsi**: Menampilkan nilai koordinat (`"lat,lng"`) sebagai tautan yang membuka Google Maps di tab baru; memvalidasi koordinat lebih dulu (`isValidCoordinate`). Dipakai pada detail WO — `<CoordinateLink coordinate={wo.site_coordinate} />`.

### 🧩 Area 4 — Penanggung Jawab: Satu Sumber, Tersinkron Tiket ↔ WO

Sebelumnya WO punya dua kolom orang yang membingungkan: `assigned_team` (admin, muncul sebagai "Engineer" di dokumen) dan `site_pic` (nama/telepon, "PIC Lapangan"). Keduanya disatukan menjadi konsep tunggal **Penanggung Jawab** = `assigned_team` (ref Admin), yang **tersinkron otomatis** dengan penanggung jawab (assignment) tiket terkait.

- [backend/src/controllers/ticket.controller.js](backend/src/controllers/ticket.controller.js)
  - **Deskripsi**: Helper `syncWorkOrderResponsible` — saat penanggung jawab tiket diubah/dihapus di drawer tiket (`changeAssignmentTicket`), `wo.assigned_team` pada WO terkait (`ticket.work_order`) ikut diperbarui. Kegagalan sync tidak menggagalkan perubahan tiket (hanya di-log).
- [backend/src/controllers/workOrder.controller.js](backend/src/controllers/workOrder.controller.js)
  - **Deskripsi**: `createTicketFromWorkOrder` kini mengisi penanggung jawab otomatis — tiket dibuat dengan `assignment = assigned_team WO || pembuat`, dan `wo.assigned_team` diisi bila masih kosong (tanpa perlu langkah "tugaskan" terpisah).
- [backend/src/services/ticket.service.js](backend/src/services/ticket.service.js)
  - **Deskripsi**: Menambahkan `work_order` ke proyeksi field tiket agar `ticket.work_order` terbaca (enabler sinkronisasi).
- [frontend/src/app/pages/services/workOrder/detail.jsx](frontend/src/app/pages/services/workOrder/detail.jsx) · [WorkOrderDetailDrawer.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx)
  - **Deskripsi**: Informasi Umum WO & drawer kini menampilkan satu baris **"Penanggung jawab"** (`assigned_team`); baris duplikat "PIC Lapangan"/"Tim Ditugaskan" dihapus.
- [frontend/src/i18n/locales/*/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: Menambah key `workOrder.responsible` (Penanggung Jawab / Person in Charge).

### 🧩 Area 5 — Dokumen WO Full i18n (ID/EN) & Penyesuaian Label

Dokumen WO dibuat sepenuhnya mengikuti bahasa aktif, dan sejumlah label disesuaikan.

- [frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx)
  - **Deskripsi**:
    - **Tanggal ikut bahasa aktif**: format tanggal memakai locale per-panggilan dari `i18n.language` (nama bulan id/en), menghapus side-effect global `dayjs.locale('id')` yang sebelumnya memaksa Bahasa Indonesia; import `dayjs/locale/en` ditambahkan.
    - **Hilangkan hardcode**: `'No'` → `d('no')`, `'Pcs'` → `d('qtyUnit')`.
    - **Label footer & seksi**: "Engineer/Tanggal Engineer" (kanan) dan "Account Manager/Tanggal Account Manager" (kiri) → **"Penanggung Jawab/Tanggal Penanggung Jawab"**; "Deskripsi Customer" → **"Deskripsi Pelanggan"**.
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) · [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json)
  - **Deskripsi**: Key baru `workOrder.doc.no` & `workOrder.doc.qtyUnit`; `workOrder.doc.customerDesc` (id) → "Deskripsi Pelanggan"; `workOrder.doc.responsible`/`responsibleDate` dipakai ulang untuk kedua kolom footer.

### 🧩 Pendukung Lain

- [backend/src/controllers/customerSO.controller.js](backend/src/controllers/customerSO.controller.js) · [backend/src/services/customerSO.service.js](backend/src/services/customerSO.service.js) · [frontend/src/components/shared/table/rows.jsx](frontend/src/components/shared/table/rows.jsx) · [users/business/profile.jsx](frontend/src/app/pages/users/business/profile.jsx) · [users/partner/profile.jsx](frontend/src/app/pages/users/partner/profile.jsx) · [users/customerPurchaseOrder/create.jsx](frontend/src/app/pages/users/customerPurchaseOrder/create.jsx) · [users/customerSalesOrder/create.jsx](frontend/src/app/pages/users/customerSalesOrder/create.jsx)
  - **Deskripsi**: Penyesuaian menyertai — sel status (`CustomerSOStatusCell`), titik masuk pembuatan WO/dokumen dari profil pelanggan, penyelarasan Customer SO.
- [backend/src/locales/*/translation.json](backend/src/locales/id/translation.json)
  - **Deskripsi**: String terjemahan backend untuk jenis WO & pesan terkait.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Work Order dapat dibuat untuk berbagai jenis pekerjaan (survei, instalasi, dismantle, backbone/backhaul, pembayaran, NOC, dll.); tiket yang dihasilkan otomatis mengikuti jenis WO.
  - Koordinat lokasi pada detail WO dapat diklik untuk membuka Google Maps.
  - **Penanggung Jawab** kini satu konsep yang konsisten di dokumen & informasi umum WO, dan **otomatis tersinkron** ketika diubah dari drawer tiket; saat tiket dibuat dari WO, penanggung jawab langsung terisi.
  - Dokumen WO tampil sesuai bahasa aktif (ID/EN), termasuk nama bulan pada tanggal.
- **Bug Fix / Solusi Masalah**:
  - Menghapus modal pratinjau & halaman cetak khusus WO, digantikan `DocumentPreviewModal` bersama — mengurangi duplikasi.
  - Menghilangkan side-effect global `dayjs.locale('id')` pada dokumen WO yang sebelumnya memaksa tanggal selalu Bahasa Indonesia meski aplikasi berbahasa Inggris.
  - Menyatukan dua kolom orang yang membingungkan (assigned_team vs site_pic) menjadi satu Penanggung Jawab.
- **Menu/Tombol Baru**: Pemilih **jenis Work Order** pada form buat/edit; **tautan koordinat** (Google Maps) pada detail WO.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Work Order digeneralisasi mengikuti enum jenis tiket; jenisnya menentukan tipe tiket & pemetaan topik saat dikonversi. Penanggung Jawab menjadi satu sumber (`assigned_team`) yang disinkronkan dua arah dengan konteks tiket. Dokumen WO memakai pratinjau bersama dan sepenuhnya i18n.
- **Langkah Penggunaan (Tutorial)**:
  1. **Buat WO** → pilih **jenis** (survey/installation/dismantle/…) → isi lokasi/PIC & target tanggal.
  2. **Konversi ke tiket**: tiket terbentuk bertipe sesuai jenis WO; **penanggung jawab terisi otomatis**.
  3. **Ubah penanggung jawab dari drawer tiket** → dokumen WO & informasi umum ikut berubah otomatis.
  4. **Pratinjau dokumen**: buka detail WO → tombol pratinjau membuka modal dokumen bersama.
  5. **Ganti bahasa aplikasi** → label & tanggal pada dokumen WO ikut berganti (ID/EN).
  6. **Buka lokasi**: klik koordinat pada detail WO untuk membuka Google Maps.
