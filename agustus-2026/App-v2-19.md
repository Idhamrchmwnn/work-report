# 📝 Daily Work Report - Idham (2026-08-19)

---

## 📅 Laporan Harian - 19 Agustus 2026

---

## 📖 Dokumentasi: Panduan NOC Corporate

### 📌 Informasi

- **Berkas**: `docs system/tutorial/tutorial-noc-corporate.html`
- **Judul**: *"Panduan NOC Corporate — Dari Work Order sampai Layanan Aktif"*
- **Status**: Berkas baru/aktif dikerjakan hari ini (di luar repo git — folder `docs system/` belum ter-track oleh Git), ukuran ±82 KB

### 📅 Rincian Pekerjaan

- **Deskripsi**: Menyusun dokumen panduan internal (HTML, satu halaman) untuk tim **NOC Corporate**, menjelaskan alur bisnis dari pendaftaran prospek sampai layanan aktif, dengan fokus pada titik serah-terima dari Sales ke NOC lewat **Work Order**.
- **Cakupan isi** (5 bagian utama):
  1. **Alur Sebelum ke NOC** — pendaftaran → dijadikan Prospek → Quotation disetujui (PO terbit otomatis) → SO ditandatangani → Work Order dibuat.
  2. **WO Adalah Panduan Kerja Anda** — cara membaca isi Work Order dan mengubahnya menjadi tiket kerja.
  3. **Dari Tiket ke Pelanggan Aktif** — kelanjutan proses setelah tiket dibuat sampai pelanggan aktif.
  4. **Jalur Tanpa WO (Bypass)** — membuat tiket pemasangan langsung tanpa menunggu dokumen dari Sales.
  5. **Tiket Mitra & Bisnis** — menangani laporan gangguan dari pelanggan non-broadband yang sudah aktif.
- **Poin penting yang didokumentasikan**:
  - Perbedaan alur **Broadband (rumahan)** — ditangani NOC reguler — vs **Non-Broadband/Korporat** (Prospek → Quotation → PO → SO → Work Order → Tiket) yang jadi fokus panduan ini.
  - Penegasan bahwa **Pelanggan Bisnis** dan **Mitra Bisnis** adalah satu data yang sama di sistem (entitas Partner).
  - Penjelasan dua cabang independen setelah SO ditandatangani: konversi Prospek → Pelanggan (administratif) vs pembuatan Work Order (operasional, tanpa info harga) — dan bagaimana keduanya bertemu kembali di tiket NOC.
  - Catatan perilaku sistem: bila pembuatan PO otomatis gagal setelah tanda tangan SO, tanda tangan tetap sah dan proses tidak dibatalkan (hanya muncul peringatan).

---

## 📊 Dokumentasi: Katalog Flowchart NOC (`demo-noc-v2`)

### 📌 Informasi

- **Lokasi**: `docs system/demo-noc-v2/` (spesifikasi: `Framework_Flowchart_NOC.md`; hasil gambar: `drawio/*.drawio`, pratinjau `drawio/pratinjau/*.png`, ekspor final `png/*.png`)
- **Status**: Materi pendukung yang sudah ada (dibuat 11 Agustus 2026) — **belum pernah dicantumkan di laporan harian mana pun**, kini ikut didokumentasikan karena menjadi rujukan visual dari Panduan NOC Corporate yang sedang disusun hari ini.
- **Cakupan**: 24 flowchart (FC-01 s.d. FC-24), disusun ulang dari analisis kode langsung (bukan asumsi), meliputi 4 modul tanggung jawab NOC — **Pengguna, Layanan, Tiket, Jaringan**.

### 🧩 Inti Alur Bisnis (Prioritas Wajib)

- **FC-01 — Sistem Keseluruhan**: Diagram pembuka satu layar yang menunjukkan kedua jalur pelanggan (broadband & non-broadband) sekaligus titik percabangannya di halaman detail Registrasi. Swimlane 5 aktor (Sales, Calon Pelanggan, Admin/CS, NOC, Sistem).
- **FC-02 — Arsitektur Teknis**: Diagram C4-style yang menjelaskan kenapa ada dua server berbeda (Backend Node.js sebagai *control plane* vs `radiusd` Go sebagai *data plane*) yang sama-sama membaca/menulis MongoDB yang sama — kunci untuk mendiagnosis "masalah data" vs "masalah koneksi".
- **FC-03 — Aliran Data Antar Entitas**: DFD Level 1, fokus ke koleksi database mana melahirkan koleksi apa (bukan aksi manusia). Menandai 3 transformasi khusus: PO yang lahir otomatis, tiket-tutup-balik-ke-WO, dan gerbang ketat pembuatan Customer.
- **FC-04 — Proses Pelanggan Broadband**: Flowchart swimlane lengkap dari pendaftaran → survey → pemasangan → pembuatan pelanggan → kredensial RadAuth → aktif, dilanjut siklus operasional (keluhan, isolir, pelepasan, restore) sebagai loop.
- **FC-05 — Proses Pelanggan Non-Broadband**: Versi korporat FC-04 — dari prospek → Quotation → tanda tangan (PO otomatis lahir) → SO → **gerbang ganda** (konversi jadi Partner **dan** buat Work Order, dua cabang paralel independen) → tiket → BAA → layanan aktif.
- **FC-06 — Rantai Dokumen Non-Broadband**: Meluruskan 7 istilah dokumen yang sering tertukar (Quotation, PO, SO, WO, BAA, PKS, SDN) — mana yang berurutan/bergerbang (Blok A), mana yang independen ke Partner (Blok B: PKS & SDN), mana dokumen vendor (Blok C).

