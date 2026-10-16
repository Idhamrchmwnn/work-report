# 📝 Daily Work Report - Idham (2026-10-16)

---

## 📅 Laporan Harian - 16 Oktober 2026

---

## 🌿 Branch: `issue-423` — Perbaikan Simpan Nilai 0 pada Daftar Barang & Penyesuaian Teks

### 📌 Informasi Issue

- **Nomor Issue**: #423
- **Judul Issue**: Stok minimum dan harga bernilai 0 tidak tersimpan saat mengubah barang gudang, serta penyesuaian beberapa label
- **Status Branch**: `Belum di-merge` (sudah `resolve`, sudah di-push ke `origin/issue-423`)

### 📅 Rincian Commit

#### [90bc01ce] - resolve #423 - 16 Oktober 2026, 16:20

- **Ringkasan**:
  - Memperbaiki bug di **Gudang > Daftar Barang**: mengubah stok minimum atau harga menjadi `0` tidak pernah tersimpan, sehingga nilai lama tetap dipakai. Penyebabnya, `cleanFormData` membuang semua nilai falsy, termasuk angka `0`.
  - Mengganti tiga label teks yang kurang tepat di halaman tiket dan gudang.

- **Komponen yang Berubah & Penjelasannya**:

  **Backend**

  - `backend/src/controllers/warehouseType.controller.js`
    - Helper baru `keepZeroNumbers(body, data)` mengembalikan nilai `0` untuk `min_stock` dan `price` bila memang dikirim di body (sebagai angka `0` atau string `'0'`). Dua nilai ini sah secara bisnis:
      - `min_stock = 0`: mematikan peringatan stok rendah.
      - `price = 0`: barang gratis.
    - Helper dipanggil tepat setelah `cleanFormData` di dua handler:
      - `warehouseTypeUpdate`: ubah satu barang.
      - `warehouseTypeUpdateBatch`: ubah massal.
    - Field yang tidak dikirim tetap tidak disentuh, jadi ubah massal tidak menimpa nilai barang lain menjadi `0`. Pola ini mengikuti AGENTS.md §2.A aturan 8 (simpan nilai `0` sebelum/sesudah sanitasi).

  **Frontend — Teks (i18n)**

  - `frontend/src/i18n/locales/id/translations.json`, `frontend/src/i18n/locales/en/translations.json`
    - `ticket.drawer.openTicket`: "Buka Tiket" menjadi **"Lihat Tiket"** ("Open Ticket" menjadi "View Ticket").
      - Ini tombol di drawer detail tiket (`TicketDetailDrawer.jsx`) yang membuka halaman detail tiket. Label lama terkesan membuka atau mengaktifkan kembali tiket, padahal fungsinya hanya menampilkan.
    - `ticket.detail.pir.footer`: nama perusahaan yang tertulis mati "Abhinawa" di kaki dokumen **Post Incident Report** (`PostIncidentReportModal.jsx`) diganti variabel `{{company}}`.
      - Kaki dokumen kini mengikuti nama perusahaan di pengaturan aplikasi, sama seperti baris hak cipta di sebelahnya.
    - `warehouse.item.discardAndDestroy`: "Buang & Hancurkan" menjadi **"Hapus"** ("Discard & Destroy" menjadi "Delete").
      - Ini tombol hapus di detail barang (`itemDetail.jsx`), supaya seragam dengan tombol hapus massal "Hapus Barang Terpilih".

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| ----- | ----- | ------------ |
| #423 | Simpan nilai 0 pada barang gudang & penyesuaian teks | Stok minimum dan harga bisa diubah menjadi 0; label tiket, gudang, dan kaki dokumen PIR lebih tepat |

### Kemampuan Baru Pengguna/Admin

- Mengatur stok minimum barang menjadi `0` untuk mematikan peringatan stok rendah, dan harga menjadi `0` untuk barang gratis, baik lewat ubah satu barang maupun ubah massal.

### Bug Fix / Solusi Masalah

- Nilai `0` pada stok minimum dan harga di Daftar Barang kini tersimpan. Sebelumnya terbuang diam-diam dan nilai lama tetap berlaku.
- Kaki dokumen Post Incident Report tidak lagi menampilkan nama perusahaan yang tertulis mati.

### Menu/Fitur Baru

- Tidak ada menu baru.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Stok minimum dan harga barang kini menerima nilai `0`. Stok minimum `0` berarti barang tersebut tidak akan memicu peringatan stok rendah.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka **Gudang > Daftar Barang**.
  2. Ubah barang (atau pilih beberapa barang untuk ubah massal), isi **Stok Minimum** atau **Harga** dengan `0`, lalu simpan.
  3. Buka kembali detail barang untuk memastikan nilainya sudah `0`.
