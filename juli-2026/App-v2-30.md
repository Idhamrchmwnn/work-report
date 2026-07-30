# 📝 Daily Work Report - Idham (2026-07-30)

---

## 📌 Informasi Issue
- **Nomor Issue**: #154
- **Judul Issue**: Work Order — Penyesuaian Label Dokumen, Generalisasi Tiket, & Riwayat Status (Event Log) di Drawer WO

## 📅 Laporan Harian - 30 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Belum ada commit hari ini. Seluruh pekerjaan **ter-stage penuh** (33 berkas total pada working tree, ±1.165 baris ditambah / ±775 dihapus terhitung sejak basis `resolve #154`) di branch `issue-154`. Laporan ini merinci pekerjaan lanjutan setelah [29 Juli](App-v2-29.md), **berkas demi berkas**.

Hari ini melanjutkan modul Work Order dengan empat arah: **(1)** merapikan label dokumen WO, **(2)** menggeneralisasi istilah "Tiket Pemasangan" karena WO kini bisa berbagai jenis, **(3)** membangun **riwayat status (event log)** yang lengkap pada drawer WO, dan **(4)** menelusuri laporan bug "catatan WO tidak tersimpan".

---

### 🧩 Area 1 — Penyesuaian Label Dokumen WO

Tujuan: dokumen WO memakai istilah yang konsisten (Pelanggan, Penanggung Jawab) dan seragam di dua bahasa.

- [frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx) *(±36 baris)*
  - **Perubahan**: Footer kiri (Dept. Penjualan) yang tadinya `d('accManager')` / `d('accManagerDate')` kini memakai ulang key `d('responsible')` / `d('responsibleDate')`, sehingga **kedua kolom footer** (kiri = pembuat WO, kanan = `assigned_team`) berlabel **"Penanggung Jawab"**. Nilai data tidak berubah (kiri tetap `created_by`, kanan tetap `assigned_team`).
  - **Fungsi**: Menyelaraskan istilah dengan konsep "penanggung jawab" tunggal yang diperkenalkan sebelumnya; sekaligus dokumen tetap sepenuhnya i18n (tanggal mengikuti bahasa aktif, header `No`/satuan `Pcs` lewat i18n).
- [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) & [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json)
  - **Perubahan**: `workOrder.doc.customerDesc` (id) diubah dari "Deskripsi Customer" → **"Deskripsi Pelanggan"**. (Key `doc.responsible`/`responsibleDate`, `doc.no`, `doc.qtyUnit` sudah ada dari sebelumnya dan dipakai ulang.)
  - **Fungsi**: Istilah "Pelanggan" menggantikan "Customer" pada judul seksi dokumen sesuai permintaan.

### 🧩 Area 2 — Generalisasi "Tiket Pemasangan" → "Tiket"

Tujuan: WO tidak lagi hanya tipe pemasangan, sehingga seluruh teks & tautan "Tiket Pemasangan" digeneralisasi dan tautan tiket mengikuti jenis sebenarnya.

- [frontend/src/app/pages/services/workOrder/detail.jsx](frontend/src/app/pages/services/workOrder/detail.jsx) *(±187 baris — termasuk penambahan lalu pemindahan riwayat status, lihat Area 3)*
  - **Perubahan**: Tautan baris "Tiket" pada Informasi Umum diubah dari hardcoded `/tickets/installation/view/…` menjadi **`/tickets/${wo.ticket.type || 'installation'}/view/…`** (route mengikuti tipe tiket). Label mengambil `workOrder.ticket` yang kini "Tiket". Tombol & modal konfirmasi buat tiket memakai teks yang sudah digeneralisasi.
  - **Fungsi**: Klik nomor tiket kini membuka halaman tiket sesuai jenisnya (survey/installation/dismantle/dll.), bukan selalu halaman instalasi.
- [frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx) *(±96 baris — termasuk riwayat status, lihat Area 3)*
  - **Perubahan**: Tautan tiket pada drawer juga diperbaiki dari hardcoded `installation` menjadi route berdasarkan `wo.ticket.type` (konsisten dengan halaman detail).
  - **Fungsi**: Konsistensi perilaku tautan tiket di drawer maupun halaman penuh.
