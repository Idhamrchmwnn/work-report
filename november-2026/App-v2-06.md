# 📝 Daily Work Report - Idham (2026-11-06)

---

## 📅 Laporan Harian - 6 November 2026

---

## 🌿 Branch: `issue-529` — Partner API: Daftar Mitra, Administrator, Karyawan, dan Hak Akses

### 📌 Informasi Issue

- **Nomor Issue**: #529
- **Judul Issue**: Partner API: endpoint daftar mitra bisnis (POP) dan data internal perusahaan (administrator, karyawan, hak akses) untuk aplikasi eksternal
- **Status Branch**: `Belum di-merge`. Satu commit `resolve #529` di atas `master` terbaru (setelah `resolve #527`), sudah ada di `origin/issue-529`.

### 📅 Rincian Commit

#### [0a319b8a] - resolve #529 - 6 November 2026, 16:29

> Commit siang (`d6ae4e7b`, 13.59, daftar mitra) diperluas dengan endpoint data internal, lalu di-amend menjadi satu commit ini.

- **Ringkasan**:
  - **Daftar mitra**: endpoint baru `POST /p-api/v1/partners/list` (alias `/partner/list`).
    - Isinya **himpunan yang sama** dengan cakupan `/p-api/v1/customer-partners/*` (reseller aktif ber-`show_in_portal: true`), jadi `partner_id` dari daftar ini bisa langsung dipakai untuk endpoint pelanggan mitra. Mitra yang sudah dihapus tidak pernah ikut.
    - Cakupan ditentukan dari **token**: sesi mitra hanya melihat dirinya sendiri, sesi admin melihat seluruh mitra yang tampil di portal. Kiriman `find`/`withDeleted` dari klien diabaikan.
    - Field dibatasi pada identitas dan kontak publik. KTP, NPWP, saldo, dan field internal tidak pernah ikut.
  - **Data internal perusahaan** (baca-saja), enam endpoint baru:
    - `POST /p-api/v1/admins/list`, `GET /p-api/v1/admins/read/:id`: administrator aktif (`super: true`);
    - `POST /p-api/v1/employees/list`, `GET /p-api/v1/employees/read/:id`: karyawan aktif (`super: false`);
    - `POST /p-api/v1/privileges/list`, `GET /p-api/v1/privileges/read/:id`: role hak akses biasa.

    Semuanya punya alias bentuk tunggal (`/admin/...`, `/employee/...`, `/privilege/...`).
  - Keenam route ini adalah **satu-satunya route p-api yang memeriksa hak akses** sesi admin. Rantainya: `protectedPartnerApp` → `partnerApiAdminOnly` (sesi mitra 403) → `checkPrivilege` dengan key yang sama dengan endpoint internalnya (`admin.list/read`, `employee.list/read`, `privilege.list/read`).
  - Sengaja **tanpa jalur tulis**: route p-api lain tidak memeriksa hak akses, jadi membuat admin atau mengganti role dari sini akan menjadi pintu eskalasi hak.
  - **Akun developer tidak pernah tampil.** Akun ini bisa membuat super password, jadi paling berharga sebagai sasaran.
  - Field yang dikirim memakai daftar-putih:
    - administrator: `admin_id`, `name`, `privilege`;
    - karyawan: ditambah `employee_id`, `position`, `division`, `join_date`;
    - hak akses: `privilege_id`, `name`, `comment`.

    Username, kontak, KTP/NPWP, kredensial, login router (`remote_password`/`remote_group`), dan `_id` tidak ikut. Daftar key hak akses (`role`) tidak pernah dikirim, dan role super ditampilkan `null`.
  - Body datatable yang salah bentuk ditolak **400** di semua endpoint daftar baru. Sebelumnya `dataTable` melempar TypeError, sehingga permintaan rusak berujung 500 dan memicu alert error.
  - Ikut diperbaiki: pemindai endpoint untuk kamus hak akses dan katalog endpoint AI Agent kini memakai prefix pemasangan yang benar per berkas route. Sebelumnya endpoint Partner API bisa tercatat sebagai `/api/v1/...` yang tidak ada.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — daftar mitra**

  - `backend/src/controllers/partnerApiPartner.controller.js`
    - Handler baru `listPartnerAppPartner`:
      - memeriksa bentuk body lewat `isValidDataTableParams` (400 `global.invalidInput`);
      - membuang `withDeleted`;
      - menentukan cakupan dari `req.authRole` (mitra: `_id` miliknya sendiri; admin: semua);
      - memanggil service.

  - `backend/src/services/partner.service.js`
    - Fungsi baru `findListPartnerForPartnerApi` dengan daftar-putih field `PARTNER_API_LIST_FIELDS`: `partner_id`, `name`, `type`, `status`, `email`, `phone`, `area`, `city`, `province`, `regency`, `address`, `coordinate`, `created_at`.
    - Cakupan dari controller selalu digabung `$and` dengan `{ reseller: true, show_in_portal: true }`, jadi controller tidak bisa melebarkannya.
    - Kegagalan dicatat `logger.error` dengan konteks, lalu dilempar ulang dengan pesan i18n `partner.failedGetList`.

  **Backend — administrator, karyawan, hak akses**

  - `backend/src/controllers/partnerApiStaff.controller.js` [NEW]
    - Factory `makeListHandler` (cek bentuk body → 400, buang `withDeleted`) dan `makeReadHandler`. Id di luar cakupan dijawab 404 generik, sama dengan id yang tidak ada, supaya keberadaannya tidak terkonfirmasi.
    - Handler: `listPartnerAppAdmin`/`readPartnerAppAdmin`, `listPartnerAppEmployee`/`readPartnerAppEmployee`, `listPartnerAppPrivilege`/`readPartnerAppPrivilege`. Respons detail berbentuk `{ admin }`, `{ employee }`, `{ privilege }`.

  - `backend/src/middlewares/partnerApiAdminOnly.middleware.js` [NEW]
    - Menolak sesi selain admin dengan 403, dengan bentuk respons yang sama seperti `checkPrivilege`.
    - Dipasang **sebelum** `checkPrivilege`: dokumen mitra tidak punya field `privilege`, sehingga tanpa ini `hasPrivilege` mencari dan meng-cache privilege `undefined` di Redis.

  - `backend/src/services/admin.service.js`
    - `findListAdminForPartnerApi` dan `findOneAdminForPartnerApi`.
    - Cakupan: `super: true`, `status: true`, dan bukan developer (`notTruthyFlag('developer')`).
    - Populate `privilege` hanya `privilege_id name`, dan role super disaring (`notTruthyFlag('isSuper')`) sehingga tampil `null`.
    - Kolom `privilege` dibuang dari `sorting`/`columnFilters` (`omitTableColumns`). Mengurutkan berdasarkan kolom ini akan mengelompokkan akun yang berbagi role super, padahal role itu sengaja disembunyikan.
    - Hasil dilewatkan `toPublicTable`/`toPublicRecord`, jadi `_id` tidak ikut.
    - Gagal baca daftar: `admin.failedGetList`. Gagal baca detail: `global.getDataFailed` (500, bukan 404).

  - `backend/src/services/employee.service.js`
    - `findListEmployeeForPartnerApi` dan `findOneEmployeeForPartnerApi`, dengan cakupan `super: false`, `status: true`, serta populate, kolom terlarang, dan penanganan error yang sama dengan administrator.

  - `backend/src/services/privilege.service.js`
    - `findListPrivilegeForPartnerApi` dan `findOnePrivilegeForPartnerApi`, dengan cakupan `type: 'role'`, `isSuper: false`, dan field `privilege_id`, `name`, `comment`.

  - `backend/src/utils/dataTableParamsInput.js` [NEW]
    - `isValidDataTableParams` memeriksa bentuk parameter datatable dari klien:
      - `sorting` dan `columnFilters` wajib array dengan entri yang sah;
      - `globalFilter` wajib string;
      - `columnVisibility` wajib objek;
      - `pageSize`/`pageIndex` wajib angka.
    - Nilai filter teks/angka hanya boleh skalar. Objek di posisi itu akan menjadi operator MongoDB (`{ $ne: null }`, `{ $regex: ... }`) bila diteruskan ke query.
    - Filter `select`, `select-search`, dan `numberRange` wajib array skalar. `dateRange` boleh `null` (dikirim UI saat filter dikosongkan).
    - `omitTableColumns`: membuang urutan/filter pada kolom tertentu tanpa mengubah input.
    - Pemeriksaan ditaruh di endpoint, bukan di dalam `dataTable`, karena `dataTable` juga dipakai controller yang menyusun filter sendiri (mis. `objIdFind` dengan `$in`).

  - `backend/src/utils/flagQuery.js` [NEW]
    - `notTruthyFlag(field)`: klausa "flag ini bukan ya" yang tahan data lama bernilai `'true'`/`1`/`'1'`.
    - Memakai `$expr` karena Mongoose meng-cast nilai query sesuai skema. Akibatnya `{ $ne: true }` maupun `{ $nin: [true, 'true', 1, '1'] }` hanya menyaring boolean `true`, dan akun developer dari data lama bisa lolos.

  - `backend/src/utils/publicRecord.js` [NEW]
    - `toPublicRecord`/`toPublicTable` membuang `_id` (ObjectId internal) sebelum data dikirim ke pihak luar. Id publik yang dipakai adalah `admin_id`/`privilege_id`.

  - `backend/src/routes/partnerApi.route.js`
    - Route `POST ['/partners/list', '/partner/list']` dengan `protectedPartnerApp`.
    - Enam route data internal (beserta alias tunggalnya) dengan rantai `partnerApiAdminOnly` → `checkPrivilege`.
    - Swagger lengkap untuk semua route baru: aturan cakupan, contoh body, skema respons, dan kode 400/401/403/404/500.

  **Backend — pemindai endpoint**

  - `backend/src/utils/routeMounts.js` [NEW]
    - Satu sumber prefix pemasangan per berkas route: bawaan `/api/v1`; `internal.route.js` → `/api/v1/internal`, `mobileCustomer.route.js` → `/m-api/v1`, `partnerApi.route.js` → `/p-api/v1`.

  - `backend/src/utils/privilege-endpoints.js`
    - `apiEndpoints` kamus hak akses kini memakai `routeFilePrefix(file)`, bukan `/api/v1` untuk semua berkas.

  - `backend/src/services/endpointCatalog.service.js`
    - Katalog endpoint AI Agent hanya memindai berkas route yang dipasang di `/api/v1` (`isCatalogRouteFile`). Route p-api yang memakai `checkPrivilege` tidak boleh masuk ke katalog, karena AI Agent memanggil endpoint dengan sesi admin aplikasi utama.

  - `backend/src/config/privilegeDictionary.json` dan `privilege.json`
    - `apiEndpoints` dibangkitkan ulang (`npm run gd`): key `admin.list/read`, `employee.list/read`, dan `privilege.list/read` kini juga mencantumkan endpoint `/p-api/v1/...`.
    - `privilege.json` hanya bergeser urutan key `privilege` (hasil generator).

  - `AGENTS.md`
    - Butir baru di Bab 9: aturan data internal perusahaan di Partner API dan daftar mitra, serta aturan flag boolean data lama (`notTruthyFlag`).
    - Bab 4.11: berkas route baru di luar `/api/v1` wajib didaftarkan di `routeMounts.js`.

  **Test**

  - `backend/test/integration/partnerApiPartner.list.test.js` [NEW], 6 skenario:
    - admin hanya melihat mitra reseller yang tampil di portal dan belum dihapus;
    - respons tidak membawa data sensitif maupun field internal;
    - mitra hanya melihat dirinya sendiri, dan `find`/`withDeleted` di body diabaikan;
    - body datatable salah bentuk ditolak 400 tanpa alert error;
    - body datatable bawaan UI tetap diterima;
    - tanpa token ditolak 401.
  - `backend/test/integration/partnerApiStaff.test.js` [NEW], 11 skenario:
    - daftar administrator hanya berisi admin aktif non-developer, tanpa field sensitif;
    - flag `developer`/`isSuper` data lama (string/angka) tetap tersaring;
    - `sorting` dan `columnFilters` pada `privilege` diabaikan;
    - daftar karyawan hanya berisi karyawan aktif, tanpa field sensitif;
    - daftar hak akses hanya berisi role biasa, tanpa daftar key;
    - detail: 200 di dalam cakupan, 404 generik di luar cakupan;
    - detail: kegagalan database dijawab 500, bukan 404;
    - admin tanpa hak akses yang sesuai ditolak 403;
    - sesi mitra selalu ditolak 403;
    - body salah bentuk 400, tanpa token 401;
    - alias tunggal memakai rantai handler yang sama.
  - `backend/test/integration/flagQuery.test.js` [NEW], 2 skenario: `notTruthyFlag` hanya menyisakan flag yang benar-benar bukan "ya", dan bukti bahwa filter biasa meloloskan nilai string/angka.
  - `backend/test/unit/routeMounts.test.js` [NEW]: prefix setiap berkas route sama dengan pemasangannya di `app.js`.
  - `backend/test/unit/endpointCatalog.test.js`: hanya berkas route yang dipasang di `/api/v1` yang masuk katalog.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #529 | `0a319b8a` resolve #529 | Partner API: daftar mitra (cakupan dari token), endpoint baca-saja administrator/karyawan/hak akses dengan pemeriksaan hak akses dan daftar-putih field, validasi bentuk body datatable (400, bukan 500), dan perbaikan prefix pemindai endpoint |
