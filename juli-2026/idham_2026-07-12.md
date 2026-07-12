# 📝 Daily Work Report - Idham (2026-07-12)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Management — Konversi Prospek → Customer + Laporan Funnel + Integrasi WO ke Customer Detail

## 📅 Laporan Harian - 12 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Seluruh pekerjaan hari ini masih berstatus WIP.

---

### 🏗️ Konteks & Arsitektur 

 **tahap puncak siklus prospek**: prospek yang sudah memiliki minimal satu Customer SO bertanda tangan (`status = signed`) dapat dikonversi menjadi customer resmi (Partner). Konversi bersifat atomik — gagal di tengah jalan tidak boleh meninggalkan data setengah jadi.

Selain konversi, perubahan ini juga menghadirkan:
- **Laporan funnel prospek** (FR-7): ringkasan status, konversi per PIC, dan rekap alasan lost
- **Kartu histori prospek** di halaman detail customer: menampilkan dari prospek mana customer ini berasal
- **Tab Work Order** di halaman detail customer: list WO + tombol buat WO dari SO signed
- Navigasi sidebar lengkap: Prospect, Work Order

---

### 🏗️ Backend — Konversi Atomik (prospect.service.js)

- `backend/src/services/prospect.service.js` (+354 baris tambahan hari ini)
  - **Deskripsi**: Empat fungsi baru:

    **`countSignedSOByProspect(prospectId)`**
    - Hitung jumlah CustomerSO milik prospek dengan `status = 'signed'`. Dipakai sebagai guard di controller konversi — konversi hanya boleh jika minimal 1 SO sudah signed.

    **`findProspectByConvertedCustomer(partnerId)`**
    - Cari prospek yang pernah dikonversi menjadi Partner tertentu (via `converted_customer` ref). Dipakai untuk kartu histori di halaman detail customer. Return `null` jika customer bukan hasil konversi.

    **`convertProspectToCustomer(prospect, partnerData, adminId)`**
    - Fungsi utama konversi — **pola transaction + fallback kompensasi manual** (identik dengan `createWorkOrderWithTicket` di Fase 1):

      *Jalur transaksional (MongoDB replica set)*: `session.withTransaction()` menjalankan `applyConversion` dalam satu transaction. Commit dan rollback otomatis.

      *Jalur fallback (standalone MongoDB)*: Tanpa session. Gunakan object `progress` untuk melacak apa yang sudah disimpan. Jika langkah berikutnya gagal, `undoConversion` menghapus data yang sudah terbentuk.

      **`applyConversion`** (langkah-langkah dalam satu transaction/session):
      1. Buat `Partner` baru dari `partnerData` → simpan → `progress.partner = partner`
      2. `CustomerQuotation.updateMany({ prospect: prospect._id }, { $set: { customer: partner._id } })` — link semua quotation ke Partner baru
      3. `CustomerPO.updateMany(...)` — idem untuk PO
      4. `CustomerSO.updateMany(...)` — idem untuk SO
      5. Update Prospect: `status = 'won'`, `converted_customer = partner._id`, `converted_at`, `converted_by`

      **`undoConversion`** (kompensasi manual saat fallback gagal di tengah jalan):
      1. Hapus Partner yang baru dibuat (via `collection.deleteOne` langsung, tanpa middleware)
      2. Lepas relink dokumen: `updateMany` dengan `{ $unset: { customer: 1 } }` pada filter `{ prospect, customer: partnerId }`
      3. Pulihkan status prospek ke status semula

    **`buildFunnelReport()`**
    - Laporan funnel via **3 aggregation pipeline MongoDB** yang dijalankan paralel (`Promise.all`):
      1. **Status counts**: Group by `status`, hitung `count`, `avgAgeDays` (rata-rata umur dalam hari dari `created_at`), `oldestDays` (prospek tertua per status)
      2. **Konversi per PIC**: Group by `created_by` (admin yang membuat prospek), hitung `total`, `won`, `lost`, `open`. Lookup ke koleksi `admins` untuk nama. Sort by total desc.
      3. **Rekap alasan lost**: Group by `lost_reason`, hitung `count`, ambil 5 contoh prospek per alasan. Sort by count desc.
      - Return: `{ total, statusCounts, perPic, lostReasons }`

---

### 🔧 Backend — Controller Baru (prospect.controller.js)

