# 📝 Daily Work Report - Idham (2026-07-13)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123
- **Judul Issue**: Customer Management — Fase 4: Work Order + Integrasi Tiket Pemasangan

## 📅 Laporan Harian - 13 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Seluruh pekerjaan hari ini masih berstatus WIP.

---

### 🏗️ Konteks & Arsitektur Fase 4

Fase 4 menghadirkan **Work Order (WO)** sebagai dokumen operasional antara sales dan tim delivery. WO dibuat dari SO yang sudah `signed` dan customernya sudah terisi (artinya prospek sudah dikonversi). Saat WO dibuat, sistem **otomatis membuat Tiket Pemasangan** — sehingga alur `Ticket` existing tidak perlu diubah dan akun pelanggan tetap dibuat melalui tiket (sesuai aturan bisnis existing).

WO tidak memuat harga — hanya spesifikasi teknis (`tech_spec`) per item layanan — agar aman dicetak dan dibagikan ke tim lapangan.

**Prinsip kunci Fase 4 (keputusan terkunci):**
- WO tanpa konversi tidak mungkin: `createWorkOrder` menolak SO yang `customer`-nya kosong.
- WO selesai (`done`) wajib upload BAA/BAST ter-tanda tangan (`baa_file`).
- `done_at` dicatat sebagai hook billing masa depan.

---

### 🏗️ Backend — Model WorkOrder

