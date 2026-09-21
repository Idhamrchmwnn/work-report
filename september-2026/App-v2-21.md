# 📝 Daily Work Report - Idham (2026-09-21)

---

## 📅 Laporan Harian - 21 September 2026

---

## 🌿 Branch: `issue-304` — Audit Keamanan & Finalisasi Kartu Informasi Mitra + Dokumen Mitra

### 📌 Informasi Issue

- **Nomor Issue**: #304 (berdiri sendiri)
- **Judul Issue**: Implementasi Kartu Informasi Mitra/POP
- **Status Branch**: **`resolve #304`** — 2 commit hari ini, keduanya bertanda `resolve` (bukan lagi `save`), menandai modul yang berjalan sejak 15 September ini akhirnya **dianggap selesai** setelah lewat sesi audit keamanan & pengerasan menyeluruh.
- Catatan penyaringan: branch ini di-rebase ke atas pekerjaan besar tim lain hari ini (termasuk subproyek baru `syslog-server` yang sama sekali tidak berkaitan) — laporan ini sudah disaring untuk hanya membahas perubahan nyata pada modul Kartu/Dokumen Mitra.

### 📅 Rincian Perubahan

#### [f19d4bec] - resolve #304 - 21 September 2026, 16:11 WIB

Sesi audit menemukan beberapa bug nyata (bukan sekadar kerapian kode) pada fitur yang sebelumnya dilaporkan "selesai" tanggal 18 September — konsisten dengan pola kerja: pekerjaan diklaim selesai, lalu diuji dengan sungguh-sungguh sebelum benar-benar ditandai `resolve`.

- **Komponen yang Berubah**:
  - [`backend/src/controllers/partnerApiPartner.controller.js`](backend/src/controllers/partnerApiPartner.controller.js) — **Bug data tidak konsisten**: skrip migrasi 18 September sengaja **tidak mengosongkan** `partner.documents[]` lama (sebagai jaring pengaman), tapi seluruh penulisan baru (unggahan lewat portal mitra maupun admin) sejak itu hanya mendarat di koleksi baru `partner_documents`. Akibatnya `GET /partners/profile` masih menampilkan daftar dokumen **sebelum migrasi** (beku), sementara `GET /partners/documents` menampilkan yang sebenarnya — dua jawaban berbeda untuk pertanyaan yang sama, tanpa satu pun error yang menandainya. Diperbaiki lewat `toPartnerProfileDTO()`, dipakai konsisten di `/partners/profile` dan `/partners/read/:id` supaya keduanya tidak lagi bisa menyimpang.
  - Bug yang sama juga menutup **kebocoran berkas** (*orphaned upload*): `setPartnerAppDocument` sebelumnya hanya membersihkan berkas yang terlanjur naik ke MinIO lewat cabang `if (!document)` — padahal fungsi penyimpanannya (`replacePartnerDocumentByType`) ternyata **melempar** saat database gagal ditulis (bukan mengembalikan nilai kosong), sehingga jalur pembersihan itu tidak pernah tereksekusi pada kegagalan sungguhan. Sekarang dibungkus `try/catch` dengan dua penanda status (`uploaded`, `documentPersisted`) supaya pembersihan berjalan tepat pada kondisi manapun, dan kegagalan yang bisa diprediksi dilaporkan sebagai 400 (bukan 500 generik).
  - [`backend/src/controllers/partnerDocument.controller.js`](backend/src/controllers/partnerDocument.controller.js) — **Celah otorisasi ditutup**: halaman admin "Dokumen Mitra" hanya menampilkan dokumen milik Mitra Bisnis (`reseller: true`), tapi operasi per-dokumen (ubah, ganti berkas, unduh, hapus) sebelumnya **tidak menegakkan batasan yang sama** — dokumen milik mitra di luar cakupan (mis. hasil migrasi dari mitra non-reseller) tetap bisa diakses langsung lewat id-nya. Ditutup lewat `loadDocumentInScope()`, dipanggil di setiap operasi tunggal, bukan cuma di daftar.
  - **Guard NoSQL injection**: `requirePartnerId()` menolak nilai `partner_id` yang bukan string (mis. objek seperti `{"$ne": null}`) **sebelum** menyentuh query Mongoose — tanpa ini, nilai semacam itu bisa diteruskan apa adanya sebagai operator filter dan mencocokkan mitra mana pun, bukan yang diminta.
  - **Pengurangan noise operasional**: id dokumen yang bukan `ObjectId` valid sebelumnya jatuh sebagai `CastError` Mongoose yang tercatat `logger.error` — dan menurut aturan internal ("AGENTS Bab 4.12") level log itu memicu alert Telegram ke tim ops. Input salah ketik dari pengguna (bukan kegagalan sistemik) sekarang ditolak lebih awal sebagai 400 biasa, tidak lagi membangunkan siapa pun.
  - [`backend/src/services/partner.service.js`](backend/src/services/partner.service.js) [-44, murni penghapusan] — Fungsi `updatePartnerDocument()` (push/pull ke array tertanam lama) dihapus total — sudah sepenuhnya digantikan modul Dokumen Mitra sejak 18 September, ini pembersihan kode mati yang tertinggal.
  - [`backend/test/integration/publicPartnerCard.test.js`](backend/test/integration/publicPartnerCard.test.js) [NEW, 283 baris, 16 test] — Cakupan keamanan yang sangat menyeluruh untuk halaman publik: token/id kartu/nama berkas storage tidak pernah bocor, bagian yang tidak diaktifkan PIC benar-benar tidak terkirim (bukan sekadar `null`), header anti-indeks/anti-cache terpasang, kartu draf/token asing/id salah format konsisten 404, kartu tercabut tetap menyebut identitas tapi tidak membuka isi, mitra nonaktif/dihapus otomatis membuat kartu tidak berlaku, **uji IDOR eksplisit** ("menolak dokumen milik mitra lain walau id-nya benar"), mematikan visibilitas dokumen benar-benar menutup akses berkasnya (bukan cuma menyembunyikan dari daftar), dan pencatatan scan tetap jalan meski kartu sudah tercabut.
  - [`backend/test/integration/partnerCard.invalidId.test.js`](backend/test/integration/partnerCard.invalidId.test.js) [NEW, 124 baris] — Menguji ketahanan endpoint print-log/print-batch terhadap id sampah di tengah daftar id yang valid, daftar kosong, dan daftar melebihi batas — semua ditolak 400 tanpa crash.
  - [`backend/test/integration/partnerApiPartner.profile.test.js`](backend/test/integration/partnerApiPartner.profile.test.js) [NEW] + `partnerApiPartner.uploadDocuments.test.js` [diperbarui] — Menguji langsung bahwa bug data-tidak-konsisten di atas benar-benar tertutup.
