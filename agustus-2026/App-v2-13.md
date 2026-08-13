# 📝 Daily Work Report - Idham (2026-08-13)

---

## 📅 Laporan Harian - 13 Agustus 2026

---

## 🌿 Branch: `master` — Customer SDN: Alur Persetujuan, Notifikasi & Dokumen Publik

### 📌 Informasi Issue

- **Nomor Issue**: Tidak tercantum di GitHub Issues (dikerjakan langsung di branch `master`, tanpa branch `issue-XXX` terpisah)
- **Judul Issue**: Melengkapi dokumen **Customer SDN** — *Service Delivery Notification* — dengan alur persetujuan internal, notifikasi Telegram, tampilan status, dan halaman publik ber-token untuk partner
- **Status Branch**: `Sudah di-commit ke master` (local, `ahead of origin/master` — belum di-push)

### 📅 Rincian Commit

#### [c3fca9e] - "save sdn" - Selasa, 13 Agustus 2026 14:43

- **Komponen yang Berubah (bagian alur persetujuan, notifikasi & publik)**:
  - `backend/src/controllers/publicCustomerSDN.controller.js` [NEW]
  - `backend/src/routes/public.route.js`
  - `backend/src/routes/files.route.js`
  - `backend/src/utils/telegram.js`
  - `backend/src/locales/id/translation.json`
  - `backend/src/locales/en/translation.json`
  - `frontend/src/app/pages/users/customerSDN/CustomerSDNReviewDrawer.jsx` [NEW]
  - `frontend/src/app/pages/users/customerSDN/CustomerSDNDocumentPreview.jsx` [NEW]
  - `frontend/src/app/pages/public/ReviewCustomerSDNPage.jsx` [NEW]
  - `frontend/src/app/pages/public/PublicCustomerSDNDocument.jsx` [NEW]
  - `frontend/src/components/shared/DocumentPreviewModal.jsx`
  - `frontend/src/components/shared/table/rows.jsx`
  - `frontend/src/app/router/protected.jsx`
  - `frontend/src/app/router/public.jsx`
  - `frontend/src/constants/privilegeDescriptions.en.json`
  - `frontend/src/constants/privilegeDescriptions.id.json`
  - `frontend/src/i18n/locales/id/translations.json`
  - `frontend/src/i18n/locales/en/translations.json`

---

## 🖥️ BACKEND

### Ringkasan

Melengkapi dokumen Customer SDN dengan siklus hidup penuh: pengajuan untuk disetujui, notifikasi otomatis ke Telegram, persetujuan/penolakan internal, penandaan terkirim ke partner, serta akses dokumen publik lewat token unik tanpa login.

- **`backend/src/services/customerSDN.service.js`**
  - Menambahkan fungsi `approveCustomerSDN` (mengisi field `approval` dan `approved_at`), `rejectCustomerSDN` (soft-delete dokumen yang ditolak), dan `findCustomerSDNByToken` (pencarian dokumen berdasarkan `share_token`, dipakai halaman publik).

- **`backend/src/controllers/customerSDN.controller.js`**
  - `submitSDN`: mengajukan dokumen untuk persetujuan internal, mengirim notifikasi Telegram ke channel approval, ditolak jika dokumen sudah disetujui/selesai.
  - `requestSDNPreview`: memungkinkan admin meminta pratinjau dikirim ke Telegram admin tertentu (by `adminId`) atau ke channel approval default, untuk review cepat sebelum submit resmi.
  - `approveSDN` / `rejectSDN`: mengubah status persetujuan internal dokumen.
  - `sendSDN`: menandai dokumen sebagai `sent` ke partner — mensyaratkan dokumen sudah disetujui dan belum pernah dikirim sebelumnya.

- **`backend/src/routes/customerSDN.route.js`**
  - Mendaftarkan endpoint lanjutan: `PATCH /customer-sdn/submit/:id`, `POST /customer-sdn/request-preview`, `PATCH /customer-sdn/approve/:id`, `PATCH /customer-sdn/reject/:id`, `POST /customer-sdn/send/:id`, masing-masing dilindungi privilege `customerSDN.update`/`customerSDN.changeStatus`, dengan dokumentasi Swagger lengkap.

- **`backend/src/controllers/publicCustomerSDN.controller.js`** [NEW]
  - Menyediakan dua endpoint publik yang **token-gated** (tanpa login, hanya lewat `share_token` unik): `getSDNByToken` untuk mengambil data dokumen (di-mapping ke DTO terbatas `toPublicSDNDTO` yang hanya mengekspos field aman ditampilkan ke publik, menyembunyikan data internal seperti `created_by`), dan `getSDNTopologyByToken` untuk menyajikan gambar topologi sebagai stream binary dengan `Content-Type` yang sesuai.
  - Dokumen hanya bisa diakses publik jika sudah melewati tahap `approval` internal — mencegah partner melihat draft yang belum final.

