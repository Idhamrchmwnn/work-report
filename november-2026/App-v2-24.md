# 📝 Daily Work Report - Idham (2026-11-24)

---

## 📅 Laporan Harian - 24 November 2026

---

## 🌿 Branch: `issue-582` — Mobile Frontend Modul H: Organisasi (penyelesaian)

### 📌 Informasi Issue

- **Nomor Issue**: #582
- **Judul Issue**: Mobile Frontend: modul Organisasi (daftar & detail mitra, karyawan, admin, hak akses — khusus admin, hanya baca)
- **Status Branch**: `Belum di-merge`.
  - Commit `resolve #582` (`3970f855`) berada di atas `master` terbaru (setelah `resolve #580`) dan sudah ada di `origin/issue-582`.
  - Riwayat sebelum squash disimpan di branch lokal `backup/issue-582-pre-squash`.

### 📅 Rincian Commit

#### [e971d5f3] - save #582 - 23 November 2026, 17:30 (di-rebase 24 November, 14:17)

- Commit `save #582` kemarin di-rebase ke `master` setelah `resolve #580` di-merge (23 November, 19:01). Isinya sama dengan laporan 23 November.

#### [edd2af41] - save #582 - 24 November 2026, 14:58

- **Ringkasan**:
  - **Detail mitra di mobile dilengkapi**:
    - ID yang bisa disalin.
    - Kartu **Data Terkait** berisi tautan ke Pelanggan Mitra, Tagihan Mitra, dan Tagihan Pelanggan Mitra, dengan mitra sudah terpilih. Setiap tautan mengikuti hak akses halaman tujuannya.
    - Mitra yang tidak ditemukan (dihapus/disembunyikan) kini tampil sebagai pemberitahuan dengan tombol kembali ke daftar, bukan galat dengan tombol coba lagi.
  - **Partner API untuk detail & dokumen mitra diperketat**:
    - Mitra yang sudah dihapus (termasuk penanda hapus data lama berupa teks/angka) dijawab 404 di detail, profil, daftar dokumen, dan unduh.
    - Unduh dokumen oleh admin kini wajib `partner_id`, dan kepemilikan berkasnya selalu diverifikasi.
    - Admin yang membaca mitra lain tidak lagi menerima data milik-pemilik (kredensial login, catatan internal, saldo, perlengkapan, KTP/NPWP).

