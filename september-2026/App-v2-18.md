# 📝 Daily Work Report - Idham (2026-09-18)

---

## 📅 Laporan Harian - 18 September 2026

---

## 🌿 Branch: `issue-304` — Modul Dokumen Mitra Mandiri & Perluasan Kartu Informasi Mitra

### 📌 Informasi Issue

- **Nomor Issue**: #304 (berdiri sendiri)
- **Judul Issue**: Implementasi Kartu Informasi Mitra/POP
- **Status Branch**: `Belum di-merge`, 2 commit hari ini dengan nama sama `save #304` (`5ff643ca`, `f14ad9d9`)

### 📅 Rincian Perubahan

Hari ini terdiri dari dua pekerjaan besar yang saling berurutan: pagi memisahkan Dokumen Mitra jadi modul sendiri, sore mendesain ulang sistem visibilitas Kartu Informasi Mitra supaya memakai modul baru itu sekaligus jadi jauh lebih fleksibel dari rencana awal.

#### Bagian 1 — Modul "Dokumen Mitra" Berdiri Sendiri (commit `0674206c`/`5ff643ca`)

- **Latar belakang** (didokumentasikan eksplisit di kode): dokumen legalitas mitra (KTP PIC, perjanjian sewa, kontrak kerja sama, dst.) sebelumnya tersimpan sebagai array tertanam `partner.documents[]` — tanpa id stabil, tanpa label, tanpa tanggal unggah, tanpa jejak siapa yang mengunggah. Halaman baru yang dibutuhkan (tabel dokumen yang bisa difilter/diurutkan/dipaginasi) tidak mungkin dibangun di atas array tertanam, karena `utils/data-table.js` (dipakai seluruh tabel di aplikasi ini) hanya bisa membungkus `Model.find()` pada koleksi biasa.
- **Komponen yang Berubah**:
  - [`backend/src/models/partnerDocument.model.js`](backend/src/models/partnerDocument.model.js) [NEW] — Koleksi baru `PartnerDocument`. Berkas fisik **tidak ikut dipindah** — tetap di bucket `documentPartner` dengan nama objek yang sama, sehingga seluruh tautan unduh lama tetap hidup. Field `source` (`admin` vs `partner_app`) membedakan dokumen yang diunggah staf dari yang diunggah mitra sendiri lewat portalnya.
  - [`backend/src/controllers/partnerDocument.controller.js`](backend/src/controllers/partnerDocument.controller.js) [NEW, 363 baris] + [`backend/src/services/partnerDocument.service.js`](backend/src/services/partnerDocument.service.js) [NEW, 310 baris] + [`backend/src/routes/partnerDocument.route.js`](backend/src/routes/partnerDocument.route.js) [NEW, 250 baris] — CRUD penuh (list/create/update/delete) untuk halaman admin baru.
  - [`backend/src/utils/migrate-partner-documents.js`](backend/src/utils/migrate-partner-documents.js) [NEW] — Script migrasi satu-jalan: memindahkan seluruh entri `partner.documents[]` lama ke koleksi baru tanpa menyentuh berkas fisiknya.
  - [`frontend/src/app/pages/users/partnerDocument/`](frontend/src/app/pages/users/partnerDocument/) [NEW] — Halaman admin baru "Dokumen Mitra" di menu **Pengguna** (`index.jsx`, `UploadDocumentDrawer.jsx`, `EditDocumentDrawer.jsx`, `schema/columns.jsx`) — tabel penuh dengan filter, label, jenis dokumen, sumber, dan tanggal unggah per baris.
  - [`backend/src/controllers/partnerCard.controller.js`](backend/src/controllers/partnerCard.controller.js) [-360 baris bersih] — Logika penanganan dokumen yang tadinya ditulis inline di modul Kartu Informasi Mitra dihapus, digantikan pemanggilan ke modul Dokumen Mitra yang baru — Kartu tidak lagi menyimpan/mengelola berkas sendiri, hanya **menunjuk** dokumen yang sudah ada di modul ini.
- **Deskripsi Perubahan & Fungsi**: Dokumen legalitas mitra sekarang punya rumah sendiri yang layak — halaman tabel penuh di menu Pengguna, terlepas dari fitur Kartu Informasi Mitra manapun yang kelak memakainya. Ini juga membuka jalan bagi fitur berikutnya di hari yang sama (lihat Bagian 2).

#### Bagian 2 — Kartu Informasi Mitra: dari Satu Saklar Koordinat, jadi Sistem Visibilitas Penuh (commit `f14ad9d9`)

