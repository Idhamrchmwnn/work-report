# 📝 Daily Work Report - Idham (2026-09-11)

---

## 📅 Laporan Harian - 11 September 2026

---

## 🌿 Branch: `issue-279` — Hardening & Perbaikan Bug pada Aksi Perubahan Layanan

### 📌 Informasi Issue

- **Nomor Issue**: #279, sub-issue dari #277 "Integrasi dengan MobileApps (Android)"
- **Judul Issue**: #277 - Integrasi Request Perubahan Layanan
- **Status Branch**: `Belum di-merge` (sudah di-push ke `origin/issue-279`) — branch ini di-rebase hari ini ke atas rantai `issue-244 → issue-278 → #284 (Dedy)`, **menyelesaikan risiko konflik `mobileApp.js` yang dicatat di dua laporan sebelumnya** (item menu "Berita/Banner" dan "Perubahan Layanan" kini sudah dalam satu riwayat, di bawah root menu yang berganti nama jadi **"Integrasi Aplikasi"**). Catatan: branch `issue-280` (Notifikasi) **masih terpisah** dari rantai ini — risiko konflik `mobileApp.js` belum sepenuhnya tuntas.

### 📅 Rincian Perubahan

#### [2422c8e9] - resolve #279 - 11 September 2026, 16:07 WIB

Selain rebase, ada pekerjaan nyata hari ini: pengerasan (*hardening*) menyeluruh terhadap fitur accept/decline/review yang dilaporkan 9 September — sejumlah bug nyata ditemukan dan diperbaiki, kemungkinan hasil review kode.

