# 📝 Daily Work Report - Idham (2026-10-15)

---

## 📅 Laporan Harian - 15 Oktober 2026

---

## 🌿 Branch: `issue-407` — Pelaporan Pelanggan (Penyesuaian)

### 📌 Informasi Issue

- **Nomor Issue**: #407
- **Judul Issue**: Penggantian istilah "Lapor Pajak" menjadi "Pelaporan" pada fitur pelaporan layanan pelanggan
- **Status Branch**: `Belum di-merge` (squash menjadi satu commit `resolve #407`, di-rebase ke `master` terbaru, dan di-push ke `origin/issue-407`)

### 📅 Rincian Commit

#### [a2be3c17] - resolve #407 (squash + rebase) - 15 Oktober 2026, 22:14

- **Ringkasan**:
  - Toggle Pelaporan di tabel Pelanggan untuk pelanggan dengan satu layanan, yang dibuat 13 Oktober, **dihapus kembali**. Kolom Pelaporan di tabel Pelanggan dan Pelanggan Bisnis kini hanya badge ringkasan.
  - Pelaporan tetap diatur dari kartu "Pelaporan" di form edit dan aksi massal tabel.
  - Penggantian istilah "Lapor Pajak" menjadi "Pelaporan" tetap dipertahankan.
  - Pekerjaan hari ini (commit `a8947218`) digabung dengan commit 13 Oktober menjadi satu commit, lalu di-rebase ke `master` terbaru.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/routes/customer.route.js`
    - Route `PATCH /api/v1/customer/tax-report/single` beserta Swagger-nya dihapus.
  - `backend/src/controllers/customer.controller.js`
    - Handler `toggleSingleCustomerTaxReport` dihapus.
  - `backend/src/services/taxReport.service.js`
    - Fungsi `setSingleCustomerTaxReport` dihapus.
    - Ringkasan per pemilik kembali mengembalikan status saja (`all`/`partial`/`none`/`no_service`), tanpa jumlah layanan.
  - `backend/src/services/customer.service.js`
    - Field `tax_report_service_count` dibuang dari datatable Pelanggan karena tidak dibutuhkan lagi.
  - `backend/src/services/business.service.js`
    - Disesuaikan kembali dengan bentuk ringkasan yang berisi status saja.
  - `backend/src/config/privilegeDictionary.json`
    - Endpoint `/customer/tax-report/single` dihapus dari daftar endpoint `customer.changeSensitive`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key `customer.taxReportNotSingle` dihapus.
  - `backend/test/integration/customerTaxReported.test.js`
    - Kelompok test toggle satu layanan dihapus bersama fiturnya.
  - `backend/test/integration/capacitySpeed.controller.test.js`
    - Ditambah komentar `eslint-disable-next-line no-unused-vars` pada variabel yang sengaja dibuang lewat destrukturisasi. Test ini datang dari `master` dan gagal lint setelah rebase.

  **Frontend**

  - `frontend/src/app/pages/users/customer/schema/columns.jsx`
    - Kolom Pelaporan kembali selalu memakai `TaxReportStatusCell` (badge ringkasan).
  - `frontend/src/components/shared/table/rows.jsx`
    - Sel `TaxReportToggleCell` dihapus.

  **Dokumentasi**

  - `AGENTS.md`
    - Pola Bab 9 "Visibilitas di Partner API" diperbarui: kolom tabel Pelanggan dan Pelanggan Bisnis hanya badge ringkasan (`tax_report_status`), bukan toggle.

---

## 🌿 Branch: `issue-412` — IP Pool & IP Statis untuk Akun PPPoE Mitra lewat Partner API

### 📌 Informasi Issue

- **Nomor Issue**: #412
- **Judul Issue**: IP Pool dan IP statis untuk akun PPPoE mitra lewat Partner API
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-412`)

### 📅 Rincian Commit

#### [6264bdbd] - resolve #412 - 15 Oktober 2026, 17:52

