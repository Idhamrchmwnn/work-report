# 📝 Daily Work Report - Idham (2026-10-08)

---

## 📅 Laporan Harian - 8 Oktober 2026

---

## 🌿 Branch: `issue-384` — Generalisasi "Dokumen Arsip Unggahan" & Modul Baru Surat Keluar

### 📌 Informasi Issue

- **Nomor Issue**: #384 (berdiri sendiri)
- **Judul Issue**: Tambahan Arsip Surat Keluar
- **Status Branch**: **Belum di-commit** — seluruh pekerjaan hari ini masih berupa perubahan staged/unstaged + file baru untracked di working directory, belum ada commit `save`/`resolve` sama sekali.

### 🧭 Latar Belakang & Pendekatan

Kebutuhan hari ini adalah menambah modul **Arsip > Surat Keluar** — arsip PDF surat keluar perusahaan, yang persis sama fungsinya dengan **Arsip > Administrasi** (dibuat 3 Oktober, issue-366): unggah PDF, beri nomor otomatis/manual, kategori, tanpa alur persetujuan/tanda tangan. Daripada menyalin ulang seluruh controller/service/komponen React Administrasi (yang berarti dua kali kerja setiap ada bug/perbaikan ke depan), hari ini **modul Administrasi digeneralisasi dulu** menjadi fondasi bersama "dokumen arsip unggahan", lalu Surat Keluar dibangun sebagai pemakai kedua dari fondasi yang sama. Belum ada commit — ini laporan progres pekerjaan yang masih berjalan.

### 📅 Rincian Perubahan (working tree saat ini)

**Backend — fondasi bersama (baru):**
- [`backend/src/models/shared/archiveDocument.model.js`](backend/src/models/shared/archiveDocument.model.js) [NEW] — `createArchiveDocumentModel({ modelName, collection })`, factory skema Mongoose. Tiap pemakai mendapat **koleksi dan penghitung nomor urut sendiri-sendiri** (kunci auto-increment memakai `modelName`), supaya nomor Surat Keluar tidak ikut melompat saat Administrasi diunggah, atau sebaliknya.
  - **Bug nyata diperbaiki sekaligus saat generalisasi**: index unik `document_number` sebelumnya berlaku ke **seluruh** dokumen termasuk yang sudah soft-delete; sekarang `partialFilterExpression: { deleted: false }` — nomor dari dokumen yang sudah dihapus bisa dipakai ulang.
- [`backend/src/services/archiveDocument.service.js`](backend/src/services/archiveDocument.service.js) [NEW] — `createArchiveDocumentService(Model, { i18nPrefix, context })`, factory 7 fungsi data (`findById`, `findAllForTable`, `create`, `update`, `deleteById`, `nextCount`, `findCategorySelect`). Service tidak tahu apa pun soal HTTP — error yang bisa diprediksi membawa `statusCode` untuk diterapkan controller (AGENTS.md §2.A aturan 3).
  - **Perbaikan pola datatable sekaligus**: versi Administrasi lama memakai `params.find = {...params.find, pid: 'master'}` (menimpa langsung objek params); versi bersama ini memakai parameter `serverFind` eksplisit yang dioper ke `dataTable()`, pola yang lebih aman sesuai AGENTS.md §4.11.5 (`dataTable` memakai keberadaan `serverFind.find` untuk memutuskan apakah `params.find` kiriman klien masih dihormati).
- [`backend/src/controllers/archiveDocument.controller.js`](backend/src/controllers/archiveDocument.controller.js) [NEW] — `createArchiveDocumentController({ service, i18nPrefix, filePrefix, context })`, factory 7 handler Express (`create`, `listAll`, `read`, `update`, `remove`, `previewNumber`, `categorySelect`), seluruhnya `asyncHandler`. Validasi nomor/kode dokumen, unggah & validasi PDF (maks 15 MB), pembersihan berkas yatim saat gagal simpan — semuanya pindah ke sini dari controller Administrasi lama.
- [`backend/src/controllers/files.controller.js`](backend/src/controllers/files.controller.js) — `createArchiveFileHandler({ filePrefix, notFoundKey, context })`: penyaji berkas PDF generik, membatasi nama berkas ke pola `{filePrefix}-{waktu}_{acak}.pdf` miliknya sendiri (supaya privilege `.read` satu modul tidak bisa dipakai membaca berkas modul lain yang sama-sama tersimpan di bucket `appFiles`) — pola keamanan yang sama persis dengan versi Administrasi (3 Oktober), kini dipakai ulang.

**Backend — dua pemakai, masing-masing jadi *thin wrapper*:**
- **Administrasi** (`administrationDocument.controller.js`/`.service.js`/`.model.js`) dirapikan jadi pemanggil factory di atas — logiknya sendiri sudah habis dipindah ke `archiveDocument.*`.
- **Surat Keluar** (baru, seluruhnya untracked): [`outgoingLetter.model.js`](backend/src/models/outgoingLetter.model.js) (koleksi `outgoing_letter`), [`outgoingLetter.service.js`](backend/src/services/outgoingLetter.service.js), [`outgoingLetter.controller.js`](backend/src/controllers/outgoingLetter.controller.js), [`outgoingLetter.route.js`](backend/src/routes/outgoingLetter.route.js) — 7 endpoint (`create`, `next-number`, `category-select`, `list-all`, `view/:document_id`, `update/:document_id`, `delete/:document_id`), persis meniru pola Administrasi. Didaftarkan ke `app.js`.
- **5 privilege baru** `outgoingLetter.{list,read,create,update,delete}` — `privilege.json`, `privilegeDictionary.json` (lengkap `requires`/`risk`), `privilegeDescriptions.{id,en}.json`.
- **i18n**: key generik dipindah ke awalan baru `archiveDocument.*` (`noFile`, `invalidType`, `tooLarge`, `title`, `numberCode`, `invalidNumberCode`, `invalidDate`), dipakai bersama; key khusus per modul (`created`, `updated`, `notFound`, `duplicateNumber`, dst.) tetap terpisah di bawah `administrationDocument.*` dan `outgoingLetter.*`.
- [`outgoingLetter.test.js`](backend/test/integration/outgoingLetter.test.js) [NEW, 7 skenario] — mencakup create/update/delete/list/kategori.

