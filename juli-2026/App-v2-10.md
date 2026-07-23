# 📝 Daily Work Report - Idham (2026-07-10)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Management — Modul Work Order (Fase 1: Backend + Frontend Lengkap)

## 📅 Laporan Harian - 10 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Seluruh pekerjaan hari ini masih berstatus WIP — Work Order modul selesai penuh (untracked, belum di-stage).

---

### 🏗️ Konteks & Arsitektur Work Order

Work Order (WO) adalah dokumen **operasional** yang melengkapi rantai dokumen komersial. Posisinya dalam alur bisnis ISP:

```
Customer SO (signed) → Work Order → Tiket Pemasangan → Akun Pelanggan
```

**Perbedaan mendasar dari SO**: SO adalah dokumen komersial berisi harga; WO adalah dokumen delivery untuk tim lapangan — **tanpa informasi harga sama sekali** (keputusan desain terkunci). WO berisi instruksi teknis: alamat site, spesifikasi layanan (bandwidth, media, IP), PIC lapangan, target tanggal, dan BAA/BAST.

**Syarat pembuatan WO**:
- SO sudah berstatus `signed` (atau `complete = true` + `signed_at` terisi untuk data lama)
- SO sudah punya `customer` (ref Partner) — berarti prospek sudah dikonversi menjadi customer resmi

**Integrasi dua arah WO ↔ Tiket**:
- Saat WO dibuat → tiket pemasangan (`type: installation`) otomatis dibuat dan ref disimpan di kedua dokumen (`wo.ticket` dan `ticket.work_order`)
- Assign teknisi di WO → WO masuk `in_progress`
- Upload BAA/BAST di WO → WO masuk `done` + `done_at` terisi (sebagai hook billing di masa depan)

---

### 🏗️ Backend (Untracked, File Baru)

- `backend/src/models/workOrder.model.js` [NEW] (+151 baris)
  - **Deskripsi**: Model Work Order — dokumen operasional delivery tanpa field harga. Field utama:
    - `wo_number` — Nomor auto-format `WO/{YYYY}/{bulan_romawi}/{seq_4_digit}` (contoh: `WO/2026/VII/0001`). Dibentuk oleh pre-save hook setelah `autoIncrementPlugin` mengisi `seq`. Bulan ditulis angka romawi menggunakan array konversi lokal.
    - `so` (ref CustomerSO, **wajib**) — SO induk yang menjadi dasar WO
    - `customer` (ref Partner, **wajib**) — Customer yang akan dilayani; selalu Partner (bukan Prospect) karena WO hanya bisa dibuat pasca konversi
    - `ticket` (ref Ticket) — Tiket pemasangan yang dibuat otomatis bersamaan dengan WO; ref dua arah
    - `site_address` (**wajib**), `site_coordinate` — Lokasi instalasi; default dari data customer jika tidak diisi
    - `pic_lapangan` (sub-schema: `name`, `phone`) — PIC di lapangan yang bisa berbeda dari PIC di customer
    - `service_items` (array `ServiceItemSchema`: `name`, `qty`, `tech_spec`) — Item layanan yang dikerjakan; dicopy dari `line_items` SO tapi **tanpa harga**. `tech_spec` diisi teknisi (bandwidth, media FO/wireless, IP, dll.)
    - `target_date` — Target tanggal penyelesaian
    - `assigned_team` (ref Admin) — Teknisi/tim yang ditugaskan
    - `status` (enum: `created → in_progress → done / cancelled`) — 4 tahap siklus hidup
    - `baa_file` — Nama file BAA/BAST ter-ttd yang diunggah saat selesai (PDF/JPG/PNG, max 15 MB). Prasyarat untuk status `done`.
    - `done_at` — Timestamp penyelesaian; akan menjadi trigger billing di Fase berikutnya
    - `notes`, `created_by` (ref Admin), `created_at`
    - Soft delete via `mongoose-delete`

