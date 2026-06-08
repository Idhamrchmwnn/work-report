# 📝 Daily Work Report - Idham (2026-06-08)

---

## 📌 Informasi Issue
- **Nomor Issue**: #104
- **Judul Issue**: Implementasi Modul Manajemen Vendor (Frontend)

## 📅 Laporan Harian - 8 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

---

- [`frontend/src/app/router/protected.jsx`](frontend/src/app/router/protected.jsx)
  - **Deskripsi**: Menambahkan import `vendorRoute` dan menyebarkannya (`...vendorRoute`) ke dalam array `protectedRoutes` di bagian grup Services. Perubahan ini menjadikan seluruh route vendor terdaftar di sistem routing aplikasi dan terlindungi oleh autentikasi admin.

- [`frontend/src/app/navigation/services.js`](frontend/src/app/navigation/services.js)
  - **Deskripsi**: Menambahkan entri navigasi sidebar baru untuk modul Vendor dengan ikon `MdOutlineStore`, path `/services/vendor`, translation key `nav.services.vendor`, dan privilege guard `vendor.list`. Item ini muncul di sidebar menu Services sehingga pengguna dapat mengakses modul Vendor dari navigasi utama.

- [`frontend/src/app/router/services/vendorRoute.jsx`](frontend/src/app/router/services/vendorRoute.jsx) [NEW]
  - **Deskripsi**: Mendefinisikan 4 route lazy-loaded untuk modul Vendor: halaman list (`/vendor`), halaman detail (`/vendor/view/:id`), halaman buat (`/vendor/create`), dan halaman edit (`/vendor/edit/:id`). Setiap route dilengkapi privilege handle untuk dikontrol oleh middleware privilege system yang sudah ada.

- [`frontend/src/app/pages/services/vendor/index.jsx`](frontend/src/app/pages/services/vendor/index.jsx) [NEW]
  - **Deskripsi**: Halaman utama daftar vendor yang menampilkan data dalam komponen `Datatables` dengan kolom dari `getVendorColumns`. Dilengkapi tombol "Tambah Vendor" yang hanya muncul jika pengguna memiliki privilege `vendor.create`. Menggunakan `useTranslation` untuk semua label UI.

- [`frontend/src/app/pages/services/vendor/schema/columns.jsx`](frontend/src/app/pages/services/vendor/schema/columns.jsx) [NEW]
  - **Deskripsi**: Mendefinisikan dua set kolom DataTable menggunakan `@tanstack/react-table`: `getVendorColumns` (untuk tabel list vendor — kolom status, kode, nama dengan link ke detail, kontak AM, kontak NOC, tanggal dibuat, dan aksi CRUD) dan `getVendorServiceColumns` (untuk tabel layanan vendor — kolom kode, nama, tipe, kapasitas, harga, SLA, status, tanggal kontrak berakhir, dan aksi edit/hapus). Kolom nama pada tabel vendor menggunakan komponen `OpenLink` yang menghormati privilege `vendor.read`.

- [`frontend/src/app/pages/services/vendor/create.jsx`](frontend/src/app/pages/services/vendor/create.jsx) [NEW]
  - **Deskripsi**: Halaman form pembuatan vendor baru menggunakan `react-hook-form` dengan validasi `vendorSchema` (Yup). Form terdiri dari dua kolom: kolom kiri berisi identitas vendor (nama, kode, status toggle) dan kolom kanan berisi informasi kontak (email & telepon Account Manager, email & telepon NOC, alamat, dan deskripsi). Setelah berhasil, menampilkan toast notifikasi dengan link ke halaman detail vendor yang baru dibuat.

- [`frontend/src/app/pages/services/vendor/edit.jsx`](frontend/src/app/pages/services/vendor/edit.jsx) [NEW]
  - **Deskripsi**: Halaman form edit vendor yang mengambil data existing via `GET /vendor/read/:id` saat halaman dimuat, kemudian mengisi form menggunakan `reset()` dari `react-hook-form`. Menampilkan skeleton loading selama data diambil. Struktur form identik dengan halaman create dan menggunakan validasi skema yang sama. Terdapat tombol navigasi ke halaman detail vendor terkait.

- [`frontend/src/app/pages/services/vendor/detail.jsx`](frontend/src/app/pages/services/vendor/detail.jsx) [NEW]
  - **Deskripsi**: Halaman detail vendor dengan layout dua kolom: kolom kiri menampilkan informasi umum vendor (kode, alamat, status, tanggal dibuat), kontak Account Manager, kontak NOC, dan deskripsi dalam komponen `Box` terpisah dengan skeleton loading. Kolom kanan menampilkan daftar Layanan Vendor dalam tabel inline yang mendukung CRUD: tambah layanan via `VendorServiceModal` (modal form untuk create & edit sekaligus), edit layanan, dan hapus layanan dengan konfirmasi `ConfirmModal`. Setiap aksi dikontrol privilege (`vendor.create`, `vendor.update`, `vendor.delete`).

- [`frontend/src/app/pages/services/vendor/schema/createSchema.js`](frontend/src/app/pages/services/vendor/schema/createSchema.js) [NEW]
  - **Deskripsi**: Mendefinisikan dua skema validasi Yup: `vendorSchema` (untuk form vendor — field `name` dan `code` wajib, `code` bersifat opsional saat mode edit via context `$isEdit`, field kontak dan alamat bersifat opsional) dan `vendorServiceSchema` (untuk form layanan vendor — field `vendor`, `name`, `type`, dan `price` wajib; `code` wajib hanya saat create; field kapasitas, SLA, tanggal kontrak, dan deskripsi bersifat opsional). Menggunakan `i18n.t()` untuk pesan error yang mendukung multi-bahasa.
