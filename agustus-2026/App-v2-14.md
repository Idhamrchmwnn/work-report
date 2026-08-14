# 📝 Daily Work Report - Idham (2026-08-14)

---

## 📅 Laporan Harian - 14 Agustus 2026

---

## 🌿 Branch: `issue-223` — Implementasi Dokumen PKS

### 📌 Informasi Issue

- **Nomor Issue**: #223
- **Judul Issue**: Implementasi Dokumen PKS
- **Status Branch**: `Belum di-merge` (sudah di-push ke `origin/issue-223`)

### 📅 Rincian Commit

#### [d188ec6] - resolve #223 - 14 Agustus 2026, 18:17 (amend/rebase dari commit awal 10 Agustus)

- **Komponen yang Berubah**:
  - `backend/src/controllers/customerPKS.controller.js` [NEW]
  - `backend/src/controllers/publicCustomerPKS.controller.js` [NEW]
  - `backend/src/models/customerPKS.model.js` [NEW]
  - `backend/src/routes/customerPKS.route.js` [NEW]
  - `backend/src/services/customerPKS.service.js` [NEW]
  - `backend/src/app.js`
  - `backend/src/config/privilege.json`
  - `backend/src/routes/public.route.js`
  - `backend/src/utils/telegram.js`
  - `backend/src/locales/en/translation.json`, `backend/src/locales/id/translation.json`
  - `frontend/src/app/pages/public/PublicCustomerPKSDocument.jsx` [NEW]
  - `frontend/src/app/pages/public/ReviewCustomerPKSPage.jsx` [NEW]
  - `frontend/src/app/pages/users/customerPKS/CustomerPKSDocumentPreview.jsx` [NEW]
  - `frontend/src/app/pages/users/customerPKS/CustomerPKSReviewDrawer.jsx` [NEW]
  - `frontend/src/app/pages/users/customerPKS/create.jsx` [NEW]
  - `frontend/src/app/pages/users/customerPKS/edit.jsx` [NEW]
  - `frontend/src/app/pages/users/customerPKS/schema/customerPKSSchema.js` [NEW]
  - `frontend/src/app/pages/users/document/index.jsx` [NEW]
  - `frontend/src/app/pages/users/document/schema/pksColumns.jsx` [NEW]
  - `frontend/src/app/router/users/documentRoute.jsx` [NEW]
  - `frontend/src/app/navigation/users.js`, `frontend/src/app/router/protected.jsx`, `frontend/src/app/router/public.jsx`
  - `frontend/src/components/shared/DocumentPreviewModal.jsx`, `frontend/src/components/shared/table/rows.jsx`
  - `frontend/src/constants/privilegeDescriptions.en.json`, `frontend/src/constants/privilegeDescriptions.id.json`
  - `frontend/src/i18n/locales/en/translations.json`, `frontend/src/i18n/locales/id/translations.json`
  - **Total**: 30 files changed, 4153 insertions(+), 6 deletions(-)
- **Deskripsi Perubahan & Fungsi**:
  - Menambahkan modul dokumen **PKS (Perjanjian Kerja Sama)** untuk customer/partner, mengikuti pola dokumen upload+sign seperti SO: file PDF PKS diunggah lalu ditandatangani admin pada posisi yang dipilih di atas PDF (komposit `pdf-lib`).
  - Backend: CRUD lengkap (`create`, `approve`, `reject`, `update`, `delete`, `nextCount`) beserta service & model baru, endpoint publik (`publicCustomerPKS.controller.js`) untuk review/approval oleh pihak eksternal via link, notifikasi Telegram saat PKS disubmit, dan penambahan hak akses (`privilege.json`).
  - Frontend: halaman `create`/`edit` PKS, drawer review (`CustomerPKSReviewDrawer`), preview dokumen (`CustomerPKSDocumentPreview`), halaman publik untuk review & tanda tangan (`ReviewCustomerPKSPage`, `PublicCustomerPKSDocument`), serta integrasi ke menu navigasi, routing terproteksi/publik, dan tabel daftar dokumen.
  - Menambahkan terjemahan (ID/EN) dan deskripsi privilege baru terkait modul PKS.

---

## 🌿 Branch: `master` — Implementasi Dokumen SDN (Service Delivery Notification)

### 📌 Informasi Issue

- **Nomor Issue**: - (belum ada issue GitHub khusus yang tercatat untuk modul ini)
- **Judul Issue**: Implementasi Dokumen SDN (Service Delivery Notification)
- **Status Branch**: `Sudah di-commit di master`, **belum di-push** ke `origin/master` (ahead 1 commit)

### 📅 Rincian Commit

#### [db70041] - save sdn - 14 Agustus 2026, 16:47 (rebase dari commit awal 11 Agustus)