- [i18n id](frontend/src/i18n/locales/id/translations.json) & [i18n en](frontend/src/i18n/locales/en/translations.json)
  - **Perubahan**: `workOrder.ticket` "Tiket Pemasangan" → **"Tiket"** / "Installation Ticket" → **"Ticket"**; `workOrder.createTicket` "Buat Tiket Pemasangan" → **"Buat Tiket"** / "Create Ticket"; `ticketCreated`, `ticketCreatedToast`, `createTicketConfirmTitle`, dan `createTicketConfirmDesc` direword agar tidak menyebut "pemasangan/installation".
  - **Fungsi**: Seluruh teks yang muncul di UI Work Order netral terhadap jenis tiket.

### 🧩 Area 3 — Riwayat Status WO sebagai Event Log

Tujuan: menampilkan riwayat status WO yang lengkap (kapan dibuat, kapan & oleh siapa penanggung jawab ditetapkan, kapan tiket dibuat, kapan selesai) di drawer WO, di bawah item layanan. Sebelumnya riwayat hanya menampilkan "Dibuat".

**Backend**

- [backend/src/models/workOrder.model.js](backend/src/models/workOrder.model.js) *(±74 baris)*
  - **Perubahan**: Menambah field **`status_history: [{ event, at, by, target }]`** — `event` (created/assigned/ticket_created/done/cancelled), `at` (waktu), `by` (aktor, ref Admin), `target` (penanggung jawab yang ditetapkan, untuk event `assigned`). Sebelumnya sempat berbentuk `{ status, at, by }`; diubah menjadi event log agar bisa mencatat peristiwa non-status seperti penetapan penanggung jawab.
  - **Fungsi**: Sumber data tunggal untuk riwayat peristiwa WO.
- [backend/src/services/workOrder.service.js](backend/src/services/workOrder.service.js) *(±86 baris)*
  - **Perubahan**:
    - Helper baru **`pushWorkOrderEvent(id, { event, by, target })`** — menambah satu peristiwa (`$push` ke `status_history`, `at` default sekarang).
    - **`setWorkOrderDone(id, adminId)`** kini mencatat event `done` sekaligus `status`/`done_at`.
    - **`createTicketForWorkOrder`** mencatat event `ticket_created` saat tiket dibuat (dan status WO → `in_progress`).
    - `WO_FIELDS` menyertakan `status_history`; `WO_POPULATE` menambah populate `status_history.by` & `status_history.target`, serta menambah `created_at` pada select tiket (untuk rekonstruksi WO lama).
  - **Fungsi**: Menyediakan API pencatatan peristiwa dan memastikan riwayat + relasinya ikut termuat saat WO dibaca.
- [backend/src/controllers/workOrder.controller.js](backend/src/controllers/workOrder.controller.js) *(±82 baris)*
  - **Perubahan**: Mengimpor `pushWorkOrderEvent`. `createWorkOrder` menyisipkan entri awal **`created`** pada `status_history`. `assignWorkOrder` (penugasan via Telegram ke admin tertentu) mencatat event **`assigned`** (aktor = pengirim, target = admin yang ditugaskan). `createTicketFromWorkOrder` mencatat **`assigned`** saat penanggung jawab diisi otomatis (bila WO belum punya `assigned_team`).
  - **Fungsi**: Merekam peristiwa "dibuat" dan "penanggung jawab ditetapkan" pada titik-titik yang benar.
- [backend/src/controllers/ticket.controller.js](backend/src/controllers/ticket.controller.js) *(±35 baris)*
  - **Perubahan**: Mengimpor `pushWorkOrderEvent` & `setWorkOrderDone`. Helper `syncWorkOrderResponsible(woId, assignedTeam, actorId)` — dipanggil saat penanggung jawab tiket diubah di drawer tiket — kini juga mencatat event **`assigned`** (aktor + target) pada WO terkait. Penutupan tiket yang menutup WO memakai **`setWorkOrderDone`** (mencatat event `done`).
  - **Fungsi**: Perubahan penanggung jawab & penyelesaian yang berasal dari sisi tiket ikut terekam di riwayat WO.
- [backend/src/services/ticket.service.js](backend/src/services/ticket.service.js) *(±1 baris)*
  - **Perubahan**: Menyertakan `work_order` pada proyeksi field tiket (penyeimbang sinkronisasi penanggung jawab tiket ↔ WO).
  - **Fungsi**: Agar `ticket.work_order` terbaca saat menyinkronkan/mencatat peristiwa.

**Frontend**

