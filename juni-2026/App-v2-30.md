# 📝 Daily Work Report - Idham (2026-06-30)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123 & #118
- **Judul Issue**: #123 - Customer Quotation Management (finalisasi commit) · #118 - Restrukturisasi Modul Vendor/PO/SO & Unifikasi Komponen Approval

## 📅 Laporan Harian - 30 Juni 2026

### 📅 Rincian Commit

#### [f1967bf4] - save #123 (#123 - Customer Quotation Management)

- **Komponen yang Berubah**: Finalisasi commit dari seluruh pekerjaan WIP modul Customer Quotation Management yang telah dikerjakan sejak 26 Juni 2026 (37 file, 4809 baris) — model `customerQuotation.model.js`, controller, service, route, halaman `customerManagement/` & `quotation/`, `QuotationDetailDrawer`, `PublicQuotationDocument`, dan seluruh terjemahan terkait.
- **Deskripsi Perubahan & Fungsi**: Modul penawaran harga ke customer (quotation dengan line item OTC/MRC, approval admin, tanda tangan digital customer via link publik) resmi tersimpan dalam riwayat commit. Lihat laporan tanggal 26 Juni 2026 untuk rincian lengkap per-file.

---

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Setelah commit `save #123`, dilanjutkan dengan pekerjaan restrukturisasi besar pada modul Vendor/PO/SO (issue #118) yang masih berstatus WIP — 96 file staged + 21 file unstaged + 7 file baru belum dilacak (untracked).

**1. Restrukturisasi Penamaan Folder Modul**:
- `vendor/` → `vendorManagement/` — modul vendor utama dipindahkan ke nama folder baru: `index.jsx`, `create.jsx`, `edit.jsx`, `detail.jsx`, `schema/columns.jsx`, `schema/vendorSchema.js`, `schema/vendorItemSchema.js`, `schema/vendorPOSchema.js` [NEW] `VendorItemDetailDrawer.jsx`, [NEW] `schema/vendorSOSchema.js`
  - **Deskripsi**: Konsisten dengan pola modul lain yang menggunakan akhiran "Management"; vendor index/create/edit/detail kini berada di satu folder mandiri dipisah dari sub-modul PO dan SO.
- `vendor/po/` → `purchaseOrder/` — `EditPODrawer.jsx`, `create.jsx` dipindah; `PODocumentPreview.jsx` di-rename dari `vendor/PODocumentPreview.jsx`; ditambah `edit.jsx` [NEW]
  - **Deskripsi**: Modul Purchase Order sepenuhnya mandiri dari `vendor/`, lengkap dengan halaman edit terpisah (sebelumnya hanya tersedia via drawer).
- `vendor/so/` → `salesOrder/` — `SODocumentPreview.jsx`, `create.jsx`, `edit.jsx` [NEW]
  - **Deskripsi**: Modul Sales Order kini juga mandiri mengikuti pola PO, lengkap dengan halaman create dan edit.
- **File lama dihapus**: `vendor/CreatePODrawer.jsx`, `vendor/create.jsx`, `vendor/detail.jsx`, `vendor/edit.jsx`, `vendor/po/*`, `vendor/schema/createSchema.js`, `vendor/so/components/SOSignModal.jsx`, `vendor/so/create.jsx`, `vendor/so/detail.jsx`, `vendor/vendorServiceDetail.jsx`

**2. Backend — Modul Sales Order (VendorSO) Lengkap [NEW, untracked]**:
- `backend/src/controllers/vendorSO.controller.js` [NEW] (+426 baris)
  - **Deskripsi**: Controller lengkap CRUD SO, approval admin, tanda tangan digital — mengikuti pola `vendor.controller.js` (PO).
- `backend/src/services/vendorSO.service.js` [NEW] (+260 baris)
  - **Deskripsi**: Service layer SO: query, create, update, approve.
- `backend/src/routes/vendorSO.route.js` [NEW] (+287 baris)
  - **Deskripsi**: Seluruh route API SO: CRUD + approval + dokumen.
- `backend/src/controllers/publicSO.controller.js` [NEW] (+72 baris)
  - **Deskripsi**: Controller akses dokumen SO publik tanpa login.

**3. Frontend — Halaman Publik & Review [NEW, untracked]**:
- `frontend/src/app/pages/public/PublicSODocument.jsx` [NEW] (+431 baris)
  - **Deskripsi**: Viewer dokumen SO publik, paralel dengan `PublicQuotationDocument` dan `PublicPODocument`.
- `frontend/src/app/pages/public/ReviewSOPage.jsx` [NEW] (+367 baris)
- `frontend/src/app/pages/public/ReviewPOPage.jsx` [NEW]
  - **Deskripsi**: Halaman review publik baru untuk PO dan SO — kemungkinan alur persetujuan/penolakan oleh pihak eksternal (vendor) melalui link, terpisah dari halaman dokumen biasa.
- `frontend/src/app/pages/public/PublicBAADocument.jsx`, `PublicBADDocument.jsx`, `publicBAPDocument.jsx`, `ReviewBAAPage.jsx`
  - **Deskripsi**: Disesuaikan agar konsisten dengan pola baru `ReviewPOPage`/`ReviewSOPage`.

**4. Unifikasi Komponen Approval Status (PO & SO)**:
- `frontend/src/app/pages/services/activation/schema/ApprovalStatusCell.jsx`
  - **Deskripsi**: Satu komponen status approval generik digunakan bersama untuk PO dan SO (sebelumnya `POApprovalStatusCell`/`SOApprovalStatusCell` terpisah dalam `poColumnsDef.jsx`/`soColumnsDef.jsx`). Tambahan kelas `shrink-0`/`whitespace-nowrap` untuk memperbaiki layout sel yang sebelumnya bisa melebar tidak konsisten.
- `frontend/src/app/pages/services/activation/schema/poColumns.jsx`, `soColumns.jsx`
  - **Deskripsi**: Diperbarui untuk memakai `ApprovalStatusCell` bersama.
- `frontend/src/app/pages/services/activation/schema/poColumnsDef.jsx` [DELETED], `soColumnsDef.jsx` [DELETED]
  - **Deskripsi**: Dihapus — fungsinya digabung kembali ke `poColumns.jsx`/`soColumns.jsx` dengan komponen sel yang dibagi (shared), menyederhanakan struktur file kolom tabel di halaman Aktivasi.

**5. Komponen Shared Baru & Diperbarui**:
- `frontend/src/components/shared/VendorItemModal.jsx` [NEW]
  - **Deskripsi**: Modal generik untuk tambah/edit item vendor (form dengan validasi Yup, dipakai dari beberapa tempat).
- `frontend/src/constants/vendor.constant.js` [NEW]
  - **Deskripsi**: `ITEM_TYPE_OPTIONS` (service/goods/contractor) dipusatkan sebagai konstanta bersama, menggantikan definisi inline yang berulang di beberapa file.
- `frontend/src/components/shared/ConfirmModal.jsx`, `GlobalConfirmModal.jsx`, `components/ui/Modal.jsx`
  - **Deskripsi**: Penyesuaian lanjutan pada komponen modal (kelanjutan dari perbaikan `zIndex` sebelumnya).
- `frontend/src/components/shared/table/RowActions.jsx`, `Table.jsx`, `table/rows.jsx`
  - **Deskripsi**: Update komponen tabel bersama untuk mendukung pola drawer/aksi baris yang dipakai PO, SO, dan Quotation.
- `frontend/src/components/shared/form/FormInput.jsx`
  - **Deskripsi**: Penyesuaian komponen input form bersama.
- `frontend/src/components/shared/DocumentPreviewModal.jsx`
  - **Deskripsi**: Update untuk mendukung tipe dokumen SO selain PO dan Quotation.

**6. Backend — Perbaikan Utilitas**:
- `backend/src/utils/data-table.js`
  - **Deskripsi**: Tambah proyeksi otomatis untuk field yang berakhiran `_id`, `id`, atau `code` saat filter data table aktif — memastikan field kunci selalu ikut ter-select meski tidak ada di daftar kolom yang ditampilkan.
- `backend/src/utils/validation-data.js`
  - **Deskripsi**: Tambah penanganan error MongoDB duplicate key (`error.code === 11000`) secara langsung — pesan error field "sudah digunakan" kini muncul untuk error duplikat yang tidak terbungkus `MongooseError.cause` (kasus yang sebelumnya tidak tertangani).
- `backend/src/controllers/document.controller.js`, `models/document.model.js`, `models/vendor.model.js`, `models/vendorPO.model.js`, `models/vendorSO.model.js`
  - **Deskripsi**: Penyesuaian model dan controller dokumen umum agar selaras dengan modul SO yang baru.

**7. Konvensi Environment Variable Baru**:
- `backend/.env.example` [NEW], `telegram-api/.env.example` [NEW], `frontend/.env.development` → `frontend/.env.example` [RENAMED]
  - **Deskripsi**: Memperkenalkan file template environment variable (`.env.example`) yang dapat di-commit ke repository sebagai acuan konfigurasi, menggantikan kebiasaan mengandalkan `.env.development` yang sebelumnya berisi nilai aktual.

**8. Dampak Lintas Modul Lain** (penyesuaian minor akibat perubahan komponen shared di atas):
  - `network/fiberCable/components/{NodeInfoDrawer,SidebarTools,SpliceTray}.jsx`
  - `network/ipv4Management/{create,edit,index}.jsx`
  - `services/hotspotVoucher/{create,edit,editBatch,index}.jsx`
  - `tickets/{CancelTicket,DismantleDevices,installation/create}.jsx`
  - `utilities/printStation/index.jsx`
  - `warehouse/items/{AddItemDrawer,itemDetail}.jsx`, `warehouse/location/edit.jsx`
  - `middleware/AuthGuard.jsx`
  - **Deskripsi**: Penyesuaian mengikuti perubahan API/props pada komponen shared (`ConfirmModal`, `Table`, `RowActions`, `FormInput`) — bukan perubahan fungsional baru di modul-modul tersebut.

**9. Terjemahan**:
- `frontend/src/i18n/locales/en/translations.json`, `id/translations.json`
  - **Deskripsi**: Pembersihan & reorganisasi besar pada struktur terjemahan (unstaged: -1262/+~700 baris di file EN) sejalan dengan unifikasi `ApprovalStatusCell` dan modul SO baru — menghapus duplikasi key lama (`pendingSign`, `signed`, dst.) dan menambah key untuk modul `purchaseOrder`/`salesOrder` yang konsisten.
- `backend/src/locales/en/translation.json`, `id/translation.json`
  - **Deskripsi**: Tambah pesan error/sukses untuk modul SO dan validasi data-table.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Kemampuan Pengguna/Admin**: Modul Customer Quotation Management (issue #123) resmi tersimpan dan siap di-review. Modul Sales Order kini memiliki backend dan halaman publik yang setara lengkap dengan Purchase Order — admin dapat membuat, menyetujui, dan membagikan link publik SO seperti halnya PO.
- **Bug Fix / Solusi Masalah**:
  - Perbaikan `data-table.js`: field `_id`/`code` tidak lagi hilang dari hasil query saat filter kolom aktif — memperbaiki potensi data kosong di tabel-tabel yang menggunakan fitur filter.
  - Perbaikan `validation-data.js`: error duplikasi data (misal nomor PO/SO/quotation yang sama) kini menampilkan pesan yang jelas ke pengguna, bukan error generik.
  - Layout `ApprovalStatusCell` yang melebar tidak konsisten di tabel Aktivasi telah diperbaiki dengan `shrink-0`/`whitespace-nowrap`.
- **Menu/Tombol Baru**: Halaman `ReviewPOPage` dan `ReviewSOPage` baru untuk alur review publik. Halaman edit PO dan SO kini tersedia sebagai halaman mandiri (`purchaseOrder/edit.jsx`, `salesOrder/edit.jsx`).

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Penjelasan Fitur**: Hari ini adalah hari konsolidasi — modul Vendor, Purchase Order, dan Sales Order yang sebelumnya berkembang terpisah (dan sedikit tidak konsisten penamaan foldernya) dirapikan menjadi struktur modular yang seragam: `vendorManagement/`, `purchaseOrder/`, `salesOrder/`. Backend SO yang sebelumnya hanya berupa model kini memiliki controller, service, dan route selengkap PO. Komponen status approval untuk PO dan SO disatukan menjadi satu komponen `ApprovalStatusCell` agar konsisten dan mudah dipelihara.
- **Langkah Penggunaan (Tutorial)**: Karena pekerjaan masih WIP (belum di-commit), fitur belum dapat diuji penuh di lingkungan produksi. Setelah commit final, alur penggunaan SO akan setara dengan PO: buat SO dari halaman vendor → admin approve dari Aktivasi tab SO → link publik SO dapat dibagikan ke vendor untuk ditandatangani.
