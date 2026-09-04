# 📝 Daily Work Report - Idham (2026-09-04)

---

## 📅 Laporan Harian - 4 September 2026

---

## 🌿 Branch: `issue-244` — Mode "Generated" untuk Dokumen Direktur

### 📌 Informasi Issue

- **Nomor Issue**: #244
- **Judul Issue**: Implementasi Dokumen Direktur
- **Status Branch**: `Belum di-merge` (sudah di-push ke `origin/issue-244`) — menambah kemampuan baru ke modul Dokumen Direktur yang sudah ada sebelumnya (mode unggah PDF), bukan modul baru dari nol.

### 📅 Rincian Perubahan

#### [d701d53a] - save #244 - 4 September 2026, 18:18:58 WIB

- **Komponen yang Berubah**:
  - [`backend/src/models/direkturDocument.model.js`](backend/src/models/direkturDocument.model.js) — Field baru `mode` (`'upload'` default, atau `'generated'`). Skema baru `ClauseSchema` (klausul bernomor: `label` mis. "KESATU", `title`, `body`, `items` sub-list, `closing`, `has_appointee_table`) dan `AppointeeSchema` (data penunjukan: nama, NIK, jabatan, alamat, area penugasan). Field dokumen tambahan untuk mode generated: `doc_type`, `recitals` (menimbang), `legal_basis` (mengingat), `clauses`, `appointee`, `place_of_signing`, `signing_date`. Field `document` (berkas PDF) tidak lagi wajib di level skema — validasinya dipindah ke controller supaya mode generated bisa dibuat tanpa berkas sama sekali.
  - [`backend/src/controllers/direkturDocument.controller.js`](backend/src/controllers/direkturDocument.controller.js) — `createDirekturDocument` bercabang berdasarkan `mode`: `'upload'` tetap memproses unggahan PDF seperti sebelumnya, `'generated'` mewajibkan minimal satu `clauses` lalu menyimpan field terstruktur lewat `pickGeneratedFields()`. `approveDirekturDocument` disesuaikan: mode generated melewati validasi `position` (halaman/koordinat tanda tangan) dan proses `stampSignatureOnPdf` sepenuhnya — tanda tangan cukup disimpan sebagai gambar tanpa ada berkas PDF yang di-stamp. `updateDirekturDocument` menerima update field terstruktur untuk dokumen mode generated.
  - [`backend/src/services/direkturDocument.service.js`](backend/src/services/direkturDocument.service.js) — Whitelist proyeksi field ditambah seluruh field baru mode generated.
  - [`frontend/src/app/pages/users/document/direktur/constants/direkturTemplates.js`](frontend/src/app/pages/users/document/direktur/constants/direkturTemplates.js) [NEW] — Preset template dokumen, mengikuti pola `customerPKS/constants/pksTemplates.js` yang sudah ada. Dua entri: `sk_pop_pic_marketing` (SK Penunjukan PIC POP & Marketing, lengkap menimbang/mengingat/12 klausul bernomor seperti tugas PIC POP, tugas marketing, batasan kewenangan teknis, larangan menjual layanan mandiri, mekanisme pemasaran, keamanan, pelaporan, sanksi) dan `custom_general` (template kosong/generik untuk disusun bebas).
  - [`frontend/src/app/pages/users/document/direktur/create.jsx`](frontend/src/app/pages/users/document/direktur/create.jsx) — Drawer create dirombak signifikan (354→1120+ baris): layar awal sekarang menampilkan pemilihan mode (kartu "Unggah PDF" vs "Buat dari Template"), lalu merender sub-form sesuai pilihan — form generated berisi pemilih template, editor menimbang/mengingat, editor klausul bernomor (tambah/hapus/urutkan), dan form data penunjukan (appointee).
  - [`frontend/src/app/pages/users/document/direktur/DirekturGeneratedDocumentPreview.jsx`](frontend/src/app/pages/users/document/direktur/DirekturGeneratedDocumentPreview.jsx) [NEW, 346 baris] — Komponen preview/cetak untuk dokumen mode generated: merender halaman HTML gaya A4 (kop, menimbang, mengingat, klausul bernomor, tabel data penunjukan bila ada, tanda tangan overlay) — dicetak lewat `react-to-print` di browser, tanpa generate PDF di server sama sekali.
  - [`frontend/src/app/pages/users/document/direktur/schema/direkturSchema.js`](frontend/src/app/pages/users/document/direktur/schema/direkturSchema.js) — Skema Yup baru `direkturGeneratedSchema`, terpisah dari skema mode upload: `clauses` wajib minimal 1 entri, field lain (recitals, legal_basis, appointee, place_of_signing, signing_date) opsional.
  - [`frontend/src/app/pages/users/document/direktur/useDirekturSignFlow.js`](frontend/src/app/pages/users/document/direktur/useDirekturSignFlow.js) — Untuk dokumen `mode === 'generated'`, langkah pemilihan posisi tanda tangan pada PDF dilewati sepenuhnya — begitu admin memilih/menggambar tanda tangan, `onApprove` langsung dipanggil (tidak ada PDF untuk ditempeli posisi).
  - [`frontend/src/app/pages/users/document/direktur/ReviewDrawer.jsx`](frontend/src/app/pages/users/document/direktur/ReviewDrawer.jsx), [`frontend/src/app/pages/public/ReviewDirekturDocumentPage.jsx`](frontend/src/app/pages/public/ReviewDirekturDocumentPage.jsx) — Disesuaikan mengikuti alur sign baru di atas untuk kedua entry point (drawer di tab Document, dan halaman dari link Telegram).
  - [`frontend/src/app/pages/users/document/direktur/schema/columns.jsx`](frontend/src/app/pages/users/document/direktur/schema/columns.jsx) — Kolom baru "Mode" di tabel list, badge berwarna beda untuk dokumen Upload vs Generated.
  - [`frontend/src/components/shared/DocumentPreviewModal.jsx`](frontend/src/components/shared/DocumentPreviewModal.jsx) — Cabang `type === 'direktur-document'` sekarang memilih komponen preview sesuai `mode` dokumen: `DirekturGeneratedDocumentPreview` untuk generated, komponen `DirekturDocumentPreview` lama untuk upload.
  - [`backend/src/locales/{en,id}/translation.json`](backend/src/locales/id/translation.json), [`frontend/src/i18n/locales/{en,id}/translations.json`](frontend/src/i18n/locales/id/translations.json) — String baru untuk pemilih mode, label template, dan validasi field mode generated (mis. `direktur.generated.clausesRequired`).
- **Deskripsi Perubahan & Fungsi**:
  - Menambahkan cara kedua membuat Dokumen Direktur: selain mengunggah PDF siap tanda tangan (mode lama), admin sekarang bisa **menyusun dokumen dari template terstruktur** (menimbang, mengingat, klausul bernomor, data penunjukan) langsung di form, dirender sebagai halaman cetak A4 di browser — tidak perlu menyiapkan PDF di luar sistem lebih dulu.
  - Alur persetujuan & tanda tangan tetap satu langkah (approve+sign digabung) seperti mode upload, tapi untuk dokumen generated tanda tangan hanya berupa gambar overlay pada tampilan HTML (bukan stempel ke berkas biner PDF) — sehingga langkah pemilihan posisi tanda tangan pada halaman PDF otomatis dilewati.
  - Desain field & template sengaja meniru pola yang sudah terbukti dipakai di modul Customer PKS (`pksTemplates.js`) supaya konsisten dan mudah dikembangkan lagi untuk jenis SK/dokumen lain di masa depan.
