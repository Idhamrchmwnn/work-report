# 📝 Daily Work Report - Idham (2026-09-10)

---

## 📅 Laporan Harian - 10 September 2026

---

## 🌿 Branch: `issue-280` — Notifikasi Aplikasi Mobile (Personal & Broadcast)

### 📌 Informasi Issue

- **Nomor Issue**: #280, sub-issue dari #277 "Integrasi dengan MobileApps (Android)"
- **Judul Issue**: #277 - Implementasi Push Notifikasi
- **Status Branch**: `Belum di-merge` (sudah di-push ke `origin/issue-280`) — dibangun di atas hasil `issue-278` (Berita/Banner), bukan di atas `issue-279` (Perubahan Layanan, lihat catatan risiko di bawah).

### 📅 Rincian Perubahan

#### [93125895] - resolve #280 - 10 September 2026, 18:17:55 WIB

- **Komponen yang Berubah**:
  - [`backend/src/models/mobileNotification.model.js`](backend/src/models/mobileNotification.model.js) [NEW] — Model `MobileNotification`. Satu dokumen = satu notifikasi yang dikirim admin, dengan dua mode sasaran: **personal** (`customer` terisi, status baca di field `read`/`read_at`) atau **broadcast** (`customer: null`, dikirim ke *seluruh* pelanggan sekaligus — status baca per pelanggan dicatat terpisah di array `read_by`, supaya satu dokumen bersama tidak ikut berubah status bacanya untuk semua pelanggan lain). Punya **TTL index 30 hari** pada `created_at` — notifikasi otomatis terhapus sendiri sebulan setelah dibuat (disengaja disamakan dengan perilaku aplikasi versi lama, supaya koleksi ini tidak membengkak tanpa batas). Business ID `notification_id` (auto-increment) dipakai di URL/link, bukan ObjectId Mongo mentah — pola yang sama dengan `news_id` di modul News tetangganya.
  - [`backend/src/services/mobileNotification.service.js`](backend/src/services/mobileNotification.service.js) [NEW, 497 baris] — 11 fungsi, terbagi jelas dua sisi:
    - **Sisi admin**: `findAllNotificationsForTable` (list datatable), `getNotificationStats` (ringkasan: total, jumlah broadcast, jumlah personal, jumlah terbaca), `findNotificationRecipient` (cari pelanggan tujuan by `customer_id` atau ObjectId, mengecualikan pelanggan blacklist/pasif), `createNewNotification` (simpan notifikasi baru — `customerId` kosong berarti broadcast), `findNotificationById`, `deleteNotificationById`.
    - **Sisi pelanggan** (dipakai endpoint mobile): `findNotificationsForCustomer` (gabungan notifikasi personal miliknya + seluruh broadcast, terbaru dulu), `countUnreadNotificationForCustomer`, `markNotificationAsRead` (menangani kedua mode: `read=true` untuk personal, push ke `read_by` untuk broadcast), `markAllNotificationAsRead`.
    - **Helper lintas-modul**: `notifyCustomer({customerId, title, message, adminId})` — fungsi kecil yang sengaja dibuat *fail-safe* (dibungkus try/catch, gagal kirim notifikasi tidak menggagalkan alur pemanggilnya, cuma di-log sebagai warning) supaya modul lain (komentar di kode menyebut contoh: tindak lanjut request perubahan layanan, atau penagihan) bisa memicu notifikasi ke pelanggan tanpa perlu menangani error notifikasi secara khusus. **Catatan: helper ini sudah tersedia tapi belum dipanggil dari modul manapun hari ini** — murni infrastruktur yang disiapkan untuk dipakai fitur lain ke depannya.
  - [`backend/src/controllers/mobileNotification.controller.js`](backend/src/controllers/mobileNotification.controller.js) [NEW, 161 baris] — 9 handler, mencerminkan pembagian service di atas: `listNotification`, `statsNotification`, `readNotification` (detail satu notifikasi), `createNotification` (validasi panjang judul/pesan, cari pelanggan penerima bila diisi, atau broadcast bila kosong), `deleteNotification` untuk sisi admin; `getMyNotificationList`, `getMyUnreadNotificationCount`, `readMyNotification`, `readAllMyNotification` untuk sisi pelanggan.
  - [`backend/src/routes/mobileNotification.route.js`](backend/src/routes/mobileNotification.route.js) [NEW, 216 baris] — Endpoint admin: `POST /mobile-notification/list`, `GET /mobile-notification/stats`, `GET /mobile-notification/read/:id`, `POST /mobile-notification/create`, `DELETE /mobile-notification/delete/:id` — semua digerbang `protectedAdmin` + `checkPrivilege('mobileNotification.*')`.
  - [`backend/src/routes/mobileCustomer.route.js`](backend/src/routes/mobileCustomer.route.js) — 4 endpoint baru untuk pelanggan (digerbang `protectedCustomer`, bukan admin): `GET /customer/notification` (daftar), `GET /customer/notification/unread-count`, `PATCH .../read/:id`, `PATCH .../read-all`.
  - [`backend/src/app.js`](backend/src/app.js) — Mendaftarkan `MobileNotificationRoute`.
  - [`backend/src/config/privilege.json`](backend/src/config/privilege.json) — Grup baru `mobileNotification`: `list`, `read`, `create`, `delete` (**tidak ada `update`** — notifikasi yang sudah terkirim tidak bisa diedit, hanya bisa dihapus).
  - [`frontend/src/app/navigation/mobileApp.js`](frontend/src/app/navigation/mobileApp.js) — Item menu baru "Notifikasi" di bawah root Mobile App.
  - [`frontend/src/app/pages/mobileApp/notification/`](frontend/src/app/pages/mobileApp/notification/) [NEW] — `index.jsx` (list + kartu ringkasan statistik dari endpoint `stats`), `create.jsx` (form kirim notifikasi baru: `Combobox` pencarian pelanggan — dikosongkan berarti kembali ke mode broadcast, bukan sekadar "tidak dipilih" — judul, dan pesan dengan penghitung karakter live terhadap batas maksimal), `detail.jsx` (lihat isi lengkap satu notifikasi), `schema/columns.jsx`, `schema/createSchema.js` (validasi Yup, termasuk `MAX_MESSAGE_LENGTH`), `schema/NotificationBadge.jsx` (badge "Broadcast" untuk notifikasi ke semua pelanggan, dan badge status Terbaca/Belum — badge status ini sengaja **tidak dirender** untuk notifikasi broadcast karena status bacanya per-pelanggan, bukan satu status tunggal).
  - [`frontend/src/app/router/protected.jsx`](frontend/src/app/router/protected.jsx) — Route baru untuk ketiga halaman di atas.
  - [`frontend/src/components/shared/table/status.js`](frontend/src/components/shared/table/status.js) — Opsi filter baru `mobileNotificationReadOptions`.
  - [`frontend/src/constants/privilegeDescriptions.{en,id}.json`](frontend/src/constants/privilegeDescriptions.id.json), [`backend/src/locales/{en,id}/translation.json`](backend/src/locales/id/translation.json), [`frontend/src/i18n/locales/{en,id}/translations.json`](frontend/src/i18n/locales/id/translations.json) — String & deskripsi privilege baru untuk seluruh modul ini.
  - [`backend/test/integration/mobileNotification.service.test.js`](backend/test/integration/mobileNotification.service.test.js) [NEW, 414 baris] — Test integrasi cukup lengkap: pembuatan notifikasi personal & broadcast, penandaan baca untuk kedua mode, penghitungan belum-baca, serta validasi bahwa `notifyCustomer` benar-benar tidak melempar error ke pemanggilnya saat gagal.
