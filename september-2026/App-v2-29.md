# 📝 Daily Work Report - Idham (2026-09-29)

---

## 📅 Laporan Harian - 29 September 2026

---

## 🌿 Branch: `issue-339` — Dashboard Operasional: Dibongkar & Dibangun Ulang dari Nol

### 📌 Informasi Issue

- **Nomor Issue**: #339 (berdiri sendiri)
- **Judul Issue**: Tambahan Dashboard Operasional
- **Status Branch**: `save #339` — HEAD saat ini di `3efdb6fc`, sudah di-push.
- **Dokumen pendamping baru**: `.agent/operasionaldashboard_plan.md` — rencana yang sama sekali baru, menggantikan rencana lama (`_work-report/dashboardOperasional_plan.md`, 25 September).

### 🔄 Keputusan Besar Hari Ini: Rebuild dari Nol

Implementasi Dashboard Operasional dari 25 September (23 kartu, 9 endpoint, perubahan skema `Ticket`, skrip migrasi) **dirombak total hari ini** dan diganti pendekatan yang jauh lebih ramping:

- **Branch `issue-339` di-reset** ke titik sebelum implementasi lama (`a1ac3ece`), bersih dari seluruh kode 25 September.
- **Implementasi lama tidak dihapus** — diamankan di branch terpisah `issue-339-old-dashboard` (berisi commit `60040e6e`, hasil kerja pagi ini yang sempat melanjutkan pendekatan lama sebelum keputusan rebuild diambil). Dokumen rencana baru menandai eksplisit: *"Jangan merge/cherry-pick dari sana tanpa persetujuan user."*
- **Cakupan baru dipersempit drastis**, mengikuti persis 4 sheet yang benar-benar ada di Excel performance review senior — bukan 9 ide tambahan yang sebelumnya diusulkan:

| Bagian | Padanan Excel |
| --- | --- |
| A. Ringkasan Umum | *Executive Summary* |
| B. Akar Gangguan | *Root Cause Analysis* |
| C. Perubahan Paket | *Package Change Analysis* |
| D. Distribusi Area (peta + tabel) | *Area Distribution Analysis* |

- **Sengaja ditiadakan** dibanding rencana lama: **tidak ada perubahan skema `Ticket`** (tidak ada `closed_at`/`dismantle_reason` baru), **tidak ada skrip migrasi/backfill**, tidak ada perubahan form tiket, tidak ada privilege baru (memakai 8 privilege `ticket*.list` yang sudah ada), tidak ada kartu SLA-per-prioritas/growth/retensi-churn/kualitas-instalasi/gangguan-berulang/device-return, tidak menyentuh `telegram-apps`. Semua angka dihitung **read-only** dari data yang sudah ada apa adanya.

### 📅 Rincian Perubahan

#### [3efdb6fc] - save #339 - 29 September 2026, 16:06 WIB (rev.4 dari plan, revisi ke-4 dalam sehari)

- **Backend** (mengikuti pola `dashboard.route.js → controller → service` yang sudah baku, dicontoh dari `getProblemDashboardStats`):
  - [`backend/src/constants/operationalDashboard.constant.js`](backend/src/constants/operationalDashboard.constant.js) [NEW] — Konstanta 8 tipe tiket, regex deteksi kata kunci upgrade/downgrade paket, default target KPI (nilai dari Excel), batas titik peta (5.000).
  - [`backend/src/services/operationalDashboard.service.js`](backend/src/services/operationalDashboard.service.js) [NEW] — `getOperationalSummary`, `getRootCauseStats`, `getPackageChangeStats`, `getAreaDistribution`, `getAreaMapPoints`, `getKpiTargets`.
  - **Endpoint dipindah mengikuti pola dashboard lain** (temuan review internal hari ini): dari rencana awal `/operational-dashboard/*` (file route sendiri) ke pola domain `/ticket/dashboard/operational-*` di `ticket.route.js`, berdampingan dengan `/ticket/dashboard/problem-stats` yang sudah ada — respons ringkasan disamakan formatnya (`{success, message, data}`) dengan dashboard problem/hotspot/broadband/whatsapp.
  - **Target KPI bisa diubah lewat Settings** (bukan halaman/model baru) — memakai mekanisme `options` yang sudah ada (`name: 'operational_dashboard_settings'`), 5 nilai target (tingkat selesai minimal, tingkat proses maksimal, tingkat batal maksimal, rata-rata & median jam penyelesaian maksimal), dengan validasi rentang ditambahkan ke cabang baru di `updateSettings` yang sudah ada.
  - [`backend/test/integration/operationalDashboard.test.js`](backend/test/integration/operationalDashboard.test.js) [NEW] — 14 test, hijau (suite penuh tetap 3 merah bawaan `master`, tidak terkait perubahan ini).