- `backend/src/models/workOrder.model.js` [NEW] (+151 baris)
  - **Deskripsi**: Schema Mongoose untuk dokumen operasional Work Order:

    **Format nomor**: `wo_number` format `WO/YYYY/ROMAN/NNNN` (misal `WO/2026/VII/0001`) — konsisten dengan konvensi autoIncrement existing; diisi via plugin `autoIncrementPlugin`.

    **Referensi dokumen**:
    - `so` (ref CustomerSO, **wajib**) — SO yang memicu pembuatan WO
    - `customer` (ref Partner, **wajib**) — dicopy dari `so.customer` saat create
    - `ticket` (ref Ticket) — ref dua arah ke tiket pemasangan yang dibuat otomatis

    **Data lapangan**:
    - `site_address` (wajib), `site_coordinate` (opsional)
    - `pic_lapangan`: sub-schema `{ name, phone }` — PIC yang menerima instalasi
    - `service_items`: array `{ name, qty, tech_spec }` — **tanpa harga** (keputusan terkunci #5)
    - `target_date`, `assigned_team` (ref Admin), `notes`

    **Status enum**: `created` → `in_progress` → `done` / `cancelled`

    **Field penyelesaian**: `baa_file` (nama file BAA di MinIO), `done_at` (timestamp selesai)

    Plugin: `softDelete` (soft delete konsisten) + `autoIncrementPlugin` (wo_number)

- `backend/src/models/ticket.model.js` (+5 baris)
  - **Deskripsi**: Tambah satu field:
    - `work_order: { type: ObjectId, ref: 'WorkOrder' }` — ref dua arah; tiket tahu dari WO mana dia lahir. Dipakai untuk badge "Dari Work Order {wo_number}" di halaman detail tiket.

---

### 🛠️ Backend — Service Layer (workOrder.service.js)

- `backend/src/services/workOrder.service.js` [NEW] (+197 baris)
  - **Deskripsi**: Lima fungsi:

    **`findWorkOrderById(id)`**
    - Query by `_id`, populate: `so`, `customer`, `ticket`, `assigned_team`, `created_by`

    **`findListWorkOrderForTable(params)`**
    - List untuk DataTable dengan pagination, search, sort, filter `status`. Mendukung filter tambahan dari luar (misal `assigned_team` untuk "mine" filter).

    **`findWorkOrdersByParent(filter)`**
    - List WO berdasarkan filter induk (`{ customer }` atau `{ so }`). Populate ringkas untuk ditampilkan di tab halaman parent.

    **`updateWorkOrderById(id, updateData)`**
    - Update data WO termasuk status, `assigned_team`, `baa_file`, `done_at`. Return dokumen setelah update.

    **`createWorkOrderWithTicket(woData, ticketData)`**
    - Fungsi inti — **pola transaction + fallback kompensasi manual** (konsisten dengan `convertProspectToCustomer`):

      *Jalur transaksional*: `session.withTransaction()` → buat WorkOrder → buat Ticket (via service existing) → set `ticket.work_order = wo._id` → set `wo.ticket = ticket._id` → commit.

      *Jalur fallback (standalone)*: Object `progress` melacak apa yang sudah disimpan. Jika salah satu langkah gagal, undo: hapus tiket dan/atau hapus WO yang sudah terbentuk.

      Return: `{ workOrder, ticket }` jika sukses. `{ error }` jika gagal setelah undo.

---

### 🔧 Backend — Controller (workOrder.controller.js)

- `backend/src/controllers/workOrder.controller.js` [NEW] (+249 baris)
  - **Deskripsi**: Enam handler:

    **`createWorkOrder`**
    - Validasi `so`: harus ada, berstatus `signed` (atau SO lama via flag `complete + signed_at`), `customer` terisi
    - Helper `sanitizeServiceItems`: buang semua field harga dari item input (nama, qty, tech_spec saja)
    - Fallback service_items: jika kosong, copy dari `so.line_items` — nama + qty saja, tanpa harga
    - Buat `ticketData` dengan prefill: `type: 'installation'`, `partner: customer._id`, `name/phone` dari `pic_lapangan`, `address/coordinate` dari WO, `description` ringkasan `service_items`
    - Panggil `createWorkOrderWithTicket(woData, ticketData)`
    - Kirim notifikasi Telegram (via `TelegramNotifTicketCreate`) untuk tiket yang baru dibuat
    - Response: `{ wo_number, ticket_id }` untuk konfirmasi ke frontend

    **`listAllWorkOrder`**
    - List semua WO untuk halaman delivery list
    - Query `?mine=1` → filter `assigned_team = req.user._id` — tampilan "ditugaskan ke saya"

    **`listWorkOrderByParent`**
    - List WO berdasarkan query `?customer=` atau `?so=`
    - Validasi customer via `findCustomerById`
    - Dipakai di tab Work Order halaman detail customer

    **`readWorkOrder`**
    - Detail satu WO; return error `WO_NOT_FOUND` jika tidak ada

    **`assignWorkOrder`**
    - Tugaskan tim delivery ke WO → `status = 'in_progress'`
    - Tanpa `admin_id` di body = tugaskan ke diri sendiri (aksi **"Mulai"** untuk tim delivery)
    - Dengan `admin_id` = cari admin by `admin_id`, tugaskan ke orang lain (aksi assign oleh koordinator)
    - Guard: status `done` atau `cancelled` → error `ALREADY_DONE`

    **`completeWorkOrder`**
    - Selesaikan WO dengan upload BAA/BAST ter-tanda tangan
    - Validasi file: wajib ada, ekstensi PDF/JPG/JPEG/PNG, ukuran maks 15 MB
    - Upload ke MinIO via `uploadAppFile`
    - Set: `baa_file`, `done_at = new Date()`, `status = 'done'`

---

### 🛣️ Backend — Route & Registrasi

- `backend/src/routes/workOrder.route.js` [NEW] (+59 baris)
  - **Deskripsi**: Enam route dengan privilege check:
    - `POST /work-order/create` — privilege `work_order.create`
    - `POST /work-order/list-all` — privilege `work_order.list`
    - `GET /work-order/list` — privilege `work_order.list` (list by parent, via query param)
    - `GET /work-order/view/:id` — privilege `work_order.read`
    - `POST /work-order/assign/:id` — privilege `work_order.assign`
    - `POST /work-order/complete/:id` — privilege `work_order.complete`

- `backend/src/app.js` (+4 baris)
  - **Deskripsi**: Registrasi dua route baru di Express app:
    - `import ProspectRoute from './routes/prospect.route.js'`
    - `import WorkOrderRoute from './routes/workOrder.route.js'`
    - `app.use('/api/v1', ProspectRoute)`
    - `app.use('/api/v1', WorkOrderRoute)`

---

### 🔑 Privilege & i18n Backend (Fase 4)

- `backend/src/config/privilege.json` (penambahan grup `work_order`)
  - **Deskripsi**: Lima privilege baru untuk Work Order:
    - `work_order.read`, `work_order.create`, `work_order.list`, `work_order.assign`, `work_order.complete`

- `backend/src/locales/en/translation.json` + `backend/src/locales/id/translation.json`
  - **Deskripsi**: Tambah grup `workOrder` (9 key):
    - `name`, `notFound`, `failedGetList`, `createFailed`, `editFailed`
    - `created: "Work Order & installation ticket created successfully"`
    - `assigned: "Work Order assigned, now in progress"`
    - `completed: "Work Order completed — BAA stored"`
    - `baaInvalidType`, `baaTooLarge`

---

### 🎨 Frontend — Halaman Work Order

- `frontend/src/app/pages/services/workOrder/index.jsx` [NEW] (+70 baris)
  - **Deskripsi**: `WorkOrderListPage` — halaman list WO untuk tim delivery:
    - Toggle **Semua WO / Ditugaskan ke Saya** (`?mine=1`)
    - DataTable dengan filter status, search nomor WO/customer
    - Tombol Create WO **tidak ada** di halaman ini — WO dibuat dari tab SO di detail customer

- `frontend/src/app/pages/services/workOrder/create.jsx` [NEW] (+281 baris)
  - **Deskripsi**: `CreateWorkOrderDrawer` — side drawer untuk buat WO dari SO signed:
    - Prop `so`: data SO yang dipilih di tab customer detail (prefill otomatis)
    - Field: `site_address` (pre-filled dari customer.address), `site_coordinate`, `pic_name`, `pic_phone`, `target_date`, `notes`
    - `service_items`: array dinamis via `useFieldArray`. Setiap item: `name`, `qty`, `tech_spec`
    - Prefill service_items dari `so.line_items` (nama + qty; PIC melengkapi `tech_spec`)
    - Hint yang muncul di drawer: "WO otomatis membuat tiket pemasangan — akun pelanggan dibuat di sana"

- `frontend/src/app/pages/services/workOrder/detail.jsx` [NEW] (+514 baris)
  - **Deskripsi**: `WorkOrderDetailPage` — halaman detail WO dengan semua aksi:

    **Modal Assign**: Combobox pilih admin/employee + submit `POST /work-order/assign/:id`. Atau tombol **Mulai** (assign ke diri sendiri tanpa modal).

    **Modal Complete + BAA**: Upload file (PDF/JPG/PNG), preview nama file, field catatan opsional, submit `POST /work-order/complete/:id` dengan `multipart/form-data`.

    **Panel info**: nomor WO, SO asal (link), customer (link), status badge, tim yang ditugaskan, target date, PIC lapangan.

    **Service items table**: nama, qty, tech_spec — tanpa kolom harga.

    **Link tiket**: link ke halaman detail tiket yang dibuat otomatis.

    **Tombol Print**: link ke `/services/work-order/print/:id`.

- `frontend/src/app/pages/services/workOrder/print.jsx` [NEW] (+182 baris)
  - **Deskripsi**: `WorkOrderPrintPage` — halaman print untuk tim delivery:
    - Render WO lengkap: nomor, customer, site, PIC lapangan, service_items + tech_spec
    - **Tanpa harga sama sekali** (keputusan terkunci #5)
    - CSS `@media print` untuk menyembunyikan navigasi dan tombol; tombol print di layar via `window.print()`

- `frontend/src/app/pages/services/workOrder/schema/columns.jsx` [NEW] (+117 baris)
  - **Deskripsi**: Kolom DataTable halaman list WO: nomor WO, customer, tim yang ditugaskan, status badge (dengan filter dropdown), target date, aksi (link detail).

- `frontend/src/app/pages/services/workOrder/schema/statusOptions.js` [NEW] (+15 baris)
  - **Deskripsi**: 4 konstanta status WO dengan warna badge: `created` (gray), `in_progress` (blue), `done` (green), `cancelled` (red). Helper `getWorkOrderStatusOption`.

---

### 🎨 Frontend — Integrasi WO di Customer Detail

- `frontend/src/app/pages/services/customerManagement/detail.jsx` (+tab WO + kartu originProspect)
  - **Deskripsi**: Dua penambahan ke halaman detail customer yang sudah ada:

    **Tab "Work Order"** (integrasi Fase 4):
    - State: `woList`, `loadingWO`, `createWOTarget`
    - `fetchWOList()` — `GET /work-order/list?customer=:id` (hanya jika punya `work_order.list`)
    - Tab baru **"Work Order"** di `TabGroup` halaman detail: list WO customer dengan kolom nomor, status badge, tim, target date
    - Tombol **Buat Work Order** muncul di baris SO yang sudah `signed` (jika punya `work_order.create`) — membuka `CreateWorkOrderDrawer` dengan data SO yang dipilih sebagai `createWOTarget`

    **Kartu "Asal Prospek"** (integrasi Fase 3 + Fase 4):
    - Fetch `GET /prospect/by-customer/:id` saat halaman dimuat (hanya jika punya `prospect.read`)
    - Jika customer adalah hasil konversi, tampilkan card info dengan: `prospect_id`, nama prospek, tanggal konversi, link ke halaman detail prospek

---

### 🗺️ Frontend — Router & Navigasi WO

- `frontend/src/app/router/services/workOrderRoute.jsx` [NEW] (+31 baris)
  - **Deskripsi**: Tiga route Work Order (lazy import):
    - `/work-order` → `WorkOrderListPage`, privilege `work_order.list`
    - `/work-order/view/:id` → `WorkOrderDetailPage`, privilege `work_order.read`
    - `/work-order/print/:id` → `WorkOrderPrintPage`, privilege `work_order.read`

- `frontend/src/app/router/protected.jsx` (+2 baris)
  - **Deskripsi**: Spread `workOrderRoute` ke dalam konfigurasi route protected.

- `frontend/src/app/navigation/services.js` (+7 baris)
  - **Deskripsi**: Tambah item navigasi sidebar:
    - **Work Order** — icon `MdOutlineEngineering`, path `/services/work-order`, role `work_order.list`

---

### 🎨 Frontend — i18n Work Order

- `frontend/src/i18n/locales/en/translations.json` + `frontend/src/i18n/locales/id/translations.json`
  - **Deskripsi**: Tambah grup `workOrder` (35+ key):
    - Label umum: `title`, `list`, `number`, `empty`, `assignedTeam`, `targetDate`, `picLapangan`, `ticket`, `baaFile`, `doneAt`, `serviceItems`, `qty`, `techSpec`
    - Aksi: `filterAll`, `filterMine`, `assign`, `start`, `complete`, `completeHint`, `print`, `createTitle`, `createAction`, `createHint`
    - Toast/feedback: `createdToast`, `assignedToast`, `completedToast`, `createFromSOHint`, `originProspect`
    - Status: `status.created/in_progress/done/cancelled`
    - Error kode: `SO_NOT_ELIGIBLE`, `BAA_REQUIRED`, `ALREADY_DONE`, `WO_NOT_FOUND`, `PARENT_REQUIRED`
    - Navigation: key `workOrder: "Work Order"` di section navigation

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

- **Rantai dokumen selesai end-to-end**: Quotation → PO (WIP Fase 2) → SO → WO → Tiket → BAA. Alur penuh dari penawaran hingga selesai instalasi kini tersedia dalam satu sistem.
- **Tim delivery punya dashboard sendiri**: Work Order list (filter "mine") memberi tim delivery tampilan WO yang ditugaskan ke mereka tanpa melihat dokumen komersial.
- **WO tanpa harga**: Print PDF WO aman dibagikan ke kontraktor/subkontraktor karena tidak memuat angka rupiah.
- **Tiket pemasangan otomatis**: Tidak ada langkah manual membuat tiket — WO create mengurusnya. Akun pelanggan tetap dibuat di alur tiket existing tanpa perubahan.
- **BAA wajib untuk selesaikan WO**: Tidak ada WO yang bisa ditutup tanpa bukti serah terima. `done_at` menjadi hook billing.

---

## 📖 Informasi & Tutorial Singkat Fitur

**Alur Work Order**:
1. Di halaman detail customer, buka tab **SO** → SO dengan status `Signed` menampilkan tombol **Buat WO**
2. `CreateWorkOrderDrawer` terbuka — data SO ter-prefill, PIC melengkapi: site address/coordinate, PIC lapangan, target date, dan `tech_spec` per item layanan
3. Submit → WO + Tiket Pemasangan terbuat dalam satu operasi atomik → notifikasi Telegram
4. Tim delivery buka menu **Work Order** → filter "Ditugaskan ke Saya" atau aksi "Mulai" di detail WO
5. Setelah instalasi selesai, klik **Selesai + BAA** → upload BAA/BAST ter-tanda tangan → WO `done`
