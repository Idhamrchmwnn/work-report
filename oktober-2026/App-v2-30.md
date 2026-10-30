# 📝 Daily Work Report - Idham (2026-10-30)

---

## 📅 Laporan Harian - 30 Oktober 2026

---

## 🌿 Branch: `issue-502` — Perbaikan Tanda Tangan Publik Berita Acara Pemasangan (BAP)

### 📌 Informasi Issue

- **Nomor Issue**: #502
- **Judul Issue**: Tanda tangan BA Pemasangan lewat link publik gagal untuk tiket Pelanggan Bisnis, serta pengamanan endpoint publiknya
- **Status Branch**: `Belum di-merge`. Satu commit `resolve #502` di atas `master` terbaru (setelah `resolve #497`), sudah ada di `origin/issue-502`.

### 📅 Rincian Commit

#### [b177572e] - resolve #502 - 30 Oktober 2026, 17:42

- **Ringkasan**:
  - Sebelumnya `POST /public-docs/bap/update` mencari pelanggan dari `cust_id` kiriman browser. Untuk tiket Pelanggan Bisnis, `cust_id` berisi `partner_id`, sehingga simpan tanda tangan selalu gagal 404.
  - Kini pemilik dokumen (pelanggan reguler atau mitra/bisnis) diturunkan dari **tiket pemasangan**, dengan urutan yang sama seperti isi BA (`getBAP`): bila tiket memuat mitra, dokumen milik mitra.
  - Endpoint publik ini sekaligus diperketat:
    - validasi bentuk body dan tanda tangan;
    - satu BAP per tiket, dijaga atomik;
    - tiket batal ditolak;
    - batas laju per tiket;
    - body tidak masuk log.
  - Ikut diperbaiki: halaman detail Dedicated/Akses Data yang error saat dirender, dan tautan layanan dedicated di detail tiket.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/controllers/publicDocument.controller.js` (`updateBAPByCode`)
    - Body disaring `pickBapSignInput`. Body tidak lengkap, nama penanda tangan kosong, atau tanda tangan tidak sah dijawab 400 (sebelumnya bisa 500).
    - Tiket dicari dengan syarat `type: 'installation'` dan belum dihapus. Tiket terhapus dijawab 404 agar keberadaannya tidak terkonfirmasi.
    - Tiket yang dibatalkan ditolak 422 (`publicDocument.ticketCanceled`). Tiket yang masih berjalan tetap boleh ditandatangani, karena BA ditandatangani di lokasi.
    - BAP yang sudah tersimpan untuk tiket itu ditolak 409.
    - Pemilik dokumen diambil dari `ticket.partner` lalu `ticket.customer`. Bila keduanya kosong, dijawab 404.
    - Admin pembuat link yang belum mengunggah tanda tangan dijawab 422, bukan 500.
    - Dokumen disimpan lewat `createUniqueTicketDocument`. Bentrok indeks pada kiriman bersamaan dijawab 409.

  - `backend/src/utils/signatureInput.js` [NEW]
    - Aturan bersama tanda tangan publik `checkSignatureInput`: nama wajib dan maksimal 100 karakter; tanda tangan wajib data URL PNG ≤ 512 KB. Pola ini juga menolak URL gambar luar yang akan dimuat browser admin.
    - Dipakai bersama BAPL (`serviceChangeDocument.service.js`). Konstanta lama diekspor ulang dari sana, jadi pemanggil lama tidak berubah.

  - `backend/src/utils/bapSignInput.js` [NEW]
    - `pickBapSignInput`: cek tipe `ticketId`/`userId` (string) dan `documentData`/`documentData.sign` (objek). Hasil `checkSignatureInput` dipetakan ke key i18n `publicDocument.*`.

  - `backend/src/models/document.model.js`
    - Indeks unik parsial `bap_ticket_active_unique` (`ticket`, untuk `type: 'BAP'` dan `deleted: false`): satu BAP aktif per tiket. BAP yang di-soft-delete membebaskan tiketnya.
    - Dokumen V1 tanpa field `deleted` tidak tercakup indeks ini, jadi cek awal di controller tetap dipertahankan.

  - `backend/src/services/document.service.js`
    - Fungsi baru `createUniqueTicketDocument`. Error duplikat (11000) dikembalikan sebagai `{ duplicate: true }`; error lain dicatat `logger.error` dengan konteks.

  - `backend/src/middlewares/publicBapSignLimiter.middleware.js` [NEW]
    - Batas 10 percobaan per 15 menit, dengan kunci **per tiket**, bukan per IP. Alasannya: di produksi `req.ip` selalu alamat proxy internal, sehingga limiter per IP menjadi satu kuota bersama untuk semua pengunjung. Jawaban 429 memakai `publicDocument.rateLimited`.

  - `backend/src/routes/public.route.js`
    - Limiter dipasang di `POST /public-docs/bap/update`.
    - Swagger ditulis ulang sesuai body sebenarnya (`ticketId`, `userId`, `documentData.sign`) dan kode respons 200/400/404/409/422/429.

  - `backend/src/middlewares/logger.middleware.js`
    - `/api/v1/public-docs/bap/update` dan `/bad/update` ditambahkan ke `BODY_REDACTED_PREFIXES`. Body keduanya berisi gambar tanda tangan serta nama, alamat, dan HP pelanggan.

  - `backend/src/services/ticket.service.js`
    - Populate `dedicated` pada tiket ditambah `capacity` (dipakai teks produk BA) dan `is_dedicated_internet` (menentukan tujuan tautan layanan di detail tiket).

  - `backend/src/locales/{id,en}/translation.json`
    - Key baru `publicDocument.ticketCanceled`, `signerNameRequired`, `rateLimited` (id dan en).

  - Test
    - `backend/test/integration/publicDocumentBap.test.js` [NEW], 14 skenario:
      - dua kiriman bersamaan: hanya satu yang tersimpan;
      - soft delete membebaskan tiket;
      - pemilik dokumen untuk tiket bisnis dan reguler;
      - `cust_id` kiriman browser diabaikan;
      - tiket batal, tiket terhapus, dan tiket tanpa pemilik;
      - simpan ulang ditolak 409;
      - body tidak lengkap dan nama kosong ditolak 400;
      - admin tanpa tanda tangan dijawab 422;
      - tiket berisi pelanggan dan mitra: dokumen milik mitra;
      - populate dedicated memuat `capacity` dan `is_dedicated_internet`.
    - `backend/test/unit/bapSignInput.test.js` [NEW]: `pickBapSignInput`, `checkSignatureInput`, dan kunci limiter (per tiket; `ticketId` bukan string berbagi kunci `invalid`).
    - `backend/test/unit/loggerSanitizer.test.js`: kedua route BA publik diredaksi.

  **Frontend (`frontend` dan `partner-frontend`)**

  - `src/app/pages/services/dataAccess/detail.jsx` dan `src/app/pages/services/dedicatedInternet/detail.jsx`
    - Field `equipment` (array objek dari tiket) dan `dismantle` dilewati saat merender tabel detail. Sebelumnya keduanya dirender mentah dan membuat React error.
    - Badge mitra hanya dirender bila datanya ada (selain itu `-`), dengan privilege yang benar `business.read` (sebelumnya `admin.read`).
    - Akses `value.name` dan `value.map` diberi optional chaining.

  - `src/app/pages/tickets/GeneralInformation.jsx`
    - Tautan layanan dedicated diarahkan ke rute yang benar, `/services/dedicated-internet/view/:id`.

---

## 🌿 Branch: `issue-501` — Persen Potongan Faktur Dihitung dari DPP

### 📌 Informasi Issue

- **Nomor Issue**: #501
- **Judul Issue**: Persentase potongan faktur (mis. PPh 23) dihitung dari DPP, bukan dari total setelah PPN
- **Status Branch**: `Belum di-merge`. Satu commit `resolve #501`, sudah ada di `origin/issue-501`.

