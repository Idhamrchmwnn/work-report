# 📝 Daily Work Report - Idham (2026-08-24)

---

## 📅 Laporan Harian - 24 Agustus 2026

---

## 🌿 Branch: `issue-228` — Implementasi Dokumen SDN

### 📌 Informasi Issue

- **Nomor Issue**: #228
- **Judul Issue**: Implementasi Dokumen SDN
- **Status Branch**: `Sudah di-commit dan di-push` — branch lokal `issue-228` kini **up to date** dengan `origin/issue-228` (tidak ada lagi commit tertinggal seperti laporan-laporan sebelumnya)

### 📅 Rincian Commit

#### [eb41f3d] - resolve #228 - 24 Agustus 2026, 14:58

**Ringkasan**: 37 files changed, 4196 insertions(+), 19 deletions(-). Ini adalah **commit final** dari pekerjaan modul SDN yang sebelumnya berulang kali dilaporkan sebagai *work in progress* (rebase 14 & 18 Agustus, staged-belum-commit 18–19 Agustus) — hari ini akhirnya di-commit dan **berhasil di-push** ke `origin/issue-228`. Berikut rincian tiap berkas yang berubah.

##### Backend — Modul Inti SDN

- **`backend/src/models/customerSDN.model.js`** [NEW] — Skema Mongoose `CustomerSDN` (koleksi `customer_sdn`). Field utama: `partner` (wajib, relasi ke `Partner`), `so` (opsional, relasi ke `CustomerSO`), `status` (`draft`/`sent`), `sdn_number` (unik), `sdn_date`, `service_id`, `project_name` (wajib), `service_ordered`, `start_date`/`end_date`/`start_billing_date`, `circuit_type`, `bandwidth`, `a_end`/`b_end` (sub-skema `SiteSchema`: `rack_room`, `building_name`, `address`), `sla`, `additional_notes`, `topology_image`, `approval`+`approved_at`, `complete`, `sent_at`, `share_token`, `created_by`. Memakai plugin `mongoose-delete` (soft delete) dan `autoIncrementPlugin` (nomor urut `seq`). Komentar di source menegaskan SDN **tidak perlu tanda tangan dua pihak** — penerimaan dianggap sah otomatis bila partner tidak mengajukan komplain "Unmatched Items" dalam 7 hari, jadi siklusnya hanya `draft → sent` (beda dari SO/PKS yang punya status `signed`).
- **`backend/src/services/customerSDN.service.js`** [NEW] — Lapisan akses data: `findCustomerSDNById` (cari by ObjectId atau `sdn_number`), `updateCustomerSDNData`, `findAllCustomerSDNForTable` (listing server-side/DataTable), `createNewCustomerSDN`, `approveCustomerSDN` (set `approval`+`approved_at`), `rejectCustomerSDN` (soft-delete), `deleteCustomerSDNById`, `findCustomerSDNByToken` (lookup via `share_token` untuk halaman publik), `nextCountCustomerSDN` (nomor urut berikutnya). Semua query dikunci ke `pid: 'master'`.
- **`backend/src/controllers/customerSDN.controller.js`** [NEW, 357 baris] — 10 endpoint handler:
  - `createSDN` — validasi `partner_id` (dengan fix prefiks `PART-`/`CUST-`, lihat bawah), wajib `project_name`, opsional tautkan ke SO (`so_number`/`so_date` disalin dari SO bila ada), upload gambar topology (validasi tipe JPG/PNG/WEBP & maks. 10 MB lewat helper `storeTopologyImage`), generate `sdn_number` berformat `SDN/{partner_id}/{urut}/{bulan}/{tahun}` dan `share_token` acak.
  - `listAllSDN`, `readSDN` — listing tabel & baca detail.
  - `submitSDN` — ajukan SDN untuk approval internal (kirim notifikasi Telegram), ditolak bila sudah `approval`/`complete`.
  - `requestSDNPreview` — kirim ulang pratinjau ke Telegram, bisa ditarget ke admin tertentu (`adminId`) atau ke channel approval default.
  - `approveSDN` / `rejectSDN` — approval internal oleh PIC (reject = soft-delete), masing-masing ditolak bila SDN sudah pernah di-approve.
  - `sendSDN` — tandai `status: 'sent', complete: true, sent_at: now` (syarat: sudah `approval`, belum pernah terkirim) — ini yang membuka akses link publik ke partner.
  - `updateSDN`, `deleteSDN` — edit & hapus data.
