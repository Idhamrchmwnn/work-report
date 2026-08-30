# 📝 Daily Work Report - Idham (2026-08-30) 

---

## 📅 Laporan Harian - 30 Agustus 2026

---

## 🌿 Branch: `issue-252` — Tambah Data Provinsi & Kabupaten/Kota ke Pelanggan

### 📌 Informasi Issue

- **Nomor Issue**: #252
- **Judul Issue**: Menambahkan data kabupaten/kota (+ provinsi, cascading) ke entitas Pelanggan
- **Status Branch**: `Belum di-commit` — pekerjaan hari ini sengaja dipecah jadi 2 sesi kerja sebelum commit final. Laporan ini mencakup **Sesi 1**: penyiapan data referensi wilayah, endpoint select, ekstensi komponen shared, dan penerapan penuh ke 3 dari 4 entitas (Customer, Business, Partner). Sesi 2 (Prospect, alur konversi, finalisasi) dilaporkan terpisah di `idham_2026-08-30-sesi2.md`.

### 🧭 Latar Belakang & Analisis Awal

Permintaan awal: menambahkan data "kabupaten/kota" ke Pelanggan. Analisis mendalam terhadap codebase menunjukkan bahwa istilah "Pelanggan" sebenarnya terpecah jadi beberapa entitas berbeda yang berbagi pola field lokasi: **Customer** (residensial, `customer.model.js`), **Business** & **Partner/Mitra** (satu model `partner.model.js`, dibedakan flag `reseller`), dan **Prospect** (lead sebelum jadi pelanggan). Semua entitas ini sudah punya field `city` (hasil auto-geocode Mapbox dari koordinat, BUKAN input manual) dan `area` (cakupan wilayah internal POP, freetext string, dipilih via distinct-aggregate select `findAreaSelect`).

Tidak ditemukan data referensi wilayah administratif Indonesia (provinsi/kabupaten/kota resmi) di project ini sama sekali — baik sebagai koleksi Mongoose, package npm, maupun file data statis. Pola yang paling mirip kebutuhan (koleksi referensi mandiri + endpoint select ajax `$regex`, BUKAN distinct-aggregate seperti `area`) ditemukan di modul **`warehouseType`** — dijadikan cetakan untuk desain Provinsi/Kabupaten-Kota.

Keputusan desain yang dikonfirmasi ke user sebelum implementasi:
1. Field disimpan sebagai **ObjectId reference** ke koleksi referensi baru (bukan string snapshot seperti `area`).
2. UX **cascading** — pilih Provinsi dulu, baru Kabupaten/Kota terfilter sesuai.
3. Sumber data: dataset publik `emsifa/api-wilayah-indonesia` (kode resmi Kemendagri), diverifikasi manual via `curl`/`gh api` sebelum dipakai — 34 provinsi, 514 kabupaten/kota di 34 file per-provinsi.
4. **Tidak membangun CRUD admin** untuk mengelola master data Provinsi/Kabupaten-Kota — data ini resmi & stabil, cukup di-seed sekali lewat script, dipakai read-only lewat endpoint select. Keputusan ini menghindari over-engineering (privilege baru, halaman admin baru) yang tidak diminta siapa pun.
5. Integrasi sekunder (variabel template WhatsApp Broadcast, kontrak PKS, dokumen PO/SO pelanggan) **sengaja di luar cakupan** — fokus hanya pada data inti: model, form, tabel, halaman detail.

---

### 🏗️ Komponen yang Dibuat/Diubah

#### A. Data Referensi & Infrastruktur Backend Baru

