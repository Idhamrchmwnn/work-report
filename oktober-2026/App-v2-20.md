# 📝 Daily Work Report - Idham (2026-10-20)

---

## 📅 Laporan Harian - 20 Oktober 2026

---

## 🌿 Branch: `issue-445` — Pencarian Tiket Terbuka lewat Nama Pelanggan/Mitra

### 📌 Informasi Issue

- **Nomor Issue**: #445
- **Judul Issue**: Pilihan tiket di form permintaan barang dan menu gudang Telegram Mini App bisa dicari lewat nama pelanggan/mitra
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-445`)

### 📅 Rincian Commit

#### [8c555b7e] - resolve #445 - 20 Oktober 2026, 18:02

- **Ringkasan**:
  - Pilihan tiket terbuka kini bisa dicari lewat **nama atau ID pelanggan/mitra** yang tertaut ke tiket, selain nomor dan nama tiket. Hasilnya juga menampilkan nama pelanggan/mitra itu.
  - Pilihan tiket ini dipakai di:
    - Form permintaan barang (web).
    - Menu **Ambil Barang**, **Pasang ke Pengguna**, dan **Pasang ke Node** di Telegram Mini App (TMA).
  - Sekaligus diperbaiki:
    - Urutan hasil kini tiket terbaru lebih dulu.
    - Kata kunci tidak lagi dieksekusi sebagai regex.
    - Tanggal tiket di TMA kini tampil benar.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/services/ticket.service.js`
    - `findTicketOpenedSelect` ditulis ulang:
      - **Pencarian per pelanggan/mitra:** kata kunci dikonversi ke daftar `_id` lewat `findCustomerIdsBySearch` dan `findPartnerIdsBySearch`, lalu ditambahkan ke `$or` bersama nomor dan nama tiket. Polanya sama dengan pencarian tiket TMA (`findMobileOpenTickets`).
      - **Nama pelanggan/mitra di hasil:** field baru `customer_name`, diambil lewat `$lookup` ke koleksi pelanggan dan mitra. Lookup dijalankan **setelah** `$limit`, jadi hanya 30 tiket yang di-join.
      - **Urutan terbaru lebih dulu:** ditambah `$sort: { created_at: -1 }` sebelum `$limit`. Sebelumnya tanpa urutan, jadi 30 hasil yang muncul bisa tiket lama mana saja.
      - **Query `.`** memuat 30 tiket terbuka terbaru tanpa filter. Ini dipakai TMA untuk daftar awal.
      - **Keamanan input:**
        - Kata kunci di-escape (`escapeRegex`) sebelum masuk regex, jadi karakter seperti `(` atau `+` dianggap teks biasa. Sebelumnya pola rusak memicu error dan pola mahal bisa memberatkan database.
        - Nilai bukan string (mis. objek operator Mongo) dianggap kosong.
      - `catch` yang sebelumnya diam kini mencatat `logger.error`.
  - `backend/src/routes/ticket.route.js`
    - Swagger `POST /ticket/opened-select` dilengkapi:
      - Deskripsi cara pencarian (termasuk query `.` dan query kosong).
      - Skema respons beserta field `customer_name`.
      - Kode error 401/404.
  - `backend/test/integration/ticketOpenedSelect.test.js` [NEW]
    - Tiket ditemukan lewat nama pelanggan dan lewat nama mitra, dengan `customer_name` terisi.
    - Tiket yang sudah selesai tidak ikut.
    - Karakter regex dianggap teks biasa dan pola rusak tidak melempar error.
    - Query `.` memuat tiket terbaru (maks 30).
    - Query kosong atau bukan string mengembalikan array kosong.

  **Telegram Mini App**

  - `telegram-apps/src/components/shared/AsyncSelect.jsx`
    - Keterangan tiap pilihan tiket kini berisi nama pelanggan/mitra, jenis tiket, dan tanggal, supaya hasil pencarian per nama pelanggan mudah dikenali.
    - **Perbaikan tanggal:** keterangan sebelumnya membaca `item.created_at`, padahal backend mengirim `date`, sehingga tanggal tiket tidak tampil benar. Sekarang membaca `date`.
  - `telegram-apps/src/pages/warehouse/takeItem.jsx`, `telegram-apps/src/pages/warehouse/installItem.jsx`, `telegram-apps/src/pages/warehouse/installSiteItem.jsx`
    - Placeholder pencarian tiket di menu Ambil Barang, Pasang ke Pengguna, dan Pasang ke Node menjadi "Cari ID tiket, nama tiket, atau pelanggan".

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #445 | Pencarian tiket terbuka lewat nama pelanggan/mitra | Teknisi dan admin bisa menemukan tiket dengan mengetik nama pelanggan/mitra saat mengambil atau memasang barang, dan saat membuat permintaan barang |

### Kemampuan Baru Pengguna/Admin

- Mencari tiket terbuka dengan mengetik nama atau ID pelanggan/mitra, di form permintaan barang (web) dan di menu gudang TMA.
- Melihat nama pelanggan/mitra di setiap pilihan tiket di TMA.

### Bug Fix / Solusi Masalah

- Tanggal tiket di pilihan tiket TMA kini tampil benar.
- Hasil pencarian tiket kini diurutkan dari yang terbaru. Sebelumnya urutannya acak dan tiket baru bisa tidak muncul di 30 hasil teratas.
- Kata kunci berisi karakter khusus (mis. tanda kurung) tidak lagi menyebabkan error.

### Menu/Fitur Baru

- Tidak ada menu baru.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Saat memilih tiket untuk pengambilan atau pemasangan barang, kolom pencarian kini mencocokkan nomor tiket, nama tiket, dan nama/ID pelanggan atau mitra yang tertaut ke tiket. Hanya tiket yang belum selesai yang ditampilkan, maksimal 30, terbaru lebih dulu.
- **Langkah Penggunaan (Tutorial)**:
  1. Di Telegram Mini App, buka menu **Ambil Barang**, **Pasang ke Pengguna**, atau **Pasang ke Node**. Di web, buka form **permintaan barang**.
  2. Pada kolom tiket, ketik nama pelanggan atau mitra (mis. "budi").
  3. Pilih tiket yang sesuai. Nama pelanggan/mitra tampil di keterangan setiap pilihan.
