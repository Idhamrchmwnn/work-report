# 📝 Daily Work Report - Idham (2026-08-12)

---

## 📅 Laporan Harian - 12 Agustus 2026

---

## 🌿 Branch: `master` — Customer SDN: Struktur Data & CRUD Dasar

### 📌 Informasi Issue

- **Nomor Issue**: Tidak tercantum di GitHub Issues (dikerjakan langsung di branch `master`, tanpa branch `issue-XXX` terpisah)
- **Judul Issue**: Membangun fondasi dokumen **Customer SDN** — *Service Delivery Notification*, mulai dari struktur data hingga operasi dasar buat/lihat/ubah/hapus (CRUD) di sisi admin
- **Status Branch**: `Sudah di-commit ke master` (local, `ahead of origin/master` — belum di-push)

### 📅 Rincian Commit

#### [c3fca9e] - "save sdn" - Selasa, 12 Agustus 2026 14:43

- **Komponen yang Berubah (bagian struktur data & CRUD)**:
  - `backend/src/models/customerSDN.model.js` [NEW]
  - `backend/src/services/customerSDN.service.js` [NEW]
  - `backend/src/controllers/customerSDN.controller.js` [NEW]
  - `backend/src/routes/customerSDN.route.js` [NEW]
  - `backend/src/app.js`
  - `backend/src/config/privilege.json`
  - `backend/src/locales/id/translation.json`
  - `backend/src/locales/en/translation.json`
  - `frontend/src/app/pages/users/customerSDN/schema/customerSDNSchema.js` [NEW]
  - `frontend/src/app/pages/users/customerSDN/create.jsx` [NEW]
  - `frontend/src/app/pages/users/customerSDN/edit.jsx` [NEW]
  - `frontend/src/app/pages/users/document/schema/sdnColumns.jsx` [NEW]
  - `frontend/src/app/pages/users/document/index.jsx`
  - `frontend/src/i18n/locales/id/translations.json`
  - `frontend/src/i18n/locales/en/translations.json`

---

## 🖥️ BACKEND

### Ringkasan

Membangun fondasi data dan operasi dasar (CRUD) untuk dokumen Customer SDN, mengikuti pola arsitektur model-service-controller-route yang sudah dipakai di modul dokumen customer lain (PO/SO/PKS).

- **`backend/src/models/customerSDN.model.js`** [NEW]
  - Mendefinisikan skema Mongoose `CustomerSDNSchema` dengan koleksi MongoDB `customer_sdn`.
  - Field utama mencakup relasi ke `Partner` (wajib) dan opsional ke `CustomerSO`, nomor & tanggal SDN, detail proyek (`project_name`, `service_id`, `service_ordered`), rentang layanan (`start_date`/SOD, `end_date`/EOD, `start_billing_date`), spesifikasi teknis (`circuit_type`, `bandwidth`, `sla`), serta detail lokasi A-End dan B-End lewat sub-skema `SiteSchema` (nomor rack/room, nama gedung, alamat).
  - Menyediakan field status (`status`, `approval`, `approved_at`, `complete`, `sent_at`, `share_token`) sebagai dasar siklus hidup dokumen yang akan dipakai pada tahap pengembangan berikutnya.
  - Menggunakan plugin `mongoose-delete` untuk *soft delete* dan plugin custom `autoIncrementPlugin` untuk penomoran otomatis (`seq`) sebagai dasar nomor SDN berurutan.
  - Field `topology_image` menyimpan nama file gambar topologi jaringan yang diunggah bersama dokumen.

- **`backend/src/services/customerSDN.service.js`** [NEW]
  - Berisi fungsi-fungsi akses data dasar: `findCustomerSDNById` (pencarian berdasarkan ObjectId **atau** nomor SDN), `findAllCustomerSDNForTable` (integrasi ke komponen DataTable server-side dengan populate relasi `partner`, `created_by`, `approval`), `createNewCustomerSDN`, `updateCustomerSDNData`, `deleteCustomerSDNById` (soft-delete), dan `nextCountCustomerSDN` (mengambil nomor urut berikutnya untuk format nomor dokumen).
  - Semua query di-scope dengan `pid: 'master'` mengikuti konvensi multi-tenant yang sudah dipakai modul lain.
  - Pesan error diambil dari sistem i18n (`i18nx()`) sehingga otomatis mengikuti bahasa aktif pengguna.