- **`backend/src/models/province.model.js`** [NEW] — Schema `{code, name}`, `code` unique (kode Kemendagri, mis. `"31"` untuk DKI Jakarta), plugin `mongoose-delete` untuk soft-delete, mirror persis konvensi `warehouseType.model.js`.
- **`backend/src/models/regency.model.js`** [NEW] — Schema `{code, name, province: ObjectId ref Province}`, index gabungan `{province: 1, name: 1}` (wajib, karena endpoint select selalu query dengan filter ini bersamaan).
- **`backend/src/data/provinceSeed.json`** [NEW] — 34 entri `{code, name}`, hasil `curl` ke `https://raw.githubusercontent.com/emsifa/api-wilayah-indonesia/master/static/api/provinces.json`.
- **`backend/src/data/regencySeed.json`** [NEW] — 514 entri `{code, province_code, name}`, digabung dari 34 file `regencies/{province_id}.json` (satu file per provinsi), dinormalisasi via script Node satu-kali di scratchpad sebelum di-commit ke repo.
- **`backend/scripts/seedRegion.js`** [NEW] — Script seed idempoten (mirror gaya `backend/scripts/migrate-fiber-cable-nodes.js`), koneksi mandiri ke MongoDB via `mongoose.connect()`, upsert Province dulu (bangun `Map<code, ObjectId>`), lalu upsert Regency memakai hasil map. **Dijalankan & diverifikasi di MongoDB lokal**: 34 provinsi + 514 kabupaten/kota tersimpan benar (`provinces count: 34`, `regencies count: 514` dikonfirmasi via `mongosh`).
- **`backend/src/services/region.service.js`** [NEW] — `findProvinceSelect(params)` & `findRegencySelect(params)`: aggregate `$match` regex `name` (+ filter `province` untuk regency) → `$project {id: $toString($_id), name}` → `$limit`. Pola ini SAMA seperti `findWarehouseTypeSelect`, BUKAN distinct-aggregate seperti `findAreaSelect` — perbedaan pola ini penting karena keduanya sekilas mirip tapi punya semantik berbeda (referensi mandiri vs freetext yang sudah pernah diketik user).
- **`backend/src/controllers/region.controller.js`** [NEW] — `provinceSelect`, `regencySelect`, keduanya `res.status(200).json(list)` array mentah tanpa wrapper `{success,data}` — mengikuti gaya `areaSelect`/`warehouseTypeSelect*` (kategori route yang memang tidak pakai amplop standar di codebase ini).
- **`backend/src/routes/region.route.js`** [NEW] — `POST /province/select`, `POST /regency/select`, keduanya **hanya** `protectedAdmin` tanpa `checkPrivilege` tambahan — deviasi yang dijustifikasi eksplisit di `REGION_PLAN.md` §2.1 karena mengikuti preseden identik (`/customer/area-select`, seluruh `/warehouse/type/select*`), bukan kelalaian.
- **`backend/src/app.js`** — mendaftarkan `RegionRoute` di `app.use('/api/v1', RegionRoute)`, ditaruh berdekatan dengan `WarehouseTypeRoute`.
- **`backend/src/utils/validate-region.js`** [NEW] — `validateRegencyProvince(regencyId, provinceId, t)`: fetch `Regency.findById`, bandingkan `.province` dengan `provinceId` yang dikirim, tolak (`throw new Error(t('global.regencyProvinceMismatch'))`) kalau tidak cocok. Dibuat sebagai util bersama (bukan diduplikasi 4x) karena dipakai di Customer, Business, Partner, dan Prospect — mencegah data korup dari request yang melewati frontend (mis. panggilan API langsung).
- **`backend/src/locales/{id,en}/translation.json`** — key baru `global.regencyProvinceMismatch` di kedua bahasa.

#### B. Ekstensi Komponen Shared Frontend

- **`frontend/src/components/shared/form/Combobox.jsx`** — Ditambah prop opsional `extraBody` yang di-merge ke body POST (`axios.post(ajaxUrl, {query, ...extraBody})`), dipakai untuk mengirim `province` terpilih saat query kabupaten/kota. Komponen ini dipakai di **107 file lain** di seluruh aplikasi — perubahan dirancang backward-compatible (default kosong), tapi lihat bagian "Bug Ditemukan" di bawah karena implementasi awal defaultnya sempat jadi sumber regresi.

#### C. Penerapan ke Entitas Customer

