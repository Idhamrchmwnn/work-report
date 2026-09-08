# 📝 Daily Work Report - Idham (2026-09-08)

---

## 📅 Laporan Harian - 8 September 2026

---

## 🌿 Branch: `issue-279` — Monitoring Request Perubahan Layanan dari Mobile App

### 📌 Informasi Issue

- **Nomor Issue**: #279, sub-issue dari #277 "Integrasi dengan MobileApps (Android)"
- **Judul Issue**: #277 - Integrasi Request Perubahan Layanan
- **Status Branch**: `Belum di-merge` (branch baru, sudah di-push ke `origin/issue-279`)

### 📅 Rincian Perubahan

#### [fcc05434] - save #279 - 8 September 2026, 14:43:19 WIB

- **Komponen yang Berubah**:
  - [`backend/src/services/mobileServiceChange.service.js`](backend/src/services/mobileServiceChange.service.js) [NEW] — Fungsi `findAllServiceChangeRequestsForTable()`: query ke model `RadiusAuthentication` (kredensial broadband pelanggan yang sudah ada — **tidak ada model baru dibuat**) dengan filter `req_change: { $exists: true, $ne: null }`, hanya menampilkan kredensial yang sedang punya permintaan perubahan produk aktif. Meng-`populate` `customer`, `bind_product` (produk aktif saat ini), `profile` (profil kecepatan aktif), dan `req_change` (produk yang diminta) supaya nama-nama tersebut langsung tampil, bukan hanya ID mentah. Ada catatan penting di komentar kode: field `req_change` dihapus lewat `$unset` (bukan di-set `null`) saat pembatalan, jadi filter harus memakai `$exists`, bukan sekadar cek nilai.
  - [`backend/src/controllers/mobileServiceChange.controller.js`](backend/src/controllers/mobileServiceChange.controller.js) [NEW] — Satu handler: `listServiceChangeRequests`, memanggil service di atas lalu mengembalikannya lewat format datatable standar (`{list, ...}`) atau 404 bila gagal.
  - [`backend/src/routes/mobileServiceChange.route.js`](backend/src/routes/mobileServiceChange.route.js) [NEW] — Satu endpoint: `POST /api/v1/mobile-service-change/list`, digerbang `protectedAdmin` + `checkPrivilege('mobileServiceChange.list')`, lengkap dokumentasi Swagger. Dipasang ke `backend/src/app.js`.
  - [`backend/src/config/privilege.json`](backend/src/config/privilege.json) — Grup privilege baru `mobileServiceChange` dengan satu aksi: `list` (murni baca, tidak ada `create`/`update`/`delete` — sesuai sifat halaman ini yang read-only).
  - [`frontend/src/app/navigation/mobileApp.js`](frontend/src/app/navigation/mobileApp.js) — Item menu baru **"Perubahan Layanan"** (ikon panah bolak-balik) di bawah root **"Mobile App"**, mengarah ke `/mobileApp/serviceChange`.
  - [`frontend/src/app/pages/mobileApp/serviceChange/index.jsx`](frontend/src/app/pages/mobileApp/serviceChange/index.jsx) [NEW] — Halaman list sederhana: satu `Datatables` yang menarik data dari endpoint di atas, diurutkan default berdasarkan `authentication_id` menurun.
  - [`frontend/src/app/pages/mobileApp/serviceChange/schema/columns.jsx`](frontend/src/app/pages/mobileApp/serviceChange/schema/columns.jsx) [NEW] — 6 kolom: ID Autentikasi, Pelanggan (tertaut ke profil pelanggan lewat `CustomerLinkCell`), Username, Produk Aktif, Profil Aktif, dan Produk yang Diminta. Tidak ada kolom aksi (approve/reject) — halaman ini murni untuk **melihat**, belum untuk memproses.
  - [`frontend/src/app/router/mobileApp/serviceChangeRoute.jsx`](frontend/src/app/router/mobileApp/serviceChangeRoute.jsx) [NEW] & [`frontend/src/app/router/protected.jsx`](frontend/src/app/router/protected.jsx) — Pendaftaran route `/mobileApp/serviceChange`.
  - [`backend/src/locales/{en,id}/translation.json`](backend/src/locales/id/translation.json), [`frontend/src/i18n/locales/{en,id}/translations.json`](frontend/src/i18n/locales/id/translations.json) — String baru untuk judul halaman, label kolom, dan pesan error daftar gagal dimuat.