- **`backend/src/controllers/customerSDN.controller.js`** [NEW]
  - `createSDN`: memvalidasi keberadaan partner, mewajibkan `project_name`, menghubungkan ke Customer SO jika `so` (ID) diberikan (otomatis mengambil `so_number`/`so_date` dari SO terkait), memproses upload gambar topologi lewat helper `storeTopologyImage` (validasi ekstensi JPG/PNG/WEBP dan ukuran maksimal 10 MB), lalu men-generate nomor dokumen dengan format `SDN/{partner_id}/{urutan}/{bulan}/{tahun}`.
  - `listAllSDN` & `readSDN`: menyediakan data untuk tabel admin dan detail dokumen.
  - `updateSDN`: memperbarui field-field dokumen (termasuk mengganti gambar topologi), dibatasi hanya untuk dokumen yang belum berstatus final.
  - `deleteSDN`: melakukan soft-delete pada dokumen.
  - Seluruh handler dibungkus `asyncHandler` agar error async otomatis diteruskan ke error-handling middleware Express, konsisten dengan controller lain di proyek.

- **`backend/src/routes/customerSDN.route.js`** [NEW]
  - Mendaftarkan endpoint REST dasar: `POST /customer-sdn/create`, `POST /customer-sdn/list-all`, `GET /customer-sdn/view/:id`, `PATCH /customer-sdn/update/:id`, `DELETE /customer-sdn/delete/:id`.
  - Setiap route dilindungi middleware `protectedAdmin` (autentikasi) dan `checkPrivilege` dengan hak akses granular (`customerSDN.create/read/update/delete`).
  - Dilengkapi dokumentasi Swagger/OpenAPI per endpoint (request body, parameter, response code).

- **`backend/src/app.js`**
  - Mendaftarkan `CustomerSDNRoute` ke Express app pada prefix `/api/v1`, mengaktifkan endpoint SDN yang sudah dibuat di server.

- **`backend/src/config/privilege.json`**
  - Menambahkan grup privilege baru `customerSDN` (`CUSTOMERSDN_CREATE`, `CUSTOMERSDN_READ`, `CUSTOMERSDN_UPDATE`, `CUSTOMERSDN_DELETE`, dan `CUSTOMERSDN_CHANGESTATUS`) yang bisa diatur per role admin di halaman Pengaturan Hak Akses.

- **`backend/src/locales/id/translation.json`** & **`backend/src/locales/en/translation.json`**
  - Menambahkan pesan-pesan dasar pada namespace `customer.sdn` yang dipakai operasi CRUD: `failedGetList`, `notFound`, `documentNotFound`, `cannotEdit`, `editFailed`, `deleteFailed`, `topologyInvalidType`, `topologyTooLarge`.

### Dampak Backend

Endpoint CRUD dasar untuk Customer SDN sudah bisa dipakai: admin dapat membuat, melihat daftar, melihat detail, mengubah, dan menghapus dokumen SDN beserta lampiran gambar topologi jaringan, dengan validasi upload yang ketat dan pembatasan akses lewat privilege granular.

---

## 💻 FRONTEND

### Ringkasan

Membangun form dan tampilan dasar untuk mengelola dokumen SDN dari sisi admin, diintegrasikan sebagai tab baru di halaman **Document** yang sudah ada.

- **`frontend/src/app/pages/users/customerSDN/schema/customerSDNSchema.js`** [NEW]
  - Skema validasi form berbasis Yup: mewajibkan `partner_id` dan `project_name`, sementara field lain (SO terkait, tanggal, spesifikasi teknis, detail lokasi A-End/B-End lewat sub-skema `siteSchema`) bersifat opsional. Pesan error diambil dari i18n.