- **Deskripsi Perubahan & Fungsi**:
  - Modul ini memberi admin kemampuan mengirim pesan/pengumuman ke pelanggan aplikasi mobile — baik ke satu pelanggan tertentu maupun broadcast ke semua sekaligus — dan pelanggan bisa melihat, menandai baca satu-per-satu atau sekaligus semua, dari sisi aplikasinya.
  - **Catatan penting soal penamaan**: judul issue-nya "Implementasi Push Notifikasi", tapi implementasi hari ini murni **notifikasi in-app** (tersimpan di database, diambil aplikasi mobile lewat polling/fetch API biasa) — **tidak ada integrasi push notification level-OS** (tidak ada pendaftaran token perangkat, tidak ada pemanggilan Firebase Cloud Messaging/APNs atau layanan sejenis di mana pun pada perubahan hari ini). Pelanggan hanya akan melihat notifikasi ini bila membuka aplikasi dan mengecek halaman notifikasinya — tidak akan muncul sebagai notifikasi dorong di layar kunci/notification tray perangkat. Perlu dikonfirmasi ke stakeholder apakah ini memang cakupan yang dimaksud untuk hari ini (mis. push notification level-OS direncanakan sebagai pekerjaan lanjutan terpisah), atau ada bagian yang tertinggal dari definisi awal issue.

