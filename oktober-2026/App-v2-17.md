# 📝 Daily Work Report - Idham (2026-10-17)

---

## 📅 Laporan Harian - 17 Oktober 2026

---

## 🌿 Branch: `issue-437` — Pop-up Tangguhkan/Lanjutkan Tiket dengan Pesan Cepat & Perbaikan Terkait

### 📌 Informasi Issue

- **Nomor Issue**: #437
- **Judul Issue**: Tangguhkan/lanjutkan tiket lewat pop-up beralasan dengan pesan cepat, serta perbaikan nama hak akses, kapitalisasi nama, dan pencarian tiket di Telegram Mini App
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-437`)

### 📅 Rincian Commit

#### [bf1b984e] - resolve #437 - 17 Oktober 2026, 19:28

- **Ringkasan**: Commit ini memuat empat perubahan.
  1. **Tangguhkan/lanjutkan tiket**: kini selalu lewat pop-up yang meminta alasan, dengan pilihan **pesan cepat** yang bisa disimpan dan dipakai ulang. Pesan biasa tidak lagi melanjutkan tiket yang sedang ditangguhkan secara diam-diam. Dulu hal ini merusak perhitungan durasi pending/MTTR.
  2. **Nama hak akses** kini unik tanpa membedakan huruf besar/kecil.
  3. **Kapitalisasi nama otomatis** tidak lagi merusak singkatan, misalnya "PT Ringnet" tidak menjadi "Pt Ringnet".
  4. **Pencarian tiket di Telegram Mini App** kini juga mencocokkan nama pelanggan/mitra yang tertaut ke tiket.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend — Tangguhkan/Lanjutkan Tiket**

  - `backend/src/controllers/ticket.controller.js`
    - `addTicketMessage`: status pending kini **hanya** berubah lewat aksi eksplisit `suspend` (Tangguhkan) / `resume` (Lanjutkan).
      - Sebelumnya, pesan biasa yang dikirim saat tiket pending otomatis melanjutkan tiket, sehingga sekadar mencatat sesuatu sudah menghentikan hitungan durasi pending.
    - Menangguhkan tiket yang sudah pending, atau melanjutkan tiket yang tidak pending, ditolak 409 (`alreadyPending` / `notPending`).
    - Perubahan status memakai syarat atomik di query (`pending: { $ne: true }` untuk suspend, `pending: true` untuk resume). Bila dua admin menekan tombol bersamaan, yang kalah dijawab 409 dan pesannya tidak ikut tersimpan.

  **Backend — Pesan Cepat (baru)**

  - `backend/src/models/ticketQuickMessage.model.js` [NEW]
    - Koleksi `ticket_quick_messages`: teks alasan siap pakai per aksi (`suspend`/`resume`), maks 200 karakter, beserta pembuatnya.
    - Index unik `{ action, text }` mencegah teks ganda pada aksi yang sama.
    - Batas 100 pesan per aksi.
  - `backend/src/services/ticketQuickMessage.service.js` [NEW]
    - `findTicketQuickMessages`: daftar per aksi.
    - `createTicketQuickMessage`: ditolak 422 bila sudah 100 pesan, dan 409 bila teksnya ganda.
    - `deleteTicketQuickMessage`: 404 bila tidak ada, 403 bila bukan pembuatnya.
    - `canDeleteQuickMessage`: yang boleh menghapus hanya pembuatnya atau superadmin.
    - Semua kegagalan tak terduga dicatat `logger.error`.
  - `backend/src/controllers/ticketQuickMessage.controller.js` [NEW]
    - Handler `getTicketQuickMessages`, `addTicketQuickMessage`, dan `removeTicketQuickMessage`.
    - Validasi aksi (400 bila bukan `suspend`/`resume`) dan teks (400 bila kosong atau lebih dari 200 karakter).
    - Status error dari service diteruskan ke `res` lewat `withServiceStatus`.
    - Respons tiap pesan membawa penanda `can_delete` untuk UI.
  - `backend/src/routes/ticket.route.js`
    - Route baru, semuanya memakai `checkTicketPrivilegeByType('update')` (hak ubah tiket sesuai jenis tiket) dan dilengkapi Swagger:
      - `GET /ticket/:type/quick-message/list?action=`
      - `POST /ticket/:type/quick-message/create`
      - `DELETE /ticket/:type/quick-message/delete/:quick_message_id`
  - `backend/src/config/privilegeDictionary.json`
    - Ketiga endpoint pesan cepat ditambahkan ke daftar endpoint privilege `update` semua jenis tiket (hasil `npm run gd`).
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `ticket.alreadyPending`, `ticket.notPending`, dan grup `ticket.quickMessage.*` (gagal muat/simpan/hapus, ganda, tidak ditemukan, bukan pembuat, aksi tidak valid, teks wajib/terlalu panjang, berhasil hapus, batas tercapai).

  **Backend — Nama Hak Akses Unik**

  - `backend/src/services/privilege.service.js`
    - Fungsi baru `isPrivilegeNameTaken(name, excludeId)`: mengecek nama hak akses yang sudah dipakai tanpa membedakan huruf besar/kecil (collation `strength: 2`).
  - `backend/src/controllers/privilege.controller.js`
    - Create dan update hak akses menolak nama yang sudah dipakai hak akses lain dengan 409, dalam bentuk `{ error: { name } }` yang sudah dibaca form.
    - Mengganti kapitalisasi nama milik sendiri tetap boleh.
  - `backend/src/locales/id/translation.json`, `backend/src/locales/en/translation.json`
    - Key baru `privilege.nameTaken`.

  **Backend — Kapitalisasi Nama**

  - `backend/src/utils/validation-data.js`
    - `capitalizeWords` tidak lagi mengecilkan seluruh huruf sebelum mengkapitalkan huruf pertama tiap kata. Singkatan dan penulisan yang disengaja tetap terjaga ("PT ringnet" menjadi "PT Ringnet", bukan "Pt Ringnet").
    - Fungsi ini dipakai saat menyimpan nama pelanggan, pelanggan mitra, mitra, bisnis, karyawan, registrasi, dan hak akses.

  **Backend — Pencarian Tiket Telegram Mini App**

  - `backend/src/services/ticket.service.js`
    - `findMobileOpenTickets`: kata kunci pencarian juga dikonversi ke daftar `_id` pelanggan/mitra (`findCustomerIdsBySearch`, `findPartnerIdsBySearch`).
    - Tiket bisa ditemukan lewat nama pelanggan/mitra yang tertaut, yaitu baris "Pelanggan:" di kartu tiket. Sisanya perapian format.
  - `telegram-apps/src/pages/tickets/TicketList.jsx`
    - Placeholder pencarian menyebut "pelanggan".

  **Backend — Test**

  - `backend/test/integration/ticketAddMessagePending.test.js` [NEW]
    - Pesan biasa saat pending tidak melanjutkan tiket.
    - Suspend/resume berhasil.
    - Suspend pada tiket pending dan resume pada tiket tidak pending ditolak 409 tanpa menambah pesan.
  - `backend/test/integration/ticketQuickMessage.test.js` [NEW]
    - Teks di-trim; teks ganda pada aksi sama 409; teks sama di aksi berbeda boleh.
    - Validasi 400; daftar per aksi dengan `can_delete`.
    - Hapus hanya oleh pembuat atau superadmin; id tidak valid 404; batas 100 pesan 422.
  - `backend/test/integration/privilegeNameCaseInsensitive.test.js` [NEW]
    - Create/update menolak nama yang hanya beda kapitalisasi; update kapitalisasi nama sendiri boleh.
  - `backend/test/unit/capitalizeWords.test.js` [NEW]
    - Kapitalisasi kata huruf kecil, mempertahankan singkatan dan huruf kapital di tengah kata, serta nilai kosong.
  - `backend/test/unit/ticketMobile.service.test.js`
    - Pencarian ikut mencocokkan nama pelanggan/mitra tertaut, dan tidak menambah klausa bila tidak ada yang cocok atau kata kunci kosong.

  **Frontend — Tiket**

  - `frontend/src/app/pages/tickets/components/PendingActionModal.jsx` [NEW]
    - Pop-up Tangguhkan / Lanjutkan Tiket:
      - Dropdown **pesan cepat** sesuai aksi, dengan tombol hapus pada pesan yang boleh dihapus pemanggil, lewat `ConfirmModal`.
      - Isian **Alasan** yang wajib diisi.
      - Centang **Simpan sebagai pesan cepat**.
    - Isian di-reset setiap kali pop-up dibuka.
  - `frontend/src/app/pages/tickets/MessageUpdate.jsx`
    - Tombol Tangguhkan/Lanjutkan kini membuka `PendingActionModal`, tidak lagi langsung mengirim isi editor. Pop-up hanya dipasang untuk pemegang hak ubah tiket.
    - Tombol **Kirim** pesan biasa kini tetap tampil saat tiket pending, karena pesan biasa tidak lagi melanjutkan tiket.
    - Banner pending diberi petunjuk bahwa pesan biasa tidak melanjutkan tiket.
    - Bila status pending sudah diubah admin lain (409), tiket dimuat ulang.
  - `frontend/src/app/pages/tickets/utils/pendingMessage.js` [NEW], `frontend/src/app/pages/tickets/utils/pendingMessage.test.js` [NEW]
    - Konstanta batas pesan cepat (200 karakter, 100 pesan; sama dengan backend).
    - `plainTextToHtml`: mengubah alasan teks polos menjadi HTML pesan tiket. Karakter HTML di-escape dan tiap baris menjadi paragraf. Disertai test-nya.
  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - Key baru untuk banner, deskripsi pop-up, label Alasan, tombol Tangguhkan/Lanjutkan, dan grup `ticket.detail.quickMessage.*`.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #437 | Pop-up Tangguhkan/Lanjutkan Tiket & perbaikan terkait | Durasi pending tiket tidak lagi terpotong oleh pesan biasa; alasan pending bisa dipilih dari pesan cepat; nama hak akses unik; singkatan nama tidak rusak; pencarian tiket TMA lebih lengkap |

### Kemampuan Baru Pengguna/Admin

- Menangguhkan atau melanjutkan tiket dengan menuliskan alasan di pop-up khusus.
- Menyimpan alasan yang sering dipakai sebagai **pesan cepat**, lalu memilihnya lagi di tiket mana pun. Pesan cepat bisa dihapus pembuatnya atau superadmin.
- Mengirim pesan/catatan biasa ke tiket yang sedang ditangguhkan tanpa melanjutkannya.
- Mencari tiket di Telegram Mini App memakai nama pelanggan atau mitra.

### Bug Fix / Solusi Masalah

- Pesan biasa tidak lagi otomatis melanjutkan tiket yang sedang ditangguhkan, jadi durasi pending dan MTTR terhitung benar.
- Dua admin yang menangguhkan/melanjutkan tiket bersamaan tidak lagi menghasilkan catatan ganda (409).
- Nama hak akses yang hanya beda huruf besar/kecil tidak bisa lagi dibuat ganda.
- Nama berawalan singkatan (mis. "PT", "CV") tidak lagi berubah menjadi "Pt"/"Cv" saat disimpan.

### Menu/Fitur Baru

- Pop-up **Tangguhkan Tiket / Lanjutkan Tiket** dengan **Pesan cepat** di halaman detail tiket.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Tiket yang ditangguhkan tetap ditangguhkan sampai admin menekan "Lanjutkan Tiket". Mengirim pesan biasa hanya menambah catatan. Setiap penangguhan dan kelanjutan wajib disertai alasan, yang bisa diambil dari daftar pesan cepat bersama.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka detail tiket, lalu klik **Tangguhkan Tiket**.
  2. Pada pop-up, pilih alasan dari **Pesan cepat** atau tulis sendiri. Centang **Simpan sebagai pesan cepat** bila alasan ini akan sering dipakai.
  3. Klik **Tangguhkan**. Selama ditangguhkan, catatan bisa tetap dikirim lewat tombol **Kirim** biasa.
  4. Saat pekerjaan bisa dilanjutkan, klik **Lanjutkan Tiket**, isi keterangan, lalu klik **Lanjutkan**.
