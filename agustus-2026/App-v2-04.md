# 📝 Daily Work Report - Idham (2026-08-04)

---

## 📌 Informasi Issue
- **Nomor Issue**: #153
- **Judul Issue**: Prospect Management — Pemantapan Konversi & Status (Privilege `changeStatus`, Cleanup Registrasi Sumber, Pesan Error i18n, Dokumentasi Swagger, & Rename Fungsi Service)

## 📅 Laporan Harian - 4 Agustus 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Belum ada commit hari ini. Lanjut di branch **`issue-153`** (Prospect). Working tree kumulatif: **22 berkas** (±925 baris ditambah / ±596 dihapus terhitung sejak basis #153). **Pekerjaan hari ini** (di atas laporan [3 Agustus](App-v2-3.md)) berupa 15 berkas dengan ±414 baris ditambah / ±126 dihapus — fokus **pemantapan (hardening) alur konversi & perubahan status prospek**, bukan fitur besar baru.

Setelah kemarin memindahkan intake prospek ke alur "Jadikan Prospek" dan mengintegrasikan funnel dokumen, hari ini merapikan sisi **keamanan (privilege), keandalan (error & cleanup), dan dokumentasi (Swagger)** dari modul Prospect.

---

### 🧩 1. Penyatuan Privilege: `convert` → `changeStatus`

- [backend/src/config/privilege.json](backend/src/config/privilege.json)
  - **Perubahan**: Key privilege prospek `"convert": "PROSPECT_CONVERT"` diganti menjadi `"changeStatus": "PROSPECT_CHANGESTATUS"`.
  - **Alasan/Fungsi**: Aksi **konversi** (prospek → customer), **mark-lost**, dan **buka kembali (reopen)** semuanya adalah *perubahan status* prospek. Menyatukannya di bawah satu privilege `changeStatus` menyederhanakan pemberian hak akses dan konsisten dengan modul lain (mis. `ticket.changeStatus`).
- [backend/src/routes/prospect.route.js](backend/src/routes/prospect.route.js)
  - **Perubahan**: Endpoint `/prospect/convert/:id`, `/prospect/mark-lost/:id`, dan perubahan status kini memakai `checkPrivilege('prospect.changeStatus')` (sebelumnya `prospect.convert`).
  - **Fungsi**: Menegakkan privilege baru pada seluruh endpoint yang mengubah status.

### 🧩 2. Cleanup Registrasi Sumber Saat Konversi/Pembuatan Prospek

- [backend/src/controllers/prospect.controller.js](backend/src/controllers/prospect.controller.js) *(±94 baris)*
  - **Perubahan**:
    - Helper baru **`cleanupSourceRegistration(registrationId)`** — mencari registrasi via `findOneRegistration({ document_id })` lalu `deleteRegistrationById`. Dipanggil di dalam handler create-prospect memakai `req.body.source_registration_id`.
    - Mengimpor `findOneRegistration` & `deleteRegistrationById` dari `registration.service.js`.
  - **Alasan/Fungsi**: Saat prospek dibuat dari sebuah registrasi (alur "Jadikan Prospek"), registrasi sumber dihapus **otomatis di backend** — bukan lewat request `DELETE` terpisah dari frontend. Dengan begitu admin **tidak perlu** privilege `registration.delete` untuk merampungkan konversi; pembersihan menyatu dengan satu aksi buat-prospek dan lebih tahan gagal-sebagian.
- [frontend/src/app/pages/users/registration/detail.jsx](frontend/src/app/pages/users/registration/detail.jsx) *(±21 baris)*
  - **Perubahan**: Tidak lagi memanggil `axios.delete('/registration/delete/:id')` manual setelah membuat prospek. Kini meneruskan **`sourceRegistrationId={id}`** ke `ProspectCreateDrawer` dan menavigasi ke daftar registrasi via `onSuccess`. Kode & komentar terkait privilege `registration.delete` dihapus.
  - **Fungsi**: Alur "Jadikan Prospek" jadi satu langkah; penghapusan registrasi ditangani server.
- [frontend/src/app/pages/services/prospect/create.jsx](frontend/src/app/pages/services/prospect/create.jsx) *(±10 baris hari ini)*
  - **Perubahan**: `ProspectCreateDrawer` menerima prop `sourceRegistrationId` dan menyertakannya pada payload create (untuk memicu cleanup di backend).
  - **Fungsi**: Menyambungkan aksi frontend dengan cleanup backend.

### 🧩 3. Pesan Error Backend Diterjemahkan (i18n), Bukan Kode Konstanta

- [backend/src/controllers/prospect.controller.js](backend/src/controllers/prospect.controller.js)
  - **Perubahan**: Kode error konstanta diganti pesan ter-terjemah — `ALREADY_CONVERTED` → `req.t('prospect.alreadyConverted')`, `INVALID_STATUS` → `req.t('prospect.invalidStatusChange')`, dsb. Status respons create disesuaikan (201 → 200 pada jalur terkait konversi).
  - **Fungsi**: Frontend cukup menampilkan `err.message` apa adanya (tanpa memetakan kode ke teks).
