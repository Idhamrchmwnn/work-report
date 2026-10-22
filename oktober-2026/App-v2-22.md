# 📝 Daily Work Report - Idham (2026-10-22)

---

## 📅 Laporan Harian - 22 Oktober 2026

---

## 🌿 Branch: `issue-462` — Pilihan Penerima Faktur Mengikuti Menu Pengguna

### 📌 Informasi Issue

- **Nomor Issue**: #462
- **Judul Issue**: Jenis penerima di form Tagihan Penjualan disamakan dengan menu Pengguna
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-462`)

### 📅 Rincian Commit

#### [924bac24] - resolve #462 - 22 Oktober 2026, 17:18

- **Ringkasan**:
  - Pilihan jenis penerima di form **Keuangan > Tagihan Penjualan** kini sama dengan menu **Pengguna**, dengan nama dan urutan yang sama: Pelanggan, Pelanggan Bisnis, Mitra Bisnis, Pelanggan Mitra.
  - Setiap jenis hanya mencari datanya sendiri. Sebelumnya, jenis "Pelanggan" juga memuat pelanggan mitra, dan jenis mitra mencampur mitra reseller dengan pelanggan bisnis.

- **Komponen yang Berubah & Penjelasannya**:

  **Frontend**

  - `frontend/src/components/shared/form/partyTypes.js` [NEW]
    - Daftar endpoint pencarian per jenis pihak dipindah ke sini dari `PartyPicker.jsx`, ditambah dua jenis baru padanan menu Pengguna:
      - `customerRegular`: `/customer/user-select-master`, pelanggan reguler saja.
      - `reseller`: `/partner/user-select`, mitra reseller saja.
    - Helper `getPartySearchUrl(type)` dan `getPartyTypeLabelKey(type)`. Label jenis yang punya padanan menu Pengguna memakai teks menu itu (`nav.users.*`), supaya namanya persis sama dengan sidebar.
  - `frontend/src/components/shared/form/partyTypes.test.js` [NEW]
    - Mengunci endpoint dan label tiap jenis penerima.
    - Memastikan jenis lama yang dipakai form lain (Transaksi, Top Up, dll.) tidak berubah.
  - `frontend/src/components/shared/form/PartyPicker.jsx`
    - Kini memakai `getPartySearchUrl` dan `getPartyTypeLabelKey` dari `partyTypes.js`; tidak ada lagi daftar endpoint di dalam komponen.
  - `frontend/src/app/pages/finance/invoices/InvoiceDrawer.jsx`
    - Jenis penerima diganti menjadi `RECIPIENT_TYPES` (`customerRegular`, `business`, `reseller`, `customerPartner`), mengikuti urutan menu Pengguna.
    - Pemetaan ke `party_type` backend ditambah: `customerRegular` menjadi `customer`, `reseller` menjadi `partner`.
    - Penerima yang dikirim ke daftar item faktur dinormalkan ke jenis backend, karena komponen item hanya mengenal `customer`/`partner`.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Label `finance.transaction.partyType.customerPartner` dan `.business` dihapus, karena kini memakai label menu Pengguna.

  **Backend — Test & Basis Pengetahuan**

  - `backend/test/integration/invoiceRecipientSelect.test.js` [NEW]
    - Pencarian "Pelanggan" hanya memuat pelanggan reguler, tanpa pelanggan mitra, blacklist, atau pasif.
    - "Pelanggan Mitra" hanya memuat pelanggan milik mitra.
    - "Mitra Bisnis" dan "Pelanggan Bisnis" tidak tercampur.
  - `backend/src/data/aiKnowledge/features.json`
    - Dibangkitkan ulang (`npm run gknow`) dengan isi yang sama seperti di #453: endpoint Pengaturan > Developer dan AI Agent.

---

## 🌿 Branch: `issue-460` — Pelaporan untuk Pelanggan Berhenti/Blacklist/Pasif, Filter & Urut Kolom Pelaporan, dan Partner API Baca-Saja

### 📌 Informasi Issue

- **Nomor Issue**: #460
- **Judul Issue**: Kolom Pelaporan bisa difilter dan diurutkan; pelaporan untuk pelanggan Berhenti Berlangganan, Blacklist, dan Pasif; pelanggan tersebut tampil baca-saja di Partner API
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-460`)

### 📅 Rincian Commit

#### [74b76c2a] - resolve #460 - 22 Oktober 2026, 16:02

