# 📝 Daily Work Report - Idham (2026-09-02)

---

## 📅 Laporan Harian - 2 September 2026

---

## 🌿 Branch: `issue-260` — Filter Tampil Mitra Bisnis di Aplikasi Mirroring

### 📌 Informasi Issue

- **Nomor Issue**: #260 (sub-issue dari #246 "Revisi & Update Aplikasi Pelaporan")
- **Judul Issue**: Implementasi Filter data master ke aplikasi pelaporan [pelanggan]
- **Status Branch**: `Belum di-merge` (sudah di-push ke `origin/issue-260`)

### 📅 Rincian Perubahan

#### [5325a6e1] - resolve #260 - 2 September 2026, 16:51:06 WIB

- **Komponen yang Berubah**:
  - [`backend/src/models/partner.model.js`](backend/src/models/partner.model.js) — Field baru `show_in_mirror: { type: Boolean, default: false }`, sejajar `status`/`reseller`, untuk mengatur tampil/tidaknya satu mitra di aplikasi mirroring secara independen dari kedua field itu.
  - [`backend/src/services/partner.service.js`](backend/src/services/partner.service.js) — `PARTNER_NON_SENSITIVE` ditambah `show_in_mirror`. Fungsi baru `setPartnerMirrorVisibility()` yang meniru `setPartnerStatus`, **plus** memanggil `redisDelete()` atas cache sesi mitra setelah update — mitra yang dinonaktifkan langsung kehilangan akses seketika, bukan menunggu TTL cache sesi 5 menit habis. `findOnePartnerForPartnerApi()` (dipakai endpoint profil & baca detail mitra) ditambah syarat `show_in_mirror: true` di query utamanya. Fungsi baru `getHiddenMirrorPartnerIds()` — daftar `_id` seluruh mitra yang sedang `show_in_mirror: false`, dipakai bersama oleh 5 controller besar di bawah untuk mengecualikan data milik mereka.
  - [`backend/src/controllers/partner.controller.js`](backend/src/controllers/partner.controller.js) — Controller baru `changeMirrorVisibilityPartner`: ambil mitra by `partner_id`, toggle `!user.show_in_mirror`, panggil service di atas.
  - [`backend/src/routes/partner.route.js`](backend/src/routes/partner.route.js) — Route baru `PATCH /api/v1/partner/change-mirror-visibility`, digerbang `checkPrivilege('partner.changeMirrorVisibility')`.
  - [`backend/src/config/privilege.json`](backend/src/config/privilege.json) — Entri `"changeMirrorVisibility": "PARTNER_CHANGEMIRRORVISIBILITY"`, dihasilkan otomatis lewat `npm run gp`.
  - [`backend/src/controllers/auth.controller.js`](backend/src/controllers/auth.controller.js) — Query login mitra `partnerAppGetToken` ditambah syarat `show_in_mirror: true`, sejajar `reseller`/`status`. Login admin di endpoint yang sama tidak terpengaruh.
  - [`backend/src/middlewares/auth.middleware.js`](backend/src/middlewares/auth.middleware.js) — `protectedPartnerApp` menambah syarat sama saat resolve `req.user` untuk role `partner`, supaya sesi yang sudah terlanjur login juga langsung ditolak begitu di-nonaktifkan.
  - [`backend/src/controllers/partnerApiNetworkDevice.controller.js`](backend/src/controllers/partnerApiNetworkDevice.controller.js), [`backend/src/controllers/partnerApiRadius.controller.js`](backend/src/controllers/partnerApiRadius.controller.js) — Endpoint yang menerima `partner_id` eksplisit untuk sesi admin (list & stats device, filter akun radius) sekarang mensyaratkan mitra target `show_in_mirror: true`; **bug tambahan ikut diperbaiki**: sebelumnya bila `partner_id` yang diminta tidak ditemukan/valid, filter jatuh ke kondisi kosong (`{}`) yang berarti admin diam-diam melihat data SEMUA mitra — sekarang dipaksa mengembalikan 404 atau hasil kosong.
  - [`backend/src/controllers/partnerApiCustomer.controller.js`](backend/src/controllers/partnerApiCustomer.controller.js), [`backend/src/controllers/partnerApiBusiness.controller.js`](backend/src/controllers/partnerApiBusiness.controller.js) — Endpoint list/read/list-status untuk sesi admin (sebelumnya tanpa filter kepemilikan sama sekali) sekarang mengecualikan record milik mitra ber-`show_in_mirror: false` lewat helper lokal `withMirrorVisibility()`. Untuk Business (disimpan di koleksi `Partner` yang sama, `reseller: false`), visibilitasnya sengaja mengikuti flag milik mitra/reseller **pemiliknya** (field `partner`), bukan flag pada dokumen Business itu sendiri.
  - [`backend/src/controllers/partnerApiMap.controller.js`](backend/src/controllers/partnerApiMap.controller.js) — Endpoint markers, tipe marker, daftar kabel, dan kapasitas core untuk sesi admin ditambah filter serupa lewat `getAdminMapVisibilityFilter()`, dengan pengecualian: infrastruktur bersama/publik (`partner` null/tidak ada) tetap tampil.
  - [`backend/src/controllers/partnerApiProductBroadband.controller.js`](backend/src/controllers/partnerApiProductBroadband.controller.js) — 5 endpoint (list, select, group-select, read) ditambah `getAdminProductVisibilityFilter()`, dengan pengecualian: produk katalog global (`available_partner: true`) tetap tampil terlepas dari kepemilikan mitra.
  - [`backend/src/controllers/partnerApiRadiusProfile.controller.js`](backend/src/controllers/partnerApiRadiusProfile.controller.js) — 3 endpoint (list, select, read) ditambah `getAdminRadiusProfileVisibilityFilter()`, dengan pengecualian: profil global/bersama (`pid: 'master'` atau tanpa `partner`) tetap tampil.
  - [`backend/test/helpers/factories.js`](backend/test/helpers/factories.js) — Fixture `createPartner` diberi default `show_in_mirror: true`, supaya 194 pemanggilan fixture lama di 18 file test tidak perlu diubah satu per satu.
  - [`frontend/src/app/pages/users/partner/schema/columns.jsx`](frontend/src/app/pages/users/partner/schema/columns.jsx) — Kolom baru `show_in_mirror` di tabel List Mitra Bisnis, memakai komponen `StatusCell` yang sudah ada — klik langsung `PATCH` ke endpoint baru dengan optimistic update.
  - [`frontend/src/i18n/locales/{id,en}/translations.json`](frontend/src/i18n/locales/id/translations.json), [`backend/src/locales/{id,en}/translation.json`](backend/src/locales/id/translation.json) — String `data.showInMirror` (frontend) dan `partner.mirrorVisibilityChange` (backend) ditambahkan di kedua bahasa.