- **Komponen yang Berubah & Penjelasannya**:

  - **Backend**
    - `backend/src/controllers/partnerApiPartner.controller.js`
      - Daftar field khusus pemilik dibuang dari respons bila admin membaca mitra lain (`/partners/read/:id`, `/partners/profile?partner_id=`).
      - Daftar dokumen hanya ikut bila admin memegang `partnerDocument.read`.
      - Unduh dokumen:
        - Tanpa `partner_id`, atau dengan `partner_id` berupa array (query berulang), ditolak 400.
        - Berkas milik mitra lain atau pelanggan bisnis ditolak 404. Bucket dokumen juga dipakai modul lain, jadi unduh "langsung lewat nama" membuka berkas di luar cakupan.
        - Nama berkas kustom dibersihkan dari kutip/karakter kontrol.
    - `backend/src/services/partner.service.js`
      - Mitra terhapus disaring lewat `notTruthyFlag('deleted')`, bukan `$ne: true`, karena data lama bisa menyimpan penanda hapus sebagai `'true'`/`1`.
    - `backend/src/controllers/partnerApiInvoice.controller.js`
      - Riwayat faktur mitra yang sudah dihapus tetap boleh dibaca admin lewat opsi eksplisit (perilaku lama dipertahankan).
    - `backend/src/routes/partnerApi.route.js`, `AGENTS.md`
      - Swagger dan catatan aturan disesuaikan.
  - **Mobile frontend**
    - `mobile-frontend/src/pages/organization/PartnerDetailPage.jsx`
      - Ditambah ID yang bisa disalin, kartu Data Terkait, dan pemberitahuan "mitra tidak ditemukan".
    - `mobile-frontend/src/components/CopyValue.jsx` (baru)
      - Nilai dengan tombol salin. Juga dipakai `DeviceDetailPage` (community SNMP), menggantikan kode salin lokalnya.
    - `mobile-frontend/src/features/organization/organizationView.js`
      - Penyusun tautan Data Terkait. Pelanggan memakai `_id`, tagihan memakai `partner_id` publik. Tautan tanpa hak akses tujuan tidak dibuat.
    - `mobile-frontend/src/hooks/listCache.js`, `mobile-frontend/src/features/authSlice.js`
      - Cache daftar dikosongkan saat keluar dan saat masuk, supaya sesi berikutnya tidak melihat daftar milik sesi sebelumnya.
    - `mobile-frontend/src/pages/organization/components/PrivilegesTab.jsx`, `StaffSheet.jsx`, `i18n/locales/{id,en}/translations.json`, `PRD.md`
      - Penyesuaian tampilan dan teks. Status modul 6 (#580) di PRD diubah menjadi "Selesai".
  - **Test**
    - `backend/test/integration/partnerApiPartner.adminScope.test.js` (baru)
      - 10 test: mitra terhapus 404 (termasuk penanda hapus teks/angka), jalur riwayat faktur, `partner_id` wajib/array, berkas milik pihak lain 404, nama berkas dibersihkan, data milik-pemilik dengan/tanpa `partnerDocument.read`, dan mitra membaca profilnya sendiri.
    - `mobile-frontend/src/features/organization/organization.test.js`
      - +3 test tautan Data Terkait.
    - `mobile-frontend/src/features/authSlice.test.js`
      - +1 test pembersihan cache saat keluar/masuk.

#### [4c7c9484] - save #582 - 24 November 2026, 15:24

- **Ringkasan**:
  - Mengetuk hak akses di tab Hak Akses kini membuka **daftar karyawan pemegangnya**, bukan sekadar detail hak akses.
  - Akun yang sudah dihapus (termasuk penanda hapus data lama berupa teks/angka) tidak lagi tampil di daftar admin/karyawan.

- **Komponen yang Berubah & Penjelasannya**:

  - **Backend**
    - `backend/src/controllers/partnerApiStaff.controller.js`
      - `employees/list` menerima `privilege_id`. Role super dan `privilege_id` tak dikenal dijawab 404 (bukan 400 berupa teks), dan tetap butuh `employee.list`.
    - `backend/src/services/privilege.service.js`
      - Mencari hak akses biasa lewat id publik untuk filter `privilege_id`.
    - `backend/src/services/admin.service.js`, `employee.service.js`
      - Akun terhapus disaring `notTruthyFlag('deleted')` di daftar, filter, dan detail. `mongoose-delete` hanya menambah `deleted: { $ne: true }`, yang meloloskan nilai `'true'`/`1`.
    - `backend/src/routes/partnerApi.route.js`, `AGENTS.md`
      - Swagger `privilege_id` dan catatan aturan.
  - **Mobile frontend**
    - `mobile-frontend/src/pages/organization/components/PrivilegeEmployeesSheet.jsx` (baru)
      - Panel bawah berisi daftar karyawan aktif pemegang satu hak akses (administrator tidak ikut).
    - `mobile-frontend/src/pages/organization/components/PrivilegesTab.jsx`
      - Membuka panel di atas. Panel detail hak akses lama dihapus.
    - `mobile-frontend/src/services/organizationApi.js`, `endpoints.js`
      - Pemanggil detail hak akses (`privileges/read`) yang tidak lagi dipakai dihapus.
    - `mobile-frontend/PRD.md`, `README.md`, `i18n/locales/{id,en}/translations.json`
      - Baris modul H diperbarui.
  - **Test**
    - `backend/test/integration/partnerApiStaff.test.js`
      - +4 test: akun terhapus data lama, filter `privilege_id`, role super/id tak dikenal 404, dan `employee.list` tetap wajib.

#### [3970f855] - resolve #582 - 24 November 2026, 15:24

- Ketiga commit `save #582` di-squash menjadi satu di atas `master` (`resolve #580`). Isinya sama dengan gabungan commit di atas.

---

## 🌿 Branch: `issue-586` — Mobile Frontend Modul I: Beranda & Statistik

### 📌 Informasi Issue

- **Nomor Issue**: #586
- **Judul Issue**: Mobile Frontend: modul Beranda & Statistik (ringkasan per peran dari Partner API)
- **Status Branch**: `Belum di-merge`.
  - Commit `resolve #586` (`5de58877`) berada di atas `master` terbaru (`resolve #580`) dan sudah ada di `origin/issue-586`.
  - Commit ini menggabungkan tiga commit `save #586` (`c808fd65` 16:08, `1d3aafae` 17:11, `d78373ee` 17:46). Rinciannya di bawah dikelompokkan per berkas.

### 📅 Rincian Commit

#### [5de58877] - resolve #586 - 24 November 2026, 17:47

- **Ringkasan**:
  - **Beranda dan Statistik** kini memakai data asli Partner API, bukan data contoh. Ini modul terakhir sebelum tahap deploy.
  - **Beranda** berisi kartu ringkasan per peran, baris **Perlu tindakan** (hal yang bisa langsung dikerjakan pengguna), empat aksi cepat, pencarian pelanggan mitra, dan **Aktivitas Terbaru**. Mitra juga mendapat bagian **Tagihan Terbaru** dan checklist **Dokumen Legalitas**, setara Home `partner-frontend`.
    - Kartu ringkasan: pelanggan, PPPoE online/offline, tagihan, perangkat jaringan, dan dokumen.
    - Kartu yang butuh perhatian ditampilkan lebih dulu.
    - Setiap kartu membuka daftar yang sudah tersaring.
  - **Statistik** menampilkan empat angka per peran, panel kesehatan jaringan (perangkat UP, PPPoE online, latensi), dan diagram batang rincian per status.
    - Diagram batang ini menggantikan "Aktivitas Mingguan", karena Partner API tidak punya deret waktu.
  - **Pemanggilan data hemat**:
    - Tiap kartu dimuat terpisah, jadi galat satu kartu tidak menjatuhkan kartu lain.
    - Kartu tanpa hak akses tidak diminta sama sekali.
    - Hasilnya di-cache di memori selama 60 detik per pengguna, sehingga berpindah Beranda ↔ Statistik tidak memanggil ulang semua endpoint.
  - **Backend**: daftar `GET /p-api/v1/invoices` kini membawa `paid_total` (aditif), sehingga nominal tagihan belum dibayar = sisa setelah cicilan.
  - **Halaman Filter dihapus**: halaman itu berisi data contoh tanpa endpoint, jadi pencarian Beranda diganti menjadi pencarian pelanggan mitra.

- **Komponen yang Berubah & Penjelasannya**:

  - **Backend**
    - `backend/src/controllers/partnerApiInvoice.controller.js`
      - Field `paid_total` ditambahkan ke daftar field tagihan mitra.
    - `backend/src/routes/partnerApi.route.js`
      - Swagger `paid_total`.
  - **Mobile frontend — logika (murni)**
    - `mobile-frontend/src/features/home/homeSummary.js` (baru)
      - Seluruh aturan Beranda & Statistik ada di sini; halaman hanya merender:
        - Sumber data per kartu beserta key hak akses dan peran yang boleh memanggilnya.
        - Normalisasi respons. Respons yang tidak dikenali menjadi `null`, sehingga kartu menampilkan "–", bukan angka palsu 0.
        - Kartu dan urutannya: kartu yang perlu perhatian lebih dulu, tetapi urutan tetap stabil selama masih memuat.
        - Baris "Perlu tindakan", aksi cepat per peran, Aktivitas Terbaru, kesehatan jaringan, dan set batang diagram.
      - Aturan yang sengaja dibuat:
        - PPPoE offline = akun AKTIF yang tidak online. Akun nonaktif dihitung terpisah.
        - Kartu Pelanggan Mitra membuka daftar TANPA filter status: angka "aktif" `list-status` tidak memuat pelanggan pasif, sedangkan daftar ber-filter aktif memuatnya.
        - Tagihan pelanggan (khusus mitra) dihitung dari dua hitungan `customer-invoices/list` (`pageSize` 1, hanya `totalDocs`): belum dibayar, dan terlambat (jatuh tempo sampai kemarin).
        - Aktivitas Terbaru diisi pelanggan dengan aktivitas PPPoE 24 jam, lalu pelanggan baru 30 hari, karena Partner API tidak punya feed aktivitas.
        - Batang diagram hanya bisa diketuk bila daftar tujuannya bisa disaring persis sesuai batang itu.
    - `mobile-frontend/src/features/home/summaryCache.js` (baru)
      - Cache memori 60 detik. Kuncinya memuat identitas pengguna, dan cache dikosongkan saat keluar (`authSlice`).
    - `mobile-frontend/src/hooks/useHomeSummary.js` (baru)
      - Memuat tiap sumber secara independen.
      - Beranda menampilkan cache lalu mengambil ulang diam-diam sekali, kecuali cache masih sangat baru.
      - Tombol Perbarui selalu ke server tanpa mengosongkan kartu yang sudah tampil.
      - Sumber yang ditolak hak akses tidak dipanggil ulang.
      - Keterangan "Diperbarui ..." memakai waktu data tertua yang sedang tampil.
    - `mobile-frontend/src/services/customerApi.js`
      - Pemanggil `list-status` mentah, dipakai untuk hitungan sekaligus Aktivitas Terbaru.
  - **Mobile frontend — halaman**
    - `mobile-frontend/src/pages/HomePage.jsx`
      - Ditulis ulang dengan data asli. Mitra baru tanpa pelanggan diajak menambah pelanggan pertama. Pencarian disembunyikan tanpa `customerPartner.list`.
    - `mobile-frontend/src/pages/StatisticPage.jsx`
      - Grid empat angka, panel kesehatan jaringan, dan diagram batang rincian per status.
    - `mobile-frontend/src/pages/home/PartnerSections.jsx`, `presentation.js` (baru)
      - Bagian Tagihan Terbaru dan Dokumen Legalitas untuk mitra (tanpa request tambahan), serta ikon dan warna per tingkat kartu.
    - `mobile-frontend/src/pages/FiltersPage.jsx`, rute `/filters` di `App.jsx`
      - Dihapus.
    - `mobile-frontend/src/pages/FinancePage.jsx`, `finance/components/InvoiceCard.jsx`
      - Label tab dipendekkan agar tidak patah dua baris di HP (`aria-pressed`, `truncate`). Kartu tagihan memakai `activateOnKey`.
    - `mobile-frontend/src/utils/normalizeUser.js`
      - Label/angka statistik bawaan yang dulu kosong dihapus, karena kini dari `homeSummary`.
    - `mobile-frontend/src/data/mockData.js`
      - Data contoh Beranda/Statistik/Filter dihapus.
    - `mobile-frontend/src/i18n/locales/{id,en}/translations.json`
      - Key Beranda & Statistik.
    - `mobile-frontend/PRD.md`
      - Ditambah "Catatan modul I" (aturan di atas). Status modul 8 diubah menjadi "#586 — Selesai, menunggu uji perangkat & PR".
  - **Test**
    - `backend/test/integration/partnerApiInvoice.test.js`
      - +2 test: `paid_total` di daftar tagihan mitra.
    - `mobile-frontend/src/features/home/homeSummary.test.js` (baru)
      - 39 test aturan kartu, normalisasi, urutan, Perlu tindakan, aksi cepat, aktivitas, kesehatan jaringan, dan diagram.
    - `mobile-frontend/src/features/home/summaryCache.test.js` (baru)
      - 2 test.
    - `mobile-frontend/src/utils/normalizeUser.test.js`
      - 1 test statistik bawaan dihapus bersama fiturnya.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #582 | `edd2af41`, `4c7c9484` save; `3970f855` resolve #582 | Detail mitra dilengkapi (ID salin, Data Terkait, pemberitahuan mitra tak ditemukan), hak akses membuka daftar karyawan pemegangnya; Partner API: mitra/akun terhapus (termasuk penanda data lama) disembunyikan, unduh dokumen admin wajib `partner_id` + verifikasi kepemilikan, data milik-pemilik tidak dikirim ke admin; +18 test |
| #586 | `c808fd65`, `1d3aafae`, `d78373ee` save; `5de58877` resolve #586 | Beranda & Statistik mobile dengan data asli (kartu per peran, Perlu tindakan, aksi cepat, aktivitas, Tagihan Terbaru & Dokumen untuk mitra, kesehatan jaringan, diagram per status), cache 60 detik per pengguna, halaman Filter dihapus, `paid_total` di daftar tagihan mitra; +43 test |

> Catatan merge: `resolve #580` (dikerjakan 23 November) sudah di-merge ke `master` pada 23 November, 19:00–19:01, termasuk penyesuaian dari Dedy S.N Putra.
>
> Catatan branch: `issue-582` dan `issue-586` sama-sama berada di atas `master` terbaru dan masing-masing tidak konflik dengan `master`. Namun simulasi merge (`git merge-tree`) keduanya saling konflik di `mobile-frontend/PRD.md`, `mobile-frontend/src/App.jsx`, dan `mobile-frontend/src/data/mockData.js`, jadi branch yang di-merge kedua perlu di-rebase lebih dulu.
