# 📝 Daily Work Report - Idham (2026-07-17)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Prospect Management — Modul Funnel Pra-Deal (Fase Dokumen: Penyempurnaan Halaman Detail Prospek, Prefill Work Order dari SO, & Perbaikan Pratinjau SO Publik)

## 📅 Laporan Harian - 17 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

> Tidak ada commit baru hari ini (commit terakhir tetap `7aa4cee save #123` tertanggal 2026-07-13). **Seluruh pekerjaan 17 Juli 2026 masih berstatus WIP** di branch `issue-123`.
>
> Working tree kini berisi **45 berkas berubah + 9 berkas baru** (±2.804 baris ditambah / ±1.280 dihapus). **Delta terhadap laporan 16 Juli** adalah +2 berkas yang mulai ikut berubah ([workOrder/create.jsx](frontend/src/app/pages/services/workOrder/create.jsx) dan [ReviewCustomerSOPage.jsx](frontend/src/app/pages/public/ReviewCustomerSOPage.jsx)) serta perluasan lanjutan pada halaman detail prospek dan berkas terjemahan.

Hari ini pekerjaan bergeser dari membangun kerangka besar (yang sudah rampung di 15–16 Juli: pendaftaran prospek publik, foto survei, PIC penawaran, alur Work Order, dan penguncian bertahap/*phase-lock*) menuju **penghalusan pengalaman pemakaian (UX) dan penyambungan data antar-fase**. Tiga area digarap: (1) menuntaskan halaman detail prospek sebagai pusat kendali seluruh funnel dokumen, (2) mengalirkan data yang sudah pernah diisi pelanggan langsung ke pembuatan Work Order agar tidak perlu diketik ulang, dan (3) memperbaiki satu bug pratinjau dokumen SO pada halaman review publik.

---

### 🏗️ Konteks & Latar Belakang

Modul **Prospect Management** (funnel pra-deal) sudah memiliki seluruh tulang punggungnya: entitas prospek, pendaftaran publik via link referal, unggah foto survei (KTP PIC & lokasi), rangkaian dokumen `Quotation → PO → SO → Work Order`, serta aturan penguncian bertahap yang menjaga agar hanya fase terdepan yang bisa disunting. Setelah fondasi itu berdiri, kebutuhan berikutnya adalah memastikan **alur kerja sehari-hari terasa mulus** bagi tiga peran: marketing (mengelola prospek), sales (menerbitkan dokumen), dan tim delivery/NOC (mengeksekusi pemasangan).

Pekerjaan 17 Juli menjawab kebutuhan tersebut. Halaman detail prospek diperluas menjadi satu layar tunggal tempat seluruh dokumen lintas-fase ditampilkan, dipratinjau, dan dikelola sesuai status penguncian. Pembuatan Work Order — langkah paling akhir yang dilakukan tim lapangan — kini otomatis terisi (prefill) dengan data alamat, koordinat, dan PIC yang sudah pernah dimasukkan pelanggan pada tahap awal, sehingga mengurangi salah ketik dan mempercepat penjadwalan pemasangan. Terakhir, sebuah bug kecil namun mengganggu pada pratinjau dokumen SO publik diperbaiki agar dokumen yang tampil benar-benar dokumen SO, bukan salah dirender sebagai PO.

---

### 🖥️ Frontend — Berkas yang Berubah

- [frontend/src/app/pages/services/prospect/detail.jsx](frontend/src/app/pages/services/prospect/detail.jsx)
  - **Deskripsi**: Perluasan lanjutan halaman detail prospek (perubahan terbesar sepanjang issue ini — ±1.200 baris tersentuh). Halaman ini menjadi pusat kendali funnel: menampilkan data survei & foto prospek, seluruh dokumen `Quotation → PO → SO → Work Order` dalam tab/seksi terpisah, tombol aksi (Buat/Edit/Hapus/Pratinjau) yang muncul kondisional mengikuti hasil `computeProspectPhase` (dari [phaseLock.js](frontend/src/app/pages/services/prospect/phaseLock.js)), serta penanda status **"Terkunci"** pada fase yang sudah final atau sudah dilewati. Penyempurnaan hari ini mempererat integrasi modal pratinjau dokumen (Quotation/WO) dan menu aksi dokumen ([DocumentActionsMenu](frontend/src/components/shared/table/DocumentActionsMenu.jsx)) ke dalam alur detail.
- [frontend/src/app/pages/services/workOrder/create.jsx](frontend/src/app/pages/services/workOrder/create.jsx)
  - **Deskripsi**: Menambahkan **prefill otomatis** pada drawer pembuatan Work Order. Sebelumnya form hanya mengambil `address` & `coordinate` dari prop `customer` dan mengosongkan PIC. Kini form menarik data dari **induk SO** — `so.prospect || so.customer` — dengan rantai fallback ke prop `customer`: `site_address` ← `parent.address`, `site_coordinate` ← `parent.coordinate`, `pic_name` ← `parent.pic_name`, dan `pic_phone` ← `parent.phone`. Dengan begitu data yang sudah pernah diisi pelanggan pada tahap prospek/SO langsung terbawa; tim delivery cukup mengoreksi bila ada perubahan di lapangan, alih-alih mengetik ulang dari nol.
- [frontend/src/app/pages/public/ReviewCustomerSOPage.jsx](frontend/src/app/pages/public/ReviewCustomerSOPage.jsx)
  - **Deskripsi**: **Bug fix pratinjau.** Prop `type` pada `DocumentPreviewModal` diperbaiki dari `"po"` menjadi `"customer-so"`. Sebelumnya, ketika pelanggan membuka pratinjau di halaman review SO publik, modal salah merender dokumen sebagai Purchase Order. Nilai `"customer-so"` kini mengarahkan modal ([DocumentPreviewModal.jsx](frontend/src/components/shared/DocumentPreviewModal.jsx)) ke cabang render dokumen Sales Order yang benar.
- [frontend/src/i18n/locales/en/translations.json](frontend/src/i18n/locales/en/translations.json) · [frontend/src/i18n/locales/id/translations.json](frontend/src/i18n/locales/id/translations.json)
  - **Deskripsi**: Penyesuaian string i18n pendukung (label detail prospek, prefill Work Order, dan pratinjau dokumen) agar seluruh teks baru tersedia dalam dua bahasa.

> **Catatan**: Seluruh berkas WIP lainnya (backend `prospectPhase.service.js`, guard fase pada controller, pendaftaran prospek publik, foto survei, PIC penawaran, komponen dokumen & pratinjau Work Order, `DocumentActionsMenu`, dll.) tetap sama seperti dilaporkan pada [15 Juli](idham_2026-07-15.md) dan [16 Juli](idham_2026-07-16.md), dan semuanya masih menunggu commit.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru (User Capabilities & Bug Fixes)
- **Kemampuan Pengguna/Admin**:
  - Admin kini memiliki satu halaman detail prospek yang lengkap sebagai pusat kendali: melihat data & foto survei, memeriksa seluruh dokumen di setiap fase, mempratinjau dokumen langsung, dan menjalankan aksi yang relevan — semuanya tunduk pada aturan penguncian bertahap sehingga tidak ada aksi terlarang yang tampil.
  - Tim delivery mendapatkan drawer pembuatan Work Order yang sudah terisi otomatis (alamat, koordinat, nama & telepon PIC) dari data pelanggan yang tersimpan, memangkas waktu input dan risiko salah ketik saat menjadwalkan pemasangan.
- **Bug Fix / Solusi Masalah**:
  - Memperbaiki pratinjau dokumen SO pada halaman review publik yang keliru menampilkan tata letak Purchase Order; pelanggan sekarang melihat dokumen Sales Order yang benar saat meninjau sebelum menandatangani.
  - Prefill Work Order menutup celah data yang sebelumnya kosong (PIC) atau hanya bersumber dari satu entitas (customer), sehingga informasi kontak lapangan tidak lagi hilang ketika induknya adalah prospek.
- **Menu/Tombol Baru**: Tidak ada menu/tombol baru yang berdiri sendiri hari ini; fokusnya pada penyempurnaan perilaku tombol & pratinjau yang sudah diperkenalkan pada 15–16 Juli agar konsisten dan bebas galat.

## 📖 Informasi & Tutorial Singkat Fitur
- **Penjelasan Fitur**: Halaman detail prospek berperan sebagai dasbor funnel dokumen pra-deal. Ia memuat dokumen dari keempat fase dan, melalui `computeProspectPhase`, menentukan fase mana yang masih dapat disunting dan mana yang sudah terkunci. Prefill Work Order memanfaatkan hubungan data SO → induk (prospek/customer) untuk menyalin informasi lokasi dan PIC secara otomatis. Perbaikan pratinjau SO memastikan jenis dokumen yang diminta (`customer-so`) diteruskan dengan benar ke komponen pratinjau bersama.
- **Langkah Penggunaan (Tutorial)**:
  1. **Kelola prospek dari satu layar**: buka detail sebuah prospek. Telusuri seksi Quotation, PO, SO, dan Work Order. Tombol Buat/Edit/Hapus hanya muncul pada fase yang belum terkunci; fase yang sudah final/dilewati ditandai "Terkunci".
  2. **Pratinjau cepat**: klik nomor dokumen (Quotation/WO) untuk membuka modal pratinjau tanpa berpindah halaman.
  3. **Buat Work Order dengan prefill**: dari sebuah SO, buka drawer "Buat Work Order". Perhatikan bahwa Alamat Site, Koordinat, Nama PIC, dan Telepon PIC sudah terisi otomatis dari data induk SO. Koreksi seperlunya, tentukan `target_date`, lalu simpan.
  4. **Tinjau SO publik**: pada halaman review SO publik, buka pratinjau — dokumen kini tampil sebagai Sales Order yang benar (bukan PO) sebelum pelanggan menandatangani.