- **`backend/src/controllers/publicCustomerSDN.controller.js`** [NEW] — Dua endpoint tanpa login berbasis `share_token`: `getSDNByToken` (kembalikan DTO terbatas — tidak expose field internal seperti `created_by`) dan `getSDNTopologyByToken` (stream gambar topology sebagai file). Keduanya menolak akses (404) bila SDN belum `approval` (belum disetujui PIC).
- **`backend/src/controllers/customerSO.controller.js`** [+10/-2] — Fix yang sebelumnya dilaporkan sebagai staged-changes (18–19 Agustus), sekarang resmi ter-commit: `listSO` memfilter `customer_id` berdasarkan prefiks — `PART-` dilepas sebelum lookup, `CUST-` langsung digagalkan (`null` → 404 "customerNotFound") karena SO seharusnya hanya berlaku untuk entitas Partner.
- **`backend/src/routes/customerSDN.route.js`** [NEW, 311 baris] — Mendaftarkan 10 route (`POST /customer-sdn/create`, `POST /customer-sdn/list-all`, `GET /customer-sdn/view/:id`, `PATCH /customer-sdn/submit/:id`, `POST /customer-sdn/request-preview`, `PATCH /customer-sdn/approve/:id`, `PATCH /customer-sdn/reject/:id`, `POST /customer-sdn/send/:id`, `PATCH /customer-sdn/update/:id`, `DELETE /customer-sdn/delete/:id`), lengkap dengan dokumentasi Swagger dan guard `protectedAdmin` + `checkPrivilege('customerSDN.*')` di tiap endpoint.

##### Backend — Integrasi & Notifikasi

- **`backend/src/app.js`** [+2] — Import dan mount `CustomerSDNRoute` ke `app.use('/api/v1', ...)`.
- **`backend/src/config/privilege.json`** [+7] — Menambahkan grup privilege baru `customerSDN` dengan 5 aksi: `create`, `read`, `update`, `changeStatus`, `delete`.
- **`backend/src/routes/files.route.js`** [+8] — Endpoint `GET /file/customer-sdn/:name` (dilindungi `customerSDN.read`) untuk menyajikan gambar topology dari bucket `appFiles` yang sama dipakai dokumen lain.
- **`backend/src/routes/public.route.js`** [+11] — Mendaftarkan dua route publik: `GET /public-docs/customer-sdn/:token` dan `.../topology`, keduanya token-gated tanpa autentikasi.
- **`backend/src/utils/telegram.js`** [+51] — Fungsi baru `TelegramNotifCustomerSDNSubmit(sdn, user, targetChatId)`: mengirim pesan berformat HTML ke Telegram berisi nomor SDN, nama proyek, nama partner, tanggal, pembuat, dan tombol link `"BERIKAN PERSETUJUAN"` yang mengarah ke `{WEB_URL}/review/customer-sdn/{id}` — halaman approval internal berbasis link.
- **`backend/src/locales/{en,id}/translation.json`** [+18 masing-masing] — 16 pesan error/sukses baru di namespace `sdn` (backend-side), mis. `notFound`, `cannotSubmit`, `cannotApprove`, `alreadySent`, `topologyInvalidType`, dll — dipakai langsung oleh controller di atas lewat `req.t(...)`.

##### Frontend — Infrastruktur Modul Dokumen (baru, generik)

- **`frontend/src/app/pages/users/document/registry.js`** [NEW] — Array `DOCUMENT_MODULES` berisi daftar modul dokumen aktif (saat ini hanya `sdnModule`). Komentar eksplisit: *"Satu-satunya tempat yang perlu diubah untuk menambah/menghapus tab"* — disiapkan untuk `pksModule`, `slaModule`, dst di masa depan.
- **`frontend/src/app/pages/users/document/index.jsx`** [NEW] — Halaman shell `/users/document`: merender tab-tab dari `DOCUMENT_MODULES`, menyimpan `activeKey` di state, dan me-render `Component` milik modul yang sedang aktif.
- **`frontend/src/app/pages/users/document/shared/useDocumentApproval.js`** [NEW] — Hook generik yang dipakai tiap tab modul dokumen (dipanggil dengan `apiPrefix`, mis. `/customer-sdn`): mengelola state buka/tutup drawer review, buka/tutup modal preview, serta handler `approve`/`reject` — supaya modul dokumen baru tidak perlu menulis ulang boilerplate ini.

##### Frontend — Modul SDN (tab, form, tabel)

