# 📝 Daily Work Report - Idham (2026-09-15)

---

## 📅 Laporan Harian - 15 September 2026

---

## 🌿 Branch: `issue-304` — Kartu Informasi Mitra/POP (Stiker QR Lokasi)

### 📌 Informasi Issue

- **Nomor Issue**: #304 (berdiri sendiri, tidak ada parent issue)
- **Judul Issue**: Implementasi Kartu Informasi Mitra/POP
- **Status Branch**: `Belum di-merge`, commit hari ini masih ditandai `save` (bukan `resolve`) — modul besar (33 file, 4544 baris), kemungkinan berlanjut ke hari berikutnya.

### 🧭 Konsep Fitur

**Kartu Informasi Mitra** adalah stiker QR yang ditempel fisik di lokasi POP (Point of Presence) milik mitra. Siapa pun yang memindainya (teknisi lapangan, petugas keamanan, aparat, dll.) membuka halaman publik tanpa login berisi profil mitra dan dokumen legalitas yang **dipilih eksplisit** oleh admin — untuk memverifikasi bahwa perangkat di lokasi itu legal dan siapa penanggung jawabnya.

### 📅 Rincian Perubahan

#### [a505ffd8] - save #304 - 15 September 2026, 17:29:53 WIB

- **Komponen yang Berubah**:

  **Backend — Model & Desain Keamanan** ([`backend/src/models/partnerCard.model.js`](backend/src/models/partnerCard.model.js) [NEW, 219 baris]):
  - Koleksi **terpisah** dari `Partner` (bukan field tambahan) — alasan didokumentasikan eksplisit di kode: token akses publik akan ikut ter-serialisasi setiap kali dokumen Partner dikembalikan lewat endpoint manapun (dan endpoint yang mengembalikan data Partner di aplikasi ini banyak — `partnerApi*`, select, datatable, export). Satu DTO yang lupa memfilter = token bocor. Koleksi terpisah membuat kebocoran karena kelalaian jadi mustahil.
  - **Token** dibuat dari `crypto.randomBytes(16)` (CSPRNG) — berbeda sengaja dari pola *share token* dokumen lain di aplikasi ini (`moment().format('MMYY') + randomString(20)`, berbasis `Math.random()`) — karena token ini tertempel fisik dan berlaku selama kemitraan berjalan (bukan sekali pakai ke satu penerima), ambang keamanannya beda.
  - **Tidak ada `expired_at`**: keabsahan kartu dihitung ulang setiap akses dari status kartu + keadaan mitra saat itu (aktif/nonaktif/dihapus) — begitu kemitraan berakhir, semua stiker mitra itu otomatis mati serentak tanpa perlu aksi manual menyinkronkan tanggal.
  - **`show_coordinate` default `false`**: yang memindai stiker sudah berdiri di lokasi (koordinat tidak menambah manfaat baginya) — tapi kalau tautannya tersebar di luar, menampilkan koordinat sama saja menerbitkan peta lokasi perangkat perusahaan ke publik.
  - Dua sumber dokumen per kartu: `partner` (merujuk berkas yang sudah ada di profil mitra, tidak disalin) atau `upload` (berkas baru khusus kartu ini) — maksimal 8 dokumen per kartu.
  - Index parsial unik menegakkan **1 mitra = 1 kartu aktif** (mitra yang kartunya pernah dihapus tetap bisa dibuatkan kartu baru).

  **Backend — Alur Kerja Admin** ([`backend/src/controllers/partnerCard.controller.js`](backend/src/controllers/partnerCard.controller.js) [NEW, 721 baris], [`backend/src/services/partnerCard.service.js`](backend/src/services/partnerCard.service.js) [NEW, 427 baris]):
  - Siklus status **`draft → published → revoked`**: kartu dibuat dulu sebagai draf (token belum aktif ke publik — diakses lewat token tetap 404 selama masih draf, supaya tidak percuma membocorkan keberadaan token yang belum pernah dicetak), admin memilih dokumen & menerbitkannya (`publishPartnerCard`), dan bisa dicabut kapan saja (`revokePartnerCard`) tanpa menghapus datanya.
  - **Privilege terpisah untuk data sensitif**: mengubah *apa yang tampil ke publik* (pilihan dokumen, tampil-tidaknya koordinat — `updatePartnerCardExposure`) digerbang privilege sendiri (`partnerCard.changeSensitive`), terpisah dari privilege ubah data biasa (nama tampilan, catatan publik).
  - **Urutan operasi file yang aman**: saat mengganti daftar dokumen kartu, berkas baru diunggah dulu, database diperbarui, **baru** berkas lama yang sudah tidak dirujuk dihapus — dan penghapusan hanya terjadi *setelah* penulisan database berhasil (bila update database gagal, berkas yang sudah terlanjur diunggah dibersihkan, bukan didiamkan jadi sampah).
  - Pelacakan `print_count`/`last_printed_at` (dicatat tiap kali admin mencetak) dan `scan_count`/`last_scanned_at` (dicatat tiap kali stiker dipindai publik) — juga endpoint `getPartnerCardPrintBatch` untuk mencetak banyak stiker sekaligus.

  **Backend — Halaman Publik** ([`backend/src/controllers/publicPartnerCard.controller.js`](backend/src/controllers/publicPartnerCard.controller.js) [NEW, 195 baris], tanpa autentikasi):
  - **4 status keabsahan**, bukan cuma aktif/nonaktif: `valid`, `revoked` (dicabut admin), `inactive` (mitra dinonaktifkan), `terminated` (mitra dihapus). Status tidak-valid **tetap** menampilkan identitas minimal (nama & ID mitra) — supaya orang yang menemukan stiker mati di lapangan tetap bisa melaporkan kartu mana yang dimaksud — tapi menyembunyikan dokumen & catatan.
  - **DTO publik whitelist ketat**: `token`, `_id`, nama berkas asli di storage, `created_by`, penghitung scan/cetak, dan seluruh data sensitif mitra (telepon, email, KTP, NPWP, saldo) **tidak pernah** ikut terkirim — didokumentasikan eksplisit di kode sebagai daftar "yang sengaja tidak dikirim".
  - Header keamanan `X-Robots-Tag: noindex, nofollow` dan `Cache-Control: private, no-store` — halaman ini memuat dokumen legalitas mitra, tidak boleh terindeks mesin pencari maupun tersimpan di cache bersama.
  - **Deduplikasi hitungan scan**: sidik jari `SHA-256(IP + User-Agent)` disimpan di Redis selama 1 jam — refresh halaman, prefetch browser, atau bot tidak menggelembungkan angka pemindaian. Pencatatan dilakukan *setelah* response terkirim tanpa `await` (fire-and-forget) — kegagalan Redis/MongoDB di sini tidak pernah menggagalkan halaman.
  - **Guard dokumen basi**: dokumen bersumber `partner` divalidasi ulang terhadap daftar dokumen mitra yang **sekarang** setiap kali diakses — karena dokumen itu bisa saja sudah diganti lewat portal mitra sejak kartu dipublikasikan (unggah ulang tipe yang sama menghapus berkas lama dari storage). Tanpa pengecekan ini, halaman publik akan menawarkan tautan yang berujung error penyimpanan.

  **Frontend — Admin** ([`frontend/src/app/pages/archive/partnerCard/`](frontend/src/app/pages/archive/partnerCard/) [NEW]):
  - `index.jsx` — tabel daftar kartu per mitra, kolom status (`PartnerCardStatusBadge` — status `none` khusus untuk mitra yang belum punya kartu sama sekali, warna netral, disebut sebagai status yang "paling sering dicari" untuk tahu siapa yang belum dibuatkan stiker).
  - `PartnerCardDrawer.jsx` [583 baris] — form kelola satu kartu: nama tampilan (override nama mitra), catatan publik, toggle tampilkan koordinat, dan `PartnerCardDocumentPicker.jsx` [273 baris] untuk memilih dokumen dari profil mitra atau mengunggah baru, dengan label yang wajib diisi manual admin (bukan diambil dari nama berkas asli — banyak entri lama cuma bernama `KTP.jpeg`, tidak layak tampil ke publik).
  - `PrintBatchDrawer.jsx` [208 baris] + `PrintSheet.jsx` — cetak stiker massal, tata letak **2 kolom × 4 baris per lembar A4** (perhitungan milimeter presisi didokumentasikan di kode: 2×95mm + gutter 5mm = 195mm lebar, muat dalam batas 200mm setelah margin).
  - [`frontend/src/components/shared/partnerCard/PartnerCardSticker.jsx`](frontend/src/components/shared/partnerCard/PartnerCardSticker.jsx) [91 baris] — **satu sumber tampilan** dipakai bersama oleh pratinjau di drawer, cetak satuan, dan cetak massal, memakai `qrcode.react` untuk merender QR langsung dari token di sisi klien (tidak disimpan sebagai gambar) — memastikan hasil cetak selalu identik dengan yang dilihat admin sebelum mencetak.
  - [`frontend/src/app/pages/public/PublicPartnerCard.jsx`](frontend/src/app/pages/public/PublicPartnerCard.jsx) [253 baris] — halaman publik yang dibuka saat stiker dipindai.
  - `PartnerCardProfileLink.jsx` — tautan cepat ke halaman kartu dari profil mitra (`frontend/src/app/pages/users/partner/profile.jsx` ikut disentuh untuk menyisipkan tautan ini).
  - Navigasi: item menu baru **"Kartu Informasi Mitra"** di bawah root **Arsip** (`/archive/partner-card`), berdampingan dengan "Ketetapan Direktur" (Dokumen Direktur yang dilaporkan 7 September, kini disebut ulang dengan nama itu — root arsip sebelumnya di `/arsip` kini `/archive`).
  - `ArchiveIndexRedirect.jsx` [NEW] — halaman indeks root Arsip yang mengarahkan ke sub-halaman yang sesuai privilege pengguna.

