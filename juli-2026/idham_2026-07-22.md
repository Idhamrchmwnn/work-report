# 📝 Daily Work Report - Idham (2026-07-22)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Management — Penyelarasan Status/Lifecycle Dokumen & Penyempurnaan UI Pasca-Audit (Split Modul PO/SO)

## 📅 Laporan Harian - 22 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Belum ada commit dari Idham hari ini. Seluruh pekerjaan masih berstatus **WIP dan belum ter-stage** — **9 berkas berubah** (±820 baris ditambah / ±732 dihapus). Branch aktif: `issue-123`.
>
> **Basis pekerjaan hari ini** adalah commit audit terbaru `e9d480e resolve #123 (audit)` (oleh Dedy, 22 Juli) yang **memecah modul dokumen Customer Management** di backend: dari `customerDocument.controller/service/route` bersama menjadi modul terpisah **customerPO** dan **customerSO** (`customerPO.controller/service/route.js`, `customerSO.controller/service/route.js`), serta merelokasi sebagian halaman frontend ke bawah direktori `users/`. Pekerjaan Idham hari ini menyesuaikan sisi antarmuka (UI) di atas struktur baru tersebut.

Fokus hari ini adalah **menyelaraskan tampilan status dokumen dengan lifecycle-nya yang sebenarnya** dan **menyederhanakan/merapikan komponen UI** dokumen penjualan. Perubahan terbesar ada pada drawer detail Quotation (disederhanakan cukup banyak) dan penambahan sel status Customer SO yang mengikuti alur nyata dokumen.

---

### 🖥️ Frontend — Berkas yang Berubah

- [frontend/src/app/pages/users/quotation/QuotationDetailDrawer.jsx](frontend/src/app/pages/users/quotation/QuotationDetailDrawer.jsx)
  - **Deskripsi**: Refactor besar drawer detail Quotation (372 baris ditambah / 587 dihapus → **net −215 baris**). Menyederhanakan struktur komponen agar lebih ringkas dan konsisten dengan lokasi/modul baru pasca-audit, tanpa mengubah tujuan fungsionalnya.
- [frontend/src/components/shared/table/rows.jsx](frontend/src/components/shared/table/rows.jsx)
  - **Deskripsi**: Menambah sel status dokumen yang mencerminkan **lifecycle sebenarnya**. `CustomerSOStatusCell` memetakan alur `draft → disetujui PIC → terkirim/menunggu tanda tangan pelanggan → selesai (signed/complete)`, dan logika status Quotation diperjelas (customer sudah ttd `complete` atau admin sudah `approve` → status final `won`). Aksi status di-gate oleh privilege (mis. `customerSO.changeStatus`).
- [frontend/src/app/pages/users/business/profile.jsx](frontend/src/app/pages/users/business/profile.jsx) · [frontend/src/app/pages/users/partner/profile.jsx](frontend/src/app/pages/users/partner/profile.jsx)
  - **Deskripsi**: Menata ulang tab dokumen pada profil pelanggan bisnis & mitra (±171 baris masing-masing) — menyambungkan sel status baru, drawer edit/review Quotation, aksi edit/hapus/review dengan pengecekan privilege (`quotation.update`, dll.).
- [frontend/src/app/pages/services/customerSalesOrder/CustomerSOReviewDrawer.jsx](frontend/src/app/pages/services/customerSalesOrder/CustomerSOReviewDrawer.jsx)
  - **Deskripsi**: Penambahan pada drawer peninjauan Customer SO (+66 baris) selaras dengan sel status & lifecycle baru.
- [frontend/src/components/shared/form/Combobox.jsx](frontend/src/components/shared/form/Combobox.jsx)
  - **Deskripsi**: Penyesuaian kecil komponen Combobox.
- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) · [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: Menambah string status/label baru (mis. status Customer SO) di dua bahasa (+7 baris masing-masing).

### 🗂️ Backend — Berkas yang Berubah

- [backend/src/config/privilege.json](backend/src/config/privilege.json)
  - **Deskripsi**: Penyesuaian kecil definisi privilege agar konsisten dengan sel status & aksi dokumen baru pada UI.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**: Status dokumen (khususnya Customer SO) kini ditampilkan sesuai tahap sebenarnya — dari draft, disetujui PIC, menunggu tanda tangan pelanggan, hingga selesai — sehingga admin lebih mudah mengetahui posisi tiap dokumen. Aksi ubah status dibatasi sesuai privilege pengguna.
- **Bug Fix / Solusi Masalah**: Menyederhanakan drawer detail Quotation (mengurangi ±215 baris) mengurangi duplikasi dan menyelaraskan komponen dengan struktur modul baru hasil audit (pemisahan PO/SO), sehingga lebih mudah dirawat dan konsisten antar halaman.
- **Menu/Tombol Baru**: Tidak ada menu baru yang berdiri sendiri; fokus pada penyempurnaan sel status, drawer, dan aksi dokumen yang sudah ada agar sesuai lifecycle dan hak akses.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Setelah audit memecah dokumen Customer Management menjadi modul PO dan SO terpisah di backend, sisi antarmuka disesuaikan agar status setiap dokumen ditampilkan mengikuti lifecycle-nya dan aksi yang muncul menghormati privilege pengguna. Sel status pada tabel dokumen kini menjadi indikator tunggal posisi dokumen.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka profil pelanggan bisnis/mitra → tab dokumen.
  2. Perhatikan kolom status Customer SO: dokumen berpindah otomatis dari `draft` → `disetujui PIC` → `terkirim (menunggu ttd pelanggan)` → `selesai` seiring proses.
  3. Klik dokumen untuk membuka drawer detail/review; tombol aksi (edit/hapus/ubah status) hanya muncul bila pengguna memiliki privilege terkait.
