# 📝 Daily Work Report - Idham (2026-07-14)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Management — Refactor Komponen Dokumen, Preview Modal WO & Quotation, Registrasi Prospek Publik

## 📅 Laporan Harian - 14 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Seluruh pekerjaan hari ini masih berstatus WIP.

---

### 🏗️ Konteks Pekerjaan Hari Ini

Hari ini berfokus pada **polish dan DX improvement** :
1. **Ekstrak `WorkOrderDocument` sebagai shared component** — menghilangkan duplikasi antara halaman cetak dan modal pratinjau WO.
2. **Preview modal WO dan Quotation** — menghadirkan pola "klik nomor dokumen → pratinjau inline" yang sudah ada di halaman vendor ke modul baru (Work Order + Quotation prospek).
3. **Shared `DocumentActionsMenu`** — komponen dropdown aksi dokumen yang reusable untuk tabel dokumen yang di-render manual (tab Quotation/PO/SO di detail prospek/customer).
4. **Update `QuotationDocumentPreview`** — sekarang mendukung render quotation milik prospek (bukan hanya customer).
5. **Halaman registrasi prospek publik** — marketing bisa share link ke calon pelanggan; prospek mengisi data sendiri tanpa perlu PIC input manual.

---

### 🔄 Refactor — WorkOrderDocument (Ekstrak Shared Component)

