# 📝 Daily Work Report - Idham (2026-07-21)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Management — Modul Dokumen Penjualan (Quotation → Customer PO → Customer SO) + Rintisan Manajemen Status Pelanggan (Blacklist & Pasif)

## 📅 Laporan Harian - 21 Juli 2026

Hari ini menghasilkan dua bagian pekerjaan yang saling berurutan: **(1)** menuntaskan dan **meng-commit** modul Customer Management yang kemarin masih ter-stage, lalu **(2)** memulai fitur baru **manajemen status pelanggan (Blacklist & Pasif)** yang kini sudah ter-stage penuh dan menunggu commit berikutnya. Branch aktif: `issue-123`, dengan basis di atas pekerjaan tim yang sudah terintegrasi (`resolve #148`, Dedy).

---

### 📅 Rincian Commit

#### [`618da7e`] - resolve #123 (#123 - Customer Management: Quotation → Customer PO → Customer SO)

- **Ringkasan**: 56 berkas berubah, ±15.210 baris ditambah / ±25 dihapus. Ini adalah keseluruhan modul Customer Management yang pada laporan sebelumnya masih berstatus *staged* dan kini resmi menjadi commit bersih di atas basis terintegrasi — bebas konflik dan siap di-review sebagai satu kesatuan fitur.
- **Konteks**: Modul ini adalah tulang punggung alur transaksi penjualan. Siklus dokumennya mengikuti pola bisnis yang lazim: sales menerbitkan **Quotation** (penawaran); bila pelanggan setuju terbit **Customer PO** (pesanan pembelian pelanggan); lalu **Customer SO** (sales order) sebagai konfirmasi order siap eksekusi. Ketiganya memiliki sisi **internal** (dikelola admin/sales) dan sisi **publik** (dibuka pelanggan lewat tautan ber-token untuk meninjau serta menyetujui/menandatangani tanpa login).
- **Komponen yang Berubah**:
  - **Backend — Quotation**: [customerQuotation.controller.js](backend/src/controllers/customerQuotation.controller.js) `[NEW]`, [customerQuotation.service.js](backend/src/services/customerQuotation.service.js) `[NEW]`, [customerQuotation.route.js](backend/src/routes/customerQuotation.route.js) `[NEW]`, [customerQuotation.model.js](backend/src/models/customerQuotation.model.js) `[NEW]`.
  - **Backend — Dokumen PO & SO (bersama)**: [customerDocument.controller.js](backend/src/controllers/customerDocument.controller.js) `[NEW]`, [customerDocument.service.js](backend/src/services/customerDocument.service.js) `[NEW]`, [customerDocument.route.js](backend/src/routes/customerDocument.route.js) `[NEW]`, model [customerPO.model.js](backend/src/models/customerPO.model.js) `[NEW]` & [customerSO.model.js](backend/src/models/customerSO.model.js) `[NEW]`.
  - **Backend — Endpoint publik**: [publicQuotation.controller.js](backend/src/controllers/publicQuotation.controller.js) `[NEW]`, [publicCustomerPO.controller.js](backend/src/controllers/publicCustomerPO.controller.js) `[NEW]`, [publicCustomerSO.controller.js](backend/src/controllers/publicCustomerSO.controller.js) `[NEW]`.
  - **Backend — Konfigurasi & pendukung**: [app.js](backend/src/app.js), [privilege.json](backend/src/config/privilege.json), [public.route.js](backend/src/routes/public.route.js), [files.controller.js](backend/src/controllers/files.controller.js), [files.route.js](backend/src/routes/files.route.js), [telegram.js](backend/src/utils/telegram.js) (notifikasi peristiwa dokumen), terjemahan backend [en](backend/src/locales/en/translation.json)/[id](backend/src/locales/id/translation.json).
  - **Frontend — Internal**: halaman & komponen Quotation ([create](frontend/src/app/pages/services/quotation/create.jsx), [edit](frontend/src/app/pages/services/quotation/edit.jsx), [quotationSchema](frontend/src/app/pages/services/quotation/quotationSchema.js), [QuotationDetailDrawer](frontend/src/app/pages/services/quotation/QuotationDetailDrawer.jsx), [QuotationDocumentPreview](frontend/src/app/pages/services/quotation/QuotationDocumentPreview.jsx), [QuotationPreviewModal](frontend/src/app/pages/services/quotation/QuotationPreviewModal.jsx), [EditQuotationDrawerCell](frontend/src/app/pages/services/quotation/EditQuotationDrawerCell.jsx)); Customer PO ([create](frontend/src/app/pages/services/customerPurchaseOrder/create.jsx), [edit](frontend/src/app/pages/services/customerPurchaseOrder/edit.jsx), [schema](frontend/src/app/pages/services/customerPurchaseOrder/schema/customerPOSchema.js), [CustomerPODocumentPreview](frontend/src/app/pages/services/customerPurchaseOrder/CustomerPODocumentPreview.jsx), [CustomerPOReviewDrawer](frontend/src/app/pages/services/customerPurchaseOrder/CustomerPOReviewDrawer.jsx)); Customer SO ([create](frontend/src/app/pages/services/customerSalesOrder/create.jsx), [edit](frontend/src/app/pages/services/customerSalesOrder/edit.jsx), [schema](frontend/src/app/pages/services/customerSalesOrder/schema/customerSOSchema.js), [CustomerSODocumentPreview](frontend/src/app/pages/services/customerSalesOrder/CustomerSODocumentPreview.jsx), [CustomerSOReviewDrawer](frontend/src/app/pages/services/customerSalesOrder/CustomerSOReviewDrawer.jsx)).
  - **Frontend — Publik**: [PublicQuotationDocument](frontend/src/app/pages/public/PublicQuotationDocument.jsx), [PublicCustomerPODocument](frontend/src/app/pages/public/PublicCustomerPODocument.jsx), [PublicCustomerSODocument](frontend/src/app/pages/public/PublicCustomerSODocument.jsx), [ReviewCustomerPOPage](frontend/src/app/pages/public/ReviewCustomerPOPage.jsx), [ReviewCustomerSOPage](frontend/src/app/pages/public/ReviewCustomerSOPage.jsx).
  - **Frontend — Bersama & konfigurasi**: [DocumentPreviewModal](frontend/src/components/shared/DocumentPreviewModal.jsx), [DocumentActionsMenu](frontend/src/components/shared/table/DocumentActionsMenu.jsx), [rows.jsx](frontend/src/components/shared/table/rows.jsx), konstanta [customer.constant](frontend/src/constants/customer.constant.js)/[quotation.constant](frontend/src/constants/quotation.constant.js), route [protected](frontend/src/app/router/protected.jsx)/[public](frontend/src/app/router/public.jsx), tab dokumen pada profil [pelanggan bisnis](frontend/src/app/pages/users/business/profile.jsx) & [mitra](frontend/src/app/pages/users/partner/profile.jsx), terjemahan [en](frontend/src/i18n/locales/en/translations.json)/[id](frontend/src/i18n/locales/id/translations.json).
