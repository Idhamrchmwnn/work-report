# 📝 Daily Work Report - Idham (2026-11-25)

---

## 📅 Laporan Harian - 25 November 2026

---

## 🌿 Branch: `issue-588` — Partner API: Tulis Faktur Pelanggan & Mitra/Bisnis untuk Aplikasi Pelaporan

### 📌 Informasi Issue

- **Nomor Issue**: #588
- **Judul Issue**: Partner API: buat/ubah/batal/aktifkan ulang faktur pelanggan dan faktur mitra/bisnis, riwayat & kirim WhatsApp, serta data pajak untuk Aplikasi Pelaporan (judul disimpulkan dari isi commit)
- **Status Branch**: `Belum di-merge`.
  - Commit `resolve #588` (`55f611cf`) berada di atas `master` terbaru (`resolve #586` hasil merge Dedy, `a09d53b1`) dan sudah ada di `origin/issue-588`.
  - Commit ini menggabungkan tiga commit `save #588` (`7cda3793` 14:24, `97bc9be1` 16:57, `e7453e57` 17:32). Rinciannya di bawah dikelompokkan per berkas.

### 📅 Rincian Commit

#### [55f611cf] - resolve #588 - 25 November 2026, 17:58

- **Ringkasan**:
  - **Faktur pelanggan** (`/p-api/v1/customer-invoices/*`):
    - Admin tanpa `partner_id` kini bekerja di lingkup **pelanggan internal yang dilaporkan**. Faktur per layanan hanya boleh untuk akun yang `tax_reported`.
    - Endpoint baru: **aktifkan ulang**, **riwayat perubahan**, dan **kirim WhatsApp**.
    - Pembatalan hanya untuk faktur `unpaid` tanpa pembayaran. Syarat ini ikut klaim atomik dan dijawab 422 bila sudah ada pembayaran.
  - **Faktur mitra/bisnis** (`/p-api/v1/invoices/create|update|cancel|reactivate|updates|whatsapp`, baru, khusus sesi admin):
    - Penerimanya wajib lolos daftar-putih: reseller aktif yang tampil di portal, atau bisnis yang tampil.
  - **Tanggal terbit**:
    - Kini bisa diisi, termasuk tanggal lalu untuk admin. Mengubahnya wajib membawa `original_date` dan alasan, dan admin butuh `financeInvoice.changeSensitive`.
    - Sesi **mitra** dibatasi ke bulan berjalan (WIB).
    - Faktur mitra bertanggal sebelum bulan berjalan tidak bisa dibatalkan, diaktifkan ulang, digeser tanggalnya, atau diubah nominal/pajaknya. Alasannya: bulan lalu bisa sudah dilaporkan ke pajak.
  - **Data untuk laporan**:
    - Daftar faktur pelanggan kini membawa `dpp`, `tax_amount`, `paid_total`, `deduction_amount`, dan akun layanan (`authentication`).
    - Nomor/tanggal Faktur Pajak hanya untuk sesi admin.
    - `online_payment` (respons mentah gateway) tidak lagi dikirim.
  - **Pagar**:
    - Batas laju tulis faktur 60/5 menit dan kirim WhatsApp 30/10 menit, dihitung per sesi.
    - Faktur top up dompet disembunyikan (404).
    - `client_ref` diberi awalan pemilik per jenis lingkup.
    - Tanggal tak sah ditolak 400.
    - Filter `dateRange` yang tidak bisa diurai ditolak 400, bukan 500.