- [backend/src/locales/id/translation.json](backend/src/locales/id/translation.json) · [en](backend/src/locales/en/translation.json)
  - **Perubahan**: Menambah kunci pesan prospek:
    - `alreadyConverted` — "Prospek sudah dikonversi menjadi customer"
    - `invalidStatusChange` — "Perubahan status ini tidak diperbolehkan"
    - `notLost` — "Hanya prospek berstatus lost yang bisa dibuka kembali"
    - `noSignedSO` — "Konversi membutuhkan minimal satu SO yang sudah ditandatangani"
    - `notFound` — "Prospek tidak ditemukan"
  - **Fungsi**: Menjelaskan aturan bisnis konversi/status secara jelas ke pengguna (mis. konversi wajib punya SO signed; hanya prospek *lost* yang bisa dibuka lagi).
- [frontend/src/app/pages/services/prospect/convert.jsx](frontend/src/app/pages/services/prospect/convert.jsx) *(±12 baris)*
  - **Perubahan**: URL konversi memakai **`prospect.prospect_id`** (bukan `_id`); respons `res.data.data` kini berupa objek **partner** langsung; error toast disederhanakan menjadi `err.message` (tak lagi `t('prospect.error.${err.message}')`).
  - **Fungsi**: Selaras dengan identifier publik prospek & format respons baru; pesan error konsisten dari backend.

### 🧩 4. Dokumentasi Swagger Endpoint Prospek

- [backend/src/routes/prospect.route.js](backend/src/routes/prospect.route.js) *(±307 baris)*
  - **Perubahan**: Menambah blok `@swagger` (OpenAPI) untuk endpoint prospek — daftar, detail, create, update, **convert**, **mark-lost**, dan perubahan status — termasuk parameter, contoh body, dan respons error (mis. "Alasan lost kosong / prospek sudah won-convert").
  - **Fungsi**: Endpoint prospek kini terdokumentasi di Swagger UI, memudahkan integrasi & pengujian.

### 🧩 5. Rename Fungsi Service (Konsistensi Penamaan)

- [backend/src/services/prospect.service.js](backend/src/services/prospect.service.js)
  - **Perubahan**: `findListProspectForTable` → **`findAllProspectsForTable`**; `updateProspectById` → **`updateProspectData`**.
  - **Fungsi**: Menyeragamkan nama fungsi dengan modul lain (pola `findAll…ForTable` / `update…Data`).
- [backend/src/utils/advanceProspectStatus.js](backend/src/utils/advanceProspectStatus.js)
  - **Perubahan**: Mengikuti rename — memakai `updateProspectData` (dari `updateProspectById`).
  - **Fungsi**: Menghindari referensi ke nama fungsi lama.

### 🧩 Penyesuaian Pendukung

- [frontend/src/app/pages/services/prospect/detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx) *(±37 baris hari ini)* · [edit.jsx](frontend/src/app/pages/services/prospect/edit.jsx) *(±7)*
  - **Perubahan**: Penyesuaian pemanggilan aksi status/konversi & identifier mengikuti perubahan privilege dan format respons.
- [backend/src/controllers/workOrder.controller.js](backend/src/controllers/workOrder.controller.js) *(±4 baris)*
  - **Perubahan**: Penyesuaian kecil terkait induk prospek pada alur WO.
- [frontend/src/i18n/locales/*/translations.json](frontend/src/i18n/locales/id/translations.json) *(±15 baris/berkas)*
  - **Perubahan**: Label pendukung konfirmasi konversi & aksi status prospek.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Hak akses perubahan status prospek (konversi, mark-lost, reopen) dikelola lewat satu privilege **`prospect.changeStatus`**.
  - Alur "Jadikan Prospek" kini **satu langkah** — registrasi sumber dibersihkan otomatis tanpa perlu privilege hapus registrasi.
  - Pesan aturan bisnis konversi/status tampil jelas (mis. "Konversi membutuhkan minimal satu SO yang sudah ditandatangani").
- **Bug Fix / Solusi Masalah**:
  - Menghilangkan ketergantungan pada privilege `registration.delete` untuk merampungkan konversi (cleanup dipindah ke backend, lebih atomik).
  - Menyeragamkan pesan error (i18n) sehingga frontend tak lagi memetakan kode konstanta.
  - Memakai `prospect_id` publik & format respons partner yang konsisten pada konversi.
- **Menu/Tombol Baru**: Tidak ada menu baru; perubahan bersifat penguatan alur, hak akses, dan dokumentasi.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Perubahan status prospek (konversi menjadi customer, menandai *lost*, membuka kembali) kini berada di bawah satu izin `changeStatus` dengan aturan yang jelas dan pesan ter-terjemah. Konversi dari registrasi menuntaskan sekaligus pembersihan data registrasi sumber di server.
- **Langkah Penggunaan (Tutorial)**:
  1. **Jadikan Prospek**: dari detail registrasi → "Jadikan Prospek" → simpan; registrasi sumber otomatis terhapus (butuh izin `prospect.create`/`changeStatus`, bukan `registration.delete`).
  2. **Konversi ke customer**: pada prospek yang punya minimal satu **SO signed**, jalankan konversi; bila belum ada SO signed, sistem menolak dengan pesan yang jelas.
  3. **Mark-lost / reopen**: tandai prospek *lost*; hanya prospek berstatus *lost* yang bisa dibuka kembali.
  4. **Referensi API**: endpoint prospek dapat ditinjau di Swagger UI.

> **Catatan teknis**: Perubahan menyentuh privilege, service (rename fungsi), dan pesan i18n — **restart backend** diperlukan; pastikan role admin memiliki `PROSPECT_CHANGESTATUS` menggantikan `PROSPECT_CONVERT`. Nama berkas laporan mengikuti pola `App-v2-<tanggal>` (hari ke-4 Agustus).
