# 📝 Daily Work Report - Idham (2026-10-02)

---

## 📅 Laporan Harian - 2 Oktober 2026

---

## 🌿 Branch: `issue-359` — Endpoint Partner API untuk Site & Node Selesai (`resolve`)

### 📌 Informasi Issue

- **Nomor Issue**: #359
- **Judul Issue**: Tambahan Endpoint Partner API
- **Status Branch**: **`resolve #359`** — ditandai selesai hari ini, setelah kemarin (1 Oktober) merancang dan mengimplementasikan draf pertamanya (`save #359`).
- Hari ini **bukan** sekadar melanjutkan kode kemarin — mencakup **pemangkasan scope atas permintaan user** dan **audit kepatuhan penuh terhadap AGENTS.md** yang mengubah cukup banyak detail implementasi. Keduanya dicatat eksplisit sebagai revisi di `.agent/apinode_plan.md` §10.

### 📅 Rincian Perubahan

#### [58bc5b3a] - resolve #359 - 2 Oktober 2026, 16:38 WIB (13 file, 2.681 baris ditambah, 74 dihapus — dibandingkan parent langsung)

**1. Pemangkasan scope atas permintaan user: endpoint DELETE site & node dihapus total**

- `DELETE /site/delete/:site_id` dan `DELETE /node/delete/:site_id/:equipment_id` — yang kemarin sudah terimplementasi lengkap dengan 5 lapis validasi relasi — **dibuang seluruhnya**. Partner API sekarang hanya **List, Read, Create, Update** untuk Site & Node; penghapusan tetap **hanya lewat halaman admin**, tidak lewat API mitra.
- Kode yang jadi tidak terpakai ikut dibuang bersih (bukan dibiarkan jadi dead code): fungsi service `deleteLocationPointById` & `removeFiberEquipment`, `countSplicesByEquipment` (`fiberCable.service.js`), `countNetworkDevicesByLocation` (`networkDevice.service.js`), key i18n `location.haveNetworkDevice`, dukungan kirim `__v` lewat query string (`?__v=`, yang kemarin ditambahkan khusus untuk DELETE), serta seluruh test skenario delete di kedua file test.
- Endpoint final: **7 untuk Site** (list, stats, read, create, update, type-select, group-select) dan **4 untuk Node** (list, read, create, update) — total 11, bukan 13 seperti rencana awal.

**2. Audit kepatuhan AGENTS.md — beberapa pola di draf kemarin diperbaiki supaya konsisten dengan pola modul lain:**

