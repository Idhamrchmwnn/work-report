# 📝 Daily Work Report - Idham (2026-07-08)

---

## 📌 Informasi Issue
- **Nomor Issue**: #123 + Hotfix #118
- **Judul Issue**: Customer Document Management — Implementasi Frontend Lengkap Modul Customer PO & SO + Bug Fix Ticket Controller & Axios Error Handler

## 📅 Laporan Harian - 8 Juli 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Seluruh pekerjaan hari ini masih berstatus WIP melanjutkan backend kemarin — implementasi frontend selesai penuh dan integrasi ke halaman Customer Detail.

---

### 🏗️ Konteks & Lanjutan dari Kemarin

Backend Customer PO dan SO (model, service, controller, route, Telegram, file handler) diselesaikan pada 7 Juli. Hari ini fokus pada sisi frontend: form create/edit, halaman publik tanda tangan, review drawer admin, integrasi tab di Customer Detail, dan registrasi route.

Arsitektur frontend mengikuti pola yang sama dengan modul Vendor:
- **Form create/edit**: Side drawer (`Dialog` dari Headless UI) dari sisi kiri layar
- **Review drawer**: Side drawer dari sisi kanan (review admin, approval, share link)
- **Halaman publik**: Tanpa login, akses via token — `react-pdf` untuk render PDF interaktif
- **Halaman review admin**: Internal, route `/review/customer-po/:id` dan `/review/customer-so/:id`

---

### 🛠️ Pekerjaan Hari Ini — Frontend (Untracked, file baru)**

**Form Create & Edit Customer PO**:

- `frontend/src/app/pages/services/customerPurchaseOrder/create.jsx` [NEW] (+284 baris)
  - **Deskripsi**: Side drawer form untuk membuat Customer PO baru (dokumen PDF yang diunggah customer):
    - Upload zone drag & click — accept `application/pdf` saja, max 15 MB, validasi ukuran dan tipe file di sisi client
    - Preview file terpilih: nama, ukuran (`formatFileSize`), tombol hapus
    - Field: `po_number` (opsional, auto-generate jika kosong), `po_date` (DatePicker), `notes` (textarea)
    - Validasi via `customerPOSchema` (Yup + `yupResolver`)
    - Submit via `multipart/form-data` ke `POST /customer-po/create`
    - Animasi masuk dari kiri layar (konsisten dengan pattern create drawer seluruh aplikasi)

- `frontend/src/app/pages/services/customerPurchaseOrder/edit.jsx` [NEW] (+283 baris)
  - **Deskripsi**: Side drawer form edit Customer PO — identik dengan `create.jsx` namun:
    - Prefill dari data PO yang ada (fetch saat drawer dibuka)
    - Replace dokumen: tampilkan nama file lama, bisa diunggah file baru untuk mengganti
    - Submit via `PATCH /customer-po/:id`

- `frontend/src/app/pages/services/customerPurchaseOrder/CustomerPODocumentPreview.jsx` [NEW] (+83 baris)
  - **Deskripsi**: Komponen preview thumbnail PDF Customer PO — embed di drawer review untuk pratinjau dokumen. Gunakan endpoint `/file/customer-po/:filename` dari backend.

**Form Create & Edit Customer SO**:

- `frontend/src/app/pages/services/customerSalesOrder/create.jsx` [NEW] (+652 baris)
  - **Deskripsi**: Side drawer form paling kompleks — Customer SO yang di-generate dari line item (mirror form Vendor PO). Fitur:
    - `useFieldArray` dari React Hook Form untuk manajemen baris line item yang dinamis (tambah/hapus baris)
    - Tiap baris: nama item, `item_type` (`Combobox` dengan opsi service/goods/contractor dari `ITEM_TYPE_OPTIONS`), satuan, `price_type` (otc/mrc), qty, harga satuan (`InputMoney`)
    - Total otomatis: kalkulasi real-time dari qty × harga satuan, ditampilkan dengan `formatMoney`
    - `total_override` opsional: admin bisa override total jika perlu
    - Field header: `so_number`, `so_date`, `notes`
    - Validasi via `customerSOSchema` (Yup): setiap baris wajib memiliki nama, tipe, harga positif, qty ≥ 1
    - Submit via `POST /customer-so/create`
    - Lampiran opsional: upload file pendukung per SO

