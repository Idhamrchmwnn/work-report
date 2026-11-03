# 📝 Daily Work Report - Idham (2026-11-03)

---

## 📅 Laporan Harian - 3 November 2026

---

## 🌿 Branch: `issue-514` — Tanggal Jatuh Tempo & Periode Tagihan pada BAPL (penyelesaian)

### 📌 Informasi Issue

- **Nomor Issue**: #514
- **Judul Issue**: BAPL: masa berlangganan berakhir pada tanggal jatuh tempo, dan dokumen mendukung paket bundling (periode tagihan lebih dari 1 bulan)
- **Status Branch**: `Belum di-merge`. Seluruh `save #514` (termasuk perubahan yang kemarin belum di-commit) sudah di-squash menjadi satu commit `resolve #514` di atas `master` terbaru, dan sudah ada di `origin/issue-514`.

### 📅 Rincian Commit

#### [afd2e03e] - resolve #514 - 3 November 2026, 16:25

- **Ringkasan**:
  - Dasar fitur (jatuh tempo, periode tagihan, `{skema_pembayaran}`) sudah tercatat di laporan 2 November. Hari ini fitur dilengkapi dan dirapikan sampai siap PR.
  - **Jatuh tempo kini opsional saat membuat dokumen.** Bila kosong, server mengisinya otomatis: minimal 12 bulan dan kelipatan periode tagihan paket baru (mis. paket 5 bulanan → 15 bulan). Alasannya, form sumber web/mobile belum tahu paket barunya.
  - **Batas atas** jatuh tempo: paling lama 60 bulan setelah tanggal berlaku (422 `dueDateTooFar`).
  - **Peringatan (bukan penolakan)** bila jatuh tempo tidak tepat di akhir periode paket bundling, lengkap dengan saran tanggal.
  - **Klausul penalti bawaan** dirumuskan ulang: biaya layanan untuk sisa periode yang belum dibayar hingga jatuh tempo, dan biaya yang sudah dibayar di muka tidak dikembalikan. Rumusan ini berlaku untuk paket bulanan maupun bundling, tanpa prorata.
  - **Dokumen lama tetap utuh:** dokumen yang belum menyimpan teks ketentuan ditampilkan dengan teks bawaan versi lama yang dibekukan, bukan bawaan terbaru, supaya isi dokumen bertanda tangan tidak berubah.
  - **Ketentuan bawaan di Pengaturan > Application** disimpan kosong bila isinya sama dengan bawaan (versi mana pun). Tanpa ini, pembaruan ketentuan bawaan tidak pernah berlaku, karena form lama selalu menyimpan salinan teksnya.