- `backend/src/services/workOrder.service.js` [NEW] (+197 baris)
  - **Deskripsi**: Service layer Work Order — 5 fungsi utama:
    - `findWorkOrderById(id)` — Cari via `wo_number` string terlebih dahulu, fallback ke ObjectId. Populate: `so`, `customer`, `ticket`, `assigned_team`, `created_by`.
    - `findListWorkOrderForTable(params)` — Query DataTable dengan filter, pagination, sorting. Paksa `wo_number` dan `assigned_team` selalu ikut di-select.
    - `findWorkOrdersByParent(parentFilter)` — Cari semua WO milik satu induk (`{ customer }` atau `{ so }`); dipakai untuk tab WO di detail customer.
    - `updateWorkOrderById(id, data)` — Update via `findOneAndUpdate` dengan `runValidators: true`.
    - `createWorkOrderWithTicket(woData, ticketData)` — **Fungsi utama**. Buat WO + tiket pemasangan dalam **satu Mongoose transaction** untuk atomisitas. Karena banyak deployment ISP menggunakan MongoDB standalone (tanpa replica set yang mendukung multi-document transaction), ada **dua jalur**:
      - **Jalur transaksional** (replica set): `session.withTransaction()` — commit dan rollback otomatis
      - **Jalur fallback** (standalone): Tidak menggunakan session, tapi melacak `progress` (apa yang sudah disimpan). Jika langkah kedua gagal, **kompensasi manual** (`undoWorkOrderCreation`) menghapus dokumen yang sudah terbentuk — mencegah data inkonsisten.

- `backend/src/controllers/workOrder.controller.js` [NEW] (+249 baris)
  - **Deskripsi**: Controller Work Order — 6 handler asyncHandler:
    - `createWorkOrder` — Validasi SO: harus `status === 'signed'` atau (`complete = true` + `signed_at` terisi untuk data lama) dan `customer` terisi. Ambil `service_items` dari form atau copy dari `line_items` SO (nama + qty saja, tanpa harga — disanitasi oleh `sanitizeServiceItems` yang membuang field harga). Set `site_address` default dari `customer.address`. Panggil `createWorkOrderWithTicket`. Kirim notifikasi Telegram tiket pemasangan via `TelegramNotifTicketCreate`.
    - `listAllWorkOrder` — List semua WO. Filter `?mine=1` (dari query string) menambah `{ assigned_team: req.user._id }` ke params — untuk tampilan tim delivery yang hanya ingin melihat WO miliknya.
    - `listWorkOrderByParent` — List WO per induk via query param `?customer=` atau `?so=`. Digunakan di tab WO pada halaman detail customer.
    - `readWorkOrder` — Detail WO tunggal dengan populate lengkap.
    - `assignWorkOrder` — Tugaskan teknisi ke WO → status `in_progress`. Jika `req.body.admin_id` tidak ada, tugaskan ke `req.user` sendiri (aksi "Mulai" oleh teknisi). Guard: WO tidak boleh sudah `done` atau `cancelled`.
    - `completeWorkOrder` — Selesaikan WO. **Wajib upload file BAA/BAST** (PDF/JPG/PNG, max 15 MB). File disimpan ke MinIO dengan nama acak. Set `status = 'done'`, `done_at = new Date()`, simpan nama file. Guard: WO tidak boleh sudah `done` atau `cancelled`.

- `backend/src/routes/workOrder.route.js` [NEW] (+59 baris)
  - **Deskripsi**: 6 route API Work Order dengan privilege group baru `work_order.*`:
    - `POST /work-order/create` — `work_order.create`
    - `POST /work-order/list-all` — `work_order.list`
    - `GET /work-order/list` — `work_order.list` (per induk via query param)
    - `GET /work-order/view/:id` — `work_order.read`
    - `POST /work-order/assign/:id` — `work_order.assign`
    - `POST /work-order/complete/:id` — `work_order.complete`