- `frontend/src/app/pages/services/customerSalesOrder/edit.jsx` [NEW] (+728 baris)
  - **Deskripsi**: Side drawer edit Customer SO — identik dengan `create.jsx` namun prefill dari data SO yang ada, update via `PATCH /customer-so/:id`. Form terbesar di modul ini karena kombinasi validasi line item + lampiran.

- `frontend/src/app/pages/services/customerSalesOrder/CustomerSODocumentPreview.jsx` [NEW] (+333 baris)
  - **Deskripsi**: Preview Customer SO berupa tabel rincian line item (nama, tipe, qty, harga, subtotal), total keseluruhan, metadata SO (nomor, tanggal, customer, catatan). Tidak berbasis PDF karena SO di-generate dari data — preview di dalam drawer review sebelum dikirimkan ke customer.

**Review Drawer Admin**:

- `frontend/src/app/pages/services/activation/components/CustomerPOReviewDrawer.jsx` [NEW] (+425 baris)
  - **Deskripsi**: Side drawer review Customer PO untuk admin — slide dari kanan layar:
    - Fetch detail PO saat drawer dibuka via `/customer-po/view/:po_number`
    - **Seksi kirim permintaan pratinjau**: Toggle `Checkbox` antara admin default atau pilih admin spesifik via `Combobox` (`/admin/user-select`). Hanya tampil jika PO belum disetujui.
    - Informasi PO: nomor, tanggal, dokumen, jumlah halaman, status, disetujui oleh
    - Share link: `LinkBadge` menuju `/p/customer-po/:token` — muncul setelah PO disetujui
    - Riwayat (timeline): created, approved, signed — dengan warna dan timestamp
    - Footer actions: tombol Reject + Approve (hanya jika belum disetujui dan user punya privilege `serviceActivation.update`)
    - `ConfirmModal` terpisah untuk konfirmasi approve dan reject
    - Dijaga privilege via `useHasPrivilege`

- `frontend/src/app/pages/services/activation/components/CustomerSOReviewDrawer.jsx` [NEW] (+529 baris)
  - **Deskripsi**: Side drawer review Customer SO — lebih panjang dari PO Review Drawer karena menampilkan rincian line item di dalam drawer. Struktur identik: kirim pratinjau, informasi SO, tabel line item + total, share link, timeline, footer actions.

**Komponen Shared**:

- `frontend/src/components/shared/ApprovePOModal.jsx` [NEW] (+168 baris)
  - **Deskripsi**: Modal konfirmasi approval dengan pilihan tanda tangan — dipakai untuk approve Customer PO dan Customer SO. Tiga opsi yang dipilih via radio button:
    - **`'existing'`** — Gunakan tanda tangan yang sudah tersimpan di profil admin (hanya muncul jika `user.sign` ada di Redux store). Admin langsung tanda tangan tanpa menggambar ulang.
    - **`'new'`** — Gambar tanda tangan baru (pad gambar, ekspor sebagai PNG). Tanda tangan baru akan disimpan ke profil.
    - **`'skip'`** — Setujui dokumen tanpa tanda tangan digital. Dokumen disetujui tapi tidak ada tanda tangan yang di-embed.
    - Nilai default: `'existing'` jika admin punya tanda tangan tersimpan, `'new'` jika tidak
    - Memanggil `onConfirm(signOption)` — parent yang menangani logika per opsi

**Halaman Publik (Tanpa Login)**:

- `frontend/src/app/pages/public/PublicCustomerPODocument.jsx` [NEW] (+406 baris)
  - **Deskripsi**: Halaman publik tanda tangan Customer PO — akses via `/p/customer-po/:token`:
    - Fetch data PO via token dari endpoint publik backend
    - Render PDF interaktif menggunakan `react-pdf` (`Document` + `Page` dari `pdfjs`)
    - Jika `complete = true`: tampilkan dokumen bertanda tangan (read-only)
    - Jika belum: tampilkan pad tanda tangan (kanvas `signature_pad` library)
    - Pilih posisi tanda tangan: klik pada halaman PDF — koordinat dinormalisasi ke (0..1) oleh komponen
    - Preview posisi real-time sebelum submit
    - Submit: base64 tanda tangan + koordinat posisi + token → `POST /public-docs/customer-po/sign`
    - `pdf.config.js` di-import untuk setup pdfjs worker
    - Responsif untuk mobile (customer sering membuka via HP)

