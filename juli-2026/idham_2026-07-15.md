# 📝 Daily Work Report - Idham (2026-07-15)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Prospect Management — Modul Funnel Pra-Deal (Fase Dokumen: Pendaftaran Prospek Publik, PIC Penawaran, Alur Work Order → Tiket Pemasangan)

## 📅 Laporan Harian - 15 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Tidak ada commit baru hari ini (commit terakhir `7aa4cee save #123` tertanggal 2026-07-13). **Seluruh pekerjaan 15 Juli 2026 masih berstatus WIP** di branch `issue-123` — 42 berkas berubah, 7 berkas baru (belum di-`git add`), total ±2.400 baris ditambah / ±1.250 dihapus.

Pekerjaan hari ini melanjutkan modul **Prospect Management** ke *fase dokumen*: pendaftaran prospek dari publik (tanpa login), pengunggahan foto survei (KTP PIC & lokasi), data penandatangan (PIC) pada penawaran, serta alur Work Order → pratinjau dokumen → konversi ke Tiket Pemasangan.

---

### 🏗️ Konteks & Arsitektur

- **Pendaftaran prospek publik** meniru pola `/registration/create` yang sudah ada: link membawa `referal` (yaitu `admin_id` marketing) yang otomatis menjadi `created_by` prospek. Prospek tercatat dengan `source = "Form Online"`.
- **Foto survei prospek** (KTP PIC & foto site) disimpan sebagai filename di MinIO bucket `appFiles`, diunggah via multipart dan diproses jadi `webp`. Foto KTP disajikan dengan **watermark nama admin** (data sensitif), foto site disajikan apa adanya.
- **PIC penawaran** ditambahkan sebagai snapshot penandatangan pada dokumen Quotation (nama, jabatan, telepon, email, tanda tangan PNG).
- **Work Order** kini punya lembar dokumen sendiri (tanpa informasi harga apa pun), pratinjau langsung dari tabel, dan aksi konversi manual menjadi **Tiket Pemasangan** oleh tim NOC (ref dua arah `wo.ticket ↔ ticket.work_order`).

---

### 🗂️ Backend — Berkas yang Berubah

- [backend/src/routes/prospect.route.js](backend/src/routes/prospect.route.js)
  - **Deskripsi**: Menambah route publik `POST /prospect/register` (tanpa `protectedAdmin`) untuk form pendaftaran prospek via link referal marketing.
- [backend/src/controllers/prospect.controller.js](backend/src/controllers/prospect.controller.js)
  - **Deskripsi**: Menambah `registerProspect` (pendaftaran publik — validasi `referal` → admin, set `created_by` & `source`). Helper `uploadProspectPhoto` untuk unggah/preserve foto KTP PIC & site; `createProspect` dan `updateProspect` kini memproses `req.files.pic_photo` / `req.files.site_photo`.
- [backend/src/models/prospect.model.js](backend/src/models/prospect.model.js)
  - **Deskripsi**: Menambah field survei awal: `coordinate`, `pic_idcard_type`, `pic_idcard_number`, `pic_photo`, `site_photo`.
- [backend/src/services/prospect.service.js](backend/src/services/prospect.service.js)
  - **Deskripsi**: Menyertakan field-field baru pada `PROSPECT_EDITABLE_FIELDS`.
- [backend/src/controllers/files.controller.js](backend/src/controllers/files.controller.js)
  - **Deskripsi**: Menambah `getProspectPicPhoto` (foto KTP dengan watermark nama admin) & `getProspectSitePhoto` (foto lokasi tanpa watermark).
- [backend/src/routes/files.route.js](backend/src/routes/files.route.js)
  - **Deskripsi**: Route `GET /file/prospect-photo/:id` & `GET /file/prospect-site/:id`, keduanya diproteksi `prospect.read`.
- [backend/src/config/privilege.json](backend/src/config/privilege.json)
  - **Deskripsi**: Merapikan blok privilege `prospect` (menambah key `read`) dan memindahkan posisinya di dekat modul terkait.
- [backend/src/routes/workOrder.route.js](backend/src/routes/workOrder.route.js)
  - **Deskripsi**: Route `POST /work-order/create-ticket/:id` (privilege `ticketInstallation.create`) — konversi WO ke tiket pemasangan.
- [backend/src/controllers/workOrder.controller.js](backend/src/controllers/workOrder.controller.js)
  - **Deskripsi**: Menambah `createInstallationTicket` (guard `TICKET_ALREADY_EXISTS`, resolve parent customer/prospect); penyesuaian pembuatan WO memakai `createWorkOrderOnly`.
- [backend/src/services/workOrder.service.js](backend/src/services/workOrder.service.js)
  - **Deskripsi**: Memisah `createWorkOrderOnly` dan menambah `createTicketForWorkOrder` (menyimpan ref dua arah `wo.ticket ↔ ticket.work_order`, append nomor WO ke deskripsi tiket).
