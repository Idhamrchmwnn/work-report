# 📝 Daily Work Report - Idham (2026-10-12)

---

## 📅 Laporan Harian - 12 Oktober 2026

---

## 🌿 Branch: `issue-394` — Lapor Pajak per Layanan & Visibilitas Pelanggan di Partner API (p-api)

### 📌 Informasi Issue

- **Nomor Issue**: #394
- **Judul Issue**: Visibilitas pelanggan & pelanggan bisnis di aplikasi eksternal (Partner API) berdasarkan status lapor pajak
- **Status Branch**: `Belum di-merge` (commit `resolve #394` di-amend dan di-push ke `origin/issue-394`)

### 📅 Rincian Commit

#### [bb88916e] - resolve #394 (amend) - 12 Oktober 2026, 22:29

- **Ringkasan**: Flag `tax_reported` (siap dilaporkan pajak) dipindah dari tingkat **pelanggan/bisnis** ke tingkat **layanan**:
  - Layanan yang dimaksud adalah akun broadband (radius) serta layanan dedicated internet dan akses data.
  - Admin memilih layanan mana yang dilaporkan lewat kartu "Lapor Pajak" di form edit, atau lewat aksi massal di tabel.
  - Pelanggan/bisnis internal baru tampil di aplikasi eksternal (p-api, sesi admin) bila punya **minimal satu layanan dilaporkan**. Hanya layanan yang dilaporkan yang ikut terlihat, begitu juga fakturnya.
  - Semua aturan visibilitas dipusatkan di satu service baru (`taxReport.service.js`).
  - Ditutup juga beberapa celah akses p-api yang ditemukan saat audit.
  - Pelanggan/bisnis milik mitra tetap mengikuti `show_in_portal` mitranya.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Model**

  - `backend/src/models/radiusAuthentication.model.js`
    - Field baru `tax_reported` (bawaan `false`, ber-index) pada akun radius, serta jejak audit `tax_reported_by` (admin) dan `tax_reported_at` (waktu).
    - Index baru pada `customer`, karena ringkasan lapor pajak dan filter p-api mencari akun per pelanggan. Tanpa index ini setiap pencarian memindai seluruh koleksi.
  - `backend/src/models/productDataAccess.model.js`
    - Field `tax_reported`, `tax_reported_by`, dan `tax_reported_at` yang sama untuk layanan dedicated internet / akses data.
  - `backend/src/models/customer.model.js`, `backend/src/models/partner.model.js`
    - Field `tax_reported` di tingkat pelanggan dan bisnis dihapus, karena flag kini hidup di layanan.

  **Backend — Service**

  - `backend/src/services/taxReport.service.js` [NEW]
    - Satu-satunya tempat aturan lapor pajak dan visibilitas p-api. Controller p-api tidak lagi menyusun filter sendiri.
    - **Siapa yang tampil**: `getReportedInternalCustomerIds` dan `getReportedInternalBusinessIds` mencari pelanggan/bisnis internal yang punya minimal satu layanan dilaporkan. Hasilnya dihitung saat dibaca dan tidak disimpan, supaya tidak ada data turunan yang bisa tidak sinkron. Layanan nonaktif/terhapus tetap dihitung (memakai `*WithDeleted`).
    - **Filter p-api**: `buildCustomerVisibleFilter`, `buildBusinessVisibleFilter`, `buildAuthenticationVisibleFilter`, `isAuthenticationVisible`, `getVisibleAuthenticationIdScope`, dan `getHiddenBusinessIds`. Semua memakai `$nor`/`$and` agar tidak bentrok dengan filter pemanggil.
    - Agar query tidak membawa ribuan `_id`, daftar ID dipilih dari sisi yang lebih kecil (`buildNonMemberClause`): kecualikan yang tersembunyi, atau batasi ke yang tampil.
    - **Faktur**: `buildInternalCustomerInvoiceFilter`. Faktur per layanan (`ref_auth`) mengikuti layanannya, sedangkan faktur lama tanpa kaitan layanan mengikuti pelanggannya.
    - **Penautan**: `canLinkInternalCustomer`. Admin p-api hanya boleh menautkan akun ke pelanggan internal yang sudah tampil atau belum punya layanan.
    - **PPPoE dedicated**: `getDedicatedLinkedAuthIds`. Akun PPPoE yang tertaut ke layanan dedicated bukan layanan tersendiri, jadi flag-nya mengikuti dedicated-nya.
    - **Form edit**: `findCustomerTaxReportServices`, `findBusinessTaxReportServices`, `setCustomerTaxReportServices`, dan `setBusinessTaxReportServices`. Form membaca semua layanan termasuk yang terhapus, lalu menyimpan pilihan dengan sidik jari `version` (409 bila data sudah diubah admin lain). Jejak audit hanya ditulis pada layanan yang benar-benar berubah.
    - **Aksi massal**: `setCustomerTaxReportBatch` dan `setBusinessTaxReportBatch` menandai atau membatalkan semua layanan untuk banyak pelanggan/bisnis sekaligus. Pelanggan/bisnis milik mitra diabaikan.
    - **Ringkasan tabel**: `getCustomerTaxReportSummary` dan `getBusinessTaxReportSummary` menghasilkan status `all` / `partial` / `none` / `no_service`, dihitung hanya untuk baris di halaman tabel yang sedang dibuka.
    - **Layanan bisnis untuk p-api**: `findBusinessServicesForPartnerApi` dan `findBusinessServiceForPartnerApi`, dengan daftar-putih field. Data teknis (IP, VLAN, SNMP, lastmile) tidak pernah dikirim.
  - `backend/src/services/customer.service.js`
    - Datatable Pelanggan kini menambahkan `tax_report_status` per baris dari ringkasan layanan.
    - Filter lama di tingkat pelanggan (`CUSTOMER_PORTAL_VISIBLE_FILTER`, `buildPortalVisibleAuthenticationFilter`) dihapus karena diganti `taxReport.service`.
    - `catch` yang sebelumnya diam kini mencatat `logger.error`.
  - `backend/src/services/business.service.js`
    - Datatable Bisnis menambahkan `tax_report_status`. Bisnis milik reseller diberi `not_applicable`.
    - Filter lama `BUSINESS_PORTAL_VISIBLE_FILTER` dan `getHiddenPortalBusinessIds` dihapus.
    - `catch` yang sebelumnya diam kini mencatat `logger.error`.
  - `backend/src/services/financeInvoice.service.js`
    - Fungsi baru `findOneInternalCustomerInvoiceForPartnerApi`: detail satu faktur pelanggan internal untuk p-api, disaring dengan filter visibilitas dari `taxReport.service`.
  - `backend/src/services/radiusAuthentication.service.js`, `backend/src/services/productDataAccess.service.js`, `backend/src/services/productDedicatedInternet.service.js`
    - `tax_reported` ditambahkan ke field datatable layanan.
    - Untuk dedicated/akses data, field `partner` milik pemilik ikut di-populate supaya tabel bisa mengenali layanan milik mitra yang tidak memakai lapor pajak.
  - `backend/src/utils/validation-data.js`
    - Helper `normalizeBooleanField` dihapus karena flag tidak lagi dikirim lewat form edit pelanggan (FormData).

  **Backend — Controller & Route (aplikasi utama)**

  - `backend/src/controllers/customer.controller.js`
    - Handler baru:
      - `getCustomerTaxReport`: daftar layanan beserta pilihannya.
      - `updateCustomerTaxReport`: menyimpan pilihan per layanan dengan pengecekan `version`.
      - `batchCustomerTaxReport`: aksi massal.
    - Ditambah helper validasi `parseServiceIds` dan `parseOwnerIds` (400 untuk body tidak sah).
  - `backend/src/controllers/business.controller.js`
    - Handler yang sama untuk bisnis: `getBusinessTaxReport`, `updateBusinessTaxReport`, `batchBusinessTaxReport`.
    - Bisnis milik reseller dijawab 422 (`taxReportOwnedByReseller`).
  - `backend/src/routes/customer.route.js`
    - Route baru dengan privilege `customer.changeSensitive` dan Swagger lengkap: `GET /customer/tax-report/:customer_id`, `PATCH /customer/tax-report`, `PATCH /customer/tax-report/batch`.
  - `backend/src/routes/business.route.js`
    - Route baru dengan privilege `business.changeSensitive`: `GET /business/tax-report/:partner_id`, `PATCH /business/tax-report`, `PATCH /business/tax-report/batch`.
  - `backend/src/routes/partner.route.js`
    - Perbaikan teks Swagger endpoint toggle portal mitra.

  **Backend — Partner API (p-api)**

  - `backend/src/controllers/partnerApiCustomer.controller.js`
    - Visibilitas pelanggan kini memakai `buildCustomerVisibleFilter`: tampil bila punya minimal satu layanan dilaporkan.
    - Pelanggan baru buatan admin p-api tidak lagi diberi flag. Pelanggan itu tampil setelah akun broadband dibuat untuknya, karena akun buatan admin p-api otomatis dilaporkan.
  - `backend/src/controllers/partnerApiBusiness.controller.js`
    - Visibilitas bisnis memakai `buildBusinessVisibleFilter`.
    - Endpoint baru `listPartnerAppBusinessServices` dan `readPartnerAppBusinessService` (helper `resolveAccessibleBusiness`):
      - Admin pada bisnis internal hanya melihat layanan yang dilaporkan.
      - Sesi mitra melihat semua layanan bisnis miliknya.
  - `backend/src/controllers/partnerApiRadius.controller.js`
    - Akun internal hanya tampil bila layanannya dilaporkan (`buildAuthenticationVisibleFilter`). Aturan ini berlaku juga untuk detail, ubah, status, log, sesi, dan putus sesi (404).
    - Helper baru `canAccessAuthentication` memakai filter visibilitas murni, bukan `partner_id` dari body. Sebelumnya isi body bisa menggeser hasil pengecekan.
    - Akun buatan admin p-api untuk pelanggan internal otomatis `tax_reported: true`. Akun untuk pelanggan mitra tidak diberi flag. Nilai `tax_reported` dari body selalu dibuang.
    - **Celah ditutup**:
      - Mitra tidak lagi bisa menautkan akunnya ke pelanggan internal (membocorkan nama/ID pelanggan internal).
      - Admin p-api tidak bisa membuka pelanggan tersembunyi lewat `customer_id` tebakan saat membuat atau memindah akun. Jawabannya 404 yang sama dengan "tidak ditemukan".
      - Akun staf (tanpa pelanggan dan tanpa mitra) tidak pernah tampil.
      - `show_in_portal` pada dokumen bisnis tidak bisa membuka akun-akunnya; yang berlaku hanya `show_in_portal` reseller.
  - `backend/src/controllers/partnerApiInvoice.controller.js`
    - Helper baru `resolveInternalInvoiceFilter`, `isInternalCustomerScope`, dan `emptyDataTable`.
    - Faktur pelanggan internal mengikuti layanan atau pelanggannya. Faktur bisnis mengikuti bisnisnya, termasuk bisnis milik reseller tersembunyi (fakturnya merujuk `_id` bisnis, bukan reseller).
  - `backend/src/controllers/partnerApiNetworkDevice.controller.js`
    - Pencarian mitra dibatasi `reseller: true`, supaya `partner_id` milik dokumen bisnis tidak bisa dipakai.
  - `backend/src/routes/partnerApi.route.js`
    - Route baru `GET /p-api/v1/business/services/list` dan `GET /p-api/v1/business/services/read/:id`.
    - Swagger seluruh endpoint pelanggan, bisnis, radius, dan faktur diperbarui sesuai aturan visibilitas per layanan.

  **Backend — Privilege & i18n**

  - `backend/src/config/privilege.json`
    - Key `customer.changeSensitive` dan `business.changeSensitive` aktif lagi, karena kini dipakai route lapor pajak.
  - `backend/src/config/privilegeDictionary.json`
    - Entri kamus kedua key di atas dengan deskripsi baru (memilih layanan yang dilaporkan pajak), daftar endpoint `/tax-report*`, dependensi `.read` + `.list`, dan risiko `critical`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `taxReportConflict` (409), `taxReportUpdated`, `taxReportBatchUpdated` (pelanggan & bisnis), dan `taxReportOwnedByReseller`.

  **Backend — Test**

  - `backend/test/integration/customerTaxReported.test.js`
    - Ditulis ulang untuk model per layanan. Isinya:
      - GET/PATCH/aksi massal, termasuk 400 untuk body tidak sah dan pelanggan mitra yang diabaikan.
      - Ringkasan `all`/`partial`/`none`/`no_service`.
      - Pelanggan dengan 1 dari 2 layanan dilaporkan hanya memperlihatkan layanan itu.
      - Akun yang tidak dilaporkan dijawab 404 di semua jalur.
      - Sesi radius dan faktur internal.
      - Layanan terhapus tetap dihitung.
      - Kelompok "Hasil audit: celah yang ditutup": jejak audit, versi basi 409, penautan pelanggan tersembunyi, akun staf, penautan oleh mitra, dan index `customer`.
  - `backend/test/integration/businessTaxReported.test.js`
    - Ditulis ulang. Isinya:
      - GET untuk ketiga jenis layanan; bisnis milik reseller dijawab 422.
      - PATCH lintas jenis layanan dan aksi massal.
      - Ringkasan `not_applicable`.
      - `/business/services` tanpa data teknis.
      - Sesi mitra.
      - Faktur bisnis.
      - Celah yang ditutup: PPPoE dedicated, `?partner_id=` bisnis, dan faktur bisnis milik reseller tersembunyi.
  - `backend/test/helpers/factories.js`
    - Factory baru `createReportedCustomer`: pelanggan reguler dengan satu akun broadband yang sudah dilaporkan, yaitu syarat minimal agar tampil di p-api.
  - `backend/test/integration/partnerApiCustomer.{changeStatus,delete,documents,list,listStatus,read,update}.test.js`, `backend/test/integration/partnerApiInvoice.test.js`
    - Data uji pelanggan reguler kini memakai `createReportedCustomer` (sebelumnya memakai flag di tingkat pelanggan).

  **Frontend — Komponen Baru**

  - `frontend/src/app/pages/users/components/TaxReportServicesCard.jsx` [NEW]
    - Kartu "Lapor Pajak" di form edit pelanggan/bisnis: menampilkan semua layanan (broadband, dedicated internet, akses data) dengan centang per layanan, serta tombol "Laporkan semua layanan" / "Batalkan laporan".
    - Hanya tampil untuk pemegang `customer.changeSensitive` / `business.changeSensitive`.
    - Mengirim `version`. Bila backend menjawab 409, data dimuat ulang supaya admin melihat keadaan terbaru.
  - `frontend/src/app/pages/users/components/TaxReportBatchModal.jsx` [NEW]
    - Modal aksi massal untuk tabel (`multiEditComponent`) yang memanggil `PATCH /{type}/tax-report/batch`, dengan varian `CustomerTaxReportBatchModal` dan `BusinessTaxReportBatchModal`.

  **Frontend — Halaman Pelanggan & Bisnis**

  - `frontend/src/app/pages/users/customer/edit.jsx`, `frontend/src/app/pages/users/business/edit.jsx`
    - Isian "Lapor Pajak" lama (pilihan ya/tidak dengan badge "Saat ini") diganti `TaxReportServicesCard`.
  - `frontend/src/app/pages/users/customer/editBatch.jsx`, `frontend/src/app/pages/users/business/editBatch.jsx`
    - Isian lapor pajak dihapus dari edit massal karena sudah digantikan modal aksi massal khusus.
  - `frontend/src/app/pages/users/customer/index.jsx`, `frontend/src/app/pages/users/business/index.jsx`
    - Menambah aksi massal lapor pajak di tabel, hanya untuk pemegang privilege `changeSensitive`.
  - `frontend/src/app/pages/users/customer/schema/columns.jsx`, `frontend/src/app/pages/users/business/schema/columns.jsx`
    - Kolom "Lapor Pajak" kini menampilkan ringkasan `tax_report_status` (Semua dilaporkan / Sebagian / Belum / Tanpa layanan) lewat `TaxReportStatusCell`. Kolom ini tanpa filter karena nilainya turunan dari layanan.
  - `frontend/src/app/pages/users/customer/profile.jsx`, `frontend/src/app/pages/users/business/profile.jsx`
    - Badge lapor pajak di tingkat pelanggan/bisnis dihapus.

  **Frontend — Halaman Layanan**

  - `frontend/src/app/pages/services/broadband/schema/columns.jsx`
    - Kolom baru "Lapor Pajak" (badge + filter) yang hanya terisi untuk akun milik pelanggan reguler.
  - `frontend/src/app/pages/services/dedicatedInternet/schema/columns.jsx`, `frontend/src/app/pages/services/dataAccess/schema/columns.jsx`
    - Kolom baru "Lapor Pajak" yang hanya terisi untuk layanan milik bisnis tanpa mitra pemilik.

  **Frontend — Komponen Bersama, Privilege & i18n**

  - `frontend/src/components/shared/table/rows.jsx`
    - Sel baru `TaxReportStatusCell` untuk ringkasan per pelanggan/bisnis (`not_applicable` tidak dirender). `TaxReportedCell` kini dipakai untuk flag per layanan.
  - `frontend/src/components/shared/table/status.js`
    - Opsi baru `taxReportStatusOptions` (warna dan ikon per status ringkasan). Label isian lama `taxReportedLabels` dihapus.
  - `frontend/src/components/shared/Badge.jsx`
    - `TaxReportedBadge` dihapus karena sudah tidak dipakai.
  - `frontend/src/components/shared/form/FormInput.jsx`
    - Prop `labels` pada `InputStatusSelect` dihapus, mengembalikannya ke bentuk semula karena hanya dipakai isian lapor pajak lama.
  - `frontend/src/constants/privilegeDescriptions.id.json`, `frontend/src/constants/privilegeDescriptions.en.json`
    - Tooltip `customer.changeSensitive` dan `business.changeSensitive`.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Key baru: `taxReportAll`, `taxReportNone`, `taxReportStatus.*` (all/partial/none/no_service), dan `taxReportServiceType.*` (broadband/internet/data_access). Key lama `current` dihapus.

  **Dokumentasi**

  - `AGENTS.md`
    - Pola Bab 9 "Visibilitas di Partner API" ditulis ulang untuk model per layanan. Isinya: perbedaan `show_in_portal` (mitra) dan `tax_reported` (layanan), aturan tampil "minimal satu layanan dilaporkan", aturan faktur, serta sepuluh aturan teknis (filter `$nor`/`$and`, daftar ID sisi terkecil, buang flag dari body p-api, bisnis internal = `partner: null`, `canLinkInternalCustomer`, akun staf, PPPoE dedicated, jejak audit + `version`, dll).

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #394 | Lapor Pajak per Layanan & Visibilitas p-api | Admin memilih layanan mana yang dilaporkan pajak. Aplikasi eksternal hanya melihat pelanggan/bisnis internal yang punya layanan dilaporkan, beserta layanan dan fakturnya. Beberapa celah akses p-api ditutup. |

