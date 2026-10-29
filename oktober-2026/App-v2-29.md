# 📝 Daily Work Report - Idham (2026-10-29)

---

## 📅 Laporan Harian - 29 Oktober 2026

---

## 🌿 Branch: `issue-498` — Tiket Opsional untuk Pelanggan Bisnis, Mitra Bisnis, dan Layanan Dedicated

### 📌 Informasi Issue

- **Nomor Issue**: #498
- **Judul Issue**: Kolom tiket instalasi bisa dibuat opsional untuk form Pelanggan Bisnis, Mitra Bisnis, dan Layanan Dedicated Internet
- **Status Branch**: `Belum di-merge`. Dua commit `save #498` sudah di-squash menjadi satu commit `resolve #498` di atas `master` terbaru, dan sudah ada di `origin/issue-498`. Cadangan sebelum squash ada di branch `backup/issue-498-pre-squash`.

### 📅 Rincian Commit

#### [717d3d02] - resolve #498 - 29 Oktober 2026, 15:42

- **Ringkasan**:
  - Sebelumnya tiket opsional hanya tersedia untuk form Pelanggan dan Broadband. Kini tersedia juga tiga sakelar baru di **Pengaturan > Developer > Pengaturan Khusus**:
    - `disable_business_ticket_mandatory`: form Pelanggan Bisnis.
    - `disable_partner_ticket_mandatory`: form Mitra Bisnis.
    - `disable_dedicated_ticket_mandatory`: form Layanan Dedicated Internet.
  - Saat sakelar aktif, data baru boleh disimpan tanpa tiket. Bila tiket tetap diisi, tiket tetap divalidasi dan ditautkan seperti alur normal.
  - Ikut diperbaiki dua celah lama:
    - Tanpa penjaga `data.ticket`, query `ticket_id: undefined` dibuang Mongoose sehingga data baru bisa tertaut ke tiket instalasi sembarang.
    - Tiket berupa objek operator Mongo (mis. `{ "$ne": null }`) dari body bisa dicocokkan sebagai query sungguhan. Kini ditolak 400 di semua form yang memakai tiket.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/utils/ticketInput.js` [NEW]
    - Helper `isValidTicketInput`: nilai tiket hanya boleh kosong, string, atau angka. Objek dan array ditolak.

  - `backend/src/controllers/business.controller.js` dan `backend/src/controllers/partner.controller.js`
    - Endpoint baru `getBusinessCreateConfig` / `getPartnerCreateConfig` yang mengembalikan nilai sakelar sebagai boolean.
    - `createBusiness` / `createPartner`:
      - menolak tiket yang bukan string/angka (400);
      - tiket hanya dicari bila diisi, dan tetap wajib tiket instalasi yang belum selesai dan belum tertaut ke mitra/bisnis;
      - tanpa tiket hanya diterima bila sakelar aktif;
      - penautan tiket hanya dilakukan bila tiketnya benar-benar ditemukan.

  - `backend/src/controllers/productDedicatedInternet.controller.js`
    - Endpoint baru `getDedicatedInternetCreateConfig`.
    - `createDedicatedInternet`: tiket wajib kecuali sakelar aktif. Bila diisi, harus tiket instalasi yang belum selesai dan belum tertaut ke layanan dedicated lain (`dedicated: null`), sesuai daftar pilihan di form. Selain itu ditolak `ticket.notFoundOrUsed`.

  - `backend/src/controllers/customer.controller.js`, `customerPartner.controller.js`, `radiusAuthentication.controller.js`
    - Ditambah penjaga `isValidTicketInput` sebelum tiket dicari (pagar operator Mongo yang sama).

  - Routes (`business.route.js`, `partner.route.js`, `productDedicatedInternet.route.js`)
    - Route baru `GET /business/create-config`, `GET /partner/create-config`, dan `GET /product/dedicated-internet/create-config`. Masing-masing memakai privilege `*.create` yang sama dengan endpoint create-nya, jadi tidak ada key privilege baru.
    - Swagger lengkap untuk ketiga route baru.

  - Swagger create (`business`, `partner`, `customer`, `customerPartner`, `productDedicatedInternet`, `radiusAuthentication`)
    - `ticket` tidak lagi tercantum wajib di skema; keterangannya menjelaskan sakelar terkait.
    - Keterangan respons 400 diperinci.
    - Path dokumentasi dedicated dibetulkan menjadi `/api/v1/product/dedicated-internet/create`.

  - `backend/src/services/option.service.js` dan `backend/src/services/settingsBackup.registry.js`
    - Ketiga key sakelar baru didaftarkan sebagai pengaturan non-sensitif dan dimasukkan ke grup Backup & Restore Pengaturan. Tanpa itu nilainya tidak akan pernah tersimpan.

  - `backend/src/config/privilegeDictionary.json`
    - `apiEndpoints` untuk `business.create`, `partner.create`, dan `dedicatedInternet.create` ditambah endpoint `create-config`.

  - Test
    - `backend/test/integration/ticketOptional.create.test.js` [NEW], untuk Bisnis/Mitra dan Dedicated:
      - `create-config` mengembalikan boolean;
      - sakelar mati: tanpa tiket ditolak 400 dan tidak ada data tersimpan;
      - sakelar aktif: tanpa tiket berhasil dibuat;
      - sakelar aktif: tiket yang diisi tetap divalidasi;
      - tiket instalasi yang terbuka diterima dan ditautkan (dedicated);
      - tiket berupa objek operator Mongo ditolak 400.
    - `backend/test/integration/capacitySpeed.controller.test.js`: sakelar dedicated diaktifkan di `beforeEach` karena body uji tidak membawa tiket, sedangkan tiket dedicated kini wajib secara bawaan.

  **Frontend**

  - `frontend/src/app/pages/settings/sections/developer/SpecialSettingsTab.jsx`
    - Dirombak menjadi daftar konfigurasi `TICKET_TOGGLES` dengan satu komponen `TicketToggleCard`, sehingga tidak ada lagi kartu yang ditulis berulang.
    - Lima kartu sakelar: Pelanggan, Pelanggan Bisnis, Mitra Bisnis, Broadband, dan Dedicated. Tiap kartu berisi badge status (Wajib/Opsional) dan kotak keterangan fungsi sakelar.

  - Form create Mitra, Bisnis, dan Dedicated (`users/partner/create.jsx`, `users/business/create.jsx`, `services/dedicatedInternet/create.jsx`)
    - Memuat `.../create-config` saat form dibuka. Bila gagal dimuat, tiket tetap wajib sebagai pilihan aman.
    - Label kolom tiket diberi "(Opsional)" saat sakelar aktif.

  - Schema (`schema/createShema.js` ketiga form)
    - Fungsi baru (`createPartnerSchema`, dan padanannya untuk bisnis dan dedicated) yang melonggarkan kolom `ticket` bila tiket opsional.

  - `frontend/src/i18n/locales/{id,en}/translations.json`
    - Judul, deskripsi, dan keterangan mode untuk tiga sakelar baru (id dan en).

---

## 🌿 Branch: `issue-497` — Tulis Faktur Pelanggan Mitra/POP dari Partner API

### 📌 Informasi Issue

- **Nomor Issue**: #497
- **Judul Issue**: Partner API: buat, ubah, dan batalkan faktur pelanggan mitra/POP, serta penahanan tagihan otomatis untuk akun mitra yang menunggak
- **Status Branch**: `Belum di-merge`. Satu commit `resolve #497` di atas `master` terbaru, sudah ada di `origin/issue-497`.

