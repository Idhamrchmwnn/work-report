# 📝 Daily Work Report - Idham (2026-11-27)

---

## 📅 Laporan Harian - 27 November 2026

---

## 🌿 Branch: `issue-593` — Pelaporan per Faktur Pelanggan untuk Partner API

### 📌 Informasi Issue

- **Nomor Issue**: #593
- **Judul Issue**: Implementasi Pelaporan per Invoice Pelanggan untuk P-API
- **Tujuan Issue**:
  - Admin bisa memilih faktur mana dari pelanggan/pelanggan bisnis internal yang dikirim ke Aplikasi Pelaporan. Sebelumnya yang bisa dipilih hanya layanannya.
  - Pelanggan yang mulai dilaporkan di tengah jalan tidak membawa faktur bulan-bulan sebelumnya.
- **Status Branch**: `Masih berjalan` (belum `resolve`).
  - Satu commit `save #593` (`fbaa5293`) di atas `master` terbaru (`resolve #588` hasil merge Dedy), sudah ada di `origin/issue-593`.

### 📅 Rincian Commit

#### [fbaa5293] - save #593 - 27 November 2026, 17:04

- **Ringkasan**:
  - Di atas pelaporan per layanan (`tax_reported`) kini ada dua lapis tambahan, **hanya untuk faktur pemilik internal**:
    1. **Coret per faktur** (`tax_report_excluded`): faktur tertentu tidak dikirim ke Partner API. Bawaannya semua faktur dilaporkan, jadi faktur lama tidak perlu diubah.
    2. **"Laporkan faktur mulai bulan"** (`tax_report_from`): dipasang di layanan (akun radius/dedicated) untuk faktur ber-layanan, atau di pelanggan/bisnis untuk faktur tanpa layanan. Faktur yang tanggal terbitnya sebelum bulan itu tidak dikirim.
  - Aturan bulan mulai **dihitung saat dibaca**, bukan coretan sekali jalan. Karena itu faktur bertanggal mundur yang dibuat belakangan (dari web, Partner API, maupun V1) ikut tertahan.
  - Faktur pelanggan mitra dan faktur mitra tidak terpengaruh; keduanya tetap mengikuti `show_in_portal`.
  - **Sebelum → Sesudah**:

    | Area | Sebelum | Sesudah |
    |---|---|---|
    | Satuan pelaporan | Per layanan: semua faktur layanan yang dilaporkan ikut terkirim, termasuk faktur bertahun-tahun sebelumnya. | Per layanan **dan** per faktur: faktur bisa dicoret satu per satu, dan bisa dibatasi mulai bulan tertentu. |
    | Mulai melaporkan layanan | Mencentang layanan langsung mengirim seluruh riwayat fakturnya. | Saat mencentang layanan baru, admin memilih "semua faktur" atau "mulai bulan X". Aturan disimpan dalam update yang sama dengan `tax_reported: true`. |
    | Tabel & detail faktur (web) | Tidak ada keterangan apakah faktur dikirim ke Aplikasi Pelaporan. | Kolom **Pelaporan** (Dilaporkan / Dicoret / Tidak dilaporkan) yang bisa difilter dan diurutkan, badge di drawer detail, serta aksi massal coret/batal coret. |
    | Admin Partner API membuat faktur bertanggal sebelum bulan mulai | Faktur tersimpan tetapi langsung 404 bagi pembuatnya sendiri. | Ditolak 422 `beforeTaxReportStart`, setelah cek `client_ref`. |
    | Hak akses | `financeInvoice.changeSensitive` hanya untuk mengubah tanggal terbit. | Juga dipakai untuk memilih faktur yang dilaporkan (coret + bulan mulai). Tanpa hak itu, simpan pelaporan yang membawa bulan mulai ditolak 403 sebelum apa pun ditulis. |

