# 📝 Daily Work Report - Idham (2026-08-27)

---

## 📅 Laporan Harian - 27 Agustus 2026

---

## 🌿 Branch: `issue-228` — Modul Dokumen Direktur (baru, belum ada nomor issue)

### 📌 Informasi

- **Status Branch**: `issue-228`, up to date dengan `origin/issue-228` dari sisi commit — tapi hari ini ada **pekerjaan besar yang masih sepenuhnya belum di-commit** (belum di-`stage` sama sekali, bukan cuma staged seperti WIP-WIP sebelumnya).
- **Cakupan**: modul dokumen baru, **Dokumen Direktur** (mis. Surat Keputusan Direksi) — PDF yang diunggah lalu ditandatangani langsung oleh admin/direktur internal di posisi pilihan, tanpa pihak eksternal sama sekali. Ini modul dokumen generik **kedua** yang mengikuti pola registry yang dibangun pada modul SDN (24 Agustus) — persis mengisi slot yang dulu disiapkan (`registry.js` sudah punya komentar contoh `pksModule, slaModule, restitusiModule`, sekarang terisi `direkturModule`).
- **16 file berubah** (16 modified, belum termasuk file baru): 340 insertions(+), 53 deletions(-) pada file existing.
- **7 file baru** (belum di-`git add`): 2 skema Mongoose bersama di backend, dan 7 file/folder modul frontend `document/direktur/`.

### 🆕 Berkas Baru — Backend

- **`backend/src/models/direkturDocument.model.js`** — Skema Mongoose `DirekturDocument` (koleksi `direktur_document`). Field: `title`, `document_number` (unik), `document_date`, `category`, `description`, `status` (`draft`/`signed` saja — cuma 2 state), `document` (sub-skema `DocumentFileSchema`), `approval`+`approved_at`, `complete`, `signature` (path gambar ttd), `signature_position` (sub-skema `SignaturePositionSchema`), `signed_file` (hasil PDF gabungan), `signed_at`, `signed_by`, `created_by`. Komentar di source menegaskan: karena tidak ada pihak eksternal yang menandatangani belakangan, **"approve" dan "sign" digabung jadi satu langkah** — beda dari VendorSO/CustomerPO yang baru ditandatangani lewat link publik setelah disetujui.
- **`backend/src/services/direkturDocument.service.js`** — Lapisan data: `findDirekturDocumentById` (by ObjectId atau `document_number`), `findAllDirekturDocumentForTable`, `createNewDirekturDocument`, `updateDirekturDocumentData`, **`approveAndSignDirekturDocument`** (set `signature`, `signature_position`, `signed_file`, `signed_at`, `signed_by`, `approval`, `approved_at`, `status: 'signed'`, `complete: true` sekaligus dalam satu update), `rejectDirekturDocument` (soft-delete), `deleteDirekturDocumentById`, `nextCountDirekturDocument`.
- **`backend/src/controllers/direkturDocument.controller.js`** [357 baris, 9 endpoint] —
  - `createDirekturDocument` — validasi `title` & `document_number` wajib, upload PDF lewat `storeDirekturDocumentFile`, rollback (`removeAppFile`) bila penyimpanan data gagal setelah file terlanjur ter-upload.
  - `listAllDirekturDocument`, `readDirekturDocument` — listing & baca detail.
  - `submitDirekturDocument` — ajukan untuk approval internal, kirim notifikasi Telegram.
  - `requestDirekturDocumentPreview` — kirim ulang pratinjau ke Telegram (admin tertentu atau channel default) — pola identik dengan modul SDN.
  - **`approveDirekturDocument`** — inti fitur: menerima `signature` (bisa `'USE_EXISTING'` untuk memakai tanda tangan tersimpan admin lewat `getAdminSign`, atau data URL gambar baru dari kanvas/upload) dan `position` (`page`, `x`, `y`, ternormalisasi 0..1). Bila `saveToProfile` dicentang, tanda tangan baru disimpan ke profil admin/employee (`updateAdminById`/`updateEmployeeById`) untuk dipakai lagi nanti. PDF asli diambil (`getAppFileBuffer`), tanda tangan ditempel di posisi tersebut lewat **`stampSignatureOnPdf`** (util `pdf-sign.js` yang sudah ada, dipakai ulang dari alur SO/PKS), hasil disimpan sebagai `signed_file` baru.
  - `rejectDirekturDocument`, `deleteDirekturDocument` — masing-masing membersihkan berkas PDF terkait (`removeAppFile`) sebelum/ setelah menghapus data.
  - `updateDirekturDocument` — edit metadata/berkas.
