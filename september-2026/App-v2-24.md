# 📝 Daily Work Report - Idham (2026-09-24)

---

## 📅 Laporan Harian - 24 September 2026

---

## 🌿 Branch: `issue-335` — Klausul Tambahan Kontrak Broadband

### 📌 Informasi Issue

- **Nomor Issue**: #335 (berdiri sendiri)
- **Judul Issue**: Tambahan Kontrak Broadband
- **Status Branch**: `resolve #335`, sudah di-push ke `origin/issue-335` — perubahan kecil dan bersih (2 file, 22 baris).

### 📅 Rincian Perubahan

#### [069e822b] - resolve #335 - 24 September 2026, 11:03:23 WIB

- **Komponen yang Berubah**:
  - [`frontend/src/app/pages/public/PublicBAPDocument.jsx`](frontend/src/app/pages/public/PublicBAPDocument.jsx), [`frontend/src/app/pages/tickets/installation/components/InstallationDocumentPreview.jsx`](frontend/src/app/pages/tickets/installation/components/InstallationDocumentPreview.jsx) — Dua klausul baru ditambahkan ke teks ketentuan pada dokumen BAP (Berita Acara Pemasangan) broadband, diterapkan identik di kedua tempat (halaman publik yang ditandatangani pelanggan, dan pratinjau internal di tiket instalasi):
    1. **Masa berlangganan minimum**: berlaku 12 bulan, dengan penalti (harga paket bulanan × sisa masa berlangganan) bila pelanggan mengakhiri layanan lebih awal.
    2. **Larangan jual-beli/modifikasi**: pelanggan dilarang menjual kembali atau memodifikasi perangkat/layanan tanpa persetujuan tertulis perusahaan.
- **Deskripsi Perubahan & Fungsi**: Memperkuat perlindungan kontraktual perusahaan pada dokumen serah-terima instalasi broadband — mengunci komitmen berlangganan minimum dan mencegah penyalahgunaan perangkat, dituangkan langsung ke dokumen yang ditandatangani pelanggan saat instalasi.

---

## 🌿 Branch: `issue-337` — Pengajuan Cuti & Izin Ditinjau per Request (bukan per Hari)

### 📌 Informasi Issue

- **Nomor Issue**: #337 (berdiri sendiri)
- **Judul Issue**: Pengajuan Cuti & Izin per request
- **Status Branch**: `resolve #337`, sudah di-push ke `origin/issue-337`.
- **⚠️ Berlanjutnya risiko konvergensi di modul Absensi**: ini branch **ketiga** dalam tiga hari terakhir yang mengubah inti modul Absensi (`attendance.controller.js`, `attendance.route.js`, `attendance.service.js`, model absensi) secara independen — setelah `issue-330` (22 September, privilege `readSensitive`) dan `issue-333` (23 September, endpoint self-service). Dikonfirmasi lewat `git merge-base`: baik `issue-330` maupun `issue-333` **bukan** leluhur dari commit hari ini — ketiganya bercabang dari titik yang sama dan belum saling mengenal satu sama lain. Modul ini sebaiknya jadi prioritas untuk direkonsiliasi sebelum bertambah cabang lagi.

### 🧭 Latar Belakang

Sebelum perubahan ini, satu pengajuan cuti/izin yang mencakup beberapa hari (mis. cuti 3 hari) tersimpan sebagai **3 dokumen terpisah** tanpa penanda bahwa ketiganya satu pengajuan yang sama — halaman Pengajuan Izin/Cuti menampilkan 3 baris berbeda untuk satu permintaan, dan HRD menyetujui/menolak **per hari**, bukan per pengajuan. Hari ini pengajuan mendapat "dokumen induk" sehingga tampil sebagai satu baris per pengajuan, dengan opsi menyetujui sebagian hari dan menolak sebagian lainnya dalam satu pengajuan yang sama (status `partial`).

### 📅 Rincian Perubahan

#### [9dd215cc] - resolve #337 - 24 September 2026, 17:57:22 WIB

