# 📝 Daily Work Report - Idham (2026-10-24)

---

## 📅 Laporan Harian - 24 Oktober 2026

---

## 🌿 Branch: `issue-480` — Pemisahan Endpoint Pelanggan Reguler & Pelanggan Mitra di Partner API

### 📌 Informasi Issue

- **Nomor Issue**: #480
- **Judul Issue**: Partner API: endpoint pelanggan reguler (`/customers`) dan pelanggan mitra (`/customer-partners`) dipisah, beserta pengerasan validasi input
- **Status Branch**: `Belum di-merge` (masih tahap `save`, sudah di-push ke `origin/issue-480`)

### 📅 Rincian Commit

#### [d5d07843] - save #480 - 24 Oktober 2026, 23:38

- **Ringkasan**:
  - Endpoint pelanggan di Partner API (p-api) kini dipisah per jenis:
    - `/p-api/v1/customers/*` khusus **pelanggan reguler**.
    - `/p-api/v1/customer-partners/*` khusus **pelanggan milik mitra**.
  - Sebelumnya satu endpoint mencampur keduanya, sehingga jenis pelanggan bisa tertukar atau berpindah lewat body update.
  - Jalur create/update juga dikeraskan terhadap input yang tidak terduga.
  - Fungsi perbaikan data `setCustomerPID` diperbaiki supaya penanda jenis pelanggan (`partner`) dan `pid` selalu sejalan.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Partner API**

  - `backend/src/controllers/partnerApiCustomer.controller.js`
    - Ditulis ulang menjadi **pabrik handler per jenis** (`CUSTOMER_KIND`: reguler/mitra): `makeListHandler`, `makeListStatusHandler`, `makeReadHandler`, `makeCreateHandler`, `makeUpdateHandler`, `makeDeleteHandler`, `makeChangeStatusHandler`, `makeSetDocumentHandler`, dan `makeDeleteAvatarHandler`.
      - Tiap pabrik menghasilkan dua handler, misalnya `listPartnerAppCustomer` dan `listPartnerAppCustomerPartner`.
    - **Cakupan per jenis** (`buildOwnerFilter`, `buildReadScope`, `buildWriteScope`):
      - Pelanggan reguler hanya ada di `/customers`, dan pelanggan mitra hanya di `/customer-partners`. Membaca silang dijawab 404.
      - Sesi mitra tidak pernah memiliki pelanggan reguler: daftar dan statistik di `/customers` kosong, dan create di `/customers` ditolak **403**.
    - **Jenis pelanggan tidak bisa berpindah lewat update:**
      - Endpoint reguler menolak 400 bila body mengirim `partner`.
      - Endpoint mitra menolak 400 bila `partner` dikosongkan (`remove-data`); nilai kosong dianggap "tidak diubah".
      - Pindah ke mitra yang tersembunyi dari portal, atau `partner_id` berbentuk operator Mongo, ditolak.
    - **Pengerasan input:**
      - `wallet` dan `customer_id` tidak pernah diterima dari body. Sebelumnya body create bisa menyuntik saldo awal tanpa transaksi, atau membajak penomoran `customer_id`.
      - Field teks (nama, HP, dll.) divalidasi bertipe string **sebelum** diproses. Sebelumnya nilai array/angka/objek memicu TypeError dan menghasilkan 500 mentah; kini 400.
      - ID pelanggan berbentuk objek operator Mongo (mis. `{ $ne: ... }`) ditolak. Sebelumnya permintaan "ubah pelanggan X" bisa berubah menjadi "ubah pelanggan apa pun yang cocok operator".
      - `documents` dikeluarkan dari daftar field yang boleh diubah lewat JSON, supaya dokumen (termasuk KTP) hanya bisa diubah lewat jalur unggah berkas yang divalidasi.
      - Penggantian dokumen dipecah menjadi dua operasi (`$pullAll` lalu `$push`), karena MongoDB menolak keduanya pada path yang sama dalam satu update.
      - Geocoding kota tetap berjalan di belakang layar, kini dengan `.catch()` supaya kegagalannya tidak menjadi unhandled rejection.
    - Populate `partner` hanya dijalankan untuk admin di endpoint pelanggan mitra; jenis lain tidak membutuhkannya.
  - `backend/src/routes/partnerApi.route.js`
    - Route baru `/customer-partners/*` (dengan alias singular `/customer-partner/*`): list, list-status, read, create, update, delete, change-status, dokumen, dan avatar.
    - Admin yang membuat pelanggan mitra wajib menyebut `partner`, yaitu mitra reseller yang tampil di portal.
    - Swagger kedua kelompok endpoint diperbarui sesuai pembagian jenis, kode error 400/403/404, dan contoh.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `customer.regularOnlyPartnerApi` (endpoint khusus pelanggan reguler) dan `customer.partnerRequired` (mitra pemilik wajib diisi).

  **Backend — Perbaikan Data**

  - `backend/src/services/recovery.service.js`
    - `setCustomerPID` menyelaraskan `pid` pelanggan dengan pemiliknya: tanpa `partner` menjadi `master`, dan dengan `partner` menyalin `pid` mitra.
      - Kini mencakup pelanggan yang sudah soft-delete.
      - Hanya menulis dokumen yang berbeda, jadi aman dijalankan berulang (idempoten).
    - Mitra yang dokumennya hilang tidak lagi membuat proses crash. Mitra seperti itu dilewati, dilaporkan di hasil (`missing_partners`), dan dicatat di log ringkasan.

  **Backend — Test**

  - `backend/test/integration/partnerApiCustomer.{list,listStatus,read,create,update,delete,changeStatus,documents}.test.js`, `backend/test/integration/partnerApiCustomer.archived.test.js`, `backend/test/integration/customerTaxReported.test.js`
    - Disesuaikan dengan pembagian endpoint.
    - Ditambah kasus admin per jenis, sesi mitra di `/customers` (kosong/403/404), larangan pindah jenis lewat update, mitra tersembunyi, dan `partner_id` berbentuk operator.
    - Ditambah kelompok test validasi input ekstrem, *mass assignment*, auth/state, dan kegagalan dependensi.
  - `backend/test/integration/recoveryCustomerPid.test.js` [NEW]
    - Penyalinan `pid` mitra termasuk pelanggan soft-delete, `master` untuk pelanggan tanpa mitra, mitra hilang dilewati tanpa crash, dan idempoten.

  **Dokumentasi**

  - `AGENTS.md`
    - Aturan ke-11 ditambahkan pada pola Bab 9 "Visibilitas di Partner API":
      - Pelanggan di p-api dipisah per endpoint.
      - Jenis diikat di route lewat pabrik `make*Handler(CUSTOMER_KIND.*)`, bukan dari body.
      - Penanda jenis adalah `Customer.partner`, bukan `pid`.
      - Syarat pemilik dan pengecualian mitra tersembunyi wajib dalam satu kunci `partner`.
      - Data yang `pid`-nya tidak sejalan dirapikan lewat `setCustomerPID` (Developer > Perbaikan Data).