- **Perubahan desain paling signifikan hari ini**: kemarin (15 September) kontrol privasi kartu cuma satu — `show_coordinate` (boolean tunggal). Hari ini digeneralisasi total jadi `visible_fields`, daftar bagian halaman publik yang bisa dinyalakan/dimatikan PIC **satu per satu**: `legal_name`, `partner_type`, `address`, `area`, `coordinate`, `photo`, `member_since`, `phone` (ditandai eksplisit "DATA PRIBADI"), `email` (juga "DATA PRIBADI"), `documents`, dan `company` (blok "Dioperasikan Oleh", memakai info perusahaan dari `fetchCompanyInfo`).
  - **Default konservatif by design**: kartu baru hanya menyalakan `legal_name`, `address`, `area`, `documents`, `company` — field yang menjelaskan **lokasi**, bukan **orang**. Koordinat, foto, dan terutama kontak pribadi (telepon/email) mitra harus dinyalakan sadar oleh PIC, tidak ikut menyala begitu saja.
  - **Ditegakkan di backend, bukan cuma UI**: field yang tidak ada di `visible_fields` sebuah kartu **tidak pernah** ikut dikirim oleh DTO publik sama sekali — bukan sekadar disembunyikan di tampilan, karena halaman publik bisa dibuka siapa saja dan isi respons API terbaca lewat *view-source*/devtools.
  - **Dirancang untuk berkembang**: menambah bagian baru ke depannya cukup satu baris konstanta, satu entri i18n, satu cabang DTO — komentar di kode menegaskan tidak ada daftar yang perlu disinkronkan manual di banyak tempat.
- **Komponen yang Berubah**:
  - [`backend/src/models/partnerCard.model.js`](backend/src/models/partnerCard.model.js) — `PARTNER_CARD_VISIBLE_FIELDS`/`PARTNER_CARD_DEFAULT_VISIBLE_FIELDS` baru, `show_coordinate` dihapus digantikan `visible_fields`. Skema dokumen kartu disederhanakan jadi murni **referensi** ke `PartnerDocument` (bukan salinan label/jenis/berkas) — nama dokumen kini cuma diketik sekali, di satu tempat.
  - [`backend/src/controllers/publicPartnerCard.controller.js`](backend/src/controllers/publicPartnerCard.controller.js) [+89] — DTO publik dirombak mengikuti `visible_fields`, helper `shows('company')` dkk. per bagian.
  - [`frontend/src/app/pages/archive/partnerCard/detail.jsx`](frontend/src/app/pages/archive/partnerCard/detail.jsx) [NEW, 571 baris] — Kartu sekarang punya **halaman detail penuh** (bukan cuma drawer): breadcrumb, pratinjau stiker, form edit, tombol cetak, salin tautan, buka di tab baru, dan pengelolaan dokumen langsung dari sini.
  - [`frontend/src/app/pages/archive/partnerCard/PartnerCardVisibilityFields.jsx`](frontend/src/app/pages/archive/partnerCard/PartnerCardVisibilityFields.jsx) [NEW] — Komponen toggle per-bagian, merender daftar `visible_fields` sebagai checklist, dengan penanda visual khusus untuk field berkategori data pribadi.
  - [`frontend/src/app/pages/archive/partnerCard/PartnerCardDrawer.jsx`](frontend/src/app/pages/archive/partnerCard/PartnerCardDrawer.jsx) [680→jauh lebih ringkas] — Tanggung jawabnya dipecah ke `detail.jsx` dan `PartnerCardVisibilityFields.jsx` di atas.
  - [`frontend/src/app/pages/public/partnerCard/`](frontend/src/app/pages/public/partnerCard/) [NEW] — **Penjelajah dokumen bergaya Google Drive** untuk halaman publik: `PartnerDocumentBrowser.jsx` (toggle tampilan petak/daftar, diingat per-peramban lewat `localStorage` — disebut di komentar kode berguna untuk "petugas yang memeriksa banyak lokasi" berulang kali), `PartnerDocumentThumbnail.jsx`, `PartnerDocumentPreview.jsx` (pratinjau dokumen langsung di halaman, tanpa perlu unduh/pindah tab).
  - [`frontend/src/app/pages/public/PublicPartnerCard.jsx`](frontend/src/app/pages/public/PublicPartnerCard.jsx) [+307/-x] — Halaman publik dirombak total merender bagian-bagian sesuai `visible_fields` yang dikembalikan API, memasang komponen penjelajah dokumen baru di atas.
- **Deskripsi Perubahan & Fungsi**:
  - Kartu Informasi Mitra yang kemarin baru punya kontrol privasi sangat sederhana (tampilkan koordinat atau tidak) sekarang jadi **pusat kelola tampilan publik yang lengkap** — PIC bisa memutuskan persis informasi apa yang boleh dilihat siapa pun yang memindai stiker, dengan pengaman berlapis di sisi server dan bukan sekadar sembunyi tampilan.
  - Pengalaman melihat dokumen di halaman publik naik kelas dari sekadar daftar tautan unduh menjadi penjelajah dengan pratinjau langsung — relevan untuk petugas lapangan yang perlu memverifikasi dokumen cepat tanpa mengunduh berkas satu per satu.

---

## 📖 Informasi & Tutorial Singkat Fitur

### 1. Dokumen Mitra (halaman admin baru)