- **Deskripsi Perubahan & Fungsi**:
  - Kemampuan pelanggan **meminta ganti paket/produk langganan lewat aplikasi mobile** sebenarnya **sudah ada sejak sebelumnya** (`requestChangeSubscription`/`cancelChangeSubscription` di `radiusAuthentication.controller.js`, dipakai lewat `mobileCustomer.route.js`) — pelanggan memilih produk baru dari aplikasi, sistem menyimpan `req_change` (ID produk yang diminta) pada kredensial broadband miliknya. **Yang belum ada sebelumnya: staf/admin di aplikasi utama sama sekali tidak punya cara melihat daftar permintaan ini** — data tersimpan di database tapi tidak terlihat di UI manapun. Pekerjaan hari ini menutup celah itu dengan menambahkan halaman monitoring khusus.
  - Ini murni **dashboard visibilitas (read-only)**, bukan alur approval berformulir. Belum ada tombol "Setujui"/"Tolak" — staf yang melihat permintaan di sini perlu memprosesnya secara manual (mis. lewat halaman edit kredensial broadband yang sudah ada di menu Layanan → Broadband) untuk benar-benar mengganti produk pelanggan tersebut.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Kegunaan Fitur**: Sebelumnya, ketika pelanggan mengajukan permintaan ganti paket internet lewat aplikasi mobile, permintaan itu **tersimpan di database tapi tidak terlihat oleh siapa pun di sisi perusahaan** — tim customer service/NOC tidak tahu ada permintaan masuk kecuali pelanggan menghubungi langsung, atau seseorang secara manual mengecek database. Halaman "Perubahan Layanan" ini menjadi **papan pantau terpusat**: satu tempat untuk melihat semua kredensial broadband yang sedang punya permintaan ganti produk tertunda, lengkap dengan info produk lama vs produk yang diminta, sehingga tim terkait bisa proaktif menindaklanjuti tanpa menunggu pelanggan komplain.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka sidebar **Mobile App → Perubahan Layanan**.
  2. Tabel yang tampil berisi seluruh pelanggan yang sedang mengajukan permintaan ganti paket — kolom **Produk Aktif** menunjukkan paket yang sedang mereka pakai, kolom **Produk yang Diminta** menunjukkan paket tujuan.
  3. Klik nama pelanggan pada kolom **Pelanggan** untuk membuka profil lengkapnya bila perlu verifikasi data lebih lanjut (riwayat, kontak, dsb).
  4. **Memproses permintaan** (langkah manual, di luar halaman ini untuk saat ini): buka menu **Layanan → Broadband**, cari kredensial pelanggan yang sama (bisa dicocokkan lewat kolom **ID Autentikasi**/**Username**), lalu ubah produk/profilnya sesuai permintaan lewat form edit yang sudah ada.
  5. Setelah produk pelanggan benar-benar diubah, permintaan tersebut **tidak otomatis hilang** dari daftar ini sampai field `req_change` dibersihkan — perlu diperhatikan apakah proses ubah produk yang sudah ada juga membersihkan field ini, atau perlu langkah tambahan (dicek terpisah, di luar cakupan perubahan hari ini).
  6. Pelanggan sendiri bisa membatalkan permintaannya dari aplikasi mobile kapan saja sebelum diproses — begitu dibatalkan, baris tersebut otomatis hilang dari daftar ini pada muat ulang berikutnya (karena `req_change` dihapus sepenuhnya, bukan dikosongkan).