### 📅 Rincian Commit

#### [23ab5d25] - resolve #497 - 29 Oktober 2026, 16:42

- **Ringkasan**:
  - Endpoint baru di Partner API untuk aplikasi mitra:
    - `POST /p-api/v1/customer-invoices/create`: buat faktur pelanggan mitra/POP.
    - `PATCH /p-api/v1/customer-invoices/update/:id`: ubah faktur.
    - `POST /p-api/v1/customer-invoices/cancel/:id`: batalkan faktur yang belum dibayar.
  - Hanya untuk **pelanggan mitra**. Sesi admin wajib menyebut `partner_id` reseller aktif yang tampil di portal; faktur pelanggan internal tidak bisa ditulis dari p-api.
  - Semua penulisan memakai service faktur admin yang sudah ada (`createNewInvoice`, `updateInvoiceDetail`, `cancelInvoice`). Total, pajak, jurnal, dan aturan kunci field moneter tetap dihitung dan ditegakkan server.
  - Faktur yang sudah menerima pembayaran tidak bisa dibatalkan dari p-api (422), karena membatalkannya berarti membalik kas, dan itu wewenang admin.
  - Tagihan otomatis untuk akun milik mitra kini **ditahan** selama pelanggannya masih punya tagihan yang lewat jatuh tempo. Akun internal tidak terpengaruh.
  - Mitra bisa menyalakan/mematikan tagihan otomatis akun radius lewat update p-api.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Partner API**

  - `backend/src/controllers/partnerApiInvoice.controller.js`
    - Handler create, update, dan cancel faktur pelanggan mitra.
    - Cakupan ditentukan `resolveCustomerInvoiceWriteScope`:
      - sesi mitra hanya pelanggannya sendiri;
      - sesi admin wajib `partner_id` dalam daftar-putih reseller aktif ber-`show_in_portal`. Partner bisnis, mitra tersembunyi, atau mitra terhapus dijawab 404.
    - Pelanggan mitra lain atau pelanggan internal ditolak 400. Faktur milik mitra lain dijawab 404 saat diubah/dibatalkan.
    - Pelanggan yang berhenti berlangganan atau di-blacklist hanya bisa dibaca (422).
    - Akun radius yang dikaitkan harus milik pelanggan faktur tersebut.
    - `client_ref` diberi awalan pemilik (`scopePartnerApiClientRef`). Kuncinya unik global, jadi tanpa awalan, mitra bisa "mengambil" faktur mitra lain lewat kunci yang sama.

  - `backend/src/utils/partnerApiInvoiceInput.js` [NEW]
    - Daftar-putih field dan cek tipe body. Tanggal terbit selalu hari ini.
    - `deductions` dan data Faktur Pajak tidak diterima dari p-api.
    - ID (`customer_id`, `authentication_id`) hanya string/angka, sehingga objek operator Mongo ditolak 400.

  - `backend/src/controllers/partnerApiRadius.controller.js`
    - `updatePartnerAppRadiusUser` menerima `auto_invoice` dengan kosakata yang sama seperti create (`monthly`/`onlimit`/`active_date`; selain itu berarti mati). Nilai ini diterapkan setelah `cleanFormData`, karena nilai mati (`false`/`0`) ikut terbuang oleh `cleanFormData`.

  - `backend/src/routes/partnerApi.route.js`
    - Tiga route baru beserta Swagger lengkap (body, aturan, dan kode error).

  **Backend — Faktur & Tagihan Otomatis**

  - `backend/src/services/financeInvoice.service.js`
    - `cancelInvoice` menerima opsi `unpaidOnly`. Syarat `status: 'unpaid'` dan `paid_total` 0 ikut **klaim atomik**, sehingga pembayaran yang masuk bersamaan tidak ikut terbalik.
    - Bila klaim gagal karena faktur sudah dibayar, error yang dilempar adalah `INVOICE_HAS_PAYMENT`, bukan pesan "sudah dibatalkan" yang menyesatkan.

  - `backend/src/services/financeAutoInvoice.service.js`
    - Fungsi baru `hasOverdueUnpaidInvoice`. Yang dihitung: tagihan `unpaid` akun itu sendiri plus tagihan manual pelanggan tanpa akun, yang jatuh temponya **sebelum** hari ini (jatuh tempo hari ini belum dihitung). Tunggakan akun lain milik pelanggan yang sama tidak menahan akun ini.
    - `generateAutoInvoices`: akun ber-`ref_partner` yang menunggak dilewati dan dihitung di `skipped_overdue`. Bulan yang tertahan tidak dikejar.

  - `backend/src/utils/finance-error.js`
    - Kode baru `INVOICE_HAS_PAYMENT` (HTTP 422).

  - `backend/src/locales/{id,en}/translation.json`
    - Key baru `finance.invoice.cancelHasPayment` (id dan en).

  - `AGENTS.md`
    - Pola baru di Bab 9: aturan tulis faktur pelanggan mitra dari Partner API dan penahanan tagihan otomatis akun mitra.

  **Test**

  - `backend/test/integration/partnerApiInvoice.write.test.js` [NEW], 13 skenario:
    - create oleh mitra: total dihitung server, dijurnal, dan `pid` milik mitra;
    - penolakan pelanggan lain/internal dan objek operator;
    - kepemilikan akun radius;
    - idempotensi `client_ref` per mitra;
    - aturan `partner_id` untuk admin;
    - pelanggan berhenti berlangganan/blacklist;
    - aturan ubah sebelum dan sesudah dijurnal;
    - 404 untuk faktur mitra lain;
    - batal faktur belum dibayar (jurnal penerbitan dibalik) dan penolakan 422 untuk faktur yang sudah dibayar.
  - `backend/test/integration/financeAutoInvoice.partnerOverdue.test.js` [NEW]:
    - tunggakan akun dan tagihan manual menahan tagihan otomatis;
    - tetap terbit bila tunggakan sudah dibayar/dibatalkan, belum jatuh tempo, atau jatuh tempo hari ini;
    - tunggakan akun lain tidak menahan;
    - akun internal tidak terpengaruh.
  - `backend/test/integration/partnerApiRadius.test.js`
    - Mitra bisa menyalakan dan mematikan tagihan otomatis lewat update.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #498 | `717d3d02` resolve #498 | Tiket opsional untuk Pelanggan Bisnis, Mitra Bisnis, dan Dedicated, plus pagar input tiket di semua form create |
| #497 | `23ab5d25` resolve #497 | Partner API: buat/ubah/batal faktur pelanggan mitra/POP, dan penahanan tagihan otomatis akun mitra yang menunggak |

> Catatan: `resolve #493` (pengurang pembayaran faktur manual, dikerjakan 28 Oktober) sudah di-merge ke `master` pagi ini.