- [backend/src/models/workOrder.model.js](backend/src/models/workOrder.model.js)
  - **Deskripsi**: Field relasi `ticket` pada Work Order.
- [backend/src/controllers/customerQuotation.controller.js](backend/src/controllers/customerQuotation.controller.js) · [backend/src/models/customerQuotation.model.js](backend/src/models/customerQuotation.model.js) · [backend/src/services/customerQuotation.service.js](backend/src/services/customerQuotation.service.js)
  - **Deskripsi**: Menambah data PIC penandatangan penawaran (`pic_name`, `pic_position`, `pic_phone`, `pic_email`, `pic_signature`) — snapshot otomatis dari pembuat dokumen atau override manual dari form.
- [backend/src/controllers/publicQuotation.controller.js](backend/src/controllers/publicQuotation.controller.js) · [backend/src/controllers/publicCustomerSO.controller.js](backend/src/controllers/publicCustomerSO.controller.js)
  - **Deskripsi**: Penyesuaian dokumen publik — pembuatan `po_number` & `share_token` pada alur penerimaan penawaran publik.
- [backend/src/models/customerPO.model.js](backend/src/models/customerPO.model.js) · [backend/src/models/customerSO.model.js](backend/src/models/customerSO.model.js) · [backend/src/services/customerDocument.service.js](backend/src/services/customerDocument.service.js)
  - **Deskripsi**: Penyesuaian model & service dokumen customer PO/SO agar konsisten dengan alur prospek/penawaran.
- [backend/src/locales/en/translation.json](backend/src/locales/en/translation.json) · [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json)
  - **Deskripsi**: String i18n backend baru (mis. `prospect.referalNotFound`, `workOrder.ticketCreated`).

### 🖥️ Frontend — Berkas Baru

- [frontend/src/app/pages/public/prospectRegistration.jsx](frontend/src/app/pages/public/prospectRegistration.jsx) **[NEW]**
  - **Deskripsi**: Halaman publik form pendaftaran prospek (route `register/:referal?`) — input data, jenis & nomor KTP, unggah foto (`InputImage`), pemilih peta (`InputMap`), pemilih bahasa, submit ke `POST /prospect/register`.
- [frontend/src/app/pages/services/quotation/QuotationPreviewModal.jsx](frontend/src/app/pages/services/quotation/QuotationPreviewModal.jsx) **[NEW]**
  - **Deskripsi**: Modal pratinjau langsung dokumen quotation (tanpa membuka drawer detail) saat nomor quotation diklik di tabel dokumen prospek/customer.
- [frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx) **[NEW]**
  - **Deskripsi**: Komponen lembar dokumen Work Order — dokumen operasional murni tanpa informasi harga (FR-6); dipakai bersama oleh halaman cetak dan modal pratinjau.
- [frontend/src/app/pages/services/workOrder/WorkOrderPreviewModal.jsx](frontend/src/app/pages/services/workOrder/WorkOrderPreviewModal.jsx) **[NEW]**
  - **Deskripsi**: Modal pratinjau lembar Work Order saat nomor WO diklik di tabel, dengan aksi cetak.
- [frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx](frontend/src/app/pages/services/workOrder/WorkOrderDetailDrawer.jsx) **[NEW]**
  - **Deskripsi**: Drawer detail Work Order (info customer/prospek, SO, tiket, status) tanpa berpindah halaman.
- [frontend/src/components/shared/table/DocumentActionsMenu.jsx](frontend/src/components/shared/table/DocumentActionsMenu.jsx) **[NEW]**
  - **Deskripsi**: Menu "Pilih Aksi" untuk baris tabel dokumen yang dirender manual (bukan Datatables), meniru pola `RowActions` vendor; `actions` kondisional (entri falsy dilewati).

### 🖥️ Frontend — Berkas yang Berubah

- [frontend/src/app/router/public.jsx](frontend/src/app/router/public.jsx)
  - **Deskripsi**: Registrasi route publik `register/:referal?` → `prospectRegistration`.
- [frontend/src/app/pages/services/prospect/create.jsx](frontend/src/app/pages/services/prospect/create.jsx) · [edit.jsx](frontend/src/app/pages/services/prospect/edit.jsx) · [detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx) · [index.jsx](frontend/src/app/pages/services/prospect/index.jsx)
  - **Deskripsi**: Menambah field survei (koordinat, KTP PIC, unggah foto) pada form create/edit; `detail.jsx` (perubahan terbesar, ±1.000 baris) menampilkan foto/dokumen prospek serta tab dokumen (quotation/WO) dengan pratinjau; penyesuaian daftar.
