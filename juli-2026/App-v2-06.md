# 📝 Daily Work Report - Idham (2026-07-06)

---

## 📌 Informasi Issue
- **Nomor Issue**: #118 & #123
- **Judul Issue**: #118 — Hotfix Import Service Path · #123 — WIP Modul Customer PO & SO (Dokumen Dua Arah Customer)

## 📅 Laporan Harian - 6 Juli 2026

---

### 📅 Rincian Commit

#### [56f283ad] - resolve #118 (#118 — Hotfix Import Service Path) (12:03 WIB)

Commit hotfix kecil namun kritikal: dua controller publik (PO dan SO) masih mengimpor function service dari `vendor.service.js` yang sudah dipisah menjadi file terpisah. Tanpa perbaikan ini, endpoint publik untuk akses dokumen PO dan SO via share token akan gagal dengan error `undefined is not a function`.

- **Komponen yang Berubah**:

  - `backend/src/controllers/publicPO.controller.js`
    - **Deskripsi**: Perbaiki import — `findVendorPOByToken` dan `signVendorPO` sebelumnya diimpor dari `'../services/vendor.service.js'` (monolith lama). Setelah refactoring service layer, fungsi-fungsi ini dipindah ke file terpisah sehingga import diperbarui ke `'../services/purchaseOrder.service.js'`.

  - `backend/src/controllers/publicSO.controller.js`
    - **Deskripsi**: Perbaiki import — `findVendorSOByToken` dan `signVendorSO` sebelumnya diimpor dari `'../services/vendor.service.js'`. Diperbarui ke `'../services/salesOrder.service.js'` sesuai pemisahan service yang dilakukan pada refactoring sebelumnya.

---

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Pekerjaan hari ini dilanjutkan pada branch `issue-123` — mengembangkan modul Customer Management dengan dua tipe dokumen baru: **Customer PO** (Purchase Order dari pelanggan, berupa PDF yang diunggah) dan **Customer SO** (Sales Order kepada pelanggan, dokumen yang di-generate dari line item). Konsep ini adalah cerminan terbalik dari alur dokumen di modul Vendor:

| Arah | Dokumen Upload | Dokumen Generate |
|------|---------------|-----------------|
| **Vendor** | SO (diunggah vendor) | PO (di-generate ISP) |
| **Customer** | PO (diunggah customer) | SO (di-generate ISP ke customer) |

**Backend — Staged (sudah `git add`)**:

- `backend/package.json`
  - **Deskripsi**: Tambah dependensi `pdf-lib` untuk backend — digunakan proses embedding tanda tangan ke dokumen PDF Customer PO.

- `backend/src/app.js`
  - **Deskripsi**: Registrasi route `customerDocument` ke aplikasi Express.

- `backend/src/config/privilege.json`
  - **Deskripsi**: Tambah dua kelompok privilege baru:
    - `quotation.*` (create, read, update, changeStatus, delete, list) — privilege untuk modul Customer Quotation yang sebelumnya belum terdefinisi secara formal
    - Privilege untuk modul Customer PO dan SO

- `backend/src/controllers/files.controller.js`
  - **Deskripsi**: Tambah tiga fungsi handler file yang diperlukan oleh modul Customer PO/SO:
    - `getAppFileBuffer(filename)` — Ambil berkas dari bucket MinIO `appFiles` sebagai Buffer (digunakan untuk membaca PDF asli sebelum ditempel tanda tangan)
    - `uploadAppFileBuffer(filename, buffer)` — Simpan buffer (misalnya PDF hasil proses tanda tangan) kembali ke bucket `appFiles` dengan nama baru
    - `uploadAdminSignBase64(filename, base64Data)` — Simpan gambar tanda tangan admin (base64) ke bucket `adminSign`

- `backend/src/routes/public.route.js`
  - **Deskripsi**: Tambah 5 route publik baru untuk akses dokumen Customer PO dan SO tanpa autentikasi (token-gated):
    - `GET /public-docs/customer-po/:token` — Ambil data Customer PO via share token
    - `GET /public-docs/customer-po/:token/file` — Ambil file dokumen Customer PO
    - `POST /public-docs/customer-po/sign` — Proses tanda tangan customer pada Customer PO
    - `GET /public-docs/customer-so/:token` — Ambil data Customer SO via share token
    - `POST /public-docs/customer-so/sign` — Proses tanda tangan customer pada Customer SO

- `backend/src/utils/telegram.js`
  - **Deskripsi**: Tambah notifikasi Telegram (+105 baris) untuk seluruh event lifecycle Customer PO dan Customer SO:
    - `TelegramNotifCustomerPOSubmit` — SO disubmit/dibuat (mirror notifikasi Vendor SO)
    - `TelegramNotifCustomerSOSubmit` — Customer SO dibuat (mirror notifikasi Vendor PO)
    - Event lanjutan: approval, rejection, penandatanganan customer

- `frontend/package.json`
  - **Deskripsi**: Tambah dependensi frontend yang diperlukan untuk render PDF dan utilitas lain di modul Customer PO/SO.