- **Penjelasan Perubahan (Sebelum → Sesudah)**:

  | Area | Sebelum | Sesudah |
  |---|---|---|
  | Lingkup admin saat **menulis** faktur pelanggan | Admin wajib mengirim `partner_id`, jadi hanya bisa menulis faktur pelanggan mitra. Lingkup pelanggan internal hanya ada untuk daftar & detail. | Tanpa `partner_id` = pelanggan internal yang tampil di Partner API. Faktur per layanan wajib untuk akun yang dilaporkan (`tax_reported`), karena faktur untuk layanan tak dilaporkan tidak akan terlihat lagi oleh pembuatnya. |
  | Aksi faktur pelanggan | Hanya daftar, detail, buat, ubah, batal. | Ditambah **aktifkan ulang**, **riwayat perubahan**, dan **kirim WhatsApp**. Aturan tunggakan pelanggan mitra kini juga berlaku saat aktifkan ulang, supaya batal-lalu-aktifkan-ulang tidak bisa melewati batas. |
  | Faktur mitra/bisnis (`type: 'partner'`) | Hanya bisa dibaca (daftar, ringkasan, detail). | Admin bisa membuat, mengubah, membatalkan, mengaktifkan ulang, melihat riwayat, dan mengirim WhatsApp. Penerima wajib lolos daftar-putih (`findWritableParty`), bukan sekadar "tidak disembunyikan", supaya mitra terhapus atau mitra lama tanpa `show_in_portal` tidak bisa ditagih. |
  | Tanggal terbit | Tidak bisa diisi maupun diubah dari Partner API: selalu tanggal hari ini. | Bisa diisi, termasuk tanggal lalu untuk admin, dengan aturan yang sama dengan form admin (tidak boleh masa depan, periode buku harus terbuka). Mengubahnya wajib `original_date` + `date_change_reason`, dan admin butuh `financeInvoice.changeSensitive`. |
  | Sesi mitra & bulan lalu | Tidak ada batas tanggal. | Mitra hanya boleh memakai tanggal di bulan berjalan (WIB). Faktur bertanggal bulan lalu tidak boleh dibatalkan, diaktifkan ulang, digeser tanggalnya, atau diubah nominal/pajaknya (422). Alasannya, faktur pelanggan mitra masuk PPN perusahaan, dan rekap PPN membaca tanggal faktur. |
  | Isi daftar faktur pelanggan | Tanpa angka pajak & pembayaran, jadi Aplikasi Pelaporan harus membuka detail satu per satu atau menghitung sendiri. | Membawa `tax_type`, `tax_inclusive`, `dpp`, `tax_amount`, `deduction_amount`, `paid_total`, `canceled_at` (hanya faktur batal), dan `authentication`. Nomor & tanggal Faktur Pajak hanya untuk sesi admin. |
  | Detail faktur | Membawa `online_payment` (respons mentah gateway). Faktur top up dompet tersembunyi di daftar, tetapi detailnya masih bisa dibuka. | `online_payment` tidak dikirim lagi. Detail faktur top up dompet dijawab 404. |
  | `client_ref` (idempotensi) | Diberi awalan pemilik. | Awalannya kini per jenis lingkup (`<mitra>`, `customer:<id>`, `party:<id>`). Tanpa pembeda jenis, `_id` reseller yang muncul di dua jalur bisa membuat retry mengembalikan faktur jenis lain. |
  | Batas laju | Tidak ada untuk faktur. | Tulis 60 per 5 menit dan WhatsApp 30 per 10 menit, per sesi (bukan per IP). Alasannya, setiap faktur/pembatalan menulis jurnal, dan template WhatsApp berbayar. |
  | Validasi input | Tanggal tak sah (mis. `1e20`) dan `dateRange` yang tak bisa diurai lolos ke service, lalu berujung 500 + alert Telegram. | Ditolak 400 di depan. Controller dan service kini memakai satu aturan tanggal (`isDateInput`). |
  | Reaktivasi faktur (semua jalur) | `canceled_at`/`canceled_by` tertinggal setelah faktur diaktifkan ulang. | Ikut dibuang, sehingga faktur aktif tidak lagi tampak "dibatalkan pada". |

