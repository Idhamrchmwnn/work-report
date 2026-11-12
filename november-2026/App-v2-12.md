# 📝 Daily Work Report - Idham (2026-11-12)

---

## 📅 Laporan Harian - 12 November 2026

---

## 🌿 Branch: `issue-543` — Lampiran Pengajuan Izin Tidak Bisa Dibuka Pemiliknya

### 📌 Informasi Issue

- **Nomor Issue**: #543
- **Judul Issue**: Lampiran pengajuan izin baru dibalas 404 saat dibuka dari halaman profil
- **Status Branch**: `Belum di-merge`. Satu commit `resolve #543` di atas `master` terbaru (setelah `resolve #536`), sudah ada di `origin/issue-543`.

### 📅 Rincian Commit

#### [74f4a3e2] - resolve #543 - 12 November 2026, 14:17

- **Ringkasan**:
  - Sejak pengajuan izin punya dokumen induk (`PermissionRequest`), lampiran hanya disimpan di dokumen induk. Dokumen harian (`Permission`) tidak lagi menyalinnya.
  - Penyaji berkas `GET /file/admin-permission/:name` masih mencari pemilik lampiran hanya di dokumen harian. Akibatnya setiap lampiran pengajuan baru, baik dari web maupun Telegram Mini App, dibalas 404 walau berkasnya ada di MinIO.
  - Kini pemilik dicari di dokumen induk lebih dulu. Dokumen harian tetap diperiksa untuk lampiran data lama yang tersimpan sebelum dokumen induk ada.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/services/attendance.service.js` (`findPermissionByAttachmentName`)
    - Mencari `PermissionRequest.findOne({ attachment })` lebih dulu, lalu jatuh ke `Permission` bila tidak ketemu. Proyeksi tetap hanya `admin`.
    - Komentar fungsi dikoreksi: lampiran karyawan lain butuh `attendance.readSensitive`, bukan `attendance.read`.

  - `backend/src/models/attendanceAbsenceRequest.model.js`
    - Indeks `attachment` (sparse), karena field ini kini dipakai penyaji lampiran untuk mencari pemilik berkas.

  - `backend/test/integration/attendanceFileAccess.test.js`
    - Skenario baru: pemilik bisa membuka lampiran yang tersimpan di dokumen induk pengajuan (200).

---

## 🌿 Branch: `issue-545` — Kartu Kinerja Marketing di Dashboard Penjualan

### 📌 Informasi Issue

- **Nomor Issue**: #545
- **Judul Issue**: Dashboard Penjualan: kinerja marketing berdasarkan tiket instalasi
- **Status Branch**: `Belum di-merge`. Masih work-in-progress (`save #545`), sudah ada di `origin/issue-545`.

### 📅 Rincian Commit

#### [039aa6c3] - save #545 - 12 November 2026, 17:07

