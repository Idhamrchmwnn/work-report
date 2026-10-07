# 📝 Daily Work Report - Idham (2026-10-07)

---

## 📅 Laporan Harian - 7 Oktober 2026

---

## 🌿 Branch: `issue-378` — Ubah Tanggal Terbit Tagihan Penjualan

### 📌 Informasi Issue

- **Nomor Issue**: #378 (berdiri sendiri)
- **Judul Issue**: Ubah Tanggal Terbit Tagihan Penjualan
- **Status Branch**: `resolve #378` — satu commit, langsung selesai, sudah di-push ke `origin/issue-378`.
- **Dokumen pendamping**: `.agent/invoice.md` — mencakup rancangan lengkap, termasuk satu kali **pembatalan rencana rev.1** (koreksi jurnal in-place) setelah data produksi dicek dan ternyata tidak relevan, lalu rev.2 → rev.2.1 setelah self-review terhadap AGENTS.md.

### 🧭 Latar Belakang & Temuan Kunci

Field `date` (tanggal terbit) di `FinanceInvoice` sebelumnya **tidak bisa diubah sama sekali** — tidak termasuk kelompok field apa pun di `updateInvoiceDetail`, jadi diabaikan diam-diam bila dikirim, dan picker-nya hanya tampil saat membuat faktur baru. Riset terhadap salinan data produksi (72.126 faktur, per 31 Agustus 2026) mengungkap kebutuhan nyata sekaligus bug lama yang baru ketahuan:

- **0 faktur** ber-`issued_journal` — fitur accrual belum pernah aktif di produksi, jadi seluruh faktur masih cash-basis. Ini membatalkan rencana awal (rev.1) yang merancang koreksi jurnal in-place dengan mutex + jurnal balik — jalur itu tidak akan pernah terpakai, dan koreksi jurnal in-place pun bukan pola yang dimiliki modul keuangan lain manapun di sistem ini.
- **1.301 faktur tercatat lunas SEBELUM tanggal terbitnya** — pola nyata: faktur dibuat terlambat (mis. dibuat 3 Oktober tapi tanggal bayarnya 1 Oktober), bukan "cron telat" seperti asumsi awal rencana.
- **Bug lama yang baru ketahuan saat merancang fitur ini**: 868 faktur `unpaid`/`paid` punya `due_date` < `date` di data produksi — akibatnya validasi jatuh tempo yang sudah berjalan sejak dulu **selalu gagal** untuk faktur-faktur itu, bahkan untuk edit sepele seperti ganti nama. Diperbaiki bersamaan sebagai bagian dari fitur ini (lihat R3.1a di bawah).

### 📅 Rincian Perubahan

#### [892baf62] - resolve #378 - 7 Oktober 2026, 18:06 WIB (21 file, 2.116 baris ditambah, 24 dihapus)

**Prinsip inti**: faktur yang **sudah dijurnal penerbitan** (accrual) tetap tidak boleh diubah tanggalnya — mengikuti prinsip baku modul keuangan di proyek ini (dokumen terjurnal dibatalkan lalu dibuat ulang, bukan ditulis ulang). Karena belum ada faktur terjurnal di produksi, fitur ini langsung berguna untuk seluruh faktur cash-basis yang ada.

- [`backend/src/utils/invoiceIssueDatePolicy.js`](backend/src/utils/invoiceIssueDatePolicy.js) [NEW, 249 baris] — Fungsi murni `resolveIssueDatePolicy()`, **satu sumber kebenaran** dipakai baik oleh validasi service maupun respons `issue_date_policy` yang dikirim ke frontend (supaya picker tanggal di UI tidak perlu menghitung aturan sendiri). Mengembalikan `editable`, `reason` (`status`/`journaled`/`period_closed`/`no_valid_range`), `same_month_only`, `min`/`max` rentang tanggal yang sah. Sengaja **tidak** memasukkan `due_date` ke batas `max` — supaya admin tetap bisa memajukan tanggal terbit sambil memundurkan jatuh tempo dalam satu simpan (ditemukan sebagai bug desain saat review rev.2, diperbaiki sebelum commit).
- **11 aturan bisnis (R1–R11)** diterapkan di `updateInvoiceDetail` ([`financeInvoice.service.js`](backend/src/services/financeInvoice.service.js), +390 baris) — hanya berlaku bila tanggal **benar-benar berubah** (beda hari; kalau sama, `date` dibuang dari proses tanpa validasi apa pun, mencegah regresi pada edit biasa):
  - Status harus `unpaid`/`paid`, belum dijurnal, periode lama **dan** baru masih terbuka, tanggal baru ≤ tanggal bayar pertama (dicari lewat `findFirstPaymentDate` — urut cek cicilan non-void → `FinancePayment.date` → `invoice.paid_date`), faktur auto-billing (`ref_auth` tanpa periode) atau berpajak wajib tetap di bulan yang sama, batas `accrual_start_date` dihormati bila suatu saat diisi, jatuh tempo efektif ≥ tanggal baru, alasan perubahan wajib diisi (maks. 500 karakter), dan privilege khusus wajib dimiliki.
  - **Konkurensi**: satu `findOneAndUpdate` atomik dengan filter `date: originalDate` + `issued_journal: { $exists: false }` + `paid_total: paidTotalAtValidation` — form basi atau pembayaran yang masuk di tengah proses sama-sama terdeteksi sebagai **409** (`INVOICE_ISSUE_DATE_CONFLICT`), tidak pernah menimpa diam-diam. **Pembayaran sendiri tidak pernah dikunci** — yang gagal hanya proses ubah-tanggalnya, admin cukup memuat ulang.
  - **Bug jebakan Mongoose yang ditemukan & dicegah sebelum sempat terjadi**: dokumen yang di-hydrate Mongoose membaca field yang sebenarnya tidak ada di database (faktur V1 lama) sebagai nilai default schema (`paid_total: 0`). Filter atomik `paid_total: 0` murni **tidak akan cocok** dengan field yang memang tidak ada — kalau tidak diperbaiki, ini akan membuat **~44.000 faktur V1** (yang memang tidak punya field `paid_total` tersimpan) **selalu gagal 409** saat tanggalnya dicoba diubah. Diperbaiki dengan filter `paid_total: { $in: [0, null] }`, dan ditegaskan test yang sengaja membuat dokumen tanpa field tersebut (insert lewat driver mentah, bukan `Model.create`).
  - **Perbaikan bug lama (§3.1a)**: validasi jatuh tempo kini hanya dijalankan bila `due_date` sendiri beda hari dari yang tersimpan, atau bila `date` ikut berubah — bukan selalu membandingkan ke `invoice.date` seperti sebelumnya. Ini yang memperbaiki kegagalan edit pada 868 faktur lama yang disebut di atas.
