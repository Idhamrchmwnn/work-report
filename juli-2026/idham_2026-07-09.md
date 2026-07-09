# 📝 Daily Work Report - Idham (2026-07-09)

---

## 📌 Informasi Issue
- **Nomor Issue**: #118
- **Judul Issue**: Vendor Document Management — Refactor Hak Akses Vendor PO & SO ke Privilege Group Tersendiri

## 📅 Laporan Harian - 9 Juli 2026

### ✅ Rincian Commit

**Branch**: `issue-118`
**Commit**: `79fa498 resolve #118`
**13 file diubah, 67 penambahan, 39 penghapusan**

---

### 🏗️ Konteks & Latar Belakang

Sebelum perbaikan ini, seluruh operasi Vendor PO dan Vendor SO menggunakan privilege grup `vendor.*` yang sama dengan operasi data master vendor (`vendor.create`, `vendor.read`, dll.). Ini berarti admin yang punya akses baca vendor otomatis bisa melihat semua dokumen PO/SO, dan admin yang punya akses ubah vendor otomatis bisa approve atau reject dokumen PO/SO — padahal keduanya seharusnya bisa dikonfigurasi secara independen.

Commit ini memisahkan privilege PO dan SO menjadi dua grup tersendiri: `purchaseOrder.*` dan `salesOrder.*` — sehingga admin dapat dikonfigurasi hak aksesnya secara granular per tipe dokumen.

---

### 🔑 Perubahan Hak Akses (Backend)

- `backend/src/config/privilege.json` (+14 baris)
  - **Deskripsi**: Tambah dua grup privilege baru yang terpisah dari `vendor.*`:

    ```json
    "purchaseOrder": {
      "read": "PURCHASEORDER_READ",
      "create": "PURCHASEORDER_CREATE",
      "update": "PURCHASEORDER_UPDATE",
      "changeStatus": "PURCHASEORDER_CHANGESTATUS",
      "delete": "PURCHASEORDER_DELETE"
    },
    "salesOrder": {
      "read": "SALESORDER_READ",
      "create": "SALESORDER_CREATE",
      "update": "SALESORDER_UPDATE",
      "changeStatus": "SALESORDER_CHANGESTATUS",
      "delete": "SALESORDER_DELETE"
    }
    ```

    Total 10 privilege baru. Grup `vendor.*` tetap ada untuk akses data master vendor (CRUD data vendor), tidak tumpang tindih dengan dokumen.

---

### 🔒 Pembaruan Route Backend

- `backend/src/routes/purchaseOrder.route.js` (+12, -12 baris)
  - **Deskripsi**: Ganti seluruh privilege check di 12 route Vendor PO dari `vendor.*` ke `purchaseOrder.*`:

    | Route | Sebelum | Sesudah |
    |-------|---------|---------|
    | `POST /vendor-po/create` | `vendor.create` | `purchaseOrder.create` |
    | `POST /vendor-po/list-all` | `vendor.read` | `purchaseOrder.read` |
    | `GET /vendor-po/list/:vendor_id` | `vendor.read` | `purchaseOrder.read` |
    | `GET /vendor-po/view/:id` | `vendor.read` | `purchaseOrder.read` |
    | `PATCH /vendor-po/submit/:id` | `vendor.update` | `purchaseOrder.update` |
    | `POST /vendor-po/request-preview` | `vendor.read` | `purchaseOrder.read` |
    | `PATCH /vendor-po/approve/:id` | `vendor.changeStatus` | `purchaseOrder.changeStatus` |
    | `PATCH /vendor-po/reject/:id` | `vendor.changeStatus` | `purchaseOrder.changeStatus` |
    | `DELETE /vendor-po/delete/:id` | `vendor.delete` | `purchaseOrder.delete` |
    | `PATCH /vendor-po/update/:id` | `vendor.update` | `purchaseOrder.update` |
    | `POST /vendor-po/attachment/:id` | `vendor.update` | `purchaseOrder.update` |
    | `DELETE /vendor-po/attachment/:id/:filename` | `vendor.update` | `purchaseOrder.update` |

- `backend/src/routes/salesOrder.route.js` (+10, -10 baris)
  - **Deskripsi**: Ganti seluruh privilege check di 10 route Vendor SO dari `vendor.*` ke `salesOrder.*`:

    | Route | Sebelum | Sesudah |
    |-------|---------|---------|
    | `POST /vendor-so/create` | `vendor.create` | `salesOrder.create` |
    | `POST /vendor-so/list-all` | `vendor.read` | `salesOrder.read` |
    | `GET /vendor-so/list/:vendor_id` | `vendor.read` | `salesOrder.read` |
    | `GET /vendor-so/view/:id` | `vendor.read` | `salesOrder.read` |
    | `PATCH /vendor-so/submit/:id` | `vendor.update` | `salesOrder.update` |
    | `POST /vendor-so/request-preview` | `vendor.read` | `salesOrder.read` |
    | `PATCH /vendor-so/approve/:id` | `vendor.changeStatus` | `salesOrder.changeStatus` |
    | `PATCH /vendor-so/reject/:id` | `vendor.changeStatus` | `salesOrder.changeStatus` |
    | `DELETE /vendor-so/delete/:id` | `vendor.delete` | `salesOrder.delete` |
    | `PATCH /vendor-so/update/:id` | `vendor.update` | `salesOrder.update` |