**Backend — Unstaged (file yang dimodifikasi, terkait WO)**:

- `backend/src/models/ticket.model.js` (+5 baris)
  - **Deskripsi**: Tambah satu field ke Ticket model untuk menyimpan ref dua arah ke WO:
    ```js
    work_order: { type: mongoose.Schema.Types.ObjectId, ref: 'WorkOrder' }
    ```
    Diisi otomatis oleh service WO saat WO dibuat. Dipakai untuk menampilkan badge dan link "Dari Work Order {wo_number}" di halaman detail tiket.

---

### 🎨 Frontend (Untracked, File Baru)

- `frontend/src/app/pages/services/workOrder/index.jsx` [NEW] (+70 baris)
  - **Deskripsi**: Halaman list Work Order (`/services/work-order`) untuk tim delivery:
    - **Toggle filter Semua / Ditugaskan ke Saya**: Dua tombol di header yang mengubah `apiUrl` DataTable — `?mine=1` untuk filter WO milik user yang login. State `mineOnly` di-pass sebagai `key` ke `<Datatables>` agar tabel remount dan reload saat toggle.
    - Tidak ada tombol Create di sini — WO dibuat dari halaman detail SO customer.

- `frontend/src/app/pages/services/workOrder/create.jsx` [NEW] (+281 baris)
  - **Deskripsi**: Drawer pembuatan Work Order dari SO signed. Dirender sebagai child komponen di halaman detail customer SO, bukan halaman tersendiri:
    - Prefill `service_items` dari `line_items` SO saat drawer dibuka — tampilkan tabel baris item dengan field `tech_spec` yang bisa diisi (bandwidth, media, IP, dll.)
    - `useFieldArray` dari React Hook Form untuk manajemen baris item (read-only nama + qty, editable tech_spec)
    - Field: `site_address` (wajib, default dari alamat customer), `site_coordinate`, `pic_name`, `pic_phone`, `target_date` (DatePicker), `notes`
    - Validasi via `woSchema` (Yup inline): hanya `site_address` yang wajib
    - Submit ke `POST /work-order/create`

- `frontend/src/app/pages/services/workOrder/detail.jsx` [NEW] (+514 baris)
  - **Deskripsi**: Halaman detail Work Order (`/services/work-order/view/:id`) — halaman paling lengkap di modul ini:

    **Informasi umum**: Tabel key-value dengan link ke customer dan tiket terkait.

    **Badge status**: Di header card, warna sesuai `statusOption.color`.

    **Tabel service items**: Nama, qty, dan tech_spec per baris — dokumen instruksi teknis untuk teknisi.

    **Tombol aksi kontekstual** (guard privilege + status):
    - **Assign Tim** (`work_order.assign`, status bukan done/cancelled): Buka modal dengan `Combobox` pencarian admin untuk dipilih, lalu konfirmasi assign.
    - **Mulai** (`work_order.assign`, status `created` saja): Tombol shortcut assign ke diri sendiri tanpa membuka modal — teknisi klik "Mulai" untuk mulai mengerjakan.
    - **Selesaikan** (`work_order.complete`, status bukan done/cancelled): Buka modal upload BAA/BAST. File wajib ada sebelum tombol submit aktif.
    - **Cetak**: Link ke `/services/work-order/print/:id` — halaman print-friendly.

    **Modal Assign**: Dialog dengan `Combobox` pencarian admin via `/admin/user-select`.

    **Modal Selesaikan**: Dialog dengan file picker untuk BAA/BAST (PDF/JPG/PNG), preview nama file, submit via `multipart/form-data`.