- **Deskripsi Perubahan & Fungsi**: Menyediakan siklus penuh dokumen penjualan pelanggan (Quotation → PO → SO) dari sisi internal maupun publik, dengan komponen pratinjau dan menu aksi dokumen yang dibuat generik agar konsisten dipakai lintas ketiga jenis dokumen, serta notifikasi Telegram untuk peristiwa dokumen.

---

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Setelah commit di atas, dimulai fitur baru **Manajemen Status Pelanggan (Blacklist & Pasif)**. Seluruhnya **sudah ter-stage penuh dan bersih** — **23 berkas** (±1.225 baris ditambah / ±45 dihapus), **0 unstaged, 0 untracked** — menunggu commit berikutnya.

**Konteks**: Selain mengelola dokumen penjualan, sistem perlu menandai pelanggan yang **di-blacklist** (mis. bermasalah/penyalahgunaan, disertai alasan) dan pelanggan **pasif** (nonaktif/tidak berlangganan lagi), lalu menyajikannya sebagai daftar tersendiri agar admin mudah memantau dan menindaklanjuti. Status ini juga bersinggungan dengan layanan sesi (Radius) dan pengguna hotspot, sehingga beberapa service terkait ikut disesuaikan.

- [backend/src/models/customer.model.js](backend/src/models/customer.model.js)
  - **Deskripsi**: Menambah field status pelanggan: `blacklist`, `blacklist_reason`, dan `pasif`.
- [backend/src/controllers/customer.controller.js](backend/src/controllers/customer.controller.js)
  - **Deskripsi**: Handler baru `listBlacklistCustomer`, `listPasifCustomer`, `setBlacklistCustomer`, dan pengatur status pasif — untuk DataTable daftar serta aksi set/batal status.
- [backend/src/services/customer.service.js](backend/src/services/customer.service.js)
  - **Deskripsi**: `findBlacklistCustomerForTable`, `findPasifCustomerForTable`, `setCustomerBlacklist`, `setCustomerPasif` — kueri daftar terfilter & mutasi status.
- [backend/src/routes/customer.route.js](backend/src/routes/customer.route.js) · [backend/src/constants/customer.constant.js](backend/src/constants/customer.constant.js) `[NEW]`
  - **Deskripsi**: Route `/customer/blacklist-list`, `/customer/pasif-list`, dan endpoint set status; konstanta status pelanggan terpusat di backend.
