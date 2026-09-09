# 📝 Daily Work Report - Idham (2026-09-09)

---

## 📅 Laporan Harian - 9 September 2026

---

## 🌿 Branch: `issue-279` — Aksi Terima/Tolak/Tinjau untuk Request Perubahan Layanan

### 📌 Informasi Issue

- **Nomor Issue**: #279, sub-issue dari #277 "Integrasi dengan MobileApps (Android)"
- **Judul Issue**: #277 - Integrasi Request Perubahan Layanan
- **Status Branch**: `Belum di-merge` (sudah di-push ke `origin/issue-279`) — menutup kekurangan yang dilaporkan kemarin (halaman kemarin masih murni read-only, hari ini sudah ada aksi penuh).

### 📅 Rincian Perubahan

#### [8d5c5cec] - resolve #279 - 9 September 2026, 17:37 WIB

- **Komponen yang Berubah**:
  - [`backend/src/models/radiusAuthentication.model.js`](backend/src/models/radiusAuthentication.model.js) — Field baru `req_change_at` (Date): mencatat kapan permintaan perubahan diajukan pelanggan, supaya admin bisa tahu sudah berapa lama permintaan itu menunggu.
  - [`backend/src/controllers/radiusAuthentication.controller.js`](backend/src/controllers/radiusAuthentication.controller.js) — `requestChangeSubscription` (endpoint mobile, sudah ada sebelumnya) kini ikut mengisi `req_change_at: new Date()` saat pelanggan mengajukan permintaan. `cancelChangeSubscription` ikut membersihkan field ini saat pelanggan membatalkan sendiri.
  - [`backend/src/services/radiusAuthentication.service.js`](backend/src/services/radiusAuthentication.service.js) — `findSubscription` (dipakai endpoint mobile untuk menampilkan status langganan ke pelanggan) meneruskan `req_change_at` sebagai `req_change.requestedAt` di response, supaya aplikasi mobile juga bisa menampilkan "diajukan sejak kapan".
  - [`backend/src/services/mobileServiceChange.service.js`](backend/src/services/mobileServiceChange.service.js) — Fungsi baru **`respondToServiceChangeRequest()`**, inti dari pekerjaan hari ini. Menangani 3 jenis respons admin:
    - **`accept`** — menerapkan perubahan: `bind_product` diganti sesuai produk yang diminta pelanggan (`req_change`), sedangkan `profile` (profil teknis RADIUS/kecepatan) **dipilih manual oleh admin** — karena permintaan pelanggan cuma berisi nama produk, bukan profil teknis yang harus dipetakan ke sana. Perubahan dicatat ke riwayat lewat `prepareHistoryChange` (util yang sudah ada, dipakai ulang), lalu seluruh field permintaan (`req_change`, `req_change_at`, `change_review`, `change_notes`) dibersihkan.
    - **`decline`** — membatalkan permintaan tanpa mengubah langganan pelanggan sama sekali, field permintaan dibersihkan.
    - **`review`** — hanya menandai `change_review: true` (opsional disertai catatan `change_notes`) untuk ditinjau lagi nanti; permintaan **tetap ada** di daftar (tidak dihapus/diproses) — komentar di kode menyebut ini setara badge "Pratinjau" pada aplikasi web versi lama.
  - [`backend/src/controllers/mobileServiceChange.controller.js`](backend/src/controllers/mobileServiceChange.controller.js) — Handler baru `respondServiceChangeRequest`: validasi `resp` harus salah satu dari `accept`/`decline`/`review`, panggil service di atas, terjemahkan kode error (`NOT_FOUND` → 404, lainnya → 400).
  - [`backend/src/routes/mobileServiceChange.route.js`](backend/src/routes/mobileServiceChange.route.js) — Endpoint baru `PATCH /api/v1/mobile-service-change/:authentication_id/respond`, digerbang privilege baru (lihat di bawah).
  - [`backend/src/config/privilege.json`](backend/src/config/privilege.json) — Grup `mobileServiceChange` bertambah aksi `respond` (sebelumnya cuma `list`).
  - [`frontend/src/app/pages/mobileApp/serviceChange/components/ReviewDrawer.jsx`](frontend/src/app/pages/mobileApp/serviceChange/components/ReviewDrawer.jsx) [NEW, 271 baris] — Drawer review: menampilkan data pelanggan, produk & profil aktif saat ini, produk yang diminta, `Combobox` untuk memilih profil RADIUS baru (`ajaxUrl="/broadband-profile/select"`, wajib diisi untuk aksi Terima), textarea catatan, dan tiga tombol aksi di footer — **Tolak** (merah), **Tinjau Ulang** (oranye), **Terima** (hijau).
  - [`frontend/src/app/pages/mobileApp/serviceChange/schema/columns.jsx`](frontend/src/app/pages/mobileApp/serviceChange/schema/columns.jsx) — Kolom aksi (tombol buka drawer review) dan kolom tanggal permintaan (`req_change_at`) ditambahkan ke tabel.
  - [`frontend/src/app/pages/mobileApp/serviceChange/index.jsx`](frontend/src/app/pages/mobileApp/serviceChange/index.jsx) — Merangkai state buka/tutup `ReviewDrawer` dan reload tabel setelah aksi berhasil.
  - [`frontend/src/components/shared/table/RowActions.jsx`](frontend/src/components/shared/table/RowActions.jsx) — **Peningkatan komponen bersama** (dipakai di seluruh aplikasi, bukan cuma halaman ini): item aksi custom pada dropdown baris tabel sekarang bisa diberi prop `color` opsional (memakai skema warna "this" yang sama dengan aksi delete bawaan) untuk menyorot aksi yang sifatnya destruktif/positif — tidak mengubah tampilan aksi custom lain yang tidak mengoper `color`.
  - [`backend/src/locales/{en,id}/translation.json`](backend/src/locales/id/translation.json), [`frontend/src/i18n/locales/{en,id}/translations.json`](frontend/src/i18n/locales/id/translations.json) — String baru untuk label tombol aksi, placeholder catatan, dan pesan error (`profileRequired`, `invalidResponse`, `failedRespond`, dll).