### Kemampuan Baru Pengguna/Admin

- Memilih per layanan (broadband, dedicated internet, akses data) mana yang siap dilaporkan pajak, langsung dari form edit Pelanggan/Bisnis.
- Menandai atau membatalkan lapor pajak untuk banyak pelanggan/bisnis sekaligus lewat aksi massal di tabel.
- Melihat ringkasan status lapor pajak tiap pelanggan/bisnis (Semua / Sebagian / Belum / Tanpa layanan), serta status per layanan di tabel Broadband, Dedicated Internet, dan Akses Data.
- Melacak siapa dan kapan sebuah layanan terakhir diubah status lapor pajaknya.
- Aplikasi eksternal (p-api) dapat membaca daftar dan detail layanan pelanggan bisnis (`/business/services/*`).

### Bug Fix / Solusi Masalah

- Mitra tidak bisa lagi menautkan akun radiusnya ke pelanggan internal.
- Admin p-api tidak bisa lagi "membuka" pelanggan tersembunyi lewat `customer_id` tebakan saat membuat atau memindah akun.
- Akun staf (tanpa pelanggan dan mitra) tidak lagi bisa diakses lewat p-api.
- Pengecekan akses satu akun radius tidak lagi bisa digeser lewat `partner_id` di body.
- `partner_id` milik dokumen bisnis tidak lagi bisa dipakai di endpoint mitra (radius, perangkat jaringan).
- Dua admin yang menyimpan pilihan lapor pajak bersamaan tidak saling menimpa (409 + muat ulang).