- [backend/src/services/customerPartner.service.js](backend/src/services/customerPartner.service.js) · [backend/src/services/hotspotUser.service.js](backend/src/services/hotspotUser.service.js) · [backend/src/services/radiusSession.service.js](backend/src/services/radiusSession.service.js)
  - **Deskripsi**: Penyesuaian layanan terkait agar status blacklist/pasif konsisten dengan sesi Radius & pengguna hotspot.
- [frontend/src/app/pages/users/blacklist/index.jsx](frontend/src/app/pages/users/blacklist/index.jsx) `[NEW]` · [schema/columns.jsx](frontend/src/app/pages/users/blacklist/schema/columns.jsx) `[NEW]`
  - **Deskripsi**: Halaman daftar pelanggan blacklist (Datatables, `apiUrl=/customer/blacklist-list`) beserta definisi kolomnya.
- [frontend/src/app/pages/users/pasif/index.jsx](frontend/src/app/pages/users/pasif/index.jsx) `[NEW]` · [schema/columns.jsx](frontend/src/app/pages/users/pasif/schema/columns.jsx) `[NEW]`
  - **Deskripsi**: Halaman daftar pelanggan pasif (Datatables, `apiUrl=/customer/pasif-list`) beserta kolomnya.
- [frontend/src/app/router/users/blacklistRoute.jsx](frontend/src/app/router/users/blacklistRoute.jsx) `[NEW]` · [frontend/src/app/router/users/pasifRoute.jsx](frontend/src/app/router/users/pasifRoute.jsx) `[NEW]` · [frontend/src/app/router/protected.jsx](frontend/src/app/router/protected.jsx) · [frontend/src/app/navigation/users.js](frontend/src/app/navigation/users.js)
  - **Deskripsi**: Route terproteksi & entri menu navigasi untuk halaman Blacklist dan Pasif.
- [frontend/src/app/pages/users/customer/edit.jsx](frontend/src/app/pages/users/customer/edit.jsx) · [frontend/src/app/pages/users/customer/profile.jsx](frontend/src/app/pages/users/customer/profile.jsx)
  - **Deskripsi**: Integrasi aksi set/batal blacklist & pasif dan penampilan statusnya pada halaman edit/profil pelanggan.
- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) · [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json) · [backend/src/locales/*/translation.json](backend/src/locales/id/translation.json) · [.gitignore](.gitignore)
  - **Deskripsi**: String terjemahan untuk menu & label status baru; penyesuaian `.gitignore`.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - (Commit) Sales/admin dapat mengelola siklus penuh dokumen penjualan (Quotation → PO → SO) dari sisi internal, sementara pelanggan meninjau & menyetujui/menandatangani via tautan publik tanpa login; tab dokumen tersedia pada profil pelanggan bisnis & mitra.
  - (WIP) Admin dapat menandai pelanggan sebagai **blacklist** (dengan alasan) atau **pasif**, serta memantau keduanya melalui halaman daftar tersendiri.
- **Bug Fix / Solusi Masalah**: Penataan ulang modul Customer Management menjadi commit bersih di atas basis terintegrasi menghilangkan risiko konflik yang melekat pada WIP monolitik sebelumnya. Penyesuaian layanan Radius/hotspot menjaga konsistensi status pelanggan dengan sesi aktif.
- **Menu/Tombol Baru**: Menu navigasi & route Quotation/Customer PO/Customer SO; halaman review publik PO & SO; serta (WIP) menu **Blacklist** dan **Pasif** pada bagian Users, plus aksi set/batal status pada halaman pelanggan.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Customer Management menghubungkan tiga dokumen penjualan dalam alur berurutan (Quotation → PO → SO), masing-masing dapat dibuat/disunting/dipratinjau dari sisi internal dan ditinjau/ditandatangani dari sisi publik oleh pelanggan lewat tautan ber-token. Fitur status pelanggan (Blacklist & Pasif) menambahkan penandaan status beserta daftar terfilter untuk memudahkan pemantauan.
- **Langkah Penggunaan (Tutorial)**:
  1. **Terbitkan Quotation** → pelanggan meninjau lewat tautan publik → **lanjut ke Customer PO** (nomor PO otomatis) → **terbitkan Customer SO** (bisa ditandatangani pada posisi terpilih) → telusuri semuanya dari tab dokumen di profil pelanggan.
  2. **Kelola status pelanggan** (WIP): buka profil/edit pelanggan → set **Blacklist** (isi alasan) atau **Pasif**; pantau daftarnya lewat menu **Users → Blacklist** dan **Users → Pasif**.
