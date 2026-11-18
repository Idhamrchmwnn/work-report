# 📝 Daily Work Report - Idham (2026-11-18)

---

## 📅 Laporan Harian - 18 November 2026

---

## 🌿 Branch: `issue-559` — Mobile Frontend Modul B: Pelanggan (penyelesaian)

### 📌 Informasi Issue

- **Nomor Issue**: #559
- **Judul Issue**: Mobile Frontend: modul Pelanggan (Reguler, Mitra, Bisnis) terhubung ke Partner API
- **Status Branch**: `Sudah di-merge`. Commit `resolve #559` (`1fd3bdc5`) di atas `master` (setelah `resolve #556`) di-merge ke `master` pada 18 November, 16:33.

### 📅 Rincian Commit

#### [1fd3bdc5] - resolve #559 - 18 November 2026, 10:55

- **Ringkasan**:
  - Empat commit `save #559` kemarin di-squash menjadi satu. Rincian modul Pelanggan ada di laporan 17 November.
  - Tambahan hari ini sebelum commit (30 berkas, +257 / −161 baris):
    - **Urutan daftar yang stabil.** Daftar berhalaman kini selalu berurutan dan berakhir di kolom unik. Tanpa urutan, MongoDB memakai urutan alami yang tidak stabil, sehingga data bisa muncul dua kali atau hilang di antara halaman.
    - **Tata letak tablet.** Lebar aplikasi dinaikkan dari 430 px ke `max-w-5xl`, daftar kartu menjadi 2 kolom di `md`, dan panel bawah tampil di tengah layar.
    - **Perapian sesi.** Refresh token tidak lagi disimpan. Login tanpa profil yang valid kini gagal dan membersihkan token.

- **Komponen yang Berubah & Penjelasannya (tambahan hari ini)**:

  **Urutan & paginasi**

  - `mobile-frontend/src/utils/dataTable.js`
    - `sort` kini boleh berisi beberapa kolom berurutan.
    - Komentar menegaskan bahwa daftar berhalaman wajib berakhir di kolom unik.
  - `mobile-frontend/src/features/customers/customerKinds.js`
    - `listSortFor(kind)`: terbaru dulu (`created_at`), lalu ID publik per jenis (`customer_id`/`partner_id`) sebagai pemutus seri.
  - `mobile-frontend/src/pages/CustomersPage.jsx`
    - Memakai `listSortFor`. Grid 2 kolom di tablet; keadaan memuat/galat/kosong dan paginasi selebar grid.
  - `mobile-frontend/src/components/PartnerPicker.jsx`
    - Urutan tetap: nama, lalu `partner_id`, supaya halaman berikutnya tidak mengulang atau melewatkan mitra.

  **Label & tampilan**

  - `mobile-frontend/src/features/customers/customerStatus.js`
    - `serviceTypeLabelKey`: jenis layanan yang tidak dikenal memakai label umum "Layanan", bukan key i18n mentah.
  - `mobile-frontend/src/pages/customers/components/CustomerServicesTab.jsx`, `CustomerDocumentsTab.jsx`
    - Memakai label baru. Daftar dokumen dibungkus grid.
  - `mobile-frontend/src/pages/{Finance,Network,Products}Page.jsx`
    - Status yang tadinya tampil mentah (`inv.status`, `dev.status`, `req.status`) kini diterjemahkan.
    - Teks "Baru saja" diganti `t('common.justNow')`.
  - `mobile-frontend/src/components/{MobileShell,BottomSheet,ConfirmDialog,ChunkLoadFailed,PwaUpdatePrompt}.jsx`, `map/MapPickerSheet.jsx`, `pages/LoginPage.jsx`, `pages/HomePage.jsx`, `pages/customers/CustomerFormPage.jsx`, `App.jsx`
    - Lebar maksimum disesuaikan untuk tablet.
  - `mobile-frontend/src/components/MobileShell.jsx`
    - Status offline memakai `useOnline` (pendengar `online`/`offline` ganda dihapus).
    - Esc juga menutup menu samping, dan menu yang terlihat dihitung `useMemo` per peran.
  - `mobile-frontend/src/components/SegmentedControl.jsx` [DIHAPUS]
    - Sudah digantikan `SettingSelect`.
  - `mobile-frontend/src/hooks/useConfig.js`
    - Ejaan "Managemen" → "Manajemen" pada nama aplikasi bawaan.

  **Sesi & klien API**

  - `mobile-frontend/src/utils/authStorage.js`
    - `refreshToken` dihapus, karena Partner API tidak punya refresh token.
  - `mobile-frontend/src/features/authSlice.js`
    - Login hanya menyimpan `authToken`. Profil yang tidak bisa dinormalkan dianggap gagal masuk (`missing-profile`), dan token dibersihkan.
  - `mobile-frontend/src/utils/apiClient.js`
    - Pengecualian 401 untuk login/logout kini mencocokkan path persis (tanpa query/hash), bukan `includes`.
    - Sebelumnya URL lain yang kebetulan memuat teks path login juga ikut dikecualikan.

  **Test** (tambahan hari ini)

  - `src/features/customers/customers.test.js`: +2 skenario (urutan daftar per jenis, label jenis layanan tak dikenal).
  - `src/utils/dataTable.test.js`: +1 skenario (urutan beberapa kolom).
  - `src/features/authSlice.test.js`: +1 skenario (respons tanpa profil valid membersihkan token dan gagal masuk).