- **Komponen yang Berubah & Penjelasannya (tambahan dibanding laporan 2 November)**:

  **Backend**

  - `backend/src/services/serviceChangeDocument.service.js`
    - `DEFAULT_BAPL_TERMS` dengan klausul penalti baru.
    - `LEGACY_BAPL_TERMS_V1`: teks bawaan lama yang dibekukan, khusus dokumen tanpa `terms_text`.
    - `isSystemDefaultBaplTerms`: mengenali teks bawaan lama maupun baru dengan membandingkan isi teks tanpa markup dan spasi berlebih.
    - `defaultContractMonths`: minimal 12 bulan, kelipatan periode.
    - `checkDueDate`: aturan setelah tanggal berlaku dan maksimal 60 bulan.
    - Create mengisi `due_date` otomatis bila kosong.
  - `backend/src/controllers/serviceChangeDocument.controller.js`
    - `due_date` opsional saat create. Bila dikirim tetapi bukan tanggal valid, ditolak 400 `dueDateInvalid`.
  - `backend/src/models/serviceChangeDocument.model.js`
    - Konstanta `SERVICE_CHANGE_MAX_CONTRACT_MONTHS` (60).
  - `backend/src/routes/serviceChangeDocument.route.js`
    - Swagger disesuaikan: `due_date` opsional saat create, dan alasan 422 batas 60 bulan.
  - `backend/src/locales/{id,en}/translation.json`
    - `dueDateInvalid`, `dueDateBeforeEffective`, `dueDateTooFar`.
  - `backend/test/integration/serviceChangeDocument.test.js`
    - Teks bawaan lama yang tersimpan dari form Pengaturan diperlakukan sebagai bawaan.
    - Jatuh tempo tidak valid ditolak 400; jatuh tempo sebelum tanggal berlaku ditolak 422.

  **Frontend**

  - `frontend/src/app/pages/users/document/serviceChange/utils.js` dan `utils.test.js`
    - Padanan frontend untuk `defaultContractMonths`, `MAX_CONTRACT_MONTHS`, `isDueDateAlignedWithPeriod`, saran jatuh tempo, `LEGACY_BAPL_TERMS_HTML_V1`, dan `isSystemDefaultBaplTerms`. Rumusnya sama dengan backend.
    - Test baru: masa otomatis kelipatan periode, peringatan dan saran jatuh tempo, batas 60 bulan, pembekuan teks lama, dan pengenalan teks bawaan.
  - `CreateDrawer.jsx` dan `schema/createSchema.js`
    - Jatuh tempo boleh dikosongkan (diisi otomatis, dengan petunjuk `dueDateAutoHint`). Bila diisi, harus setelah tanggal berlaku dan paling lama 60 bulan.
  - `ServiceChangeDocumentPanel.jsx`
    - Peringatan `dueDateNotAligned` beserta saran tanggal.
    - Pada dokumen lama tanpa `due_date`, mengubah tanggal berlaku tidak ikut mengirim jatuh tempo kecuali PIC mengubahnya. Tanpa aturan ini, menggeser tanggal berlaku memendekkan masa berlangganan diam-diam.
  - `ServiceChangeDocumentView.jsx`
    - Dokumen tanpa `terms_text` memakai teks lama yang dibekukan.
  - `frontend/src/app/pages/settings/sections/Application.jsx`
    - Ketentuan BAPL yang sama dengan bawaan dimuat sebagai bawaan terbaru dan disimpan kosong.
    - Teks bantuan mencantumkan variabel baru.
  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - Pesan validasi dan peringatan jatuh tempo.
    - Label "Biaya Layanan per Periode" dan "Periode Tagihan".
    - Kunci `doc.statement.*` lama yang tidak terpakai dihapus.

  **Dokumentasi**

  - `AGENTS.md` §2.A butir 35
    - Jatuh tempo otomatis, batas 60 bulan, peringatan keselarasan periode, klausul penalti, kewajiban membekukan teks bawaan lama, dan penyimpanan ketentuan bawaan sebagai kosong.

---

## 🌿 Branch: `issue-520` — Titik Awal & Koordinat Opsional saat Mengubah Layanan

### 📌 Informasi Issue

- **Nomor Issue**: #520
- **Judul Issue**: Titik Awal dan Koordinat bisa dibuat opsional saat mengubah layanan Broadband, Dedicated, dan Akses Data (untuk data lama tanpa lokasi)
- **Status Branch**: `Belum di-merge`. Satu commit `resolve #520` di atas `master` terbaru, sudah ada di `origin/issue-520`.

### 📅 Rincian Commit

#### [93f2e9f5] - resolve #520 - 3 November 2026, 18:06

