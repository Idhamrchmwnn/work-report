# 📝 Daily Work Report - Idham (2026-07-11)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Management — Dokumen (Quotation/PO/SO) Terhubung ke Prospek (XOR prospect/customer)

## 📅 Laporan Harian - 11 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Seluruh pekerjaan hari ini masih berstatus WIP.

---

### 🏗️ Konteks & Arsitektur Fase 2

Tambahan **ikatan dokumen komersial ke entitas Prospek**, sebelum konversi menjadi customer resmi. Sebelumnya, Quotation/PO/SO hanya bisa dibuat untuk Partner (customer). Sekarang bisa dibuat untuk Prospek (entitas pra-deal) dengan pola **XOR (exclusive OR)**: setiap dokumen harus punya tepat satu induk — boleh Prospect *atau* Customer, tidak boleh keduanya saat pembuatan.

**Rantai dokumen yang dibangun**:
```
Prospect → Quotation → Customer PO (nilai po_value vs. grand_total quotation)
                     → Customer SO (dibuat dari quotation yang won)
```

Setelah konversi di Fase 3, field `customer` pada dokumen-dokumen ini diisi tanpa menghapus `prospect` (dipertahankan sebagai histori).

---

### 🏗️ Backend — Perubahan Schema Model

- `backend/src/models/customerQuotation.model.js` (+40 baris)
  - **Deskripsi**: Tiga penambahan:
    1. Field `prospect` (ref Prospect, opsional) — induk dokumen saat dibuat dari prospek
    2. `customer` tidak lagi `required` (XOR dengan `prospect`)
    3. Export `QUOTATION_STATUSES` enum: `draft, sent, approved, rejected, won, lost, expired`
    4. Field `status` (default `draft`) dengan enum `QUOTATION_STATUSES`
    5. Pre-validate hook XOR: saat `isNew`, jika keduanya terisi → error `PARENT_XOR`; jika keduanya kosong → error `REQUIRED`

- `backend/src/models/customerPO.model.js` (+32 baris)
  - **Deskripsi**: Empat penambahan:
    1. Field `prospect` (ref Prospect, opsional) + `customer` tidak lagi `required`
    2. Field `quotation` (ref CustomerQuotation, **wajib**) — PO wajib menempel ke quotation; pembuatan PO men-set quotation → `won`
    3. Field `po_value` — nilai PO menurut dokumen customer (input PIC). Dicek terhadap `grand_total` quotation; selisih hanya warning, tidak memblokir.
    4. Field `value_mismatch` (Boolean, default `false`) — penanda bahwa nilai PO berbeda dari total quotation
    5. Pre-validate hook XOR identik dengan Quotation

- `backend/src/models/customerSO.model.js` (+45 baris)
  - **Deskripsi**: Lima penambahan:
    1. Field `prospect` + `customer` tidak `required` + pre-validate hook XOR
    2. Field `quotation` (ref CustomerQuotation) — SO dibuat dari quotation `won`
    3. Field `po` (ref CustomerPO) — ref PO yang memicu pembuatan SO (opsional)
    4. Export `SO_STATUSES`: `draft, sent, signed`
    5. Field `status` (default `draft`) — siklus hidup SO: draft → sent → signed
    6. Field `signed_file_uploaded` (Boolean) — penanda jalur tanda tangan basah (scan manual diunggah, bukan digital)

---

### 🛠️ Backend — Perubahan Service Layer

- `backend/src/services/customerQuotation.service.js` (+75 baris)
  - **Deskripsi**: Tiga penambahan:
    1. **`applyExpiredStatus(quotations)`** — Pengecekan kedaluwarsa secara *lazy* (saat dibaca, bukan via cron): quotation berstatus `draft` atau `sent` yang melewati `valid_until` di-update ke `expired` dalam satu `updateMany`. Mutasi objek in-memory juga dilakukan agar response langsung menampilkan status yang benar tanpa query ulang.
    2. **`findQuotationsByParent(parentFilter)`** — Daftar quotation berdasarkan induk (`{ prospect }` atau `{ customer }`), populate `prospect` dengan field ringkas, terapkan `applyExpiredStatus`.
    3. Update `findQuotationById` dan `findQuotationsByCustomer`: tambah `.populate('prospect', ...)` dan panggil `applyExpiredStatus` di akhir.

- `backend/src/services/customerDocument.service.js` (+55 baris)
  - **Deskripsi**: Tambah fungsi list PO dan SO berdasarkan induk prospect untuk mendukung tab dokumen di halaman detail prospek: `findCustomerPOsByParent(filter)` dan `findCustomerSOsByParent(filter)` — query dengan filter `{ prospect }` atau `{ customer }`.

