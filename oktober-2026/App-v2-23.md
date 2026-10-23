# 📝 Daily Work Report - Idham (2026-10-23)

---

## 📅 Laporan Harian - 23 Oktober 2026

---

## 🌿 Branch: `issue-466` — Pengurang/Fee pada Faktur Penjualan

### 📌 Informasi Issue

- **Nomor Issue**: #466
- **Judul Issue**: Pengurang/fee faktur penjualan yang memotong total setelah pajak
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-466`)

### 📅 Rincian Commit

#### [9f9f1900] - resolve #466 - 23 Oktober 2026, 15:34

- **Ringkasan**:
  - Faktur penjualan (**Keuangan > Tagihan Penjualan**) kini bisa diberi satu atau beberapa baris **pengurang/fee**, misalnya fee marketplace atau potongan kerja sama. Tiap baris berupa persen atau nominal Rupiah.
  - Urutan hitung: subtotal item, lalu pajak, lalu total berpajak, baru dikurangi fee. DPP dan pajak tidak berubah karena fee.
  - Fee dibukukan sebagai debit ke akun kontra-pendapatan **4-9000 Diskon & Potongan Penjualan**, sehingga jurnal tetap seimbang dan laporan laba rugi benar.
  - Ditutup juga celah aplikasi mobile pelanggan: detail faktur bisa dibaca pelanggan lain lewat nomor faktur tebakan.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Model & Perhitungan**

  - `backend/src/models/financeInvoice.model.js`
    - Sub-schema baru `InvoiceDeductionSchema` berisi `description`, `type` (`percent`/`nominal`), `value`, dan `amount`. Dibuat sebagai schema tersendiri karena punya field `type`.
    - Field baru `deductions` (array) dan `deduction_amount` (total fee). Berlaku `dpp + tax_amount === total + deduction_amount`, dengan `total` = nominal yang ditagihkan.
    - Index sparse `deductions.description` untuk saran keterangan.
  - `backend/src/services/financeInvoice.service.js`
    - `computeInvoiceDeductions`: satu-satunya penghitung fee. Persen dihitung dari total berpajak, nominal dipakai apa adanya, dan `amount` selalu dihitung server; nilai `amount` kiriman klien diabaikan.
    - `validateInvoiceDeductions`:
      - Keterangan wajib, maks 200 karakter, maks 20 baris, persen maks 100. Pelanggaran dijawab 400.
      - Baris bernilai 0 dibuang.
      - Total fee yang sama dengan atau melebihi total tagihan ditolak **422**, karena faktur Rp0 tidak akan pernah bisa dilunasi.
    - Create dan update faktur menghitung `total = total berpajak − fee`.
      - Saat hanya item yang diubah, fee persen lama dihitung ulang terhadap total baru.
      - Array fee kosong menghapus seluruh fee.
    - `findInvoiceDeductionSelect`: saran keterangan fee dari faktur sebelumnya.
      - Diurutkan dari yang paling sering dipakai, maksimal 10.
      - Tidak membedakan huruf besar/kecil.
      - Kata kunci di-escape sebelum masuk regex.
    - Baris jurnal kini mendukung sisi `debit` (untuk baris fee), selain kredit.
    - Laporan penjualan per produk dihitung **setelah** fee: porsi fee dibagi ke item secara proporsional, supaya laporan cocok dengan laporan berbasis `total`.
    - Field fee ikut dipilih di jalur baca yang membutuhkannya. Partner API hanya menerima `deduction_amount`, tanpa keterangan per baris.
  - `backend/src/services/financeInvoiceClassifier.service.js`
    - `classifyInvoiceAmount` membagi pembayaran faktur berfee:
      - Pendapatan dan pajak diakui dari nilai kotor.
      - Ditambah baris debit ke akun 4-9000 sebesar porsi fee.
    - Porsi fee pada pembayaran cicilan dihitung **kumulatif**, sehingga jumlah porsi seluruh cicilan selalu persis `deduction_amount` tanpa selisih Rp1.
    - Pengaman: faktur yang dimuat tanpa field `deduction_amount` ditolak keras (500 + `logger.error`), supaya tidak terbukukan diam-diam dengan angka salah.
    - Baris pajak residual tidak lagi bisa bernilai negatif bila data DPP tidak konsisten.
  - `backend/src/services/financePayment.service.js`, `backend/src/services/financeWallet.service.js`
    - Pembayaran manual dan pembayaran dari saldo dompet meneruskan `paidAfter` (total terbayar setelah cicilan) ke classifier, dan menulis baris debit/kredit sesuai sisinya.
  - `backend/src/services/financeCoa.service.js`
    - Konstanta `SALES_DEDUCTION_CODE = '4-9000'` (Diskon & Potongan Penjualan).
  - `backend/src/services/financeReport.service.js`
    - **Perbaikan laporan Laba Rugi dan Neraca:** saldo akun pendapatan/beban kini dihitung menurut golongannya (pendapatan = kredit − debit, beban = debit − kredit), bukan menurut `normal_balance` per akun.
    - Dengan rumus lama, akun kontra seperti 4-9000 justru **menambah** laba. Kini akun itu tampil negatif dan mengurangi pendapatan.
  - `backend/src/services/financeGateway.service.js`
    - Field fee ikut dimuat saat membuat dan menyelesaikan pembayaran online.
  - `backend/src/services/paymentGateways/ipaymu.gateway.js`
    - Faktur berfee dikirim ke iPaymu sebagai **satu baris** senilai total. Rincian item (harga kotor) melebihi nominal yang dibayar, sehingga iPaymu akan menolak atau menampilkan rincian yang salah.
    - Faktur lain tetap dikirim per item seperti sebelumnya.
  - `backend/src/utils/finance-error.js`
    - Kode error baru `INVOICE_DEDUCTION_EXCEEDS_TOTAL`, yang dipetakan ke 422.

  **Backend — Controller, Route & Keamanan**

  - `backend/src/controllers/financeInvoice.controller.js`
    - Create/update meneruskan `deductions`.
    - Handler baru `financeInvoiceDeductionSelect`.
    - **Perbaikan keamanan `getMyInvoice` (aplikasi mobile pelanggan):** detail faktur kini dibatasi ke pelanggan yang login. Sebelumnya pelanggan mana pun bisa membaca faktur pelanggan lain lewat nomor faktur tebakan. Faktur milik orang lain dijawab 404, sama seperti faktur yang tidak ada.
  - `backend/src/controllers/publicInvoice.controller.js`
    - Halaman faktur publik hanya menerima total `deduction_amount`. Keterangan per baris bersifat internal dan tidak pernah dikirim.
  - `backend/src/routes/financeInvoice.route.js`
    - Route baru `POST /finance/invoice/deduction-select` (`financeInvoice.list`).
    - Swagger create/update diperbarui untuk field `deductions` dan respons 422.
  - `backend/src/routes/mobileCustomer.route.js`, `backend/src/routes/public.route.js`, `backend/src/routes/partnerApi.route.js`
    - Swagger diperbarui: 404 untuk faktur milik orang lain di mobile, dan field `deduction_amount` di respons.
  - `backend/src/config/privilegeDictionary.json`
    - Endpoint `deduction-select` ditambahkan ke `financeInvoice.list`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `invalidDeduction`, `deductionExceedsTotal`, dan `deductionFieldMissing`.

  **Backend — Test**

  - `backend/test/integration/financeInvoice.deduction.test.js` [NEW]
    - **Perhitungan dan validasi:**
      - Persen dari total berpajak, nominal apa adanya.
      - `amount` kiriman klien diabaikan, dan faktur tanpa fee tidak menyimpan field fee.
      - Validasi 400 dan 422.
      - Urutan hitung pajak dulu, termasuk mode harga sudah termasuk pajak.
    - **Jurnal:**
      - Jurnal penerbitan seimbang.
      - Cicilan seimbang, dengan porsi fee kumulatif persis.
      - Faktur tanpa field fee ditolak.
    - **Lainnya:**
      - Update item menghitung ulang fee persen.
      - Saran keterangan (urutan, aman dari regex, tidak membedakan huruf besar/kecil).
      - Laporan penjualan per produk.
      - Laba Rugi dan Neraca tetap seimbang.
  - `backend/test/integration/financeInvoice.mobileOwnership.test.js` [NEW]
    - Pemilik bisa membaca fakturnya; pelanggan lain dijawab 404.
  - `backend/test/integration/ipaymuGateway.deductionPayload.test.js` [NEW]
    - Faktur berfee dikirim satu baris; faktur biasa tetap per item.

  **Frontend**

  - `frontend/src/app/pages/finance/invoices/InvoiceItemsField.jsx`
    - Bagian baru **Pengurang / Fee** di form faktur: tambah atau hapus baris, isi keterangan, tipe (persen/nominal), dan nilai.
    - Keterangan memakai dropdown saran (`/finance/invoice/deduction-select`) bila memegang `financeInvoice.list`; tanpa hak itu, keterangan berupa input teks biasa.
    - Ringkasan menampilkan "Total Setelah Pajak", tiap fee, lalu total tagihan.
    - Ada petunjuk bahwa keterangan bersifat internal (pelanggan hanya melihat "Fee") dan tidak boleh berisi data pribadi.
  - `frontend/src/app/pages/finance/invoices/utils/invoiceDeductions.js` [NEW], `frontend/src/app/pages/finance/invoices/utils/invoiceDeductions.test.js` [NEW]
    - Pratinjau perhitungan fee dengan rumus yang sama persis dengan backend; server tetap menjadi sumber kebenaran.
    - Juga berisi helper membaca fee tersimpan dan menyusun payload (tanpa `amount`, tanpa baris kosong), beserta test-nya.
  - `frontend/src/app/pages/finance/invoices/schema/invoiceSchema.js`
    - Validasi Yup untuk fee: tipe, nilai minimal 0, persen maks 100, keterangan wajib bila bernilai (maks 200 karakter), maks 20 baris.
  - `frontend/src/app/pages/finance/invoices/InvoiceDrawer.jsx`
    - Memuat fee tersimpan saat mengubah faktur dan mengirimnya saat simpan.
  - `frontend/src/app/pages/finance/invoices/detail.jsx`
    - Detail faktur menampilkan setiap baris fee beserta keterangannya.
  - `frontend/src/app/pages/finance/invoices/InvoiceDocument.jsx`
    - Dokumen faktur untuk pelanggan menampilkan "Total Setelah Pajak" dan satu baris total **"Fee"**, tanpa keterangan per baris.
  - `frontend/src/components/shared/form/Combobox.jsx`
    - Mode `tags`: teks yang diketik ikut menjadi nilai saat kolom ditinggalkan (blur), tanpa harus mengklik "Buat ...". Sebelumnya teks bebas hilang bila admin langsung pindah kolom atau menekan Simpan.
    - Ketikan yang sama dengan opsi yang ada (tanpa membedakan huruf besar/kecil) memakai opsi itu, supaya tidak muncul duplikat "fee" dan "Fee".
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Label bagian Pengurang/Fee, petunjuk, tipe, nilai, "Total Setelah Pajak", dan pesan validasi.

  **Dokumentasi & Basis Pengetahuan**

  - `AGENTS.md`
    - Aturan baru §2.A nomor 30, "Pengurang/fee faktur penjualan": urutan hitung, satu-satunya penghitung, dan pembukuan ke akun kontra.
    - Pola Bab 9 "Akun kontra di laporan keuangan": saldo dihitung per golongan, bukan per `normal_balance`.
  - `backend/src/data/aiKnowledge/features.json`
    - Dibangkitkan ulang (`npm run gknow`).

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #466 | Pengurang/fee faktur penjualan | Faktur bisa dipotong fee setelah pajak dengan jurnal dan laporan keuangan yang tetap benar; celah baca faktur di aplikasi mobile ditutup |

### Kemampuan Baru Pengguna/Admin

- Menambahkan pengurang/fee (persen atau nominal) ke faktur penjualan, dengan saran keterangan dari fee yang pernah dipakai.
- Pelanggan melihat total setelah pajak dan satu baris "Fee" di dokumen faktur; rincian keterangan tetap internal.

### Bug Fix / Solusi Masalah

- Pelanggan di aplikasi mobile tidak bisa lagi membaca faktur pelanggan lain lewat nomor faktur tebakan.
- Laporan Laba Rugi dan Neraca kini menghitung akun kontra (mis. Diskon & Potongan Penjualan) sebagai pengurang pendapatan. Sebelumnya menambah laba.
- Teks bebas di kolom combobox mode `tags` tidak lagi hilang saat admin langsung pindah kolom atau menyimpan.

### Menu/Fitur Baru

- Bagian **Pengurang / Fee** di form **Keuangan > Tagihan Penjualan**.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Fee dipotong dari total tagihan **setelah** pajak. Pajak tetap dihitung dari harga item, lalu total berpajak dikurangi fee menjadi nominal yang ditagihkan ke pelanggan. Di pembukuan, fee dicatat ke akun 4-9000 Diskon & Potongan Penjualan sehingga pendapatan bersih di laporan sudah memperhitungkan fee.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Keuangan > Tagihan Penjualan**, lalu buat atau ubah faktur.
  2. Isi item dan pajak seperti biasa.
  3. Di bagian **Pengurang / Fee**, klik **Tambah Pengurang/Fee**. Isi keterangan (mis. "Fee marketplace"), pilih tipe **%** atau **Rp**, lalu isi nilainya.
  4. Periksa ringkasan "Total Setelah Pajak", baris fee, dan total tagihan, lalu simpan.
