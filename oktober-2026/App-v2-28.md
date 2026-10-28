# 📝 Daily Work Report - Idham (2026-10-28)

---

## 📅 Laporan Harian - 28 Oktober 2026

---

## 🌿 Branch: `issue-493` — Pengurang pada Pembayaran Faktur Manual

### 📌 Informasi Issue

- **Nomor Issue**: #493
- **Judul Issue**: Pengurang (mis. biaya transfer yang dipotong) saat mencatat pembayaran faktur manual
- **Status Branch**: `Belum di-merge`. Sudah satu commit `resolve #493` di atas `master` terbaru dan sudah ada di `origin/issue-493`.

### 📅 Rincian Commit

#### [013022ea] - resolve #493 - 28 Oktober 2026, 17:17

- **Ringkasan**:
  - Saat mencatat pembayaran faktur secara manual, admin kini bisa mengisi **Pengurang** (nominal). Contohnya biaya transfer yang dipotong bank pelanggan.
  - Tagihan tetap berkurang sebesar **nominal pembayaran penuh**. Kas/bank hanya menerima **nominal − pengurang**.
  - Selisihnya dijurnal **debit** ke akun **4-9000 Diskon & Potongan Penjualan**, sehingga jurnal tetap seimbang.
  - Pengurang hanya berlaku untuk pembayaran manual. Pembayaran lewat payment gateway tidak terpengaruh, karena fee gateway sudah dihitung sendiri dari data verifikasi gateway.
  - Validasi di frontend dan backend: pengurang harus bilangan bulat ≥ 0 dan lebih kecil dari nominal pembayaran.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/models/financePayment.model.js`
    - Field baru `deduction` (Number) pada cicilan pembayaran (`PaymentInstallmentSchema`).
    - Hanya diisi bila ada pengurang, jadi dokumen lama dan cicilan tanpa pengurang tidak berubah bentuk.

  - `backend/src/services/financePayment.service.js`
    - `recordInvoicePayment` menerima parameter `deduction`.
      - Jalur gateway selalu memakai pengurang `0`.
      - Kas bersih (`netAmount`) = nominal (dikurangi fee gateway bila ditanggung merchant) − pengurang.
      - Pengurang ditolak dengan `INVALID_AMOUNT` dan pesan `finance.payment.deductionInvalid` bila bukan bilangan bulat, negatif, atau ≥ nominal. Penolakan ini terjadi sebelum faktur, kas, atau jurnal tersentuh.
      - Cicilan menyimpan `deduction` bila nilainya > 0.
    - `writeInstallmentJournal` menerima `deductionAmount`.
      - Bila > 0, ditambahkan baris debit ke akun `SALES_DEDUCTION_CODE` (4-9000) dengan keterangan "(Pengurang)".
      - Bila akun COA tersebut tidak ada, proses dihentikan dengan `COA_NOT_FOUND` alih-alih membuat jurnal tidak seimbang.
    - Komentar fungsi diperbarui untuk menjelaskan perlakuan pengurang.

  - `backend/src/controllers/financePayment.controller.js`
    - `financePaymentCreate` membaca `deduction` dari body dan meneruskannya ke service sebagai angka (kosong = `0`).

  - `backend/src/routes/financePayment.route.js`
    - Swagger `POST /finance/payment/create`: properti `deduction` beserta aturannya, dan respons `400` untuk data tidak valid atau pengurang ≥ nominal.

  - `backend/src/locales/{id,en}/translation.json`
    - Key baru `finance.payment.deductionInvalid` dan `finance.payment.deductionLabel` (id dan en).

  - `backend/test/integration/financePayment.deduction.test.js` [NEW]
    - Pembayaran 100.000 dengan pengurang 6.500:
      - faktur lunas (`paid_total` 100.000),
      - saldo bank bertambah 93.500,
      - jurnal seimbang,
      - baris debit 6.500 masuk ke akun 4-9000.
    - Void cicilan berpengurang mengembalikan saldo bank ke 0 dan faktur kembali `unpaid` dengan `paid_total` 0.
    - Pengurang 100.000, 150.000, −1, dan 10,5 ditolak `INVALID_AMOUNT` tanpa mengubah faktur.

  **Frontend**

  - `frontend/src/app/pages/finance/invoices/PaymentDrawer.jsx`
    - Input **Pengurang** (`InputMoney`) di bawah nominal, dengan ikon info dan tooltip penjelasan (`data-tooltip`).
    - Saat pengurang diisi, tampil rincian: Nominal, Pengurang, dan **Diterima Kas/Bank**. Kotak rincian berwarna merah bila pengurang ≥ nominal.
    - Pengurang dikirim ke `/finance/payment/create`. Bila kosong, field tidak dikirim sama sekali.

  - `frontend/src/app/pages/finance/invoices/schema/paymentSchema.js`
    - Validasi Yup `deduction`: angka ≥ 0, opsional, dan harus lebih kecil dari nominal (`finance.payment.deductionExceeds`).

  - `frontend/src/app/pages/finance/invoices/detail.jsx`
    - Riwayat pembayaran di drawer detail faktur menampilkan "Pengurang: Rp …" di bawah nominal cicilan yang berpengurang.

  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - Key baru `finance.payment.deduction`, `deductionHint`, `deductionExceeds`, `netReceived` (id dan en).

---

## 🌿 Branch: `issue-490` — Berita Acara Perubahan Layanan (BAPL)

### 📌 Informasi Issue

- **Nomor Issue**: #490
- **Status Branch**: `Belum di-merge`. Sudah di-squash menjadi `resolve #490` dan ada di `origin/issue-490`.

### 📅 Rincian Commit

#### [fa9ff0d4] - resolve #490 - 28 Oktober 2026, 17:32

- **Ringkasan**:
  - Commit `save #490` (BAPL, rinciannya ada di laporan 27 Oktober) di-rebase ke `master` terbaru lalu di-squash menjadi satu commit `resolve #490`, sesuai alur Git proyek.
  - Hari ini tidak ada perubahan isi fitur di branch ini.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #493 | `013022ea` resolve #493 | Pengurang pada pembayaran faktur manual: backend, jurnal ke 4-9000, frontend, test integrasi |
| #490 | `fa9ff0d4` resolve #490 | Rebase dan squash BAPL menjadi satu commit siap PR |
