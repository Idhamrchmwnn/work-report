# 📝 Daily Work Report - Idham (2026-11-19)

---

## 📅 Laporan Harian - 19 November 2026

---

## 🌿 Branch: `issue-565` — Mobile Frontend Modul D: Paket Broadband (penyelesaian)

### 📌 Informasi Issue

- **Nomor Issue**: #565
- **Judul Issue**: Mobile Frontend: modul Paket Broadband (katalog, paket turunan mitra, permohonan) + perbaikan penulisan paket di Partner API
- **Status Branch**: `Belum di-merge`.
  - Commit `resolve #565` (`c313f6bc`) di atas `master` terbaru (setelah `resolve #571`), sudah ada di `origin/issue-565`.
  - Riwayat sebelum squash disimpan di branch lokal `backup/issue-565-sebelum-squash`.

### 📅 Rincian Commit

#### [6f97f286] - save #565 - 18 November 2026, 16:59 (di-rebase 19 November, 10:26)

- Commit `save #565` kemarin di-rebase ke `master` setelah `resolve #561` di-merge (18 November, 19:13). Isinya sama dengan laporan 18 November.

#### [90b8226d] - save #565 - 19 November 2026, 16:42

- **Ringkasan**:
  - **Bug harga**: isian `180.000` terkirim apa adanya dan dibaca server sebagai **180**.
    - Kini kolom harga memakai `MoneyField`. Tampilan diberi pemisah ribuan, tetapi nilai form selalu digit saja.
    - Skema juga menolak harga bertitik, jadi tidak pernah lolos sebagai angka kecil.
  - **Halaman Paket kini tiga tab**: Katalog Paket, Paket Turunan, dan Pengajuan.
    - Pengajuan sebelumnya berada di dalam tab Paket Turunan. Tautan lama `?tab=derived&view=requests` tetap diarahkan ke tab Pengajuan.

- **Komponen yang Berubah & Penjelasannya**:

  - `mobile-frontend/src/utils/money.js`
    - `toMoneyDigits`: isian pengguna menjadi digit saja (`'180.000'` → `'180000'`).
    - `moneyFromNumber`: nilai server menjadi digit untuk nilai awal form. Fungsi ini tidak lewat `toMoneyDigits`, karena `'180000.00'` akan menjadi 18.000.000.
    - `formatMoneyInput`: tampilan kolom dengan pemisah ribuan.
  - `mobile-frontend/src/components/form/fields.jsx`
    - `MoneyField` [NEW]: memformat saat diketik dan mengembalikan posisi kursor setelah titik ditambah atau dibuang. Jumlah digit dibatasi di bawah `Number.MAX_SAFE_INTEGER`.
  - `mobile-frontend/src/features/products/{packagePayload,requestPayload,productSchema}.js`
    - Harga selalu dikirim digit, dan skema menolak nilai bertitik.
  - `mobile-frontend/src/pages/products/{PackageFormPage,DerivedFormPage}.jsx`
    - Kolom harga memakai `MoneyField`.
  - `mobile-frontend/src/pages/ProductsPage.jsx`
    - Tiga tab, dengan tab dikenali dari `?tab=`. Pindah tab membuang pencarian, filter, dan halaman sebelumnya.
  - `mobile-frontend/src/data/mockData.js`, `i18n/locales/{id,en}/translations.json`
    - Penyesuaian label tab.
  - **Test**:
    - `src/utils/money.test.js` (3): isian bertitik menjadi digit utuh, desimal dari server dibulatkan (bukan digabung), dan pemisah ribuan pada tampilan.
    - `src/features/products/products.test.js` (+1): harga bertitik dikirim sebagai digit utuh.

#### [53842047] - save #565 - 19 November 2026, 22:47

