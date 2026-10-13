# 📝 Daily Work Report - Idham (2026-10-13)

---

## 📅 Laporan Harian - 13 Oktober 2026

---

## 🌿 Branch: `issue-407` — Toggle Pelaporan di Tabel Pelanggan & Penggantian Istilah "Lapor Pajak"

### 📌 Informasi Issue

- **Nomor Issue**: #407
- **Judul Issue**: Toggle pelaporan langsung di tabel Pelanggan (pelanggan dengan satu layanan) dan penggantian istilah "Lapor Pajak" menjadi "Pelaporan"
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-407`)

### 📅 Rincian Commit

#### [945fe0e1] - resolve #407 - 13 Oktober 2026, 18:12

- **Ringkasan**: Lanjutan fitur #394.
  - Pelanggan reguler yang punya **tepat satu layanan** kini bisa ditandai terlaporkan langsung dari tabel Pelanggan lewat toggle, tanpa membuka form edit.
  - Pelanggan dengan lebih dari satu layanan (atau tanpa layanan) tetap menampilkan label ringkasan dan diatur lewat form edit.
  - Istilah UI "Lapor Pajak / Siap Dilaporkan" diganti menjadi **"Pelaporan / Terlaporkan"** di seluruh teks.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/services/taxReport.service.js`
    - Fungsi baru `setSingleCustomerTaxReport({ customerObjectId, value, adminId })`. Fungsi ini menetapkan nilai `tax_reported` untuk satu-satunya layanan pelanggan, dengan jejak audit (`tax_reported_by/at`).
    - Bila jumlah layanan pelanggan bukan tepat satu (termasuk layanan terhapus), fungsi ini menolak dengan `notSingle`.
    - Nilai dikirim eksplisit, bukan dibalik, jadi mengirim nilai yang sama dua kali tidak mengubah apa pun (idempoten). Perubahan dicatat dengan `logger.info`.
    - Ringkasan per pemilik (`countByOwner`) kini mengembalikan `{ status, total }`, bukan hanya status, supaya tabel tahu jumlah layanan tiap pelanggan.
  - `backend/src/services/customer.service.js`
    - Datatable Pelanggan menambahkan field `tax_report_service_count` (jumlah layanan) di samping `tax_report_status`. Field ini dipakai frontend untuk memilih antara toggle dan label.
  - `backend/src/services/business.service.js`
    - Disesuaikan dengan bentuk ringkasan baru (`summary[...]?.status`).
  - `backend/src/controllers/customer.controller.js`
    - Handler baru `toggleSingleCustomerTaxReport`:
      - Membalas 400 bila `customer_id` bukan string atau `tax_reported` bukan boolean. Body seperti `{ $ne: 'x' }` juga ditolak.
      - Membalas 404 bila pelanggan tidak ditemukan atau milik mitra.
      - Membalas 422 bila jumlah layanannya bukan satu.
      - Bila berhasil, membalas `{ tax_reported, message }`.
  - `backend/src/routes/customer.route.js`
    - Route baru `PATCH /api/v1/customer/tax-report/single` dengan `protectedAdmin` dan `checkPrivilege('customer.changeSensitive')`, lengkap dengan dokumentasi Swagger.
  - `backend/src/config/privilegeDictionary.json`
    - Endpoint `/customer/tax-report/single` ditambahkan ke `customer.changeSensitive`. Deskripsi kedua privilege lapor pajak memakai istilah baru "terlaporkan".
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `customer.taxReportNotSingle` ("Pelanggan ini punya lebih dari satu layanan (atau tidak punya layanan). Atur pelaporan lewat halaman ubah pelanggan.").
    - Pesan `taxReportConflict`, `taxReportUpdated`, `taxReportBatchUpdated`, dan `taxReportOwnedByReseller` memakai istilah "pelaporan".
  - `backend/test/integration/customerTaxReported.test.js`
    - Kelompok test baru "Toggle lapor pajak di tabel Pelanggan (tepat satu layanan)":
      - Daftar membawa jumlah layanan.
      - Nilai ditetapkan eksplisit dengan jejak audit, dan bersifat idempoten.
      - 422 untuk pelanggan dengan dua layanan atau tanpa layanan; layanan lain tidak tersentuh.
      - 400 untuk body tidak sah, dan 404 untuk pelanggan mitra.

  **Frontend**

  - `frontend/src/components/shared/table/rows.jsx`
    - Sel baru `TaxReportToggleCell`: switch yang memanggil `PATCH /customer/tax-report/single` dan hanya aktif untuk pemegang `customer.changeSensitive`.
    - Bila berhasil, sel diperbarui tanpa memuat ulang tabel. Bila backend menjawab 422 (jumlah layanan sudah berubah), tabel dimuat ulang supaya kolom kembali menjadi label.
  - `frontend/src/app/pages/users/customer/schema/columns.jsx`
    - Kolom Pelaporan menampilkan `TaxReportToggleCell` bila `tax_report_service_count === 1`, dan label `TaxReportStatusCell` untuk selebihnya.
  - `frontend/src/app/pages/users/components/TaxReportServicesCard.jsx`, `frontend/src/app/pages/users/components/TaxReportBatchModal.jsx`
    - Hanya komentar: istilah "Lapor Pajak" diganti "Pelaporan".
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Penggantian istilah:
      - Judul kolom/kartu: "Lapor Pajak" menjadi "Pelaporan".
      - "Siap Dilaporkan" menjadi "Terlaporkan", dan "Semua dilaporkan" menjadi "Semua terlaporkan".
      - Tombol: "Laporkan semua layanan" menjadi "Tandai semua terlaporkan", dan "Batalkan laporan" menjadi "Batalkan pelaporan".
      - Teks info kartu dan aksi massal ikut disesuaikan.
  - `frontend/src/constants/privilegeDescriptions.id.json`, `frontend/src/constants/privilegeDescriptions.en.json`
    - Tooltip `customer.changeSensitive` dan `business.changeSensitive` memakai istilah baru.

  **Dokumentasi**

  - `AGENTS.md`
    - Pola Bab 9 "Visibilitas di Partner API" diperbarui: label UI kini "Pelaporan / Terlaporkan", dan ditambah catatan toggle di tabel Pelanggan khusus pelanggan dengan tepat satu layanan (`/customer/tax-report/single`, 422 bila jumlah layanan sudah bukan satu).