- `frontend/src/app/pages/services/customerManagement/detail.jsx` (staged, +347 baris — refactor besar)
  - **Deskripsi**: Halaman detail customer direfactor secara signifikan:
    - **Struktur layout baru**: Dari grid dua kolom (`col-span-4` info + `col-span-8` konten) menjadi layout vertikal dengan section info di atas dan konten tab di bawah
    - **Navigasi tab berbasis Headless UI**: Menggunakan `TabGroup`, `TabList`, `Tab`, `TabPanels`, `TabPanel` dari `@headlessui/react` untuk navigasi antar konten customer — memungkinkan penambahan tab PO dan SO di samping tab Quotation yang sudah ada
    - **Breadcrumbs**: Diekstrak ke variabel `breadcrumbs` yang lebih bersih
    - **State baru**: `selectedIndex` untuk melacak tab yang aktif
    - **Import baru**: `DocumentTextIcon` dan `QueueListIcon` sebagai ikon untuk tab-tab baru

**Backend — Untracked (belum di-stage)**:

- `backend/src/models/customerPO.model.js` [NEW] (+118 baris)
  - **Deskripsi**: Model Purchase Order dari pelanggan — dokumen PDF yang diunggah oleh customer kepada ISP (kebalikan dari Vendor PO yang di-generate ISP). Sub-schema:
    - `DocumentFileSchema` — metadata berkas yang diunggah (name, file path, size, MIME, pages)
    - `SignaturePositionSchema` — koordinat posisi tanda tangan dinormalisasi (0..1): page, x, y, width, height
    - Schema utama: `customer` (ObjectId ref Partner), `po_number` (unik, auto-increment), `po_date`, `notes`, `document` (DocumentFileSchema, wajib), `approval`, `approved_at`, `complete`, `signed_file`, `signature`, `signature_position`, `signed_at`, `signed_by`

- `backend/src/models/customerSO.model.js` [NEW] (+186 baris)
  - **Deskripsi**: Model Sales Order kepada pelanggan — dokumen yang di-generate ISP dari line item harga (kebalikan dari Vendor SO yang diunggah vendor). Sub-schema:
    - `AttachmentSchema` — lampiran dengan upload_by tracking
    - `LineItemSchema` — item baris dengan: nama, tipe item (service/goods/contractor), satuan, tipe harga (otc/mrc), qty, harga satuan
    - Schema utama: `customer`, `so_number`, `so_date`, `notes`, line items, total, `approval`, `approved_at`, `complete`, field tanda tangan

- `backend/src/services/customerDocument.service.js` [NEW] (+421 baris)
  - **Deskripsi**: Service layer gabungan untuk Customer PO dan Customer SO — 22 fungsi terbagi rata:
    - **Customer PO** (11 fungsi): `findCustomerPOById`, `createNewCustomerPO`, `updateCustomerPOData`, `findAllCustomerPOsForTable`, `findCustomerPOsByCustomer`, `approveCustomerPO`, `rejectCustomerPO`, `deleteCustomerPOById`, `findCustomerPOByToken`, `signCustomerPO`, `nextCountCustomerPO`
    - **Customer SO** (11 fungsi): `findCustomerSOById`, `createNewCustomerSO`, `updateCustomerSOData`, `findAllCustomerSOsForTable`, `findCustomerSOsByCustomer`, `approveCustomerSO`, `rejectCustomerSO`, `deleteCustomerSOById`, `findCustomerSOByToken`, `signCustomerSO`, `nextCountCustomerSO`

- `backend/src/controllers/customerDocument.controller.js` [NEW] (+621 baris)
  - **Deskripsi**: Controller gabungan Customer PO dan Customer SO — 20 controller function:
    - **Customer PO** (10): `createPO`, `listPO`, `listAllPO`, `readPO`, `submitPO`, `requestPOPreview`, `approvePO`, `rejectPO`, `deletePO`, `updatePO`
    - **Customer SO** (10): `createSO`, `listSO`, `listAllSO`, `readSO`, `submitSO`, `requestSOPreview`, `approveSO`, `rejectSO`, `deleteSO`, `updateSO`
    - Juga men-export ulang `findCustomerPOByToken` dan `signCustomerPO` untuk digunakan public controller

- `backend/src/controllers/publicCustomerPO.controller.js` [NEW] (+159 baris)
  - **Deskripsi**: Controller akses publik Customer PO tanpa autentikasi (token-gated):
    - `getPOByToken` — Ambil data Customer PO via share token; validasi token dan status approval
    - `getPOFileByToken` — Sajikan file PDF Customer PO; jika sudah ditandatangani kembalikan `signed_file`, jika belum kembalikan dokumen asli
    - `signPOByToken` — Proses tanda tangan customer: terima gambar tanda tangan + posisi, embed ke PDF menggunakan `stampSignatureOnPdf` dari `pdf-sign.js`, simpan ke MinIO