- **`frontend/src/app/pages/users/document/sdn/index.jsx`** [NEW] — Komponen `SDNTab`: merangkai `Datatables` (kolom dari `getSDNColumns`), tombol "Buat SDN" (privilege `customerSDN.create`), drawer create/review, dan `DocumentPreviewModal` — lalu diekspor sebagai `sdnModule = { key: 'sdn', label: 'customer.sdn.title', Component: SDNTab }` untuk didaftarkan di `registry.js`.
- **`frontend/src/app/pages/users/document/sdn/create.jsx`** [NEW, 545 baris] — Drawer form pembuatan SDN. Field: Partner (dengan pencarian), SO terkait (opsional — memicu auto-fill `service_id`, `service_ordered`, `so_number`, `so_date` dari SO terpilih), Nama Proyek, Circuit Type, Bandwidth, tanggal Mulai/Berakhir/Mulai Billing, SLA, detail lokasi A-End & B-End (Rack/Room, Nama Gedung, Alamat masing-masing), Catatan Tambahan, dan upload gambar Topology Design (`accept="image/jpeg,image/png,image/webp"`, dikirim sebagai `multipart/form-data`).
- **`frontend/src/app/pages/users/document/sdn/edit.jsx`** [NEW, 519 baris] — Drawer edit dengan field yang sama seperti create, memuat data existing dan mengizinkan penggantian gambar topology; hanya bisa diakses untuk SDN yang belum di-`approval` (lihat kolom Aksi di `columns.jsx`).
- **`frontend/src/app/pages/users/document/sdn/ReviewDrawer.jsx`** [NEW, 516 baris] — Drawer review internal (`CustomerSDNReviewDrawer`): menampilkan detail SDN, tombol kirim pratinjau ke Telegram (ke admin tertentu atau default), tombol Approve/Reject untuk PIC, tombol "Kirim ke Partner" (memicu `sendSDN` setelah disetujui) beserta salin-link, dan riwayat dokumen (dibuat → disetujui → terkirim).
- **`frontend/src/app/pages/users/document/sdn/DocumentPreview.jsx`** [NEW, 249 baris] — Komponen `CustomerSDNDocumentPreview`: layout dokumen SDN siap cetak (kop surat, info pelanggan, detail proyek/circuit/site A-End & B-End, teks baku pemberitahuan termasuk klausul "Unmatched Items" 7 hari) — dipakai baik di modal preview internal maupun halaman publik.
- **`frontend/src/app/pages/users/document/sdn/schema/columns.jsx`** [NEW, 102 baris] — Definisi kolom tabel: Tanggal Dibuat, No. SDN (klik untuk preview), Nama Proyek, Partner (tautan ke halaman Business/Partner sesuai `reseller`), No. SO, Status (`CustomerSDNStatusCell`), dan kolom Aksi (Edit hanya muncul bila belum di-approve; Delete).
- **`frontend/src/app/pages/users/document/sdn/schema/createSchema.js`** [NEW] — Skema validasi Yup: `partner_id` & `project_name` wajib, sisanya opsional (termasuk sub-skema `siteSchema` untuk A-End/B-End).
- **`frontend/src/app/pages/users/document/sdn/schema/editSchema.js`** [NEW, 3 baris] — Cukup me-re-export `createSchema as editSchema` dari file sebelumnya, dengan komentar bahwa form edit divalidasi dari sumber skema yang sama alih-alih diduplikasi.

##### Frontend — Halaman Publik & Approval via Link

- **`frontend/src/app/pages/public/PublicCustomerSDNDocument.jsx`** [NEW, 136 baris] — Halaman publik tanpa login di `/p/customer-sdn/:token`: mengambil data lewat `share_token`, menampilkan `CustomerSDNDocumentPreview`, dan tombol cetak. Ini yang dibuka partner setelah SDN ditandai "Kirim ke Partner".
- **`frontend/src/app/pages/public/ReviewCustomerSDNPage.jsx`** [NEW, 295 baris] — Halaman internal di `/review/customer-sdn/:id` (tetap perlu login, digerbang privilege `customerSDN.changeStatus`) yang diakses lewat tombol "BERIKAN PERSETUJUAN" pada notifikasi Telegram — memuat detail SDN, modal preview, dan aksi approve/reject dengan modal konfirmasi.

##### Frontend — Integrasi ke Aplikasi