- **Kegunaan**: Sebelumnya dokumen legalitas mitra (KTP PIC, perjanjian sewa lokasi, kontrak kerja sama, izin lokasi, foto lokasi) hanya bisa dilihat sebagai daftar sederhana di dalam profil mitra — tidak bisa dicari, difilter, atau diurutkan, dan tidak jelas siapa yang mengunggah atau kapan. Halaman baru ini memberi tim admin satu tempat terpusat untuk mengelola seluruh dokumen dari semua mitra sekaligus, layaknya tabel data pada umumnya di aplikasi ini.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka sidebar **Pengguna → Dokumen Mitra**.
  2. Tabel menampilkan seluruh dokumen dari semua mitra — bisa dicari/difilter per mitra, jenis dokumen, atau sumber unggahan (staf admin vs mitra sendiri lewat portalnya).
  3. Klik **Unggah Dokumen** untuk menambah dokumen baru: pilih mitra tujuan, beri **label** yang jelas dan mudah dibaca (bukan nama berkas asli seperti "KTP.jpeg"), pilih jenis dokumen bila relevan, lalu unggah berkasnya.
  4. Untuk mengubah label/jenis atau mengganti berkas, klik dokumen yang bersangkutan lalu **Edit**.
  5. Dokumen yang diunggah di sini otomatis tersedia untuk dipilih saat menyusun **Kartu Informasi Mitra** (lihat bagian berikut) — tidak perlu diunggah ulang.

### 2. Kartu Informasi Mitra — Pengaturan Visibilitas per Bagian

- **Kegunaan**: Kemarin, satu-satunya kontrol privasi kartu adalah tampil/tidaknya koordinat lokasi. Hari ini kartu jadi jauh lebih rinci: PIC/admin bisa menentukan **persis bagian mana** dari profil mitra yang boleh dilihat siapa pun yang memindai stikernya — termasuk memilih untuk tidak menampilkan data pribadi (telepon, email) sama sekali bila tidak diperlukan. Ini penting karena halaman yang dibuka lewat stiker bersifat publik tanpa login — sekali stiker tertempel di lokasi, siapa pun yang lewat bisa memindainya.
- **Langkah Penggunaan (Tutorial — Sisi Admin)**:
  1. Buka **Arsip → Kartu Informasi Mitra**, lalu klik kartu mitra yang ingin diatur (kini punya halaman detail sendiri, bukan cuma drawer kecil).
  2. Pada bagian **Visibilitas**, centang/hilangkan centang tiap bagian yang boleh tampil publik: Nama Resmi, Jenis Mitra, Alamat, Area/Kota/Provinsi, Koordinat, Foto, Bermitra Sejak, Telepon, Email, Dokumen, dan blok "Dioperasikan Oleh".
  3. Bagian yang berkaitan dengan data pribadi (Telepon, Email) ditandai secara visual berbeda — perhatikan baik-baik sebelum mengaktifkannya, karena begitu kartu diterbitkan dan stikernya tersebar, siapa pun yang punya tautannya bisa melihat bagian yang diaktifkan.
  4. Secara default, kartu baru hanya menampilkan info lokasi (nama resmi, alamat, area, dokumen, dan info perusahaan pengelola) — cukup untuk kebutuhan verifikasi dasar tanpa membocorkan data pribadi PIC.
  5. Di bagian **Dokumen**, pilih dokumen mana saja (dari yang sudah diunggah lewat halaman Dokumen Mitra di atas) yang ikut ditampilkan — maksimal 8 dokumen per kartu.
  6. Simpan — perubahan visibilitas langsung berlaku begitu kartu berstatus Diterbitkan; tidak perlu mencetak ulang stiker karena QR-nya tidak berubah.
- **Sisi Publik (yang memindai stiker)**: halaman yang terbuka sekarang menampilkan dokumen dalam **penjelajah bergaya Google Drive** — bisa beralih tampilan petak/daftar, dan tiap dokumen bisa dilihat langsung (pratinjau) tanpa perlu diunduh dulu. Pilihan tampilan (petak/daftar) diingat otomatis oleh peramban untuk kunjungan berikutnya.

---

## ⚠️ Catatan Tambahan

- Modul Dokumen Mitra baru (Bagian 1) dan redesain visibilitas Kartu (Bagian 2) terjadi **di hari yang sama** — menunjukkan Bagian 1 kemungkinan besar dipicu langsung oleh kebutuhan Bagian 2 (kartu perlu menunjuk dokumen yang punya identitas stabil, bukan entri array tanpa id). Keduanya erat terkait, disarankan direview sebagai satu kesatuan, bukan dua PR terpisah.
- Karena field `documents` pada skema Kartu berubah bentuk (dari salinan metadata menjadi referensi murni ke `PartnerDocument`), kartu yang sempat dibuat kemarin (15 September, bila ada yang sudah dites di lingkungan manapun) kemungkinan perlu diperiksa ulang kompatibilitas datanya sebelum modul ini dianggap stabil.