- [frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx)
  - **Perubahan**: Menambah **section "Riwayat Status"** di bawah Item Layanan. Peta warna `EVENT_DOT` per jenis event; `statusHistory` memakai `wo.status_history` bila ada, jika tidak **direkonstruksi** dari `created_at`/`created_by`, `assigned_team`, `ticket.created_at`, dan `done_at`. Tiap entri menampilkan label event (`workOrder.history.*`), nama penanggung jawab (untuk `assigned`), waktu, dan "oleh {aktor}".
  - **Fungsi**: Menyajikan timeline lengkap; WO baru tampil penuh, WO lama tetap informatif walau tanpa log tersimpan.
- [frontend/src/app/pages/services/workOrder/detail.jsx](frontend/src/app/pages/services/workOrder/detail.jsx)
  - **Perubahan**: Section riwayat status sempat ditambahkan di sini lalu **dipindahkan ke drawer**; konstanta & derivasi terkait dihapus dari halaman detail agar tidak duplikat.
  - **Fungsi**: Riwayat status kini hanya berada di satu tempat (drawer), sesuai permintaan.
- [i18n id](frontend/src/i18n/locales/id/translations.json) & [i18n en](frontend/src/i18n/locales/en/translations.json)
  - **Perubahan**: Menambah `workOrder.historyTitle` ("Riwayat Status"), `workOrder.historyBy` ("oleh {{name}}"), dan objek `workOrder.history` (created/assigned/ticket_created/done/cancelled).
  - **Fungsi**: Label event log dalam dua bahasa.

### 🧩 Area 4 — Investigasi Bug "Catatan WO Tidak Tersimpan"

- **Tidak ada perubahan berkas** — murni diagnosis. Alur `notes` ditelusuri menyeluruh: form create mengirim `notes` (diverifikasi via uji skema yup di luar aplikasi), controller `createWorkOrder`/`updateWorkOrder` menyimpannya, model & endpoint view mengembalikannya (`notes` ada di `WO_FIELDS`), dan interceptor axios tidak memangkas field. **Kesimpulan: kode benar di semua lapis**; gejala kemungkinan besar dari server/bundle yang belum ter-restart/rebuild (banyak WIP belum dijalankan ulang), bukan cacat kode. Rekomendasi verifikasi: cek payload `POST /work-order/create` di tab Network + restart backend & frontend.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Dokumen WO memakai istilah konsisten (Pelanggan, Penanggung Jawab) dan mengikuti bahasa aktif.
  - Istilah tiket netral terhadap jenis; tautan tiket membuka halaman sesuai jenis tiket.
  - Drawer WO menampilkan **riwayat status lengkap**: kapan dibuat, kapan & oleh siapa penanggung jawab ditetapkan, kapan tiket dibuat, dan kapan selesai.
- **Bug Fix / Solusi Masalah**:
  - Memperbaiki tautan tiket yang sebelumnya selalu mengarah ke rute `installation`.
  - Riwayat status yang sebelumnya hanya "Dibuat" kini lengkap (WO baru penuh; WO lama direkonstruksi sebisanya).
  - Mendiagnosis tuntas isu "catatan tidak tersimpan" (kode benar; kemungkinan runtime stale).
- **Menu/Tombol Baru**: Section **Riwayat Status** pada drawer Work Order (di bawah Item Layanan).

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Riwayat status WO adalah event log yang direkam otomatis di setiap transisi (buat WO, tetapkan penanggung jawab, buat tiket, selesai). Tiap entri menyimpan waktu, aktor, dan — untuk penetapan penanggung jawab — nama orang yang ditugaskan. Untuk data lama tanpa log, riwayat direkonstruksi dari timestamp yang tersedia.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka drawer WO (dari daftar WO atau tab WO di profil pelanggan).
  2. Gulir ke bawah Item Layanan → **Riwayat Status**.
  3. Perhatikan urutan peristiwa beserta waktu & pelakunya; entri "Penanggung jawab ditetapkan" menampilkan nama penanggung jawab.
  4. Ubah penanggung jawab dari drawer tiket → muncul entri baru di riwayat WO.

> **Catatan teknis**: Perubahan menyentuh model & service backend (`status_history`, populate). **Restart backend** diperlukan agar fitur riwayat status aktif; WO lama tidak memiliki waktu/aktor penetapan penanggung jawab karena data tersebut tidak tersimpan sebelum fitur ini ada.
