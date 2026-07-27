# 📝 Daily Work Report - Idham (2026-07-27)

---

## 📌 Informasi Issue
- **Nomor Issue**: #154
- **Judul Issue**: Work Order — Modul Perintah Kerja (Lifecycle WO, Konversi ke Tiket Pemasangan, & Berita Acara Aktivasi/BAA)

## 📅 Laporan Harian - 27 Juli 2026

> Seluruh pekerjaan hari ini **sudah di-commit** dan working tree dalam keadaan **bersih** (tidak ada perubahan WIP tersisa). Branch aktif: `issue-154`, di atas basis tim terbaru (`resolve #156`, Dedy).

Hari ini menuntaskan **modul Work Order (Perintah Kerja)** sebagai branch fitur ketiga dalam strategi pemisahan dari #123 (setelah `#153` Prospect dan `#123` Customer Management). Modul ini menjembatani dokumen penjualan (SO) dengan eksekusi lapangan: menerbitkan Work Order, memindahkannya melalui siklus status, mengonversinya menjadi tiket pemasangan bagi tim NOC, dan pada akhirnya mencetak **Berita Acara Aktivasi (BAA)** sebagai bukti serah terima saat pemasangan selesai.

---

### 📅 Rincian Commit

#### [`dcb349f`] - resolve #154 (#154 - Work Order: Perintah Kerja, Tiket Pemasangan, & BAA)

- **Ringkasan**: 35 berkas berubah, ±6.013 baris ditambah / ±1.598 dihapus.
- **Deskripsi**: Modul Work Order end-to-end dari backend hingga frontend, terhubung dengan modul Ticket (konversi WO → tiket pemasangan) dan dokumen Customer SO sebagai induknya.

- **Komponen yang Berubah — Backend**:
  - [backend/src/controllers/workOrder.controller.js](backend/src/controllers/workOrder.controller.js) `[NEW]` · [backend/src/services/workOrder.service.js](backend/src/services/workOrder.service.js) `[NEW]` · [backend/src/routes/workOrder.route.js](backend/src/routes/workOrder.route.js) `[NEW]` · [backend/src/models/workOrder.model.js](backend/src/models/workOrder.model.js) `[NEW]`
    - **Deskripsi**: Modul Work Order penuh — `createNewWorkOrder`, `findAllWorkOrdersForTable`, `findWorkOrdersByParent`, `findWorkOrderById`, `updateWorkOrderData`, `deleteWorkOrderById`, serta `createTicketForWorkOrder` (membuat tiket pemasangan dari WO, menyimpan relasi dua arah WO ↔ tiket).
  - [backend/src/controllers/ticket.controller.js](backend/src/controllers/ticket.controller.js) · [backend/src/models/ticket.model.js](backend/src/models/ticket.model.js)
    - **Deskripsi**: Integrasi tiket pemasangan dengan Work Order (relasi `work_order`, penyesuaian alur pembuatan/penutupan tiket).
  - [backend/src/models/prospect.model.js](backend/src/models/prospect.model.js) `[NEW]`
    - **Deskripsi**: Model Prospect disertakan pada branch ini sebagai dependensi induk WO (WO dapat bersumber dari prospek atau customer).
  - [backend/src/app.js](backend/src/app.js) · [backend/src/config/privilege.json](backend/src/config/privilege.json) · [backend/src/routes/files.route.js](backend/src/routes/files.route.js) · [backend/src/utils/telegram.js](backend/src/utils/telegram.js) · terjemahan [en](backend/src/locales/en/translation.json)/[id](backend/src/locales/id/translation.json)
    - **Deskripsi**: Registrasi route & privilege modul Work Order, penyajian berkas dokumen, notifikasi Telegram untuk peristiwa WO, dan string terjemahan backend.