- **Ringkasan**: Lanjutan fitur Pelaporan (#394/#407), terdiri dari empat bagian.
  1. **Kolom Pelaporan bisa difilter dan diurutkan** di tabel Pelanggan dan Bisnis. Nilainya turunan dari layanan, jadi filter dan urutannya diterjemahkan server.
  2. **Pelaporan di tabel Berhenti Berlangganan, Blacklist, dan Pasif:** kolom ringkasan, aksi massal, dan aksi baris.
  3. **Partner API:** pelanggan internal yang berhenti berlangganan, di-blacklist, atau pasif ikut tampil bila punya layanan terlaporkan, dengan penanda `subscription_state`. Yang berhenti atau blacklist **hanya bisa dibaca**.
  4. **Perbaikan:**
     - Pelanggan blacklist yang belum dihapus kini muncul di tabel Blacklist.
     - Penanda datatable khusus server tidak lagi bisa dikirim dari aplikasi eksternal.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Filter & Urut Kolom Pelaporan**

  - `backend/src/services/taxReport.service.js`
    - `applyTaxReportTableParams(ownerType, params)` mengambil filter dan urutan kolom `tax_report_status` dari permintaan tabel:
      - **Filter** diubah menjadi klausa `_id` di `params.find`.
      - **Urutan** diubah menjadi `sortBuckets`: kelompok berurutan `all`, `partial`, `none`, `no_service`, lalu `not_applicable` (pelanggan mitra) selalu di akhir.
      - Urutan ini hanya berlaku sebagai urutan **pertama**; sebagai urutan kedua (Shift + klik) diabaikan.
    - Indeks status per pemilik (`buildTaxReportStatusIndex`) membaca seluruh layanan, jadi hasilnya di-cache 60 detik di memori.
      - Permintaan bersamaan berbagi satu hitungan, dan hitungan yang gagal tidak disimpan.
      - Cache dibuang (`invalidateTaxReportStatusIndex`) setiap kali pelaporan diubah, termasuk saat akun baru dibuat lewat Partner API.
      - Badge per baris tidak memakai cache, jadi selalu akurat.
    - Akun PPPoE yang tertaut ke dedicated mana pun (termasuk milik bisnis lain) kini tidak ikut dihitung sebagai layanan bisnis, baik di badge maupun filter.
    - Aksi massal pelanggan kini memakai `findWithDeleted`, jadi juga berlaku untuk pelanggan yang sudah soft-deleted (Berhenti Berlangganan/Blacklist).
    - Fungsi baru untuk Partner API:
      - `isInternalCustomerVisible`: pelanggan internal punya layanan terlaporkan.
      - `isAuthenticationOwnerArchived`: pemilik akun radius berhenti atau blacklist. Pemilik dibaca dari `customer` mentah, bukan populate, karena populate tanpa data terhapus menghasilkan `null`.
  - `backend/src/utils/data-table.js`
    - Dukungan `sortBuckets`: hasil dibagi ke beberapa kelompok berurutan dan halaman diambil lintas kelompok, sehingga paginasi tetap konsisten tanpa baris ganda. Di dalam kelompok berlaku urutan biasa, atau `_id` menurun.
    - Opsi server baru `includeDeleted`: memuat dokumen terhapus maupun yang belum terhapus. `params.withDeleted` tetap berarti hanya yang terhapus.
    - Kode baca/hitung dirapikan menjadi satu jalur (`countDocs`/`fetchDocs`), sekaligus memperbaiki hitungan total pada mode terhapus.
  - `backend/src/services/customer.service.js`
    - Helper `attachTaxReportSummary` menempelkan `tax_report_status` per baris. Pelanggan mitra diberi `not_applicable`.
    - Datatable Pelanggan, **Berhenti Berlangganan**, **Blacklist**, dan **Pasif** kini mendukung filter/urut Pelaporan dan membawa ringkasannya.
    - **Perbaikan Blacklist:** mem-blacklist tidak menghapus pelanggan, tetapi tabel Blacklist sebelumnya hanya memuat yang terhapus, sehingga pelanggan blacklist yang belum dihapus tidak pernah muncul di tabel mana pun. Kini memakai `includeDeleted`.
    - Partner API:
      - Daftar pelanggan membawa `subscription_state`. Field sumbernya dibaca terpisah dan tidak ikut pencarian, supaya alasan pasif tidak bisa ditebak lewat kata kunci.
      - Statistik menambah kategori `pasif`, dan untuk admin juga `blacklist` dan `unsubscribed`.
      - Detail bisa membaca data terhapus.
    - `catch` yang sebelumnya diam kini mencatat `logger.error`.
  - `backend/src/services/business.service.js`
    - Datatable Bisnis mendukung filter/urut Pelaporan.
  - `backend/src/utils/customerSubscriptionState.js` [NEW]
    - `getCustomerSubscriptionState`: menghasilkan `blacklist` (didahulukan), `unsubscribed`, `pasif`, atau `active`, beserta daftar field sumbernya.

  **Backend — Partner API**

  - `backend/src/controllers/partnerApiCustomer.controller.js`
    - Filter baca admin (`READABLE_ADMIN_CUSTOMER`) memuat pelanggan internal yang berhenti/blacklist (bila terlaporkan), sedangkan pelanggan mitra yang berhenti/blacklist tetap tersembunyi. Filter ini dibungkus `$and` agar tidak bentrok dengan `$or` pemanggil.
    - Daftar admin memakai `includeDeleted`; nilai `withDeleted` kiriman klien dibuang.
    - Detail membawa `subscription_state`.
    - Jalur tulis (ubah, hapus, ganti status, dokumen, avatar) pada pelanggan berhenti/blacklist dijawab **422** "hanya bisa dibaca" (`throwCustomerNotWritable`). Yang memang tidak terlihat tetap dijawab 404. Pelanggan pasif tetap bisa diubah.
  - `backend/src/controllers/partnerApiRadius.controller.js`
    - Akun radius tidak bisa dibuat untuk, atau dipindah ke, pelanggan berhenti/blacklist (422).
    - Pelanggan berhenti/blacklist yang tersembunyi dijawab sama dengan ID yang tidak ada, supaya keberadaannya tidak terkonfirmasi.
    - Ubah, hapus, dan ganti status akun milik pelanggan berhenti/blacklist ditolak 422 (`assertAuthenticationWritable`).
  - `backend/src/middlewares/partnerApiBody.middleware.js` [NEW]
    - `stripServerOnlyBodyKeys` membuang `withDeleted` dan `allowDiskUse` dari body semua route Partner API, supaya aplikasi eksternal tidak bisa melihat data terhapus atau memaksa query berat.
  - `backend/src/routes/partnerApi.route.js`
    - Memasang middleware di atas.
    - Swagger diperbarui untuk `subscription_state`, statistik baru, dan respons 422.
  - `backend/src/routes/customer.route.js`, `backend/src/routes/business.route.js`
    - Swagger parameter `sorting` dan `columnFilters` menjelaskan filter/urutan kolom `tax_report_status`.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `customer.readOnlyPartnerApi`.

  **Backend — Test**

  - `backend/test/integration/customerTaxReported.test.js`
    - Filter dropdown, urutan naik/turun lintas halaman tanpa baris ganda, urutan kedua diabaikan, dan cache dibuang saat aksi massal.
    - Ringkasan dan aksi massal di tabel Berhenti Berlangganan, Blacklist, dan Pasif.
  - `backend/test/integration/businessTaxReported.test.js`
    - PPPoE dedicated bisnis lain tidak dihitung.
    - Filter dan urutan tabel bisnis; bisnis milik reseller di akhir.
  - `backend/test/integration/partnerApiCustomer.archived.test.js` [NEW]
    - Daftar memuat keempat keadaan dengan `subscription_state`.
    - Yang tersembunyi tetap tersembunyi, dan detail terbaca.
    - Jalur tulis 422; body `withDeleted` dibuang.
    - Tabel Blacklist memuat yang belum maupun sudah dihapus.
    - Akun radius terkunci, statistik benar, dan alasan pasif tidak bisa ditebak lewat pencarian.

  **Frontend**

  - `frontend/src/app/pages/users/customer/schema/columns.jsx`, `frontend/src/app/pages/users/business/schema/columns.jsx`
    - Kolom Pelaporan kini punya filter dropdown status (`taxReportStatusOptions`) dan bisa diurutkan.
  - `frontend/src/app/pages/users/unsubscribe/schema/columns.jsx`, `frontend/src/app/pages/users/blacklist/schema/columns.jsx`, `frontend/src/app/pages/users/pasif/schema/columns.jsx`
    - Kolom pilih baris dan kolom **Pelaporan** (badge, filter, urutan) ditambahkan.
    - Aksi baris memakai `TaxReportRowActions`.
    - Di Berhenti Berlangganan, tautan detail dan hak bacanya dibedakan untuk pelanggan reguler (`customer.read`) dan pelanggan mitra (`customerPartner.read`).
  - `frontend/src/app/pages/users/unsubscribe/index.jsx`, `frontend/src/app/pages/users/blacklist/index.jsx`, `frontend/src/app/pages/users/pasif/index.jsx`
    - Aksi massal **Pelaporan** (`CustomerTaxReportBatchModal`) ditambahkan, hanya untuk pemegang `customer.changeSensitive`.

  **Dokumentasi & Basis Pengetahuan**

  - `AGENTS.md`
    - Pola baru Bab 9: filter dan urutan kolom turunan di datatable (klausa `_id` + `sortBuckets`, hanya urutan pertama, cache singkat).
    - Pola Partner API diperbarui: pelanggan berhenti/blacklist/pasif dan `subscription_state`.
  - `backend/src/data/aiKnowledge/features.json`
    - Dibangkitkan ulang (`npm run gknow`).

---

## 🌿 Branch: `issue-458` — Link Pembayaran Faktur Bisa Diklik

### 📌 Informasi Issue

- **Nomor Issue**: #458
- **Judul Issue**: Link pembayaran di bawah QR faktur bisa diklik, termasuk di PDF faktur publik
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-458`)

### 📅 Rincian Commit

#### [1357e158] - resolve #458 - 22 Oktober 2026, 12:14

- **Ringkasan**:
  - Perubahan yang di laporan kemarin ([App-v2-21.md](App-v2-21.md)) masih tercatat "Belum di-commit" kini di-commit sebagai issue tersendiri, #458. Isinya sama persis.
  - Link pembayaran di bawah QR faktur kini bisa diklik, baik di halaman faktur maupun di PDF faktur publik yang diunduh.

- **Komponen yang Berubah & Penjelasannya**:

  - `frontend/src/components/shared/ExternalLink.jsx` [NEW]
    - Tautan ke alamat luar aplikasi: tab baru, `rel="noopener noreferrer"`.
  - `frontend/src/app/pages/finance/invoices/InvoiceDocument.jsx`
    - Link pembayaran di bawah QR "Scan untuk bayar online" dibungkus `ExternalLink`.
  - `frontend/src/app/pages/public/PublicInvoiceDocument.jsx`
    - Setiap `<a href>` di dokumen diberi anotasi link jsPDF di posisi yang sama, supaya tetap bisa diklik walau PDF berupa gambar.
  - `backend/src/data/aiKnowledge/features.json`
    - Dibangkitkan ulang (`npm run gknow`).

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #462 | Penerima faktur mengikuti menu Pengguna | Jenis penerima Tagihan Penjualan sama dengan menu Pengguna dan tidak lagi tercampur |
| #460 | Pelaporan: filter/urut, pelanggan arsip, Partner API baca-saja | Kolom Pelaporan bisa difilter/diurutkan; pelaporan bisa diatur dari tabel Berhenti Berlangganan/Blacklist/Pasif; pelanggan berhenti/blacklist tampil baca-saja di Partner API |
| #458 | Link pembayaran faktur bisa diklik | Link pembayaran di faktur dan PDF faktur publik bisa diklik |

### Kemampuan Baru Pengguna/Admin

- Memfilter dan mengurutkan kolom **Pelaporan** di tabel Pelanggan, Bisnis, Berhenti Berlangganan, Blacklist, dan Pasif.
- Mengatur pelaporan (aksi massal dan aksi baris) langsung dari tabel Berhenti Berlangganan, Blacklist, dan Pasif.
- Memilih penerima Tagihan Penjualan dengan kategori yang sama dengan menu Pengguna.
- Aplikasi eksternal melihat status langganan pelanggan (`subscription_state`) dan bisa membaca pelanggan berhenti/blacklist yang terlaporkan.

### Bug Fix / Solusi Masalah

- Pelanggan yang di-blacklist tetapi belum dihapus kini muncul di tabel Blacklist. Sebelumnya tidak muncul di tabel mana pun.
- Pilihan "Pelanggan" di form faktur tidak lagi memuat pelanggan mitra, dan mitra tidak lagi tercampur pelanggan bisnis.
- Akun PPPoE milik dedicated bisnis lain tidak lagi dihitung sebagai layanan sebuah bisnis.
- Aplikasi eksternal tidak bisa lagi mengirim `withDeleted`/`allowDiskUse` untuk melihat data terhapus atau memaksa query berat.
- Data pelanggan berhenti/blacklist dan akun radiusnya tidak bisa diubah lewat Partner API.

### Menu/Fitur Baru

- Filter dan urutan kolom **Pelaporan**.
- Aksi massal **Pelaporan** di tabel **Pengguna > Berhenti Berlangganan / Blacklist / Pasif**.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur (Filter & Urut Pelaporan)**: Kolom Pelaporan menunjukkan berapa banyak layanan pelanggan/bisnis yang sudah terlaporkan (Semua, Sebagian, Belum, Tanpa layanan). Kolom ini kini bisa disaring per status dan diurutkan. Pelanggan mitra (tidak memakai pelaporan) selalu tampil paling akhir saat diurutkan.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Pengguna > Pelanggan** (atau Bisnis, Berhenti Berlangganan, Blacklist, Pasif).
  2. Pada kolom **Pelaporan**, pilih status di filter dropdown, misalnya "Belum" dan "Tanpa layanan".
  3. Klik judul kolom **Pelaporan** untuk mengurutkan.
  4. Centang beberapa baris, lalu pilih aksi massal **Pelaporan** untuk menandai atau membatalkan pelaporan semua layanannya sekaligus.