- **Deskripsi Perubahan & Fungsi**:
  - Tidak ada kemampuan baru dari sisi pengguna — murni audit keamanan & korektifitas atas fitur yang sudah dibangun sejak 15 September. Tiga bug nyata ditemukan dan ditutup: data yang bisa terlihat tidak sinkron antar dua endpoint, kebocoran berkas yatim di storage saat gagal, dan celah otorisasi yang membiarkan dokumen di luar cakupan tetap bisa diakses lewat id.
  - Rangkaian test baru (>400 baris) secara eksplisit menyasar kelas-kelas bug ini (IDOR, injection, kebocoran data), bukan sekadar menguji jalur bahagia — kualitas pengujian yang jauh melampaui rata-rata fitur lain di aplikasi ini.

---

## 📢 Ringkasan Dampak

Modul **Kartu Informasi Mitra** dan **Dokumen Mitra**, yang berjalan sejak 15 September, hari ini resmi ditandai selesai (`resolve #304`) setelah melalui audit yang menemukan dan menutup tiga bug nyata (inkonsistensi data, kebocoran berkas, celah otorisasi) plus penambahan cakupan test keamanan yang cukup ketat — siap untuk direview lebih lanjut sebelum di-merge.

---

## 📖 Penjelasan Bug & Dampaknya bagi Pengguna

Ketiga temuan hari ini tidak menambah tombol atau menu baru — dampaknya terasa lewat **hilangnya masalah yang sebelumnya diam-diam ada**. Berikut penjelasan tiap bug dengan bahasa yang lebih sederhana, termasuk skenario nyata yang sebelumnya bisa terjadi:

1. **Data dokumen mitra yang "beku" di satu halaman, padahal sudah berubah di halaman lain.**
   - *Skenario sebelum diperbaiki*: seorang mitra mengunggah KTP baru lewat portalnya sendiri. Halaman **Dokumen Mitra** (admin) langsung menampilkan KTP yang baru. Tapi kalau admin membuka **profil mitra tersebut lewat Partner API** (`/partners/profile`), yang muncul justru KTP versi **lama** — seolah-olah unggahan barusan tidak pernah terjadi. Tidak ada pesan error, jadi tidak ada yang sadar datanya berbeda sampai ada yang membandingkan langsung.
   - *Penyebab*: saat data dokumen dipindah ke tempat penyimpanan baru (18 September), data lama sengaja tidak langsung dihapus (untuk jaga-jaga), tapi satu titik kode lupa diarahkan untuk membaca dari tempat yang baru.
   - *Setelah diperbaiki*: kedua halaman/endpoint sekarang **selalu** membaca dari sumber data yang sama dan terkini — tidak mungkin lagi menampilkan dua jawaban berbeda untuk pertanyaan yang sama.