- **`backend/src/routes/public.route.js`**
  - Menambahkan dua endpoint publik: `GET /public-docs/customer-sdn/:token` (detail dokumen) dan `GET /public-docs/customer-sdn/:token/topology` (gambar topologi), tanpa autentikasi.

- **`backend/src/routes/files.route.js`**
  - Menambahkan endpoint `GET /file/customer-sdn/:name` untuk menyajikan gambar topologi kepada admin yang sudah login, dilindungi privilege `customerSDN.read`.

- **`backend/src/utils/telegram.js`**
  - Menambahkan fungsi `TelegramNotifCustomerSDNSubmit` yang mengirim notifikasi ke Telegram (channel approval atau chat admin spesifik) saat SDN diajukan/diminta pratinjau, memuat ringkasan proyek, nama partner, tanggal SDN, dan tautan langsung ke halaman review internal (`/review/customer-sdn/:id`), mengikuti pola penamaan dan format pesan yang sama dengan notifikasi PO/SO/PKS yang sudah ada.

- **`backend/src/locales/id/translation.json`** & **`backend/src/locales/en/translation.json`**
  - Menambahkan pesan-pesan terkait alur persetujuan: `cannotSubmit`, `submitted`, `previewRequestSent`, `cannotApprove`, `cannotReject`, `notApproved`, `alreadySent`, `sent`.

### Dampak Backend

Dokumen SDN kini punya siklus hidup lengkap: diajukan → notifikasi Telegram → disetujui/ditolak PIC internal → dikirim ke partner dengan link publik ber-token. Endpoint publik memastikan partner hanya bisa melihat dokumen yang sudah final disetujui, tanpa perlu akun login.

---

## 💻 FRONTEND

### Ringkasan

Membangun antarmuka review/persetujuan untuk admin, tampilan status dokumen, serta halaman publik yang bisa diakses partner untuk melihat dan mencetak dokumen SDN yang sudah dikirim.