- [frontend/src/app/pages/services/prospect/schema/prospectSchema.js](frontend/src/app/pages/services/prospect/schema/prospectSchema.js) · [schema/columns.jsx](frontend/src/app/pages/services/prospect/schema/columns.jsx)
  - **Deskripsi**: Menambah validasi field baru pada schema & kolom tabel.
- [frontend/src/app/pages/services/quotation/create.jsx](frontend/src/app/pages/services/quotation/create.jsx) · [quotationSchema.js](frontend/src/app/pages/services/quotation/quotationSchema.js) · [QuotationDocumentPreview.jsx](frontend/src/app/pages/services/quotation/QuotationDocumentPreview.jsx)
  - **Deskripsi**: Field PIC penandatangan pada form & schema quotation; dokumen penawaran menampilkan blok tanda tangan/PIC.
- [frontend/src/app/pages/services/workOrder/detail.jsx](frontend/src/app/pages/services/workOrder/detail.jsx) · [index.jsx](frontend/src/app/pages/services/workOrder/index.jsx) · [print.jsx](frontend/src/app/pages/services/workOrder/print.jsx) · [schema/columns.jsx](frontend/src/app/pages/services/workOrder/schema/columns.jsx)
  - **Deskripsi**: Integrasi pratinjau/drawer/dokumen WO, aksi konversi ke tiket, `print.jsx` memakai komponen `WorkOrderDocument` bersama.
- [frontend/src/app/pages/services/customerManagement/detail.jsx](frontend/src/app/pages/services/customerManagement/detail.jsx) · [customerSalesOrder/CustomerSODocumentPreview.jsx](frontend/src/app/pages/services/customerSalesOrder/CustomerSODocumentPreview.jsx) · [public/PublicCustomerSODocument.jsx](frontend/src/app/pages/public/PublicCustomerSODocument.jsx) · [public/PublicQuotationDocument.jsx](frontend/src/app/pages/public/PublicQuotationDocument.jsx)
  - **Deskripsi**: Penyesuaian tampilan dokumen customer/SO/quotation (termasuk versi publik) agar konsisten dengan blok PIC & aksi dokumen baru.
- [frontend/src/components/shared/form/Combobox.jsx](frontend/src/components/shared/form/Combobox.jsx)
  - **Deskripsi**: Penyesuaian kecil komponen Combobox.
- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) · [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: String i18n frontend baru (±76 baris masing-masing) untuk pendaftaran prospek, PIC penawaran, dokumen & tiket Work Order.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Calon pelanggan dapat mendaftar sebagai prospek sendiri melalui link publik referal marketing (tanpa login), lengkap dengan foto KTP PIC, foto lokasi, koordinat, dan data identitas.
  - Admin/marketing dapat melihat foto survei prospek (KTP ber-watermark nama admin, foto site) langsung dari halaman detail.
  - Penawaran (quotation) kini memuat data PIC penandatangan + tanda tangan.
  - Tim dapat mempratinjau dokumen Quotation & Work Order langsung dari tabel (modal/drawer), serta mengonversi Work Order menjadi Tiket Pemasangan.
- **Bug Fix / Solusi Masalah**: Merapikan definisi privilege `prospect` di `privilege.json` (menambah key `read` yang sebelumnya belum lengkap). Guard `TICKET_ALREADY_EXISTS` mencegah pembuatan tiket ganda dari satu Work Order.
- **Menu/Tombol Baru**: Menu aksi dokumen (`DocumentActionsMenu`) pada tabel dokumen; tombol pratinjau/cetak Work Order; tombol konversi WO → Tiket; form pendaftaran prospek publik (`/register/:referal`).

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Fase dokumen dari modul Prospect Management menghubungkan funnel pra-deal (prospek → quotation → work order → tiket pemasangan) menjadi satu alur dokumen. Pendaftaran prospek publik memakai pola link referal yang sama seperti registrasi pelanggan yang sudah ada, sementara Work Order memiliki lembar dokumen operasional terpisah yang sengaja tidak menampilkan harga.
- **Langkah Penggunaan (Tutorial)**:
  1. **Daftar prospek (publik)**: buka `/register/<admin_id_marketing>`, isi data + unggah foto KTP PIC & lokasi, pilih titik peta, submit. Prospek muncul di daftar dengan `source = Form Online` dan `created_by` = marketing terkait.
  2. **Lihat foto survei**: buka detail prospek → foto KTP tampil ber-watermark nama admin, foto site tampil apa adanya.
  3. **Pratinjau dokumen**: pada tabel dokumen prospek/customer, klik nomor Quotation/WO untuk membuka modal pratinjau; pada tabel Work Order klik nomor WO untuk membuka drawer/pratinjau dan mencetak lembar WO.
  4. **Konversi ke tiket**: dari Work Order, gunakan aksi konversi (`POST /work-order/create-ticket/:id`) untuk membuat Tiket Pemasangan bagi tim NOC; percobaan kedua ditolak (`TICKET_ALREADY_EXISTS`).