- **Ringkasan**:
  - Mitra kini bisa memilih **IP Pool miliknya**, meminta daftar IP kosong, dan membuat atau mengubah akun PPPoE ber-IP statis lewat aplikasi eksternal (p-api), dengan jaminan yang sama seperti jalur internal: IP harus di dalam rentang pool, tidak bentrok, dan reservasinya selalu sinkron.
  - Admin menetapkan mitra pemilik pool dari menu **Network → IPv4**.
  - Aturan "pemilik pool = pemilik akun" juga ditegakkan di jalur internal (Broadband, Radius Non-Pelanggan, Dedicated, Akses Data).
  - Sebelumnya `ip_static` yang dikirim lewat p-api tidak divalidasi. Akibatnya IP bisa ganda dengan pelanggan lain, mitra bisa memakai pool internal, dan reservasi tidak ikut berpindah saat IP diubah.
  - Rencana dan catatan implementasinya ada di `.agent/pppoe-mitra-ip-static_plan.md`.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Model**

  - `backend/src/models/networkIPv4.model.js`
    - Field baru `partner` (ref `Partner`, bawaan `null`) untuk mitra pemilik pool. `null` berarti pool internal.
    - Kepemilikan berlaku per blok dan tidak diwariskan ke sub-blok.
    - Index baru `{ partner, usage }`.
  - `backend/src/models/networkIPv4Used.model.js`
    - Index baru pada `toLong`, untuk mencari IP terpakai berdasarkan rentang alamat di semua blok.

  **Backend — Service & Utilitas**

  - `backend/src/services/networkIPv4.service.js`
    - `findAvailableAddressesInPool(pool, limit)`: menghitung IP kosong dalam pool. Logika ini dipindah dari controller ke service.
    - IP dianggap terpakai berdasarkan rentang alamat di semua blok, bukan hanya blok yang sama. Sebelumnya IP yang tercatat di blok induk atau blok tumpang tindih bisa muncul sebagai "kosong".
    - `findRadiusIPv4SelectForOwner`: daftar pool `usage: 'radius'` milik satu pemilik, dengan pencarian.
    - `findOwnedRadiusPool`: satu pool, hanya bila pemiliknya cocok.
    - Endpoint `select-radius` internal dan `read` blok kini ikut mem-populate `partner`.
  - `backend/src/services/networkIPv4Used.service.js`
    - Pencarian IP terpakai lintas blok berdasarkan rentang `toLong`. `findUsedIpsForRange` (per blok) dihapus karena tidak dipakai lagi.
    - `syncStaticIpReservation` kini menambahkan pool baru ke catatan reservasi saat IP-nya sama tetapi pool-nya berganti.
  - `backend/src/services/radiusAuthentication.service.js`
    - Fungsi baru `findAuthenticationIpPoolId`: membaca `ip_pool` mentah sebuah akun, termasuk akun terhapus. Dipakai untuk mendeteksi apakah pool diganti saat update.
  - `backend/src/services/partner.service.js`
    - `findPartnerSelect` mendapat mode `browse: true` yang menampilkan daftar mitra reseller tanpa kata kunci (maks 1.000, urut nama). Mode ini dipakai pilihan "Mitra Pemilik".
    - Kata kunci kini di-escape sebelum masuk regex, dan `catch` yang sebelumnya diam kini mencatat `logger.error`.
  - `backend/src/utils/staticIpInput.js` [NEW]
    - Validasi bersama `ip_static` terhadap pool:
      - Format IPv4 harus valid.
      - Alamat harus di dalam rentang host pool. Network/broadcast dikecualikan, sedangkan /31 dan /32 menganggap semua alamat host.
    - Rentang dihitung dari `address/cidr`, karena `lastAddressInt` ternyata tersimpan sebagai alamat broadcast.
  - `backend/src/utils/radiusPoolOwner.js` [NEW]
    - Helper `assertRadiusPoolOwner` untuk jalur internal. Pool yang dipilih harus milik pemilik akun (`ref_partner`; kosong berarti internal), kalau tidak dijawab 422 `poolOwnerMismatch`.
    - Saat update, pengecekan hanya berjalan bila pool diganti, supaya akun lama yang sudah memakai pool dengan pemilik berbeda tetap bisa diedit.

  **Backend — Partner API**

  - `backend/src/controllers/partnerApiIPv4.controller.js` [NEW]
    - Endpoint daftar pool milik pemanggil dan daftar IP kosong (maks 255) dalam satu pool.
    - Sesi mitra memakai `req.user._id` sebagai pemilik. Sesi admin menyebut pemilik lewat `partner_id`; tanpa itu yang dipakai pool internal.
    - Pool milik pihak lain dijawab 404.
  - `backend/src/controllers/partnerApiRadius.controller.js`
    - Helper baru `resolveStaticIp` memvalidasi `ip_pool`/`ip_static` secara berurutan:
      - 400 bila IP statis diisi tanpa pool.
      - 400 bila pool tidak ditemukan atau bukan milik pemilik akun.
      - 400 bila format IP salah.
      - 422 bila IP di luar rentang pool.
      - 409 bila IP sudah dipakai.
    - **Create:** reservasi IP dilakukan **sebelum** akun disimpan (`_id` akun dibuat lebih dulu). Bila IP bentrok, akun tidak tercipta. Bila penyimpanan akun gagal, reservasi dilepas lagi.
    - **Update:** validasi hanya berjalan bila field IP disentuh, atau bila pemilik akun berubah dan akun punya pool.
      - Reservasi dipindah sebelum akun diubah.
      - Bila update akun gagal, reservasi dikembalikan ke keadaan sebelumnya; kegagalan pengembalian ini dicatat `logger.error`.
      - `ip_static: null`/`''` berarti mengosongkan dan melepas reservasi.
      - Hasil update yang berisi error kini dijawab 400 (sebelumnya lolos 200).
    - Field `_id`, `ref_partner`, dan `pid` dibuang dari body create/update. Akun yang dipindah ke pelanggan internal meng-`$unset` `ref_partner`/`pid`.
  - `backend/src/routes/partnerApi.route.js`
    - Route baru `POST /p-api/v1/network/ipv4/select` dan `GET /p-api/v1/network/ipv4/available-address/:pool_id`.
    - Swagger kedua route baru dan endpoint create/update akun radius diperbarui dengan aturan IP statis dan kode errornya.

  **Backend — Jalur Internal**

  - `backend/src/controllers/networkIPv4.controller.js`
    - Create/update blok IPv4 menerima `partner`, yang harus mitra reseller (400 bila bukan).
    - Pemilik pool yang rentangnya masih berisi IP statis milik pihak lain tidak bisa diubah atau dikosongkan: 409 `ownerHasReservations`. Tanpa ini, mitra bisa "mewarisi" IP pelanggan internal.
    - `availableIPv4` kini cukup memanggil `findAvailableAddressesInPool`.
  - `backend/src/routes/networkIPv4.route.js`
    - Swagger create/update/read diperbarui dengan field `partner` dan kode error barunya.
  - `backend/src/controllers/radiusAuthentication.controller.js`
    - Form Broadband menegakkan `assertRadiusPoolOwner`. Pemilik pool harus sama dengan pemilik pelanggan.
  - `backend/src/controllers/radiusAuthenticationNonCustomer.controller.js`, `backend/src/controllers/productDedicatedInternet.controller.js`, `backend/src/controllers/productDataAccess.controller.js`
    - Akun non-pelanggan, dedicated internet, dan akses data selalu internal, jadi hanya boleh memakai pool internal. Dicek sebelum ada penulisan, dan saat update hanya bila pool diganti.
  - `backend/src/routes/radiusAuthentication.route.js`, `backend/src/routes/radiusAuthenticationNonCustomer.route.js`, `backend/src/routes/productDataAccess.route.js`, `backend/src/routes/productDedicatedInternet.route.js`, `backend/src/routes/partner.route.js`
    - Swagger ditambah respons 422 `poolOwnerMismatch` dan dirapikan.

  **Backend — i18n**

  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru di `network.ipv4`: `poolRequired`, `addressOutOfRange`, `ownerHasReservations`, `poolOwnerMismatch`, `invalidAddress`.

  **Backend — Test**

  - `backend/test/unit/staticIpInput.test.js` [NEW]
    - Format IPv4 salah; IP network/broadcast; di luar dan di dalam rentang; /31 dan /32.
  - `backend/test/integration/partnerApiIPv4.test.js` [NEW]
    - Mitra hanya melihat pool miliknya; pool internal atau milik mitra lain dijawab 404; daftar IP kosong benar; sesi admin.
  - `backend/test/integration/partnerApiRadius.test.js`
    - **Create:** IP statis tanpa pool 400; pool mitra lain 400; di luar rentang 422; IP terpakai 409 dan akun tidak tercipta; sukses tercatat di `network_ipv4_used`.
    - **Update:** ganti IP memindahkan reservasi; kosongkan IP melepas reservasi; konflik 409 tanpa mengubah akun maupun reservasi lama.
  - `backend/test/integration/networkIPv4Owner.test.js` [NEW]
    - Mitra non-reseller ditolak 400; ganti pemilik pool yang berisi reservasi pihak lain ditolak 409.
  - `backend/test/integration/networkIPv4Used.service.test.js`
    - IP kosong mengecualikan IP yang tercatat di blok lain dalam rentang yang sama.
  - `backend/test/integration/partnerSelect.service.test.js`
    - Mode `browse` pada select mitra.
  - `backend/test/integration/radiusAuthentication.controller.updateIp.test.js`, `backend/test/integration/radiusAuthenticationNonCustomer.controller.test.js`
    - Penegakan aturan pemilik pool di jalur internal.
  - `backend/test/helpers/factories.js`
    - Factory baru `createIPv4Pool`: membuat blok IPv4 `usage: radius` (bawaan pool internal `10.<n>.0.0/24`), bisa diberi `partner` untuk pool milik mitra.

  **Frontend (Network → IPv4)**

  - `frontend/src/app/pages/network/ipv4Management/components/PoolOwnerSelect.jsx` [NEW]
    - Isian "Mitra Pemilik" berbasis `Combobox` dan `/partner/user-select` (mode `browse`).
  - `frontend/src/app/pages/network/ipv4Management/create.jsx`, `frontend/src/app/pages/network/ipv4Management/edit.jsx`
    - Menambah isian Mitra Pemilik untuk blok `usage: radius`.
    - Form edit hanya mengirim `partner` bila diubah, supaya mitra yang sudah terhapus atau bukan reseller tidak menggagalkan simpan atau mengganti pemilik diam-diam.
  - `frontend/src/app/pages/network/ipv4Management/schema/createSchema.js`
    - Field `partner` opsional.
  - `frontend/src/app/pages/network/ipv4Management/index.jsx`
    - Pemilik pool ditampilkan di kartu detail blok; "Internal" bila kosong.
  - `frontend/src/app/pages/network/ipv4Management/utils/poolOwner.js` [NEW], `frontend/src/app/pages/network/ipv4Management/utils/poolOwner.test.js` [NEW]
    - Fungsi murni `toPoolOwnerOption` yang mengubah data mitra pemilik hasil populate menjadi opsi combobox (`partner_id - nama`), atau `null` untuk pool internal, beserta test-nya.
  - `frontend/src/components/shared/form/Combobox.jsx`
    - Prop baru `resetQueryOnClose`: kata kunci dikosongkan saat dropdown ditutup, supaya membukanya lagi menampilkan daftar awal.
  - `frontend/src/components/shared/form/FormInput.jsx`
    - Label pilihan IP Pool di form layanan menambahkan nama mitra pemilik, supaya pool mitra tidak tertukar dengan pool internal.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Label "Mitra Pemilik", "Internal", dan teks bantuan.

  **Dokumentasi**

  - `AGENTS.md`
    - Tanggal pembaruan menjadi 2026-10-15.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #407 | Pelaporan Pelanggan (penyesuaian) | Toggle satu-layanan di tabel dihapus; pelaporan cukup lewat form edit dan aksi massal |