- `frontend/src/app/pages/public/PublicCustomerSODocument.jsx` [NEW] (+441 baris)
  - **Deskripsi**: Halaman publik tanda tangan Customer SO — akses via `/p/customer-so/:token`:
    - Fetch data SO via token (Customer SO di-generate, bukan upload PDF)
    - Tampilkan rincian SO: line item, total, metadata
    - Pad tanda tangan customer jika belum `complete`
    - Jika SO sudah ditandatangani: tampilkan status selesai + unduhan dokumen

**Halaman Review Internal Admin**:

- `frontend/src/app/pages/public/ReviewCustomerPOPage.jsx` [NEW] (+268 baris)
  - **Deskripsi**: Halaman review PO internal admin di `/review/customer-po/:id` — berbeda dari drawer review (bukan overlay). Halaman penuh untuk review detail PO termasuk preview PDF inline. Diakses dari notifikasi Telegram atau langsung oleh admin.

- `frontend/src/app/pages/public/ReviewCustomerSOPage.jsx` [NEW] (+388 baris)
  - **Deskripsi**: Halaman review SO internal admin di `/review/customer-so/:id` — menampilkan rincian SO beserta kontrol approval, share link, dan status tanda tangan customer. Halaman terpanjang karena memuat preview line item + semua aksi admin.

**Integrasi ke Customer Detail (Staged)**:

- `frontend/src/app/pages/services/customerManagement/detail.jsx` (+774 baris, major rewrite)
  - **Deskripsi**: Halaman detail customer (`/services/customer/view/:id`) direfaktor besar-besaran untuk mendukung navigasi multi-tab menggunakan `TabGroup` / `TabList` / `Tab` / `TabPanel` dari Headless UI:

    **Tab yang ditambahkan**:
    - **Tab "Quotation"**: List quotation customer yang sudah ada (tidak berubah)
    - **Tab "Customer PO"**: List Customer PO dengan tombol Create, tabel data, dan buka `CustomerPOReviewDrawer`
    - **Tab "Customer SO"**: List Customer SO dengan tombol Create, tabel data, dan buka `CustomerSOReviewDrawer`

    **State management yang ditambahkan**:
    - PO: `loadingPO`, `createPOOpen`, `deletePOTarget`, `isDeletingPO`, `isPOReviewOpen`, `selectedPOReview`, `isPOProcessing`
    - SO: `loadingSO`, `createSOOpen`, `deleteSOTarget`, `isDeletingSO`, `isSOReviewOpen`, `selectedSOReview`, `isSOProcessing`

    **Handler yang ditambahkan**:
    - `fetchPOList(customerId)` — fetch list PO customer, update state
    - `fetchSOList(customerId)` — fetch list SO customer, update state
    - Handler approve, reject, delete per tipe dokumen

    **Komponen yang diimport**:
    - `CreateCustomerPODrawer` (alias `create.jsx` Customer PO)
    - `EditCustomerPODrawer` (alias `edit.jsx` Customer PO)
    - `CreateCustomerSODrawer` (alias `create.jsx` Customer SO)
    - `EditCustomerSODrawer` (alias `edit.jsx` Customer SO)
    - `CustomerPOReviewDrawer`, `CustomerSOReviewDrawer`
    - `ApprovePOModal`

**Registrasi Route (Staged)**:

- `frontend/src/app/router/protected.jsx` (+15 baris)
  - **Deskripsi**: Tambah 2 route internal admin:
    - `/review/customer-po/:id` → `ReviewCustomerPOPage` (lazy import)
    - `/review/customer-so/:id` → `ReviewCustomerSOPage` (lazy import)
    - Kedua route dijaga privilege `customerDocument.read`