- **Komponen yang Berubah & Penjelasannya**:

  - **Backend — Model**
    - `backend/src/models/financeInvoice.model.js`
      - Field `tax_report_excluded` (tanpa default) beserta jejak `tax_report_excluded_by`/`tax_report_excluded_at`.
      - Mengubah coretan tidak menulis entri `finance_invoice_updates`.
    - `backend/src/models/radiusAuthentication.model.js`, `productDataAccess.model.js`, `customer.model.js`, `partner.model.js`
      - Field `tax_report_from` (`Date` awal bulan WIB).
  - **Backend — Service & Utils**
    - `backend/src/services/taxReport.service.js` (bagian terbesar)
      - `buildInternalCustomerInvoiceFilter` dan `buildInternalBusinessInvoiceFilter` kini juga menyaring:
        - faktur yang dicoret;
        - faktur sebelum bulan mulai (klausa `$nor` dikelompokkan per bulan).
      - `buildInvoiceTaxReportClauses` adalah satu sumber klausa untuk filter Partner API, kolom Pelaporan di tabel web (`applyInvoiceTaxReportTableParams`, `attachInvoiceTaxReportStatus`), dan badge detail (`findInvoiceTaxReportStatus`). Ketiganya memakai sumber yang sama supaya tidak pernah berbeda, dan sengaja tanpa cache.
      - Penulisan aturan bulan mulai saat layanan baru dilaporkan:
        - Hanya layanan yang BARU dicentang; aturan pemilik hanya bila pemiliknya baru tampil.
        - `null` = hapus aturan; tidak dikirim = tidak disentuh.
        - Aturan dedicated disalin ke akun PPPoE tertautnya (`syncLinkedReportFrom`).
    - `backend/src/utils/taxReportMonth.js` (baru)
      - Konversi bulan WIB ↔ `YYYY-MM` dan awal bulan sebagai `Date`.
      - Batas maksimal 12 bulan ke depan (`taxReportFromTooFar`).
    - `backend/src/services/financeInvoice.service.js`
      - Daftar faktur web memakai kolom turunan `tax_report_status`.
      - `catch` yang sebelumnya diam kini mencatat `logger.error`.
    - `backend/src/utils/validation-data.js`
      - `cleanFormData` membuang keempat field pelaporan, sehingga hanya bisa ditulis lewat endpoint khusus.
  - **Backend — Controller & Route**
    - `backend/src/controllers/invoiceTaxReport.controller.js` (baru) + `backend/src/routes/financeInvoice.route.js`, semua `financeInvoice.changeSensitive`:
      - `GET` / `PATCH /finance/invoice/tax-report/:owner_type/:owner_id`
        - Baca dan ubah per pemilik.
        - Body PATCH berupa selisih `{ exclude, include }`, bukan daftar penuh + versi. Alasannya, tagihan otomatis yang terus bertambah akan memicu 409 terus-menerus.
      - `PATCH .../from`: ubah bulan mulai per layanan/pemilik.
      - `PATCH /finance/invoice/tax-report/batch`: aksi massal dari tabel faktur. Faktur non-internal dilaporkan sebagai `skipped`.
      - Pemilik non-internal dijawab 422 `taxReportInternalOnly`.
    - `backend/src/controllers/customer.controller.js`, `business.controller.js` + `customer.route.js`, `business.route.js`
      - `PATCH /customer|business/tax-report(+batch)` menerima `invoice_report_from`.
      - Hak `financeInvoice.changeSensitive` diperiksa lebih dulu lewat `resolveServiceReportFrom`.
    - `backend/src/controllers/financeInvoice.controller.js`
      - Detail faktur membawa `tax_report_status`.
    - `backend/src/controllers/partnerApiInvoice.controller.js`
      - Pagar 422 `beforeTaxReportStart` saat admin Partner API membuat/mengubah faktur internal sebelum bulan mulai.
  - **Backend — Konfigurasi & i18n**
    - `backend/src/config/privilege.json`, `privilegeDictionary.json`
      - Deskripsi `financeInvoice.changeSensitive` diperluas dan `apiEndpoints` diperbarui.
    - `backend/src/utils/sync-privilege-dictionary.js`
      - `expectedEndpoints` kini menggabungkan endpoint yang dicatat manual (pemeriksaan di controller) dengan hasil pemindai route. Sebelumnya daftar manual menimpa hasil pemindai.
    - `backend/src/locales/{id,en}/translation.json`
      - 8 key baru, antara lain `taxReportUpdated`, `taxReportInternalOnly`, `taxReportFromTooFar`, dan `beforeTaxReportStart`.
  - **Frontend**
    - `frontend/src/app/pages/users/components/InvoiceTaxReportDrawer.jsx` (baru)
      - Drawer "Atur faktur" per pelanggan/bisnis: daftar faktur dengan centang coret/laporkan dan pengaturan bulan mulai per layanan/pemilik.
    - `frontend/src/app/pages/users/components/invoiceTaxReport.js` (baru)
      - Logika murni drawer: menyusun selisih `{ exclude, include }` dan opsi bulan.
    - `frontend/src/app/pages/users/components/TaxReportServicesCard.jsx`
      - Saat layanan baru dicentang, admin memilih "semua faktur" atau "mulai bulan X".
      - Tombol "Atur faktur" membuka drawer.
      - Keduanya hanya untuk pemegang `financeInvoice.changeSensitive`.
    - `frontend/src/app/pages/finance/invoices/index.jsx`, `schema/columns.jsx`, `detail.jsx`, `InvoiceTaxReportStatus.jsx` (baru)
      - Kolom Pelaporan, aksi massal coret/batal coret, dan badge di drawer detail.
    - `frontend/src/components/shared/table/rows.jsx`, `status.js`
      - Wrapper cell `InvoiceTaxReportStatusCell` dan peta status/warna badge.
    - `frontend/src/constants/privilegeDescriptions.{id,en}.json`, `frontend/src/i18n/locales/{id,en}/translations.json`
      - Deskripsi hak akses dan teks UI baru.
  - **Dokumentasi**
    - `AGENTS.md`
      - Butir §9 baru: "Pelaporan per FAKTUR ke Partner API".
  - **Test**
    - `backend/test/integration/invoiceTaxReport.test.js` (baru), 17 test:
      - coret/batal coret;
      - bulan mulai per layanan dan pemilik;
      - faktur bertanggal mundur yang dibuat belakangan tetap tertahan;
      - pemilik non-internal 422 dan tanpa hak 403;
      - kolom tabel dan badge sama dengan filter Partner API;
      - aturan dedicated tersalin ke akun PPPoE;
      - pagar `beforeTaxReportStart` di Partner API.
    - `frontend/src/app/pages/users/components/invoiceTaxReport.test.js` (baru), 4 test untuk logika drawer.
    - `frontend/src/components/shared/table/TextAutocompleteFilter.test.jsx`
      - Mock disesuaikan dengan ekspor baru `rows.jsx`.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #593 | `fbaa5293` save #593 | Pelaporan per faktur ke Partner API: coret faktur satu per satu dan aturan "laporkan mulai bulan" per layanan/pemilik (dihitung saat dibaca); kolom Pelaporan + aksi massal di tabel faktur web, drawer "Atur faktur", pilihan bulan mulai saat mencentang layanan; pagar 422 di Partner API; +21 test |

> Catatan merge: `resolve #588` (dikerjakan 25 November) sudah di-merge ke `master` oleh Dedy S.N Putra pada 25 November pukul 19:36, bersama `resolve #589` milik Dedy. Merge #588 memuat penyesuaian dari Dedy.
>
> Catatan: tidak ada commit pada 26 November, jadi tidak ada laporan untuk tanggal itu.
>
> Catatan branch: `issue-593` masih berupa `save` dan berada di atas `master` terbaru. Perlu di-squash menjadi `resolve #593` setelah selesai.
