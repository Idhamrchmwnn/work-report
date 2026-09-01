# 📝 Daily Work Report - Idham (2026-09-01)

---

## 📅 Laporan Harian - 1 September 2026

---

## ⚠️ Koreksi dari Laporan Sebelumnya

Laporan ini sebelumnya menyatakan "tidak ada perubahan hari ini" karena `git status` bersih dan tidak ada commit baru terlihat di histori. **Itu keliru** — pekerjaan hari ini dilakukan lewat `git commit --amend --no-edit` pada commit `resolve #252` yang sudah ada, sehingga:
- Pesan commit **tidak berubah** ("resolve #252"),
- Hash commit **berubah** (`29d266f2` → `7a9066e8`), tapi ini tidak kentara dari `git log` biasa karena nama branch & pesan commit identik,
- `git status` tetap bersih karena working tree sudah ikut ke-commit lewat amend tsb.

Ditemukan lewat `git reflog show issue-252`, lalu isi perubahan digali dengan `git diff 29d266f2 7a9066e8`. Hasilnya: perubahan hari ini **jauh lebih besar dari sekadar revisi kecil** — ini adalah **pivot arsitektur** yang membongkar sebagian besar pekerjaan Sebelumnya (dilaporkan di `App-v2-30.md`/`App-v2-31.md`) dan menggantinya dengan pendekatan yang sama sekali berbeda. **Kedua laporan tersebut kini sebagian sudah usang** untuk bagian Provinsi/Kabupaten-Kota — bagian di bawah ini adalah kondisi final yang sebenarnya berlaku.

---

## 🌿 Branch: `issue-252` — Pivot: Provinsi/Kabupaten-Kota dari Manual ke Auto-Geocode

### 🧭 Perubahan Keputusan Desain

**Pendekatan Sebelumnya (dibongkar hari ini)**: Provinsi/Kabupaten-Kota adalah field `ObjectId` yang merujuk ke koleksi referensi baru (`Province`/`Regency`, di-seed dari dataset resmi Kemendagri, 34+514 entri), dipilih manual oleh admin lewat dropdown cascading (Combobox Provinsi → Combobox Kabupaten/Kota terfilter), dengan validasi silang bahwa kabupaten/kota yang dipilih benar-benar berada di provinsi yang dipilih.

**Pendekatan baru hari ini**: Provinsi/Kabupaten-Kota menjadi **field `String` biasa, terisi otomatis dari reverse-geocoding koordinat lewat Mapbox** — persis mekanisme yang sudah lama dipakai field `city` yang sudah ada. Tidak ada lagi input manual, tidak ada lagi koleksi referensi, tidak ada lagi validasi silang (karena tidak ada lagi kombinasi yang bisa "salah" — datanya selalu ikut hasil geocoding).

Perubahan ini kemungkinan besar didorong oleh penyederhanaan: pendekatan manual butuh infrastruktur baru yang cukup besar (model, service, controller, route, seed data, util validasi, UI cascading di 12 form berbeda) untuk sesuatu yang polanya sudah ada & terbukti jalan (`city` via `getCity()`) — auto-geocode konsisten dengan pola existing dan menghilangkan seluruh beban maintenance data referensi wilayah.

### 🏗️ Infrastruktur yang Dihapus Total (30-31 agustus, sekarang tidak lagi ada)

- `backend/src/models/province.model.js`, `backend/src/models/regency.model.js`
- `backend/src/data/provinceSeed.json` (34 entri), `backend/src/data/regencySeed.json` (514 entri)
- `backend/scripts/seedRegion.js` (script seed)
- `backend/src/services/region.service.js`, `backend/src/controllers/region.controller.js`, `backend/src/routes/region.route.js` (endpoint `/province/select`, `/regency/select`)
- `backend/src/utils/validate-region.js` (`validateRegencyProvince`)
- Registrasi route-nya di `backend/src/app.js` juga dilepas.
- Seluruh Combobox Provinsi→Kabupaten/Kota di 12 form frontend: `customer/{create,edit,editBatch}.jsx`, `business/{create,edit,editBatch}.jsx`, `partner/{create,edit,editBatch}.jsx`, `prospect/{create,edit,convert}.jsx` — beserta `useController`/`watch`/`useMemo(extraBody)` pendukungnya.
- Validasi Yup `.test('regency-requires-province', ...)` di `schema/createShema.js` (customer/business/partner) dan `prospectSchema.js`.
- Branch render khusus `key === 'province' || key === 'regency' ? value?.name : ...` di `customer/profile.jsx`, `business/profile.jsx`, `partner/profile.jsx` (tidak perlu lagi karena bukan objek populate, cukup fallback string biasa).
- `.populate('province')`/`.populate('regency')` di seluruh service (`customer.service.js`, `business.service.js`, `partner.service.js`, `customerPartner.service.js`, `prospect.service.js`) dan Partner API (`partnerApiCustomer.controller.js`, `partnerApiBusiness.controller.js`).

### 🆕 Yang Dibangun sebagai Gantinya

