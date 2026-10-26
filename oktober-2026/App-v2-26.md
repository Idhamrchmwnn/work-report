# 📝 Daily Work Report - Idham (2026-10-26)

---

## 📅 Laporan Harian - 26 Oktober 2026

> Pekerjaan `save #479` tanggal 24 Oktober (fondasi Live Invoice) sudah dilaporkan di [App-v2-24.md](App-v2-24.md). Laporan ini hanya berisi lanjutan hari ini.

---

## 🌿 Branch: `issue-479` — Live Invoice: Log Aktivitas & Penyempurnaan

### 📌 Informasi Issue

- **Nomor Issue**: #479
- **Judul Issue**: Live Invoice: admin memantau secara realtime aktivitas pelanggan di link pembayaran faktur
- **Status Branch**: `Belum di-merge`. Di-rebase ke `master` terbaru, di-squash menjadi satu commit `resolve #479`, dan sudah ada di `origin/issue-479`. Siap PR.

### 📅 Rincian Commit

#### [ebc7c6a4] - resolve #479 - 26 Oktober 2026, 13:13

- **Ringkasan**:
  - Panel Live Invoice di detail faktur kini punya **log aktivitas**: riwayat kejadian di link pembayaran, misalnya perangkat membuka link, memilih metode, pindah ke aplikasi lain, kembali, menutup, instruksi pembayaran terbit, lunas, kedaluwarsa, pembayaran manual dicatat, atau faktur diubah/dibatalkan.
  - Indikator tahap bertingkat diganti dengan linimasa log tersebut.
  - Endpoint presence publik dilindungi batas laju tambahan untuk permintaan yang gagal.
  - Beberapa service kini mencatat error sebelum melempar ulang.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Log Aktivitas**

  - `backend/src/utils/liveInvoicePresence.js`
    - Fungsi murni baru untuk log:
      - `presenceLogType`: menentukan jenis entri dari perubahan tahap pengunjung (dibuka, memilih, menunggu bayar, ke latar belakang, kembali, keluar). Heartbeat tanpa perubahan tidak dicatat.
      - `buildPresenceLogEntry`: menyusun entri log; metode bayar hanya disertakan untuk tahap memilih/menunggu.
      - `isDuplicatePaymentLog`: mencegah kejadian pembayaran yang sama tercatat dua kali, misalnya callback gateway dan cek status pelanggan sama-sama melaporkan lunas.
      - `toAdminLog`: mengubah log mentah menjadi bentuk untuk admin, dengan urutan terbaru di atas dan ID perangkat diganti nomor "Perangkat N".
    - Daftar jenis kejadian pembayaran (`PAYMENT_LOG_TYPES`) dan batas 100 entri per faktur (`MAX_LOG_ENTRIES`).
  - `backend/src/services/liveInvoice.service.js`
    - `appendLiveInvoiceLog` dan `getLiveInvoiceLog`: log disimpan sementara di Redis (list per faktur, maksimal 100 entri, kedaluwarsa 24 jam). Tidak disimpan ke MongoDB.
    - Perubahan presence yang bermakna otomatis dicatat.
    - Tab yang hilang tanpa sempat melapor "tutup" (sinyal putus, HP terkunci lama) dicatat sebagai *timeout* pada waktu laporan terakhirnya.
    - Snapshot admin kini membawa field `log`.
  - `backend/src/sockets/liveInvoice.controller.js`
    - `notifyLiveInvoiceChanged` menerima parameter kejadian dan mencatatnya ke log. Pencatatan tetap berjalan walau socket tidak tersedia.
    - Helper baru `gatewayStatusLogEvent` memetakan status gateway: `paid`/`settled` menjadi lunas, `expired`/`canceled`/`failed` menjadi ditutup.
    - Snapshot lewat socket ikut membawa log.

  **Backend — Pemicu Kejadian**

  - `backend/src/controllers/publicPayment.controller.js`
    - Instruksi pembayaran terbit dicatat beserta metode/kanalnya, untuk transaksi satu faktur maupun gabungan.
    - Cek status oleh pelanggan mencatat status hasilnya.
  - `backend/src/controllers/financeGateway.controller.js`
    - Callback gateway mencatat lunas atau ditutup. Penutupan yang ditolak verifikasi gateway (transaksi masih hidup) tidak dicatat.
  - `backend/src/controllers/financePayment.controller.js`
    - Mencatat instruksi pembayaran yang diterbitkan admin, cek status admin, pembayaran manual beserta nominalnya, dan pembatalan pembayaran.
    - Pembayaran massal **hanya** mencatat faktur yang benar-benar terbayar; item yang gagal tidak muncul sebagai "pembayaran dicatat".
  - `backend/src/controllers/financeInvoice.controller.js`
    - Faktur diubah, dibatalkan, atau diaktifkan kembali dicatat di log.
  - `backend/src/controllers/internal.controller.js`
    - Sweep cron mencatat status tiap transaksi yang berubah, bukan sekadar memberi tahu satu kali untuk semuanya.
  - `backend/src/services/financeGateway.service.js`
    - Callback `onChanged` pada sweep kini menyertakan status transaksi.
    - `findGatewayTransactionInvoiceRefs` mencatat `logger.error` sebelum melempar ulang bila pembacaan gagal.
  - `backend/src/services/publicPaymentTarget.service.js`
    - `resolvePaymentTargetForInvoice` dan `buildPublicPaymentOptions` kini mencatat `logger.error` sebelum melempar ulang (AGENTS.md §2.A.3).

  **Backend — Keamanan & Dokumentasi Endpoint**

  - `backend/src/routes/public.route.js`
    - Limiter baru `publicPresenceMissLimiter` di `POST /public/invoice/:payment_code/presence`: maksimal 20 respons **gagal** per 15 menit per IP.
    - Tujuannya supaya endpoint presence tidak menjadi jalan menebak `payment_code` yang lebih longgar daripada `GET /public/invoice/:payment_code`. Tab sah pada faktur yang sudah lunas berhenti setelah satu 404, jadi tidak terkena.
  - `backend/src/routes/financeInvoice.route.js`
    - Swagger `GET /finance/invoice/:invoice_id/live` dilengkapi field `log` beserta jenis-jenis entrinya.

  **Backend — Test**

  - `backend/test/integration/liveInvoice.test.js`
    - Perubahan presence tercatat di log, sedangkan heartbeat tidak.
    - Kejadian pembayaran tercatat sekali per transaksi.
    - Pembayaran massal hanya mencatat faktur yang berhasil.
  - `backend/test/unit/liveInvoicePresence.test.js`
    - `presenceLogType` (dibuka, tutup, latar belakang/kembali, heartbeat vs ganti kanal), `buildPresenceLogEntry`, `isDuplicatePaymentLog`, dan `toAdminLog`.

  **Frontend — Panel Live Invoice**

  - `frontend/src/app/pages/finance/invoices/live/LiveInvoicePanel.jsx`
    - Indikator tahap bertingkat (4 langkah) diganti **linimasa log aktivitas** memakai komponen `Timeline`.
    - Tiap entri menampilkan nomor perangkat, kejadian, metode/nominal bila ada, dan waktu. Waktu disesuaikan ke jam server, supaya "x detik lalu" tidak meleset.
    - Entri terbaru berdenyut saat ada pengunjung aktif.
    - Warna indikator memakai token tema (`success-*`), bukan warna Tailwind bawaan.
  - `frontend/src/app/pages/finance/invoices/live/liveInvoiceStage.js`
    - Fungsi baru `describeMethod` (label metode/kanal), `resolveLogKey` (kunci teks per jenis kejadian), dan `logColor` (warna per jenis). Penghitungan langkah lama dihapus.
  - `frontend/src/app/pages/finance/invoices/live/liveInvoiceStage.test.js`
    - Test fungsi-fungsi baru di atas.
  - `frontend/src/app/pages/finance/invoices/live/useLiveInvoice.js`
    - Event socket `live-invoice:presence` kini memperbarui daftar pengunjung **dan** log sekaligus, jadi linimasa bertambah tanpa memuat ulang snapshot.
  - `frontend/src/app/pages/finance/invoices/detail.jsx`
    - Posisi panel Live Invoice di halaman detail faktur dipindah.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Teks judul log, log kosong, dan teks tiap jenis kejadian. Teks langkah lama yang tidak dipakai dihapus.

  **Dokumentasi**

  - `AGENTS.md`
    - Aturan §2.A nomor 31 "Live Invoice" diperbarui dengan aturan log aktivitas: penyimpanan sementara di Redis, batas entri, pencegahan duplikat kejadian pembayaran, dan entri *timeout*.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #479 | Live Invoice: log aktivitas & penyempurnaan | Admin melihat riwayat lengkap aktivitas di link pembayaran faktur, bukan hanya tahap terakhir; endpoint presence publik lebih tahan penebakan kode |

