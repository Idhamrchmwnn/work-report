# 📝 Daily Work Report - Idham (2026-11-02)

---

## 📅 Laporan Harian - 2 November 2026

---

## 🌿 Branch: `issue-514` — Tanggal Jatuh Tempo & Periode Tagihan pada Berita Acara Perubahan Layanan (BAPL)

### 📌 Informasi Issue

- **Nomor Issue**: #514
- **Judul Issue**: BAPL: masa berlangganan berakhir pada tanggal jatuh tempo yang dipilih PIC, dan dokumen mendukung paket bundling (periode tagihan lebih dari 1 bulan)
- **Status Branch**: `Belum di-merge`. Masih tahap `save`:
  - dua commit `save #514` sudah ada di `origin/issue-514`;
  - sebagian perubahan lanjutan masih di working tree dan belum di-commit (`AGENTS.md`, model/route/service BAPL, test integrasi, tampilan dokumen, `utils.js` beserta test-nya, dan terjemahan).

### 📅 Rincian Commit

#### [1a9ab659] - save #514 - 2 November 2026, 16:48
#### [23d6735d] - save #514 - 2 November 2026, 16:56
#### (belum di-commit) - lanjutan #514

Rincian di bawah adalah hasil gabungan ketiganya dibanding `master`.

- **Ringkasan**:
  - Sebelumnya masa berlangganan BAPL selalu **12 bulan** sejak tanggal berlaku. Kini PIC memilih **tanggal jatuh tempo** (akhir masa berlangganan), yang wajib setelah tanggal berlaku. Nilai awalnya tetap tanggal berlaku + 12 bulan.
  - Dokumen kini mendukung **paket bundling**: periode tagihan paket lama dan baru disalin saat dokumen dibuat, memakai aturan yang sama dengan tagihan otomatis (`getEffectiveBillingPeriod`: override akun > periode paket > 1).
  - Harga tetap ditampilkan sebagai nominal satu periode penuh. Kalimat cara bayar di ketentuan mengikuti periodenya (bulanan, atau dibayar di muka untuk setiap N bulan).
  - Dokumen lama tetap aman: tanpa `due_date` tetap memakai hitungan lama (tanggal berlaku + 12 bulan), dan tanpa periode tagihan dianggap bulanan.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/models/serviceChangeDocument.model.js`
    - Field baru:
      - `due_date`: akhir masa berlangganan. Tidak wajib di skema karena dokumen lama belum memilikinya; untuk dokumen baru diwajibkan controller.
      - `old_billing_period_months` dan `new_billing_period_months`: periode tagihan saat dokumen dibuat.
    - Komentar `SERVICE_CHANGE_CONTRACT_MONTHS` diperbarui: kini hanya bawaan untuk dokumen lama.

  - `backend/src/controllers/serviceChangeDocument.controller.js`
    - `parseEffectiveDate` diganti nama menjadi `parseDateInput`, dipakai untuk kedua tanggal.
    - Create: `due_date` wajib berupa tanggal valid (400 `dueDateRequired`).
    - Update: `due_date` opsional; bila dikirim harus tanggal valid; bila kosong, jatuh tempo tersimpan tidak diubah.

  - `backend/src/services/serviceChangeDocument.service.js`
    - Template ketentuan bawaan (`DEFAULT_BAPL_TERMS`): butir harga memakai `{skema_pembayaran}`, dan butir masa berlangganan kini "sejak `{tanggal_berlaku}` sampai dengan tanggal jatuh tempo `{tanggal_jatuh_tempo}`".
    - Variabel baru di `renderBaplTerms`: `{tanggal_jatuh_tempo}`, `{skema_pembayaran}`, `{periode_tagihan}`.
    - Fungsi baru:
      - `describePaymentScheme`: kalimat cara bayar sesuai periode. Teksnya harus sama persis dengan versi frontend.
      - `resolveContractPeriod`: akhir masa berlangganan = jatuh tempo, atau tanggal berlaku + 12 bulan untuk dokumen lama. `contract_months` = selisih bulan penuh.
      - `isDueDateAfterEffective`: jatuh tempo wajib setelah tanggal berlaku, dibandingkan per hari.
      - `formatTermsDate`: format tanggal Indonesia untuk teks ketentuan.
    - `resolveSource` ikut mengambil `billing_period_months_override` (akun) dan `billing_period_months` (paket lama dan baru).
    - `createServiceChangeDocument` menolak jatuh tempo yang tidak setelah tanggal berlaku (422 `dueDateBeforeEffective`). Fungsi ini menyimpan `due_date`, periode tagihan lama/baru, dan `contract_months` hasil hitung.
    - `updateServiceChangeDocument` menghitung ulang masa berlangganan dan teks ketentuan dari jatuh tempo baru (atau yang tersimpan), dan menyimpan `contract_months` yang baru.
    - `contract_end_date` di respons kini mengikuti `due_date` bila ada.
    - Field baru ikut di daftar, detail, dan bentuk publik (`toPublicDocument`).

  - `backend/src/routes/serviceChangeDocument.route.js`
    - Swagger: `due_date` (wajib saat create, opsional saat update), field periode tagihan, dan keterangan 422 untuk jatuh tempo yang tidak setelah tanggal berlaku.

  - `backend/src/locales/{id,en}/translation.json`
    - Key baru `serviceChangeDocument.dueDateRequired` dan `dueDateBeforeEffective` (id dan en).

  - `backend/test/integration/serviceChangeDocument.test.js`
    - Paket baru 12 bulan: harga satu periode dibayar di muka.
    - Override periode di akun menang atas periode paket.
    - Halaman publik membawa periode tagihan.
    - Mengubah jatuh tempo ikut memperbarui teks ketentuan; update tanpa `due_date` tidak mengubahnya.
    - Create tanpa jatuh tempo ditolak 400; jatuh tempo sebelum tanggal berlaku ditolak 422.

  **Frontend**

  - `frontend/src/app/pages/users/document/serviceChange/utils.js` dan `utils.test.js`
    - Fungsi baru:
      - `defaultDueDate`: tanggal berlaku + 12 bulan.
      - `isDueDateAfterEffective`: aturan yang sama dengan backend.
      - `normalizeBillingPeriod` dan `describePaymentScheme`: teks sama dengan backend.
    - Variabel ketentuan baru juga didukung di render pratinjau.
    - Test: nilai awal jatuh tempo, aturan "setelah tanggal berlaku", skema pembayaran bulanan dan bundling, serta template bawaan.

  - `CreateDrawer.jsx` dan `schema/createSchema.js`
    - Input **Tanggal Jatuh Tempo** dengan nilai awal +12 bulan.
    - Validasi Yup: wajib diisi dan harus setelah tanggal berlaku.

  - `ServiceChangeDocumentPanel.jsx` (panel BAPL di drawer permintaan web/mobile)
    - Input jatuh tempo saat membuat dan mengubah dokumen. Dokumen lama diisi awal dari `contract_end_date`.
    - Tombol simpan muncul bila salah satu tanggal berubah.
    - Dokumen terkunci (sudah ditandatangani) menampilkan kedua tanggal.

  - `ServiceChangeDocumentView.jsx`
    - Baris **Periode Tagihan** (paket lama vs baru); disembunyikan untuk dokumen lama yang tidak menyimpan periode.
    - Label harga menjadi "Biaya Layanan" saat periode diketahui.
    - Baris tanggal jatuh tempo, serta variabel baru untuk teks ketentuan.

  - `schema/columns.jsx`
    - Kolom **Tanggal Jatuh Tempo** di tabel Dokumen > Perubahan Layanan, dengan filter rentang tanggal.

  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - Label dan pesan baru: jatuh tempo, periode tagihan (bulanan / dibayar di muka), dan biaya layanan.

  **Dokumentasi**

  - `AGENTS.md` §2.A butir 35 (BAPL)
    - Aturan jatuh tempo, periode tagihan lewat `getEffectiveBillingPeriod`, variabel `{skema_pembayaran}`/`{periode_tagihan}`, dan perlakuan dokumen lama.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #514 | `1a9ab659`, `23d6735d` save #514 + perubahan belum di-commit | Tanggal jatuh tempo dan periode tagihan (paket bundling) pada BAPL |

> Catatan merge (bukan pekerjaan hari ini): `resolve #501` dan `resolve #502` sudah di-merge ke `master` pada 30 Oktober malam.
