# 📝 Daily Work Report - Idham (2026-07-03)

---

## 📌 Informasi Issue
- **Nomor Issue**: #126 & #118
- **Judul Issue**: #126 — Bug Fix Privilege Key Names yang Tidak Konsisten · #118 — Finalisasi & Merge Modul Sales Order (SO) ke Master

## 📅 Laporan Harian - 3 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Tidak ada pekerjaan WIP — seluruh perubahan hari ini sudah ter-commit. Working tree bersih.

---

### 📅 Rincian Commit

---

#### [b71ff917] - resolve #126 (#126 — Bug Fix Privilege Key Names)

Commit kecil namun berdampak besar pada sistem kontrol akses: sejumlah komponen frontend menggunakan nama privilege yang tidak sesuai dengan yang terdaftar di `privilege.json` backend, menyebabkan fitur tersembunyi secara salah meskipun admin memiliki akses.

- **Komponen yang Berubah**:

  - `frontend/src/app/pages/services/broadbandProfile/detail.jsx`
    - **Deskripsi**: Komponen `BroadbandProfileDetailDrawer` menggunakan `useHasPrivilege('profileBroadband.read')` — nama ini tidak ada di `privilege.json`. Diubah ke `'broadbandProfile.read'` yang benar. Akibatnya, badge dan tautan detail Broadband Profile di halaman tabel sebelumnya tidak pernah muncul bagi siapapun, termasuk admin superuser.

  - `frontend/src/app/pages/services/dedicatedInternet/index.jsx`
    - **Deskripsi**: Halaman daftar Dedicated Internet menggunakan `useHasPrivilege('dedicated.create')` untuk memunculkan tombol "Tambah". Diubah ke `'dedicatedInternet.create'` sesuai definisi privilege. Akibatnya, tombol tambah Dedicated Internet tidak pernah muncul walau admin sudah memiliki privilege tersebut.

  - `frontend/src/app/pages/tickets/installation/detail.jsx`
    - **Deskripsi**: Dua perbaikan dalam satu file:
      1. `useHasPrivilege('installationTicket.update')` → `'ticketInstallation.update'` untuk guard tombol update tiket instalasi
      2. `useHasPrivilege('dedicated.create')` → `'dedicatedInternet.create'` untuk guard tombol tambah layanan Dedicated Internet dari halaman detail tiket instalasi

  - `frontend/src/app/pages/tickets/payment/detail.jsx`
    - **Deskripsi**: `useHasPrivilege('ticketPayment.close')` → `'ticketPayment.update'`. Privilege `ticketPayment.close` tidak terdaftar di sistem; aksi penutupan tiket pembayaran dilindungi oleh `ticketPayment.update`. Akibatnya, admin tidak dapat menutup tiket pembayaran walau memiliki privilege yang benar.

  - `frontend/src/app/router/services/dedicatedInternetRoute.jsx`
    - **Deskripsi**: Empat guard `handle.privilege` di definisi route Dedicated Internet dikoreksi:
      - `'dedicated.read'` → `'dedicatedInternet.read'` (halaman detail)
      - `'dedicated.update'` → `'dedicatedInternet.update'` (halaman edit, ×2)
      - `'dedicated.create'` → `'dedicatedInternet.create'` (halaman create)
      Dengan guard route yang salah, sistem `AuthGuard` memblokir akses halaman edit dan detail Dedicated Internet bahkan untuk admin yang seharusnya punya izin.

- **Tabel Ringkasan Koreksi**:

  | File | Privilege Lama (Salah) | Privilege Baru (Benar) |
  |------|------------------------|------------------------|
  | `broadbandProfile/detail.jsx` | `profileBroadband.read` | `broadbandProfile.read` |
  | `dedicatedInternet/index.jsx` | `dedicated.create` | `dedicatedInternet.create` |
  | `tickets/installation/detail.jsx` | `installationTicket.update` | `ticketInstallation.update` |
  | `tickets/installation/detail.jsx` | `dedicated.create` | `dedicatedInternet.create` |
  | `tickets/payment/detail.jsx` | `ticketPayment.close` | `ticketPayment.update` |
  | `dedicatedInternetRoute.jsx` | `dedicated.read/update/create` | `dedicatedInternet.read/update/create` |