- **Komponen yang Berubah**:
  - [`backend/src/models/radiusAuthentication.model.js`](backend/src/models/radiusAuthentication.model.js) — Index parsial baru: `{ req_change_at: -1 }` dengan `partialFilterExpression: { req_change: { $exists: true } }`. Tanpa ini, query daftar "Perubahan Layanan" (yang menyaring `req_change` lalu mengurutkan `req_change_at` menurun) jadi *collection scan* + *in-memory sort* di seluruh koleksi langganan — index parsial memastikan index hanya memuat baris yang benar-benar sedang punya permintaan (segelintir dari total).
  - [`backend/scripts/backfill-req-change-at.js`](backend/scripts/backfill-req-change-at.js) [NEW] — Script migrasi sekali-jalan: mengisi `req_change_at` untuk permintaan lama yang dibuat *sebelum* field ini ada (dari fitur 9 September). Tanpa dijalankan, permintaan-permintaan lama itu akan tampil "-" dan tenggelam ke bagian bawah daftar yang diurutkan terbaru-dulu — padahal justru yang **paling lama menunggu**. Diisi dari `created_at` dokumen sebagai perkiraan terbaik.
  - [`backend/src/services/mobileServiceChange.service.js`](backend/src/services/mobileServiceChange.service.js) [+281] — Beberapa bug fix penting:
    - **Kegagalan senyap diperbaiki**: `updateAuthenticationById` tidak pernah melempar error — kegagalan (dokumen hilang, validator menolak, dll.) dikembalikan sebagai objek `{error, message}`. Kemarin hasil ini tidak pernah dicek, jadi kalau update gagal, API tetap melaporkan "berhasil" ke admin padahal tidak ada satu field pun yang berubah. Helper baru `isAuthenticationUpdated()` menutup ini, mengembalikan kode `CONFLICT` (409) bila update ternyata tidak diterapkan.
    - **Bug crash (500) diperbaiki**: bila produk yang diminta pelanggan (`req_change`) sudah dihapus di antara waktu pelanggan mengajukan dan admin meninjau, `populate` menghasilkan `null` — mengakses `authentication.req_change._id` tanpa guard akan melempar `TypeError` yang jatuh sebagai 500. Sekarang diperiksa eksplisit, mengembalikan 404 yang jelas.
    - **Bug data hilang diperbaiki**: field `customer.partner` (dipakai frontend untuk memutuskan mengarahkan link ke halaman pelanggan biasa atau pelanggan-mitra) sebelumnya tidak ikut di-`populate` — pelanggan yang dimiliki mitra akan tertaut ke halaman yang salah. Ditambahkan `populate` bersarang untuk `customer.partner`.
    - **Catatan admin tidak lagi hilang saat menolak**: sebelumnya aksi *decline* menghapus `change_notes` sekaligus (`$unset`) tanpa jejak. Sekarang, bila admin mengisi catatan, catatan itu disalin dulu ke array riwayat (`history_change`, dengan tanggal & admin yang bertindak) sebelum field permintaan dibersihkan — alasan penolakan tidak hilang tanpa jejak.
    - **Catatan kosong ditangani benar**: catatan kosong sekarang di-*unset* (bukan disimpan sebagai string kosong), supaya tampilan lain yang membaca `change_notes` tetap jatuh ke pesan default, bukan menampilkan string kosong.
    - **Pencarian kolom relasi ditambahkan** di `listServiceChangeRequests` (controller) — kolom seperti Pelanggan/Profil/Produk berbasis `ObjectId` tidak bisa difilter langsung dengan regex; sekarang kata kunci pencarian diterjemahkan dulu jadi daftar `_id` yang cocok (pola yang sama dipakai modul Broadband) sebelum dikirim ke datatable.
  - [`backend/src/controllers/mobileServiceChange.controller.js`](backend/src/controllers/mobileServiceChange.controller.js) [+94] — Peta kode error → HTTP status diperluas (`NOT_FOUND`→404, `CONFLICT`→409, `VALIDATION`→400, sebelumnya cuma dua kondisi). Helper `resolveRelationFilter` untuk pencarian relasi di atas.
  - [`backend/src/routes/mobileServiceChange.route.js`](backend/src/routes/mobileServiceChange.route.js) — Endpoint respond diubah dari `POST` menjadi `PATCH` (lebih sesuai secara semantik REST untuk aksi mengubah status), privilege-nya diubah dari `mobileServiceChange.update` menjadi **`mobileServiceChange.changeStatus`** (penamaan yang lebih konsisten dengan modul lain, mis. `customerSDN.changeStatus`).
  - [`frontend/src/utils/setThisClass.js`](frontend/src/utils/setThisClass.js) — **Bug resiliensi diperbaiki**: fungsi ini sebelumnya melempar `Error` untuk nama warna yang tidak dikenal di *color map* — karena dipanggil saat render setiap item aksi baris tabel, satu typo nama warna dari pemanggil manapun akan meruntuhkan **seluruh tabel**, bukan cuma salah warna. Sekarang jatuh ke warna default (`primary`) dengan peringatan di console (khusus mode development), tidak pernah melempar.
  - [`frontend/src/app/pages/mobileApp/serviceChange/components/ReviewDrawer.jsx`](frontend/src/app/pages/mobileApp/serviceChange/components/ReviewDrawer.jsx) [271→335 baris], [`frontend/src/app/pages/mobileApp/serviceChange/schema/columns.jsx`](frontend/src/app/pages/mobileApp/serviceChange/schema/columns.jsx) [+133] — Disesuaikan mengikuti perubahan API (endpoint `PATCH`, pencarian kolom relasi, pesan error baru per kode).
  - [`frontend/src/app/pages/services/broadband/detail.jsx`](frontend/src/app/pages/services/broadband/detail.jsx) — Catatan admin saat menindaklanjuti permintaan (accept/decline) kini ikut **tampil di linimasa riwayat langganan pelanggan** di halaman detail Broadband — sebelumnya catatan ini cuma tersimpan di database tanpa terlihat di UI manapun.
  - [`backend/test/integration/mobileServiceChange.service.test.js`](backend/test/integration/mobileServiceChange.service.test.js) [NEW, 506 baris, 16 test case] — Mencakup seluruh bug fix di atas secara eksplisit: filter permintaan aktif, pencarian relasi, accept lengkap dengan pemetaan riwayat, penanganan produk terhapus (bukan 500), pelaporan CONFLICT saat update gagal diam-diam, decline dengan/tanpa catatan, review dengan catatan kosong, dan validasi umum.
- **Deskripsi Perubahan & Fungsi**:
  - Ini murni pekerjaan pengerasan kualitas atas fitur yang sudah dilaporkan fungsional 9 September — tidak ada kemampuan baru dari sisi pengguna, tapi menutup beberapa bug nyata (kegagalan senyap yang dilaporkan sebagai sukses, crash 500 pada data yang sudah dihapus, tautan pelanggan yang salah, catatan hilang tanpa jejak) yang berpotensi menyesatkan admin atau membuat aplikasi crash pada kondisi tepi tertentu.

---

## 🌿 Branch: `issue-291` — Endpoint Faktur/Tagihan Mitra pada Partner API

### 📌 Informasi Issue

- **Nomor Issue**: #291 (berdiri sendiri, tidak ada parent issue)
- **Judul Issue**: Endpoint Faktur dan Tagihan untuk Partner API
- **Status Branch**: `Belum di-merge` (branch baru, sudah di-push ke `origin/issue-291`)

### 📅 Rincian Perubahan