- **`frontend/src/app/pages/users/customerSDN/create.jsx`** [NEW] (540 baris)
  - Drawer form untuk membuat SDN baru, memakai `react-hook-form` + `yupResolver`. Menyediakan pemilihan Partner via `Combobox`, pemilihan Customer SO terkait secara opsional (auto-fill nomor/tanggal SO), input detail proyek & spesifikasi layanan, input detail lokasi A-End/B-End, serta area upload gambar topologi dengan validasi klien (tipe JPG/PNG/WEBP, maksimal 10 MB), lengkap dengan pratinjau gambar dan estimasi ukuran file (`formatFileSize`).

- **`frontend/src/app/pages/users/customerSDN/edit.jsx`** [NEW] (519 baris)
  - Drawer form untuk mengedit SDN yang masih bisa diubah, me-reuse skema dan sebagian besar UI dari `create.jsx`, dengan tambahan logika memuat data existing (termasuk gambar topologi saat ini) dan opsi mengganti gambar.

- **`frontend/src/app/pages/users/document/schema/sdnColumns.jsx`** [NEW]
  - Definisi kolom tabel DataTable untuk daftar SDN: tanggal dibuat, nomor SDN, nama proyek, partner (dengan link ke halaman partner/bisnis), nomor SO terkait, dan kolom aksi edit/hapus, masing-masing dengan pengecekan privilege granular.

- **`frontend/src/app/pages/users/document/index.jsx`**
  - Menambahkan tab baru **"SDN"** di halaman Document (sebelumnya hanya berisi tab PKS), lengkap dengan tabel daftar dokumen, tombol **Buat SDN** (dibatasi privilege `customerSDN.create`), dan drawer create/edit terpasang pada tab tersebut.

- **`frontend/src/i18n/locales/id/translations.json`** & **`en/translations.json`**
  - Menambahkan string UI dasar pada namespace `customer.sdn`: judul, label form (nama proyek, service ID, bandwidth, SLA, tanggal mulai/berakhir, detail lokasi A-End/B-End), teks upload topologi, dan pesan sukses buat/ubah dokumen.

### Dampak Frontend

Admin sudah bisa membuat, melihat, mengubah, dan menghapus dokumen SDN langsung dari tab baru di halaman Document, lengkap dengan validasi form dan unggah gambar topologi jaringan.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| — (master) | Fondasi Customer SDN — struktur data & CRUD | Model data, endpoint CRUD, dan form admin untuk mengelola dokumen SDN |

### Kemampuan Baru Pengguna/Admin

- Admin dapat membuat dokumen SDN baru dari tab **Document → SDN**, mengaitkannya secara opsional dengan Customer SO yang sudah ada, serta melampirkan gambar topologi jaringan.
- Admin dapat melihat daftar, membuka detail, mengubah, dan menghapus dokumen SDN yang sudah dibuat.

### Bug Fix / Solusi Masalah

- Tidak ada perbaikan bug pada sesi ini — murni pengembangan fitur baru.

### Menu/Fitur Baru

- Tab **"SDN"** baru di halaman **Document**.
- Endpoint CRUD dasar untuk Customer SDN di backend.
- Privilege granular baru (`customerSDN.create/read/update/delete/changeStatus`) di halaman Pengaturan Hak Akses.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Customer SDN (Service Delivery Notification) adalah dokumen yang mencatat detail layanan yang dipesan pelanggan — nama proyek, spesifikasi teknis (bandwidth, SLA, tipe sirkuit), rentang waktu layanan, dan detail lokasi A-End/B-End — sebagai data dasar sebelum dokumen ini nantinya melalui proses persetujuan dan dikirim ke partner.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka menu **Document**, pilih tab **SDN**, klik **Buat SDN**.
  2. Pilih Partner, isi nama proyek (wajib), opsional kaitkan ke Customer SO terkait, lengkapi detail layanan dan lokasi A-End/B-End, unggah gambar topologi jaringan bila ada.
  3. Simpan dokumen — dokumen akan tersimpan dan muncul di daftar tabel SDN.
  4. Dokumen yang sudah dibuat dapat diubah lewat tombol **Edit** atau dihapus lewat tombol **Hapus** pada tabel, selama belum berstatus final.