---

### 🔧 Backend — Perubahan Controller

- `backend/src/controllers/customerQuotation.controller.js` (+115 baris)
  - **Deskripsi**: Dua perubahan:
    1. **`createQuotation` direfaktor** — tambah validasi XOR `prospect_id`/`customer_id` di awal: keduanya ada → error `PARENT_XOR`; keduanya kosong → error `PARENT_REQUIRED`. Lookup prospect jika `prospect_id` dikirim (guard: belum dikonversi). Set `prospect` atau `customer` di data quotation sesuai induk. Setelah berhasil, panggil `advanceProspectStatus(prospect, 'quoted', ['new', 'contacted'])` — fungsi internal yang memajukan status funnel prospek ke `quoted` hanya jika saat ini masih `new` atau `contacted` (tidak pernah menurunkan `won`/`lost`).
    2. **`listQuotationByParent`** — Handler baru: list quotation per induk via query `?prospect=` atau `?customer=`. Lookup entitas induk dulu untuk validasi, lalu panggil `findQuotationsByParent`.

- `backend/src/controllers/customerDocument.controller.js` (+270 baris)
  - **Deskripsi**: Update controller PO dan SO untuk mendukung induk prospect:
    - `createPO` dan `createSO` diperbarui: terima `prospect_id` atau `customer_id` (XOR). Saat PO dibuat dari quotation, set status quotation → `won` via `updateQuotationStatus`.
    - Handler baru `listPOByParent` dan `listSOByParent` via query `?prospect=` atau `?customer=` untuk tab dokumen di detail prospek.
    - Status SO dikelola: `createSO` set `status = 'draft'`; `submitSO` (kirim ke customer) set `status = 'sent'`; endpoint sign (pelanggan tanda tangan) set `status = 'signed'`.

---

### 🛣️ Backend — Perubahan Route

- `backend/src/routes/customerQuotation.route.js` (+19 baris)
  - **Deskripsi**: Tambah route baru: `GET /customer-quotation/list` dengan privilege `quotation.read` — list quotation per induk via query `?prospect=` atau `?customer=`. Dipakai oleh tab Quotation di halaman detail prospek.

- `backend/src/routes/customerDocument.route.js` (+46 baris)
  - **Deskripsi**: Tambah route list per parent untuk PO dan SO: `GET /customer-po/list?prospect=&customer=` dan `GET /customer-so/list?prospect=&customer=`.

- `backend/src/routes/files.route.js` (+8 baris)
  - **Deskripsi**: Tambah route akses file dokumen berdasarkan konteks prospect (file PO/SO yang diunggah dari prospek).

---

### 🎨 Frontend — Form & Schema

- `frontend/src/app/pages/services/quotation/create.jsx` (+11 baris)
  - **Deskripsi**: Tambah prop `prospectId` — saat form dibuka dari konteks prospek, `prospect_id` ter-set di form data dan `customer_id` tidak wajib diisi.

- `frontend/src/app/pages/services/quotation/quotationSchema.js` (+12 baris)
  - **Deskripsi**: Tambah `prospect_id` (opsional) ke schema Yup. Field `customer_id` berubah menjadi conditional: **wajib** hanya jika `prospect_id` kosong (via `.when('prospect_id', ...)`). Ini mencerminkan pola XOR di level validasi frontend.

- `frontend/src/app/pages/services/customerPurchaseOrder/create.jsx` (+115 baris)
  - **Deskripsi**: Tambah prop `prospectId` — form upload Customer PO sekarang bisa dipakai dari konteks prospek. Tambah field `quotation_id` (pilih quotation yang akan dijadikan induk PO) dan `po_value` (nilai PO menurut dokumen customer, dibandingkan dengan total quotation untuk deteksi mismatch).

- `frontend/src/app/pages/services/customerSalesOrder/create.jsx` (+132 baris)
  - **Deskripsi**: Tambah prop `prospectId` — form Customer SO sekarang bisa dipakai dari konteks prospek. Tambah field `quotation_id` untuk ikatan SO ke quotation, serta field `po_id` (opsional, SO bisa dibuat tanpa PO jika punya privilege `customer_so.withoutPo`).

- `frontend/src/app/pages/services/customerManagement/schema/customerPOSchema.js` (+18 baris)
  - **Deskripsi**: Tambah `quotation_id` (wajib) dan `po_value` (angka opsional) ke Yup schema Customer PO.

