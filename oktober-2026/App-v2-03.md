# 📝 Daily Work Report - Idham (2026-10-03)

---

## 📅 Laporan Harian - 3 Oktober 2026

---

## 🌿 Branch: `issue-366` — Modul Baru: Arsip Dokumen Administrasi

### 📌 Informasi Issue

- **Nomor Issue**: #366 (berdiri sendiri)
- **Judul Issue**: Tambahan Arsip Dokumen Administrasi
- **Status Branch**: `resolve #366` — langsung selesai dalam satu commit, sudah di-push ke `origin/issue-366`.

### 🧭 Latar Belakang & Tujuan

Modul **Arsip > Administrasi** baru ditambahkan sebagai tempat arsip berkas PDF yang **diunggah** (perjanjian, surat, memo, dsb. yang jenisnya beragam) — **tanpa** alur persetujuan/tanda tangan seperti modul Ketetapan Direktur (`direkturDocument`) yang sudah ada. Fitur ini sengaja dirancang sebagai arsip pasif: unggah, beri nomor, cari/filter, lihat, edit metadata, hapus — tidak ada status `draft`/`signed`/`approval`.

### 📅 Rincian Perubahan

#### [8a553e9c] - resolve #366 - 3 Oktober 2026, 18:05 WIB (28 file, 2.681 baris ditambah, murni penambahan — tidak ada baris dihapus)

- **Backend — 7 endpoint baru** di [`administrationDocument.route.js`](backend/src/routes/administrationDocument.route.js) [294 baris, NEW], digerbang 4 privilege baru (`administrationDocument.list/read/create/update/delete` — 5 privilege, vocabulary standar §4.11):
  - `POST /administration-document/create` — unggah PDF baru (multipart).
  - `GET /administration-document/next-number` — pratinjau angka urutan berikutnya (hanya pratinjau; angka final diambil ulang saat benar-benar disimpan, supaya dua pengunggah bersamaan tidak kebagian nomor yang sama).
  - `POST /administration-document/category-select` — autocomplete kategori (pola agregasi `$group`+`$limit` standar, lihat AGENTS.md Bab 9).
  - `POST /administration-document/list-all` — datatable.
  - `GET /administration-document/view/:document_id` — detail (menerima ObjectId **atau** nomor dokumen langsung).
  - `PATCH /administration-document/update/:document_id`, `DELETE /administration-document/delete/:document_id` (soft delete).
  - [`administrationDocument.controller.js`](backend/src/controllers/administrationDocument.controller.js) [380 baris, NEW] → [`administrationDocument.service.js`](backend/src/services/administrationDocument.service.js) [211 baris, NEW] — mengikuti pola controller→service baku.
  - [`administrationDocument.model.js`](backend/src/models/administrationDocument.model.js) [82 baris, NEW] — koleksi `administration_document`, soft-delete (`mongoose-delete`), auto-increment `seq` sendiri terpisah dari modul lain.

- **Format nomor dokumen** mengikuti persis pola Ketetapan Direktur: `{urutan}/{kode}/{bulanRomawi}/{tahun}` (contoh `001/PKS-RMN/POP-MKT/VIII/2026`). Dua mode input: (1) ketik `number_code` saja (kode disensor jadi kapital, divalidasi format `[A-Z0-9]` dipisah `-`/`/`/`.`, lalu nomor disusun otomatis di backend), atau (2) ketik `document_number` lengkap secara manual. Validasi kode dijalankan **sebelum** berkas diunggah supaya input tidak valid tidak meninggalkan berkas yatim di storage.
- **Validasi berkas**: harus `.pdf` dengan mimetype `application/pdf`, maksimal 15 MB, jumlah halaman dibaca lewat `getPdfPageCount` (util yang sudah ada). Pesan error **reuse** key `direktur.document.*` yang sudah ada (bukan duplikat baru) — sesuai aturan "cari dulu sebelum membuat key baru" (§4.3).
- **Endpoint file terpisah** — [`files.controller.js`](backend/src/controllers/files.controller.js) [+13], [`files.route.js`](backend/src/routes/files.route.js) [+41]: `GET /file/administration-document/:name`. Secara teknis melayani lewat `getQuotationFile` (bucket `appFiles` yang sama), tapi **dibungkus** validasi pola nama berkas (`administrasi-<timestamp>_<random>.pdf`) supaya privilege `administrationDocument.read` tidak bisa dipakai membaca berkas modul lain yang kebetulan tersimpan di bucket yang sama — detail keamanan yang dicatat eksplisit di komentar kode.
- **Backend lain**: `app.js` [+2] mendaftarkan router baru; `privilege.json` [+7], `privilegeDictionary.json` [+79] — hasil `npm run gp`; `locales/{id,en}/translation.json` [+19 masing-masing].
- [`administrationDocument.test.js`](backend/test/integration/administrationDocument.test.js) [291 baris, NEW, 14 skenario] — mencakup create (mode otomatis & manual), validasi kode/berkas, duplikat nomor (409), update, delete, dan list.