### 🧭 Journey Pelanggan

- **FC-07 — Journey Broadband**: Customer journey map 11 tahap dari sudut pandang pelanggan rumahan, menandai titik rawan seperti "menunggu tanpa notifikasi" sebagai titik keluhan tertinggi.
- **FC-08 — Journey Non-Broadband**: Versi korporat, 12 tahap — menyoroti kesalahpahaman umum di tahap PO ("pelanggan kira harus kirim PO sendiri, padahal otomatis").

### 🗺️ Peta Fungsi per Modul (mind-map, bukan alur proses)

- **FC-09 — Modul Pengguna**: Daftar fungsi Customer, Business, Partner, Document (PKS/SDN), Registration, Unsubscribe/Blacklist/Pasif, Admin/Employee/Privilege.
- **FC-10 — Modul Layanan**: Broadband, korporat (Data Access/Dedicated Internet), Hotspot, Vendor, Activation (BAA/PO/SO vendor), Prospect, Work Order.
- **FC-11 — Modul Tiket**: Menunjukkan 8 menu tiket berbagi satu siklus hidup yang sama (buka, ubah prioritas/penanggung jawab, tahan, tutup, batalkan) meski field buat/tutupnya beda-beda.
- **FC-12 — Modul Jaringan**: Sites, Fiber Cable (termasuk trace OTDR), NAS/Router, Radius Session, IPv4 — plus dua halaman "di luar menu" yang tetap tanggung jawab NOC (Dashboard Radius, Isolir Massal).

### 🔄 Diagram Status

- **FC-13 — Siklus Hidup Tiket**: State diagram yang meluruskan bahwa status tiket bukan satu nilai, tapi kombinasi 3 boolean (`complete`, `canceled`, `pending`) — termasuk jebakan bahwa tiket "dibatalkan" tetap ber-`complete: true`.
- **FC-14 — Status Registrasi & Prospect**: Dua state diagram berdampingan untuk siklus status pendaftaran (waiting→survey→review→...) dan status prospek (new→...→won/lost, dengan alur reopen).

### 📚 Tutorial Langkah-demi-Langkah (linear, satu kolom, wajib ada cabang gagal)

- **FC-15**: Membuat & menutup Tiket Survey (termasuk validasi format koordinat).
- **FC-16**: Membuat & menutup Tiket Pemasangan.
- **FC-17**: Membuat Pelanggan Broadband baru dari tiket pemasangan (gerbang: tiket harus valid & belum terpakai).
- **FC-18**: Mengaktifkan layanan/kredensial RadAuth-PPPoE, sampai verifikasi sesi tersambung.
- **FC-19**: Menangani Tiket Keluhan Pelanggan, termasuk mekanisme tahan/lanjutkan.
- **FC-20**: Memproses Dismantle — gerbang validasi perangkat sudah lepas semua + kombinasi opsi hapus yang sah.
- **FC-21**: Menjadikan Registrasi sebagai Prospek (titik masuk jalur non-broadband).
- **FC-22**: Mengonversi Work Order menjadi Tiket (khusus NOC) — langkah manual yang sering dikira otomatis.
- **FC-23**: Monitoring & kelola sesi Radius, termasuk troubleshoot pelanggan tidak bisa konek.
- **FC-24**: Trace gangguan fiber pakai OTDR — dari input jarak ukur sampai tiket backbone.

- **Keterkaitan dengan Panduan NOC Corporate**: FC-01 (peta alur lengkap), FC-05 (proses non-broadband/korporat), FC-06 (rantai dokumen Quotation→PO→SO→WO), FC-21 (Jadikan Prospek), dan FC-22 (WO ke Tiket) adalah diagram yang paling relevan sebagai ilustrasi visual dari isi teks panduan `tutorial-noc-corporate.html`.
- **Catatan isi penting** (dari `Framework_Flowchart_NOC.md`): setiap FC punya struktur identik — Identitas, Swimlane, Tabel Node (ID, bentuk, label persis), Tabel Panah, dan Rujukan kode — sehingga bisa langsung digambar ulang di draw.io/Lucidchart/Figma tanpa riset ulang. Sudah ada 24 berkas `.drawio` dan render `.png` final untuk seluruh katalog ini.

---

## 🌿 Branch: `issue-228` — Implementasi Dokumen SDN

### 📌 Status

- Tidak ada commit baru maupun perubahan kode tambahan hari ini pada branch ini.
- Perubahan yang sudah di-`stage` sejak laporan 18 Agustus (perbaikan validasi `partner_id`/`customer_id` pada `createSDN` & `listSO`, serta auto-fill `service_id`/`service_ordered` di form create SDN) **masih berstatus staged, belum di-commit** — tidak ada penambahan/pengurangan dari kondisi kemarin.
- Commit `2e63628` (`resolve #223`, membawa modul PKS ke branch ini) juga masih belum di-push ke `origin/issue-228`.

---

## ⚠️ Catatan Tambahan

- Fokus kerja hari ini bergeser ke **dokumentasi/pelatihan** (panduan NOC Corporate), bukan perubahan kode pada aplikasi utama.
- Folder `docs system/` (berisi panduan ini beserta materi demo NOC, tutorial teknisi/gudang, dan materi belajar lainnya) belum ter-track oleh Git — perlu diputuskan apakah akan dimasukkan ke repo (mis. sebagai submodule/folder docs) atau tetap dikelola terpisah.
- Item WIP dari kemarin (perbaikan SDN/SO & auto-fill) masih menunggu untuk di-commit dan di-push — belum ada progres baru hari ini pada item tersebut.