- **Ringkasan**:
  - **Aturan paket turunan kini berlaku juga untuk sesi ADMIN** di Partner API saat membuat atau mengubah akun PPPoE. Sebelumnya hanya sesi mitra yang diperiksa.
    - Akun pelanggan mitra hanya boleh memakai paket turunan milik mitra itu.
    - Profil akun selalu mengikuti paket; profil kiriman klien diabaikan.
    - Akun yang dipindah admin ke pelanggan mitra lain diperiksa ulang paketnya walau paketnya tidak dikirim.
    - Paket yang sudah terhapus ditolak.
    - Akun lama yang masih memakai paket master tetap bisa diedit.
  - **Pilihan paket `select-list` hanya menawarkan paket yang sah**:
    - sesi mitra mendapat paket turunan aktif miliknya;
    - admin dengan `?partner_id` mendapat paket turunan mitra itu (404 bila mitra tak dikenal atau tersembunyi, bukan diam-diam semua paket);
    - admin tanpa `partner_id` mendapat semua paket yang belum terhapus.
  - **Katalog** (kedua peran) hanya menampilkan paket katalog utama yang bisa dijual kembali mitra (`resellable-list`), bukan paket turunan.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/controllers/partnerApiRadius.controller.js`
    - `resolveDerivedOwnerId`: pemilik akun dihitung dari pelanggan baru di body, selain itu dari `ref_partner` akun. `null` berarti akun internal, yang tidak terkena aturan ini.
    - Pemasangan baru: profil harus milik mitra pemilik akun dan berupa profil turunan hasil persetujuan. Sesi mitra mendapat "tidak ditemukan", sedangkan sesi admin mendapat alasan yang jelas.
    - Bila akun pindah pemilik, paket lama diperiksa ulang terhadap pemilik baru dan profilnya diselaraskan. Sesi mitra tidak pernah memindah akun ke pemilik lain.
  - `backend/src/controllers/partnerApiProductBroadband.controller.js`
    - `select-list` per sesi/`partner_id` seperti di atas.
  - `backend/src/routes/partnerApi.route.js`
    - Swagger `select-list` (`partner_id`) dan aturan paket akun.
  - `backend/src/locales/{id,en}/translation.json`
    - Pesan penolakan paket untuk sesi admin.
  - `AGENTS.md` §2.A aturan 37
    - Butir (d) diperluas: aturan berlaku untuk sesi mitra **dan** admin, termasuk perilaku `select-list`.
    - Dicatat juga bahwa web utama (`/api/v1/broadband/*`) belum menegakkan aturan ini untuk admin.
    - Butir (h) baru: `create-derived`, daftar-putih create/update, dan resolusi `radius_profile` sebelum pembanding field terkunci.

  **Mobile Frontend**

  - `mobile-frontend/src/pages/products/components/ParentCatalog.jsx`
    - Hanya paket katalog utama (`resellable-list`) untuk kedua peran.
    - Mitra mendapat tombol "Ajukan Turunan" dan rincian di panel bawah.
    - Admin mendapat tombol "Tambah Paket Turunan" (paket induk sudah terisi), ketuk kartu membuka halaman detail, dan `+` menambah paket katalog.
  - `mobile-frontend/src/pages/radius/AccountFormPage.jsx`
    - Pilihan paket mengikuti pemilik. Pelanggan mitra hanya mendapat paket turunan mitra itu, dengan profil ditentukan server.
    - Bila pemilik berganti, paket dipilih ulang.
  - `mobile-frontend/src/services/radiusApi.js`
    - `selectPackages({ partnerId })`.
  - `mobile-frontend/src/pages/ProductsPage.jsx`, `PRD.md`, `i18n`
    - Penyesuaian katalog dan baris modul D.

  **Test** (+8)

  - `backend/test/integration/partnerApiRadius.test.js` (+6), aturan paket turunan untuk sesi admin:
    - paket master atau turunan mitra lain ditolak untuk pelanggan mitra;
    - paket turunan mitra itu diterima, dan profil mengikuti paket;
    - pelanggan internal tetap boleh paket master dengan profil pilihan admin;
    - paket terhapus ditolak;
    - akun lama berpaket master tetap bisa diedit;
    - pindah pemilik memeriksa ulang paket.
  - `backend/test/integration/partnerApiProductBroadband.test.js` (+2): `select-list` sesi mitra; admin dengan `partner_id`, dan `partner_id` tak dikenal 404.
  - `backend/test/integration/customerTaxReported.test.js`
    - Disesuaikan dengan aturan paket baru.

#### [c313f6bc] - resolve #565 - 19 November 2026, 22:49

- Keempat commit `save #565` di-squash menjadi satu di atas `master` terbaru (`resolve #571`). Isinya sama dengan gabungan commit di atas.

---

## 🌿 Branch: `issue-570` — Hak Akses Admin di Partner API

### 📌 Informasi Issue

- **Nomor Issue**: #570
- **Judul Issue**: Partner API: sesi admin diperiksa hak aksesnya seperti web utama, dan tombol aksi mengikuti hak akses di mobile/partner-frontend
- **Status Branch**: `Belum di-merge`.
  - Commit `resolve #570` (`08dffc31`) di atas `resolve #561`, sudah ada di `origin/issue-570`.
  - Riwayat sebelum squash disimpan di branch lokal `backup/issue-570-sebelum-squash`.

### 📅 Rincian Commit

#### [0287a979] - save #570 - 19 November 2026, 17:09

- **Ringkasan**:
  - **Masalah utama**: route Partner API (selain enam route data internal) tidak memeriksa hak akses sesi admin.
    - Admin yang di web hanya boleh **melihat** pelanggan bisa **mengubah/menghapus** lewat Partner API.
    - Dokumen identitas (KTP) cukup dibuka dengan hak `read`.
  - Kini setiap route Partner API dijaga `partnerApiPrivilege` dengan key yang **sama** dengan route web padanannya.
    - Sesi mitra tetap diteruskan, karena cakupan datanya dibatasi controller.
    - Dropdown yang di web tidak dijaga juga tidak dijaga di sini. Kalau dijaga, form PPPoE macet bagi admin tanpa hak paket.
  - Penolakan berbentuk `403 { code: 'NO_PRIVILEGE' }`, supaya frontend bisa membedakannya dari 403 aturan bisnis.
  - Endpoint baru `GET /p-api/v1/privilege/me` (`{ isSuper, role }`, bentuk sama dengan web).
    - `mobile-frontend` dan `partner-frontend` memakainya untuk menyembunyikan tombol aksi.
    - Halaman, tab, dan kartu **tidak** disembunyikan; penolakan ditampilkan sebagai pemberitahuan tidak punya akses.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/middlewares/partnerApiPrivilege.middleware.js` [NEW]
    - `partnerApiPrivilege(...keys)`:
      - beberapa key berarti cukup salah satu;
      - sesi mitra diteruskan, dan super admin lolos;
      - jenis sesi lain ditolak (fail-closed);
      - kegagalan membaca hak akses diteruskan ke error handler, bukan diloloskan.
    - `sendNoPrivilege` dipakai controller yang memeriksa hak akses sendiri. Daftar key ditandai untuk test "semua route terjaga".
  - `backend/src/utils/has-privilege.js`
    - `isSuperUser` membandingkan secara **ketat**. Dokumen lama bisa menyimpan `super: '0'` (string), yang truthy di JS sehingga salah memberi akses superadmin penuh.
    - `getPrivilegeRoles` menjadi sumber bersama untuk `hasPrivilege` dan `privilege/me`.
  - `backend/src/controllers/partnerApiPrivilege.controller.js` [NEW]
    - `privilege/me`: mitra `{ isSuper: true, role: [] }`.
    - Admin mendapat role dari sumber yang sama dengan `hasPrivilege`. Role tak ditemukan berarti daftar kosong, bukan 404, karena 404 akan menggagalkan login.
    - Isi role tidak di-log.
  - `backend/src/routes/partnerApi.route.js` (+762 baris)
    - Seluruh route diberi `partnerApiPrivilege('<key web>')` setelah `protectedPartnerApp`, beserta Swagger 403 `NO_PRIVILEGE`.
    - Key ditulis sebagai argumen string terpisah, supaya dikenali `npm run gp`/`gd`/`routeScanner`.
  - `backend/src/controllers/partnerApiPartner.controller.js`
    - Profil sendiri tidak digerbangi, karena dipanggil saat login untuk semua sesi. Profil mitra lain (`?partner_id`) butuh `partner.read` (pola §4.11.5).
  - `backend/src/controllers/partnerApiIPv4.controller.js`
    - Pemeriksaan `ipv4.create`/`ipv4.delete` dipindah ke route.
  - `backend/src/utils/{generate-permissions,privilege-endpoints,routeScanner}.js`
    - `partnerApiPrivilege(...)` ikut dipindai. Tanpa itu, key yang pemakaian terakhirnya hanya di Partner API akan lenyap dari `privilege.json`.
  - `backend/src/config/privilegeDictionary.json`, `swaggerGroups.js`
    - `apiEndpoints` diperbarui dengan endpoint Partner API, dan grup Swagger baru.
  - `AGENTS.md` §9
    - Butir baru "Hak akses Partner API".
    - Butir "Data internal perusahaan" disesuaikan.

  **Mobile Frontend**

  - `src/features/privilegeSlice.js` [NEW], `hooks/usePrivilege.js` [NEW], `utils/hasPrivilege.js` [NEW]
    - Hak akses hanya di memori.
    - `checkPrivilege` sama dengan web: super berarti semua akses, dan `a|b` berarti cukup salah satu.
  - `src/features/authSlice.js`, `services/authApi.js`, `services/endpoints.js`, `store.js`
    - Hak akses dimuat saat login dan saat sesi dipulihkan.
    - Galat jaringan saat pemulihan mempertahankan token.
    - Galat lain tidak menggagalkan login: mitra tetap akses penuh, admin tanpa akses.
  - `src/utils/noPrivilege.js` [NEW], `utils/apiClient.js`, `utils/errors.js`, `components/AppBootstrap.jsx`
    - Penolakan `NO_PRIVILEGE` memicu event `p-api:no-privilege`, dan hak akses dimuat ulang supaya tombol yang haknya dicabut ikut hilang.
  - `src/components/ListState.jsx`
    - `ErrorState` menerima `error` dan menampilkan pemberitahuan tidak punya akses untuk `NO_PRIVILEGE`.
  - Halaman Pelanggan (daftar, detail, form, tab Dokumen/Layanan) dan RADIUS (akun, profil, sesi, riwayat, putus sesi)
    - Tombol aksi disembunyikan `useHasPrivilege('<key route P-API>')`.
  - `src/features/customers/customerKinds.js`
    - Key hak akses per jenis pelanggan.
  - `PRD.md`
    - Catatan hak akses terpasang (#570).

  **Partner Frontend**

  - `src/app/contexts/auth/Provider.jsx`
    - Memuat `privilege/me` ke `roleSlice` yang sama dengan web, sehingga komponen bersama (`RowActions`, `OpenLink`, dsb.) ikut menyembunyikan tombol.
    - Muat ulang saat ada penolakan, dengan jeda minimum.
  - `src/components/shared/NoAccessNotice.jsx` [NEW], `components/shared/table/Table.jsx`
    - Tabel bersama menampilkan pemberitahuan tidak punya akses.
  - `src/utils/noPrivilege.js` [NEW], `utils/axios.js`, `middleware/AuthGuard.jsx`, `i18n`

  **Test** (39 skenario)

  - `backend/test/integration/partnerApiPrivilege.test.js` [NEW]
    - Setiap route dijaga atau ada di daftar pengecualian, dan pengecualian tidak basi.
    - Setiap key ada di `privilege.json`, dan pemeriksaan selalu setelah `protectedPartnerApp`.
    - `privilege/me` untuk admin terbatas, super, tanpa role, mitra, dan tanpa token.
    - Hak lihat tidak memberi hak ubah.
    - KTP butuh `readSensitive`.
    - Key alternatif berlaku, dan dropdown tetap bisa dipakai.
  - `backend/test/unit/partnerApiPrivilege.middleware.test.js` [NEW]
    - Mitra, admin ber-key, 403 `NO_PRIVILEGE`, key alternatif, super, flag super string, fail-closed, dan galat diteruskan.
  - `backend/test/helpers/partnerApiRoute.js` [NEW]
    - Pembaca daftar route Partner API untuk test.
  - `backend/test/integration/partnerApiIPv4.test.js`, `partnerApiPartner.profile.test.js`
    - Disesuaikan.
  - `mobile-frontend/src/features/privilegeSlice.test.js` [NEW]
    - `checkPrivilege`, normalisasi, deteksi `NO_PRIVILEGE`, pemulihan/login/keluar, dan galat jaringan mempertahankan token.
  - `partner-frontend/src/utils/noPrivilege.test.js` [NEW]
    - Body yang diratakan interceptor, error axios utuh, dan 403 bisnis tanpa kode.

#### [c57c050a] - save #570 - 19 November 2026, 17:46

- **Ringkasan**:
  - **Pemilih tanpa hak akses daftar**, seperti di web:
    - `/customers/user-select`, `/customer-partners/user-select`, dan `/partners/user-select` memakai handler daftar yang sama (cakupan sesi sama).
    - Field-nya sempit dan sekaligus menjadi whitelist pencarian/filter/urutan, sehingga endpoint tanpa key tidak bisa dipakai menebak telepon/email.
  - **Layanan pelanggan bisnis disaring per jenis**:
    - Route menerima salah satu key `dedicatedInternet`/`dataAccess`/`radiusNonCustomer`, lalu controller menyaring daftar hanya ke jenis yang key `.list`-nya dimiliki.
    - Detail butuh `.read` jenis itu dan diperiksa **setelah** visibilitas, jadi layanan yang tidak terlihat tetap 404.
  - Enam route data internal: sesi mitra kini ditolak 403 berkode `NO_PRIVILEGE`, dan gerbangnya memakai `partnerApiPrivilege`.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/controllers/partnerApiCustomer.controller.js`, `partnerApiPartner.controller.js`
    - Handler `user-select` dengan `CUSTOMER_PARTNER_API_SELECT_FIELDS` / `PARTNER_API_SELECT_FIELDS`.
  - `backend/src/controllers/partnerApiBusiness.controller.js`
    - `resolveAllowedServiceTypes` untuk daftar, dan pemeriksaan `.read` per jenis untuk detail.
  - `backend/src/services/customer.service.js`, `partner.service.js`
    - Dukungan field select. Dokumen terhapus tidak pernah ikut, termasuk lewat mode kiriman klien.
  - `backend/src/middlewares/partnerApiAdminOnly.middleware.js`
    - Penolakan sesi mitra berbentuk `NO_PRIVILEGE`.
  - `backend/src/routes/partnerApi.route.js`
    - Route `user-select` dan gerbang data internal memakai `partnerApiPrivilege`.
  - `backend/src/config/privilege.json`
    - Hasil regenerasi `npm run gp` (hanya urutan key `privilege`).
  - `AGENTS.md` §9
    - Butir "Hak akses Partner API" dan "Data internal perusahaan" ditambah aturan pemilih dan penyaringan layanan bisnis.

  **Mobile Frontend**

  - `src/components/{CustomerPicker,PartnerPicker}.jsx`, `services/{customerApi,partnerApi,endpoints}.js`
    - Pemilih memakai endpoint `user-select`.
  - `src/pages/customers/components/CustomerDocumentsTab.jsx`
    - Melihat daftar dokumen (KTP) butuh `.readSensitive`, sedangkan unggah cukup `.update` (sama dengan web). Tanpa hak lihat, unggah tetap tersedia di bawah pemberitahuan tidak punya akses.
  - `src/features/privilegeSlice.js`
    - Admin tanpa akses diberi tahu sekali bahwa tombol aksi disembunyikan.

  **Partner Frontend**

  - `src/app/pages/customers/{customer,business}/view.jsx`
    - Penolakan hak akses menampilkan pemberitahuan di halaman (tidak dialihkan ke daftar), dan tidak dianggap "tidak ditemukan".
  - `src/utils/axios.js`, `i18n`

  **Test** (+9)

  - `backend/test/integration/partnerApiPrivilege.test.js`:
    - layanan bisnis disaring per key `.list`, dan detail butuh `.read` jenisnya;
    - layanan tak terlihat tetap 404;
    - mitra melihat semua jenis miliknya;
    - data internal: admin tanpa key dan mitra sama-sama `NO_PRIVILEGE`;
    - pemilih dengan field terbatas;
    - pencarian pemilih tidak bisa menebak telepon/email;
    - cakupan pemilih pelanggan mitra.
  - `backend/test/integration/businessTaxReported.test.js`, `mobile-frontend/src/features/privilegeSlice.test.js`
    - Disesuaikan atau ditambah.

#### [08dffc31] - resolve #570 - 19 November 2026, 17:47

- Kedua commit `save #570` di-squash menjadi satu di atas `resolve #561`. Isinya sama dengan gabungan commit di atas.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #565 | `90b8226d`, `53842047` save; `c313f6bc` resolve #565 | Bug harga `180.000` → 180 diperbaiki (`MoneyField`), Paket jadi tiga tab, aturan paket turunan di Partner API kini berlaku untuk sesi admin, `select-list` hanya paket sah; +12 test |
| #570 | `0287a979`, `c57c050a` save; `08dffc31` resolve #570 | Hak akses sesi admin di seluruh route Partner API (`partnerApiPrivilege`, 403 `NO_PRIVILEGE`, `privilege/me`), pemilih `user-select` berfield sempit, layanan bisnis disaring per jenis, tombol aksi mengikuti hak akses di `mobile-frontend` dan `partner-frontend`; +48 test |

> Catatan merge: `resolve #561` (dikerjakan 18 November) sudah di-merge ke `master` pada 18 November, 19:13.
>
> Catatan branch: `issue-565` dan `issue-570` sama-sama mengubah `backend/src/routes/partnerApi.route.js`. Simulasi merge (`git merge-tree`) menunjukkan konflik di berkas itu, jadi branch yang di-merge kedua perlu di-rebase lebih dulu. Selain itu, `issue-570` masih di atas `resolve #561`, belum di atas `resolve #571`.