---

## 🌿 Branch: `issue-405` — Ubah Status Aktif/Nonaktif Lewat Menu Aksi dengan Konfirmasi

### 📌 Informasi Issue

- **Nomor Issue**: #405
- **Judul Issue**: Ganti switch status di kolom tabel dengan aksi "Aktifkan / Nonaktifkan" di menu aksi, dengan konfirmasi
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-405`)

### 📅 Rincian Commit

#### [1cbf6376] - resolve #405 - 13 Oktober 2026, 16:51

- **Ringkasan**: Sebelumnya status aktif/nonaktif di banyak tabel bisa diubah langsung lewat switch di kolom Status, satu klik tanpa konfirmasi, sehingga rawan tersentuh tidak sengaja.
  - Switch itu kini diganti badge status yang hanya bisa dibaca.
  - Perubahan status dipindah ke aksi **Aktifkan / Nonaktifkan** di menu aksi baris (`RowActions`), dan selalu melewati modal konfirmasi.

- **Komponen yang Berubah & Penjelasannya**:

  **Komponen Bersama**

  - `frontend/src/components/shared/table/RowActions.jsx`
    - Aksi baru `status` di menu aksi baris. Label dan ikon mengikuti status saat ini: "Aktifkan" (hijau, ikon centang) atau "Nonaktifkan" (kuning, ikon larangan). Label bisa diganti per halaman lewat `activateLabel` / `deactivateLabel`.
    - Selalu membuka `ConfirmModal` dengan pesan "Status **[nama]** akan diubah menjadi **[status]**. Lanjutkan?".
    - Mendukung dua jalur:
      - `url` + `body`: PATCH bawaan. Status akhir diambil dari respons server, karena endpoint membalik status yang ada di database, bukan yang dilihat admin. Sel diperbarui lalu tabel dimuat ulang.
      - `onToggle(row)`: untuk halaman yang mengurus request-nya sendiri.
    - Ada penjaga klik ganda. Endpoint change-status membalik nilai di server, jadi klik kedua akan mengembalikan status ke semula.
    - Aksi hanya tampil bila user memegang privilege `role` yang diberikan.
  - `frontend/src/components/shared/table/rows.jsx`
    - Sel switch khusus `RecurringActiveSwitchCell` (Transaksi Berulang) dan `CoaStatusSwitchCell` (Akun Perkiraan) dihapus karena digantikan aksi `status`.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Key baru `alert.confirmChangeStatus` untuk pesan konfirmasi.

  **Halaman Pengguna**

  - `frontend/src/app/pages/users/customer/schema/columns.jsx`
    - Kolom Status menjadi `StatusBadgeCell`. Aksi status memakai `/customer/change-status/` (`customer.changeStatus`).
  - `frontend/src/app/pages/users/business/schema/columns.jsx`
    - Pola yang sama dengan `/business/change-status/` (`business.changeStatus`).
  - `frontend/src/app/pages/users/partner/schema/columns.jsx`
    - Pola yang sama dengan `/partner/change-status/` (`partner.changeStatus`).
  - `frontend/src/app/pages/users/customerPartner/schema/columns.jsx`
    - Pola yang sama dengan `/customer-partner/change-status/` (`customerPartner.changeStatus`).
  - `frontend/src/app/pages/users/admin/schema/columns.jsx`
    - Pola yang sama dengan `/admin/change-status/` (`admin.changeStatus`).
  - `frontend/src/app/pages/users/employee/schema/columns.jsx`
    - Pola yang sama dengan `/employee/change-status/` (`employee.changeStatus`).

  **Halaman Layanan & Jaringan**

  - `frontend/src/app/pages/services/broadband/schema/columns.jsx`
    - Pola yang sama dengan `/broadband/change-status/` (`broadband.changeStatus`).
  - `frontend/src/app/pages/services/hotspotUser/schema/columns.jsx`
    - Pola yang sama dengan `/hotspot-user/toggle-status/:username` (`userHotspot.update`).
  - `frontend/src/app/pages/services/vendorManagement/schema/columns.jsx`
    - Pola yang sama dengan `/vendor/change-status` (`vendor.update`).
  - `frontend/src/app/pages/network/radiusNas/schema/columns.jsx`
    - Pola yang sama dengan `/radius-nas/change-status` (`radiusNas.changeStatus`). Sel yang diperbarui adalah kolom `enabled`.
  - `frontend/src/app/pages/network/radiusNonCustomer/schema/columns.jsx`
    - Pola yang sama dengan `/radius-non-customer/change-status/` (`radiusNonCustomer.changeStatus`). Sisanya hanya perapian format oleh Prettier.

  **Halaman Keuangan**

  - `frontend/src/app/pages/finance/coa/index.jsx`
    - Handler baru `handleToggleStatus` yang memanggil `PATCH /finance/coa/:coa_id/status`, menampilkan toast, lalu memuat ulang tabel. Handler ini diteruskan ke kolom sebagai `onToggleStatus`.
  - `frontend/src/app/pages/finance/coa/schema/columns.jsx`
    - Switch diganti badge. Aksi status memakai `onToggle` (`financeCoa.changeStatus`), dan disembunyikan untuk akun sistem (`is_system`) yang memang ditolak backend.
  - `frontend/src/app/pages/finance/accounts/schema/columns.jsx`
    - Pola yang sama (`financeAccount.changeStatus`). Akun sistem tidak diberi aksi status.
  - `frontend/src/app/pages/finance/accounts/AccountStatusSwitchCell.jsx`
    - Dihapus karena digantikan aksi status.
  - `frontend/src/app/pages/finance/recurring/schema/columns.jsx`
    - Switch diganti aksi status dengan label khusus "Lanjutkan" / "Jeda" (`financeRecurring.changeStatus`). Jadwal yang sudah berakhir (`ended`) tidak diberi aksi karena tidak bisa diaktifkan lagi.

  **Halaman Aktivitas**

  - `frontend/src/app/pages/activities/attendanceDevice/schema/deviceColumns.jsx`
    - Kolom status perangkat absensi kini badge baca-saja (`StatusBadgeCell`). Dokumen lama tanpa field `active` dianggap aktif.
  - `frontend/src/app/pages/activities/attendanceDevice/schema/DeviceStatusSwitchCell.jsx`
    - Dihapus karena switch status di kolom tidak dipakai lagi.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #407 | Toggle Pelaporan di Tabel Pelanggan | Pelanggan dengan satu layanan bisa ditandai terlaporkan langsung dari tabel; istilah UI menjadi "Pelaporan / Terlaporkan" |
| #405 | Ubah Status Lewat Menu Aksi | Status aktif/nonaktif di 14 tabel tidak lagi berubah karena klik tidak sengaja; selalu lewat konfirmasi |

### Kemampuan Baru Pengguna/Admin

- Menandai atau membatalkan pelaporan pelanggan yang punya satu layanan langsung dari kolom **Pelaporan** di tabel Pelanggan.
- Mengubah status aktif/nonaktif lewat menu aksi baris, dengan modal konfirmasi yang menyebut nama data dan status tujuannya.

### Bug Fix / Solusi Masalah

- Status pelanggan, mitra, admin, karyawan, layanan, NAS, akun keuangan, dll tidak lagi berubah karena switch di kolom tersentuh tidak sengaja.
- Klik ganda pada konfirmasi ubah status tidak lagi mengembalikan status ke semula.
- Tabel menampilkan status akhir dari server, bukan status yang mungkin sudah basi di layar.

### Menu/Fitur Baru

- Toggle **Pelaporan** di tabel **Pengguna > Pelanggan** (khusus pelanggan dengan satu layanan).
- Aksi **Aktifkan / Nonaktifkan** di menu aksi baris tabel Pelanggan, Bisnis, Mitra, Pelanggan Mitra, Admin, Karyawan, Broadband, Hotspot, Vendor, NAS Radius, Radius Non-Pelanggan, Akun Perkiraan, Akun Keuangan, dan Transaksi Berulang.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur (Ubah Status Lewat Menu Aksi)**: Kolom Status di tabel kini hanya menampilkan badge. Untuk mengubah status, buka menu aksi baris lalu pilih Aktifkan atau Nonaktifkan, kemudian konfirmasi.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka salah satu tabel, misalnya **Pengguna > Pelanggan**.
  2. Klik tombol aksi (⋮) di baris yang ingin diubah.
  3. Pilih **Aktifkan** atau **Nonaktifkan**, lalu klik konfirmasi pada modal.

- **Penjelasan Fitur (Toggle Pelaporan)**: Pelanggan yang hanya punya satu layanan menampilkan switch di kolom Pelaporan. Pelanggan dengan beberapa layanan tetap menampilkan label ringkasan dan diatur dari form edit.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Pengguna > Pelanggan**.
  2. Pada kolom **Pelaporan**, klik switch untuk menandai atau membatalkan pelaporan.
  3. Untuk pelanggan dengan beberapa layanan, buka form edit lalu atur lewat kartu **Pelaporan**.