---

## ⚠️ Catatan Risiko: Percabangan Branch `mobileApp.js` Makin Melebar

- Branch ini dibangun di atas `issue-278` (Berita/Banner) — **bukan** di atas `issue-279` (Perubahan Layanan, dilaporkan 9 September) yang juga mengubah `frontend/src/app/navigation/mobileApp.js`. Dikonfirmasi lewat `git merge-base --is-ancestor`: commit `issue-279` masih **bukan** leluhur dari commit hari ini.
- Artinya sekarang ada (setidaknya) dua garis riwayat yang sama-sama mengubah `mobileApp.js` secara independen: **{issue-278 → issue-280}** (berisi item menu Berita/Banner + Notifikasi) di satu sisi, dan **{issue-279}** (berisi item menu Perubahan Layanan) di sisi lain. Risiko konflik merge yang dicatat kemarin **belum berkurang, malah isi yang perlu direkonsiliasi makin banyak** (bukan cuma 2 item menu yang perlu digabung, tapi berpotensi 3).
- Disarankan: sebelum ada branch baru lagi yang menyentuh file navigasi Mobile App ini, segera satukan `issue-279` ke dalam rantai `issue-278`/`issue-280` (atau sebaliknya) lewat rebase, supaya `mobileApp.js` final berisi seluruh item menu (Berita/Banner, Perubahan Layanan, Notifikasi) dalam satu riwayat yang konsisten sebelum di-review/merge ke `master`.

---

## 📖 Informasi & Tutorial Singkat Fitur

- **Kegunaan Fitur**: Sebelumnya tidak ada cara bagi perusahaan untuk mengirim pesan/pengumuman langsung ke pelanggan lewat aplikasi mobile — informasi seperti gangguan terjadwal, promo, atau info penting lain hanya bisa disampaikan lewat WhatsApp/Telegram/kanal lain di luar aplikasi. Modul ini memungkinkan tim untuk mengirim pengumuman **massal** (ke semua pengguna aplikasi sekaligus) maupun pesan **personal** (ke satu pelanggan spesifik, mis. terkait tiket/keluhan tertentu) langsung dari aplikasi utama, dan pelanggan melihatnya di dalam aplikasi mobile mereka.
- **Langkah Penggunaan (Tutorial — Sisi Admin)**:
  1. Buka sidebar **Mobile App → Notifikasi**. Halaman list menampilkan kartu ringkasan (total notifikasi, jumlah broadcast, jumlah personal, jumlah terbaca) di bagian atas, diikuti daftar seluruh notifikasi yang pernah dikirim.
  2. Klik tombol **Kirim Notifikasi Baru**.
  3. **Untuk mengirim ke satu pelanggan tertentu**: cari & pilih nama pelanggan di kolom pencarian pelanggan.
  4. **Untuk broadcast ke semua pelanggan**: biarkan kolom pelanggan kosong (klik ikon hapus pilihan bila sudah sempat memilih) — sistem akan menandainya sebagai notifikasi ke seluruh pengguna aplikasi.
  5. Isi **Judul** dan **Pesan** (ada penghitung karakter yang menunjukkan sisa kuota panjang pesan).
  6. Kirim — notifikasi langsung tersimpan dan akan muncul di aplikasi mobile penerima saat mereka membuka halaman notifikasinya (bukan sebagai notifikasi dorong instan di layar perangkat, lihat catatan di bagian Rincian Perubahan).
  7. Notifikasi yang sudah terkirim tidak bisa diedit — bila ada kesalahan, satu-satunya opsi adalah menghapusnya (tombol hapus di daftar/detail) dan mengirim ulang yang baru.
  8. Notifikasi otomatis terhapus dari sistem setelah 30 hari, tidak perlu dibersihkan manual.
- **Sisi Pelanggan (aplikasi mobile, untuk konteks)**: pelanggan melihat gabungan notifikasi personal miliknya dan seluruh notifikasi broadcast dalam satu daftar (terbaru di atas), bisa menandai satu atau semua sekaligus sebagai sudah dibaca, dan melihat jumlah notifikasi belum dibaca (biasanya ditampilkan sebagai badge angka pada ikon lonceng).
