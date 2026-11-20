# 📝 Daily Work Report - Idham (2026-11-20)

---

## 📅 Laporan Harian - 20 November 2026

---

## 🌿 Branch: `issue-576` — Mobile Frontend Modul F: Akun (Profil & Dokumen Mitra)

### 📌 Informasi Issue

- **Nomor Issue**: #576
- **Judul Issue**: Mobile Frontend: modul Akun (Profil & Pengaturan, Dokumen Mitra) + batas ukuran unggah dokumen di Partner API
- **Status Branch**: `Belum di-merge`.
  - Commit `resolve #576` (`8bed325c`) berada di atas `master` terbaru (setelah `resolve #565`) dan sudah ada di `origin/issue-576`.
  - Commit ini menggabungkan dua commit `save #576` (`0dc43e85` 16:21 dan `b1b1f49b` 16:50). Rinciannya di bawah dikelompokkan per berkas.

### 📅 Rincian Commit

#### [8bed325c] - resolve #576 - 20 November 2026, 16:56

- **Ringkasan**:
  - **Halaman Profil** (`ProfilePage`) kini memakai data asli Partner API, bukan data contoh:
    - Profil dimuat ulang setiap kali halaman dibuka.
    - Ada kartu **Informasi Akun** dan, untuk mitra, kartu **Tim Dukungan** (petugas teknis & penagihan).
    - Tiga tombol aksi bundar menghubungi petugas dukungan teknis (mitra) atau kontak sendiri (admin).
    - Keluar kini meminta konfirmasi.
  - **Halaman baru Dokumen Mitra** (`/profile/documents`, khusus mitra):
    - Lima jenis dokumen legalitas dengan indikator kelengkapan n/5.
    - Gambar dan PDF bisa dipratinjau di dalam aplikasi, lalu diunduh.
    - Unggah/ganti dari kamera atau galeri.
  - **PDF ditampilkan lewat pdf.js** (`react-pdf`), bukan `<iframe>`, karena Chrome Android tidak bisa menampilkan PDF di dalam halaman.
  - **Backend Partner API** (`/p-api/v1/partners/documents`):
    - Unggahan dokumen kini dibatasi 15 MB. Sebelumnya tidak dibatasi sama sekali, dan berkas yang dipotong middleware tersimpan rusak sambil menghapus berkas lama.
    - Gagal membaca database kini dijawab 500, bukan daftar kosong palsu.
    - Daftar dokumen kini membawa `mime` dan `uploaded_at`.