- `frontend/src/app/router/public.jsx` (+14 baris)
  - **Deskripsi**: Tambah 2 route publik tanpa auth:
    - `/p/customer-po/:token` → `PublicCustomerPODocument` (lazy import)
    - `/p/customer-so/:token` → `PublicCustomerSODocument` (lazy import)
    - Tidak ada auth guard — halaman ini diakses oleh customer eksternal via link

**Terjemahan (Staged)**:

- `frontend/src/i18n/locales/en/translations.json` (+371 baris)
- `frontend/src/i18n/locales/id/translations.json` (+371 baris)
  - **Deskripsi**: Tambah namespace terjemahan `customer.po.*` dan `customer.so.*` — 371 baris setiap bahasa. Kunci yang dicakup:
    - CRUD labels: `customer.po.create`, `customer.po.edit`, `customer.po.delete`, `customer.po.list`
    - Field labels: `customer.po.number`, `customer.po.date`, `customer.po.document.label`, `customer.po.notes`, `customer.po.pages`, `customer.po.pageUnit`
    - Status labels: `customer.po.approvedBy`, `customer.po.signLink`, `customer.po.waitingSign`, `customer.po.completed`
    - Upload hints: `customer.po.uploadDocumentHint`, `customer.po.uploadDocumentSub`, `customer.po.document.invalidType`, `customer.po.document.tooLarge`, `customer.po.document.noFile`
    - History timeline: `customer.po.history.title`, `customer.po.history.created`, `customer.po.history.approved`, `customer.po.history.signed`
    - Auto-number hint: `customer.po.autoNumberHint`
    - Error messages: `customer.po.notFound`, `customer.po.alreadyApproved`, `customer.po.alreadySigned`
    - SO equivalents: semua kunci paralel `customer.so.*` termasuk `customer.so.lineItem.*` (name, type, unit, qty, price, subtotal, total)
    - Item type labels: service, goods, contractor (dalam bahasa Indonesia)

- `frontend/src/components/shared/DocumentPreviewModal.jsx` (+16 baris, staged)
  - **Deskripsi**: Update komponen shared DocumentPreviewModal untuk mendukung tipe dokumen `customer-po` dan `customer-so` — tambah case di switch handler tipe dokumen sehingga modal preview terpusat bisa dipakai untuk PO dan SO customer.

---

### 🐛 Bug Fix Tambahan — Branch `issue-118` (WIP, belum commit)

- `backend/src/controllers/ticket.controller.js` (~+30, -11 baris)
  - **Deskripsi**: Dua bug fix pada controller `createTicket`:

    **Bug Fix 1 — Partner ObjectId Detection**:
    - **Masalah**: Saat `req.body` di-spread ke `ticketData`, field `partner` menjadi string mentah (`partner_id` seperti `"667899018"`), bukan ObjectId. Kondisi lama `if (req.body.partner && !ticketData.partner)` selalu `false` karena string `partner_id` truthy — blok resolusi partner tidak pernah dieksekusi untuk kasus ini, sehingga `ticketData.partner` tersimpan sebagai string bukan ObjectId valid.
    - **Fix**: Ganti kondisi menjadi `if (req.body.partner && !/^[0-9a-fA-F]{24}$/.test(ticketData.partner))` — cek apakah string sudah berbentuk MongoDB ObjectId (24 hex chars). Jika belum, eksekusi blok lookup partner via `findOnePartnerAndBusiness`.

    **Bug Fix 2 — Dedicated Field Lookup Fallback**:
    - **Masalah**: Field `dedicated` pada ticket bisa berasal dari dua koleksi berbeda — `DedicatedInternet` atau `DataAccess` (keduanya muncul di dropdown `partnerServiceSelect` di frontend). Controller sebelumnya hanya mencari di `findOneDedicatedInternetWithDeleted` dan langsung throw error jika tidak ditemukan — menyebabkan error 400 ketika `dedicated` adalah ID dari koleksi `DataAccess`.
    - **Fix**: Tambah fallback — jika `findOneDedicatedInternetWithDeleted` tidak menemukan data, coba `findOneDataAccessWithDeleted` dengan ID yang sama sebelum melempar error.