- **Temuan implementasi yang didokumentasikan** (§9 rencana):
  - **Tiket batal juga ber-`complete: true`** — status dihitung dengan urutan prioritas batal → selesai → proses, dan durasi hanya dihitung untuk yang selesai-tidak-batal.
  - **Deteksi perubahan paket** memakai kata kunci upgrade/downgrade **plus** topik bawaan "Perubahan Layanan"; tiket tanpa kata arah ditandai `unspecified`; tiket pembayaran dikecualikan.
  - **Area** untuk tiket tanpa koordinat langsung (pelanggan/pelepasan) memakai koordinat pelanggan; survey (yang belum punya `customer`) mengambil area dari tiket instalasi lanjutannya via `survey_id` — satu-satunya index baru yang ditambahkan (`{survey_id: 1}`), bukan index tanggal seperti dugaan awal karena mode default dashboard tidak memfilter tanggal sama sekali.
  - **Optimasi query**: `$lookup` sub-pipeline ternyata ~3× lebih lambat dari `$lookup` localField/foreignField biasa + ringkas belakangan — dipilih yang kedua. Agregasi area/peta di data dev (7 ribu tiket) ~250ms, jadi cache Redis belum dipasang.

- **Frontend** (`frontend/src/app/pages/dashboards/operational/`, tanpa dependency baru — chart tetap ApexCharts):
  - `index.jsx`, `hooks/useOperationalStats.js` (pola `use<Nama>Stats` yang sama dengan dashboard lain, auto-refresh 30 detik + badge live), komponen `OperationalKpiStrip`, `ActivityCompositionCard`, `RootCauseCard`, `PackageChangeCard`, `AreaDistributionTable`, `OperationalDashboardSkeleton`.

---

## ⚠️ Catatan Risiko: Ada Pekerjaan Hari Ini yang Masih Tersimpan di `git stash`, Belum Di-commit

Ditemukan lewat `git stash list`: satu stash (`stash@{0}`, "issue-339: perbaikan UI/UX dashboard operasional", pukul 18:08) berisi **penyempurnaan UI/UX yang cukup besar** di atas commit `3efdb6fc` — mencakup file baru yang belum pernah ter-commit sama sekali:

- [`TicketListDrawer.jsx`](frontend/src/app/pages/dashboards/operational/components/TicketListDrawer.jsx), [`SectionNav.jsx`](frontend/src/app/pages/dashboards/operational/components/SectionNav.jsx), [`ExportMenu.jsx`](frontend/src/app/pages/dashboards/operational/components/ExportMenu.jsx), [`exportReport.js`](frontend/src/app/pages/dashboards/operational/utils/exportReport.js) — tabel detail dipindah ke drawer, navigasi antar-section yang menempel, dan **ekspor laporan ke Excel (5 sheet) & PDF (A4 lanskap)** langsung dari data yang sudah dimuat di halaman.
- Perbaikan pada file yang sudah ada: diagram Pareto untuk Akar Gangguan (kolom + garis kumulatif, ringkasan "N kategori = 80%"), pembanding otomatis dengan periode sebelumnya di kartu KPI, tabel area dengan pencarian/urut/heatmap per tipe tiket, format angka yang konsisten mengikuti bahasa aktif.

**Ini pekerjaan nyata yang berisiko hilang** — stash tidak ikut ter-push ke remote, dan bisa hilang tanpa sengaja (mis. `git stash clear`, checkout branch lain lalu lupa, atau konflik saat `stash pop` di kemudian hari). Disarankan segera di-`stash pop` dan di-commit sebagai lanjutan `save #339` sebelum sesi kerja berikutnya, supaya tidak perlu dikerjakan ulang.

---

## 📖 Informasi Singkat Fitur

- **Kenapa dibongkar ulang**: implementasi 25 September mencakup jauh lebih banyak (23 kartu, perubahan skema, migrasi data) daripada yang benar-benar dibutuhkan untuk mereplikasi 4 sheet Excel yang jadi acuan awal — risiko dan kompleksitasnya tidak sepadan dengan kebutuhan sebenarnya. Versi baru sengaja dibatasi ketat ke 4 bagian yang benar-benar ada padanannya di Excel, tanpa menyentuh skema data sama sekali.
- **Kegunaan**: Dashboard ini menggantikan proses cek manual Excel — Ringkasan Umum (KPI inti + target yang bisa diatur lewat Pengaturan), Akar Gangguan (topik tiket mana yang paling sering, dengan analisis Pareto), Perubahan Paket (rasio upgrade vs downgrade), dan Distribusi Area (peta + tabel per wilayah) — semuanya dihitung langsung dari data tiket yang sudah ada, tidak perlu proses input tambahan apa pun dari tim lapangan.
