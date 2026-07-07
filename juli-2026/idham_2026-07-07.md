# 📝 Daily Work Report - Idham (2026-07-07)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Document Management — Implementasi Backend Lengkap Modul Customer PO & SO

## 📅 Laporan Harian - 7 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Seluruh pekerjaan hari ini masih berstatus WIP — backend selesai penuh dan fondasi frontend sudah disiapkan.

---

### 🏗️ Konteks & Arsitektur

Modul Customer Document Management memperkenalkan dua tipe dokumen baru dalam ekosistem Customer Management:

| | **Customer PO** | **Customer SO** |
|--|----------------|----------------|
| **Pengertian** | Purchase Order dari pelanggan kepada ISP | Sales Order dari ISP kepada pelanggan |
| **Dibuat oleh** | Customer (unggah PDF) | Admin ISP (generate dari line item) |
| **Konten** | Dokumen PDF yang diunggah | Line item layanan/barang dengan harga OTC/MRC |
| **Mirror arsitektur** | Vendor SO (upload + tanda tangan berposisi) | Vendor PO (generated + line item) |

Pola tanda tangan digital, share token, notifikasi Telegram, dan alur approval mengikuti persis yang sudah dibangun di modul Vendor (issue #118), memastikan konsistensi di seluruh sistem.

---

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

**Backend — Untracked (file baru belum di-stage)**:

- `backend/src/models/customerPO.model.js` [NEW] (+118 baris)
  - **Deskripsi**: Model Purchase Order dari pelanggan. Menggunakan pola upload dokumen yang sama dengan `vendorSO.model.js`. Sub-schema:
    - `DocumentFileSchema` — metadata berkas PDF yang diunggah: `name`, `file` (path MinIO), `size` (bytes), `mime`, `pages` (jumlah halaman, default 1)
    - `SignaturePositionSchema` — koordinat tanda tangan dinormalisasi (0..1): `page`, `x`, `y`, `width`, `height` — bebas resolusi karena relatif terhadap ukuran halaman
    - Schema utama: `customer` (ObjectId ref Partner), `po_number` (unique, auto-increment), `po_date`, `notes`, `document` (wajib), `approval`, `approved_at`, `complete`, `signed_file`, `signature`, `signature_position`, `signed_at`, `signed_by`. Soft delete via `mongoose-delete`.

- `backend/src/models/customerSO.model.js` [NEW] (+186 baris)
  - **Deskripsi**: Model Sales Order yang di-generate ISP kepada pelanggan. Menggunakan pola line item yang sama dengan `vendorPO.model.js` / `customerQuotation.model.js`. Sub-schema:
    - `AttachmentSchema` — lampiran opsional: name, file, size, uploaded_at, uploaded_by
    - `LineItemSchema` — item baris: nama, `item_type` (enum: service/goods/contractor), satuan, `price_type` (enum: otc/mrc), qty, harga satuan
    - Schema utama: `customer`, `so_number` (unique, auto-increment), `so_date`, `notes`, `line_items` (array LineItem), `total_override` (opsional), `approval`, `approved_at`, `complete`, field tanda tangan admin

- `backend/src/services/customerDocument.service.js` [NEW] (+421 baris)
  - **Deskripsi**: Service layer gabungan untuk Customer PO dan Customer SO — 22 fungsi yang terbagi rata 11 per tipe dokumen:

    **Customer PO (11 fungsi)**:
    - `findCustomerPOById(id)` — Cari PO berdasarkan `_id`, populate `customer` dan `approval`
    - `createNewCustomerPO(data)` — Buat PO baru; auto-increment `po_number`
    - `updateCustomerPOData(id, data)` — Update metadata PO (nomor, tanggal, catatan, dokumen)
    - `findAllCustomerPOsForTable(params)` — Query tabel PO dengan filter, pagination, sorting untuk DataTable
    - `findCustomerPOsByCustomer(customerId)` — Ambil semua PO milik customer tertentu
    - `approveCustomerPO(id, adminId)` — Set `approval = adminId`, `approved_at = now`
    - `rejectCustomerPO(id)` — Soft delete PO yang ditolak
    - `deleteCustomerPOById(id)` — Soft delete PO
    - `findCustomerPOByToken(token)` — Cari PO berdasarkan share token (untuk akses publik)
    - `signCustomerPO(id, adminId, signatureData, signedFilePath, position)` — Simpan hasil tanda tangan: `complete = true`, path file bertanda tangan, data tanda tangan, posisi
    - `nextCountCustomerPO()` — Counter auto-increment untuk nomor PO berikutnya

    **Customer SO (11 fungsi)**:
    - `findCustomerSOById`, `createNewCustomerSO`, `updateCustomerSOData`, `findAllCustomerSOsForTable`, `findCustomerSOsByCustomer`, `approveCustomerSO`, `rejectCustomerSO`, `deleteCustomerSOById`, `findCustomerSOByToken`, `signCustomerSO`, `nextCountCustomerSO` — Paralel dengan Customer PO namun untuk SO.

- `backend/src/controllers/customerDocument.controller.js` [NEW] (+621 baris)
  - **Deskripsi**: Controller gabungan — 20 handler asyncHandler plus fungsi internal:

    **Customer PO (10 controller)**:
    - `createPO` — Validasi, proses upload file dokumen PO (max 15 MB), ekstrak metadata (nama, ukuran, MIME, jumlah halaman PDF via `pdf-sign.js`), simpan ke MinIO, buat record PO
    - `listPO` — Daftar PO milik customer tertentu (untuk tab PO di detail customer)
    - `listAllPO` — Daftar semua PO dari semua customer (untuk halaman Aktivasi)
    - `readPO` — Detail PO tunggal dengan populate lengkap
    - `submitPO` — Kirim PO untuk direview admin (ubah status menjadi submitted)
    - `requestPOPreview` — Generate/regenerasi share link (token) PO
    - `approvePO` — Setujui PO; embed tanda tangan admin ke dokumen PDF menggunakan `stampSignatureOnPdf`, simpan dokumen bertanda tangan ke MinIO, set `approval` dan `approved_at`
    - `rejectPO` — Tolak PO; hapus soft-delete dan kirim notifikasi
    - `deletePO` — Hapus PO dengan guard jika sudah disetujui
    - `updatePO` — Update metadata PO termasuk penggantian dokumen

    **Customer SO (10 controller)**: `createSO`, `listSO`, `listAllSO`, `readSO`, `submitSO`, `requestSOPreview`, `approveSO`, `rejectSO`, `deleteSO`, `updateSO` — Paralel dengan PO namun menangani dokumen yang di-generate dari line item.

- `backend/src/controllers/publicCustomerPO.controller.js` [NEW] (+159 baris)
  - **Deskripsi**: Controller publik Customer PO — endpoint tanpa autentikasi menggunakan share token:
    - `getPOByToken` — Ambil data PO via token; validasi token valid dan PO sudah disetujui sebelum data dikirim
    - `getPOFileByToken` — Sajikan file PDF; jika `complete = true` kembalikan `signed_file`, jika belum kembalikan `document.file` asli
    - `signPOByToken` — Proses tanda tangan: terima gambar tanda tangan + posisi, embed ke PDF menggunakan `stampSignatureOnPdf` dari `pdf-sign.js`, simpan ke MinIO, update PO

- `backend/src/controllers/publicCustomerSO.controller.js` [NEW] (+85 baris)
  - **Deskripsi**: Controller publik Customer SO — `getSOByToken` dan `signSOByToken` untuk alur tanda tangan pelanggan pada SO yang di-generate.

- `backend/src/routes/customerDocument.route.js` [NEW] (+177 baris)
  - **Deskripsi**: Seluruh route API Customer PO dan SO dengan privilege guard:
    - Customer PO: `POST /customer-po`, `GET /customer-po`, `GET /customer-po/:id`, `PATCH /customer-po/:id`, `PATCH /customer-po/approve/:id`, `PATCH /customer-po/reject/:id`, `POST /customer-po/preview/:id`, `DELETE /customer-po/:id`
    - Customer SO: route paralel untuk SO
    - `GET /customer-po/page-count/:filename` — Endpoint untuk ambil jumlah halaman PDF (digunakan frontend saat memilih posisi tanda tangan)

- `backend/src/utils/pdf-sign.js` [NEW] (+73 baris)
  - **Deskripsi**: Utilitas `getPdfPageCount` dan `stampSignatureOnPdf` dari `pdf-lib` + `sharp` — salinan dari modul Vendor dengan fungsi identik. Akan dipertimbangkan untuk dijadikan shared utility di masa mendatang.

**Backend — Staged (sudah `git add`)**:

- `backend/package.json` (+1) & `backend/package-lock.json` (+50)
  - **Deskripsi**: Tambah dependensi `pdf-lib ^1.17.1` untuk manipulasi PDF di backend (embed tanda tangan).

- `backend/src/app.js` (+2)
  - **Deskripsi**: Registrasi route `customerDocument` ke aplikasi Express.

- `backend/src/config/privilege.json` (+16 baris)
  - **Deskripsi**: Tambah dua kelompok privilege baru:
    - `quotation` (create, read, update, changeStatus, delete, list) — formalisasi privilege Customer Quotation yang sebelumnya belum terdaftar secara eksplisit
    - `customerDocument` (create, read, update, changeStatus, delete, list) — privilege baru untuk Customer PO & SO

- `backend/src/controllers/files.controller.js` (+31 baris)
  - **Deskripsi**: Tambah tiga fungsi handler file yang dibutuhkan proses tanda tangan:
    - `getAppFileBuffer(filename)` — Ambil berkas dari bucket MinIO `appFiles` sebagai Node.js Buffer — dipakai untuk membaca PDF asli sebelum ditempel tanda tangan
    - `uploadAppFileBuffer(filename, buffer)` — Simpan Buffer hasil (misalnya PDF bertanda tangan) kembali ke MinIO `appFiles` dengan nama baru
    - `uploadAdminSignBase64(filename, base64Data)` — Simpan gambar tanda tangan admin dari format base64 ke bucket `adminSign`

- `backend/src/routes/files.route.js` (+9 baris)
  - **Deskripsi**: Tambah route akses file Customer PO dan SO: `GET /file/customer-po/:filename` dan `GET /file/customer-so/:filename`.

- `backend/src/routes/public.route.js` (+18 baris)
  - **Deskripsi**: Tambah 5 route publik token-gated untuk Customer PO dan SO:
    - `GET /public-docs/customer-po/:token` — Data Customer PO
    - `GET /public-docs/customer-po/:token/file` — File dokumen Customer PO
    - `POST /public-docs/customer-po/sign` — Proses tanda tangan Customer PO
    - `GET /public-docs/customer-so/:token` — Data Customer SO
    - `POST /public-docs/customer-so/sign` — Proses tanda tangan Customer SO

- `backend/src/utils/telegram.js` (+105 baris)
  - **Deskripsi**: Tambah fungsi notifikasi Telegram untuk seluruh event lifecycle Customer PO dan SO:
    - `TelegramNotifCustomerPOSubmit` — Customer PO disubmit (mirror `TelegramNotifVendorSOSubmit`)
    - `TelegramNotifCustomerSOSubmit` — Customer SO dibuat (mirror `TelegramNotifVendorPOSubmit`)
    - Event lanjutan: approval, rejection, penandatanganan oleh customer

- `backend/src/locales/en/translation.json` (+56 baris) & `id/translation.json` (+54 baris)
  - **Deskripsi**: Tambah pesan error/sukses backend untuk Customer PO dan SO: `notFound`, `alreadyApproved`, `alreadySigned`, `cannotReject`, `documentNotFound`, dll — dalam dua bahasa.

**Frontend — Fondasi (Untracked)**:

- `frontend/src/configs/pdf.config.js` [NEW] (+13 baris)
  - **Deskripsi**: Konfigurasi `react-pdf` (pdfjs worker + CSS layer anotasi/teks) terpusat. Salinan dari konfigurasi yang sama di modul Vendor.

- `frontend/src/utils/formatFileSize.js` [NEW] (+11 baris)
  - **Deskripsi**: `formatFileSize(bytes)` — konversi byte ke B/KB/MB/GB dengan satu desimal.

- `frontend/src/constants/customer.constant.js` [NEW] (+6 baris)
  - **Deskripsi**: `ITEM_TYPE_OPTIONS` untuk line item Customer SO: service, goods, contractor — paralel dengan `vendor.constant.js`.

- `frontend/src/app/pages/services/customerManagement/schema/customerPOSchema.js` [NEW] (+11 baris)
  - **Deskripsi**: Skema validasi Yup form Customer PO: `po_number`, `po_date` wajib, `notes` opsional.

- `frontend/src/app/pages/services/customerManagement/schema/customerSOSchema.js` [NEW] (+43 baris)
  - **Deskripsi**: Skema validasi Yup form Customer SO: `so_number`, `so_date` wajib; validasi line item (nama wajib, tipe item valid, harga positif, qty minimal 1).

- `frontend/package.json` (+2) & `frontend/package-lock.json` (+392)
  - **Deskripsi**: Tambah dependensi frontend: `react-pdf ^10.4.1` untuk render PDF interaktif di halaman tanda tangan, dan satu paket pendukung lainnya.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **Kemampuan Admin**: Backend Customer PO dan SO selesai penuh — seluruh operasi CRUD, approval, share link, tanda tangan digital, dan notifikasi Telegram sudah siap. Begitu frontend selesai besok, admin dapat langsung menggunakan fitur ini dari halaman detail customer.
- **Konsistensi Sistem**: Arsitektur Customer PO dan SO konsisten dengan modul Vendor (service layer, controller pattern, model schema, public endpoint, Telegram notifications) — meminimalkan learning curve untuk pengembangan dan pemeliharaan.
- **Tidak ada perubahan visible ke pengguna hari ini** — backend WIP, belum dicommit.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Penjelasan Fitur**: Modul Customer Document Management melengkapi ekosistem dokumen ISP dari sisi customer. Selama ini ISP dapat mengelola dokumen dari vendor (PO dan SO vendor). Dengan modul ini, ISP juga dapat mengelola dokumen dari sisi customer: menerima dan mengarsipkan PO dari customer (Customer PO), serta menerbitkan SO kepada customer lengkap dengan harga dan alur tanda tangan digital (Customer SO).

**Alur yang Dibangun Hari Ini (Backend)**:
1. Admin buat Customer PO (unggah PDF dari customer) atau Customer SO (isi line item dan harga)
2. Backend simpan ke MinIO, auto-increment nomor dokumen, kirim notifikasi Telegram
3. Admin approve → backend embed tanda tangan digital ke PDF, simpan ke MinIO
4. Backend generate share token untuk akses publik tanpa login
5. Customer buka link publik → backend sajikan data SO/PO dan file dokumen via token
6. Customer tanda tangan digital → backend embed ke PDF → simpan dokumen final
