# 📝 Daily Work Report - Idham (2026-09-07)

---

## 📅 Laporan Harian - 7 September 2026

---

## 🌿 Branch: `issue-244` — Pemindahan Dokumen Direktur ke Menu Arsip

### 📌 Informasi Issue

- **Nomor Issue**: #244
- **Judul Issue**: Implementasi Dokumen Direktur
- **Status Branch**: `Belum di-merge` (sudah di-push ke `origin/issue-244`) — reorganisasi navigasi, bukan penambahan fungsi baru pada dokumennya sendiri.

### 📅 Rincian Perubahan

#### [06b2ff20] - resolve #244 - 7 September 2026, 11:37:08 WIB

- **Komponen yang Berubah**:
  - [`frontend/src/app/navigation/arsip.js`](frontend/src/app/navigation/arsip.js) [NEW] — Menu sidebar root baru **"Arsip"** (ikon arsip), dengan satu item awal "Dokumen" mengarah ke `/arsip/document`, digerbang privilege `direkturDocument.read`. Didaftarkan ke `frontend/src/app/navigation/index.js`.
  - [`frontend/src/app/pages/arsip/document/`](frontend/src/app/pages/arsip/document/) [NEW, seluruh folder] — Modul Dokumen Direktur (`create.jsx`, `edit.jsx`, `ReviewDrawer.jsx`, `SignPositionStep.jsx`, `DocumentPreview.jsx`, `DirekturGeneratedDocumentPreview.jsx`, `useDirekturSignFlow.js`, `constants/direkturTemplates.js`, `schema/columns.jsx`, `schema/direkturSchema.js`, `registry.js`, `index.jsx`) — dipindahkan utuh dari `frontend/src/app/pages/users/document/direktur/` ke lokasi baru ini, mengikuti pola registry-modul yang sama (siap ditambah modul arsip lain).
  - [`frontend/src/app/pages/users/document/index.jsx`](frontend/src/app/pages/users/document/index.jsx) — Direktur dikeluarkan dari daftar tab (kini hanya SDN & PKS). Sekaligus **direfactor**: pengecekan privilege per-tab yang tadinya hardcode per `mod.key` (`if (mod.key === 'pks') return canReadPKS...`) diganti generik — tiap modul cukup mendeklarasikan `privilege: '...'` sendiri di registry-nya, dicek lewat `checkPrivilege()` satu baris. Pola yang sama diterapkan di `arsip/document/index.jsx` yang baru.
  - [`frontend/src/hooks/useDocumentApproval.js`](frontend/src/hooks/useDocumentApproval.js) — Dipindah dari `pages/users/document/shared/` ke `hooks/` (barrel `hooks/index.js`) supaya bisa dipakai bersama oleh registry `users/document` (SDN, PKS) **dan** registry `arsip/document` (Direktur) tanpa import lintas-folder yang janggal.
  - [`frontend/src/app/router/protected.jsx`](frontend/src/app/router/protected.jsx) — Route baru `/arsip` dan `/arsip/document`.
  - [`backend/src/routes/files.route.js`](backend/src/routes/files.route.js), `models/shared/*`, `direkturDocument.*` (model/service/controller/route), `utils/telegram.js`, `utils/roman-numeral.js` — ikut terbawa dalam commit ini (rebase/carry-over dari pekerjaan modul Direktur sebelumnya), tidak ada perubahan fungsional baru di baliknya.
- **Deskripsi Perubahan & Fungsi**:
  - Dokumen Direktur (SK Direksi, dsb — dokumen internal korporat) dipisahkan dari menu "Pengguna → Document" (yang isinya dokumen terkait pelanggan/mitra: SDN, PKS) ke menu tersendiri **"Arsip"** di root sidebar — kategorisasi yang lebih sesuai karena sifatnya dokumen internal perusahaan, bukan dokumen pelanggan.
  - Sekaligus merapikan pola privilege-check di halaman tab dokumen supaya generik (baca dari registry), sehingga menambah/memindah modul dokumen berikutnya tidak perlu mengubah logika di `index.jsx` lagi.
  - Tidak ada perubahan pada data, alur approve/sign, maupun kemampuan yang sudah ada di modul Dokumen Direktur itu sendiri (mode upload maupun generated) — murni pemindahan lokasi menu dan folder kode.