---

#### [8c0ede88] - resolve #118 (#118 — Vendor Sales Order)

Commit finalisasi modul Sales Order (SO) yang digabungkan ke branch utama — 35 file, 4508 baris ditambahkan. Ini adalah versi `resolve` (siap produksi) dari `save #118` yang sebelumnya di-commit pada 1–2 Juli 2026, dengan minor penyesuaian pada `ReviewSOPage.jsx`. Modul SO kini memiliki fitur setara penuh dengan Purchase Order (PO).

- **Komponen yang Berubah**:

  **Backend — Dependensi**:
  - `backend/package.json` — Tambah `pdf-lib: ^1.17.1` sebagai dependensi baru untuk manipulasi dokumen PDF di sisi server
  - `backend/package-lock.json` (+50 baris) — Update lock file

  **Backend — Utilitas PDF**:
  - `backend/src/utils/pdf-sign.js` [NEW] (+73 baris)
    - **Deskripsi**: Modul utilitas PDF yang bersih dan terdokumentasi penuh, berisi dua fungsi:
      1. **`getPdfPageCount(pdfBuffer)`** — Memuat buffer PDF menggunakan `pdf-lib`, membaca jumlah halaman, dan mengembalikan angka (minimal 1). Digunakan oleh backend untuk memberi tahu frontend berapa halaman PDF agar pengguna dapat memilih halaman yang tepat untuk meletakkan tanda tangan.
      2. **`stampSignatureOnPdf(pdfBuffer, signatureBuffer, position)`** — Fungsi inti embedding tanda tangan ke PDF. Alur kerja:
         - Konversi gambar tanda tangan ke PNG (via `sharp`) agar transparansi terjaga
         - Embed PNG ke dokumen PDF menggunakan `pdf-lib`
         - Terapkan koordinat posisi yang sudah dinormalisasi (nilai 0..1) dengan konversi sistem koordinat: frontend menggunakan titik acuan pojok kiri-atas, sedangkan spesifikasi PDF menggunakan pojok kiri-bawah, sehingga perlu transformasi `drawY = pageHeight - y * pageHeight - drawHeight`
         - Gambar tanda tangan di posisi yang tepat dan simpan PDF hasil
    - Parameter `position` menyimpan `page`, `x`, `y`, `width`, `height` — semua dalam nilai 0..1 relatif terhadap ukuran halaman, sehingga posisi tanda tangan tidak bergantung pada resolusi atau zoom level saat penandatanganan.

  **Backend — Model**:
  - `backend/src/models/vendorSO.model.js` (+111 baris, perbaruan besar)
    - **Deskripsi**: Model SO diperbarui dengan tambahan sub-schema dan field baru:
      - **`DocumentFileSchema`**: Sub-schema untuk berkas dokumen SO yang diunggah — menyimpan `name` (nama file), `file` (path di MinIO), `size` (ukuran byte), `mime` (MIME type), dan `pages` (jumlah halaman PDF, default 1).
      - **`SignaturePositionSchema`**: Sub-schema posisi tanda tangan dengan koordinat dinormalisasi `page`, `x`, `y`, `width`, `height` — semua bernilai 0..1.
      - Field baru pada schema utama: `document` (DocumentFileSchema, wajib), `approved_at` (Date), `approval` (ObjectId ref Admin), `complete` (Boolean), `signed_file` (String, path file bertanda tangan di MinIO), `signature` (String, data base64/path gambar tanda tangan), `signature_position` (SignaturePositionSchema), `signed_at` (Date), `signed_by` (ObjectId ref Admin).
      - Field `so_number` menjadi `unique: true` dengan auto-increment melalui plugin `autoIncrementPlugin`.

  **Backend — Service Layer**:
  - `backend/src/services/vendor.service.js` (+207 baris)
    - **Deskripsi**: Tambah seluruh service function untuk operasi CRUD dan lifecycle SO:
      - `findVendorSOById(id)` — Cari SO berdasarkan `_id`, populate relasi `vendor` dan `approval` (admin)
      - `createNewVendorSO(data)` — Buat SO baru dengan validasi dan auto-increment nomor
      - `updateVendorSOData(id, data)` — Update data SO (nomor, tanggal, catatan, dokumen)
      - `findAllSOsForTable(params)` — Query tabel SO dengan filter, pagination, dan sorting untuk DataTable
      - `findSOsByVendor(vendorId)` — Ambil semua SO milik vendor tertentu untuk ditampilkan di halaman detail vendor
      - `approveVendorSO(id, adminId)` — Set `approval = adminId` dan `approved_at = now`
      - `rejectVendorSO(id)` — Soft delete SO yang ditolak
      - `deleteVendorSOById(id)` — Soft delete SO
      - `findVendorSOByToken(token)` — Cari SO berdasarkan share token (untuk akses publik tanpa login)
      - `signVendorSO(id, adminId, signatureData, signedFilePath, position)` — Set `complete = true`, simpan data tanda tangan, path file bertanda tangan, dan posisi tanda tangan
      - `nextCountVendorSO()` — Ambil counter berikutnya untuk nomor SO auto-increment

  **Backend — Controller Vendor (SO)**:
  - `backend/src/controllers/vendor.controller.js` (+279 baris)
    - **Deskripsi**: Tambah controller SO yang lengkap sebagai bagian dari `vendor.controller.js`:
      - `createSO` — Validasi dan simpan SO baru; proses unggah file dokumen SO (max 15 MB), ekstrak metadata (nama, ukuran, MIME, jumlah halaman PDF), simpan ke MinIO
      - `listSO` — Daftar SO milik vendor tertentu (untuk tab SO di halaman detail vendor)
      - `listAllSO` — Daftar semua SO dari semua vendor (untuk halaman Aktivasi tab SO)
      - `readSO` — Detail SO tunggal dengan populate lengkap
      - `submitSO` — Kirim SO ke admin untuk direview (mengubah status menjadi submitted)
      - `requestSOPreview` — Generate/regenerasi link publik SO untuk dibagikan ke vendor
      - `approveSO` — Setujui SO; set `approval` dan `approved_at`
      - `rejectSO` — Tolak SO; hapus dari sistem
      - `deleteSO` — Hapus SO (dengan cek guard jika sudah disetujui)
      - `updateSO` — Update data SO termasuk penggantian dokumen
      - `storeSODocument` (internal) — Handler internal upload, validasi, dan penyimpanan file dokumen SO ke MinIO

  **Backend — Controller Publik SO**:
  - `backend/src/controllers/publicSO.controller.js` [NEW] (+154 baris)
    - **Deskripsi**: Controller khusus untuk endpoint publik yang tidak memerlukan autentikasi — digunakan vendor untuk mengakses dan menandatangani SO:
      - `toPublicSODTO(so)` — Fungsi mapper: mengubah dokumen SO Mongoose menjadi objek DTO publik yang aman (hanya field yang diperlukan vendor: `_id`, `so_number`, `so_date`, `notes`, `vendor`, `document`, `signed_file`, `signature`, `signature_position`, `complete`, `signed_at`, `signed_by`)
      - `getSOByToken(req, res)` — Ambil data SO berdasarkan share token; validasi token dan status approval sebelum data dikirim — SO yang belum disetujui tidak dapat diakses publik
      - `getSOFileByToken(req, res)` — Sajikan file dokumen SO (PDF/gambar) melalui share token; jika SO sudah ditandatangani (`complete = true`), kembalikan `signed_file`, jika belum kembalikan `document.file` asli
      - `signSOByToken(req, res)` — Proses tanda tangan vendor: ambil gambar tanda tangan yang dikirim, gunakan `stampSignatureOnPdf` untuk embed ke PDF, simpan file bertanda tangan ke MinIO, update SO dengan data tanda tangan dan path file baru

  **Backend — Routes**:
  - `backend/src/routes/vendor.route.js` (+315 baris) — Tambah 10 route baru untuk SO: `POST /vendor-so`, `GET /vendor-so`, `GET /vendor-so/:id`, `PATCH /vendor-so/:id`, `PATCH /vendor-so/approve/:id`, `PATCH /vendor-so/reject/:id`, `PATCH /vendor-so/submit/:id`, `POST /vendor-so/preview/:id`, `DELETE /vendor-so/:id`, `GET /vendor-so/page-count/:filename` (ambil jumlah halaman PDF)
  - `backend/src/routes/files.route.js` (+39 baris) — Tambah route akses file SO: `GET /file/vendor-so/:filename`
  - `backend/src/routes/public.route.js` (+102 baris) — Route publik SO: `GET /public/so/:token` (data SO), `GET /public/so/:token/file` (file dokumen), `POST /public/so/:token/sign` (proses tanda tangan)

  **Backend — Notifikasi & Terjemahan**:
  - `backend/src/utils/telegram.js` (+47 baris) — Tambah notifikasi Telegram untuk event SO: SO baru dibuat (ke channel admin), SO disetujui, SO ditolak, SO ditandatangani vendor
  - `backend/src/locales/en/translation.json` (+30 baris) — Pesan error/sukses SO: `notFound`, `alreadyApproved`, `alreadySigned`, `cannotReject`, `documentNotFound`, `pageCount`
  - `backend/src/locales/id/translation.json` (+29 baris) — Terjemahan ID untuk pesan SO yang sama

  **Frontend — Dependensi**:
  - `frontend/package.json` — Tambah `react-pdf: ^10.4.1` untuk render PDF di browser
  - `frontend/package-lock.json` (+391 baris) — Update lock file

  **Frontend — Konfigurasi & Utilitas Baru**:
  - `frontend/src/configs/pdf.config.js` [NEW] (+13 baris)
    - **Deskripsi**: File konfigurasi terpusat untuk `react-pdf` (pdfjs). Mengatur `pdfjs.GlobalWorkerOptions.workerSrc` ke `pdfjs-dist/build/pdf.worker.min.mjs` menggunakan `new URL(..., import.meta.url)` agar bundler (Vite) dapat mengelola worker dengan benar. Juga mengimpor CSS layer anotasi (`AnnotationLayer.css`) dan layer teks (`TextLayer.css`) yang wajib ada agar PDF dirender dengan lengkap. File ini di-export `Document` dan `Page` dari `react-pdf` sehingga komponen lain cukup `import { Document, Page } from 'configs/pdf.config'` tanpa perlu setup ulang worker.
  - `frontend/src/utils/formatFileSize.js` [NEW] (+11 baris)
    - **Deskripsi**: Fungsi utilitas `formatFileSize(bytes)` yang mengubah ukuran file dalam satuan byte ke teks yang mudah dibaca (B, KB, MB, GB) dengan satu desimal. Menggunakan logaritma basis 1024 untuk menentukan unit yang tepat. Digunakan pada form upload SO untuk menampilkan ukuran file yang dipilih pengguna.

  **Frontend — Model Schema Validasi**:
  - `frontend/src/app/pages/services/vendorManagement/schema/vendorSOSchema.js` [NEW] (+11 baris)
    - **Deskripsi**: Skema validasi Yup untuk form SO — memvalidasi field wajib (`so_number`, `so_date`, `vendor`) dan field opsional (`notes`, `total_override`).

  **Frontend — Halaman Modul Sales Order**:
  - `frontend/src/app/pages/services/salesOrder/create.jsx` (+273 baris)
    - **Deskripsi**: Halaman form pembuatan SO baru. Fitur: pilih vendor (Combobox dengan search), isi nomor SO dan tanggal, tambah catatan, upload dokumen SO (validasi tipe file PDF/image dan ukuran maksimum 15 MB menggunakan `formatFileSize` untuk tampilkan ukuran). Setelah berhasil menyimpan, redirect ke halaman detail SO.
  - `frontend/src/app/pages/services/salesOrder/edit.jsx` (+283 baris)
    - **Deskripsi**: Halaman form edit SO — mengambil data SO yang ada via API, mengisi form dengan data yang sudah ada, mendukung penggantian dokumen SO dengan file baru, serta validasi dan submit.
  - `frontend/src/app/pages/services/salesOrder/SODocumentPreview.jsx` (+83 baris)
    - **Deskripsi**: Komponen preview dokumen SO inline — menampilkan PDF atau gambar SO dalam iframe yang ter-embed di dalam drawer atau halaman detail. Mendukung loading state dan fallback jika dokumen tidak tersedia.

  **Frontend — Vendor Management (Tab SO)**:
  - `frontend/src/app/pages/services/vendorManagement/detail.jsx` (+261 baris)
    - **Deskripsi**: Halaman detail vendor diperbarui dengan penambahan tab **Sales Order**: menampilkan tabel daftar SO milik vendor tersebut (nomor SO, tanggal, status), tombol "Buat SO" yang mengarahkan ke `salesOrder/create`, dan aksi per baris (lihat detail, edit, hapus).

  **Frontend — Halaman Publik SO**:
  - `frontend/src/app/pages/public/PublicSODocument.jsx` (+406 baris)
    - **Deskripsi**: Halaman viewer dokumen SO yang dapat diakses publik tanpa login menggunakan share token dari URL. Menampilkan informasi lengkap SO (nomor, tanggal, vendor, catatan), daftar item, dan dokumen PDF/gambar menggunakan `react-pdf` untuk render PDF interaktif langsung di browser. Jika SO sudah ditandatangani (`complete = true`), menampilkan dokumen versi bertanda tangan.
  - `frontend/src/app/pages/public/ReviewSOPage.jsx` (+274 baris)
    - **Deskripsi**: Halaman review SO internal untuk admin yang memiliki privilege `vendor.changeStatus`. Menampilkan detail SO beserta dokumen preview, dengan tombol Approve dan Reject yang dilindungi `ConfirmModal`. Berbeda dari `PublicSODocument` yang untuk vendor eksternal, `ReviewSOPage` adalah untuk admin yang mengakses via URL langsung (misalnya dari notifikasi Telegram). Menggunakan `LinkBadge` untuk menampilkan link share publik SO.

  **Frontend — Halaman Aktivasi (Tab SO)**:
  - `frontend/src/app/pages/services/activation/index.jsx` (+93 baris)
    - **Deskripsi**: Halaman Aktivasi diperluas dengan tab **SO** yang sejajar dengan tab BAA dan PO. Tambahan:
      - State: `isSOReviewOpen`, `selectedSO`, `isSOProcessing`, `isSOPreviewOpen`, `selectedSOPreview`
      - Handler: `handleSOReview`, `handleCloseSODrawer`, `handleSOApprove`, `handleSOReject`, `handleSOViewDocument`, `handleCloseSOPreview`
      - Komponen: `SOReviewDrawer` + `DocumentPreviewModal` untuk tab SO
  - `frontend/src/app/pages/services/activation/components/SOReviewDrawer.jsx` (+425 baris)
    - **Deskripsi**: Drawer review SO lengkap yang dibuka dari halaman Aktivasi. Fitur:
      - Tampilkan detail SO (nomor, tanggal, vendor, catatan, dokumen) dengan loading skeleton
      - Pilih admin yang akan di-assign menggunakan `Combobox` (search admin berdasarkan nama)
      - Checkbox "Gunakan admin default" untuk memilih admin yang sudah di-set di pengaturan sistem
      - Tombol "Preview Dokumen" untuk membuka preview file SO di modal terpisah
      - Tombol **Approve** dan **Reject** dengan konfirmasi `ConfirmModal`
      - Tampilkan `LinkBadge` berisi link publik SO jika SO sudah disetujui
      - Kontrol akses berbasis `useHasPrivilege('serviceActivation.update')`
  - `frontend/src/app/pages/services/activation/schema/soColumns.jsx` (+105 baris)
    - **Deskripsi**: Definisi kolom tabel SO di halaman Aktivasi: kolom nomor SO, vendor, tanggal, status approval (menggunakan `ApprovalStatusCell` yang sudah diperbarui sebelumnya — menampilkan avatar admin jika sudah disetujui, tombol Review jika pending dan memiliki privilege, atau ikon pending jika tidak memiliki privilege), dan kolom aksi.

  **Frontend — Router**:
  - `frontend/src/app/router/protected.jsx` (+7 baris) — Daftarkan route: `/services/vendor/view/:vendorId/so/create`, `/services/vendor/so/:id`, `/services/vendor/so/:id/edit`
  - `frontend/src/app/router/public.jsx` (+6 baris) — Daftarkan route publik: `/so/:token` (PublicSODocument) dan `/review/so/:id` (ReviewSOPage)
  - `frontend/src/app/router/services/vendorRoute.jsx` (+10 baris) — Tambah path SO ke lazy-loaded vendor route dengan privilege guard

  **Frontend — Komponen Shared**:
  - `frontend/src/components/shared/DocumentPreviewModal.jsx` (+36 baris) — Update mendukung preview dokumen bertipe `'so'`: menentukan URL dokumen dari path SO yang tepat (dokumen asli atau dokumen bertanda tangan)
  - `frontend/src/components/shared/table/rows.jsx` (+14 baris) — Penyesuaian minor pada komponen baris tabel untuk mendukung kolom status SO

  **Frontend — Terjemahan**:
  - `frontend/src/i18n/locales/en/translations.json` (+186 baris) — Terjemahan lengkap modul SO (EN): label form, status (`pendingApproval`, `approved`, `signed`), konfirmasi approve/reject, label halaman publik (`companyLogo`, `signatureAlt`), info kontrak, history SO, pesan sukses/error, label item SO (`total`, `subtotal`, `totalOverride`, `period`)
  - `frontend/src/i18n/locales/id/translations.json` (+186 baris) — Terjemahan ID setara, termasuk: `"Buat Sales Order"`, `"Setujui Sales Order?"`, `"Tanda Tangan Customer"`, `"Riwayat Dokumen"`, `"Disetujui Oleh"`, `"Diketahui Oleh"`, dll.