---

## 🌿 Branch: `issue-563` — Partner API: Cakupan Akun PPPoE Bocor & Body Tanpa Daftar-Putih

### 📌 Informasi Issue

- **Nomor Issue**: #563
- **Judul Issue**: Partner API RADIUS: pencarian/filter menampilkan akun mitra lain, dan body create/update diteruskan utuh ke database
- **Status Branch**: `Sudah di-merge`. Commit `resolve #563` (`2b021438`) di atas `master` (setelah `resolve #556`) di-merge ke `master` pada 18 November, 16:38.

### 📅 Rincian Commit

#### [2b021438] - resolve #563 - 18 November 2026, 14:47

- **Ringkasan**:
  - Ditemukan saat menghubungkan modul RADIUS `mobile-frontend` (#561). Perbaikannya di backend.
  - **Cakupan kepemilikan bisa dilebarkan klien.**
    - Cakupan mitra dipasang sebagai `$or` di `params.find`, lalu ikut dilebarkan pencarian global (`globalFilter`) dan filter kolom `select`. Akibatnya mitra bisa melihat akun PPPoE milik mitra lain.
    - Kini cakupan dipasang server lewat `serverScope` (`serverFind`), digabung `$and` **setelah** pencarian/filter klien diproses.
    - `find`/`populate` kiriman klien dibuang, termasuk operator seperti `$where` yang sebelumnya sampai ke MongoDB pada daftar sesi.
  - **`find.customer` tidak berfungsi.** Filter per pelanggan (tab Layanan detail pelanggan) tertimpa `customer: { $exists: true }` di service, sehingga semua akun tampil.
    - Kini menjadi satu-satunya penyempitan klien yang diterima. ID tidak sah, objek, atau pelanggan di luar cakupan dijawab daftar kosong, bukan 500.
  - **Body create/update akun diteruskan utuh.**
    - Mitra bisa menulis field milik sistem: `usage` (mereset pemakaian kuota), `isLimited`, `isOnline`, `last_session` (sasaran putus sesi), `authentication_id`, dan `monthly_report`.
    - Kini memakai **daftar-putih** field + cek tipe. Operator Mongo pada field skalar ditolak 400 tanpa mengubah akun.
  - **Putus sesi**:
    - `username` wajib teks. Objek seperti `{ $ne: null }` sebelumnya memilih akun sembarang.
    - Pesan galat Inggris mentah diganti i18n. NAS yang tidak terdaftar dicatat `warn`.
  - **Pilihan profil**: teks cari di-escape (mencegah ReDoS), dan non-string dijawab kosong.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/utils/partnerApiRadiusInput.js` [NEW]
    - `PARTNER_API_RADIUS_USER_FIELDS`: 27 field yang boleh dikirim aplikasi eksternal.
    - Sengaja tidak memuat penanda pemilik (`ref_partner`, `pid`, `_id`), `tax_reported`, mode tagihan yang dikunci server, dan status runtime radius/penagihan.
    - `pickPartnerApiRadiusUserInput`: hanya `speed_option` yang boleh berupa objek (isinya skalar). Objek/array pada field lain ditolak.
  - `backend/src/controllers/partnerApiRadius.controller.js`
    - `applyRadiusUserInput` menggantikan `stripServerOwnedFields` (daftar-hitam 3 field). Body disaring di tempat karena handler membacanya lagi.
    - Daftar akun memakai `serverScope` dengan `$and` [cakupan mitra, `find.customer` bila ada].
    - Daftar sesi dan log memakai `serverScope`. Pada log, filter kolom `authentication` (ada di daftar-putih datatable) tidak bisa lagi menunjuk akun mitra lain.
    - Putus sesi: validasi `username` teks, `errorKey` i18n, dan 404 untuk akun tak ditemukan/di luar cakupan.
  - `backend/src/controllers/partnerApiProductBroadband.controller.js`
    - Daftar pelanggan per paket memakai `serverScope` yang sama.
  - `backend/src/services/radiusAuthentication.service.js`
    - `findListAuthenticationForTable(params, serverScope)`. Tanpa `serverScope`, perilaku lama aplikasi utama dipertahankan.
    - Populate daftar dipindah ke konstanta `AUTHENTICATION_LIST_POPULATE`.
  - `backend/src/services/radiusSession.service.js`
    - `findRadiusSessionsForTable` dan `findRadiusLogsForTable` menerima opsi `serverScope`.
  - `backend/src/services/radiusProfile.service.js`
    - `findProfileBroadbandSelectForPartnerApi`: query wajib string dan di-escape `escapeRegex`.
    - `catch` kini mencatat `logger.error` (sebelumnya menelan galat).
  - `backend/src/routes/partnerApi.route.js`
    - Swagger: cakupan yang tidak bisa dilebarkan klien, `find.customer`, dan daftar field yang diterima.
  - `backend/src/locales/{id,en}/translation.json`
    - Key `radius.session.usernameRequired` dan `radius.session.nasNotRegistered`.

  **Test**

  - `backend/test/integration/partnerApiRadius.test.js`: +11 skenario (35 → 46)
    - pencarian global dan filter `select` tidak menampilkan akun mitra lain;
    - `find.customer` menyempitkan ke pelanggan itu, sedangkan milik mitra lain/tidak sah/operator menghasilkan daftar kosong;
    - `find` lain dari klien diabaikan;
    - log: filter `authentication` tidak bisa menunjuk akun mitra lain;
    - sesi: `find` klien termasuk `$where` tidak diteruskan;
    - update: field milik sistem diabaikan, dan operator Mongo ditolak 400;
    - disconnect: `username` objek tidak memilih akun sembarang;
    - pilihan profil: teks di-escape, non-string dijawab kosong.
  - `backend/test/integration/partnerProductDerived.test.js`
    - Regresi: akun mitra lain pada paket turunan yang sama tidak bocor lewat pencarian atau filter `select`.

---

## 🌿 Branch: `issue-561` — Mobile Frontend Modul C: Layanan RADIUS (penyelesaian)

### 📌 Informasi Issue

- **Nomor Issue**: #561
- **Judul Issue**: Mobile Frontend: modul Layanan RADIUS (akun PPPoE, sesi online, profil) terhubung ke Partner API
- **Status Branch**: `Belum di-merge`. Commit `resolve #561` (`a2a0dbd2`) sudah di-rebase ke `master` terbaru (setelah `resolve #563`) dan sudah ada di `origin/issue-561`.
  - Masalah dari laporan kemarin (#556 bisa masuk dua kali) sudah selesai.
  - Riwayat sebelum squash disimpan di branch lokal `backup/issue-561-pre-squash`, dan posisi sebelum rebase sore disimpan di `backup/issue-561-pre-rebase`.

### 📅 Rincian Commit

#### [c2324d49] - save #561 - 18 November 2026, 12:22

#### [a2a0dbd2] - resolve #561 - 18 November 2026, 15:41 (rebase ke `master` pukul 17:19; sebelumnya `1cd22e86`)

- **Ringkasan**:
  - Commit `save #561` kemarin di-rebase ke `resolve #559`, lalu ditambah perbaikan hari ini dan di-squash menjadi `resolve #561`. Rincian modul RADIUS ada di laporan 17 November.
  - Tambahan hari ini (18 berkas, +426 / −129 baris):
    - **Urutan daftar yang stabil** untuk akun, sesi, profil, dan log, mengikuti pola #559.
    - **Penjadwal penyegaran berkala yang tidak bisa berlipat.**
    - **Putus sesi yang lebih rapi.**

- **Komponen yang Berubah & Penjelasannya (tambahan hari ini)**:

  - `mobile-frontend/src/features/radius/listSort.js` [NEW]
    - `ACCOUNT_SORT` (terbaru dulu, sama dengan tabel web).
    - `SESSION_SORT` (mulai terbaru, `sessionID` pemutus seri).
    - `PROFILE_SORT` (nama A–Z, lalu `profile_id`).
    - `LOG_SORT` (`date` + `topic`). Daftar-putih log tidak memuat kolom unik, jadi log dengan waktu persis sama masih bisa bertukar. Batasan ini dicatat di komentar.
  - `mobile-frontend/src/pages/radius/components/{AccountsTab,SessionsTab,ProfilesTab,AccountHistoryTabs}.jsx`, `components/CustomerPicker.jsx`, `pages/radius/AccountFormPage.jsx`
    - Memakai urutan tetap di atas.
  - `mobile-frontend/src/hooks/useAutoRefresh.js`
    - Inti penjadwal dipisah menjadi fungsi murni `startAutoRefresh` dengan **satu** rantai timer.
    - Tab yang terlihat lagi saat penyegaran masih berjalan tidak memulai rantai kedua.
    - Setelah `stop()` tidak ada panggilan maupun timer baru, termasuk dari penyegaran yang sedang berjalan.
  - `mobile-frontend/src/pages/radius/components/useDisconnect.jsx`
    - 404 (akun/sesi tidak ditemukan) dan 409 (NAS tidak terdaftar) diringkas menjadi satu pesan "tidak dapat diputus".
    - Muat ulang berkala setelah sukses (`reloadDelays`) dibatalkan saat halaman ditinggalkan, jadi tidak memanggil API untuk halaman yang sudah ditutup.
  - `mobile-frontend/src/pages/radius/AccountDetailPage.jsx`, `ProfileFormPage.jsx`, `pages/customers/components/CustomerServicesTab.jsx`
    - Penyesuaian pemanggilan hook dan urutan.
  - `mobile-frontend/src/utils/time.js`
    - Waktu relatif di masa depan (jam server/perangkat tidak selaras) dibatasi ke "sekarang", bukan "dalam 2 menit" (pola revisi #479 di web).
  - `mobile-frontend/src/i18n/locales/{id,en}/translations.json`
    - Pesan baru untuk putus sesi.

  **Test** (tambahan hari ini)

  - `src/hooks/autoRefresh.test.js` [NEW], 5 skenario: tick berikutnya setelah selesai, tab terlihat saat berjalan/diam tidak menggandakan rantai, tab tersembunyi dilewati, tidak ada `run` setelah `stop`.
  - `src/utils/time.test.js` [NEW], 2 skenario: waktu relatif dan tanggal tidak sah.
  - `src/features/radius/radius.test.js`: +1 skenario (setiap urutan berakhir di kolom unik).

---

## 🌿 Branch: `issue-565` — Mobile Frontend Modul D: Paket Broadband

### 📌 Informasi Issue

- **Nomor Issue**: #565
- **Judul Issue**: Mobile Frontend: modul Paket Broadband (katalog, paket turunan mitra, permohonan) + perbaikan penulisan paket di Partner API
- **Status Branch**: `Belum di-merge`. Masih work-in-progress (`save #565`). Dibangun di atas `resolve #561`, jadi menunggu #561 di-merge lebih dulu.
  - Sudah di-rebase lokal ke `resolve #561` yang baru (`2f750f5d`), tetapi **belum di-push**: `origin/issue-565` masih berisi versi lama `f108cb80`. Posisi sebelum rebase disimpan di `backup/issue-565-pre-rebase`.

### 📅 Rincian Commit

#### [2f750f5d] - save #565 - 18 November 2026, 16:59 (rebase pukul 17:19; sebelumnya `f108cb80`)

- **Ringkasan**:
  - Modul Paket Broadband dihubungkan ke Partner API. Tab **Katalog** dan **Paket Turunan** mengikuti desain awal.
    - **Mitra:** katalog paket induk dengan tombol "Ajukan Turunan", paket turunan miliknya, ajukan perubahan, serta riwayat dan pembatalan permohonan.
    - **Admin:** semua paket; tambah paket katalog internal **atau** paket turunan untuk mitra terpilih (langsung aktif); ubah/hapus; riwayat permohonan semua mitra (hanya baca).
    - Persetujuan permohonan tetap di aplikasi utama.
  - **Perbaikan backend penulisan paket** di Partner API:
    - Create meneruskan body utuh, sehingga sesi admin bisa menulis penanda pemilik (`partner`, `pid`, `ref_partner`) dan penanda paket turunan (`is_derived`, `parent_product`, `source_request`). Kini memakai daftar-putih + cek tipe.
    - Update tidak lagi memakai `cleanFormData`, yang membuang harga `0` (paket gratis tidak tersimpan).
    - Mitra yang mengirim ulang `radius_profile` paket turunannya tidak lagi ditolak 422. Sebelumnya `profile_id` dibandingkan dengan `_id`, sehingga selalu dianggap berubah.
    - Endpoint baru `create-derived`: admin membuat paket turunan untuk mitra.
  - Data contoh paket dihapus dari `data/mockData.js`.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/utils/partnerApiProductInput.js` [NEW]
    - `pickPartnerApiProductInput(body, { mode })` menerima hanya field `PARTNER_API_PRODUCT_FIELDS` dan menolak objek/array (operator Mongo).
    - Validasi nilai:
      - `price` >= 0;
      - `billing_period_months` bilangan bulat 1–120;
      - `vlan_id` 1–4094;
      - boolean wajib boolean.
    - Update: `code` dibuang, dan `'remove-data'` dipertahankan kecuali pada field wajib.
  - `backend/src/controllers/partnerApiProductBroadband.controller.js`
    - Create paket katalog internal: penanda pemilik dan paket turunan selalu dari server.
    - `radius_profile` diresolusi dari `profile_id` ke `_id` profil yang boleh dilihat pemanggil. Profil asing/tak ada dijawab 400.
    - Update: daftar-putih, perbandingan field terkunci memakai `_id`, dan tanpa perubahan mengembalikan data saat ini (`$set` kosong ditolak MongoDB).
    - `createDerivedPartnerAppProductBroadband` [NEW]: hanya admin. Mitra harus reseller yang tampil di portal; mitra tersembunyi/tidak ada dijawab 404. Hasilnya tercatat sebagai permohonan yang disetujui.
    - Admin tidak bisa mengubah/menghapus paket milik mitra tersembunyi.
    - Daftar membawa nama mitra pemilik; detail membawa `owner` tanpa mengubah `product.partner`.
    - Alasan batal permohonan wajib teks <= 500 karakter (objek dulu memicu CastError 500).
  - `backend/src/controllers/partnerApiRadiusProfile.controller.js`, `services/radiusProfile.service.js`
    - `getAdminRadiusProfileVisibilityFilter` untuk pilihan profil admin.
  - `backend/src/services/productBroadband.service.js`
    - `deleteProductBroadbandByIdForPartnerApi`: paket non-turunan dihapus lewat soft delete.
    - Populate nama mitra untuk daftar.
  - `backend/src/routes/partnerApi.route.js`
    - Route `POST /p-api/v1/products/broadband/create-derived` beserta Swagger, dan dokumentasi field create/update.
  - `backend/src/locales/{id,en}/translation.json`
    - Key galat baru.
  - `AGENTS.md`
    - §2.A aturan 37 diperluas untuk `create-derived`.
    - §2.M butir 4: payload paket wajib lewat `packagePayload.js`/`requestPayload.js`.

  **Mobile Frontend**

  - `mobile-frontend/src/features/products/` [NEW], logika murni:
    - `packagePayload.js`
      - Tambah: kosong tidak dikirim, boolean selalu dikirim.
      - Ubah: hanya field berubah, dan kosong menjadi `'remove-data'`.
      - Mitra pada paket turunan tidak pernah mengirim field komersial.
    - `requestPayload.js`
      - Ajukan paket turunan (admin menyertakan `partner_id`) dan ajukan perubahan. Kecepatan hanya dikirim bila diubah.
    - `productSchema.js`
      - Skema paket, paket turunan (kecepatan wajib dan bukan `0`), dan perubahan (alasan wajib).
    - `packageDisplay.js`
      - Aturan tombol per peran, field terkunci, dan periode tagihan.
    - `requestStatus.js`
      - `approving` ditampilkan sebagai "menunggu". Batal hanya oleh mitra dan hanya untuk status `pending`.
  - `mobile-frontend/src/services/productApi.js` [NEW], `services/endpoints.js`
  - `mobile-frontend/src/pages/ProductsPage.jsx`
    - Tab Katalog/Paket Turunan dan daftar permohonan, menggantikan data contoh.
  - `mobile-frontend/src/pages/products/` [NEW]
    - `PackageDetailPage.jsx`, `PackageFormPage.jsx` (admin tambah/ubah; katalog internal atau turunan untuk mitra).
    - `DerivedFormPage.jsx` (ajukan turunan/perubahan).
    - `RequestDetailPage.jsx`.
    - Komponen `PackageList`, `ParentCatalog`, `RequestsList`, `PackageKindSwitch`, `CancelRequestDialog`.
  - `mobile-frontend/src/features/radius/speedLimit.js`
    - Satuan kecepatan Mikrotik `k/M/G` dibaca sebagai kbps, tanpa satuan = bit/detik. Burst tanpa `use_burst` dikenali dari bentuk `speed_limit`.
  - `mobile-frontend/src/components/form/SearchSelectField.jsx`, `components/ConfirmDialog.jsx`, `App.jsx`
    - Penyesuaian untuk form paket dan rute `/products/*`.
  - `mobile-frontend/src/i18n/locales/{id,en}/translations.json`
    - Key `products.*`, sekitar 160 baris masing-masing.
  - `mobile-frontend/PRD.md`
    - Modul D diperbarui (termasuk admin tambah paket turunan untuk mitra) dan ditandai "Selesai, menunggu PR".

  **Test**

  - `backend/test/integration/partnerApiProductBroadband.write.test.js` [NEW], 12 skenario:
    - create mengabaikan penanda pemilik dan paket turunan dari body;
    - meresolusi `radius_profile`;
    - menolak operator Mongo dan periode tidak sah;
    - update menyimpan periode, harga 0, dan `remove-data`;
    - mitra mengirim ulang `radius_profile` tidak 422;
    - mitra tersembunyi tidak bisa diubah/dihapus admin;
    - soft delete;
    - `create-derived` (berhasil, 403/404/400);
    - permohonan mitra tersembunyi disembunyikan;
    - nama pemilik di daftar/detail;
    - alasan batal <= 500 karakter.
  - `mobile-frontend/src/features/products/products.test.js` [NEW], 18 skenario: aturan tombol, payload paket, payload paket turunan, payload ajukan perubahan, status permohonan, dan satuan kecepatan Mikrotik.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #559 | `1fd3bdc5` resolve #559 | Penyelesaian modul Pelanggan mobile: urutan daftar stabil, tata letak tablet, label status terjemahan, perapian sesi; +4 test (sudah di-merge) |
| #563 | `2b021438` resolve #563 | Backend Partner API RADIUS: cakupan mitra tidak bisa dilebarkan pencarian/filter/`find` klien, `find.customer` berfungsi, daftar-putih body akun, validasi putus sesi & pencarian profil; +12 test (sudah di-merge) |
| #561 | `c2324d49` save, `a2a0dbd2` resolve #561 | Penyelesaian modul RADIUS mobile: urutan daftar stabil, penjadwal penyegaran satu rantai, putus sesi lebih rapi; +8 test; di-rebase ke `master` terbaru |
| #565 | `2f750f5d` save #565 | Modul Paket Broadband mobile (katalog, paket turunan, permohonan, admin tambah paket/turunan) + backend: daftar-putih body paket, harga 0, `create-derived`; 30 test (WIP) |

> Catatan merge: `resolve #559` dan `resolve #563` sudah di-merge ke `master` pada 18 November (16:33 dan 16:38).
>
> Catatan branch: `issue-565` dibangun di atas `resolve #561`, jadi PR #565 sebaiknya menunggu #561 di-merge.

### 🔀 Rebase Sore (17:19)

Setelah #559 dan #563 di-merge, `issue-561` di-rebase ke `master` terbaru, lalu `issue-565` ke `issue-561` yang baru.

- **Konflik `issue-561`**: saat merge #559, Dedy menambah perubahan sendiri, yaitu `columnVisibility` pada daftar dan pengecualian gambar avatar publik di AGENTS.
  - `mobile-frontend/src/hooks/usePagedList.js`: memakai versi #561 (muat ulang senyap), dan `columnVisibility` dari Dedy dimasukkan kembali ke `buildListParams`.
  - `AGENTS.md` §2.M butir 4: memakai rumusan #561 yang lebih lengkap. Pengecualian avatar di butir 2 tetap ada.
- **Konflik `issue-565`**: `backend/src/services/radiusProfile.service.js`, hanya import. Keduanya dipertahankan: `escapeRegex` dari #563 dan `getHiddenPortalPartnerIds` dari #565.
- **Verifikasi setelah rebase**:
  - `issue-561`: 152 test mobile lulus, ESLint dan build lulus.
  - `issue-565`: 170 test mobile lulus, ESLint dan build lulus. 145 test backend terkait (Partner API Paket, RADIUS, paket turunan, cakupan i18n) lulus.