- **`frontend/src/app/navigation/users.js`** [+10] — Menambahkan item menu sidebar baru **"Document"** (ikon `DocumentTextIcon`, path `/users/document`, digerbang privilege `customerSDN.read`) di antara Customer Partner dan pemisah menu.
- **`frontend/src/app/router/protected.jsx`** [+9] — Mendaftarkan `usersDocumentRoute` ke rute users, plus rute baru `review/customer-sdn/:id` yang lazy-load `ReviewCustomerSDNPage`.
- **`frontend/src/app/router/public.jsx`** [+8] — Mendaftarkan rute publik `p/customer-sdn/:token` yang lazy-load `PublicCustomerSDNDocument`.
- **`frontend/src/app/router/users/documentRoute.jsx`** [NEW] — Definisi rute `document` (lazy-load halaman `document/index.jsx`), digerbang privilege `customerSDN.read`.
- **`frontend/src/components/shared/DocumentPreviewModal.jsx`** [+37/-2] — Menambahkan cabang `type === 'customer-sdn'`: fetch ulang data lengkap via `/customer-sdn/view/:ref` sebelum ditampilkan, merender `CustomerSDNDocumentPreview`, serta menyertakan `sdn_number` pada judul dokumen saat dicetak/diunduh.
- **`frontend/src/components/shared/table/rows.jsx`** [+77] — Komponen baru `CustomerSDNStatusCell`: badge status tabel dengan 4 kondisi tampilan — *Terkirim* (centang hijau), *Disetujui PIC* (centang hijau + avatar approver), tombol *Review* (bila user punya privilege `customerSDN.changeStatus`), atau *Menunggu Persetujuan* (ikon kuning) untuk user tanpa privilege tersebut.
- **`frontend/src/constants/privilegeDescriptions.{en,id}.json`** [+5 masing-masing] — Deskripsi human-readable untuk 5 privilege `customerSDN.*` baru, ditampilkan di halaman kelola Privilege.
- **`frontend/src/i18n/locales/{en,id}/translations.json`** [+94 masing-masing] — Label UI lengkap untuk modul SDN: judul, field form, status (`draft`/`approvedPic`/`sent`), riwayat dokumen, serta teks isi dokumen (salam pembuka, klausul "Unmatched Items" 7 hari, penutup surat) untuk halaman publik dan preview cetak.
- **`frontend/package-lock.json`** [+44/-6] — Update versi dependency (mengikuti perubahan `package.json`/instalasi ulang; tidak ada dependency baru yang signifikan untuk fitur SDN).

##### Catatan Arsitektur

- **Refactor struktural**: halaman frontend SDN dipindah dari pola lama `pages/users/customerSDN/` (satu folder datar per jenis dokumen) menjadi `pages/users/document/sdn/` mengikuti **pola modul dokumen generik** (`registry.js` + `useDocumentApproval` hook bersama) — supaya modul dokumen baru berikutnya (PKS, SLA, dst) tinggal plug-in tanpa menulis ulang boilerplate tab/review/preview/approve/reject.
- Modul **PKS belum dimigrasikan** ke pola `document/` yang baru — masih memakai struktur lama (`pages/users/customerPKS/`). Perlu direfactor menyusul bila pola generik ini ingin dipakai konsisten untuk semua jenis dokumen.
- **Perbaikan validasi & auto-fill dari laporan 18–19 Agustus sudah terverifikasi masuk** ke commit final ini: fix resolusi ID Partner vs Customer (prefiks `PART-`/`CUST-`) pada `createSDN` & `listSO`, serta auto-fill `service_id`/`service_ordered` dari SO terpilih pada form create.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul | Dampak Utama |
| --- | --- | --- |
| #228 | Implementasi Dokumen SDN | Modul SDN resmi rilis ke branch `issue-228` (sudah di-push), dengan arsitektur frontend baru yang generik untuk seluruh modul dokumen ke depan |

### Bug Fix / Solusi Masalah

- Validasi ID Partner (`PART-`) vs Customer (`CUST-`) pada pembuatan SDN & listing SO — mencegah SDN dibuat dengan ID entitas Customer biasa — resmi masuk ke commit final hari ini (sebelumnya tertahan di staged changes sejak 18 Agustus).

### Menu/Fitur Baru

- Tidak ada menu baru di luar SDN yang sudah dilaporkan sebelumnya — fokus hari ini adalah **finalisasi & refactor arsitektur**, bukan fitur baru.

---

## ⚠️ Catatan Tambahan

- Rantai kerja modul SDN (rebase 14 Agustus → staged 18–19 Agustus → **commit & push final 24 Agustus**) akhirnya tuntas; branch `issue-228` tidak lagi punya perubahan tertinggal.
- `registry.js` menyiapkan slot eksplisit untuk modul dokumen lain (`pksModule`, dst), tapi **PKS belum dipindahkan** ke pola baru ini — masih di struktur lama. Perlu dijadikan item kerja lanjutan agar konsisten.
- Tidak ada aktivitas pada `docs system/` (dokumentasi/tutorial) hari ini.
