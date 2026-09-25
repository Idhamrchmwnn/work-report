# 📝 Daily Work Report - Idham (2026-09-25)

---

## 📅 Laporan Harian - 25 September 2026

---

## 🌿 Branch: `issue-339` — Dashboard Operasional (Rencana + Implementasi Satu Hari)

### 📌 Informasi Issue

- **Nomor Issue**: #339 (berdiri sendiri)
- **Judul Issue**: Tambahan Dashboard Operasional
- **Status Branch**: `save #339` (belum `resolve`) — sudah di-push ke `origin/issue-339`.
- **Dokumen pendamping**: `_work-report/dashboardOperasional_plan.md`, ditulis dan diberi tanggal hari ini juga — jadi hari ini mencakup **merancang dari nol sampai mengimplementasikan sebagian besar rencananya**, bukan cuma eksekusi rencana lama.

### 🧭 Latar Belakang & Tujuan

Selama ini performance review operasional (pemasangan, dismantle, gangguan, perubahan paket) dikerjakan manual lewat Excel oleh senior. Dashboard baru ini menjawab pertanyaan **"bagaimana kinerja lapangan periode ini, di mana, dan apa trennya"** — melengkapi (bukan menduplikasi) dashboard yang sudah ada:

| Dashboard | Menjawab | Sifat |
| --- | --- | --- |
| Problem | Apa yang rusak sekarang? | Realtime |
| Sales | Prospek & closing? | Akuisisi |
| **Operasional (baru)** | Kinerja lapangan periode ini, di mana, trennya? | Historis/periode |
| Warehouse | Stok & aset di mana? | Inventaris |

Aturan pemisah yang ditetapkan eksplisit: **kartu realtime tetap di Problem, kartu berbasis periode masuk ke Operasional** — tidak boleh ada kartu yang sama persis di dua dashboard.

### 📅 Rincian Perubahan

#### [241b592b] - save #339 - 25 September 2026, 15:55:11 WIB

**Fase 0 — Fondasi data** (prasyarat, dikerjakan lebih dulu sesuai rencana):
- [`backend/src/models/ticket.model.js`](backend/src/models/ticket.model.js) — 3 field baru: `dismantle_reason` (enum 8 nilai: tunggakan, relokasi, pribadi, tutup usaha, harga, kompetitor, kualitas, lainnya — sebelumnya cuma teks bebas di `description`, tidak bisa diagregasi), `reason_source` (`manual`/`auto` — memisahkan input petugas dari hasil terkaan skrip migrasi, supaya angka hasil tebakan tidak dianggap fakta di dashboard), dan **`closed_at`** (satu field tanggal selesai untuk **semua** jenis tiket — sebelumnya tersebar di 5 field laporan berbeda dengan tipe yang kadang string, sekarang diisi otomatis begitu `complete` jadi `true`). Plus 2 index majemuk baru khusus pola query dashboard (`{deleted,type,created_at}` dan `{deleted,type,closed_at}`).
- [`backend/src/utils/migrate-operational-dashboard.js`](backend/src/utils/migrate-operational-dashboard.js) [NEW, 298 baris] — Migrasi satu-kali, idempoten, mendukung `--dry-run`, 4 langkah: isi `closed_at` dari laporan lama, konversi tanggal string→Date, terka `dismantle_reason` dari kata kunci di `description` (selalu ditandai `reason_source: 'auto'`), dan buat master klaster wilayah/kategori akar masalah (seluruh area lama masuk klaster `unassigned` sebagai titik awal yang harus dirapikan admin).
- [`backend/src/utils/validation-data.js`](backend/src/utils/validation-data.js) — `prepareHistoryChange` menyimpan `from_product` (produk asal), tidak cuma produk tujuan — supaya arah upgrade/downgrade paket bisa dihitung (kartu 5.3/5.4).