---

## 🌿 Branch: `issue-479` — Live Invoice (Pantau Aktivitas Pelanggan di Link Pembayaran)

### 📌 Informasi Issue

- **Nomor Issue**: #479
- **Judul Issue**: Live Invoice: admin memantau secara realtime aktivitas pelanggan di link pembayaran faktur
- **Status Branch**: `Belum di-merge` (masih tahap `save`, sudah di-push ke `origin/issue-479`)

### 📅 Rincian Commit

#### [56e7537b] - save #479 - 24 Oktober 2026, 19:36

- **Ringkasan**:
  - Admin keuangan yang membagikan link faktur kini bisa melihat **secara realtime** di detail faktur apa yang sedang terjadi di sisi pelanggan:
    - Belum dibuka atau sedang dibuka.
    - Sedang memilih metode bayar (mis. QRIS, VA BCA).
    - Menunggu pembayaran, dengan QR/nomor VA dan hitung mundur yang sama.
    - Tidak aktif di halaman.
    - Lunas, kedaluwarsa, atau batal.
  - Tampilannya sama dengan halaman pelanggan karena memakai komponen bersama. Ini bukan berbagi layar: yang terlihat hanya tahap, metode, dan data pembayaran.
  - Rancangan dan catatan implementasi ada di `.agent/liveinvoices.md`. Fase 1 dan 2 dikerjakan; Fase 3 sengaja tidak.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Presence & Snapshot**

  - `backend/src/services/publicPaymentTarget.service.js` [NEW]
    - Logika penentuan target pembayaran (satu faktur, gabungan, diarahkan ke faktur lebih lama, nonaktif, tertutup) dan pembentukan data pembayaran publik dipindah dari controller ke service ini.
    - Fungsi `buildPublicPaymentOptions` dipakai bersama oleh endpoint publik dan snapshot admin, jadi datanya dijamin sama.
  - `backend/src/controllers/publicPayment.controller.js`
    - Kini hanya memanggil service di atas, tanpa perubahan perilaku.
    - Handler baru `recordPublicInvoicePresence`.
    - Transaksi baru dan cek status yang menghasilkan pelunasan memberi tahu Live Invoice.
  - `backend/src/utils/liveInvoicePresence.js` [NEW]
    - Fungsi murni:
      - `parsePresenceBody` (JSON maupun teks dari `sendBeacon`).
      - `sanitizePresenceInput`: format `viewer_id`, tahap dari daftar tetap (tidak pernah `paid`; status bayar bukan dari klien), metode hanya pada tahap memilih/menunggu.
      - `isPresenceStale`: tahap aktif basi setelah 60 detik, `away` lebih lama.
      - `isMeaningfulPresenceChange`: heartbeat biasa tidak disiarkan.
      - `toAdminViewers`: alias "Perangkat N", tanpa `viewer_id`.
  - `backend/src/services/liveInvoice.service.js` [NEW]
    - Menyimpan presence sementara di Redis (hash per faktur, TTL 15 menit), maksimal 10 perangkat per faktur.
    - Snapshot untuk admin berisi status faktur, mode pembayaran, opsi pembayaran (bentuk SAMA dengan halaman publik), daftar perangkat, tahap tertinggi, dan `server_time`, supaya hitung mundur tidak bergantung jam komputer admin.
    - Bila Redis bermasalah, fitur ini gagal diam (`live_available: false`) dan pembayaran pelanggan tetap berjalan.
  - `backend/src/sockets/liveInvoice.controller.js` [NEW]
    - Event `live-invoice:subscribe` dan `live-invoice:unsubscribe`. Hak `financeInvoice.read` dibaca **dari database**, bukan dari input klien, dan jumlah room per socket dibatasi.
    - Siaran `live-invoice:presence` (daftar perangkat yang sudah disaring) dan `live-invoice:changed` (admin memuat ulang snapshot).
    - `notifyLiveInvoiceChanged` bersifat best-effort: tidak pernah melempar error, juga saat `io` kosong. Transaksi gabungan memberi tahu semua faktur alokasinya.
  - `backend/src/sockets/socket-io.js`
    - Mendaftarkan handler Live Invoice.

  **Backend — Endpoint & Pemicu Pemberitahuan**

  - `backend/src/routes/public.route.js`
    - Endpoint baru `POST /public/invoice/:payment_code/presence`:
      - Rate limit sendiri, dan menerima `text/plain` (maks 2 KB) untuk `sendBeacon`.
      - Balasan `204` tanpa isi; faktur lunas atau tidak dikenal dijawab 404.
      - Dilengkapi Swagger.
  - `backend/src/routes/financeInvoice.route.js`
    - Endpoint baru `GET /finance/invoice/:invoice_id/live` (`financeInvoice.read`, key yang sudah ada) beserta Swagger.
  - `backend/src/controllers/financeInvoice.controller.js`
    - Handler `financeInvoiceLive`. Perubahan faktur memberi tahu Live Invoice.
  - `backend/src/controllers/financeGateway.controller.js`, `backend/src/controllers/financePayment.controller.js`, `backend/src/controllers/internal.controller.js`
    - Memberi tahu Live Invoice saat transaksi berubah: callback gateway (lunas/pending/tutup), cek status oleh admin, pembayaran manual/void, dan sweep cron.
  - `backend/src/services/financeGateway.service.js`
    - `findGatewayTransactionInvoiceRefs` mencari faktur terkait sebuah transaksi; transaksi lama tanpa field `invoice` diturunkan dari `reference_id`.
    - Sweep transaksi menerima callback `onChanged`, jadi service tetap tanpa socket.
  - `backend/src/services/financeInvoice.service.js`
    - Helper `findInvoiceStatusByPaymentCode` dan `findInvoiceIdsByObjectIds`.
  - `backend/src/config/privilegeDictionary.json`
    - Endpoint `/live` ditambahkan ke `financeInvoice.read`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Satu key pesan baru.

  **Backend — Test**

  - `backend/test/integration/liveInvoice.test.js` [NEW]
    - **Endpoint presence:** 400 untuk laporan tidak valid; 404 untuk faktur lunas/tidak dikenal; 204 dan hanya perubahan tahap yang disiarkan; `closed` menghapus perangkat; batas 10 perangkat.
    - **Snapshot:** 404; `payment_options` sama persis dengan endpoint publik; faktur lunas.
    - **Socket:** subscribe ditolak tanpa hak atau tanpa login.
    - **Pemberitahuan:** gabungan dan transaksi lama; tidak pernah melempar.
  - `backend/test/unit/liveInvoicePresence.test.js` [NEW]
    - Seluruh fungsi murni presence.
  - `backend/test/setup/redis.stub.js`
    - Stub Redis ditambah perintah hash yang dipakai presence.

  **Frontend — Komponen Pembayaran Bersama**

  - `frontend/src/components/shared/payment/PaymentInstructionView.jsx` [NEW]
    - Tampilan instruksi pembayaran yang diekstrak dari halaman publik: QR/QRIS, nomor VA + salin, total, fee, hitung mundur, dan instruksi kanal.
    - Prop baca-saja untuk admin.
  - `frontend/src/components/shared/payment/PaymentChannelPicker.jsx` [NEW]
    - Pemilih metode bayar. Di panel admin tampil dalam mode baca, dengan kanal pilihan pelanggan tersorot.
  - `frontend/src/components/shared/payment/paymentDisplay.js` [NEW], `frontend/src/components/shared/payment/paymentDisplay.test.js` [NEW]
    - Util murni tampilan pembayaran (hitung mundur dan sejenisnya) beserta test-nya.
  - `frontend/src/app/pages/public/PublicInvoicePayment.jsx`
    - Halaman bayar publik kini memakai komponen bersama di atas (sekitar 600 baris berkurang), dengan tampilan tetap sama.

  **Frontend — Pelaporan dari Halaman Pelanggan**

  - `frontend/src/app/pages/public/hooks/useInvoicePresence.js` [NEW]
    - Melaporkan tahap pelanggan: dibuka, memilih metode (debounce), menunggu pembayaran, pindah tab (`away`), dan tutup halaman (`sendBeacon`).
    - Heartbeat 20 detik hanya saat tab terlihat; respons 404 menghentikan laporan.
    - Memakai `fetch`/`sendBeacon`, bukan axios, supaya loading bar global tidak berkedip di layar pelanggan.
  - `frontend/src/app/pages/public/utils/invoicePresence.js` [NEW], `frontend/src/app/pages/public/utils/invoicePresence.test.js` [NEW]
    - Logika murni presence (`viewer_id` per tab di `sessionStorage`, penentuan tahap) beserta test-nya.

  **Frontend — Panel Admin**

  - `frontend/src/app/pages/finance/invoices/live/LiveInvoicePanel.jsx` [NEW]
    - Indikator tahap berurutan, badge "Live", jumlah perangkat, dan aktivitas terakhir.
    - Metode yang sedang dipilih pelanggan, dan instruksi pembayaran baca-saja bila QR/VA sudah terbit.
    - Mode khusus: diarahkan ke faktur lain, dibayar gabungan, link bayar nonaktif.
  - `frontend/src/app/pages/finance/invoices/live/useLiveInvoice.js` [NEW]
    - Memuat snapshot, berlangganan room socket, dan memuat ulang snapshot saat ada perubahan, socket tersambung ulang, atau tab kembali terlihat. Respons basi ditolak.
    - Perangkat basi disaring tiap 15 detik, dan snapshot disegarkan tiap 60 detik sebagai jaring pengaman.
  - `frontend/src/app/pages/finance/invoices/live/liveInvoiceStage.js` [NEW], `frontend/src/app/pages/finance/invoices/live/liveInvoiceStage.test.js` [NEW]
    - Penentuan tahap dan label untuk panel, beserta test-nya.
  - `frontend/src/app/pages/finance/invoices/detail.jsx`
    - Menampilkan `LiveInvoicePanel` untuk faktur belum lunas yang punya link bayar, hanya bagi pemegang `financeInvoice.read`.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Label tahap, mode, dan teks panel Live Invoice.

  **Dokumentasi**

  - `AGENTS.md`
    - Aturan baru §2.A nomor 31 "Live Invoice": laporan halaman publik hanya informasi tampilan, status bayar tetap dari server, dan presence sementara di Redis.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #480 | Pemisahan endpoint pelanggan Partner API | Pelanggan reguler dan pelanggan mitra tidak lagi tercampur atau bisa berpindah jenis lewat p-api; input berbahaya ditolak rapi |