- `backend/src/controllers/prospect.controller.js` (+88 baris tambahan hari ini)
  - **Deskripsi**: Tiga handler baru:

    **`convertProspect`**
    - **Guard 1**: Prospect harus ada, belum dikonversi (`converted_customer` kosong)
    - **Guard 2**: `countSignedSOByProspect` harus > 0 — konversi hanya boleh setelah ada SO signed (keputusan desain terkunci #1)
    - Ambil `partnerData` dari `req.body` via `pick(PARTNER_CONVERT_FIELDS)`. Field Partner yang wajib: `type`, `npwp`, `area`, `coordinate`, `address`
    - Prefill otomatis dari data prospek untuk field yang tidak diisi ulang: `name`, `phone`, `email`, `address`, `city`
    - Set `reseller = false` (hasil konversi = pelanggan bisnis, bukan reseller)
    - Panggil `convertProspectToCustomer`. Kembalikan `partner._id`, `partner_id`, `name` sebagai response.

    **`readProspectByCustomer`**
    - Cari customer (Partner) berdasarkan `req.params.id`
    - Panggil `findProspectByConvertedCustomer(customer._id)`
    - Kembalikan data prospek asal (atau `null` jika bukan hasil konversi). Dipakai untuk kartu histori di detail customer.

    **`funnelReportProspect`**
    - Panggil `buildFunnelReport()`, kembalikan `data` mentah. Frontend yang memformat tampilan.

---

### 🛣️ Backend — Route & Privilege Baru

- `backend/src/routes/prospect.route.js` (+3 endpoint baru)
  - **Deskripsi**:
    - `GET /prospect/by-customer/:id` — privilege `prospect.read` — Histori prospek di detail customer
    - `POST /prospect/convert/:id` — privilege `prospect.convert` — Eksekusi konversi
    - `GET /prospect/funnel-report` — privilege `prospect.list` — Data laporan funnel

- `backend/src/config/privilege.json` (+19 baris)
  - **Deskripsi**: Tambah dan lengkapi grup privilege baru:
    - `prospect.convert: "PROSPECT_CONVERT"` — ditambahkan ke grup `prospect` yang sudah ada
    - Grup `work_order`: read, create, list, assign, complete
    - Grup `customer_so`: `withoutPo: "CUSTOMER_SO_WITHOUTPO"` (izin buat SO tanpa PO) + `send: "CUSTOMER_SO_SEND"` (izin kirim SO ke customer)

---

### 🎨 Frontend — Form Konversi

- `frontend/src/app/pages/services/prospect/convert.jsx` [NEW] (+289 baris)
  - **Deskripsi**: `ProspectConvertDrawer` — side drawer dari kiri layar, form pelengkap data Partner saat konversi:

    **Schema validasi inline (`convertSchema` Yup)**:
    - Wajib: `name`, `phone`, `type`, `npwp`, `area`, `coordinate`, `address`
    - Opsional: `email`, `ktp`, `city`, `notes`

    **Prefill otomatis**: Saat drawer dibuka (`useEffect` watch `show + prospect`), reset form dengan data dari prospek yang ada (`name`, `phone`, `email`, `address`, `city`, `notes`). Field Partner yang tidak ada di Prospek (`type`, `npwp`, `ktp`, `area`, `coordinate`) dibiarkan kosong untuk diisi manual.

    **Field yang ditampilkan**:
    - `name` — Nama perusahaan (pre-filled dari prospek)
    - `phone` — Telepon (pre-filled)
    - `type` — Tipe customer (`InputCustomerTypeSelect`)
    - `npwp` — Nomor NPWP (wajib untuk Partner)
    - `ktp` — KTP PIC (opsional)
    - `area` — Area layanan (`InputCustomerAreaSelect`)
    - `coordinate` — Koordinat GPS (`InputMap` dengan peta interaktif)
    - `address` — Alamat site (pre-filled)
    - `email`, `city`, `notes`

    **Submit**: `POST /prospect/convert/:id` → jika sukses, panggil `onConverted(partner)` → parent mengarahkan ke halaman customer baru.

    **Error handling**: Server-side field errors di-map ke form via `setError` per field. Error kode (`NO_SIGNED_SO`, `ALREADY_CONVERTED`) diterjemahkan via `prospect.error.*` i18n key.

---

### 📊 Frontend — Halaman Laporan Funnel

- `frontend/src/app/pages/services/prospect/report.jsx` [NEW] (+229 baris)
  - **Deskripsi**: `ProspectFunnelReportPage` (`/services/prospect/report`) — laporan funnel sederhana tanpa grafik interaktif (FR-7):

    **Kartu ringkasan status** (7 kartu): Satu kartu per status funnel (`new`, `contacted`, `quoted`, `negotiation`, `won`, `lost`, `expired`). Tiap kartu menampilkan: jumlah prospek, rata-rata umur dalam hari (`avgAgeDays`), prospek tertua (`oldestDays`), badge warna status. Layout grid responsif.

    **Tabel konversi per PIC**: Nama admin/employee + `admin_id`, total prospek yang dibuat, won, lost, masih aktif (`open = total - won - lost`), win rate (%). Diurutkan dari PIC dengan total terbanyak.

    **Tabel rekap alasan lost**: Alasan (text dari `lost_reason`), jumlah (`count`), dan contoh prospek (link ke detail, max 5 per alasan). Diurutkan dari alasan paling sering.

    **Helper `winRate(row)`**: Kalkulasi persentase `won/total`, return `-` jika `total = 0`.

    Semua data di-fetch sekali dari `GET /prospect/funnel-report` saat halaman dimuat.

---

### 🎨 Frontend — Integrasi di Halaman Detail Customer

- `frontend/src/app/pages/services/customerManagement/detail.jsx` (+182 baris)
  - **Deskripsi**: Dua penambahan ke halaman detail customer:

    **Kartu "Asal Prospek" (originProspect)**:
    - Fetch `GET /prospect/by-customer/:id` saat halaman dimuat (hanya jika punya `prospect.read`)
    - Jika customer adalah hasil konversi, tampilkan card info dengan: `prospect_id`, nama prospek, tanggal konversi, link ke halaman detail prospek
    - Jika bukan hasil konversi, card tidak ditampilkan sama sekali

    **Tab Work Order (Fase 4 integration)**:
    - State: `woList`, `loadingWO`, `createWOTarget`
    - `fetchWOList()` — `GET /work-order/list?customer=:id` (hanya jika `work_order.list`)
    - Tab baru **"Work Order"** di `TabGroup` halaman detail: list WO customer dengan kolom nomor, status badge, tim, target date
    - Tombol **Buat Work Order** muncul di baris SO yang sudah `signed` (jika punya `work_order.create`) — membuka `CreateWorkOrderDrawer` dengan data SO yang dipilih (`createWOTarget`)
    - Import: `CreateWorkOrderDrawer`, `getWorkOrderStatusOption`

---

### 🗺️ Frontend — Navigasi & Router

- `frontend/src/app/navigation/services.js` (+14, net baris)
  - **Deskripsi**: Tambah dua item navigasi sidebar di bawah Services:
    - **Prospect** — icon `MdOutlinePersonSearch`, path `/services/prospect`, role `prospect.list`
    - **Work Order** — icon `MdOutlineEngineering`, path `/services/work-order`, role `work_order.list`

- `frontend/src/app/router/services/prospectRoute.jsx` (+9 baris)
  - **Deskripsi**: Tambah route `/prospect/report` → `ProspectFunnelReportPage` (lazy import), privilege `prospect.list`

- `frontend/src/app/router/protected.jsx` (+4 baris)
  - **Deskripsi**: Spread `prospectRoute` dan `workOrderRoute` ke dalam konfigurasi route protected. Semua route Prospect dan Work Order kini aktif.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **Siklus lead-to-customer selesai penuh**: Dari pencatatan prospek (Fase 1) → dokumen (Fase 2) → konversi resmi (Fase 3) → delivery (Fase 4 Work Order). Seluruh alur kini tersedia dalam satu sistem.
- **Atomisitas konversi**: Prospek tidak akan pernah berada dalam state setengah-jadi (Partner terbuat tapi dokumen belum terhubung). Dua jalur (transaction + kompensasi manual) memastikan ini di semua environment deployment.
- **Traceability penuh**: Admin bisa melihat dari prospek mana seorang customer berasal, lengkap dengan tanggal konversi dan link ke histori prospek.
- **Laporan funnel**: Manajemen dapat melihat distribusi status prospek, performa per PIC (win rate), dan alasan loss terbanyak — tanpa perlu laporan terpisah di luar sistem.
- **Work Order dapat dibuat langsung dari detail customer**: Tab WO di halaman customer menampilkan semua WO dan tombol buat WO muncul di baris SO yang sudah signed — satu halaman untuk semua.
- **Privilege granular**: `prospect.convert` terpisah dari `prospect.update` — hanya admin tertentu yang bisa mengeksekusi konversi. `customer_so.send` dan `customer_so.withoutPo` untuk kontrol alur SO yang lebih ketat.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Alur Konversi Prospek → Customer**:
1. Prospek harus punya minimal 1 Customer SO dengan status `signed`
2. Tombol **Konversi ke Customer** muncul di halaman detail prospek (privilege `prospect.convert`)
3. `ProspectConvertDrawer` terbuka — data prospek ter-prefill, PIC melengkapi field wajib Partner: tipe customer, NPWP, area layanan, koordinat GPS
4. Submit → sistem membuat Partner baru secara atomik + melink semua dokumen (Quotation/PO/SO) ke Partner baru + set prospek `status = won`
5. Redirect ke halaman detail customer baru
6. Di halaman detail customer, kartu "Asal Prospek" menampilkan histori dari prospek yang dikonversi

**Laporan Funnel**:
- Buka menu **Prospect** → tombol **Laporan** di header
- Lihat distribusi status, perbandingan performa antar PIC, dan alasan-alasan deal gagal