- **Komponen yang Berubah**:
  - `backend/src/controllers/customerSDN.controller.js` [NEW]
  - `backend/src/controllers/publicCustomerSDN.controller.js` [NEW]
  - `backend/src/models/customerSDN.model.js` [NEW]
  - `backend/src/routes/customerSDN.route.js` [NEW]
  - `backend/src/services/customerSDN.service.js` [NEW]
  - `backend/src/app.js`, `backend/src/config/privilege.json`
  - `backend/src/routes/files.route.js`, `backend/src/routes/public.route.js`
  - `backend/src/utils/telegram.js`
  - `backend/src/locales/en/translation.json`, `backend/src/locales/id/translation.json`
  - `frontend/src/app/pages/public/PublicCustomerSDNDocument.jsx` [NEW]
  - `frontend/src/app/pages/public/ReviewCustomerSDNPage.jsx` [NEW]
  - `frontend/src/app/pages/users/customerSDN/CustomerSDNDocumentPreview.jsx` [NEW]
  - `frontend/src/app/pages/users/customerSDN/CustomerSDNReviewDrawer.jsx` [NEW]
  - `frontend/src/app/pages/users/customerSDN/create.jsx` [NEW]
  - `frontend/src/app/pages/users/customerSDN/edit.jsx` [NEW]
  - `frontend/src/app/pages/users/customerSDN/schema/customerSDNSchema.js` [NEW]
  - `frontend/src/app/pages/users/document/index.jsx` [NEW]
  - `frontend/src/app/pages/users/document/schema/sdnColumns.jsx` [NEW]
  - `frontend/src/app/router/protected.jsx`, `frontend/src/app/router/public.jsx`
  - `frontend/src/components/shared/DocumentPreviewModal.jsx`, `frontend/src/components/shared/table/rows.jsx`
  - `frontend/src/constants/privilegeDescriptions.en.json`, `frontend/src/constants/privilegeDescriptions.id.json`
  - `frontend/src/i18n/locales/en/translations.json`, `frontend/src/i18n/locales/id/translations.json`
  - `frontend/package-lock.json`
  - **Total**: 30 files changed, 4048 insertions(+), 13 deletions(-)
- **Deskripsi Perubahan & Fungsi**:
  - Menambahkan modul dokumen **SDN (Service Delivery Notification)** yang terhubung ke SO customer (`so` / `so_number` / `so_date`) — bukti serah-terima layanan yang sudah aktif.
  - Fitur khusus: unggah **gambar topologi** (`topology_image`, maks. 10MB, format jpg/jpeg/png/webp) sebagai lampiran bukti instalasi, di samping data proyek/layanan (`project_name`, `service_id`, `service_ordered`).
  - Backend: CRUD lengkap dengan alur approve/reject, endpoint publik untuk review eksternal, notifikasi Telegram saat SDN disubmit.
  - Frontend: halaman `create`/`edit` SDN, drawer & preview review, halaman publik review, serta integrasi menu/routing/tabel dokumen — mengikuti pola yang sama dengan modul PKS di atas.
  - **Catatan**: commit ini belum tertaut ke nomor issue GitHub tertentu — perlu dikonfirmasi ke tracker apakah modul SDN sudah punya issue resmi.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul                                          | Dampak Utama                                                                                     |
| ----- | ----------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| #223  | Implementasi Dokumen PKS                        | Customer/partner kini bisa mengunggah & menandatangani dokumen Perjanjian Kerja Sama secara digital |
| -     | Implementasi Dokumen SDN (Service Delivery Notification) | Tim bisa menerbitkan bukti serah-terima layanan (SDN) terkait SO, lengkap dengan foto topologi     |

### Kemampuan Baru Pengguna/Admin

- Admin dapat membuat, mereview, menyetujui/menolak, dan menandatangani dokumen PKS & SDN langsung dari sistem.
- Customer/partner dapat mereview dan menandatangani dokumen melalui halaman publik (tanpa login) via link yang dikirim.
- Notifikasi otomatis ke Telegram saat dokumen PKS/SDN baru disubmit oleh customer.

### Bug Fix / Solusi Masalah

- Tidak ada bug fix pada aktivitas hari ini — murni penambahan fitur baru (modul PKS & SDN) beserta rebase kedua branch ke `master` terbaru.

### Menu/Fitur Baru

- Menu **Dokumen PKS** (Perjanjian Kerja Sama) di navigasi users, dengan sub-halaman create/edit/preview/review.
- Menu **Dokumen SDN** (Service Delivery Notification) di navigasi users, dengan tambahan unggah gambar topologi jaringan.
- Halaman publik baru untuk review & tanda tangan dokumen PKS dan SDN oleh pihak eksternal.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Modul PKS dan SDN adalah dua jenis dokumen customer baru yang mengikuti pola "upload PDF lalu tanda tangan digital di posisi tertentu" (untuk PKS) dan "form data + lampiran foto topologi" (untuk SDN). Keduanya punya alur: dibuat oleh admin/sales → dikirim ke customer via link publik → customer review & approve/reject → jika disetujui, dokumen final tersimpan dan bisa diunduh.
- **Langkah Penggunaan (Tutorial)**:
  1. Buka menu **Dokumen** pada sidebar users, pilih tab **PKS** atau **SDN**.
  2. Klik **Buat Baru**, pilih partner/customer terkait, lengkapi data (untuk PKS: unggah file PDF perjanjian; untuk SDN: pilih SO terkait & unggah foto topologi).
  3. Simpan — sistem akan mengirim notifikasi Telegram dan link review ke customer.
  4. Customer membuka link publik, mereview dokumen, lalu approve (tanda tangan) atau reject dengan alasan.
  5. Setelah approve, dokumen final dapat dilihat/diunduh dari halaman preview dokumen di sistem internal.

---

## ⚠️ Catatan Tambahan

- Kedua commit hari ini (`d188ec6` pada `issue-223` dan `db70041` pada `master`) merupakan hasil **rebase** pekerjaan yang aslinya dikerjakan pada 10–11 Agustus 2026 (author date), yang di-rebase ulang ke atas `master` terbaru pada 14 Agustus (commit date). Isi perubahan yang dilaporkan di atas adalah snapshot final setelah rebase.
- Branch `master` lokal saat ini **ahead 1 commit** dari `origin/master` (`save sdn` belum di-push) — perlu di-push agar tidak hilang/tertinggal.
- Modul SDN belum tertaut ke nomor issue GitHub — disarankan membuat/mengaitkan issue resmi untuk tracking.