- **Deskripsi Perubahan & Fungsi**:
  - Kemarin halaman "Perubahan Layanan" baru sebatas menampilkan daftar permintaan (read-only) — saya catat sebagai kekurangan bahwa admin harus memproses secara manual lewat halaman Broadband terpisah. Hari ini kekurangan itu ditutup: admin sekarang bisa langsung **Terima**, **Tolak**, atau **Tinjau Ulang** setiap permintaan langsung dari drawer di halaman ini, tanpa berpindah halaman.
  - Poin desain penting: saat **Terima**, admin *wajib* memilih profil RADIUS secara manual (bukan otomatis) — karena permintaan pelanggan hanya menyebut nama produk yang diinginkan, sedangkan pemetaan produk ke profil teknis (kecepatan/limit) tetap butuh keputusan admin.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Kegunaan Fitur**: Melengkapi halaman monitoring kemarin menjadi alur kerja penuh — admin/CS tidak perlu lagi berpindah ke menu Broadband secara manual untuk memproses permintaan ganti paket pelanggan. Semua keputusan (terima dengan pemilihan profil teknis yang tepat, tolak, atau tandai untuk ditinjau lagi) bisa dilakukan dari satu tempat, dengan jejak riwayat perubahan otomatis tercatat.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka sidebar **Mobile App → Perubahan Layanan**.
  2. Klik tombol aksi pada baris pelanggan yang ingin diproses — drawer review terbuka menampilkan detail: produk & profil aktif saat ini, serta produk yang diminta pelanggan.
  3. **Untuk menyetujui**: pilih **Profil RADIUS** yang sesuai dengan produk baru tersebut lewat kolom pencarian profil (wajib diisi), tambahkan catatan bila perlu, lalu klik tombol hijau **Terima**. Sistem langsung mengubah produk & profil pelanggan dan mencatatnya ke riwayat.
  4. **Untuk menolak**: klik tombol merah **Tolak** — permintaan dibatalkan, langganan pelanggan tidak berubah sama sekali.
  5. **Untuk menunda keputusan**: isi catatan (opsional) lalu klik tombol oranye **Tinjau Ulang** — permintaan tetap muncul di daftar (ditandai untuk ditinjau), belum diputuskan, bisa diproses lagi kapan saja.
  6. Setelah aksi Terima/Tolak berhasil, baris tersebut otomatis hilang dari daftar (karena `req_change` sudah dibersihkan); aksi Tinjau Ulang membuat baris tetap tampil.