**Backend — Modul `operationalDashboard`** (mengikuti pola route→controller→service Mongoose aggregate yang sudah baku di aplikasi ini):
- **9 endpoint**, dipecah per section supaya satu kartu lambat tidak memblokir seluruh halaman: `summary` (KPI utama), `growth` (tren pasang/dismantle/net-add), `area` (matriks klaster), `map` (titik lokasi, dibatasi 5.000 & tanpa PII), `performance` (SLA/MTTR/backlog), `incidents` (Pareto akar masalah, titik gangguan berulang), `retention` (alasan dismantle, churn, perubahan paket), `quality` (gangguan dini pasca-pasang, pengembalian perangkat), `tickets` (drill-down berpaginasi + ekspor XLSX).
- [`backend/src/services/operationalDashboard*.service.js`](backend/src/services/operationalDashboardSummary.service.js) — 7 file service terpisah per section (Summary/Growth/Performance/Incident/Retention/Quality/Ticket), total >3.000 baris agregasi Mongoose.
- **Dua metrik unggulan** (ditandai prioritas tertinggi di rencana, tidak bisa dilihat dari Excel manual):
  - **Gangguan dini pasca-pasang** (`operationalDashboardQuality.service.js`) — persentase pelanggan baru yang lapor gangguan ≤30 hari sejak aktif. Detail statistik yang dijaga: pelanggan yang baru aktif <30 hari lalu belum bisa dinilai "lulus" jendela 30 hari, jadi dipisah jadi `early_incident` (mentah) vs `mature_early_incident` (jendela sudah lewat penuh) — supaya angkanya tidak bias rendah oleh pelanggan yang baru saja dipasang.
  - **Churn setelah keluhan** (`operationalDashboardRetention.service.js`) — persentase pelanggan dismantle yang sebelumnya punya tiket gangguan ≤90 hari. Dipisah lagi jadi `with_complaint` vs `with_complaint_non_arrears` (mengecualikan yang dismantle-nya murni karena tunggakan) supaya sinyal "gangguan memicu churn" tidak tercampur alasan pembayaran.
- [`backend/src/controllers/ticket.controller.js`](backend/src/controllers/ticket.controller.js), `ticket.service.js` — Penutupan tiket (semua jenis) sekarang otomatis mengisi `closed_at`.
- Privilege baru `dashboardOperational` (`read`, `readSensitive` — untuk data biaya/PII di drill-down, `update` — untuk halaman pengaturan master klaster/kategori).
- [`backend/test/integration/operationalDashboard.test.js`](backend/test/integration/operationalDashboard.test.js) [NEW, 552 baris] + [`unit/operationalDashboardMetrics.test.js`](backend/test/unit/operationalDashboardMetrics.test.js) [NEW, 238 baris].

**Frontend** (`frontend/src/app/pages/dashboards/operational/`, mengikuti pola folder dashboard Problem yang sudah ada, tanpa dependency baru — chart tetap `react-apexcharts`, peta tetap Leaflet):
- `index.jsx` + `hooks/useOperationalFilters.js` (filter tersinkron ke URL query, supaya tautan bisa dibagikan) + `hooks/useOperationalSection.js` (fetch per-section paralel).
- **14 komponen kartu**: KPI strip, tren pertumbuhan, peta wilayah, matriks klaster, SLA per jenis, distribusi durasi, backlog (tertaut-klik ke dashboard Problem, bukan duplikat), Pareto akar masalah, tabel titik gangguan berulang, alasan dismantle, perubahan paket, kualitas instalasi, pengembalian perangkat, plus drawer drill-down tiket dengan ekspor.
- [`frontend/src/app/pages/settings/sections/OperationalMaster.jsx`](frontend/src/app/pages/settings/sections/OperationalMaster.jsx) [NEW, 456 baris] — Halaman pengaturan untuk merapikan master klaster wilayah & kategori akar masalah hasil migrasi (yang tadinya lumped ke `unassigned`).
- Formulir tiket Dismantle (`tickets/dismantle/close.jsx`) mendapat field pilihan alasan pembongkaran terstruktur; beberapa formulir tiket lain (`backbone`, `customer`, `partner`, `other`) mendapat penyesuaian kecil terkait skema akar masalah.