- **`backend/src/models/customer.model.js`** — tambah field `province`/`regency` (`ObjectId ref Province/Regency`), ditaruh berdekatan dengan `city`/`area` existing, **tidak `required`** (rollout progresif, data lama tanpa provinsi/kabupaten tetap valid).
- **`backend/src/services/customer.service.js`** — `province`/`regency` ditambahkan ke **dua** whitelist proyeksi (`CUSTOMER_NON_SENSITIVE` dan `CUSTOMER_PARTNER_API_FIELDS`, yang terakhir untuk endpoint eksternal `/p-api/v1/customers`), plus `.populate('province','name code').populate('regency','name code')` di `findOneCustomerWithDeleted` (dipakai edit & profile) dan `serverFind.populate` di `findListCustomerForTable` (dipakai tabel list).
- **`backend/src/services/customerPartner.service.js`** — daftar `CUSTOMER_NON_SENSITIVE` **terpisah/duplikat** dari file di atas (sub-panel "pelanggan milik mitra") — mudah kelewat kalau tidak dicek eksplisit, sudah diupdate juga (field + populate).
- **`backend/src/controllers/customer.controller.js`** — `province`/`regency` ditambahkan ke pick-list `createCustomer` (via spread `{...req.body}`, otomatis lolos, tapi validasi tetap dipasang), `updateCustomer`, dan `updateBatchCustomer` (2 pick-list terpisah untuk update tunggal vs bulk-edit) — plus panggilan `validateRegencyProvince` di ketiga titik create/update/batch-update.
- **`backend/src/controllers/customerPartner.controller.js`** — pola sama persis (create + 2 pick-list update terpisah + validasi), untuk sub-panel mitra.
- **`backend/src/routes/customer.route.js`, `customerPartner.route.js`** — swagger docs properti `province`/`regency` ditambahkan di 3 blok request body per file (create/update/batch).
- **Frontend** `frontend/src/app/pages/users/customer/{create,edit,editBatch}.jsx` — dua `Combobox` baru (Provinsi → Kabupaten/Kota) ditaruh setelah field `area` existing, dengan `useController` untuk `province`/`regency`, `watch('province')` + `useMemo` untuk `extraBody` stabil, reset `regency` + `key` remount saat provinsi diganti. **Tidak ada mekanisme prefill** untuk field ini di `edit.jsx`/`editBatch.jsx` — dikonfirmasi dulu bahwa field referensi lain (`area`, `partner`, `tech_support`, `pay_support`) di form yang sama JUGA tidak di-prefill (submit bersifat PATCH parsial), jadi province/regency mengikuti pola existing demi konsistensi, bukan dibuatkan pengecualian.
- **`schema/createShema.js`** — `province` nullable, `regency` nullable dengan `.test()` cross-validation "regency wajib provinsi dulu".
- **`schema/columns.jsx`** (customer & customerPartner) — 2 kolom baru `province`/`regency`, `cell: ({getValue}) => getValue() || '-'` (field lokasi belum pernah jadi kolom tabel sebelumnya, jadi ini penambahan murni, bukan modifikasi kolom lama).
- **`profile.jsx`** — halaman detail merender field secara dinamis via `Object.keys(customerData).map()`; ditambahkan branch eksplisit `key === 'province' || key === 'regency' ? value?.name || '-' : ...` sebelum fallback, supaya render nama hasil populate, bukan silent-empty atau `[object Object]`.
- **i18n** `id`/`en` `translations.json` — key `form.province`, `form.regency`, `data.province`, `data.regency` ditambahkan di 2 titik masing-masing bahasa (namespace `form` dan `data`), lokasinya digrep ulang dari posisi key `area`/`city` existing.

#### D. Penerapan ke Entitas Business & Partner/Mitra

Business dan Partner/Mitra berbagi satu model (`partner.model.js`, dibedakan flag `reseller`), tapi punya service & controller terpisah masing-masing.

- **`backend/src/models/partner.model.js`** — field `province`/`regency` ditambah sekali, otomatis berlaku untuk Business maupun Partner.
- **`backend/src/services/business.service.js`** — `BUSINESS_NON_SENSITIVE` + populate di `findOneBusinessWithDeleted` & `findListBusinessForTable`.
- **`backend/src/services/partner.service.js`** — `PARTNER_NON_SENSITIVE` + populate di `findOnePartnerWithDeleted` & `findListPartnerForTable`.
- **`backend/src/controllers/business.controller.js`**, **`partner.controller.js`** — pola identik dengan Customer: create (spread + validasi), 2 pick-list update/batch-update + validasi di masing-masing.
- **`backend/src/routes/business.route.js`, `partner.route.js`** — swagger docs, 3 blok per file.
- **Frontend**: `business/{create,edit,editBatch}.jsx`, `partner/{create,edit,editBatch}.jsx` — pola cascading Combobox identik dengan Customer (termasuk tanpa-prefill yang sama, dikonfirmasi ulang di masing-masing file bahwa field referensi lain juga tidak prefill).
- **`schema/createShema.js`**, **`schema/columns.jsx`** — identik polanya dengan Customer.
- **`profile.jsx`** (business & partner) — **catatan penting**: berbeda dari `customer/profile.jsx`, versi Business/Partner **tidak punya** fallback `typeof value === 'object' ? null : value` sama sekali sebelum perubahan ini — kalau branch eksplisit untuk `province`/`regency` tidak ditambahkan, React akan **crash** ("Objects are not valid as a React child") saat mencoba merender objek hasil populate secara langsung. Ditambahkan branch eksplisit SEKALIGUS fallback `typeof value === 'object' ? null` sebagai pengaman, karena field baru ini yang pertama kali membawa risiko itu ke file tersebut.

---

### 🐛 Bug Ditemukan & Diperbaiki Sesi Ini

#### 1. 🔴 Regresi glitch di `Combobox.jsx` — berdampak ke 107 file, bukan cuma fitur ini

**Gejala**: setelah field Provinsi/Kabupaten-Kota diimplementasikan, user melaporkan "sering glitch" saat memilih provinsi — dropdown flicker/loading berulang.