- `backend/src/controllers/publicCustomerSO.controller.js` [NEW] (+85 baris)
  - **Deskripsi**: Controller akses publik Customer SO:
    - `getSOByToken` — Ambil data Customer SO via token
    - `signSOByToken` — Proses tanda tangan customer pada SO yang di-generate

- `backend/src/routes/customerDocument.route.js` [NEW] (+177 baris)
  - **Deskripsi**: Seluruh route API Customer PO dan SO: CRUD, approval, submit, generate preview, delete — masing-masing terlindungi privilege yang sesuai.

- `backend/src/utils/pdf-sign.js` [NEW] (+73 baris)
  - **Deskripsi**: Salinan utilitas `pdf-sign.js` dari modul Vendor — `getPdfPageCount` dan `stampSignatureOnPdf` — digunakan untuk proses embedding tanda tangan ke dokumen Customer PO.

**Frontend — Untracked (belum di-stage)**:

- `frontend/src/app/pages/services/customerManagement/schema/customerPOSchema.js` [NEW] (+11 baris)
  - **Deskripsi**: Skema validasi Yup untuk form Customer PO — validasi field wajib (`po_number`, `po_date`) dan upload dokumen.

- `frontend/src/app/pages/services/customerManagement/schema/customerSOSchema.js` [NEW] (+43 baris)
  - **Deskripsi**: Skema validasi Yup untuk form Customer SO — validasi field wajib (`so_number`, `so_date`) dan line item.

- `frontend/src/configs/pdf.config.js` [NEW] (+13 baris)
  - **Deskripsi**: Konfigurasi terpusat `react-pdf` — mengatur pdfjs worker dan mengimpor CSS layer anotasi/teks. Salinan dari konfigurasi yang sama di modul Vendor.

- `frontend/src/constants/customer.constant.js` [NEW] (+6 baris)
  - **Deskripsi**: Konstanta `ITEM_TYPE_OPTIONS` untuk tipe item Customer SO: service, goods, contractor — paralel dengan `vendor.constant.js`.

- `frontend/src/utils/formatFileSize.js` [NEW] (+11 baris)
  - **Deskripsi**: Fungsi utilitas `formatFileSize(bytes)` — konversi byte ke B/KB/MB/GB. Salinan dari utilitas yang sama di modul Vendor.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)

- **Bug Fix (#118)**: Endpoint publik akses dokumen PO dan SO via share token kembali berfungsi setelah import service path dikoreksi. Sebelum fix ini, vendor yang mencoba membuka link publik dokumen PO/SO akan mendapatkan error karena fungsi `findVendorPOByToken` / `signVendorSO` tidak ditemukan di `vendor.service.js` yang sudah direfactor.

- **Fungsionalitas Baru (WIP #123)**: Modul Customer Management diperluas dengan dua tipe dokumen baru — Customer PO (dokumen yang diunggah customer) dan Customer SO (dokumen yang di-generate ISP). Keduanya memiliki alur approval admin, share link publik, dan tanda tangan digital. Halaman detail customer direfactor menggunakan navigasi tab yang lebih terstruktur untuk mengakomodasi tiga jenis konten: Quotation, PO, dan SO.

- **Menu/Tombol Baru**: Navigasi tab baru di halaman detail customer (Headless UI TabGroup). Endpoint publik baru `/public-docs/customer-po/:token` dan `/public-docs/customer-so/:token`.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Penjelasan Konsep — Dokumen Dua Arah Customer**:

Modul Customer Management kini mendukung dua arah alur dokumen:

| | Customer PO | Customer SO |
|--|------------|------------|
| **Dibuat oleh** | Customer (upload ke sistem) | ISP (generate dari line item) |
| **Isi** | PDF Purchase Order dari customer | Dokumen Sales Order dengan daftar harga OTC/MRC |
| **Alur** | Customer upload → Admin approve → Admin embed tanda tangan → Selesai | ISP buat SO → Admin approve → Link publik ke customer → Customer tanda tangan digital |
| **Mirror dari** | Vendor SO (yang juga diunggah) | Vendor PO (yang juga di-generate ISP) |

**Alur Penggunaan Customer PO**:
1. Customer mengirim dokumen PO (PDF) kepada ISP.
2. Admin buka halaman detail customer → tab **Purchase Order** → unggah dokumen PO.
3. Admin ber-privilege **Approve** Customer PO.
4. Admin dapat meng-embed tanda tangan digital ke dokumen PO menggunakan share link.
5. PO tersimpan sebagai dokumen resmi terkait customer tersebut.

**Alur Penggunaan Customer SO**:
1. Admin buka halaman detail customer → tab **Sales Order** → **Buat SO**.
2. Tambah line item layanan/barang (OTC atau MRC), isi harga dan qty → Simpan.
3. Admin **Approve** SO → salin link publik → kirim ke customer.
4. Customer buka link tanpa login → tanda tangan digital pada dokumen SO.
5. SO tersimpan dengan status *Signed* dan dokumen bertanda tangan di MinIO.