- **`backend/src/utils/get-city.js`** — diperluas: sebelumnya cuma mengembalikan `city` dari hasil Mapbox reverse-geocoding, sekarang juga mengembalikan `province` (diambil dari context level `region` milik Mapbox) dan `regency` (dari context level `place`, lebih presisi dari `city` yang kadang berisi kecamatan/`locality`). Catatan di source: `province` bisa `undefined` untuk beberapa daerah (mis. DKI Jakarta) karena Mapbox tidak selalu menyertakan level itu.
- **`backend/src/utils/update-customer-city.js`, `update-partner-city.js`** — cron/util refresh kota berkala yang sudah ada, disesuaikan supaya `province`/`regency` ikut diperbarui bersamaan dengan `city` (satu panggilan `getCity()`, satu update).
- **`backend/src/controllers/prospect.controller.js`** — fungsi baru `updateProspectRegion(id, coordinate)`: memperbarui `province`/`regency` Prospect secara **fire-and-forget** (tidak memblokir response create/update menunggu Mapbox) — dipanggil di `createProspect` dan `updateProspect`. `convertProspect` disederhanakan: tinggal salin `prospect.province`/`prospect.regency` apa adanya ke data Partner baru (bukan lagi `?._id` dari objek populate), validasi silang dihapus karena tidak relevan lagi.
- **`backend/src/controllers/registration.controller.js`** (BARU — di luar cakupan sebelumnya) — `createRegistration` dan `requestNewSubscription` kini ikut mengisi `province`/`regency` dari `getCity()`, sejalan dengan `city` yang sudah lebih dulu diisi di titik yang sama. Ini menutup celah yang di laporan 31 Agustus sengaja **tidak** dikerjakan ("Temuan #1: form registrasi publik... user memutuskan tidak memperluas ke sini") — dengan pendekatan auto-geocode, perluasan ini jadi trivial karena tinggal ikut pola `city` yang sudah ada di titik yang sama.
- **`backend/src/services/recovery.service.js`** — `changeRegistrationStatus` (proses recovery/reprocessing pendaftaran) ikut mengisi `province`/`regency` bersamaan dengan `city`.
- **Model** (`customer.model.js`, `partner.model.js`, `prospect.model.js`) — field `province`/`regency` diubah dari `{type: ObjectId, ref: 'Province'/'Regency'}` menjadi `{type: String}` sederhana.
- **Frontend** — `schema/columns.jsx` di semua entitas: accessor diubah dari `row.province?.name` menjadi `row.province` langsung (string, bukan objek). `profile.jsx` fallback kembali ke default (`typeof value === 'object' ? null : value`), tidak perlu branch khusus lagi.
- **`PARTNER_API.md`** — deskripsi `province`/`regency` diperbarui: dari *"objek populate `{name, code}`... referensi data wilayah resmi yang dikelola terpusat di sisi admin internal"* menjadi *"string, auto-terisi dari geocoding koordinat (sama seperti `city`)"*. Status **read-only** tetap sama, hanya alasannya yang berubah.

### 📢 Dampak Perbandingan

| Aspek | commit 30-31 agustus (dibongkar) | Hari Ini (final) |
| --- | --- | --- |
| Sumber data | Koleksi referensi wilayah resmi (di-seed manual) | Reverse-geocoding Mapbox otomatis |
| Input | Dropdown cascading, dipilih admin | Tidak ada input — otomatis ikut koordinat |
| Tipe field | `ObjectId ref` | `String` |
| Validasi | Cross-check kabupaten↔provinsi | Tidak perlu (tidak ada kombinasi manual) |
| Cakupan entitas | Customer, Business, Partner, Prospect | + Registrasi & alur Recovery (baru, sebelumnya sengaja dilewati) |
| Partner API | Read-only, populate `{name, code}` | Read-only, string polos |
| Kompleksitas kode | Tinggi (7 file backend baru, 12 form frontend diubah) | Rendah (1 util diperluas, dipakai ulang di titik-titik yang sudah ada) |

---

## ⚠️ Catatan Tambahan

- **Rekomendasi proses**: `git commit --amend` pada pekerjaan besar lintas hari membuat histori sulit ditelusuri (persis seperti yang terjadi di sini — laporan kemarin sempat salah menyimpulkan "tidak ada kerjaan"). Untuk perubahan sebesar ini (pivot arsitektur, bukan revisi kecil), commit baru terpisah lebih aman untuk keterlacakan, terutama karena branch ini sudah pernah di-rebase & di-amend beberapa kali sebelumnya (lihat `App-v2-31.md`).
- Karena field `province`/`regency` kini bergantung penuh pada Mapbox, data yang sudah lanjut diisi manual di masa Sesi 1-2 (kalau sempat ada yang disimpan ke DB produksi/staging) kemungkinan perlu di-refresh ulang lewat `update-customer-city.js`/`update-partner-city.js` supaya konsisten dengan skema baru — perlu dicek apakah ada data lama bertipe ObjectId yang tertinggal di database.
- Laporan `App-v2-30.md` dan `App-v2-31.md` sebaiknya dibaca dengan catatan ini di tangan — bagian "Komponen yang Dibuat/Diubah" di kedua laporan tersebut menjelaskan desain yang **sudah tidak lagi berlaku** per hari ini.