| #412 | IP Pool & IP Statis PPPoE Mitra | Mitra bisa memakai IP statis dari pool miliknya lewat Partner API tanpa risiko IP ganda; aturan pemilik pool berlaku juga di jalur internal |

### Kemampuan Baru Pengguna/Admin

- Admin dapat menetapkan **mitra pemilik** sebuah IP Pool radius dari **Network → IPv4**.
- Aplikasi eksternal mitra dapat mengambil daftar IP Pool miliknya dan IP yang masih kosong, lalu membuat atau mengubah akun PPPoE ber-IP statis.
- Label pilihan IP Pool di form layanan kini menampilkan nama mitra pemilik.

### Bug Fix / Solusi Masalah

- IP statis yang dikirim lewat Partner API kini divalidasi (wajib pool, harus dalam rentang, tidak boleh bentrok). Sebelumnya bisa ganda dengan pelanggan lain.
- Akun PPPoE p-api tidak lagi tercipta bila IP statisnya bentrok. Sebelumnya error reservasi ditelan dan akun tetap dibuat.
- Mengubah IP statis lewat p-api kini memindahkan reservasinya. Sebelumnya catatan lama basi dan IP lama tetap terkunci.
- Mitra tidak bisa lagi memakai IP Pool internal atau milik mitra lain.
- Daftar IP kosong tidak lagi menampilkan IP yang sudah tercatat di blok induk atau blok tumpang tindih.
- Pemilik pool yang masih berisi IP statis pihak lain tidak bisa diubah diam-diam.