- **`backend/src/routes/direkturDocument.route.js`** [9 route] — `POST create`, `POST list-all`, `GET view/:id`, `PATCH submit/:id`, `POST request-preview`, `PATCH approve/:id`, `PATCH reject/:id`, `PATCH update/:id`, `DELETE delete/:id` — semua digerbang `protectedAdmin` + `checkPrivilege('direkturDocument.*')`, lengkap dokumentasi Swagger. **Tidak ada route publik** — konsisten dengan sifat modul yang murni internal.
- **`backend/src/models/shared/documentFile.schema.js`** & **`backend/src/models/shared/signaturePosition.schema.js`** — **Ekstraksi refactor**: dua sub-skema (`DocumentFileSchema`, `SignaturePositionSchema`) yang tadinya didefinisikan berduplikasi persis di `customerPO.model.js` dan `vendorSO.model.js`, sekarang dipindah ke `models/shared/` supaya bisa dipakai bersama oleh ketiga model (PO, VendorSO, dan DirekturDocument baru) tanpa duplikasi. Komentar di file baru secara eksplisit menyebut kontrak `SignaturePositionSchema` ini juga dikonsumsi oleh `stampSignatureOnPdf`.

### ✏️ Berkas Existing yang Ikut Berubah — Backend

- **`backend/src/models/customerPO.model.js`** [-23] & **`backend/src/models/vendorSO.model.js`** [-24] — Definisi `DocumentFileSchema`/`SignaturePositionSchema` inline dihapus, diganti `import` dari `models/shared/` yang baru (lihat di atas). Perilaku tidak berubah, murni deduplikasi.
- **`backend/src/app.js`** [+2] — Import & mount `DirekturDocumentRoute`.
- **`backend/src/config/privilege.json`** [+7] — Grup privilege baru `direkturDocument` (5 aksi: create/read/update/changeStatus/delete).
- **`backend/src/routes/files.route.js`** [+40] — Endpoint `GET /file/direktur-document/:name` (digerbang `direkturDocument.read`) untuk menyajikan PDF hasil unggahan/bertanda tangan, memakai handler `getQuotationFile` yang sama dipakai dokumen lain.
- **`backend/src/utils/telegram.js`** [+51] — Fungsi baru `TelegramNotifDirekturDocumentSubmit`: pesan Telegram berisi nomor dokumen, judul, tanggal, nama & jumlah halaman berkas, pengaju, dan tombol link `"BERIKAN PERSETUJUAN"` ke `{WEB_URL}/review/direktur-document/{id}` — pola identik SDN, tapi **catatan**: halaman `/review/direktur-document/:id` itu sendiri **belum dibuat** hari ini (lihat bagian Catatan).
- **`backend/src/locales/{en,id}/translation.json`** [+21 masing-masing] — Pesan error/sukses namespace `direktur` di backend (`notFound`, `cannotApprove`, `signatureRequired`, `positionRequired`, dst).

### 🆕 Berkas Baru — Frontend

- **`frontend/src/app/pages/users/document/direktur/index.jsx`** — Komponen tab `DirekturTab` (pola sama seperti `sdn/index.jsx`): merangkai `Datatables`, tombol "Buat Dokumen" bergerbang privilege `direkturDocument.create`, drawer create & review, `DocumentPreviewModal`. Diekspor sebagai `direkturModule = { key: 'direktur', label: ..., Component: DirekturTab }` untuk didaftarkan ke `registry.js`.
- **`frontend/src/app/pages/users/document/direktur/create.jsx`** — Form upload: field `title`, `document_number`, `document_date`, `category`, `description`, plus input file `accept="application/pdf"` untuk berkas dokumen. Lebih ringkas dari form SDN karena Dokumen Direktur cuma menampung metadata + satu berkas PDF, bukan data proyek/circuit yang detail.
- **`frontend/src/app/pages/users/document/direktur/edit.jsx`** — Drawer edit metadata/ganti berkas, hanya untuk dokumen yang belum `signed`.
- **`frontend/src/app/pages/users/document/direktur/ReviewDrawer.jsx`** — Drawer review **paling kompleks** di modul ini: merender PDF halaman-per-halaman (`react-pdf` via `configs/pdf.config`), memakai hook baru `usePdfSignaturePlacement` untuk klik-untuk-taruh & drag-untuk-geser kotak tanda tangan di atas render PDF, pilihan pakai tanda tangan tersimpan (`USE_EXISTING`) atau gambar baru (kanvas tanda tangan) dengan opsi "simpan ke profil", lalu tombol approve+sign mengirim `signature` + `position` ke backend sekaligus.
- **`frontend/src/app/pages/users/document/direktur/DocumentPreview.jsx`** — Komponen `DirekturDocumentPreview` untuk tampilan pratinjau/cetak (dipakai di `DocumentPreviewModal` maupun di dalam `ReviewDrawer`).
- **`frontend/src/app/pages/users/document/direktur/schema/columns.jsx`** — Kolom tabel: tanggal dibuat, no. dokumen, judul, kategori, status (`DirekturDocumentStatusCell`), aksi (edit hanya bila belum signed, delete).
- **`frontend/src/app/pages/users/document/direktur/schema/direkturSchema.js`** — Skema Yup: `title` & `document_number` wajib, sisanya opsional; validasi berkas PDF ditangani terpisah di komponen input file (bukan lewat Yup).
- **`frontend/src/hooks/usePdfSignaturePlacement.js`** — **Ekstraksi hook baru**: logika geometri murni untuk memilih & menggeser posisi tanda tangan di atas render PDF, **diekstrak dari `app/pages/public/PublicSODocument.jsx`** (halaman publik tanda-tangan SO yang sudah ada) supaya bisa dipakai ulang di halaman internal (review Dokumen Direktur) tanpa duplikasi logika drag/klik. Posisi disimpan ternormalisasi (0..1) — kontrak yang sama dengan `SignaturePositionSchema` di backend.