- **Komponen yang Berubah**:
  - [`backend/src/models/attendancePermitRequest.model.js`](backend/src/models/attendancePermitRequest.model.js) [NEW, 87 baris] + [`attendanceAbsenceRequest.model.js`](backend/src/models/attendanceAbsenceRequest.model.js) [NEW, 92 baris] — Dua koleksi baru sebagai **dokumen induk** (`permission_requests`/`vacation_requests`): menyimpan `start_date`/`end_date`/`total_days`, ringkasan `accepted_days`/`rejected_days` (ditulis ulang tiap ada keputusan supaya bisa difilter/disortir di tabel tanpa agregasi per baris), dan `status` (`wait`/`accepted`/`rejected`/`partial`). Dokumen harian yang sudah ada (koleksi `vacations`/`permissions`) **tetap dipertahankan apa adanya** — tidak dihapus atau digabung — karena kuota cuti dan slip gaji menghitungnya per tanggal; dokumen harian ini sekarang menautkan balik ke dokumen induknya.
  - [`backend/src/models/attendanceAbsence.model.js`](backend/src/models/attendanceAbsence.model.js), [`attendancePermit.model.js`](backend/src/models/attendancePermit.model.js) — Field referensi ke dokumen induk ditambahkan ke skema harian yang sudah ada.
  - [`backend/scripts/backfill-attendance-requests.js`](backend/scripts/backfill-attendance-requests.js) [NEW, 243 baris] — Script migrasi **idempoten** (aman dijalankan ulang, dokumen yang sudah punya induk dilewati) untuk mengelompokkan dokumen harian lama menjadi dokumen induk: dikelompokkan berdasarkan kombinasi `admin` + `title` + `reason` + `created_at` yang dibulatkan ke menit (satu pengajuan lahir dari satu proses, jadi selisih waktu antar harinya cuma milidetik). Mendukung mode `--dry-run`. Sengaja ditulis memakai model Mongoose (bukan driver database mentah) supaya nomor urut dokumen induk (`permission_request_id`/`vacation_request_id`) tetap konsisten dengan cara aplikasi biasa membuatnya, dan validasi skema tetap berlaku.
  - [`backend/src/controllers/attendance.controller.js`](backend/src/controllers/attendance.controller.js) [+353/-x, refactor besar] — Handler baru `reviewPermissionRequest`/`reviewPaidLeaveRequest`, dibangun dari factory bersama `createReviewHandler()`: menerima daftar keputusan per hari (`days`) dalam satu pengajuan, memvalidasi format ID dan **versi dokumen** (`__v`, mekanisme *optimistic concurrency* — mencegah dua penyetuju memproses pengajuan yang sama secara bersamaan dan saling menimpa), lalu mengirim notifikasi keputusan ke karyawan terkait.
  - [`backend/src/services/attendance.service.js`](backend/src/services/attendance.service.js) [+333/-x] — Logika inti pengelompokan permintaan, perhitungan ulang `accepted_days`/`rejected_days`/`status` (termasuk `partial`), dan query datatable yang sekarang mengembalikan satu baris per pengajuan.
  - [`backend/src/routes/attendance.route.js`](backend/src/routes/attendance.route.js) [+262] — Endpoint review baru untuk permintaan izin & cuti per-request.
  - [`frontend/src/app/pages/activities/components/AttendanceRequestDetailModal.jsx`](frontend/src/app/pages/activities/components/AttendanceRequestDetailModal.jsx) [502 baris berubah, perombakan besar] — Modal review sekarang menampilkan seluruh hari dalam satu pengajuan sekaligus, dengan **toggle keputusan per hari** (Setuju/Tolak individual), tombol pintas "Setujui Semua"/"Tolak Semua", penghitung langsung jumlah hari disetujui vs ditolak, dan badge status termasuk **"Sebagian"** (partial) berwarna berbeda dari disetujui/ditolak penuh.
  - [`frontend/src/app/pages/activities/paidLeave/`](frontend/src/app/pages/activities/paidLeave/), [`permission/`](frontend/src/app/pages/activities/permission/), [`frontend/src/app/pages/users/components/UserAttendanceTabs.jsx`](frontend/src/app/pages/users/components/UserAttendanceTabs.jsx) — Tabel daftar & kolom disesuaikan menampilkan satu baris per pengajuan (bukan lagi per hari).
  - [`frontend/src/components/shared/table/rows.jsx`](frontend/src/components/shared/table/rows.jsx) [+95], [`status.js`](frontend/src/components/shared/table/status.js) [+5] — Komponen sel/badge status baru untuk mendukung tampilan `partial`.
  - [`backend/test/integration/attendanceRequestReview.test.js`](backend/test/integration/attendanceRequestReview.test.js) [NEW, 248 baris] — Menguji alur review per-request: persetujuan penuh, penolakan penuh, persetujuan sebagian (status `partial`), dan penegakan versi dokumen (mencegah race condition antar penyetuju).
- **Deskripsi Perubahan & Fungsi**:
  - Mengubah cara HRD meninjau pengajuan cuti/izin multi-hari dari "satu keputusan per hari, terpisah-pisah" menjadi "satu pengajuan, satu layar review" — dengan tetap mempertahankan fleksibilitas menyetujui sebagian hari dan menolak sebagian lainnya bila diperlukan (mis. cuti 5 hari diajukan, hanya 3 hari yang disetujui).
  - Migrasi data lama dirancang aman (idempoten, mode dry-run, tidak menghapus data asli) mengingat data absensi terhubung ke perhitungan kuota cuti dan slip gaji yang sensitif terhadap kesalahan.

---

## ⚠️ Catatan Risiko Gabungan

Modul Absensi kini punya **3 branch independen** yang saling tumpang tindih di berkas inti yang sama (`attendance.controller.js`, `attendance.route.js`, `attendance.service.js`, model-model absensi) selama tiga hari terakhir:

| Branch | Tanggal | Fokus |
| --- | --- | --- |
| `issue-330` | 22 September | Privilege `attendance.readSensitive` — batasi lihat data karyawan lain |
| `issue-333` | 23 September | Endpoint `/attendance/self/*` — perbaikan akses halaman Profil sendiri |
| `issue-337` | 24 September | Model dokumen induk pengajuan — review per-request, bukan per-hari |

Ketiganya secara desain **saling melengkapi** (bukan saling bertentangan tujuannya), tapi karena dikerjakan di branch terpisah tanpa saling rebase, penggabungannya nanti kemungkinan besar butuh kerja rekonsiliasi manual yang tidak kecil — terutama karena `issue-337` merombak besar-besaran fungsi yang sama yang disentuh dua branch sebelumnya. Semakin lama ditunda, semakin besar potensi konfliknya. Disarankan salah satu (kemungkinan `issue-330`, yang paling awal) dijadikan basis, lalu `issue-333` dan `issue-337` di-rebase berurutan di atasnya sebelum review lebih lanjut.