### Menu/Fitur Baru

- Isian **Mitra Pemilik** di form tambah/ubah blok **Network → IPv4**.
- Endpoint Partner API `/p-api/v1/network/ipv4/select` dan `/p-api/v1/network/ipv4/available-address/:pool_id`.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur (IP Pool Mitra)**: Setiap IP Pool radius bisa ditandai milik satu mitra. Pool milik mitra hanya bisa dipakai akun PPPoE pelanggan mitra itu, dan pool internal hanya untuk layanan internal. Lewat Partner API, mitra bisa memilih pool miliknya, melihat IP yang kosong, lalu memasang IP statis ke akun PPPoE. Server memastikan IP itu berada di dalam pool, belum dipakai, dan reservasinya ikut berpindah saat IP diubah atau dikosongkan.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Network → IPv4**, lalu tambah atau ubah blok dengan penggunaan **radius**.
  2. Pilih mitra pada isian **Mitra Pemilik**, lalu simpan. Kosongkan bila pool untuk internal.
  3. Dari aplikasi mitra: panggil `POST /p-api/v1/network/ipv4/select` untuk daftar pool, lalu `GET /p-api/v1/network/ipv4/available-address/:pool_id` untuk IP kosong.
  4. Kirim `ip_pool` dan `ip_static` saat membuat atau mengubah akun lewat `/p-api/v1/radius/users/create` atau `/update/:id`.