- `frontend/src/app/pages/services/workOrder/print.jsx` [NEW] (+182 baris)
  - **Deskripsi**: Halaman cetak Work Order — halaman khusus print-friendly yang memuat:
    - Header: Nomor WO, tanggal, nama customer, nomor SO terkait
    - Informasi site: alamat, koordinat, PIC lapangan + kontak
    - Tabel service items: nama layanan, qty, spesifikasi teknis
    - Tim yang ditugaskan dan target tanggal
    - **Tidak memuat informasi harga sama sekali** (prinsip desain terkunci FR-6)
    - Tombol Print dan Kembali (tersembunyi saat mencetak via `@media print`)

- `frontend/src/app/pages/services/workOrder/schema/columns.jsx` [NEW] (+117 baris)
  - **Deskripsi**: Definisi kolom DataTable Work Order: `wo_number` (link ke detail), `customer.name`, `status` (badge berwarna + filter select), `assigned_team.name`, `target_date` (filter dateRange), `created_at` (filter dateRange), aksi view.

- `frontend/src/app/pages/services/workOrder/schema/statusOptions.js` [NEW] (+15 baris)
  - **Deskripsi**: Konstanta 4 status Work Order dengan mapping warna Badge:
    - `created` → info (biru), `in_progress` → warning (kuning)
    - `done` → success (hijau), `cancelled` → error (merah)

- `frontend/src/app/router/services/workOrderRoute.jsx` [NEW] (+31 baris)
  - **Deskripsi**: Definisi 3 route lazy-load untuk modul Work Order:
    - `/work-order` (list) → privilege `work_order.list`
    - `/work-order/view/:id` (detail) → privilege `work_order.read`
    - `/work-order/print/:id` (print) → privilege `work_order.read`

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **Alur delivery end-to-end tersedia**: Setelah SO ditandatangani customer, admin bisa membuat Work Order langsung dari halaman detail SO. Tim lapangan mendapat dokumen operasional berisi instruksi teknis tanpa informasi harga.
- **Tiket pemasangan otomatis**: Membuat WO otomatis membuat tiket pemasangan (`type: installation`) yang terhubung dua arah — mencegah tiket ganda dan memastikan data customer sudah tepat sejak awal.
- **Notifikasi Telegram**: Admin mendapat notifikasi tiket pemasangan baru saat WO dibuat — konsisten dengan alur tiket manual.
- **Pemisahan peran yang jelas**: Admin sales membuat WO → tim delivery melihat daftar WO (filter "Ditugaskan ke Saya") → teknisi klik "Mulai" → upload BAA/BAST saat selesai → `done_at` tersimpan sebagai hook billing.
- **Atomisitas data**: Pembuatan WO + tiket dalam satu operasi — tidak akan ada WO tanpa tiket atau tiket tanpa WO. Fallback kompensasi manual untuk deployment standalone MongoDB tanpa replica set.
- **Dokumen cetak**: Halaman print WO tersedia langsung dari detail WO — tim lapangan bisa print instruksi sebelum berangkat ke site.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Alur kerja Work Order end-to-end**:

1. Customer SO ditandatangani digital → status SO berubah menjadi `signed`
2. Admin buka halaman detail customer → Tab "Sales Order" → buka SO yang sudah signed
3. Klik tombol **Buat Work Order** → drawer terbuka
4. Service item ter-prefill dari line item SO; admin/teknisi isi `tech_spec` per item (bandwidth, jenis media, IP yang akan digunakan, dll.)
5. Isi lokasi site, koordinat, PIC lapangan, target tanggal → Submit
6. Sistem buat WO dengan nomor `WO/2026/VII/0001` dan tiket pemasangan otomatis
7. Notifikasi Telegram dikirim ke admin
8. Tim delivery buka `/services/work-order` → toggle "Ditugaskan ke Saya" → lihat daftar WO
9. Teknisi klik **Mulai** di detail WO → WO masuk `in_progress`, teknisi ter-assign
10. Setelah pekerjaan selesai, teknisi klik **Selesaikan** → upload BAA/BAST ter-ttd
11. WO masuk status `done`, `done_at` tersimpan → siap jadi trigger billing