- **Deskripsi Perubahan & Fungsi**:
  - Modul Sales Order (SO) kini setara penuh dengan Purchase Order (PO) — baik dari sisi backend (controller, service, model, route, notifikasi) maupun frontend (halaman CRUD, halaman publik, drawer review, tab Aktivasi, terjemahan lengkap).
  - Alur tanda tangan digital vendor pada SO menggunakan utilitas `pdf-sign.js` yang bersih — vendor membuka link publik, melihat PDF dengan `react-pdf`, memilih posisi tanda tangan, menggambar tanda tangan di canvas, lalu backend meng-embed tanda tangan ke PDF menggunakan `stampSignatureOnPdf` dan menyimpan dokumen baru ke MinIO.
  - `ReviewSOPage` (untuk admin via URL langsung) dan `PublicSODocument` (untuk vendor eksternal via share token) memisahkan dua jalur akses dokumen SO dengan kebutuhan autentikasi dan konten yang berbeda.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

### Bug Fixes (#126)
- **Broadband Profile**: Admin dengan privilege `broadbandProfile.read` kini dapat melihat badge dan detail profil broadband di halaman tabel — sebelumnya selalu tersembunyi akibat nama privilege yang salah.
- **Dedicated Internet**: Tombol "Tambah Dedicated Internet" kembali muncul bagi admin ber-privilege. Halaman edit dan detail Dedicated Internet tidak lagi diblokir oleh `AuthGuard` karena nama privilege di route sudah benar.
- **Tiket Instalasi**: Tombol update di halaman detail tiket instalasi dan tombol tambah layanan Dedicated Internet dari tiket kembali aktif bagi admin yang seharusnya punya akses.
- **Tiket Pembayaran**: Admin dengan privilege `ticketPayment.update` kini dapat menutup/mengupdate tiket pembayaran — sebelumnya tombol aksi tidak muncul karena guard mengecek `ticketPayment.close` yang tidak ada.