- **Ringkasan**:
  - Kartu baru **Kinerja Marketing** di Dashboard Penjualan. Kartu ini menampilkan jumlah instalasi selesai per marketing, survei, dan instalasi yang masih berjalan.
  - Atribusinya memakai `Ticket.marketing` dengan kriteria yang **sama dengan dasar `marketing_fee` Payroll**:
    - tiket instalasi `complete` dan tidak `canceled`;
    - tanggal diambil dari `installation_report.date`.

    Bedanya, tiket yang di-soft-delete dikecualikan.
  - Ada empat pilihan periode: bulan ini, bulan lalu, tahun ini, dan semua.
    - Periode berjalan dibandingkan dengan rentang **sama panjang** di periode sebelumnya, misalnya 1–12 Nov dengan 1–12 Okt. Membandingkan bulan berjalan dengan bulan penuh selalu tampak turun di awal bulan.
  - Hak akses:
    - Endpoint memakai `ticketInstallation.list`.
    - Jumlah survei hanya dihitung bila pemanggil juga punya `ticketSurvey.list`. Tanpa hak itu nilainya `null` dan tampil "–", bukan 0.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/controllers/marketingPerformance.controller.js` [NEW]
    - `marketingPerformance`: memvalidasi `period` (di luar daftar dijawab 400 `salesDashboard.invalidPeriod`) dan memeriksa `hasPrivilege(req, 'ticketSurvey.list')`.
    - Respons memuat `period { key, start, end }`, `summary`, dan `rows`.

  - `backend/src/services/marketingPerformance.service.js` [NEW], hanya membaca. Empat agregasi berjalan paralel:
    - instalasi selesai per marketing dalam periode, termasuk kelompok tanpa marketing;
    - survei bermarketing yang dibuat dalam periode, tanpa yang batal;
    - instalasi yang masih berjalan saat ini, tidak bergantung periode;
    - total instalasi selesai pada rentang pembanding.

    Rincian lainnya:
    - `installation_report.date` bertipe campuran warisan V1 (string/Date), jadi dinormalkan dengan `$convert`. Tanggal yang tidak terbaca tidak masuk periode mana pun, tetapi tetap terhitung pada periode `all`.
    - Agregasi melewati plugin `mongoose-delete`, jadi `deleted` disaring eksplisit.
    - Nama marketing diambil lewat `findMultipleAdminWithDeleted`. Admin yang sudah nonaktif atau terhapus tetap tampil (kreditnya nyata) dengan penanda `active: false`.
    - Baris diurutkan berdasarkan instalasi selesai, lalu survei, tiket berjalan, dan nama.
    - `summary` berisi `completed`, `previous_completed`, `surveys`, `in_progress`, `unassigned` (instalasi tanpa marketing), dan `marketers`.
    - Kegagalan dicatat `logger.error` lalu dilempar dengan `salesDashboard.marketingFailed`.

  - `backend/src/utils/marketingPerformancePeriod.js` [NEW]
    - `resolveMarketingPerformanceRange(period, now)` menghitung batas bulan/tahun menurut kalender **Asia/Jakarta** secara eksplisit, karena container backend berjalan UTC. `end` bersifat eksklusif.
    - Pembanding bulan ini dibatasi akhir bulan lalu bila bulan lalu lebih pendek.
    - Bulan lalu adalah bulan penuh, menyeberang tahun dengan benar.
    - Periode `all` tanpa batas dan tanpa pembanding.

  - `backend/src/routes/ticket.route.js`
    - Route `GET /ticket/dashboard/marketing-performance` dengan `checkPrivilege('ticketInstallation.list')`, beserta Swagger lengkap.

  - `backend/src/config/privilegeDictionary.json`
    - Endpoint baru tercatat di `apiEndpoints` key `ticketInstallation.list`.

  - `backend/src/locales/{id,en}/translation.json`
    - Key baru `salesDashboard.marketingFailed` dan `salesDashboard.invalidPeriod`.

  **Frontend**

  - `frontend/src/app/pages/dashboards/sales/components/MarketingPerformanceCard.jsx` [NEW]
    - Header menampilkan jumlah marketing dan pemilih periode.
    - Tiga ringkasan:
      - instalasi selesai, dengan chip tren naik/turun/tetap terhadap periode pembanding;
      - survei;
      - tiket berjalan.
    - Peringatan bila ada instalasi selesai tanpa marketing.
    - Papan peringkat:
      - warna peringkat 1–3 sama dengan kartu Performa Sales;
      - bar relatif terhadap marketing teratas dan porsi persen dari total;
      - penanda marketing nonaktif.

      Marketing yang hanya punya survei atau tiket berjalan tetap tampil tanpa nomor peringkat.
    - Ada keadaan skeleton, galat dengan tombol muat ulang, daftar kosong, dan tanpa hak akses. Tautan menuju `/tickets/installation`.
    - Tooltip memakai `data-tooltip`, dan layout responsif (1 kolom di mobile, 3 kolom di `lg`).

  - `frontend/src/app/pages/dashboards/sales/hooks/useMarketingPerformance.js` [NEW]
    - Data kartu dimuat terpisah dari `useSalesStats`, jadi mengganti periode hanya memuat ulang kartu ini.
    - Penyegaran otomatis setiap 30 detik.
    - Respons yang tiba terlambat dibuang lewat nomor permintaan, dan penyegaran yang gagal mempertahankan data lama.
    - `isStale` mencegah data periode lama tampil sebagai milik periode baru.

  - `frontend/src/app/pages/dashboards/sales/utils/marketingPerformance.js` [NEW]
    - Logika murni: `computeTrend` (persentase `null` bila pembanding 0), `barPercent`, `sharePercent`, dan `formatPeriodRange` (hari terakhir inklusif, zona WIB, tahun tampil di kedua sisi bila berbeda).

  - `frontend/src/app/pages/dashboards/sales/index.jsx`
    - Kartu dipasang sebagai bagian ke-5 dashboard, digerbangi `useHasPrivilege('ticketInstallation.list')`.
    - Tombol muat ulang dashboard ikut memuat ulang kartu ini.

  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - Key `salesDashboard.marketing.*` (judul, ringkasan, tren, periode, peringatan, keadaan kosong, dll.), masing-masing id dan en.

  **Test**

  - `backend/test/integration/marketingPerformance.test.js` [NEW], 5 skenario:
    - kriteria hitung sama dengan Payroll;
    - periode `all` ikut menghitung tanggal laporan rusak;
    - marketing nonaktif/terhapus tetap dihitung;
    - periode tak dikenal ditolak 400;
    - survei disembunyikan tanpa `ticketSurvey.list`.
  - `backend/test/unit/marketingPerformancePeriod.test.js` [NEW], 6 skenario: batas kalender WIB, pembanding sama panjang, bulan pendek, pergantian tahun, tahun ini, dan periode `all`.
  - `frontend/.../MarketingPerformanceCard.test.jsx` [NEW], 7 skenario: ringkasan dan peringkat, survei tanpa hak tampil "–", periode `all` tanpa tren, data basi disembunyikan, galat, daftar kosong, dan tanpa hak akses.
  - `frontend/.../utils/marketingPerformance.test.js` [NEW], 8 skenario untuk tren, bar, porsi, dan label rentang periode.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #543 | `74f4a3e2` resolve #543 | Lampiran pengajuan izin baru bisa dibuka lagi oleh pemiliknya (pencarian pemilik di dokumen induk) |
| #545 | `039aa6c3` save #545 | Kartu Kinerja Marketing di Dashboard Penjualan: endpoint agregasi, periode WIB dengan pembanding sama panjang, kartu frontend, dan test (WIP) |

> Catatan merge: `resolve #529` (Partner API, dikerjakan 6 November) sudah di-merge ke `master` pada 7 November.