#### [2a78d715] - resolve #291 - 11 September 2026, 16:59:52 WIB

- **Komponen yang Berubah**:
  - [`backend/src/controllers/partnerApiInvoice.controller.js`](backend/src/controllers/partnerApiInvoice.controller.js) [NEW, 144 baris] — Dua endpoint, tidak ada model/service baru (memakai `financeInvoice.service.js`/`financePayment.service.js` yang sudah ada):
    - **`listPartnerAppInvoice`** (`GET /p-api/v1/invoices`) — daftar seluruh faktur mitra (lunas, belum bayar, refund, dibatalkan), tanpa pagination (klien menangani filter/sorting sisi mereka sendiri). `role: partner` otomatis dibatasi ke fakturnya sendiri; `role: admin` **wajib** menyertakan query `partner_id` (ditolak 400 bila tidak ada) — endpoint ini sengaja tidak dibuat untuk melihat lintas-mitra sekaligus (untuk itu pakai modul Finance Invoice di aplikasi admin).
    - **`readPartnerAppInvoice`** (`GET /p-api/v1/invoices/:id`) — detail satu faktur berdasarkan `invoice_id` publik, sekaligus rincian pembayarannya (metode, penyetor, tanggal, dll.) bila sudah dibayar. `role: partner` yang mencoba baca faktur milik mitra lain atau faktur non-partner (tipe pelanggan biasa) mendapat **404 generik** (tidak membocorkan bahwa `invoice_id` itu sebenarnya ada) — pola konsisten dengan endpoint Partner API lain yang sudah ada.
    - Kedua endpoint menerapkan `withPortalVisibility()`/mengecualikan mitra dengan `show_in_portal: false` untuk sesi admin — memakai ulang persis mekanisme visibilitas portal mitra yang sudah dibangun untuk modul Customer/Business sebelumnya (`getHiddenPortalPartnerIds`), bukan implementasi baru.
  - [`backend/src/routes/partnerApi.route.js`](backend/src/routes/partnerApi.route.js) [+167] — Registrasi 2 route di atas (`GET /invoices`, `GET /invoices/:id`), digerbang `protectedPartnerApp` (tanpa privilege internal tambahan — konsisten dengan seluruh route Partner API lain yang memang diautentikasi lewat token mitra/admin, bukan sistem privilege aplikasi utama), lengkap dokumentasi Swagger termasuk skema response.
  - [`backend/test/integration/partnerApiInvoice.test.js`](backend/test/integration/partnerApiInvoice.test.js) [NEW, 404 baris, 14 test case] — Cakupan cukup lengkap: mitra hanya lihat faktur sendiri, array kosong (bukan error) saat tidak ada faktur, admin wajib `partner_id` (400 tanpa itu), admin sukses dengan `partner_id`, 404 untuk mitra yang `show_in_portal: false`, 404 generik lintas-mitra, 404 untuk faktur non-partner, 401 tanpa token, serta **tidak crash (bukan 500) untuk input tidak wajar** pada parameter `:id`.
- **Deskripsi Perubahan & Fungsi**:
  - Sebelumnya, aplikasi eksternal mitra (yang mengakses lewat `/p-api/v1`) tidak punya cara melihat tagihan/faktur mereka sendiri sama sekali lewat API — modul faktur hanya bisa diakses dari aplikasi admin internal. Fitur ini membuka akses baca (read-only, tidak ada endpoint bayar/ubah) untuk mitra melihat riwayat tagihan dan status pembayarannya sendiri, serta memungkinkan admin memeriksa tagihan mitra tertentu lewat jalur yang sama.
  - Desain keamanannya mengikuti persis pola yang sudah mapan di endpoint Partner API lain: pemisahan cakupan berbasis peran token, 404 generik (bukan 403) untuk mencegah *enumeration*, dan pengecualian otomatis untuk mitra yang sedang disembunyikan dari portal — tidak ada pola baru yang perlu direview terpisah dari yang sudah ada.

---

## 📢 Ringkasan Dampak

| Branch | Dampak Utama |
| --- | --- |
| `issue-279` | Fitur accept/decline/review (9 September) diperkeras: kegagalan senyap, crash 500 pada data terhapus, tautan pelanggan salah, dan catatan admin hilang tanpa jejak — semuanya diperbaiki. Konflik `mobileApp.js` dengan `issue-278` selesai; dengan `issue-280` masih terbuka. |
| `issue-291` | Mitra kini bisa melihat faktur/tagihan miliknya sendiri lewat Partner API, memakai ulang pola keamanan visibilitas portal yang sudah ada. |
