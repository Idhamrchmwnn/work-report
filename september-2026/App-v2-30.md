# 📝 Daily Work Report - Idham (2026-09-30)

---

## 📅 Laporan Harian - 30 September 2026

---

## 🌿 Branch: `issue-339` — Dashboard Operasional Selesai (`resolve`)

### 📌 Informasi Issue

- **Nomor Issue**: #339
- **Judul Issue**: Tambahan Dashboard Operasional
- **Status Branch**: **`resolve #339`** — akhirnya ditandai selesai hari ini, setelah dua hari sebelumnya (25 & 29 September) melalui rebuild total dan penyempurnaan UI/UX.
- **✅ Risiko yang di-flag kemarin sudah ditindaklanjuti**: berisi penyempurnaan UI/UX (drawer detail, navigasi antar-section, ekspor Excel/PDF)

### 📅 Rincian Perubahan

#### ⚠️ Koreksi Struktur Commit

- **[191c1fac] `save #339`, 12:01 WIB** — commit sementara yang melanjutkan penyempurnaan UI/UX. Digantikan (bukan dilanjutkan) oleh commit berikutnya.
- **[98b63013] `resolve #339`, 14:40 WIB** — dibangun dari titik lain (parent `110cd48e`, bukan `191c1fac`). Karena branch ini di-rebase di atas commit-commit lain sejak kemarin, diff commit ini terhadap parent langsungnya otomatis mencakup **seluruh** hasil kerja kumulatif fitur (bukan cuma delta hari ini) — itulah kenapa jumlah filenya lebih banyak dari `save`.

**Rincian lengkap [191c1fac] `save #339` (29 file, 1.847 baris ditambah, 472 dihapus):**

- **Backend** (perbaikan kecil menyusul, tidak ada perubahan struktural): [`operationalDashboard.constant.js`](backend/src/constants/operationalDashboard.constant.js) [+6], [`operationalDashboard.controller.js`](backend/src/controllers/operationalDashboard.controller.js) [+41/-15], [`operationalDashboard.service.js`](backend/src/services/operationalDashboard.service.js) [+62/-9], [`ticket.route.js`](backend/src/routes/ticket.route.js) [+30/-7], [`operationalDashboard.test.js`](backend/test/integration/operationalDashboard.test.js) [+57/-3], `locales/{en,id}/translation.json` [1 baris masing-masing].
- **Frontend — file baru** (hasil `stash pop` kemarin): [`ExportMenu.jsx`](frontend/src/app/pages/dashboards/operational/components/ExportMenu.jsx) [94 baris] — menu ekspor Excel (5 sheet) & PDF (A4 lanskap); [`SectionNav.jsx`](frontend/src/app/pages/dashboards/operational/components/SectionNav.jsx) [38 baris] — navigasi antar-section yang menempel (sticky); [`TicketListDrawer.jsx`](frontend/src/app/pages/dashboards/operational/components/TicketListDrawer.jsx) [111 baris] — tabel detail dipindah dari inline ke drawer; [`exportReport.js`](frontend/src/app/pages/dashboards/operational/utils/exportReport.js) [302 baris] — logika penyusun file ekspor; [`OperationalSettings.jsx`](frontend/src/app/pages/settings/sections/OperationalSettings.jsx) [111 baris] — halaman target KPI versi baru.
- **Frontend — file dihapus**: [`TicketTables.jsx`](frontend/src/app/pages/dashboards/operational/components/TicketTables.jsx) [-47] (digantikan `TicketListDrawer.jsx`), [`OperationalDashboard.jsx`](frontend/src/app/pages/settings/sections/OperationalDashboard.jsx) [-118] (digantikan `OperationalSettings.jsx`), route mandiri di [`settingsRoute.jsx`](frontend/src/app/router/settings/settingsRoute.jsx) [-11] dan entri menu di [`navigation/settings.js`](frontend/src/app/navigation/settings.js) [-10] — halaman pengaturan dipindah **masuk ke tab Application** di Settings, tidak lagi punya route/menu sendiri.
- **Frontend — file dimodifikasi besar**: [`AreaDistributionTable.jsx`](frontend/src/app/pages/dashboards/operational/components/AreaDistributionTable.jsx) [+201/-79] — penambahan heatmap; [`OperationalKpiStrip.jsx`](frontend/src/app/pages/dashboards/operational/components/OperationalKpiStrip.jsx) [+212/-44] — pembanding otomatis dengan periode sebelumnya; [`RootCauseCard.jsx`](frontend/src/app/pages/dashboards/operational/components/RootCauseCard.jsx) [+127/-37] — diagram Pareto (kolom + garis kumulatif, ringkasan "N kategori = 80%"); [`index.jsx`](frontend/src/app/pages/dashboards/operational/index.jsx) [+144/-39] — merangkai `SectionNav` & drawer baru; [`Application.jsx`](frontend/src/app/pages/settings/sections/Application.jsx) [+63/-4] — menambahkan tab target KPI; sisanya (`ActivityCompositionCard.jsx`, `PackageChangeCard.jsx`, `constants.js`, `useOperationalStats.js`, `columns.jsx`, `format.js`, `i18n/locales/*`) perbaikan kecil mendukung fitur di atas.