- **Frontend — halaman baru** `frontend/src/app/pages/archive/administration/`:
  - [`index.jsx`](frontend/src/app/pages/archive/administration/index.jsx) [80 baris] — halaman datatable, mengikuti pola halaman Ketetapan Direktur.
  - [`AdministrationDocumentDrawer.jsx`](frontend/src/app/pages/archive/administration/AdministrationDocumentDrawer.jsx) [563 baris] — drawer create/edit, dengan **pratinjau nomor dokumen live** saat mengetik kode (pakai util baru `romanNumeral.js` di sisi frontend, padanan `roman-numeral.js` backend, supaya preview tidak perlu round-trip ke server).
  - [`AdministrationDocumentPreviewModal.jsx`](frontend/src/app/pages/archive/administration/AdministrationDocumentPreviewModal.jsx) [134 baris] — modal pratinjau PDF.
  - `schema/administrationSchema.js` [126 baris] (Yup, batas karakter disamakan persis dengan `TEXT_LIMITS` di backend), `schema/columns.jsx` [115 baris], `downloadAdministrationFile.js` [29 baris].
  - [`ArchiveIndexRedirect.jsx`](frontend/src/app/pages/archive/ArchiveIndexRedirect.jsx) [+4] — menambah Administrasi ke urutan fallback redirect `/archive` (setelah Ketetapan Direktur, sebelum Kartu Informasi Mitra) berdasarkan privilege yang dimiliki user.
  - [`navigation/archive.js`](frontend/src/app/navigation/archive.js) [+10] — item menu baru "Administrasi", digerbang `administrationDocument.list`.
  - [`router/protected.jsx`](frontend/src/app/router/protected.jsx) [+8] — route lazy-loaded.
- **Komponen baru yang di-generalisasi untuk dipakai modul lain juga**:
  - [`FormInput.jsx`](frontend/src/components/shared/form/FormInput.jsx) [+64] — `InputAddons`, varian input dengan teks tetap di kiri/kanan (mis. `001/` + [kode yang diketik] + `/VIII/2026`) — ditambahkan karena prop `prefix`/`suffix` pada `Input` yang sudah ada hanya cukup untuk ikon kecil, tidak untuk teks panjang yang berubah-ubah. Sesuai aturan "jangan bangun ulang dari nol kalau sudah ada" (§2.B.4) — di sini sebaliknya, varian memang belum ada sehingga baru ditambahkan ke tempat yang benar (komponen shared, bukan inline).
  - [`rows.jsx`](frontend/src/components/shared/table/rows.jsx) [+32] — `AdministrationDocumentNumberCell` (klik nomor dokumen membuka pratinjau, hanya bila user punya `administrationDocument.read`) dan `CreatedByAdminCell` (sel pembuat dokumen via `AdminLink`).
  - `constants/privilegeDescriptions.{id,en}.json` [+5 masing-masing] — deskripsi 5 privilege baru untuk halaman manajemen Hak Akses.
  - `i18n/locales/{id,en}/translations.json` [+21 masing-masing].

### 📖 Informasi Singkat Fitur

- **Kegunaan**: Sebelum ini, dokumen administratif umum (perjanjian kerja sama, surat, memo, dsb. yang bentuknya beragam dan tidak ikut alur tanda tangan digital) tidak punya tempat arsip terstruktur — modul Ketetapan Direktur yang ada sengaja khusus untuk dokumen yang **butuh** alur persetujuan/tanda tangan. Modul baru ini mengisi kekosongan itu: admin tinggal mengunggah PDF, memberi judul/kategori/nomor, dan dokumen langsung bisa dicari lewat tabel dengan filter kategori — tanpa birokrasi approval.
- **Cara pakai**: Buka **Arsip → Administrasi**, klik tombol unggah, isi judul dan (opsional) kategori, lalu pilih salah satu dari dua cara memberi nomor: ketik kode pendek (mis. `PKS-RMN/POP-MKT`) dan sistem menyusun nomor lengkap otomatis sesuai tanggal dokumen, atau ketik nomor lengkap sendiri secara manual. Unggah berkas PDF (maksimal 15 MB), simpan. Dokumen langsung muncul di tabel dan bisa dicari berdasarkan nomor/judul/kategori; klik nomornya untuk pratinjau PDF.

---

## 📢 Ringkasan Dampak

| Branch | Dampak Utama |
| --- | --- |
| `issue-366` | Modul arsip baru (Arsip > Administrasi) untuk dokumen PDF umum yang tidak memerlukan alur persetujuan/tanda tangan, lengkap dengan penomoran otomatis bergaya Ketetapan Direktur. |