- **Deskripsi Perubahan & Fungsi**:
  - Memberi mitra bisnis (pemilik lokasi POP) identitas fisik terverifikasi di lapangan: satu stiker QR yang bisa dipindai siapa saja untuk memastikan sebuah perangkat/lokasi memang milik mitra sah dan melihat dokumen legalitasnya, tanpa perlu login atau menghubungi kantor pusat.
  - Kontrol penuh ada di tangan admin: dokumen apa yang boleh dilihat publik dipilih eksplisit per kartu (bukan otomatis menampilkan semua dokumen di profil mitra), dan bisa dicabut kapan saja tanpa menghapus riwayatnya.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Kegunaan Fitur**: Sebelumnya tidak ada cara terstandardisasi untuk memverifikasi keabsahan sebuah lokasi POP/perangkat mitra secara fisik di lapangan — mis. saat ada pemeriksaan dari pihak berwenang, warga sekitar bertanya, atau audit internal. Kartu Informasi Mitra menjawab ini dengan stiker QR yang tertempel permanen, terhubung ke halaman verifikasi yang datanya selalu terkini (bukan dokumen cetak yang bisa basi).
- **Langkah Penggunaan (Tutorial — Sisi Admin)**:
  1. Buka sidebar **Arsip → Kartu Informasi Mitra**, atau dari halaman profil mitra klik tautan cepat kartu.
  2. **Buat kartu baru** untuk mitra yang belum punya (status akan menampilkan "Belum Ada Kartu").
  3. Isi **Nama Tampilan** (opsional, bila nama dagang POP berbeda dari nama badan hukum mitra) dan **Catatan Publik** bila perlu.
  4. Di bagian **Dokumen**, pilih dari dokumen yang sudah ada di profil mitra (beri label yang jelas, bukan nama berkas asli) atau unggah dokumen baru khusus kartu ini — maksimal 8 dokumen.
  5. Putuskan apakah **koordinat lokasi** perlu ditampilkan di halaman publik (default tidak, karena pemindai sudah berdiri di lokasi — hanya aktifkan bila ada alasan khusus).
  6. Klik **Terbitkan** — kartu berubah status jadi Diterbitkan, tautan/QR-nya kini aktif dan bisa diakses publik.
  7. **Cetak stiker**: cetak satuan dari drawer kartu, atau gunakan **Cetak Massal** untuk mencetak banyak kartu sekaligus dalam satu lembar A4 (8 stiker per lembar).
  8. Tempelkan stiker hasil cetak di lokasi POP fisik.
  9. Bila kemitraan berakhir atau ada masalah, klik **Cabut** — stiker yang sudah tertempel otomatis tidak lagi menampilkan dokumen/data (cukup menyebut identitas kartu), tanpa perlu menariknya secara fisik.
- **Sisi Publik (siapa pun yang memindai stiker)**: membuka halaman ringkas berisi nama & ID mitra, alamat, dan dokumen legalitas yang dipilih admin — tanpa perlu login, tanpa data sensitif (kontak, KTP/NPWP, saldo) yang ditampilkan.