### Fungsionalitas Baru (#118)
- **Modul SO End-to-End**: Admin dapat membuat, mengedit, menghapus, dan mengelola Sales Order dari halaman detail vendor (tab SO baru). Admin ber-privilege dapat menyetujui atau menolak SO dari halaman Aktivasi (tab SO baru).
- **Tanda Tangan Digital via Link Publik**: Setelah SO disetujui, admin dapat menyalin link publik dan mengirimkannya ke vendor. Vendor membuka link di browser tanpa perlu login, melihat dokumen PDF secara interaktif, menggambar tanda tangan, menentukan posisi tanda tangan pada halaman PDF yang diinginkan, lalu mengkonfirmasi — sistem secara otomatis meng-embed tanda tangan ke dokumen dan menyimpan versi bertanda tangan.
- **Notifikasi Telegram Otomatis**: Seluruh event lifecycle SO (dibuat, disetujui, ditolak, ditandatangani vendor) mengirim notifikasi Telegram ke channel admin yang dikonfigurasi.
- **Utilitas PDF Terpusat**: `pdf-sign.js` dan `pdf.config.js` menjadi fondasi yang dapat digunakan kembali untuk fitur PDF signing di modul lain di masa depan (misalnya: tanda tangan PO atau Customer Quotation).

### Menu / Halaman Baru
- Tab **Sales Order** di halaman detail vendor
- Tab **SO** di halaman Aktivasi
- Halaman baru: `/services/vendor/so/create`, `/services/vendor/so/:id/edit`
- Halaman publik: `/so/:token` (vendor), `/review/so/:id` (admin)