- **Deskripsi Perubahan & Fungsi**:
  - Menambahkan kontrol visibilitas per-mitra untuk aplikasi eksternal ("aplikasi mirroring") yang menarik data lewat `/p-api/v1` — admin sekarang punya switch untuk menampilkan/menyembunyikan satu mitra dari aplikasi tersebut, independen dari status aktif/nonaktif mitra maupun status reseller-nya.
  - Penegakannya berlapis tiga: mitra yang disembunyikan tidak bisa login ke aplikasi mirroring, sesi yang sudah aktif langsung ditolak begitu di-nonaktifkan (invalidasi cache Redis seketika, bukan menunggu kedaluwarsa), dan — lapisan paling luas — seluruh endpoint `p-api` yang bisa diakses admin lintas-mitra (baik yang menerima parameter `partner_id` eksplisit maupun endpoint list yang sebelumnya tidak difilter sama sekali per-mitra: pelanggan, pelanggan bisnis, peta jaringan, profil RADIUS, dan produk broadband) sekarang konsisten mengecualikan data milik mitra yang disembunyikan, dengan pengecualian eksplisit untuk data/infrastruktur yang memang bersifat global/bersama supaya tidak ikut tersembunyi secara keliru.
  - Diverifikasi: `npm run lint` (backend & frontend) bersih, `npx vitest run` backend 806/808 lolos (2 kegagalan di `networkDevice.service.test.js` tidak berkaitan dengan perubahan ini).