- **Komponen yang Berubah — Frontend**:
  - **Halaman inti WO**: [index.jsx](frontend/src/app/pages/services/workOrder/index.jsx) `[NEW]`, [create.jsx](frontend/src/app/pages/services/workOrder/create.jsx) `[NEW]`, [edit.jsx](frontend/src/app/pages/services/workOrder/edit.jsx) `[NEW]`, [detail.jsx](frontend/src/app/pages/services/workOrder/detail.jsx) `[NEW]`, [print.jsx](frontend/src/app/pages/services/workOrder/print.jsx) `[NEW]`, [schema/columns.jsx](frontend/src/app/pages/services/workOrder/schema/columns.jsx) `[NEW]`, [schema/statusOptions.js](frontend/src/app/pages/services/workOrder/schema/statusOptions.js) `[NEW]`
    - **Deskripsi**: Daftar, buat, edit, detail, dan cetak Work Order. Status mengikuti enum backend: `created → in_progress → done → cancelled`.
  - **Komponen dokumen & pratinjau**: [WorkOrderDocument.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx) `[NEW]`, [WorkOrderPreviewModal.jsx](frontend/src/app/pages/services/workOrder/WorkOrderPreviewModal.jsx) `[NEW]`, [WorkOrderDetailDrawer.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx) `[NEW]`
    - **Deskripsi**: Lembar dokumen WO (dokumen operasional murni — tanpa informasi harga), modal pratinjau cepat, dan drawer detail tanpa berpindah halaman.
  - **Berita Acara Aktivasi (BAA)**: [baa.jsx](frontend/src/app/pages/services/workOrder/baa.jsx) `[NEW]`, [BAADocument.jsx](frontend/src/app/pages/services/workOrder/BAADocument.jsx) `[NEW]`
    - **Deskripsi**: Lembar cetak **Berita Acara Aktivasi** — hanya bermakna untuk WO berstatus `done` (tiket pemasangan sudah ditutup). BAA **dicetak** (bukan diunggah) sebagai bukti serah terima pemasangan.
  - **Navigasi, route & integrasi**: [router/services/workOrderRoute.jsx](frontend/src/app/router/services/workOrderRoute.jsx) `[NEW]`, [navigation/services.js](frontend/src/app/navigation/services.js), [router/protected.jsx](frontend/src/app/router/protected.jsx), [components/shared/table/rows.jsx](frontend/src/components/shared/table/rows.jsx), [users/business/profile.jsx](frontend/src/app/pages/users/business/profile.jsx), [users/partner/profile.jsx](frontend/src/app/pages/users/partner/profile.jsx), [users/customerSalesOrder/CustomerSOReviewDrawer.jsx](frontend/src/app/pages/users/customerSalesOrder/CustomerSOReviewDrawer.jsx), terjemahan [en](frontend/src/i18n/locales/en/translations.json)/[id](frontend/src/i18n/locales/id/translations.json)
    - **Deskripsi**: Menu & route Work Order, sel status WO pada tabel dokumen, serta titik masuk pembuatan WO dari Customer SO/profil pelanggan.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Admin/operasional dapat menerbitkan **Work Order** dari Customer SO, mengelola siklus statusnya (`created → in_progress → done`), serta membatalkannya bila perlu.
  - Work Order dapat **dikonversi menjadi tiket pemasangan** untuk tim NOC, dengan relasi dua arah yang menjaga konsistensi antara WO dan tiketnya.
  - Setelah pemasangan selesai (`done`), admin dapat **mencetak Berita Acara Aktivasi (BAA)** sebagai bukti serah terima.
  - Lembar dokumen WO tersedia untuk pratinjau/cetak tanpa memuat informasi harga (dokumen operasional).
- **Bug Fix / Solusi Masalah**: Menyelesaikan pemisahan branch fitur (#154 Work Order) menjadi commit bersih di atas basis tim terbaru, sehingga modul berdiri utuh dan siap di-review terpisah dari Prospect (#153) dan Customer Management (#123).
- **Menu/Tombol Baru**: Menu navigasi & route **Work Order**; tombol cetak dokumen WO & **BAA**; aksi konversi WO → tiket pemasangan; sel status WO pada tabel dokumen pelanggan.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Work Order adalah jembatan antara dokumen penjualan (SO) dan eksekusi lapangan. Sebuah WO dibuat dari SO, dijalankan melalui siklus status, dan bila melibatkan pemasangan, dikonversi menjadi tiket untuk tim NOC. Saat pekerjaan tuntas, sistem menghasilkan Berita Acara Aktivasi yang dicetak sebagai dokumen serah terima. Lembar WO sengaja tidak menampilkan harga karena bersifat operasional.
- **Langkah Penggunaan (Tutorial)**:
  1. **Buat Work Order**: dari Customer SO (atau profil pelanggan), buka aksi buat WO; isi data lokasi/PIC (sebagian ter-prefill dari induk SO) dan target tanggal.
  2. **Jalankan siklus**: ubah status WO dari `created` → `in_progress` seiring pengerjaan; konversikan ke **tiket pemasangan** untuk tim NOC bila diperlukan.
  3. **Selesaikan**: setelah tiket pemasangan ditutup, tandai WO `done`.
  4. **Cetak BAA**: buka halaman BAA WO tersebut lalu cetak Berita Acara Aktivasi sebagai bukti serah terima.
  5. **Pratinjau/cetak dokumen WO**: gunakan modal pratinjau atau halaman cetak untuk lembar Work Order operasional (tanpa harga).