### Kemampuan Baru Pengguna/Admin

- Melihat **log aktivitas** link pembayaran di detail faktur: kapan dibuka, metode yang dipilih, pindah ke aplikasi lain, instruksi pembayaran terbit, lunas/kedaluwarsa, pembayaran manual, dan perubahan faktur. Setiap entri dilengkapi nomor perangkat dan waktunya.

### Bug Fix / Solusi Masalah

- Pembayaran massal tidak lagi memberi tahu faktur yang gagal dibayar.
- Kejadian lunas yang dilaporkan dua jalur (callback dan cek status) tidak lagi tercatat ganda.
- Endpoint presence publik tidak lagi bisa dipakai menebak kode pembayaran secara massal.

### Menu/Fitur Baru

- Linimasa **Log aktivitas** di panel Live Invoice (**Keuangan > Tagihan Penjualan > detail faktur**).

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Setiap kejadian di link pembayaran faktur dicatat sementara (sampai 100 entri, bertahan 24 jam) dan ditampilkan sebagai linimasa di panel Live Invoice. Kejadian dari sisi pelanggan berasal dari laporan halaman publik, sedangkan kejadian pembayaran berasal dari server (gateway, pembayaran manual, perubahan faktur), sehingga status lunas tetap bersumber dari server.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Keuangan > Tagihan Penjualan**, lalu buka detail faktur yang belum lunas dan punya link bayar.
  2. Bagikan link faktur ke pelanggan.
  3. Pantau bagian **Log aktivitas** di panel Live Invoice. Entri baru muncul otomatis di paling atas.
