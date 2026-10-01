# 📝 Daily Work Report - Idham (2026-10-01)

---

## 📅 Laporan Harian - 1 Oktober 2026

---

## 🌿 Branch: `issue-359` — Endpoint Partner API untuk Site & Node (Perangkat Fiber)

### 📌 Informasi Issue

- **Nomor Issue**: #359 (berdiri sendiri)
- **Judul Issue**: Tambahan Endpoint Partner API
- **Status Branch**: `save #359` — belum `resolve`, sudah di-push ke `origin/issue-359`. Hari ini mencakup **merancang dari nol sampai mengimplementasikan seluruh rencananya**: dokumen rencana `.agent/apinode_plan.md` ditulis dan ditandai "TERIMPLEMENTASI" tanggal hari ini juga.

### 🧭 Latar Belakang & Tujuan

Partner API (`/p-api/v1`, dipakai mitra dengan token `role: partner` dan admin dengan `role: admin`) sebelumnya hanya bisa **membaca** infrastruktur lokasi lewat `/map/*` — belum ada cara bagi mitra untuk **mengelola** (membuat/mengubah/menghapus) site maupun perangkat fiber di dalamnya sendiri lewat API. Dua istilah diperjelas lebih dulu di rencana karena berasal dari satu model yang sama (`LocationPoint`/`LocationPOP`):

| Istilah | Arti |
| --- | --- |
| **Site** | Dokumen `LocationPoint` itu sendiri — nama, tipe, koordinat, grup, kapasitas, PIC, catatan (padanan halaman admin `/network/sites`). |
| **Node** | Perangkat fiber di dalam site (array `fiber_equipments[]`: SPLITTER/ODP/ODC/PATCH_PANEL beserta port-nya), padanan `NodeEquipment.jsx` di halaman Fiber Cable. |

Di luar scope secara sengaja: perangkat gudang (`equipment[]`, tetap read-only karena dikelola lewat alur tiket/warehouse), sambungan core/splice, dan perubahan skema `LocationPoint`.

### 📅 Rincian Perubahan

#### [67376404] - save #359 - 1 Oktober 2026, 17:54 WIB (13 file, 2.880 baris ditambah, 74 dihapus)

- **13 endpoint baru**, semua singular kebab-case & Bahasa Inggris, digerbang `protectedPartnerApp` (login wajib, konsisten dengan pola p-api yang sudah ada — lihat catatan risiko di bawah) — [`backend/src/routes/partnerApi.route.js`](backend/src/routes/partnerApi.route.js) [+892]:

  **Site** — [`partnerApiSite.controller.js`](backend/src/controllers/partnerApiSite.controller.js) [NEW, 422 baris], 8 handler: `POST /site/list` (datatable), `GET /site/stats`, `GET /site/read/:site_id`, `POST /site/create`, `PATCH /site/update/:site_id`, `DELETE /site/delete/:site_id`, `POST /site/type-select`, `POST /site/group-select`.

  **Node** — [`partnerApiNode.controller.js`](backend/src/controllers/partnerApiNode.controller.js) [NEW, 221 baris], 5 handler: `GET /node/list/:site_id`, `GET /node/read/:site_id/:equipment_id`, `POST /node/create/:site_id`, `PATCH /node/update/:site_id/:equipment_id`, `DELETE /node/delete/:site_id/:equipment_id`.

- **Aturan scoping kepemilikan** (diterapkan lewat filter query di service, bukan fetch-lalu-bandingkan — agar cek dan tulis atomik dalam satu query):
  - Mitra hanya bisa **mengubah/menghapus** site miliknya sendiri; site publik (`partner: null`) atau milik mitra lain → **404 generik** (tidak membocorkan keberadaannya).
  - Mitra tetap bisa **membaca** site publik, tapi `pic_name`/`pic_contact` dibuang dari response (lanjutan audit privasi issue-236).
  - Admin memakai `withPortalVisibility()` — hanya melihat site milik mitra yang `show_in_portal: true`.
  - List site mitra **mengecualikan** site publik, supaya tabel mitra tidak penuh infrastruktur bersama.

- **Optimistic locking (`__v`) wajib** di seluruh aksi tulis site & node — versi yang tidak cocok ditolak **409** (`fiber.error.optimisticConflict`), bukan 400/500, karena endpoint ini rawan diedit bersamaan dengan admin yang sedang membuka `NodeInfoDrawer` di aplikasi utama.

- **Validasi & cek relasi sebelum hapus site** (urutan: equipment terpasang → port fiber terhubung → kabel terpasang → **perangkat jaringan terpasang [baru]** → baru dieksekusi hapus), seluruhnya **422** bukan 400 generik, sesuai AGENTS.md.