- `frontend/src/utils/axios.js` (+5 baris)
  - **Deskripsi**: Bug fix pada Axios response error interceptor:
    - **Masalah**: Interceptor error mengambil `error.response.data` untuk semua tipe response. Ketika request PDF menggunakan `responseType: 'arraybuffer'` gagal (misalnya token tidak valid, file tidak ditemukan), `error.response.data` adalah `ArrayBuffer` — bukan string/object yang bisa dilempar sebagai pesan error bermakna. Ini menyebabkan pesan error kosong atau corrupt di frontend saat fetch dokumen PDF gagal.
    - **Fix**: Tambah early return sebelum parsing: `if (error.config?.responseType === 'arraybuffer') return Promise.reject(error)` — untuk response `arraybuffer`, lempar raw error object agar caller (komponen halaman publik / preview PDF) bisa menangani sendiri tanpa data error yang korup.

---

## 📢 Dampak Perubahan & Fungsionalitas Baru

### Fungsionalitas Baru yang Tersedia (setelah commit)

- **Admin dapat membuat Customer PO**: Upload dokumen PDF dari customer langsung dari tab di halaman detail customer. PO otomatis diberi nomor urut jika tidak diisi manual.
- **Admin dapat membuat Customer SO**: Buat SO dengan line item lengkap (nama, tipe layanan, harga OTC/MRC, qty), kalkulasi total otomatis. SO dapat dikirimkan ke customer via share link.
- **Approval dengan tanda tangan digital**: Admin approve PO/SO melalui `ApprovePOModal` — pilih tanda tangan tersimpan, gambar baru, atau skip. Tanda tangan di-embed langsung ke dokumen PDF.
- **Share link tanda tangan customer**: Setelah disetujui, admin bisa generate link publik yang dikirimkan ke customer. Customer membuka link di browser (tanpa login), membaca dokumen, dan menandatangani secara digital.
- **Notifikasi Telegram**: Admin mendapat notifikasi saat PO disubmit, saat SO dibuat, serta saat customer menandatangani.
- **Halaman review internal**: Admin bisa langsung mengakses halaman review PO/SO dari link Telegram tanpa harus navigasi ke customer tertentu.
- **Tab navigation di Customer Detail**: Halaman detail customer sekarang memiliki tab Quotation / PO / SO yang terpisah — lebih terorganisir dan mudah dinavigasi.

### Konsistensi dengan Modul Vendor

Seluruh UX pattern Customer PO/SO identik dengan Vendor PO/SO (issue #118):
- Drawer create/edit dari kiri, review dari kanan
- ApprovePOModal dengan 3 opsi tanda tangan
- Halaman publik dengan `react-pdf` + signature pad
- Halaman review internal admin
- Share link via LinkBadge
- Timeline riwayat dokumen

---

## 📖 Informasi & Tutorial Singkat Fitur

**Alur Customer PO (End-to-End)**:
1. Customer kirim dokumen PO (PDF) ke ISP
2. Admin buka halaman detail customer → Tab "Customer PO" → klik Create
3. Upload file PDF customer, isi metadata, simpan
4. PO muncul di tabel dengan status "Pending Approval"
5. Admin klik PO → `CustomerPOReviewDrawer` terbuka
6. Opsional: Kirim preview ke admin tertentu
7. Admin klik Approve → `ApprovePOModal` muncul → pilih opsi tanda tangan
8. Backend embed tanda tangan ke PDF, set status Approved
9. Admin salin share link `/p/customer-po/:token` → kirim ke customer
10. Customer buka link → baca PDF → gambar tanda tangan di posisi yang dipilih → submit
11. Backend embed tanda tangan customer, set `complete = true`
12. PO selesai — dokumen final tersimpan di MinIO

**Alur Customer SO (End-to-End)**:
1. Admin buka halaman detail customer → Tab "Customer SO" → klik Create
2. Isi line item: nama layanan, tipe, harga, qty. Total kalkulasi otomatis.
3. Simpan → SO dibuat, notifikasi Telegram dikirim
4. Admin klik SO → `CustomerSOReviewDrawer` → kirim preview → Approve
5. Admin kirim share link ke customer
6. Customer buka link → lihat rincian SO → tanda tangan digital
7. SO complete — dokumen final siap diarsipkan