---

## 📖 Informasi & Tutorial Singkat Fitur

### Fitur Baru: Modul Sales Order (#118)

**Penjelasan Fitur**: Sales Order (SO) adalah dokumen resmi yang diterima ISP dari vendor sebagai konfirmasi penjualan layanan atau barang. Modul ini memungkinkan pencatatan, penyimpanan dokumen SO, alur persetujuan internal oleh admin, serta tanda tangan digital vendor melalui link publik. Seluruh alur tercatat di sistem beserta siapa yang menyetujui dan kapan.

**Alur Lengkap Penggunaan**:

1. **Buat SO**: Buka halaman detail vendor → tab **Sales Order** → klik **Buat SO** → isi nomor SO, tanggal, catatan, dan upload dokumen SO (PDF, maksimum 15 MB) → klik **Simpan**.

2. **Review oleh Admin**: SO berstatus *Pending Approval* akan muncul di halaman **Aktivasi** → tab **SO**. Admin ber-privilege klik tombol **Review** → `SOReviewDrawer` terbuka → pilih admin penyetuju (atau gunakan admin default), klik **Preview Dokumen** untuk melihat isi SO → klik **Approve** atau **Reject** → konfirmasi.

3. **Bagikan ke Vendor**: Setelah SO disetujui, buka drawer detail SO → klik tombol **Salin Link Publik** → kirim URL ke vendor (via email, WhatsApp, atau platform lain).

