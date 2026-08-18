# 📝 Daily Work Report - Idham (2026-08-18)

---

## 📅 Laporan Harian - 18 Agustus 2026

---

## 🌿 Branch: `issue-228` — Implementasi Dokumen SDN

### 📌 Informasi Issue

- **Nomor Issue**: #228
- **Judul Issue**: Implementasi Dokumen SDN
- **Status Branch**: `Belum sepenuhnya di-push` — 1 commit lokal (`2e63628`) ahead dari `origin/issue-228`, ditambah perubahan yang sudah di-`stage` namun **belum di-commit**

### 📅 Rincian Commit

#### [e1dfceb] - save #228 - 18 Agustus 2026, 12:05 (rebase, sudah ada di `origin/issue-228`)

- Commit lama (author date 11 Agustus) yang di-rebase ulang ke atas `master` terbaru; isi (modul SDN) sudah pernah dilaporkan pada laporan 14 Agustus, tidak ada perubahan konten baru dari rebase ini.

#### [2e63628] - resolve #223 - 18 Agustus 2026, 12:34 (rebase, **belum di-push**)

- **Komponen yang Berubah**: menyusulkan seluruh modul **PKS (Perjanjian Kerja Sama)** (`customerPKS.controller.js`, `publicCustomerPKS.controller.js`, model/service/route PKS, halaman `create`/`edit`/preview/review PKS, dsb — lihat rincian lengkap di laporan 14 Agustus) ke atas branch `issue-228`.
- **Deskripsi**: commit hasil rebase dari pekerjaan issue #223 yang kini juga tergabung secara lokal di branch `issue-228`, di atas `save #228`. Belum di-push ke `origin/issue-228`.

### 🚧 Perubahan Belum Di-commit (Work In Progress, sudah di-`stage`)

- **Komponen yang Berubah**:
  - `backend/src/controllers/customerSDN.controller.js` (+11/-1)
  - `backend/src/controllers/customerSO.controller.js` (+10/-2)
  - `frontend/src/app/pages/users/customerSDN/create.jsx` (+5)
  - **Total**: 3 files changed, 23 insertions(+), 3 deletions(-)
- **Deskripsi Perubahan & Fungsi**:
  - **Perbaikan resolusi ID partner vs customer** pada `createSDN` (backend): sebelumnya `partner_id` yang dikirim dari frontend langsung dipakai apa adanya untuk `findCustomerById`. Sekarang ID diperiksa prefiks-nya — jika berawalan `PART-` maka prefiks dilepas sebelum lookup (entitas Partner, sesuai [[customer-po-so-documents]]), jika berawalan `CUST-` maka lookup langsung digagalkan (`null`) sehingga request akan mendapat error `404 customerNotFound`. Ini mencegah SDN dibuat menggunakan ID entitas **Customer** biasa, yang seharusnya tidak berlaku untuk SDN (SDN hanya berlaku untuk Partner yang punya SO).
  - **Perbaikan serupa** pada `listSO` (backend, `customerSO.controller.js`): parameter `customer_id` di endpoint list SO kini juga difilter dengan pola prefiks `PART-`/`CUST-` yang sama sebelum dipakai untuk mencari customer, agar konsisten dengan aturan di atas.
  - **Auto-fill data servis dari SO terpilih** pada form create SDN (frontend, `customerSDN/create.jsx`): saat user memilih SO pada dropdown, field `service_id` kini otomatis terisi dari `so_number` milik SO tersebut, dan field `service_ordered` otomatis terisi dari gabungan nama-nama `line_items` SO (dipisah koma) — mengurangi input manual saat membuat dokumen SDN.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul                                  | Dampak Utama                                                                                              |
| ----- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| #228  | Implementasi Dokumen SDN                | Validasi ID partner/customer pada pembuatan SDN & listing SO diperketat; form create SDN kini auto-fill data servis dari SO terkait |

### Bug Fix / Solusi Masalah

- Mencegah pembuatan dokumen SDN dengan ID **Customer** (bukan Partner) yang secara bisnis tidak seharusnya memiliki dokumen SDN — request semacam ini sekarang konsisten ditolak dengan `404 customerNotFound` di `createSDN` maupun `listSO`.

### Menu/Fitur Baru

- Tidak ada menu baru hari ini — murni perbaikan validasi backend dan peningkatan UX (auto-fill) pada form SDN yang sudah ada.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Saat membuat dokumen SDN dan memilih SO terkait, form sekarang otomatis mengisi kolom **Service ID** (dari nomor SO) dan **Service Ordered** (dari daftar item pada SO) tanpa perlu diketik manual. Di sisi backend, sistem sekarang memvalidasi bahwa ID yang dipakai untuk membuat SDN maupun menampilkan daftar SO benar-benar milik entitas **Partner** (prefiks `PART-`), bukan entitas **Customer** biasa (prefiks `CUST-`).
- **Langkah Penggunaan (Tutorial)**:
  1. Buka menu **Dokumen SDN** → **Buat Baru**.
  2. Pilih Partner terkait, lalu pilih SO yang berlaku dari dropdown.
  3. Field **Service ID** dan **Service Ordered** akan terisi otomatis mengikuti data SO yang dipilih — dapat disunting manual bila perlu.
  4. Jika ID yang digunakan ternyata milik entitas Customer (bukan Partner), sistem akan menolak dengan pesan "customer not found".

---

## ⚠️ Catatan Tambahan

- Perubahan pada `customerSDN.controller.js`, `customerSO.controller.js`, dan `customerSDN/create.jsx` **masih berstatus staged, belum di-commit**. Perlu dibuatkan commit (mis. lanjutan `save #228` atau commit baru) sebelum di-push.
- Commit `2e63628` (`resolve #223`) membawa modul PKS ke branch `issue-228` secara lokal dan **belum di-push** ke `origin/issue-228` — perlu dicek apakah penggabungan PKS ke branch SDN ini disengaja, atau seharusnya tetap dipisah di branch `issue-223`.
- Tidak ada aktivitas commit baru dari kolaborator lain (Dedy S.N Putra) sejak 16 Agustus 2026 (`save #217` di branch terkait #216/#217).