**Rincian lengkap [98b63013] `resolve #339` (37 file, 5.697 baris ditambah, 17 dihapus) — menggantikan rincian sebelumnya:**

- **Backend — 7 endpoint final**, pola domain `/ticket/dashboard/operational-*` di [`ticket.route.js`](backend/src/routes/ticket.route.js) [476 baris]: `operational-summary`, `operational-root-cause`, `operational-package-change` (+ `.../list` datatable), `operational-area`, `operational-area/map`, `operational-ticket/list`. Digerbang OR dari 8 privilege tiket yang sudah ada — tidak ada privilege baru.
  - [`operationalDashboard.controller.js`](backend/src/controllers/operationalDashboard.controller.js) [176 baris, NEW].
  - [`operationalDashboard.service.js`](backend/src/services/operationalDashboard.service.js) [934 baris, NEW] — 18 fungsi: agregasi utama per bagian (`getOperationalSummary`, `getRootCauseStats`, `getPackageChangeStats` + `getPackageChangeList`, `getAreaDistribution`, `getAreaMapPoints`, `getOperationalTicketList`), plus helper murni yang bisa diuji terpisah (`computeDurationStats`, `buildTicketMatch`, `classifyPackageChange`, `parseCoordinate`, `normalizeKpiTargets`/`findInvalidKpiTargets`/`pickKpiTargets`).
  - [`operationalDashboard.constant.js`](backend/src/constants/operationalDashboard.constant.js) [106 baris, NEW] — 8 tipe tiket, regex kata kunci upgrade/downgrade, default target KPI.
  - [`ticket.model.js`](backend/src/models/ticket.model.js) [+3] — index baru `{survey_id: 1}`, satu-satunya perubahan skema, dipakai `$lookup` menurunkan area tiket survey dari tiket instalasi lanjutannya.
  - [`settings.controller.js`](backend/src/controllers/settings.controller.js) [+26], [`option.service.js`](backend/src/services/option.service.js) [+40] — cabang `operational_dashboard_settings` pada `updateSettings` (validasi 5 target KPI).
  - `locales/{en,id}/translation.json` [+8 masing-masing].
  - [`operationalDashboard.test.js`](backend/test/integration/operationalDashboard.test.js) [514 baris, NEW] — mencakup seluruh 7 endpoint final.