- `backend/src/routes/files.route.js` (+2, -2 baris)
  - **Deskripsi**: Ganti privilege akses file dokumen:
    - `GET /file/vendor-po/:name`: `vendor.read` → `purchaseOrder.read`
    - `GET /file/vendor-so/:name`: `vendor.read` → `salesOrder.read`

---

### 🎨 Pembaruan Frontend

- `frontend/src/app/pages/public/ReviewPOPage.jsx` (+1, -1 baris)
  - **Deskripsi**: Halaman review Vendor PO internal admin — ganti `useHasPrivilege('vendor.changeStatus')` menjadi `useHasPrivilege('purchaseOrder.changeStatus')` untuk kontrol visibilitas tombol approve/reject.

- `frontend/src/app/pages/public/ReviewSOPage.jsx` (+1, -1 baris)
  - **Deskripsi**: Halaman review Vendor SO internal admin — ganti `useHasPrivilege('vendor.changeStatus')` menjadi `useHasPrivilege('salesOrder.changeStatus')`.

- `frontend/src/app/pages/services/activation/schema/poColumns.jsx` (+2, -2 baris)
  - **Deskripsi**: Kolom tabel PO di halaman Aktivasi — ganti role guard aksi:
    - Edit: `vendor.update` → `purchaseOrder.update`
    - Delete: `vendor.delete` → `purchaseOrder.delete`

- `frontend/src/app/pages/services/activation/schema/soColumns.jsx` (+2, -2 baris)
  - **Deskripsi**: Kolom tabel SO di halaman Aktivasi — ganti role guard aksi:
    - Edit: `vendor.update` → `salesOrder.update`
    - Delete: `vendor.delete` → `salesOrder.delete`

- `frontend/src/app/pages/services/vendorManagement/detail.jsx` (+14, -6 baris)
  - **Deskripsi**: Halaman detail vendor — pemisahan privilege terbesar di frontend. Sebelumnya menggunakan `hasCreate`, `hasUpdate`, `hasDelete` dari `vendor.*` untuk semua tombol (baik data vendor maupun dokumen PO/SO). Sekarang ditambahkan 6 hook privilege terpisah:
    ```js
    const hasCreatePO = useHasPrivilege('purchaseOrder.create');
    const hasUpdatePO = useHasPrivilege('purchaseOrder.update');
    const hasDeletePO = useHasPrivilege('purchaseOrder.delete');
    const hasCreateSO = useHasPrivilege('salesOrder.create');
    const hasUpdateSO = useHasPrivilege('salesOrder.update');
    const hasDeleteSO = useHasPrivilege('salesOrder.delete');
    ```
    Semua 6 tombol aksi di panel PO dan SO diganti menggunakan hook yang sesuai.

- `frontend/src/app/router/services/vendorRoute.jsx` (+2, -2 baris)
  - **Deskripsi**: Route halaman create PO dan SO vendor — ganti privilege guard:
    - `/vendor/purchase-order/create`: `vendor.create` → `purchaseOrder.create`
    - `/vendor/sales-order/create`: `vendor.create` → `salesOrder.create`

- `frontend/src/components/shared/table/rows.jsx` (+3, -1 baris)
  - **Deskripsi**: Komponen `VendorApprovalStatusCell` (shared, dipakai di tabel PO dan SO) — ganti dari privilege statis `vendor.changeStatus` menjadi dinamis berdasarkan prop `type`:
    ```js
    const hasUpdatePrivilege = useHasPrivilege(
      type === 'so' ? 'salesOrder.changeStatus' : 'purchaseOrder.changeStatus'
    );
    ```
    Satu komponen kini menangani kedua tipe dokumen dengan privilege yang tepat.

- `frontend/src/i18n/locales/en/translations.json` (+2) & `id/translations.json` (+2)
  - **Deskripsi**: Tambah label nama grup privilege baru di konfigurasi privilege UI:
    - `"purchaseOrder": "Purchase Order"` (EN & ID)
    - `"salesOrder": "Sales Order"` (EN & ID)

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **Pemisahan hak akses PO dan SO dari vendor**: Admin kini bisa dikonfigurasi secara granular — misalnya, admin keuangan bisa punya `purchaseOrder.read` tanpa `vendor.update`, atau admin operasional bisa punya `salesOrder.changeStatus` tanpa akses ke data master vendor.
- **Tidak ada breaking change terhadap data**: Privilege lama `vendor.*` tetap ada untuk akses data master vendor. Superadmin yang sebelumnya sudah punya semua privilege hanya perlu ditambahkan privilege `purchaseOrder.*` dan `salesOrder.*` di konfigurasi role masing-masing.
- **Konsistensi privilege guard**: Seluruh 22 route backend (PO + SO + file), 4 halaman frontend, 2 kolom tabel, dan 1 komponen shared kini menggunakan privilege yang konsisten dan tepat sasaran.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Kenapa dipisahkan?**

Sebelumnya, jika admin perlu mengakses daftar Purchase Order vendor, admin harus diberi privilege `vendor.read` — yang secara tidak langsung juga memberikan akses ke seluruh data master vendor (nama, NPWP, kontak, dll.). Dengan pemisahan ini, admin bisa mendapat akses ke dokumen PO/SO tanpa perlu akses ke data master vendor, sesuai prinsip *least privilege*.

**Yang perlu dikonfigurasi setelah deploy**:
Untuk setiap role admin yang saat ini menggunakan fitur PO/SO vendor, tambahkan privilege group `purchaseOrder` dan/atau `salesOrder` sesuai kebutuhan operasionalnya di halaman manajemen role/akses.