- **Ringkasan**:
  - Data layanan lama yang belum punya Titik Awal (`pop`) atau Koordinat tidak bisa disimpan saat diubah, karena form mewajibkan keduanya.
  - Ada tiga sakelar baru di **Pengaturan > Developer > Pengaturan Khusus**:
    - `disable_broadband_location_mandatory`
    - `disable_dedicated_location_mandatory`
    - `disable_data_access_location_mandatory`
  - Saat sakelar aktif, kedua field boleh kosong **saat mengubah** layanan. Saat **membuat** layanan, keduanya tetap wajib. Koordinat yang diisi tetap divalidasi formatnya.
  - Aturan dinilai pada **nilai akhir** setelah update parsial: nilai tersimpan yang tidak dikirim dianggap tetap ada, jadi mengubah field lain tidak pernah menghapus lokasi.
  - Ikut diperbaiki di update Broadband: field `pop`/`coordinate` yang dikirim kosong sebelumnya ikut di-`$unset` (lokasi terhapus). Kini kosong berarti tidak diubah, dan penghapusan hanya lewat penanda eksplisit `remove-data`, yang juga tunduk pada sakelar.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/utils/serviceLocationInput.js` [NEW]
    - `SERVICE_LOCATION_SETTING_KEYS`: kunci sakelar per jenis layanan.
    - `findMissingServiceLocation`: menghitung field lokasi yang akan kosong setelah update. Dasarnya nilai baru, nilai tersimpan, `remove-data`, atau `unset`. ObjectId dan objek populate dianggap terisi.
    - `isServiceLocationOptional`: membaca sakelar dari pengaturan sistem.
    - `assertServiceLocationComplete`: menolak 400 `form.required` (label "Titik Awal"/"Koordinat") sebelum data ditulis, bila sakelar mati.

  - Controller update:
    - `radiusAuthentication.controller.js` (Broadband): kosong = tidak diubah. Hanya `remove-data` yang meng-`$unset` lokasi. Pemeriksaan wajib-isi dijalankan setelah dokumen ditemukan.
    - `productDedicatedInternet.controller.js` dan `productDataAccess.controller.js`: pemeriksaan wajib-isi sebelum update.
    - Handler baru `getBroadbandEditConfig`, `getDedicatedInternetEditConfig`, `getDataAccessEditConfig`: mengembalikan nilai sakelar sebagai boolean.

  - Routes:
    - `GET /broadband/edit-config` (`broadband.update`), `GET /product/dedicated-internet/edit-config` (`dedicatedInternet.update`), dan `GET /data-access/edit-config` (`dataAccess.update`), lengkap dengan Swagger.
    - Keterangan 400 di Swagger update ketiga layanan diperbarui.
    - Tidak ada key privilege baru: route ini memakai key `update` yang sama.

  - `backend/src/config/privilegeDictionary.json`
    - `apiEndpoints` ketiga key `update` ditambah endpoint `edit-config`.

  - `backend/src/services/option.service.js` dan `settingsBackup.registry.js`
    - Tiga sakelar didaftarkan sebagai pengaturan non-sensitif dan masuk grup Backup & Restore Pengaturan.

  - `backend/src/locales/{id,en}/translation.json`
    - Label `form.pop` dan `form.coordinate`.

  - Test
    - `backend/test/integration/serviceLocationOptional.update.test.js` [NEW], untuk ketiga layanan:
      - `edit-config` mengembalikan boolean;
      - sakelar mati: data lama tanpa lokasi ditolak 400 dan tidak berubah;
      - lokasi tersimpan dipertahankan saat field lain diubah;
      - data lama boleh disimpan setelah lokasinya dilengkapi;
      - sakelar aktif: data lama tanpa lokasi tetap bisa diubah;
      - Broadband: `remove-data` ditolak saat sakelar mati, diizinkan saat aktif.
    - `backend/test/unit/serviceLocationInput.test.js` [NEW]: aturan `findMissingServiceLocation`.
    - `backend/test/helpers/serviceLocation.js` [NEW]: helper untuk mengatur sakelar di test.
    - Test lama (`capacitySpeed`, `updateIp`, `invoiceOnActivated`, `speedLimit`) disesuaikan agar data ujinya punya lokasi.

  **Frontend**

  - `frontend/src/app/pages/services/utils/serviceLocationSchema.js` [NEW] dan `serviceLocationSchema.test.js`
    - Skema Yup bersama untuk Titik Awal dan Koordinat (format `lat, lng` dengan rentang sah).
    - Wajib di form tambah; boleh kosong di form ubah saat sakelar aktif; koordinat yang diisi tetap divalidasi.

  - `frontend/src/app/pages/services/hooks/useServiceLocationOptional.js` [NEW]
    - Memuat `.../edit-config` saat form dibuka. Bila gagal dimuat, lokasi tetap wajib sebagai pilihan aman.

  - Form ubah Broadband, Dedicated, dan Akses Data (`edit.jsx` beserta skemanya)
    - Memakai hook dan skema bersama di atas, menggantikan aturan Titik Awal/Koordinat yang sebelumnya ditulis berulang di tiap skema.

  - `frontend/src/app/pages/settings/sections/developer/SpecialSettingsTab.jsx`
    - Tiga kartu sakelar baru beserta keterangannya.

  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - Judul, deskripsi, dan keterangan mode untuk tiga sakelar.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #514 | `afd2e03e` resolve #514 | Penyelesaian BAPL: jatuh tempo otomatis (kelipatan periode, maks 60 bulan), klausul penalti baru, pembekuan ketentuan lama, penyimpanan ketentuan bawaan; squash jadi satu commit |
| #520 | `93f2e9f5` resolve #520 | Titik Awal & Koordinat opsional saat mengubah Broadband/Dedicated/Akses Data, plus perbaikan lokasi Broadband yang terhapus saat dikirim kosong |