- `frontend/src/app/pages/services/workOrder/WorkOrderDocument.jsx` [NEW] (+117 baris)
  - **Deskripsi**: Komponen `WorkOrderDocument` diekstrak dari `print.jsx` menjadi shared component terpisah. Sebelumnya logika render dokumen WO berada di `print.jsx` secara inline.

    **Layout dokumen** (tanpa harga — keputusan terkunci #5):
    - Header: nomor WO + tanggal + status badge
    - Tabel info dua kolom: customer, nomor SO, tiket pemasangan, site address/coordinate, PIC lapangan, target date, tim assigned, catatan
    - Tabel `service_items`: No / Nama / Qty / Tech Spec (empat kolom, tanpa kolom harga)
    - Kolom tanda tangan serah terima: "Tim Delivery" dan "Customer" dengan garis kosong untuk tanda tangan basah

    **Dipakai bersama oleh**:
    - `print.jsx` — render dokumen di halaman cetak (`@media print`)
    - `WorkOrderPreviewModal.jsx` — render dokumen di dalam modal pratinjau

- `frontend/src/app/pages/services/workOrder/print.jsx` (refactor)
  - **Deskripsi**: Halaman cetak disederhanakan — konten dokumen diganti dengan `<WorkOrderDocument wo={wo} />`. Toolbar (tombol Back + Print via `window.print()`) tetap di sini dan disembunyikan saat cetak via `print:hidden`.

---

### 🆕 Preview Modal Work Order

- `frontend/src/app/pages/services/workOrder/WorkOrderPreviewModal.jsx` [NEW] (+107 baris)
  - **Deskripsi**: Modal pratinjau dokumen WO — meniru pola pratinjau dokumen vendor (klik nomor/status di tabel → modal muncul langsung, tanpa navigasi ke halaman detail):

    **State**: `loading`, `wo` (data WO yang di-fetch)

    **Fetch**: `GET /work-order/view/:woNumber` saat `open + woNumber` berubah. Reset `wo = null` saat modal ditutup.

    **Layout modal**:
    - Backdrop `bg-gray-900/70` + blur
    - Header di luar `DialogPanel`: tombol Print (link ke halaman print) di kiri, tombol ✕ di kanan
    - Isi: `<WorkOrderDocument wo={wo} />` dalam shadow container
    - Loading state: `<Spinner>` di tengah

    **Transisi**: Headless UI `Transition` + `TransitionChild` (fade + scale) — konsisten dengan pola modal existing.

- `frontend/src/app/pages/services/workOrder/index.jsx` (update)
  - **Deskripsi**: Halaman list WO diperbarui untuk menggunakan preview modal:
    - State baru: `woPreviewNumber` (string nomor WO yang sedang di-preview, atau `null`)
    - `getWorkOrderColumns` sekarang menerima callback `(wo) => setWoPreviewNumber(wo.wo_number)` sebagai `onPreview`
    - `<WorkOrderPreviewModal>` ditambahkan di bawah tabel

- `frontend/src/app/pages/services/workOrder/schema/columns.jsx` (update)
  - **Deskripsi**: Kolom tabel WO diperbarui:
    - `getWorkOrderColumns(t, onPreview)` — parameter `onPreview` opsional
    - Kolom `wo_number`: jika `onPreview` ada → render `<span onClick>` clickable; jika tidak → render `<OpenLink>` biasa
    - Kolom `status` badge: jika `onPreview` ada → badge dibungkus `<span onClick>` (pola yang sama dengan kolom vendor)
    - Efek: klik nomor WO atau badge status di list → buka preview modal

---

### 🆕 Preview Modal Quotation

- `frontend/src/app/pages/services/quotation/QuotationPreviewModal.jsx` [NEW] (+101 baris)
  - **Deskripsi**: Modal pratinjau dokumen Quotation untuk dipakai di tabel dokumen prospek/customer — meniru pola `WorkOrderPreviewModal`:

    **Fetch**: `GET /customer-quotation/view/${encodeURIComponent(quotationNumber)}` — `encodeURIComponent` wajib karena nomor quotation mengandung karakter `/` (contoh: `QTN/00001/06/2026`).

    Juga fetch `companyInfo` (via `fetchCompanyInfo()`) untuk menampilkan kop surat.

    **Isi**: render `<QuotationDocumentPreview quotation={quotation} companyInfo={companyInfo} />` — komponen yang sama dengan halaman publik quotation. Tanpa `customerSignatureSlot` (tidak perlu slot tanda tangan di pratinjau).

    **Tidak ada tombol Print** — pratinjau quotation hanya untuk melihat, berbeda dengan WO yang punya tombol Print karena dibagikan ke lapangan.

---

### 🔧 Update QuotationDocumentPreview — Dukungan Prospek

- `frontend/src/app/pages/services/quotation/QuotationDocumentPreview.jsx` (modifikasi)
  - **Deskripsi**: Komponen layout dokumen quotation diperbarui untuk mendukung render quotation milik **prospek** (bukan hanya customer):

    **Sebelum**: Penerima surat selalu diambil dari `quotation.customer` (data Partner).

    **Sesudah**: Logika fallback ke prospek:
    ```js
    const prospect = quotation.prospect || {};
    const recipientContact = prospect.pic_name || '';
    const recipientCompany = customer.name || prospect.name || '-';
    const recipientAddress = customer.address || '';
    ```
    Jika quotation milik prospek (belum dikonversi), nama perusahaan diambil dari `prospect.name`, PIC dari `prospect.pic_name`.

    Pengaruh: modal pratinjau quotation di tab Quotation halaman detail prospek kini menampilkan data penerima yang benar meski customer belum terbentuk.

- `frontend/src/app/pages/public/PublicQuotationDocument.jsx` (modifikasi)
  - **Deskripsi**: Halaman publik quotation (untuk tanda tangan pelanggan via share_token) diperbarui mengikuti perubahan `QuotationDocumentPreview`. Import `QuotationDocumentPreview` dari path yang sama; `customerSignatureSlot` masih dipakai untuk area interaktif tanda tangan.

---

### 🆕 Shared Component — DocumentActionsMenu

- `frontend/src/components/shared/table/DocumentActionsMenu.jsx` [NEW] (+67 baris)
  - **Deskripsi**: Komponen dropdown "Pilih Aksi" yang reusable untuk baris tabel dokumen yang di-render manual (bukan DataTables) — dipakai di tab Quotation/PO/SO di halaman detail prospek dan tab WO di halaman detail customer:

    **Props**: `actions` — array objek `{ key, label, icon, onClick, className }`. Entri `falsy` dilewati sehingga pemanggil bisa menyusun aksi secara kondisional:
    ```js
    actions={[
      canSend && { key: 'send', label: t('prospect.docs.send'), icon: PaperAirplaneIcon, onClick: () => ... },
      canDelete && { key: 'delete', label: t('global.delete'), className: 'text-error-600', onClick: () => ... },
    ]}
    ```

    **Tampilan**: mengikuti pola `RowActions` existing — tombol `GoMultiSelect + "Pilih Aksi"` sebagai trigger, dropdown posisi `bottom end` via Headless UI `Menu`.

    Jika `items.length === 0` (semua entri falsy) → return `null` — tidak ada tombol yang muncul.

---

### 🆕 Halaman Registrasi Prospek Publik

- `frontend/src/app/pages/public/prospectRegistration.jsx` [NEW] (+209 baris)
  - **Deskripsi**: `ProspectRegistration` — halaman publik (tanpa login) untuk self-registration prospek oleh calon pelanggan. Meniru pola `registration.jsx` yang sudah ada:

    **Konteks penggunaan**: Marketing membagikan link dengan format `/register/:admin_id` kepada calon pelanggan. Prospek mengisi form sendiri → data tersimpan dengan atribusi ke marketing yang membagikan link (`referal = admin_id`).

    **Field**:
    - Wajib: `name` (nama perusahaan/instansi), `pic_name`, `phone`
    - Opsional: `email`, `city`, `address`, `notes`

    **Validasi**: Yup schema yang sama dengan form "Buat Prospek" internal (`prospect/schema/prospectSchema.js`).

    **Submit**: `POST /prospect/register` dengan `data.referal = referal` dari URL param.

    **UX**:
    - Kop logo perusahaan dari server (`${SERVER_URL}/file/logo/landscape`)
    - Checkbox persetujuan Terms & Privacy wajib dicentang sebelum tombol Submit aktif
    - `SuccessModalAlert` muncul setelah berhasil, form di-reset
    - `LanguageSelector` di pojok kanan atas (bilingual EN/ID)
    - Server-side error di-map ke field form via `setError`

- `frontend/src/app/router/public.jsx` (update)
  - **Deskripsi**: Tambah route publik baru:
    - `path: 'register/:referal?'` → `ProspectRegistration` (lazy import)
    - Ditempatkan sejajar dengan `registration/:referal?` yang sudah ada (untuk Customer existing), bukan menggantikannya

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **UX preview dokumen konsisten**: Pola "klik nomor/status → pratinjau modal inline" kini tersedia di semua modul dokumen — vendor (sudah ada), Work Order (baru), dan Quotation (baru). Tidak perlu buka halaman baru hanya untuk melihat isi dokumen.
- **DRY pada layout dokumen WO**: `WorkOrderDocument` di-render oleh dua jalur berbeda (print page + preview modal) dari satu sumber. Perubahan layout dokumen cukup di satu tempat.
- **Quotation bisa dirender untuk prospek**: `QuotationDocumentPreview` kini handle kasus quotation yang belum punya customer resmi — penerima diambil dari data prospek. Halaman publik quotation tidak terpengaruh.
- **`DocumentActionsMenu` mengurangi boilerplate**: Komponen shared menggantikan implementasi Headless UI Menu berulang di setiap tab dokumen manual. Aksi disusun secara kondisional langsung di props.
- **Akuisisi prospek tanpa input manual PIC**: Marketing cukup share link sekali → prospek isi sendiri → masuk antrian prospek `new` dengan atribusi ke marketing yang membagikan. Mengurangi beban input PIC untuk prospek masuk dari campaign/referral.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Pratinjau dokumen WO dari list**:
- Buka menu **Work Order** → klik nomor WO atau badge status di baris tabel → modal pratinjau terbuka langsung tanpa navigasi
- Tombol Print di header modal membuka halaman cetak

**Pratinjau Quotation dari tab prospek**:
- Di halaman detail prospek → tab **Quotation** → klik nomor quotation → modal pratinjau terbuka

**Self-registration prospek**:
- Marketing buka profil mereka → salin link `/register/:admin_id`
- Bagikan ke calon pelanggan (email/WA/medsos)
- Calon pelanggan buka link → isi form (nama, PIC, telepon, dll) → centang persetujuan → kirim
- Prospek masuk otomatis di daftar Prospect dengan `status = new` dan `source` teratribusi ke marketing