4. **Tanda Tangan Vendor**: Vendor membuka URL tersebut di browser tanpa perlu login. Halaman menampilkan detail SO dan dokumen PDF secara interaktif (scroll antar halaman). Vendor klik **Tandatangani** → gambar tanda tangan di canvas yang tersedia → seret kotak tanda tangan ke posisi yang diinginkan pada halaman PDF → klik **Konfirmasi Tanda Tangan**.

5. **Dokumen Final**: Sistem mengambil PDF asli dari MinIO, meng-embed gambar tanda tangan tepat di posisi yang dipilih vendor menggunakan `pdf-lib`, menyimpan PDF baru ke MinIO, dan memperbarui status SO menjadi *Signed* (complete). Dokumen bertanda tangan dapat diunduh dari halaman detail SO atau diakses kembali melalui link publik.

### Bug Fix: Privilege Key (#126)

**Penjelasan**: Sistem kontrol akses Dekasimal menggunakan string privilege (misal `'dedicatedInternet.create'`) yang didefinisikan di `privilege.json` backend dan dikecek di frontend menggunakan `useHasPrivilege(key)`. Jika key yang digunakan di frontend tidak persis sama dengan yang terdaftar di backend (termasuk capitalization dan format `modul.aksi`), fungsi selalu mengembalikan `false` dan fitur tersembunyi. Hari ini 9 privilege key yang salah di 5 file diperbaiki — tidak ada perubahan tampilan bagi pengguna kecuali fitur yang sebelumnya tersembunyi kini muncul kembali.