### 📅 Rincian Commit

#### [6b12f90b] - resolve #501 - 30 Oktober 2026, 18:22

- **Ringkasan**:
  - Potongan persen pada faktur sebelumnya dihitung dari total **setelah** PPN. Untuk PPh 23, dasar pemotongannya seharusnya **DPP** (sebelum PPN).
    - Contoh: DPP Rp1.000.000 dengan PPN 11%. PPh 23 2% kini Rp20.000 (dari DPP), bukan Rp22.200 (dari total berpajak).
  - Potongan tetap dikurangkan dari total setelah pajak, dan batas atasnya tetap total berpajak. Rumus `dpp + tax_amount === total + deduction_amount` tetap berlaku.
  - Potongan bernilai nominal tidak berubah.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/services/financeInvoice.service.js`
    - `validateInvoiceDeductions` kini menerima dua angka terpisah: `baseTotal` (DPP, dasar persen) dan `taxedTotal` (total berpajak, batas atas). Pengurang ≥ total berpajak tetap ditolak 422 `INVOICE_DEDUCTION_EXCEEDS_TOTAL`.
    - `createNewInvoice` dan `updateInvoiceDetail` memanggilnya dengan `dpp` dan `taxedTotal`. Saat hanya item yang diubah, potongan lama dihitung ulang terhadap DPP baru.
    - Komentar `computeInvoiceDeductions` diperbarui (contoh PPh 23).

  - `backend/src/routes/financeInvoice.route.js`
    - Swagger field `deductions` pada create dan update: persen dihitung dari DPP.

  - `backend/test/integration/financeInvoice.deduction.test.js`
    - Angka harapan disesuaikan dengan dasar DPP.

  - `AGENTS.md`
    - Aturan §2.A butir 30 diperbarui: persen dihitung dari DPP (dasar pemotongan PPh 23).

  **Frontend**

  - `frontend/src/app/pages/finance/invoices/InvoiceItemsField.jsx`
    - Pratinjau potongan memakai `dpp` sebagai dasar persen, sama seperti server.

  - `frontend/src/app/pages/finance/invoices/utils/invoiceDeductions.js` dan `invoiceDeductions.test.js`
    - Komentar diperbarui. Test kini memakai contoh PPh 23 2% dari DPP 1.000.000 = 20.000.

  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - `deductionHint` menjelaskan bahwa persen dihitung dari DPP.
    - Placeholder keterangan diganti menjadi "Mis. PPh 23".

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #502 | `b177572e` resolve #502 | Tanda tangan publik BA Pemasangan untuk tiket bisnis dan pengamanan endpointnya, perbaikan render detail Dedicated/Akses Data, dan tautan dedicated di detail tiket |
| #501 | `6b12f90b` resolve #501 | Persen potongan faktur (PPh 23) dihitung dari DPP |

> Catatan merge: `resolve #498` dan `resolve #497` (dikerjakan 29 Oktober) sudah di-merge hari ini. `resolve #490` (BAPL) sudah di-merge semalam (29 Oktober, 23.09).