### ✏️ Berkas Existing yang Ikut Berubah — Frontend

- **`frontend/src/app/pages/users/document/registry.js`** [+2] — Mendaftarkan `direkturModule` sebagai tab kedua di `DOCUMENT_MODULES`, setelah `sdnModule`.
- **`frontend/src/hooks/index.js`** [+1] — Re-export `usePdfSignaturePlacement` dari barrel hooks.
- **`frontend/src/components/shared/DocumentPreviewModal.jsx`** [+30/-1] — Cabang baru `type === 'direktur-document'`: fetch ulang data via `/direktur-document/view/:ref`, render `DirekturDocumentPreview`, serta `document_number` disertakan pada judul cetak/unduh.
- **`frontend/src/components/shared/table/rows.jsx`** [+62] — Komponen baru `DirekturDocumentStatusCell`: hanya 2 tampilan status (beda dari SDN yang 3-state) — *Signed* (centang hijau + avatar penanda tangan) atau tombol *Review*/label *Draft* menunggu tanda tangan, sesuai privilege user.
- **`frontend/src/constants/privilegeDescriptions.{en,id}.json`** [+5 masing-masing] — Deskripsi 5 privilege `direkturDocument.*` baru.
- **`frontend/src/i18n/locales/{en,id}/translations.json`** [+45 masing-masing] — Label UI modul Direktur: judul, field form, status (`draft`/`signed`), riwayat dokumen, teks kanvas tanda tangan, dll.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Modul | Dampak Utama |
| --- | --- |
| Dokumen Direktur (baru) | Admin/direktur bisa mengunggah dokumen resmi (mis. SK Direksi) dan menandatanganinya langsung di posisi pilihan pada PDF, tanpa perlu pihak eksternal — sekaligus jadi modul dokumen generik kedua yang membuktikan pola `registry.js` dari 24 Agustus bisa dipakai ulang |

### Menu/Fitur Baru

- Tab **"Direktur"** baru di halaman **Pengguna → Document**, di samping tab SDN yang sudah ada.
- Alur approve+sign satu langkah dengan pemilihan posisi tanda tangan interaktif di atas render PDF (klik untuk taruh, seret untuk geser).
- Opsi memakai tanda tangan tersimpan admin, atau menggambar/mengunggah baru dengan opsi simpan ke profil untuk dipakai lagi.

### Refactor / Deduplikasi

- `DocumentFileSchema` & `SignaturePositionSchema` yang tadinya terduplikasi di `customerPO.model.js` dan `vendorSO.model.js` disatukan ke `models/shared/`.
- Logika drag-and-drop penempatan tanda tangan di atas PDF diekstrak dari `PublicSODocument.jsx` menjadi hook `usePdfSignaturePlacement` yang bisa dipakai ulang.

---

## ⚠️ Catatan Tambahan

- **Semua perubahan hari ini belum di-`git add`/commit sama sekali** (16 file modified + 7 file/folder baru, semuanya masih di working directory) — risiko lebih tinggi dari WIP-WIP sebelumnya karena belum ada checkpoint di git.
- **Kemungkinan potongan yang belum lengkap**: `TelegramNotifDirekturDocumentSubmit` sudah mengarah ke link `/review/direktur-document/:id`, tapi halaman/rute React untuk `/review/direktur-document/:id` (setara `ReviewCustomerSDNPage` untuk SDN) **belum terlihat dibuat** — perlu dicek apakah approval dari notifikasi Telegram sudah bisa diakses, atau masih harus lewat tab Document di aplikasi utama.
- Modul **PKS masih belum dimigrasikan** ke pola `document/` generik (dicatat sejak laporan 24 Agustus) — kini ada 2 dari kemungkinan banyak modul dokumen (SDN, Direktur) yang sudah pakai pola baru, PKS masih tertinggal di struktur lama.
