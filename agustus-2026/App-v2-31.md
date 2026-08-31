# 📝 Daily Work Report - Idham (2026-08-31)

---

## 📅 Laporan Harian - 31 Agustus 2026

---

## 🌿 Branch: `issue-252` — Tambah Data Provinsi & Kabupaten/Kota ke Pelanggan

### 📌 Informasi Issue

- **Nomor Issue**: #252
- **Judul Issue**: Menambahkan data kabupaten/kota (+ provinsi, cascading) ke entitas Pelanggan
- **Status Branch**: issue-252`, sudah di-push ke origin

### 🏗️ Komponen yang Dibuat/Diubah

#### A. Backend — Model, Service, Controller Prospect

- **`backend/src/models/prospect.model.js`** — field `province`/`regency` (`ObjectId ref Province/Regency`) ditambah berdekatan dengan `address`/`city` existing.
- **`backend/src/services/prospect.service.js`**:
  - `PROSPECT_FIELDS` (whitelist proyeksi untuk list/read) — tambah `province`, `regency`.
  - `PROSPECT_EDITABLE_FIELDS` (whitelist create+update sekaligus) — tambah `province`, `regency`. Karena file ini cuma satu whitelist (bukan 2 seperti entitas lain), field otomatis berlaku untuk `createProspect` maupun `updateProspect` tanpa perlu sentuh 2 titik terpisah.
  - `findProspectById` — ditambah `.populate('province', 'name code')` & `.populate('regency', 'name code')` di **kedua** jalur pencarian (`prospect_id` dan fallback `_id` kalau bukan business-id yang valid) — kedua jalur ini eksis karena fungsi ini menerima baik business-id publik maupun ObjectId internal.
  - `findAllProspectsForTable` — `serverFind.populate` ditambah untuk `province`/`regency` (dipakai tabel list `getProspectColumns`).
- **`backend/src/controllers/prospect.controller.js`**:
  - `createProspect` — validasi `validateRegencyProvince(data.regency, data.province, req.t)` ditambahkan setelah `pick(req.body, PROSPECT_EDITABLE_FIELDS)`.
  - `updateProspect` — validasi yang sama ditambahkan.
  - **`convertProspect`** — lihat bagian bug di bawah, ini titik dengan perubahan paling signifikan di sesi ini.

#### B. Frontend — Form & Detail Prospect

- **`frontend/src/app/pages/services/prospect/create.jsx`** (`ProspectCreateDrawer`) — drawer cepat, ditambah 2 `Combobox` (Provinsi → Kabupaten/Kota) di grid form, memakai `useController`+`watch`+`useMemo` pola sama seperti entitas lain.
- **`frontend/src/app/pages/services/prospect/edit.jsx`** (`ProspectEditDrawer`) — **satu-satunya form di antara 4 entitas yang benar-benar prefill penuh**: `reset({...})` ditambah `province: prospect.province?._id || ''` dan `regency: prospect.regency?._id || ''`, PLUS Combobox diberi `defaultValue` eksplisit dari `prospect.province`/`prospect.regency` (bentuk `{id, name}`) supaya tampilan dropdown langsung menunjukkan nilai tersimpan saat drawer dibuka — bukan cuma value di form-state react-hook-form, tapi juga representasi visualnya di komponen Combobox itu sendiri (dua hal yang terpisah karena `Combobox.jsx` punya `internalValue` sendiri untuk ditampilkan).
- **`frontend/src/app/pages/services/prospect/detail.jsx`** — baris `infoRows` ditambah 2 entri: `{label: t('data.province'), value: prospect.province?.name}` dan padanan `regency`. Struktur render `infoRows.map()` di file ini sudah punya fallback `value || '-'` bawaan, jadi tidak perlu penanganan tambahan seperti di `profile.jsx` entitas lain.
- **`frontend/src/app/pages/services/prospect/schema/prospectSchema.js`** — **satu file schema shared** untuk create+edit (bukan 2 file terpisah) — dikonfirmasi ini konsisten dengan `work-report/CODING_RULES.md` Rule 15 (default satu file shared, split cuma kalau field create/edit genuinely beda; field Prospect identik di create & edit, jadi satu file sudah benar). Ditambah `.test('regency-requires-province', ...)` cross-validation; `province` sendiri tidak perlu entry eksplisit karena `city`/`address` yang sudah ada pun tidak divalidasi eksplisit di file ini (pola minimal existing, `this.parent.province` tetap bisa diakses dari context Yup walau tidak dideklarasikan di shape).
- **`frontend/src/app/pages/services/prospect/schema/columns.jsx`** — 2 kolom baru, memakai `cell: NormalTextCell` (komponen shared yang **sudah diimpor** di file ini untuk kolom lain) alih-alih menulis ulang cell renderer inline — pola reuse yang lebih tepat dibanding entitas lain karena komponen ini kebetulan sudah tersedia di file yang sama.

---

### 🐛 Bug Ditemukan & Diperbaiki: Kebocoran Mapping Prospect → Customer

**Alur data lengkap yang ditelusuri**: **Registrasi publik** (`frontend/src/app/pages/public/registration.jsx`, tanpa login) → admin klik "Jadikan Prospek" di halaman review → `ProspectCreateDrawer` (`prospect/create.jsx`) dengan data prefill dari registrasi → `POST /prospect/create` → Prospect tersimpan → admin klik "Convert" di `prospect/convert.jsx` (butuh SO signed lebih dulu) → `POST /prospect/convert/:id` → `convertProspect` di backend → Partner (Customer) baru tercipta.

**Temuan #1 (di luar keputusan scope, tidak diperbaiki)**: form registrasi publik memang tidak punya field kota/provinsi/kabupaten sama sekali — `city` di sana didapat OTOMATIS dari reverse-geocoding koordinat (Mapbox), bukan input manual, jadi arsitekturnya berbeda total dari dropdown ObjectId yang dipakai fitur ini. Menambahkan dropdown ke situ butuh endpoint select **publik tanpa auth** baru (`/province/select`/`/regency/select` saat ini di-gate `protectedAdmin`) — user memutuskan **tidak** memperluas ke sini, cukup menutup kebocoran mapping yang sudah ada.

**Temuan #2 (bug nyata, diperbaiki)**: pada titik konversi Prospect → Customer, ditemukan **dua lapis kebocoran independen**:

1. `backend/src/controllers/prospect.controller.js`, konstanta `PARTNER_CONVERT_FIELDS` (whitelist field yang boleh diisi dari form konversi) **tidak menyertakan `province`/`regency`** — padahal model `Partner` sudah punya kedua field itu, dan model `Prospect` juga sudah punya (bisa terisi manual saat admin membuat Prospect via `create.jsx`).
2. `frontend/src/app/pages/services/prospect/convert.jsx` **sama sekali tidak punya field Provinsi/Kabupaten-Kota** — tidak ada Combobox untuk itu, beda dari `create.jsx`/`edit.jsx` yang sudah punya sebelumnya.

Akibatnya: walau admin sudah rajin mengisi Provinsi/Kabupaten-Kota manual saat membuat/mengedit Prospect, begitu Prospect itu dikonversi jadi Customer (Partner) via tombol Convert, **data lokasi terstruktur itu hilang total** — Customer hasil konversi selalu punya `province`/`regency` kosong, tanpa terkecuali. Ini bug yang murni ditemukan lewat audit alur data ujung-ke-ujung, bukan dari laporan gagal fungsi.

**Perbaikan**:

- `backend/src/controllers/prospect.controller.js`:
  ```js
  const PARTNER_CONVERT_FIELDS = [
    'type', 'name', 'phone', 'email', 'address', 'city',
    'province', 'regency',              // ← ditambahkan
    'coordinate', 'npwp', 'ktp', 'area', 'notes',
  ];
  ```
  Di `convertProspect`, ditambahkan prefill fallback dari data Prospect (mengikuti pola field lain di fungsi yang sama) dan validasi silang:
  ```js
  partnerData.province = partnerData.province || prospect.province?._id;
  partnerData.regency = partnerData.regency || prospect.regency?._id;
  ...
  if (partnerData.regency) {
    await validateRegencyProvince(partnerData.regency, partnerData.province, req.t);
  }
  ```
  Dikonfirmasi juga bahwa `convertProspectToCustomer` (service) memanggil `new Partner(partnerData).save()` langsung tanpa whitelist tambahan di lapisan itu — jadi perbaikan di controller sudah cukup, tidak ada lapisan tersembunyi lain yang perlu disentuh.

- `frontend/src/app/pages/services/prospect/convert.jsx`:
  - Ditambah 2 `Combobox` cascading (pola identik file lain), DENGAN prefill `defaultValue` dari `prospect.province`/`prospect.regency` — form ini juga di-`reset()` dengan `province: prospect.province?._id || ''` dan `regency: prospect.regency?._id || ''`, jadi kalau Prospect sudah punya data lokasi, itu langsung terlihat & ikut terkirim ulang saat konversi (bukan cuma "tidak hilang", tapi juga ditampilkan ke admin untuk dikonfirmasi/diubah kalau perlu).
  - `convertSchema` (Yup) ditambah field `province`/`regency` dengan cross-validation yang sama seperti `prospectSchema.js`.

**Verifikasi**: `npm run lint` di kedua modul tetap 0 error setelah perbaikan ini; alur `new Partner(partnerData)` dikonfirmasi menerima field apa adanya dari objek yang sudah lengkap, tidak butuh perubahan tambahan di service layer.

---

## 🌿 Lanjutan — 31 Agustus 2026: Partner API & Finalisasi Commit

### 🧭 Latar Belakang

User bertanya apakah **Partner API** (`/p-api/v1/...`, dipakai aplikasi pihak ketiga milik mitra POP ISP, didokumentasikan di `PARTNER_API.md`) juga sudah ikut ter-update dengan field `province`/`regency`. Investigasi menunjukkan **belum cukup** — field sudah ditambahkan ke whitelist proyeksi sejak Sesi 1 (`CUSTOMER_PARTNER_API_FIELDS` dkk), tapi field tersebut **tidak pernah di-`.populate()`** di jalur Partner API mana pun, sehingga yang keluar (kalaupun keluar) cuma ObjectId mentah tanpa nama. Satu endpoint (`GET /p-api/v1/partners/*`, profil POP) bahkan punya `.select()` yang di-hardcode terpisah total dari konstanta yang sudah diupdate, jadi field itu **tidak akan pernah muncul** di sana sampai daftar itu sendiri disentuh.

### 🔍 Keputusan Desain

Sebelum implementasi, dua keputusan dikonfirmasi ke user:

1. **Read-only** — mitra eksternal bisa MELIHAT `province`/`regency` lewat Partner API, tapi TIDAK BISA mengubahnya dari luar. Alasan: field `city` (konsep paling mirip, lokasi terstruktur) sudah read-only di Partner API, dan proses auto-geocode-nya (`updateCity()`, dipanggil fire-and-forget di SETIAP create & update Customer via Partner API) akan menimpa ulang nilai apa pun kalau field serupa dibuat writable. Field `area` (freetext) memang writable tapi tanpa rasional terdokumentasi — bukan preseden yang diikuti.
2. **Sekalian update dokumentasi** — `PARTNER_API.md` dan swagger schema response diupdate di iterasi yang sama, supaya dokumentasi API eksternal akurat sejak awal.

Karena ini API eksternal (kontrak dengan pihak ketiga), scope sengaja dibatasi: hanya menutup kebocoran read (populate/select) + dokumentasi — TIDAK menyentuh whitelist update manapun.

### 🏗️ Komponen yang Diperbaiki

- **`backend/src/controllers/partnerApiCustomer.controller.js`** (`listPartnerAppCustomer`) — `serverFind.populate` diubah dari objek tunggal (`partner` saja, disembunyikan dari role `partner`) jadi array yang SELALU menyertakan `province`/`regency` (tidak sensitif seperti `partner`, jadi tidak perlu disembunyikan dari role manapun), plus `partner` tetap kondisional untuk non-partner.
- **`backend/src/services/customer.service.js`** (`findOneCustomerForPartnerApi`) — tambah `.populate('province', 'name code')` & `.populate('regency', 'name code')`. Karena perbaikan di level service (bukan tiap controller call-site), otomatis berlaku untuk 6 titik pemanggil sekaligus.
- **`backend/src/controllers/partnerApiBusiness.controller.js`** (`listPartnerAppBusiness`) — pola identik dengan Customer.
- **`backend/src/services/business.service.js`** (`findOneBusinessForPartnerApi`) — tambah populate ke `queryBuilder`; `selectFields` tidak perlu diubah karena sudah include `province`/`regency` lewat `BUSINESS_NON_SENSITIVE` sejak Sesi 1.
- **`backend/src/services/partner.service.js`** (`findOnePartnerForPartnerApi`) — **titik paling kritis**: fungsi ini punya `.select([...])` hardcoded sendiri, TERPISAH dari `PARTNER_NON_SENSITIVE` yang sudah diupdate Sesi 1 — inilah kenapa update sebelumnya tidak berpengaruh sama sekali ke endpoint profil POP (`GET /p-api/v1/partners/profile`, `/partners/read/:id`). Ditambahkan `'province'`, `'regency'` ke array select tsb DAN populate ke query chain.
- **`PARTNER_API.md`** — §1 (Customer), §2 (Business), §3 (Partner/POP) semua diupdate menyebut `province`/`regency` sebagai **read-only**, dengan catatan eksplisit supaya konsumen API tidak salah asumsi field ini bisa di-`PATCH`.
- **`backend/src/routes/partnerApi.route.js`** — skema swagger response `PartnerApiCustomerListItem` & `PartnerApiCustomerDetail` ditambahkan properti `province`/`regency` (object `{name, code}`). **Koreksi di tengah jalan**: rencana awal mengira ada skema response khusus Business di baris tertentu, ternyata itu adalah skema Customer kedua (`PartnerApiCustomerDetail`) — dicek ulang lewat `api-docs.json` hasil generate, ternyata Business **sama sekali tidak punya skema response field-level** di swagger (gap dokumentasi lama, di luar scope diperbaiki sekarang).

**Verifikasi**: `npm run lint` backend 0 error; server dev di-restart, `curl /api-docs.json` dicek terparse valid dan properti `province`/`regency` terkonfirmasi ada di kedua skema Customer via query Node langsung terhadap JSON hasil generate.

### 🔧 Git: Rebase `issue-252` ke Atas `master` Terbaru

Menjalankan rebase supaya commit hari ini ada di posisi paling atas histori. Temuan & langkah:

1. Local `master` ternyata tertinggal **12 commit** dari `origin/master` (bukan cuma commit terbaru `resolve #245` di modul `network-monitor` yang diduga awal) — sisanya adalah commit-commit (`#228`, `#223`, `#248`, `#233`, `#247`) yang sudah lebih dulu jadi bagian dari histori `issue-252` sendiri, jadi fast-forward local master aman (tidak ada konten baru yang bentrok).
2. Fast-forward `master` ke `origin/master` (`git merge --ff-only`), lalu `git rebase master` di branch `issue-252` — hanya perlu me-replay **1 commit** (`resolve #252`, berisi seluruh pekerjaan Sesi 1+2), karena commit-commit lain sudah jadi ancestor bersama. **Tidak ada konflik** — dikonfirmasi lebih dulu tidak ada file yang tumpang-tindih antara commit baru master (`network-monitor`/`telegram-api`/`whatsapp-api`) dan commit `issue-252` (backend customer/business/partner/prospect + frontend).
3. Hasil: commit `resolve #252` (hash baru, riwayat lama `134262b2`) sekarang jadi commit teratas, di atas `resolve #245`.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru (Keseluruhan Issue #252)

| Issue | Judul                                            | Dampak Utama                                                                                                                        |
| ----- | ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| #252  | Data Provinsi & Kabupaten/Kota untuk Pelanggan     | 4 entitas (Customer, Business, Partner, Prospect) dapat lokasi administratif terstruktur & cascading; data konsisten terbawa sampai ke tahap konversi Prospect→Customer, dan kini juga terlihat (read-only) oleh mitra eksternal lewat Partner API |

### Kemampuan Baru Pengguna/Admin

- Admin bisa memilih Provinsi lalu Kabupaten/Kota (terfilter otomatis) di 4 entitas pelanggan, termasuk saat mengkonversi Prospect jadi Customer.
- Data lokasi terstruktur kini konsisten dari titik Prospect dibuat sampai jadi Customer — tidak lagi hilang di tengah alur konversi.
- Mitra eksternal (Partner API) bisa melihat Provinsi/Kabupaten-Kota pelanggan & profil POP miliknya sendiri (read-only).

### Bug Fix / Solusi Masalah

- **Kebocoran mapping Prospect → Customer**: `province`/`regency` yang sebelumnya hilang total saat konversi, sekarang ikut terbawa + tervalidasi.
- (Dilaporkan di Sesi 1) Regresi flicker `Combobox.jsx` yang berdampak ke 107 file pemakai komponen shared ini.
- (Dilaporkan di Sesi 1) Konfigurasi koneksi MongoDB lokal yang gagal diam-diam akibat kredensial server tim ikut terbawa ke `.env` lokal.
- **(31 Agustus) `province`/`regency` tidak muncul di Partner API eksternal**: field sudah di whitelist proyeksi sejak Sesi 1 tapi tidak pernah di-`populate()` di 5 titik (list & detail Customer, list & detail Business, profil Partner/POP) — profil POP bahkan punya `.select()` terpisah yang membuatnya kebal terhadap update whitelist manapun. Diperbaiki seluruhnya, read-only (tidak writable oleh mitra eksternal).
- **(31 Agustus) Dokumentasi hilang dari commit**: `PARTNER_API.md` sempat kehilangan seluruh edit `province`/`regency` akibat perubahan tidak tersimpan konsisten antar giliran kerja — dibuat ulang & di-amend ke commit final.

### Menu/Fitur Baru

- Dropdown cascading Provinsi/Kabupaten-Kota di form create/edit/convert Prospect.
- Kolom & baris info Provinsi/Kabupaten-Kota di tabel list & halaman detail Prospect.
- Data lokasi terstruktur (read-only) kini juga terlihat oleh mitra eksternal lewat Partner API (list & detail Customer/Business, profil Partner/POP).

---

## 📖 Informasi & Tutorial Singkat Fitur — Alur Prospect → Customer

- **Penjelasan Fitur**: Prospect adalah lead/calon pelanggan sebelum resmi jadi Customer. Data Provinsi/Kabupaten-Kota yang diisi sejak tahap Prospect kini **ikut terbawa otomatis** saat Prospect dikonversi jadi Customer via tombol Convert — admin tidak perlu mengisi ulang dari nol, tapi tetap bisa mengubahnya di form konversi kalau perlu sebelum submit final.
- **Langkah Penggunaan (Tutorial)**:
  1. Buat Prospect baru (drawer cepat) — isi Provinsi/Kabupaten-Kota kalau sudah diketahui saat itu (opsional di tahap ini).
  2. Kalau belum diisi, bisa dilengkapi kapan saja lewat drawer Edit Prospect — dropdown akan menampilkan pilihan yang sudah tersimpan sebelumnya.
  3. Setelah Prospect punya SO (Sales Order) yang sudah signed, tombol **Convert** aktif di halaman Prospect.
  4. Buka form Convert — Provinsi/Kabupaten-Kota yang sudah terisi di Prospect otomatis tampil terisi di sini; bisa diubah kalau perlu sebelum submit.
  5. Submit Convert — Customer (Partner) baru tercipta dengan data Provinsi/Kabupaten-Kota yang sama, siap dipakai untuk kebutuhan berikutnya (tabel list, halaman detail, dsb).