- **Komponen yang Berubah & Penjelasannya**:

  - **Backend — Controller & Route**
    - `backend/src/controllers/partnerApiInvoice.controller.js`
      - Penentu lingkup faktur pelanggan:
        - `resolveCustomerInvoiceScope` untuk jalur tulis: pelanggan mitra, pelanggan reseller lewat `partner_id`, atau pelanggan internal yang tampil.
        - `resolveCustomerInvoiceReadScope` untuk jalur baca. Riwayat mitra terhapus tetap terbaca.
      - Handler baru:
        - Pelanggan: `reactivatePartnerAppCustomerInvoice`, `listPartnerAppCustomerInvoiceUpdates`, `sendPartnerAppCustomerInvoiceWhatsapp`.
        - Mitra/bisnis: `createPartnerAppInvoice`, `updatePartnerAppInvoice`, `cancelPartnerAppInvoice`, `reactivatePartnerAppInvoice`, `listPartnerAppInvoiceUpdates`, `sendPartnerAppInvoiceWhatsapp`.
      - `findWritableParty`: daftar-putih penerima faktur mitra/bisnis untuk jalur tulis.
      - Pagar tanggal terbit:
        - `ensureIssueDateChangeAllowed`: 403 `NO_PRIVILEGE` bila admin tanpa `changeSensitive` mengubah tanggal.
        - `assertPartnerIssueDateInCurrentMonth` dan `assertPartnerMayChangePastInvoice`: 422 untuk sesi mitra.
      - Aturan tunggakan (2 tagihan `unpaid` atau 1 lewat jatuh tempo) hanya untuk pelanggan mitra, dan berlaku saat membuat **dan** saat mengaktifkan ulang.
      - Kirim WhatsApp selalu ke nomor terdaftar, `phone` dari body diabaikan. Respons hanya `{ ok }`.
      - Riwayat hanya berisi aksi + waktu. `reason`/`created_by` hanya untuk sesi admin.
      - Filter `status` array tanpa `type` dinormalkan menjadi `select`, dan filter `overdue` diterjemahkan ke klausa query.
      - `stripAdminOnlyInvoiceFields` membuang data Faktur Pajak untuk sesi mitra.
    - `backend/src/routes/partnerApi.route.js`
      - Route baru `/customer-invoices/reactivate|updates|whatsapp/:id` (beserta alias tunggal), dan enam route `/invoices/*` dengan `partnerApiAdminOnly` + `partnerApiPrivilege`.
      - Limiter dipasang di semua route tulis faktur.
      - Swagger semua endpoint diperbarui, termasuk tanggal lalu + isolir otomatis bila jatuh tempo sudah lewat.
    - `backend/src/middlewares/partnerApiInvoiceLimiter.middleware.js` (baru)
      - `partnerApiInvoiceWriteLimiter` (60/5 menit): satu penghitung untuk semua aksi tulis faktur.
      - `partnerApiInvoiceWhatsappLimiter` (30/10 menit), karena template WhatsApp berbayar.
      - Keduanya per sesi pengguna (memakai kunci limiter pelanggan), bukan per IP.
    - `backend/src/controllers/partnerApiCustomer.controller.js`
      - Validasi body daftar pelanggan memakai opsi baru `ignorePagination`, menggantikan penyalinan body manual.
  - **Backend — Service & Utils**
    - `backend/src/services/financeInvoice.service.js`
      - Daftar field Partner API ditambah `tax_type`, `tax_inclusive`, `dpp`, `tax_amount`, `deduction_amount`, `paid_total`, `canceled_at`, dan `ref_auth`.
      - Konstanta baru `PARTNER_API_TAX_DOCUMENT_FIELDS` (Faktur Pajak, hanya untuk admin).
      - `shapeCustomerInvoiceListItem`: `ref_auth` dikeluarkan sebagai `authentication`, dan `canceled_at` hanya ada pada faktur batal.
      - `online_payment` dibuang dari detail.
      - Detail faktur top up dompet dijawab 404.
      - `reactivateInvoice` kini juga meng-`$unset` `canceled_at`/`canceled_by`.
    - `backend/src/services/taxReport.service.js`
      - `buildInternalBusinessInvoiceFilter` (baru): faktur bisnis internal yang merujuk layanan (`data_access`/`ref_auth`) hanya tampil bila layanannya dilaporkan.
    - `backend/src/utils/partnerApiInvoiceInput.js`
      - `date` diterima saat membuat, dan `date`/`original_date`/`date_change_reason` saat mengubah.
      - Daftar-putih baru untuk faktur mitra/bisnis: `PARTNER_API_PARTY_INVOICE_CREATE_FIELDS` dan `..._UPDATE_FIELDS`.
      - `isDateInput`: teks kosong, epoch > 0, atau teks yang menghasilkan `Date` sah (menolak `0` dan `1e20`). Satu aturan untuk controller dan service.
    - `backend/src/utils/dataTableParamsInput.js`
      - Opsi `ignorePagination`.
      - Batas `dateRange` divalidasi dengan `moment` (pengurai yang sama dengan `dataTable`), supaya teks tak sah tidak lagi memicu 500 + alert Telegram.
  - **Backend — Konfigurasi & i18n**
    - `backend/src/config/privilegeDictionary.json`
      - `apiEndpoints` diperbarui dengan route faktur Partner API yang baru.
    - `backend/src/locales/{id,en}/translation.json`
      - Key baru: `whatsappRateLimited`, `writeRateLimited`, `partnerIssueDateCurrentMonth`, `partnerPastMonthInvoiceLocked`.
  - **Dokumentasi**
    - `AGENTS.md`
      - Butir §9 "Tulis faktur dari Partner API" diperbarui sesuai semua aturan di atas.
  - **Test**
    - `backend/test/integration/partnerApiInvoice.reporting.test.js` (baru)
      - 31 test yang mencakup:
        - lingkup internal/mitra/reseller dan pelanggan internal tak dilaporkan;
        - field pajak dan akun di daftar, termasuk Faktur Pajak hanya untuk admin;
        - tanggal lalu dan hak `changeSensitive`;
        - batas bulan berjalan untuk mitra dan penguncian faktur bulan lalu;
        - aturan tunggakan saat create/reactivate;
        - batal dengan pembayaran 422;
        - faktur mitra/bisnis dan daftar-putih penerimanya;
        - riwayat, WhatsApp ke nomor terdaftar, top up dompet 404, `client_ref` per lingkup, tanggal tak sah 400.
    - `backend/test/integration/partnerApiInvoiceWriteLimiter.test.js` (baru)
      - Test batas laju: request ke-61 dari sesi yang sama dijawab 429 dengan pesan i18n, sedangkan sesi lain tidak terkena. Ada juga satu `it.each` untuk semua route tulis faktur.
    - `backend/test/unit/dataTableParamsInput.test.js` (baru)
      - 3 test: `ignorePagination` dan validasi batas `dateRange`.
    - `backend/test/integration/partnerApiInvoice.write.test.js`
      - Disesuaikan: `date` kini diterima, dan admin tanpa `partner_id` = lingkup internal. Jumlah test tetap 17.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #588 | `7cda3793`, `97bc9be1`, `e7453e57` save; `55f611cf` resolve #588 | Partner API: faktur pelanggan untuk lingkup internal yang dilaporkan + aktifkan ulang/riwayat/WhatsApp; faktur mitra/bisnis baru (khusus admin, daftar-putih penerima); tanggal terbit bisa diisi (mitra dibatasi bulan berjalan, faktur bulan lalu dikunci); data pajak & pembayaran di daftar; batas laju tulis & WhatsApp; validasi tanggal dan `dateRange`; +35 test |