- **Komponen yang Berubah & Penjelasannya**:

  - **Backend**
    - `backend/src/models/partnerDocument.model.js`
      - Konstanta baru `PARTNER_DOCUMENT_MAX_SIZE` (15 MB). Satu sumber angka untuk unggahan admin dan Partner API, supaya kedua jalur tidak bisa berbeda lagi.
    - `backend/src/controllers/partnerDocument.controller.js`
      - Konstanta lokal `MAX_DOCUMENT_SIZE` diganti `PARTNER_DOCUMENT_MAX_SIZE`.
    - `backend/src/controllers/partnerApiPartner.controller.js`
      - `setPartnerAppDocument`: berkas > 15 MB **atau** `truncated` ditolak 400 (`partner.document.fileTooLarge`), dan berkas sementaranya dihapus.
        - `truncated` perlu diperiksa karena middleware `fileUpload` global (20 MB, tanpa `abortOnLimit`) memotong berkas besar lalu meneruskannya, bukan menolaknya.
        - Berkas sementara juga dihapus saat format berkas ditolak.
      - `getPartnerAppDocuments`:
        - Memanggil service dengan `throwOnError`, sehingga kegagalan database dijawab 500.
        - Respons mendapat field tambahan `mime` dan `uploaded_at`. Klien lama tidak terpengaruh.
    - `backend/src/services/partnerDocument.service.js`
      - `findPartnerDocumentsLegacyShape` menerima opsi `{ throwOnError }`:
        - Portal mitra mencatat `logger.error` lalu melempar `partner.document.failedGetList`.
        - Pemanggil lain (profil) tetap menelan error seperti sebelumnya, supaya profil tetap tampil.
      - Ikut mengembalikan `mime` dan `created_at`.
    - `backend/src/routes/partnerApi.route.js`
      - Swagger diperbarui: field `type`, `mime`, `uploaded_at`, respons 500, dan batas 15 MB pada keterangan 400.
  - **Mobile frontend — Profil**
    - `mobile-frontend/src/pages/ProfilePage.jsx`
      - Ditulis ulang dengan data asli: kartu Informasi Akun dan Tim Dukungan, tombol aksi `tel:`/`mailto:`/WhatsApp (nonaktif bila kontak kosong), tautan ke Dokumen (mitra), pilihan tema/bahasa, dan konfirmasi keluar.
      - Header disamakan dengan tab menu bawah lain.
    - `mobile-frontend/src/features/account/profileView.js` (baru)
      - Penyusun tampilan murni.
      - Menyusun baris Informasi Akun (baris kosong disembunyikan; telepon, tanggal, dan tipe mitra diformat).
      - Memilih kontak tombol aksi: petugas teknis, lalu petugas penagihan bila petugas teknis tidak punya kontak. Admin memakai kontaknya sendiri.
    - `mobile-frontend/src/features/authSlice.js`, `mobile-frontend/src/hooks/useAuth.js`
      - Thunk baru `refreshProfile`: memuat ulang profil diam-diam dan bisa dibatalkan lewat `signal` saat halaman ditutup.
        - Bila gagal, profil lama dipertahankan.
        - Pengguna yang keluar selagi permintaan berjalan tidak dihidupkan lagi, dan peran tidak bisa berganti lewat penyegaran.
    - `mobile-frontend/src/services/authApi.js`
      - `getProfile` menerima `signal`.
    - `mobile-frontend/src/utils/normalizeUser.js`
      - Menambah `created_at` (tanggal bergabung) dan `supports` (petugas teknis/penagihan) untuk mitra.
  - **Mobile frontend — Dokumen Mitra**
    - `mobile-frontend/src/pages/profile/DocumentsPage.jsx` (baru)
      - Daftar lima jenis dokumen beserta kelengkapan, pratinjau, unduh, dan unggah/ganti.
      - Hanya satu unggahan yang boleh berjalan dalam satu waktu: ketukan ganda bisa menghasilkan dua dokumen untuk tipe yang sama di server.
      - Alamat Blob pratinjau dicabut saat halaman ditinggalkan.
      - Daftar dimuat ulang setelah unggah, karena respons unggah berbentuk lama.
      - Admin tidak membuka halaman ini (dokumen mitra milik modul H).
    - `mobile-frontend/src/pages/profile/components/DocumentCard.jsx`, `DocumentCompleteness.jsx` (baru)
      - Kartu per jenis dokumen (status, tanggal unggah, tombol Lihat/Ganti) dan bilah kelengkapan n/5.
    - `mobile-frontend/src/features/account/documentTypes.js`, `documentSummary.js` (baru)
      - Lima jenis dokumen (sama dengan backend dan `partner-frontend`).
      - Hitungan kelengkapan: dokumen tanpa `type` (unggahan admin berlabel bebas) dibuang, sama dengan web. Bila satu tipe punya lebih dari satu dokumen, yang terbaru dipakai.
    - `mobile-frontend/src/services/accountApi.js` (baru)
      - `listPartnerDocuments`, `downloadPartnerDocument` (Blob, karena endpoint butuh Bearer), `uploadPartnerDocument`.
    - `mobile-frontend/src/components/FileSourceSheet.jsx` (baru)
      - Panel pilihan **Ambil Foto** (kamera belakang) atau **Pilih Berkas**, dengan dua input terpisah: tanpa itu sebagian Android langsung membuka penyimpanan.
      - Jenis berkas ditulis eksplisit supaya iOS mengonversi foto HEIC ke JPEG.
    - `mobile-frontend/src/components/FilePreviewSheet.jsx`, `PdfViewer.jsx` (baru)
      - Pratinjau gambar dan PDF di dalam aplikasi.
      - PDF digambar ke canvas lewat pdf.js, dengan lebar halaman mengikuti lebar panel.
      - `PdfViewer` dimuat lazy, jadi pdf.js tidak ikut bundel awal.
    - `mobile-frontend/src/utils/fileValidation.js`, `blobFile.js` (baru)
      - Validasi klien sebelum unggah (JPG/PNG/PDF, maks. 15 MB), aturannya sama persis dengan backend.
      - Helper unduh/buka Blob.
    - `mobile-frontend/package.json`, `package-lock.json`
      - Dependensi baru `pdfjs-dist` 5.4.296 dan `react-pdf` ^10.4.1, versi sama dengan `partner-frontend`.
    - `mobile-frontend/src/App.jsx`
      - Rute baru `/profile/documents` (lazy, `ProtectedRoute`).
    - `mobile-frontend/src/data/mockData.js`
      - Data contoh profil/dokumen dihapus (aturan §2.M.8).
    - `mobile-frontend/src/i18n/locales/{id,en}/translations.json`
      - Key baru untuk Profil, Tim Dukungan, Dokumen, pratinjau, dan validasi berkas.
    - `mobile-frontend/PRD.md`
      - Baris modul F diperbarui dan status modul 5 diubah menjadi "#576 — Selesai, menunggu uji perangkat & PR".
  - **Partner frontend**
    - `partner-frontend/src/app/pages/account/document/configs.js`, `src/i18n/locales/{id,en}/translations.json`
      - Batas unggah disamakan dari 10 MB menjadi 15 MB, sesuai backend.
  - **Test**
    - `backend/test/integration/partnerApiPartner.uploadDocuments.test.js`
      - +3 test: berkas > 15 MB ditolak tanpa menyentuh storage maupun dokumen lama, berkas `truncated` ditolak, dan berkas tepat 15 MB diterima.
    - `backend/test/integration/partnerApiPartner.documents.test.js`
      - +2 test: daftar membawa `type`/`mime`/`uploaded_at`, dan DB error dijawab 500.
    - `mobile-frontend/src/features/account/account.test.js` (baru)
      - 15 test untuk penyusun profil, kontak tombol aksi, dan kelengkapan dokumen.
    - `mobile-frontend/src/utils/fileValidation.test.js` (baru)
      - 9 test validasi berkas.
    - `mobile-frontend/src/features/authSlice.test.js`
      - +5 test `refreshProfile`: profil baru menggantikan, gagal mempertahankan profil lama, bisa dibatalkan, tanpa pengguna tidak memanggil server, dan pengguna yang sudah keluar tidak dihidupkan lagi.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #576 | `0dc43e85`, `b1b1f49b` save; `8bed325c` resolve #576 | Modul Akun di mobile: Profil dengan data asli (Informasi Akun, Tim Dukungan, tombol hubungi, konfirmasi keluar), halaman Dokumen Mitra (kelengkapan n/5, pratinjau gambar/PDF lewat pdf.js, unggah dari kamera/galeri); batas unggah 15 MB + penolakan berkas terpotong di Partner API dan daftar dokumen 500 saat DB gagal; +34 test |

> Catatan merge: `resolve #570` dan `resolve #565` (dikerjakan 19 November) sudah di-merge ke `master` hari ini, masing-masing pukul 08:55 dan 10:13/11:39. Kedua merge memuat penyesuaian dari Dedy S.N Putra.
>
> Catatan branch: `issue-576` sudah berada di atas `master` terbaru (`resolve #565`). Setelah itu `resolve #569` dari Dedy masuk (17:09), dan simulasi merge (`git merge-tree`) menunjukkan konflik di `mobile-frontend/PRD.md` (tabel status modul), jadi branch perlu di-rebase sebelum di-merge. Menurut PRD, modul F masih menunggu uji di perangkat nyata.