**Root cause**: penambahan prop `extraBody` ditulis dengan default `extraBody = {}` di destructuring parameter komponen. Di JavaScript, literal objek sebagai default parameter **dievaluasi ulang setiap kali fungsi komponen dipanggil** (setiap render React), sehingga menghasilkan objek baru secara identitas walau isinya sama-sama kosong. Objek baru ini masuk ke dependency array `useEffect` yang menjalankan pencarian ajax (`}, [ajaxUrl, options, query, displayField, extraBody]);`), sehingga effect itu **selalu dianggap berubah** dan dijalankan ulang setiap kali parent re-render — bukan cuma saat user benar-benar mengetik query baru.

**Dampak diverifikasi**: `grep -rl "components/shared/form/Combobox'" frontend/src | wc -l` → **107 file** memakai komponen ini. Karena default `{}` berlaku untuk SEMUA pemakai yang tidak eksplisit mengoper `extraBody` (termasuk Combobox Provinsi milik fitur ini sendiri, yang memang tidak butuh `extraBody`), regresi ini berpotensi memengaruhi hampir seluruh dropdown ajax-select di aplikasi, bukan cuma yang baru ditambahkan.

**Perbaikan**: ganti default jadi referensi stabil di luar komponen —
```js
const EMPTY_EXTRA_BODY = {}; // dibuat sekali saat modul dimuat, bukan tiap render
...
extraBody = EMPTY_EXTRA_BODY,
```
Untuk Combobox Kabupaten/Kota yang MEMANG butuh `extraBody` dinamis (`{province: provinceId}`), tetap dibungkus `useMemo(() => ({province: provinceId}), [provinceId])` di sisi pemanggil supaya referensinya cuma berubah saat `provinceId` benar-benar berubah.

**Pelajaran** (dicatat di `REGION_PLAN.md`): default parameter berupa objek/array literal di komponen React **wajib** pakai referensi stabil (module-level constant atau `useMemo` di pemanggil) kalau dipakai di dependency array hook manapun — literal langsung di destructuring adalah jebakan identitas-baru-tiap-render yang mudah terlewat saat review.

#### 2. 🔴 Konfigurasi `.env` lokal — MongoDB gagal konek diam-diam

**Gejala**: setelah `.env` diubah untuk memakai MongoDB lokal (blok `[B]`), dropdown Provinsi/Kabupaten-Kota masih "tidak bisa menampilkan list" — awalnya diduga data belum ter-seed.

**Diagnosis**: dicek langsung dengan `mongosh` — MongoDB lokal (`127.0.0.1:27017`) berhasil diakses **tanpa kredensial** (`mongosh --quiet dekasimal` sukses), tapi `.env` blok `[B]` masih menyertakan `MONGODB_USERNAME=adminUser` & `MONGODB_PASSWORD=...` (nilai yang sama dipakai untuk server tim). `backend/src/config/database.js` membangun connection string dengan SCRAM auth **kalau kedua variabel itu terisi** (`if (dbUser && dbPass) {...}`) — terhadap MongoDB lokal yang tidak punya user itu, koneksi gagal login. Dikonfirmasi empiris: `mongosh "mongodb://adminUser:...@127.0.0.1:27017/..."` → `MongoServerError: Authentication failed`.

Kegagalan ini **tidak terlihat sebagai error** di UI — `Combobox.jsx` punya `catch (err) { console.error(...); setData([]); }`, jadi request yang gagal (baik karena data kosong maupun DB tidak konek) sama-sama tampil sebagai "list kosong" ke user, membuat root cause sebelumnya (data belum di-seed) dan root cause ini (auth gagal) terlihat identik dari sisi UI.

**Perbaikan**: baris `MONGODB_USERNAME`/`MONGODB_PASSWORD` di blok `[B]` `.env` dikomentari — `database.js` otomatis fallback ke connection string tanpa auth. Diverifikasi: server backend restart, log menunjukkan `🔥 Connected to database => dekasimal` ke `127.0.0.1`, endpoint `/province/select` merespons 401 (butuh token, bukan 404/500) — koneksi sehat.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Penjelasan Fitur**: Provinsi & Kabupaten/Kota adalah field referensi terstruktur baru (bukan teks bebas seperti `city`/`area` lama yang tetap dipertahankan berdampingan untuk kompatibilitas data historis), bersumber dari data resmi Kemendagri (34 provinsi, 514 kabupaten/kota).
- **Langkah Penggunaan (Tutorial, berlaku untuk Customer/Business/Partner)**:
  1. Buka form tambah/edit Customer, Business, atau Partner.
  2. Cari & pilih **Provinsi** (ketik nama provinsi untuk mempersempit hasil pencarian).
  3. Dropdown **Kabupaten/Kota** otomatis aktif dan hanya menampilkan pilihan dari provinsi yang dipilih.
  4. Kalau provinsi diganti setelah kabupaten/kota terpilih, pilihan kabupaten/kota lama otomatis ter-reset (mencegah kombinasi data yang tidak nyambung).
  5. Simpan form — provinsi/kabupaten akan tampil di kolom tabel list dan baris info halaman detail.