- `frontend/src/app/pages/services/customerManagement/schema/customerSOSchema.js` (+23 baris)
  - **Deskripsi**: Tambah `quotation_id` (wajib) dan `po_id` (opsional) ke Yup schema Customer SO.

- `frontend/src/app/pages/services/prospect/schema/statusOptions.js` (+27 baris)
  - **Deskripsi**: Tambah dua set konstanta status untuk dokumen di tab prospek:
    - `quotationStatusOptions` — 7 status (draft/sent/approved/rejected/won/lost/expired) dengan warna badge
    - `soStatusOptions` — 3 status (draft/sent/signed)
    - Helper `getQuotationStatusOption` dan `getSOStatusOption`

---

### 🎨 Frontend — Tab Dokumen di Halaman Detail Prospek

- `frontend/src/app/pages/services/prospect/detail.jsx` (+684 baris tambahan hari ini)
  - **Deskripsi**: Penambahan besar pada halaman detail prospek — integrasi **TabGroup** dari Headless UI dengan 3 tab dokumen:

    **State baru yang ditambahkan**:
    - Quotation: `quotations`, `loadingQuotations`, `createQuotationOpen`, `quotationDetailOpen`, `selectedQuotation`, `sendingQuotationId`
    - PO: `poList`, `loadingPO`, `createPOOpen`, `poPreview`
    - SO: `soList`, `loadingSO`, `createSOOpen`, `soPreview`, `sendingSOId`, `uploadSignedTarget`, `signedFile`, `isUploadingSigned`
    - Convert: `convertOpen`

    **Fetch handler baru**: `fetchQuotations`, `fetchPOList`, `fetchSOList` — masing-masing hit endpoint `?prospect={id}`.

    **Privilege baru**: `canConvert` (`prospect.convert`), `canCreateDoc` (`quotation.create`), `canSendQuotation` (`quotation.update`), `canSendSO` (`customer_so.send`).

    **Tab Quotation**: List quotation milik prospek, badge status, nilai total, tombol kirim ke customer, tombol lihat detail (`QuotationDetailDrawer`).

    **Tab Customer PO**: List PO milik prospek, status approval, nama file dokumen, tombol preview via `DocumentPreviewModal`.

    **Tab Customer SO**: List SO milik prospek, status (draft/sent/signed), `formatMoney` untuk total, tombol kirim, tombol upload tanda tangan basah (scan), tombol lihat dokumen.

    **Drawer yang diimport**: `CreateQuotationDrawer`, `QuotationDetailDrawer`, `CreateCustomerPODrawer`, `CreateCustomerSODrawer`, `ProspectConvertDrawer`.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **Dokumen pra-deal tersedia**: PIC/sales kini bisa membuat Quotation, PO, dan SO langsung dari halaman prospek — tanpa harus mendaftarkan prospek sebagai customer resmi terlebih dahulu. Deal bisa diproses penuh dari lead hingga SO sebelum konversi.
- **Funnel status otomatis**: Saat quotation pertama dibuat, status prospek naik ke `quoted`. Saat PO diterima (quotation menjadi `won`), funnel berlanjut ke `negotiation` (dikerjakan di controller). Status tidak pernah turun jika sudah di status lanjut.
- **Validasi nilai PO**: Jika nilai PO yang diinput berbeda dari total quotation, sistem menyimpan `value_mismatch = true` sebagai flag warning tanpa memblokir proses.
- **Status kedaluwarsa otomatis**: Quotation yang melewati tanggal berlaku otomatis berubah status ke `expired` saat dibaca — tanpa perlu cron job terpisah.
- **Rantai dokumen terjaga**: SO dibuat dari quotation yang sudah `won`. PO wajib menempel ke quotation. Rantai ini memastikan traceability dokumen komersial penuh.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Alur kerja baru**:
1. PIC buka halaman detail prospek → Tab **Quotation** → klik **Buat Quotation**
2. Quotation dibuat → status prospek otomatis naik ke `quoted`
3. Customer setuju → PIC input Customer PO (upload PDF dokumen PO) → pilih quotation induk → input nilai PO
4. Sistem cek: nilai PO vs total quotation. Selisih → muncul warning `value_mismatch`, proses tetap berlanjut.
5. Quotation otomatis berubah ke status `won`
6. PIC buat Customer SO dari tab SO → pilih quotation yang sudah `won`
7. SO dikirim ke customer (status `sent`) → customer tanda tangan digital (status `signed`)
8. Setelah SO `signed`,