- [`financePeriod.service.js`](backend/src/services/financePeriod.service.js) [+19] — `isPeriodOpen()` (dipakai cek periode lama & baru, pakai `.lean()`).
- [`finance-error.js`](backend/src/utils/finance-error.js) [+11] — 3 kode error baru: `INVOICE_ISSUE_DATE_LOCKED` (422), `INVOICE_ISSUE_DATE_CONFLICT` (409), `INVOICE_ISSUE_DATE_REASON_REQUIRED` (400). Status tidak sah dipetakan ke kode `INVOICE_NOT_EDITABLE` yang sudah ada (409) untuk konsisten dengan jalur edit lain, bukan 422 seperti draf awal rencana.
- [`financeInvoice.controller.js`](backend/src/controllers/financeInvoice.controller.js) [+24/-x] — bila `date` yang dikirim beda hari dari yang tersimpan, wajib privilege `financeInvoice.changeSensitive` (403 bila tidak ada); detail faktur kini menyertakan `issue_date_policy`.
- **Privilege baru `financeInvoice.changeSensitive`**: `privilege.json` [+3], `privilegeDictionary.json` [+14] (`requires: ['financeInvoice.update', 'financeInvoice.read']`), `privilegeDescriptions.{id,en}.json` [+1 masing-masing]. `sync-privilege-dictionary.js` [+2] — didaftarkan ke `CONTROLLER_LEVEL_ENDPOINTS` supaya `npm run gd` tetap mengisi `apiEndpoints`-nya (kalau tidak, `privilegeDictionary.test.js` gagal).
- **Fail-closed saat query pendukung gagal**: `findIssueDatePolicy` menangkap kegagalan (mis. koneksi ke `FinancePayment` bermasalah), mencatatnya lewat `logger.error` dengan `invoice_id`, lalu mengembalikan kebijakan **terkunci** (`reason: 'unavailable'`) — detail faktur tetap tampil, tidak ikut gagal (AGENTS.md §2.A aturan 3).
- **Bug ditemukan lewat test, diperbaiki sebelum commit**: detail faktur mem-*populate* `ref_auth`; akun radius yang sudah terhapus ter-*populate* jadi `null`, sehingga aturan "bulan sama" sempat hilang dari respons `issue_date_policy` ke UI padahal server tetap menolaknya di belakang. Diperbaiki dengan membaca ID asli lewat `invoice.populated('ref_auth')`.
- Locales `backend/src/locales/{id,en}/translation.json` [+9 masing-masing], `frontend/src/i18n/locales/{id,en}/translations.json` [+16 masing-masing].