- **Backend — service**:
  - [`locationPoint.service.js`](backend/src/services/locationPoint.service.js) [+595/-5] — 10 fungsi baru untuk CRUD site & node versi Partner API (`findListLocationPointForPartnerApi`, `findOneLocationPointForPartnerApi`, `getLocationPointStats`, `createLocationPointForPartnerApi`, `updateLocationPointForPartnerApi`, `deleteLocationPointForPartnerApi`, `listFiberEquipments`/`addFiberEquipment`/`updateFiberEquipment`/`removeFiberEquipment`), memakai `serverFind` (bukan fungsi datatable lama yang masih mempercayai `params.find` dari klien — celah keamanan yang sengaja dihindari).
  - [`networkDevice.service.js`](backend/src/services/networkDevice.service.js) [+14] — `countNetworkDevicesByLocation()`, dipakai validasi hapus site di atas.
  - [`fiberCable.service.js`](backend/src/services/fiberCable.service.js) [+25] — `countSplicesByEquipment()` **(di luar rencana awal)** — cek tambahan saat hapus node: sambungan core/splice yang masih mengarah ke perangkat itu ikut ditolak 422, bukan hanya port `USED`.
  - `locationPoint.controller.js` [+4/-33], `partnerApiMap.controller.js` [+4/-34] — disederhanakan karena sebagian logic dipindah ke service baru di atas.
  - `backend/src/locales/{id,en}/translation.json` [+12/+1 masing-masing] — 9 key baru (`location.haveNetworkDevice`, `location.invalidCoordinate`, `location.nameUsed`, `location.versionRequired`, `fiber.equipment.notFound`/`invalidType`/`portNotFound`/`portInUse`/`invalidPortCount`), diisi `id` & `en` sekaligus.

- **Test**: [`partnerApiSite.test.js`](backend/test/integration/partnerApiSite.test.js) [NEW, 326 baris, 10/10 lulus], [`partnerApiNode.test.js`](backend/test/integration/partnerApiNode.test.js) [NEW, 331 baris, 8/8 lulus], `test/helpers/factories.js` [+22] — factory `createLocationPoint`. Mencakup: isolasi antar-mitra, klien tidak bisa menggeser cakupan lewat `find` di body, konflik `__v` (race condition dua update bersamaan), seluruh kombinasi penolakan hapus 422, serta validasi koordinat/duplikat nama.

#### ⚙️ Penyimpangan dari Rencana Awal (dicatat eksplisit di `.agent/apinode_plan.md` §10)

- **Port node dibuat dari `spec`** (format rasio `IN:OUT`, default `1:8`), bukan dari `port_count` seperti draf awal — menyesuaikan pola form admin `NodeEquipment.jsx` yang sebenarnya. Batasnya 64 IN / 256 OUT.
- **Update node tidak bisa menambah/mengurangi jumlah port** — hanya `name`, `spec`, dan status/catatan per port. Perubahan jumlah port ditunda sampai ada kebutuhan nyata.
- **Bug nyata ketemu lewat test**: `req.user._id` perlu di-cast eksplisit ke ObjectId di filter scoping (`getPartnerObjectId`) — `req.user` dari cache bisa membawa `_id` berupa string, dan aggregate (stats, type/group-select) tidak otomatis melakukan cast seperti `find`.
- **`$inc: { __v: 1 }` ikut ditambahkan** ke 3 fungsi update `LocationPoint` versi lama (`updateLocationPoint`, `updateMultipleLocationPoint`, `updateLocationPointByMapsId`) yang dipakai admin — sebelumnya edit admin lewat `NodeInfoDrawer` **tidak** menaikkan versi dokumen, sehingga lock optimistic di Partner API baru tidak akan pernah mendeteksi perubahan dari sisi admin. Perbaikan ini aman karena form admin tidak pernah mengirim `__v` sendiri.
- **`__v` untuk DELETE dikirim lewat query string** (`?__v=`), bukan body, karena request DELETE umumnya tanpa body.

### ⚠️ Catatan Risiko (didokumentasikan sendiri di rencana, bukan temuan baru)

- **Token admin di Partner API tidak digerbang `checkPrivilege`** — seluruh route `/p-api/v1` (termasuk yang sudah ada sebelumnya) hanya memakai `protectedPartnerApp` (login saja). Endpoint baru ini konsisten dengan pola yang sudah ada, tapi konsekuensinya: admin tanpa privilege `locationPoint.delete` di aplikasi utama tetap bisa menghapus site lewat Partner API. Diusulkan sebagai issue terpisah untuk menambahkan pengecekan privilege khusus sesi admin di p-api.
- Tidak ada perubahan `privilege.json` (tidak ada `checkPrivilege` baru), jadi `npm run gp` tidak perlu dijalankan untuk perubahan ini.

### 📖 Informasi Singkat Fitur

- **Kegunaan**: Sebelumnya mitra yang ingin menambah/mengubah data site (lokasi POP) atau perangkat fiber (splitter/ODP/ODC/patch panel) di dalamnya harus menghubungi admin secara manual. Endpoint baru ini memungkinkan integrasi otomatis dari sisi mitra (lewat Partner API) untuk CRUD penuh, dengan pengamanan: mitra tidak bisa melihat/mengubah data infrastruktur milik mitra lain, dan perubahan bersamaan dengan admin terdeteksi (bukan saling menimpa diam-diam) lewat optimistic locking.
- **Belum bisa dipakai produksi sebelum**: `resolve #359` (status saat ini masih `save`) dan uji manual via Swagger UI dengan token partner & admin sungguhan — keduanya langkah terakhir yang belum ditandai selesai di rencana.

---

## 📢 Ringkasan Dampak

| Branch | Dampak Utama |
| --- | --- |
| `issue-359` | Mitra kini bisa mengelola (bukan hanya membaca) site & perangkat fiber miliknya sendiri lewat Partner API, dengan isolasi antar-mitra dan optimistic locking terhadap edit bersamaan oleh admin. |