| #479 | Live Invoice | Admin melihat realtime apakah link faktur sudah dibuka, metode apa yang dipilih, dan kapan pelanggan membayar |

### Kemampuan Baru Pengguna/Admin

- Di detail faktur, admin melihat panel **Live Invoice**: status link, jumlah perangkat yang membuka, metode yang sedang dipilih, serta QR/VA dan hitung mundur yang sama dengan pelanggan.
- Aplikasi eksternal mengelola pelanggan mitra lewat endpoint tersendiri `/p-api/v1/customer-partners/*`.

### Bug Fix / Solusi Masalah

- Body create pelanggan p-api tidak bisa lagi menyuntik saldo `wallet` atau `customer_id`.
- Input bertipe salah di p-api kini dijawab 400, bukan 500.
- ID pelanggan berbentuk operator Mongo tidak bisa lagi dipakai untuk mengubah pelanggan lain.
- Dokumen pelanggan (termasuk KTP) tidak bisa lagi ditimpa lewat JSON mentah.
- Perbaikan data `setCustomerPID` tidak lagi crash saat ada mitra yang dokumennya hilang, dan kini ikut merapikan pelanggan yang sudah dihapus.

### Menu/Fitur Baru

- Panel **Live Invoice** di detail **Keuangan > Tagihan Penjualan**.
- Endpoint Partner API `/p-api/v1/customer-partners/*`.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur (Live Invoice)**: Setelah link faktur dibagikan ke pelanggan, halaman bayar pelanggan melaporkan tahapnya ke server: dibuka, memilih metode, menunggu pembayaran, atau pindah aplikasi. Admin melihatnya di detail faktur tanpa perlu menyegarkan halaman. Status lunas tetap hanya ditentukan dari gateway atau pembayaran yang dicatat admin. Aktivitas di aplikasi bank tidak terlihat; admin paling jauh melihat "tidak aktif di halaman" lalu "Lunas" saat callback masuk.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Keuangan > Tagihan Penjualan**, lalu buka detail faktur yang belum lunas dan punya link bayar.
  2. Bagikan link faktur ke pelanggan.
  3. Pantau panel **Live Invoice**: tahap berubah otomatis saat pelanggan membuka link, memilih metode, dan menerbitkan QRIS/VA.
  4. Setelah pelanggan membayar, panel menampilkan **Lunas** dan data faktur dimuat ulang.