- **Frontend — menu & routing**: [`navigation/dashboards.js`](frontend/src/app/navigation/dashboards.js) [+10] — item menu **"Operational"** di sidebar Dashboards, tampil untuk siapa pun dengan salah satu dari 8 privilege tiket; [`router/protected.jsx`](frontend/src/app/router/protected.jsx) [+11] — route `/dashboards/operational` lazy-loaded dengan gerbang privilege OR yang sama.
- **Frontend — 4 bagian dashboard, seluruhnya file baru**:
  - Bagian A (Ringkasan Umum): [`OperationalKpiStrip.jsx`](frontend/src/app/pages/dashboards/operational/components/OperationalKpiStrip.jsx) [369 baris] — 6 kartu KPI + pembanding periode sebelumnya; [`ActivityCompositionCard.jsx`](frontend/src/app/pages/dashboards/operational/components/ActivityCompositionCard.jsx) [140 baris] — donut chart + tabel komposisi tiket per tipe.
  - Bagian B (Akar Gangguan): [`RootCauseCard.jsx`](frontend/src/app/pages/dashboards/operational/components/RootCauseCard.jsx) [268 baris] — Pareto (kolom + garis kumulatif).
  - Bagian C (Perubahan Paket): [`PackageChangeCard.jsx`](frontend/src/app/pages/dashboards/operational/components/PackageChangeCard.jsx) [156 baris] — rasio upgrade vs downgrade.
  - Bagian D (Distribusi Area): [`AreaDistributionTable.jsx`](frontend/src/app/pages/dashboards/operational/components/AreaDistributionTable.jsx) [242 baris] — tabel heatmap (`color-mix` CSS, kompatibel warna tema hex maupun oklch di Tailwind v4); [`AreaMapCard.jsx`](frontend/src/app/pages/dashboards/operational/components/AreaMapCard.jsx) [154 baris] + [`hooks/useAreaMapPoints.js`](frontend/src/app/pages/dashboards/operational/hooks/useAreaMapPoints.js) [55 baris] — peta titik tiket (Leaflet, pola sama dengan `FiberMap.jsx`), maksimal 5.000 titik tanpa PII.
  - Kerangka bersama: [`SectionCard.jsx`](frontend/src/app/pages/dashboards/operational/components/SectionCard.jsx) [87 baris] — kartu generik + state kosong; [`SectionNav.jsx`](frontend/src/app/pages/dashboards/operational/components/SectionNav.jsx) [38 baris] — navigasi antar-section sticky; [`OperationalDashboardSkeleton.jsx`](frontend/src/app/pages/dashboards/operational/components/OperationalDashboardSkeleton.jsx) [45 baris] — kerangka loading awal; [`OperationalFilterBar.jsx`](frontend/src/app/pages/dashboards/operational/components/OperationalFilterBar.jsx) [81 baris] — filter preset tanggal (bulan ini/30 hari/90 hari/tahun ini/semua); [`TicketListDrawer.jsx`](frontend/src/app/pages/dashboards/operational/components/TicketListDrawer.jsx) [111 baris] — drill-down daftar tiket per kartu; [`ExportMenu.jsx`](frontend/src/app/pages/dashboards/operational/components/ExportMenu.jsx) [94 baris] + [`utils/exportReport.js`](frontend/src/app/pages/dashboards/operational/utils/exportReport.js) [302 baris] — ekspor Excel (5 sheet) & PDF (A4 lanskap).
  - Pendukung: [`hooks/useOperationalStats.js`](frontend/src/app/pages/dashboards/operational/hooks/useOperationalStats.js) [125 baris] — fetch per-section + auto-refresh 30 detik; [`index.jsx`](frontend/src/app/pages/dashboards/operational/index.jsx) [291 baris] — merangkai seluruh bagian; [`schema/columns.jsx`](frontend/src/app/pages/dashboards/operational/schema/columns.jsx) [135 baris] — kolom tabel drill-down (termasuk `TicketOwnerCell`/`PackageChangeDirectionCell` baru di `rows.jsx`); [`constants.js`](frontend/src/app/pages/dashboards/operational/constants.js) [103 baris], [`utils/format.js`](frontend/src/app/pages/dashboards/operational/utils/format.js) [29 baris].
  - [`components/shared/table/rows.jsx`](frontend/src/components/shared/table/rows.jsx) [+29] — 2 cell wrapper baru: `TicketOwnerCell` (tautan pelanggan/mitra pemilik tiket) dan `PackageChangeDirectionCell` (badge upgrade/downgrade/tidak disebutkan) — mengikuti aturan wajib AGENTS.md §2.B.17 (`Badge` mentah tidak boleh diimpor langsung di `columns.jsx`).
  - Halaman pengaturan target KPI: [`settings/sections/OperationalSettings.jsx`](frontend/src/app/pages/settings/sections/OperationalSettings.jsx) [111 baris] (terintegrasi ke tab **Application**, [`Application.jsx`](frontend/src/app/pages/settings/sections/Application.jsx) [+77/-13]) + [`settings/schema/operationalDashboardSchema.js`](frontend/src/app/pages/settings/schema/operationalDashboardSchema.js) [27 baris] — validasi Yup 5 target KPI (persentase 0–100, jam 0–8760), sinkron dengan aturan di `operationalDashboard.constant.js` backend.
  - `i18n/locales/{en,id}/translations.json` [+158 masing-masing].
- **Deskripsi Perubahan & Fungsi**: Ini adalah commit yang benar-benar menyelesaikan Dashboard Operasional — 4 bagian penuh (Ringkasan Umum dengan KPI + komposisi, Akar Gangguan dengan Pareto, Perubahan Paket, Distribusi Area dengan tabel heatmap + peta), lengkap dengan filter tanggal, drill-down per drawer, dan ekspor Excel/PDF; 7 endpoint mengikuti pola domain dashboard lain, satu index baru yang benar-benar dibutuhkan, tanpa privilege baru — persis sesuai batas cakupan yang diputuskan sejak rebuild 29 September.