- **`frontend/src/app/pages/users/customerSDN/CustomerSDNReviewDrawer.jsx`** [NEW] (516 baris)
  - Drawer review untuk admin approver: menampilkan detail lengkap dokumen (fetch ulang by nomor SDN saat drawer dibuka), tombol untuk mengirim permintaan pratinjau ke Telegram (ke admin tertentu via `Combobox` atau ke channel default), serta aksi **Setujui**/**Tolak** dengan modal konfirmasi (`ConfirmModal`), dibatasi oleh privilege `customerSDN.changeStatus`.

- **`frontend/src/app/pages/users/customerSDN/CustomerSDNDocumentPreview.jsx`** [NEW] (249 baris)
  - Komponen presentasional yang merender tampilan dokumen SDN dalam format surat resmi (logo perusahaan, salam pembuka, detail proyek/layanan, tabel A-End/B-End, catatan penerimaan layanan, tanda tangan digital pihak internal) — dipakai bersama oleh modal preview admin maupun halaman publik agar tampilan konsisten.

- **`frontend/src/app/pages/public/ReviewCustomerSDNPage.jsx`** [NEW] (295 baris)
  - Halaman internal (butuh login) yang diakses lewat tautan notifikasi Telegram (`/review/customer-sdn/:id`), memungkinkan admin approver langsung membuka detail & melakukan approve/reject dari luar halaman daftar dokumen.

- **`frontend/src/app/pages/public/PublicCustomerSDNDocument.jsx`** [NEW] (136 baris)
  - Halaman publik tanpa login yang diakses partner lewat link `share_token` (`/p/customer-sdn/:token`), menampilkan dokumen SDN final memakai komponen `CustomerSDNDocumentPreview`, dengan tombol cetak (`useReactToPrint`) dan penanganan state loading/error/dokumen tidak ditemukan.

- **`frontend/src/components/shared/DocumentPreviewModal.jsx`**
  - Modal preview dokumen generik (dipakai lintas modul: PO, SO, PKS, Quotation, Work Order) diperluas untuk mendukung `type="customer-sdn"`: mengambil data terbaru dari endpoint `/customer-sdn/view/:id`, merender lewat `CustomerSDNDocumentPreview`, serta menyesuaikan judul preview dan nama file saat dicetak/diunduh.

- **`frontend/src/components/shared/table/rows.jsx`**
  - Menambahkan komponen `CustomerSDNStatusCell` (77 baris) — sel status pada tabel SDN yang menampilkan tiga kondisi visual: **terkirim** (ikon centang hijau + label "Terkirim"), **disetujui PIC** (ikon centang hijau + avatar admin yang menyetujui), atau **menunggu persetujuan** (ikon jam pasir kuning, atau tombol "Review Dokumen" bagi yang punya privilege `customerSDN.changeStatus`). Kolom status pada tabel SDN kini terhubung ke drawer review yang baru dibuat, dan halaman Document menambahkan handler approve/reject/reload serta modal preview khusus tab SDN.

- **`frontend/src/app/router/protected.jsx`** & **`frontend/src/app/router/public.jsx`**
  - Mendaftarkan dua rute baru: `review/customer-sdn/:id` (halaman internal, lazy-loaded) dan `p/customer-sdn/:token` (halaman publik, lazy-loaded).

- **`frontend/src/constants/privilegeDescriptions.en.json`** & **`.id.json`**
  - Menambahkan deskripsi human-readable untuk privilege `customerSDN.changeStatus` dan privilege SDN lainnya agar tampil rapi di halaman pengaturan hak akses admin.

- **`frontend/src/i18n/locales/id/translations.json`** & **`en/translations.json`**
  - Menambahkan string status (`draft`, `approvedPic`, `sent`), riwayat dokumen (dibuat/disetujui/terkirim), serta teks lengkap surat notifikasi publik (salam, ucapan terima kasih, penjelasan "deemed delivered" jika tidak ada komplain dalam 7 hari, catatan tim Customer Care 7x24 jam, dan penutup) yang dirender di halaman publik `PublicCustomerSDNDocument.jsx`.

### Dampak Frontend

Admin kini punya antarmuka lengkap untuk meninjau, menyetujui, dan mengirim dokumen SDN ke partner, dengan notifikasi Telegram otomatis di setiap tahap persetujuan. Partner mendapat halaman publik khusus untuk melihat/mencetak dokumen tersebut tanpa perlu login.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| — (master) | Alur Persetujuan, Notifikasi & Dokumen Publik Customer SDN | Siklus persetujuan internal via Telegram, pengiriman ke partner, dan halaman publik ber-token untuk dokumen SDN |

### Kemampuan Baru Pengguna/Admin

- Admin approver menerima notifikasi Telegram saat SDN diajukan, dapat melihat pratinjau, lalu menyetujui atau menolak langsung dari Telegram (link ke halaman review) maupun dari tabel daftar dokumen.
- Setelah disetujui, admin dapat menandai dokumen sebagai "Terkirim" ke partner, yang menghasilkan link publik unik (`share_token`) untuk dibagikan.
- Partner dapat membuka link publik tanpa login untuk melihat dan mencetak dokumen SDN resmi berisi detail layanan, lokasi A-End/B-End, gambar topologi, dan syarat "deemed delivered" (7 hari kalender untuk komplain).

### Bug Fix / Solusi Masalah

- Tidak ada perbaikan bug pada sesi ini — murni pengembangan fitur baru.

### Menu/Fitur Baru

- Drawer review/persetujuan dokumen SDN dengan integrasi notifikasi Telegram.
- Halaman publik pencetakan dokumen SDN di `/p/customer-sdn/:token`.
- Halaman review internal via notifikasi Telegram di `/review/customer-sdn/:id`.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Setelah dokumen SDN diajukan, alur persetujuan berjalan lewat notifikasi Telegram ke admin approval. Berbeda dari PKS atau Customer SO yang membutuhkan tanda tangan digital dua pihak, SDN cukup disetujui secara internal (PIC) lalu dikirimkan sebagai link ke partner; penerimaan layanan dianggap sah kecuali partner mengajukan komplain "Unmatched Items" dalam 7 hari kalender sejak tanggal SDN.
- **Langkah Penggunaan (Tutorial)**:
  1. Pada dokumen SDN berstatus draft, klik **Review Dokumen** untuk mengirim permintaan persetujuan — notifikasi otomatis terkirim ke Telegram admin approval.
  2. Admin approver membuka drawer/halaman review, dapat meminta pratinjau tambahan ke Telegram admin tertentu, lalu klik **Setujui** atau **Tolak**.
  3. Setelah disetujui, klik **Kirim ke Partner** untuk menandai dokumen `sent` dan mendapatkan link publik; salin link tersebut dan bagikan ke partner.
  4. Partner membuka link publik untuk melihat/mencetak dokumen SDN tanpa perlu login.