- [`backend/src/utils/location-error.js`](backend/src/utils/location-error.js) [NEW, 72 baris] — Domain error terstruktur (`LOCATION_ERROR` enum + `LOCATION_ERROR_STATUS`), mengikuti pola yang **sudah ada** di `finance-error.js`/`payroll-error.js` (bukan pola ad-hoc `err.status` yang dipakai draf kemarin). Service melempar error berkode (`createLocationError('PORT_IN_USE', ...)`), controller menerjemahkan ke status HTTP via `rethrowLocationError(res, error)` di dalam `catch` — konsisten dengan controller payroll yang sudah ada. Helper `withServiceStatus` ad-hoc yang dipakai draf kemarin dihapus.
- [`backend/src/utils/partner-api-scope.js`](backend/src/utils/partner-api-scope.js) [NEW, 128 baris] — 6 helper scoping kepemilikan (`buildSiteReadFilter`, `buildSiteWriteFilter`, `buildSiteListFilter`, `resolvePortalPartnerId`, `getPartnerObjectId`, `parseVersion`) **dipindah keluar** dari controller (rencana awal menaruhnya di `partnerApiSite.controller.js` lalu di-export) ke util bersama — lebih sesuai pola "Reusability: extract common logic ke utils/helpers" (AGENTS.md §4.9).
- [`locationPoint.service.js`](backend/src/services/locationPoint.service.js) [+585/-5] — `updateLocationPointForPartnerApi`: query cek keberadaan site dipindah ke **dalam** blok `try`, supaya kegagalan teknisnya (bukan cuma "tidak ditemukan") ikut ter-log lewat `logger.error` sebelum re-throw — menutup celah kepatuhan §2.A aturan "jangan menelan error" yang terlewat di draf kemarin.
- **Validasi update node diperketat**: `spec` kini divalidasi formatnya (400 bila salah format) **dan** wajib cocok dengan jumlah port IN/OUT yang sudah ada di perangkat — tidak cocok → **422** `SPEC_MISMATCH` (key baru `fiber.equipment.specMismatch`). Port yang disebut di body tapi tidak ada di perangkat sekarang dijawab **422** (bukan 404 seperti kemarin — port yang tidak ada itu masalah bisnis/validasi, bukan resource yang hilang).
- **Pesan konflik versi diubah jadi netral**: key baru `location.versionConflict` ("Data telah diubah oleh pengguna lain, muat ulang data lalu coba lagi"), menggantikan `fiber.error.optimisticConflict` kemarin yang menyebut "Admin lain" secara spesifik — lebih tepat karena endpoint ini juga dipakai mitra, bukan hanya admin.
- Swagger `/site/list` memakai schema respons ramping `PartnerApiSiteListItem` (bukan skema `LocationPoint` penuh), konsisten dengan aturan "Minimal Data Response" (§2.A aturan 7).
- [`backend/src/routes/partnerApi.route.js`](backend/src/routes/partnerApi.route.js) [+839] — Swagger JSDoc disesuaikan untuk 11 endpoint final (bukan 13), termasuk skema error baru.
- `backend/src/locales/{id,en}/translation.json` [+13/+1] — Key final: `location.invalidCoordinate`, `location.nameUsed`, `location.versionRequired`, `location.versionConflict`, `fiber.equipment.notFound`/`invalidType`/`portNotFound`/`portInUse`/`invalidPortCount`/`specMismatch`. (`location.haveNetworkDevice` dari draf kemarin dicabut karena endpoint delete-nya sudah tidak ada.)
- [`partnerApiNode.test.js`](backend/test/integration/partnerApiNode.test.js) [301 baris, 8 skenario] & [`partnerApiSite.test.js`](backend/test/integration/partnerApiSite.test.js) [259 baris, 8 skenario] — disesuaikan: skenario delete dibuang, ditambah skenario baru untuk `SPEC_MISMATCH` dan validasi port-tidak-ditemukan versi 422.

### 🧭 Latar Belakang Perubahan Scope

Rencana awal (ditulis & diimplementasikan 1 Oktober) mencakup CRUD penuh termasuk DELETE, lengkap dengan 5 lapis pengecekan relasi (equipment gudang, port terhubung, kabel, perangkat jaringan, splice) sebelum sebuah site/node boleh dihapus lewat Partner API. User memutuskan untuk **mencabut kewenangan hapus dari mitra sama sekali** — penghapusan infrastruktur tetap jadi keputusan administratif yang hanya bisa dilakukan lewat halaman admin, bukan diotomatisasi lewat API pihak ketiga. Keputusan ini diikuti dengan pembersihan kode yang konsisten (bukan sekadar menghapus route-nya saja, tapi seluruh fungsi pendukung yang jadi tidak terpakai).

### 📖 Informasi Singkat Fitur

- **Kegunaan final**: Mitra (dan admin) kini bisa **membaca, membuat, dan memperbarui** data site (lokasi POP) serta perangkat fiber (splitter/ODP/ODC/patch panel) di dalamnya lewat Partner API — dengan isolasi ketat antar-mitra dan optimistic locking (`__v`) terhadap edit bersamaan oleh admin di aplikasi utama. **Penghapusan sengaja tidak disediakan** di jalur ini; infrastruktur hanya bisa dihapus oleh admin lewat halaman `/network/sites` atau `NodeInfoDrawer` seperti biasa.
- **Siap digunakan**: status sudah `resolve`, tinggal menunggu squash & PR sesuai alur Git Workflow (AGENTS.md §5).

---

## 📢 Ringkasan Dampak

| Branch | Dampak Utama |
| --- | --- |
| `issue-359` | Partner API punya 11 endpoint baru (CRUD minus Delete) untuk Site & Node fiber, dengan pola error & scoping yang diselaraskan ke standar AGENTS.md lewat audit kepatuhan di hari penyelesaiannya. |