---

## 🌿 Branch: `issue-348` — Alasan Wajib Saat Menonaktifkan (Pasif) Pelanggan

### 📌 Informasi Issue

- **Nomor Issue**: #348 (berdiri sendiri)
- **Judul Issue**: Alasan Ubah Pelanggan Pasif
- **Status Branch**: `resolve 348`, sudah di-push — perubahan kecil dan bersih (6 file, 301 baris).

### 🧭 Latar Belakang

**Bug nyata ditemukan**: saat admin menandai pelanggan sebagai "Pasif" lewat halaman edit, field `pasif` (yang seharusnya berisi ALASAN pelanggan dinonaktifkan, ditampilkan di kolom "Alasan Pasif" pada halaman daftar Pelanggan Pasif) sebelumnya **diisi otomatis dengan literal string `'active'`** — nilai penanda status yang sama sekali bukan alasan. Kolom "Alasan Pasif" di seluruh sistem berisi teks "active" untuk setiap pelanggan yang di-pasifkan lewat halaman ini, alih-alih alasan sebenarnya.

### 📅 Rincian Perubahan

#### [af25c020] - resolve 348 - 30 September 2026, 17:57:52 WIB

- **Komponen yang Berubah**:
  - [`frontend/src/app/pages/users/customer/PasifReasonModal.jsx`](frontend/src/app/pages/users/customer/PasifReasonModal.jsx) [NEW, 238 baris] — Modal baru yang **mewajibkan** admin mengetik alasan (textarea, maksimal 500 karakter, disamakan dengan batas validasi backend) sebelum pelanggan bisa ditandai pasif. Dialog dibangun manual (bukan memakai `ConfirmModal`/`GlobalConfirmModal` yang sudah ada) — komentar di kode menjelaskan alasannya: komponen konfirmasi generik itu menggabung `description` lewat `lodash.merge` setiap render, yang membuat `<Textarea>` di dalamnya ikut di-mount ulang setiap kali diketik dan kehilangan fokus — bug spesifik yang sudah ditemukan dan dihindari lebih dulu.
  - [`frontend/src/app/pages/users/customer/edit.jsx`](frontend/src/app/pages/users/customer/edit.jsx) — `handlePasifToggle` dipecah dua jalur: **mengaktifkan status pasif** membuka modal alasan di atas (wajib diisi); **membatalkan status pasif** tetap lewat konfirmasi sederhana yang sudah ada, alasannya otomatis dikosongkan. Ditambah kotak peringatan (alert) yang menampilkan alasan pasif tersimpan langsung di halaman edit pelanggan yang sedang pasif — sebelumnya alasan itu tidak ditampilkan di halaman ini sama sekali, hanya di daftar Pelanggan Pasif.
  - [`backend/src/controllers/customer.controller.js`](backend/src/controllers/customer.controller.js) — `setPasifCustomer` menambah validasi: `pasif` harus berupa string (bukan tipe lain), dan maksimal 500 karakter — sebelumnya field ini diterima apa adanya tanpa validasi bentuk/panjang.
  - [`backend/src/routes/customer.route.js`](backend/src/routes/customer.route.js) — Dokumentasi Swagger diperbarui menjelaskan `pasif` sebagai alasan (bukan flag), dengan `maxLength: 500` dan penjelasan string kosong = batalkan status pasif.
- **Deskripsi Perubahan & Fungsi**: Menutup bug di mana kolom "Alasan Pasif" tidak pernah benar-benar berisi alasan — sekarang admin wajib menjelaskan kenapa seorang pelanggan dinonaktifkan, tersimpan dan terlihat baik di halaman edit pelanggan maupun daftar Pelanggan Pasif.

---

## 📢 Ringkasan Dampak

| Branch | Dampak Utama |
| --- | --- |
| `issue-339` | Dashboard Operasional (4 bagian: Ringkasan, Akar Gangguan, Perubahan Paket, Distribusi Area+Peta) resmi selesai, menggantikan proses manual Excel. |
| `issue-348` | Kolom "Alasan Pasif" pelanggan sekarang benar-benar berisi alasan yang diketik admin, bukan lagi teks placeholder yang tidak bermakna. |