### 📖 Informasi & Tutorial Singkat

- **Penjelasan**: Karena ini pemindahan lokasi (bukan fitur baru), yang perlu diketahui pengguna hanyalah **di mana sekarang mencari menu Dokumen Direktur** setelah update ini dipasang — URL lama `/users/document` (tab Direktur) tidak lagi menampilkan tab tersebut.
- **Langkah Menemukan Menu Baru**:
  1. Buka sidebar utama, cari root menu baru **"Arsip"** (ikon kotak arsip) — biasanya muncul di bagian bawah daftar menu, setelah Utilities.
  2. Klik **Arsip → Dokumen**.
  3. Halaman yang tampil persis sama seperti sebelumnya (tab Direktur di menu Document lama) — daftar dokumen, tombol buat baru, drawer review, dan alur tanda tangan semuanya tidak berubah.
  4. Menu **Pengguna → Document** sekarang hanya berisi tab **SDN** dan **PKS** — dokumen pelanggan/mitra.

---

## 🌿 Branch: `issue-278` — Manajemen Berita/Banner untuk Mobile App

### 📌 Informasi Issue

- **Nomor Issue**: #278, sub-issue dari #277 "Integrasi dengan MobileApps (Android)"
- **Judul Issue**: #277 - Integrasi Banner & Informasi
- **Status Branch**: `Belum di-merge` (branch baru, sudah di-push ke `origin/issue-278`)

### 📅 Rincian Perubahan

#### [f00ae7ad] - resolve #278 - 7 September 2026, 17:55:08 WIB

- **Komponen yang Berubah**:
  - [`backend/src/models/news.model.js`](backend/src/models/news.model.js) — Field `updated_at` ditambahkan ke model `MobileNews` yang sudah ada (`title`, `type`: `news`/`article`, `image`, `content`, `show`, `created_by`).
  - [`backend/src/controllers/news.controller.js`](backend/src/controllers/news.controller.js), [`backend/src/services/news.service.js`](backend/src/services/news.service.js) — Sebelumnya hanya ada `getBanner` (endpoint publik baca banner untuk mobile app, tanpa antarmuka kelola). Ditambahkan CRUD admin lengkap: `listNews`, `readNews`, `createNews` (wajib unggah gambar), `updateNews` (ganti gambar opsional, hapus gambar lama otomatis), `deleteNews` (hapus data + berkas gambar).
  - [`backend/src/routes/news.route.js`](backend/src/routes/news.route.js) [NEW, 182 baris] — Sebelumnya `getBanner` tidak punya file route sendiri; sekarang seluruh endpoint (banner publik + 5 endpoint CRUD admin) didaftarkan di sini, digerbang privilege `mobileNews.*` untuk yang admin-only. Dipasang di `backend/src/app.js`.
  - [`backend/src/controllers/files.controller.js`](backend/src/controllers/files.controller.js) — Handler baru `getNewsImage`: menyajikan gambar berita dari bucket `appFiles`, mendukung mode inline atau attachment (`?download=true`) dengan nama berkas custom.
  - [`backend/src/routes/files.route.js`](backend/src/routes/files.route.js) — Route baru untuk `getNewsImage` di atas.
  - [`backend/src/config/privilege.json`](backend/src/config/privilege.json) — Grup privilege baru `mobileNews` (`list`, `read`, `create`, `update`, `delete`).
  - [`frontend/src/app/navigation/mobileApp.js`](frontend/src/app/navigation/mobileApp.js) [NEW] — Menu "Mobile App" baru di sidebar dengan item "Berita/Banner", didaftarkan ke `navigation/index.js`.
  - [`frontend/src/app/pages/mobileApp/news/`](frontend/src/app/pages/mobileApp/news/) [NEW] — Halaman list (`index.jsx`), form create/edit (`create.jsx`, `edit.jsx`), skema kolom tabel (`schema/columns.jsx`), skema validasi upload (`schema/createSchema.js`), badge tipe berita (`schema/NewsTypeBadge.jsx`), dan kartu grid custom (`schema/GridCard.jsx`) untuk menampilkan thumbnail gambar tiap berita di tampilan grid.
  - [`frontend/src/app/router/protected.jsx`](frontend/src/app/router/protected.jsx) — Route baru untuk halaman-halaman di atas.
  - [`frontend/src/components/shared/table/Table.jsx`](frontend/src/components/shared/table/Table.jsx), [`frontend/src/components/shared/table/GridView.jsx`](frontend/src/components/shared/table/GridView.jsx) — **Peningkatan komponen bersama** (dipakai tabel data di seluruh aplikasi, bukan cuma News): `Datatables` sekarang menerima prop `gridCard` (fungsi render kartu custom) dan `defaultViewType` (`'list'`/`'grid'`) — sebelumnya `GridView` hanya bisa merender kartu generik bawaan, sekarang halaman pemanggil boleh menyuntikkan tampilan kartu sendiri (dipakai News untuk menampilkan thumbnail gambar).
  - [`frontend/src/components/shared/table/status.js`](frontend/src/components/shared/table/status.js) — Opsi filter baru `newsTypeOptions` (`news`/`article`).
  - [`backend/src/locales/{en,id}/translation.json`](backend/src/locales/id/translation.json), [`frontend/src/i18n/locales/{en,id}/translations.json`](frontend/src/i18n/locales/id/translations.json) — String baru untuk modul berita (label, pesan error, tipe konten).
