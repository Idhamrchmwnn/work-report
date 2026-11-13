# 📝 Daily Work Report - Idham (2026-11-13)

---

## 📅 Laporan Harian - 13 November 2026

---

## 🌿 Branch: `issue-545` — Kinerja Marketing & Corong Pemasaran di Dashboard Penjualan

### 📌 Informasi Issue

- **Nomor Issue**: #545
- **Judul Issue**: Dashboard Penjualan: kinerja marketing berdasarkan tiket instalasi dan konversi calon pelanggan
- **Status Branch**: `Belum di-merge`. Sudah di-squash menjadi satu commit `resolve #545` di atas `master` terbaru (setelah `resolve #543`) dan sudah ada di `origin/issue-545`.

### 📅 Rincian Commit

#### [33b785dc] - resolve #545 - 13 November 2026, 17:33

> Kartu Kinerja Marketing (jumlah tiket per marketing, periode WIB, pembanding sama panjang) sudah dilaporkan kemarin (`save #545`). Laporan ini hanya memuat tambahan hari ini.

- **Ringkasan**:
  - Kartu Kinerja Marketing kini juga menampilkan **corong pemasaran**: pendaftaran → survei → pemasangan. Isinya konversi, median lama sampai terpasang, calon tertahan, posisi calon per tahap, dan jalur masuk calon.
  - Ada **dua cara hitung yang sengaja berbeda** dalam satu kartu:
    - **Jumlah** (survei, instalasi) dihitung per **tiket** pada periode tiket, dengan kriteria yang sama dengan `marketing_fee` Payroll.
    - **Ukuran corong** dihitung per **calon** yang **mulai** pada periode itu (kohort), lalu dilacak sampai tahap terakhirnya. Dengan cara ini konversi tidak pernah melebihi 100%.
  - Jumlah per calon dari corong **tidak pernah ditampilkan sebagai angka**. Sebelumnya muncul dua angka "terpasang" yang berbeda untuk orang yang sama (mis. 412 tiket vs 408 calon), karena satu survei bisa punya lebih dari satu pemasangan sah, seperti layanan kedua atau pasang ulang. Dari corong hanya persen, jumlah hari, calon tertahan, dan posisi calon yang tampil.
  - Kolom tabel dikelompokkan menjadi "Calon yang mulai di periode ini" dan "Tiket di periode ini", supaya keduanya tidak dibaca sebagai pembilang dan pembagi satu sama lain.
  - Hak akses corong: selain `ticketInstallation.list`, pemanggil wajib memegang `registration.list` **dan** `ticketSurvey.list`.
    - Backend menjawab 403 bila salah satu hak itu tidak ada.
    - Di frontend, kolom corong tampil "–" dengan keterangan hak akses, sedangkan jumlah tiket tetap tampil.
  - Bila corong gagal dimuat, kartu menampilkan peringatan dan tombol muat ulang. Kolomnya tidak diam-diam berubah menjadi "–".

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/utils/marketingFunnel.js` [NEW] berisi logika corong yang murni, tanpa akses database.
    - Satu perjalanan = satu calon, dengan tiga titik awal:
      - `registration`: dokumen pendaftaran;
      - `walk_in`: survei tanpa pendaftaran;
      - `direct`: pemasangan tanpa pendaftaran dan tanpa survei. Hampir semuanya data 2022–2023, jadi dihitung terpisah dan tidak ikut pembagi konversi.
    - `resolveJourneyStatus`: tahap diturunkan dari **tiket**, bukan dari `register_status`. Status pendaftaran terbukti tidak selalu sejalan dengan tiketnya, jadi hanya dipakai untuk alasan berhenti `uncoverage`/`cancelled`. Ada sembilan status, dikelompokkan menjadi berproses, terpasang, dan berhenti.
    - `resolveJourneyMarketing`: pemilik calon = marketing yang tercatat **paling akhir** (pemasangan selesai → pemasangan → survei → pendaftaran), sejalan dengan Payroll.
    - `toDocFormNumber`: mengubah `document_id` pendaftaran (teks) ke bentuk `Ticket.doc_form` (angka).
    - Pemasangan ditelusuri lewat `doc_form` **dan** `survey_id`, karena sebagian pemasangan lama hanya tertaut ke surveinya. Pemasangan tetap dihitung sekali per calon.
    - `buildMarketingFunnel` menghasilkan:
      - `summary`: calon masuk, sudah survei, terpasang, berproses, berhenti, tertahan, tanpa marketing, dan median hari;
      - `statuses`: jumlah per tahap;
      - `paths`: rincian per jalur masuk;
      - `rows`: angka per marketing, termasuk konversi dan median hari.
    - `STUCK_DAYS` (14 hari): calon yang masih berproses lebih lama dari ini dihitung tertahan.

  - `backend/src/services/marketingFunnel.service.js` [NEW]
    - Hanya memuat data minimal, lalu menyerahkan perhitungan ke `buildMarketingFunnel`. Data yang dimuat:
      - pendaftaran dalam kohort;
      - survei milik pendaftaran itu dan survei tanpa pendaftaran;
      - pemasangan yang tertaut;
      - pemasangan langsung.
    - Tiket dan dokumen terhapus disaring.
    - Nama marketing diambil lewat `findMultipleAdminWithDeleted`. Marketing yang sudah dihapus tetap tampil.
    - Baris diurutkan berdasarkan instalasi, lalu konversi, calon masuk, pasang langsung, dan nama.
    - Respons membawa `stuck_days`, supaya teks UI tidak menyalin angkanya sendiri.
    - Respons hanya berisi angka agregat dan nama marketing, tanpa data pribadi calon pelanggan.
    - Kegagalan dicatat `logger.error`, lalu dilempar dengan pesan `salesDashboard.funnelFailed`.

  - `backend/src/controllers/marketingPerformance.controller.js`
    - Pembacaan periode dipisah menjadi helper `readPeriod`, dipakai kedua handler.
    - Handler baru `marketingFunnel`:
      - periode tidak valid dijawab 400;
      - memeriksa `registration.list` dan `ticketSurvey.list` lewat `hasPrivilege` (403 bila kurang);
      - menjawab `{ period, stuck_days, summary, statuses, paths, rows }`.

  - `backend/src/services/marketingPerformance.service.js`
    - Setiap baris kini membawa `admin.id` (ObjectId marketing) sebagai kunci tetap. Tanpa kunci ini, beberapa marketing yang sudah dihapus permanen (`admin_id` dan nama kosong) bisa tergabung menjadi satu baris saat digabung dengan data corong.

  - `backend/src/routes/ticket.route.js`
    - Route `GET /ticket/dashboard/marketing-funnel` dengan `checkPrivilege('ticketInstallation.list')`, beserta Swagger lengkap (aturan kohort, hak akses tambahan, skema respons, 400/403).

  - `backend/src/config/privilegeDictionary.json` dan `frontend/src/constants/privilegeDescriptions.{id,en}.json`
    - `apiEndpoints` `ticketInstallation.list` menambah endpoint corong.
    - Deskripsi `registration.list` dan `ticketSurvey.list` menjelaskan perannya di kartu Kinerja Marketing. Teks kamus dan tooltip dijaga tetap sama persis.

  - `backend/src/locales/{id,en}/translation.json`
    - Key baru `salesDashboard.funnelFailed` (id dan en).

  - `AGENTS.md`
    - Aturan §2.A butir 40 tentang Kinerja Marketing, yang mencakup:
      - dua cara hitung;
      - jebakan data `doc_form` angka vs `document_id` teks;
      - `register_status` yang tidak sejalan dengan tiket;
      - atribusi marketing paling akhir dan konsekuensinya;
      - `MIN_SAMPLE`, `STUCK_DAYS`, dan kunci baris `admin.id`;
      - aturan hak akses.

  **Frontend**

  - `frontend/src/app/pages/dashboards/sales/components/MarketingPerformanceCard.jsx` (ditulis ulang)
    - Ringkasan:
      - instalasi selesai, dengan tren;
      - survei;
      - konversi calon;
      - median hari sampai terpasang.
    - Bagian **posisi calon**: status per tahap, jumlah calon tertahan, dan calon tanpa marketing.
    - Bagian **jalur masuk calon**: pendaftaran, survei langsung, dan pasang langsung.
    - Tabel per marketing:
      - kolom dikelompokkan menjadi calon vs tiket;
      - pilihan urutan: instalasi, konversi, atau kecepatan;
      - warna konversi dibanding rata-rata tim;
      - penanda sampel kecil, nonaktif, dan tertahan.
    - Catatan kohort untuk periode berjalan.
    - Keadaan corong gagal dan corong tanpa hak akses masing-masing punya tampilan sendiri.

  - `frontend/src/app/pages/dashboards/sales/components/marketingShared.jsx` [NEW]
    - Potongan UI bersama (`StatTile`, `StateMessage`, `PeriodToolbar`, `TILE_CLASS`), agar tampilan seragam dengan kartu lain di dashboard.

  - `frontend/src/app/pages/dashboards/sales/utils/marketingFunnel.js` [NEW]
    - Logika murni:
      - `percentOf`;
      - `conversionTone`: dibanding rata-rata tim; sampel di bawah `MIN_SAMPLE` (5) tidak diberi warna;
      - `mergeMarketingRows`: gabung baris tiket dan corong per `admin.id`;
      - `sortMarketingRows`: sampel kecil diletakkan di bawah pada urutan konversi/kecepatan;
      - `rankByCompleted`: peringkat tetap menurut instalasi;
      - `formatDays`.

  - `frontend/src/app/pages/dashboards/sales/hooks/usePeriodData.js` [NEW]
    - Pola muat data per periode diangkat dari `useMarketingPerformance`: nomor permintaan untuk membuang respons basi, data lama dipertahankan saat penyegaran gagal, dan `isStale`.

  - `frontend/src/app/pages/dashboards/sales/hooks/useMarketingFunnel.js` [NEW] dan `useMarketingPerformance.js`
    - Keduanya kini memakai `usePeriodData`. Corong disegarkan setiap 5 menit karena berubah lambat dan perhitungannya lebih berat; kinerja tetap 30 detik.

  - `frontend/src/app/pages/dashboards/sales/index.jsx`
    - `canReadFunnel` = `ticketInstallation.list` + `registration.list` + `ticketSurvey.list`, sama dengan pemeriksaan backend.
    - Satu periode dipakai bersama untuk jumlah tiket dan corong.
    - Tombol muat ulang dan coba lagi ikut memuat ulang corong.

  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - Key `salesDashboard.marketing.*` untuk corong: status, jalur, kelompok kolom, urutan, keterangan kohort, sampel kecil, tertahan, gagal memuat, tanpa hak akses, dan lainnya.

  **Test**

  - `backend/test/unit/marketingFunnel.test.js` [NEW], 8 skenario:
    - konversi `document_id`;
    - tahap diturunkan dari tiket, bukan `register_status`;
    - urutan tahap dan alasan berhenti;
    - atribusi marketing paling akhir;
    - median;
    - ringkasan, status, dan jalur;
    - angka per marketing;
    - pemasangan tidak dihitung dua kali lewat `doc_form` dan `survey_id`.
  - `backend/test/integration/marketingFunnel.test.js` [NEW], 3 skenario:
    - pelacakan pendaftaran → survei → pemasangan dari database;
    - 403 tanpa hak pendaftaran atau survei;
    - 400 untuk periode tidak dikenal, dan diterima dengan hak lengkap.
  - `backend/test/integration/marketingPerformance.test.js`: admin yang terhapus permanen tetap punya kunci baris sendiri.
  - `frontend/.../utils/marketingFunnel.test.js` [NEW], 10 skenario: persen, warna konversi, penggabungan baris (termasuk marketing terhapus permanen), urutan, peringkat, dan format hari.
  - `frontend/.../MarketingPerformanceCard.test.jsx`, kini 11 skenario. Skenario baru:
    - jumlah dari tiket, konversi dan lama dari corong;
    - pengelompokan kolom;
    - peringatan calon tanpa marketing;
    - corong gagal dimuat;
    - jumlah per calon tidak pernah ditampilkan;
    - tanpa hak corong.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #545 | `33b785dc` resolve #545 | Corong pemasaran di kartu Kinerja Marketing (kohort calon, konversi, lama terpasang, tertahan, jalur masuk), hak akses berlapis, kunci baris `admin.id`, dan squash menjadi satu commit siap PR |

> Catatan merge: `resolve #543` (dikerjakan 12 November) sudah di-merge ke `master` pada 12 November, 18.45.