- **Deskripsi Perubahan & Fungsi**:
  - Ini adalah hari perancangan **dan** implementasi sekaligus — dokumen rencana (`dashboardOperasional_plan.md`) yang menjabarkan 23 kartu dalam 6 section, 9 ide analitik tambahan, gap data, dan pembagian 4 fase pengerjaan, ditulis hari ini juga, lalu sebagian besar cakupannya (Fase 0 penuh, dan hampir seluruh kartu Fase 1–3) langsung diimplementasikan dalam commit yang sama.
  - **Validasi silang terhadap Excel senior** ditetapkan sebagai langkah wajib sebelum dashboard dipakai serius (rencanakan bandingkan periode 1 Jul–16 Sep 2026) — status commit `save` (bukan `resolve`) konsisten dengan ini: fitur belum dianggap selesai sampai validasi tersebut dilakukan dan didokumentasikan.

#### [ea89edef] - save #339 - 25 September 2026, 17:09:05 WIB

Commit susulan di hari yang sama, memperkeras isolasi kegagalan antar-kartu:

- [`frontend/src/app/pages/dashboards/operational/components/CardErrorBoundary.jsx`](frontend/src/app/pages/dashboards/operational/components/CardErrorBoundary.jsx) [NEW — file yang sedang dibuka di IDE] — Error boundary React (ditulis sebagai class component, satu-satunya pengecualian dari aturan "functional component saja" di `AGENTS.md`, karena React belum punya padanan hook untuk `getDerivedStateFromError`/`componentDidCatch`). Menangkap kegagalan **render** (mis. konfigurasi chart yang ditolak ApexCharts) — bukan kegagalan fetch, yang sudah ditangani state loading/kosong/error terpisah sebelumnya. Tanpa ini, error render pada satu kartu akan ditangkap boundary React Router di level atas dan **seluruh halaman** (14 kartu lainnya) berganti jadi layar "Oops! Something went wrong". Mendukung `resetKey`: begitu filter diganti atau tombol coba-lagi ditekan, kartu yang tadinya gagal diberi kesempatan render ulang, tidak terkunci gagal sampai halaman dimuat ulang penuh.
- [`frontend/src/app/pages/dashboards/operational/components/OperationalCard.jsx`](frontend/src/app/pages/dashboards/operational/components/OperationalCard.jsx) — Kerangka kartu bersama membungkus `children` dengan `CardErrorBoundary` di atas, jadi seluruh 14 kartu otomatis terlindungi tanpa perlu diubah satu-satu.
- `DismantleReasonCard.jsx`, `DurationDistributionCard.jsx`, `PackageChangeCard.jsx`, `RootCauseParetoCard.jsx` — Penyesuaian kecil supaya konsisten dengan pola isolasi kegagalan di atas.

**Deskripsi Perubahan & Fungsi**: Menutup celah pada implementasi awal hari ini — ketentuan desain "satu kartu gagal tidak menjatuhkan halaman" di `dashboardOperasional_plan.md` sebelumnya hanya berlaku untuk kegagalan **mengambil data**; commit ini memperluasnya sampai ke kegagalan **menampilkan** data yang sudah berhasil diambil, skenario yang sebelumnya masih bisa merusak seluruh halaman.

---

## 📖 Informasi Singkat Fitur

- **Kegunaan**: Dashboard Operasional menggantikan proses manual Excel untuk melihat kinerja lapangan per periode & wilayah — mulai dari angka ringkas (KPI strip) sampai analisis akar masalah dan retensi pelanggan, dengan setiap angka bisa di-drill-down ke daftar tiket asli dan diekspor.
- **Dua wawasan baru yang sebelumnya tidak mungkin dilihat manual**: (1) seberapa sering pelanggan baru melapor gangguan dalam 30 hari pertama (indikator kualitas instalasi), dan (2) seberapa besar keluhan yang belum ditangani benar-benar berujung pelanggan berhenti berlangganan (bukan cuma karena telat bayar).
- **Belum bisa dipakai penuh sebelum**: (1) data lama dirapikan lewat halaman Pengaturan → master klaster wilayah (saat ini semua area lama masuk kategori "Belum Dikelompokkan"), dan (2) validasi silang angka dashboard vs Excel senior selesai dilakukan — keduanya bagian dari alasan status masih `save`, bukan `resolve`.