**Frontend — fondasi bersama (baru), folder `uploadedDocument/` (hasil `git mv` dari `administration/` + modifikasi):**
- [`configs.js`](frontend/src/app/pages/archive/uploadedDocument/configs.js) [NEW] — `ADMINISTRATION_DOCUMENT_CONFIG` & `OUTGOING_LETTER_CONFIG`: satu objek per modul berisi `apiBase`, `fileBase`, `privileges`, `labels` (key i18n). Komponen bersama **hanya** mengambil nilai spesifik modul dari sini.
- `UploadedDocumentPage.jsx`, `UploadedDocumentDrawer.jsx`, `UploadedDocumentPreviewModal.jsx`, `schema/columns.jsx`, `schema/uploadedDocumentSchema.js`, `downloadUploadedDocument.js` — seluruhnya menerima prop/parameter `config`, menggantikan nilai yang sebelumnya di-*hardcode* untuk Administrasi.
- [`getUploadedDocumentEditDrawer.jsx`](frontend/src/app/pages/archive/uploadedDocument/getUploadedDocumentEditDrawer.jsx) [NEW] — RowActions/Datatables (AGENTS.md §2.B.14) hanya meneruskan prop standar (`open`, `onClose`, `cellData`, `reloadTable`) ke komponen drawer, sehingga tiap konfigurasi butuh komponen pembungkusnya sendiri. Di-cache per `config.id` lewat `Map` supaya identitas komponennya stabil antar-render — komponen baru di setiap render berarti *remount* drawer setiap kali.
- `frontend/src/components/shared/table/rows.jsx` — sel kolom tabel yang sebelumnya khusus Administrasi digeneralisasi menerima `config`.

**Frontend — dua halaman tipis:**
- [`archive/administration/index.jsx`](frontend/src/app/pages/archive/administration/index.jsx) — kini cuma `<UploadedDocumentPage config={ADMINISTRATION_DOCUMENT_CONFIG} />`.
- [`archive/outgoingLetter/index.jsx`](frontend/src/app/pages/archive/outgoingLetter/index.jsx) [NEW] — `<UploadedDocumentPage config={OUTGOING_LETTER_CONFIG} />`.
- [`navigation/archive.js`](frontend/src/app/navigation/archive.js) — item menu baru "Surat Keluar" (ikon pesawat kertas), digerbang `outgoingLetter.list`.
- [`router/protected.jsx`](frontend/src/app/router/protected.jsx) — route `/archive/outgoing-letter` lazy-loaded.
- [`ArchiveIndexRedirect.jsx`](frontend/src/app/pages/archive/ArchiveIndexRedirect.jsx) — Surat Keluar ditambahkan ke urutan fallback redirect `/archive` (setelah Administrasi, sebelum Kartu Informasi Mitra).
- `privilegeDescriptions.{id,en}.json`, `i18n/locales/{id,en}/translations.json` — deskripsi & teks UI untuk 5 privilege dan label modul baru.

### 🔧 Perbaikan Sampingan (ditemukan tidak sengaja, di luar scope utama)

- **Kebocoran `process.env.TZ` antar-berkas test** di [`attendanceDeviceClient.test.js`](backend/test/unit/attendanceDeviceClient.test.js) dan [`computeNextBillingDate.test.js`](backend/test/unit/computeNextBillingDate.test.js): `afterAll` keduanya menulis `process.env.TZ = originalTz`, dan bila `originalTz` awalnya `undefined`, assignment itu justru **menyimpan string literal `"undefined"`** (bukan menghapus variabelnya) — zona waktu tak dikenal yang lalu bocor ke berkas test berikutnya dalam fork proses yang sama. Diperbaiki: `delete process.env.TZ` bila awalnya memang kosong.

### 📖 Catatan

- **Belum ada commit** — status masih working tree. Berkas lama `administration/` sudah bersih (hanya menyisakan `index.jsx` tipis di atas), jadi tidak ada duplikasi kode tersisa antara Administrasi dan Surat Keluar.
- Belum terlihat `npm run gp`/`npm run gd` dijalankan ulang untuk memastikan `privilege.json`/`apiEndpoints` ter-generate otomatis dari route baru — perlu dicek sebelum commit, sesuai AGENTS.md §4.11.4.
- Pendekatan generalisasi-lebih-dulu ini konsisten dengan arahan AGENTS.md §4 poin 0 ("cari dulu referensi implementasi serupa... jangan memperkenalkan pola baru untuk hal yang sudah punya solusi standar") — di sini solusinya dijadikan dua kali reusable alih-alih disalin.

---

## 📢 Ringkasan Dampak

| Branch | Dampak Utama |
| --- | --- |
| `issue-384` (belum commit) | Modul Administrasi digeneralisasi jadi fondasi "dokumen arsip unggahan" bersama, dipakai untuk membangun modul baru Arsip > Surat Keluar tanpa duplikasi kode — sekaligus memperbaiki bug index unik dokumen terhapus dan kebocoran `TZ` antar-test. |