- **Frontend** — [`InvoiceDrawer.jsx`](frontend/src/app/pages/finance/invoices/InvoiceDrawer.jsx) [+119/-x]:
  - Picker tanggal terbit kini **juga tampil di mode edit**, aktif hanya bila `issue_date_policy.editable` **dan** privilege dimiliki; kalau tidak, tetap tampil nonaktif dengan hint alasan (sesuai AGENTS.md §2.B.20 — disembunyikan hanya bila memang tidak ada kebutuhan UX khusus, di sini sengaja ditampilkan nonaktif agar admin tahu kenapa).
  - **Jebakan flatpickr yang dihindari**: awalnya direncanakan memakai prop `minDate`/`maxDate`, tapi itu akan **membuang** (menghilangkan dari tampilan) tanggal tersimpan yang berada di luar rentang kebijakan saat ini — kasus nyata: faktur yang terbit setelah dibayar. Diganti memakai prop `enable` (rentang kebijakan **plus** hari yang sudah tersimpan tetap selalu bisa ditampilkan).
  - Alasan perubahan tanggal (textarea, wajib) hanya muncul bila tanggal benar-benar berubah per hari.
  - [`issueDate.js`](frontend/src/app/pages/finance/invoices/utils/issueDate.js) [NEW, 95 baris] — fungsi murni pembanding tanggal & pembangun payload (`date`/`original_date`/`date_change_reason` hanya dikirim bila harinya berubah), dipisah supaya bisa di-test tanpa merender komponen.
  - `schema/invoiceSchema.js` [+59/-x] — `date`/`due_date` diubah dari `yup.string()` ke `yup.mixed()` (string akan mengubah objek `Date` jadi teks `Date#toString()`, merusak validasi); validasi jatuh tempo ≥ tanggal terbit kini berlaku juga di mode **buat** faktur, menyamai aturan backend.

- **Test** (verifikasi langsung, bukan hanya klaim dokumen rencana):
  - [`invoiceIssueDatePolicy.test.js`](backend/test/unit/invoiceIssueDatePolicy.test.js) [306 baris, 23 skenario] — seluruh kombinasi `reason`/`same_month_reason`, termasuk rentang kosong → `no_valid_range`.
  - [`financeInvoice.issueDate.test.js`](backend/test/integration/financeInvoice.issueDate.test.js) [558 baris, 28 skenario] — regresi (edit nama/catatan pada faktur lunas/terjurnal/V1 tanpa `paid_total` tetap sukses dengan jam asli tidak tertimpa), kasus positif (geser dalam & lintas bulan, faktur `activation_period` lintas bulan), seluruh kasus negatif R1–R11, dan kegagalan query pendukung (fail-closed).
  - [`issueDate.test.js`](frontend/src/app/pages/finance/invoices/utils/issueDate.test.js) [142 baris, 10 skenario].
  - **Hasil**: backend 3.340 lulus/1 gagal (`profanityFilter.test.js`, kegagalan performa yang sudah ada sebelumnya, tidak terkait perubahan ini — dibuktikan dengan menjalankannya sendiri terpisah). Frontend 376/376 lulus. ESLint bersih, `npm run build` frontend berhasil.

### ⚠️ Belum Dikerjakan / Perlu Tindak Lanjut

- **Uji manual di aplikasi** (faktur V1 lunas dimundurkan ke tanggal bayar, faktur mitra lintas bulan, faktur `ref_auth`, periode tertutup, konflik dua tab, admin tanpa privilege) — belum dilakukan.
- **Privilege `financeInvoice.changeSensitive` wajib di-assign ke role keuangan setelah rilis** — tanpa langkah ini, tidak ada admin yang bisa memakai fitur ini sama sekali meski kodenya sudah aktif.
- Konfirmasi `accrual_start_date` dan timezone server **produksi** (asumsi Asia/Jakarta) belum dilakukan — keduanya krusial karena seluruh batas tanggal fitur ini bergantung pada asumsi itu.
- **Lampiran A (Fase 2 — koreksi tanggal faktur yang sudah terjurnal accrual)** didesain tapi sengaja belum dikerjakan; wajib dikerjakan **sebelum atau bersamaan** dengan pengaktifan accrual di masa depan, kalau tidak hampir semua faktur baru akan langsung terjurnal dan tanggalnya tidak bisa dikoreksi lagi oleh fitur ini.
- Riwayat perubahan tanggal (`changes.date.from/to` + alasan) sudah tersimpan di database tapi **belum ditampilkan** di halaman detail faktur — komponen riwayat yang ada belum merender `changes`.

### 📖 Informasi Singkat Fitur

- **Kegunaan**: Admin keuangan kini bisa mengoreksi tanggal terbit faktur penjualan yang salah input atau terlambat dibuat (termasuk lintas bulan), langsung lewat form Edit yang sudah ada — sebelumnya field ini sama sekali tidak bisa diubah dan perubahannya diabaikan diam-diam tanpa pemberitahuan apa pun ke pengguna.
- **Cara pakai**: Buka faktur, klik Edit, ubah tanggal terbit (picker hanya mengizinkan tanggal yang sah secara aturan — tetap tampil walau nonaktif dengan alasan bila tidak boleh diubah). Begitu tanggal digeser, kolom "Alasan perubahan tanggal" wajib diisi. Simpan seperti biasa. Faktur yang sudah dijurnal penerbitan (accrual, belum aktif di sistem saat ini) tidak bisa diubah tanggalnya lewat cara ini — harus dibatalkan lalu dibuat ulang.

---

## 📢 Ringkasan Dampak

| Branch | Dampak Utama |
| --- | --- |
| `issue-378` | Tanggal terbit faktur penjualan kini bisa dikoreksi (termasuk lintas bulan) lewat 11 aturan bisnis yang ketat, sekaligus memperbaiki bug lama yang memblokir edit pada 868 faktur ber-`due_date` tidak konsisten dan mencegah bug baru yang akan memblokir ~44.000 faktur V1. |