2. **Berkas "hantu" yang menumpuk di penyimpanan setiap kali unggahan gagal di saat yang tidak tepat.**
   - *Skenario sebelum diperbaiki*: mitra mengunggah dokumen baru untuk mengganti yang lama. Berkasnya berhasil terkirim ke server penyimpanan, tapi sesaat kemudian proses pencatatannya ke database gagal (mis. koneksi database sempat terputus). Berkas yang sudah terlanjur diunggah **tidak pernah dibersihkan** — tertinggal selamanya di storage tanpa ada yang merujuknya, memenuhi ruang penyimpanan pelan-pelan tanpa disadari.
   - *Setelah diperbaiki*: sistem sekarang selalu tahu persis di titik mana sebuah unggahan gagal, dan langsung membersihkan berkas yang tidak jadi terpakai — tidak ada lagi "sampah" yang menumpuk diam-diam.

3. **Dokumen yang seharusnya tidak boleh diakses, ternyata masih bisa dibuka lewat tautan langsung.**
   - *Skenario sebelum diperbaiki*: halaman Dokumen Mitra memang hanya *menampilkan* dokumen milik Mitra Bisnis di daftarnya. Tapi kalau seseorang sudah tahu (atau menebak) ID sebuah dokumen milik entitas lain yang seharusnya tidak muncul di daftar itu, ternyata dokumen itu **tetap bisa dibuka, diganti, bahkan dihapus** langsung lewat ID-nya — pintu belakang yang tidak dijaga sama seperti pintu depan.
   - *Setelah diperbaiki*: setiap kali dokumen diakses lewat ID langsung (bukan cuma lewat daftar), sistem sekarang memeriksa ulang apakah dokumen itu memang berada dalam cakupan yang diizinkan — kalau tidak, ditolak dengan pesan "tidak ditemukan", persis seperti perlakuan terhadap dokumen yang benar-benar tidak ada.

## 📚 Tutorial Ringkas — Rekap Alur Kartu Informasi Mitra & Dokumen Mitra (Fitur Final)

Karena hari ini modul ini resmi ditandai selesai, berikut rekap alur pemakaian dari awal sampai akhir untuk admin yang belum familiar (detail per-fitur lebih lengkap ada di laporan 15 dan 18 September):

1. **Kelola dokumen legalitas mitra dulu** — buka **Pengguna → Dokumen Mitra**, unggah/kelola KTP PIC, perjanjian sewa lokasi, kontrak kerja sama, izin lokasi, dan foto lokasi untuk tiap mitra. Setiap dokumen diberi label yang jelas (bukan nama berkas asli).
2. **Buat Kartu Informasi Mitra** — buka **Arsip → Kartu Informasi Mitra**, buat kartu baru untuk mitra yang belum punya (kartu berstatus "Belum Ada Kartu" sampai dibuat).
3. **Atur visibilitas** — di halaman detail kartu, tentukan bagian mana yang boleh tampil ke publik (nama resmi, alamat, area, koordinat, foto, kontak, dokumen, info perusahaan pengelola) — defaultnya hanya info lokasi dasar yang aktif, data pribadi PIC harus dinyalakan sadar.
4. **Pilih dokumen yang ditampilkan** — dari dokumen yang sudah diunggah di langkah 1, pilih maksimal 8 yang relevan untuk ditampilkan di kartu ini.
5. **Terbitkan kartu** — status berubah jadi Diterbitkan, QR/tautannya aktif.
6. **Cetak stiker** — satuan atau massal (8 stiker per lembar A4), lalu tempel di lokasi POP.
7. **Verifikasi lewat pemindaian** — siapa pun yang memindai QR melihat halaman ringkas sesuai bagian yang diaktifkan, termasuk penjelajah dokumen bergaya Google Drive untuk melihat pratinjau dokumen langsung.
8. **Cabut bila perlu** — begitu kemitraan berakhir atau ada masalah, cabut kartu dari halaman detail; stiker yang sudah tertempel otomatis berhenti menampilkan data tanpa perlu ditarik fisik.

**Yang berubah berkat perbaikan hari ini (tidak terlihat sebagai fitur baru, tapi memengaruhi keandalan seluruh alur di atas)**: data dokumen yang ditampilkan di mana pun sekarang dijamin selalu konsisten dan terkini, berkas yang gagal terunggah tidak lagi menumpuk sebagai sampah di penyimpanan, dan dokumen yang seharusnya di luar jangkauan benar-benar tidak bisa diakses lewat cara apa pun — termasuk saat halaman publik memindai dokumen.