- **Deskripsi Perubahan & Fungsi**:
  - Sebelumnya endpoint banner mobile app (`getBanner`) sudah ada tapi datanya kemungkinan diisi manual langsung ke database — tidak ada antarmuka admin untuk mengelolanya. Perubahan ini memberi tim admin halaman penuh untuk membuat, mengubah, dan menghapus konten berita/banner (dengan gambar) yang tampil di aplikasi mobile, lengkap dengan toggle tampil/sembunyikan (`show`) tanpa menghapus datanya.
  - Form create/edit terdiri dari: **Judul** (teks, wajib), **Konten** (rich text editor dengan format, tinggi kotak 200px), **Gambar** (wajib saat buat baru, opsional saat edit — mengganti gambar otomatis menghapus berkas lama dari storage), **Tipe** (pilihan `Berita`/`Artikel`, menentukan warna badge di daftar), dan **Tampilkan** (switch aktif/nonaktif — berita yang dimatikan tetap tersimpan, tidak terhapus, tinggal ditampilkan lagi kapan saja).
  - Halaman list punya dua mode tampilan: **grid** (kartu berisi thumbnail gambar, judul, badge tipe, tanggal dibuat, serta tombol edit/hapus — default saat halaman dibuka) dan **list/tabel** biasa (bisa dipilih lewat toggle tampilan yang sudah ada di komponen tabel bersama).
  - Peningkatan pada `Table.jsx`/`GridView.jsx` bersifat generik dan bisa dipakai ulang oleh halaman list lain yang butuh tampilan grid dengan kartu custom (gambar, bukan sekadar teks) — bukan perubahan khusus News semata.

### 📖 Informasi & Tutorial Singkat Fitur

- **Penjelasan Fitur**: Modul "Berita/Banner" adalah panel kelola konten yang tampil di halaman utama/banner aplikasi mobile pelanggan — sebelumnya konten ini tidak bisa diubah lewat aplikasi utama sama sekali (harus diedit langsung di database). Sekarang admin bisa mengelola sepenuhnya lewat UI: menambah berita/artikel baru dengan gambar, mengedit atau menyembunyikannya kapan saja, tanpa perlu akses database ataupun deploy ulang.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka sidebar **Aplikasi Seluler → Banner & Informasi**.
  2. Klik tombol **Tambah** di kanan atas.
  3. Isi **Judul**, tulis isi lengkap di kotak **Konten** (bisa diformat: bold, list, dsb), unggah **Gambar** (wajib untuk berita baru), pilih **Tipe** (Berita atau Artikel), dan atur switch **Tampilkan** sesuai kebutuhan (aktifkan agar langsung muncul di aplikasi mobile).
  4. Simpan — berita baru langsung muncul di daftar (tampilan kartu bergambar secara default).
  5. Untuk mengubah: klik ikon edit pada kartu/baris terkait, ganti field yang perlu (gambar lama otomatis terhapus jika diganti), simpan.
  6. Untuk menyembunyikan sementara tanpa menghapus: matikan switch **Tampilkan** lewat form edit.
  7. Untuk menghapus permanen: klik ikon hapus, konfirmasi pada modal yang muncul (turut menghapus berkas gambar dari penyimpanan).