### Menu/Fitur Baru

- Kartu **Lapor Pajak** di form edit **Pengguna > Pelanggan** dan **Pengguna > Bisnis**.
- Aksi massal **Lapor Pajak** di tabel Pelanggan dan Bisnis.
- Kolom **Lapor Pajak** di tabel **Layanan > Broadband / Dedicated Internet / Akses Data**.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Status "lapor pajak" kini melekat pada tiap layanan, bukan pada pelanggan. Aplikasi eksternal hanya menerima layanan yang sudah ditandai, dan pelanggan/bisnis internal baru terlihat di sana bila punya minimal satu layanan yang ditandai. Layanan yang sudah nonaktif atau terhapus tetap dihitung selama tandanya masih aktif. Pelanggan/bisnis milik mitra tidak memakai fitur ini; visibilitasnya mengikuti pengaturan portal mitra pemiliknya.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Pengguna > Pelanggan** (atau **Pengguna > Bisnis**), lalu buka form edit salah satu pelanggan.
  2. Pada kartu **Lapor Pajak**, centang layanan yang siap dilaporkan, atau klik **Laporkan semua layanan**, lalu simpan.
  3. Untuk banyak pelanggan sekaligus, centang barisnya di tabel lalu pilih aksi massal **Lapor Pajak**.
  4. Kolom **Lapor Pajak** di tabel menunjukkan ringkasannya: Semua dilaporkan, Sebagian, Belum, atau Tanpa layanan.
