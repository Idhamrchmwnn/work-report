# 📝 Daily Work Report - Idham (2026-09-22)

---

## 📅 Laporan Harian - 22 September 2026

---

## 🌿 Branch: `issue-330` — Pembatasan Hak Akses Data Presensi Antar-Karyawan

### 📌 Informasi Issue

- **Nomor Issue**: #330 (berdiri sendiri)
- **Judul Issue**: Limitasi hak akses presensi [mb lia]
- **Status Branch**: `resolve #330` — sudah di-push ke `origin/issue-330`, satu commit bersih (tanpa noise rebase).

### 📅 Rincian Perubahan

#### [c18d195f] - resolve #330 - 22 September 2026, 22:09:11 WIB

- **Latar belakang**: sebelumnya, siapa pun yang punya privilege dasar `attendance.list`/`attendance.read` bisa melihat data kehadiran, pengajuan izin, dan pengajuan cuti **seluruh karyawan lain** — bukan cuma miliknya sendiri. Penyaringan per-karyawan yang ada (`resolveAdminFilter.js`, kini dihapus total) hanya berfungsi sebagai *pencarian* (mempersempit ke satu karyawan atas permintaan), bukan *pembatasan* (memaksa hanya melihat diri sendiri bila tidak berhak lebih). Pekerjaan hari ini menambahkan privilege baru **`attendance.readSensitive`** sebagai syarat untuk melihat data siapa pun selain diri sendiri, ditegakkan di server pada setiap endpoint terkait — bukan disembunyikan di UI saja.
- **Komponen yang Berubah**:
  - [`backend/src/config/privilege.json`](backend/src/config/privilege.json) — Privilege baru `attendance.readSensitive`, terpisah dari `attendance.read`/`attendance.list` yang sudah ada.
  - [`backend/src/controllers/attendance.controller.js`](backend/src/controllers/attendance.controller.js) — Helper baru `resolveRequestOwnershipFilter()`: tanpa `readSensitive`, pemanggil dipaksa `{admin: req.user._id}` (data sendiri saja); dengan `readSensitive`, boleh menyempitkan ke satu karyawan tertentu lewat `find.adminId` (aman karena hanya bisa mempersempit, tidak pernah memperluas). **Detail keamanan penting**: `params.find`/`params.populate` yang datang dari body klien sekarang **dihapus paksa** sebelum diproses — filter kepemilikan yang sebenarnya dikirim lewat parameter server terpisah (`extraFind`), bukan digabung dengan apa pun yang dikirim klien, supaya klien tidak bisa menimpa batasannya sendiri. Diterapkan ke `getPermissionList`, `getPaidLeaveList`, `getAttendanceList`, dan `getMonthlyAttendanceSummary` (yang terakhir juga mengembalikan field `scope: 'self'|'all'` eksplisit di response).
  - [`backend/src/controllers/files.controller.js`](backend/src/controllers/files.controller.js) [+69] — **Dua kerawanan nyata ditutup**:
    - **Akses foto/lampiran tanpa otorisasi**: nama berkas foto absensi mengikuti pola yang bisa ditebak (`{admin_id}_IN_{unix}.png`), sehingga sebelum perbaikan ini siapa pun yang tahu/menebak pola tersebut bisa membuka foto absensi (atau lampiran pengajuan izin, mis. surat dokter) milik karyawan lain langsung lewat URL — tanpa privilege apa pun. Sekarang setiap permintaan berkas mencari dulu pemilik sebenarnya di database, lalu memeriksa apakah pemanggil adalah pemiliknya sendiri **atau** pemegang `attendance.readSensitive` — kalau tidak, ditolak. Berkas yang tidak terdaftar dibalas **404** (bukan 403) supaya keberadaannya sendiri tidak ikut terkonfirmasi ke penebak.
    - **Guard path traversal**: `isSafeStoredFileName()` menolak nama berkas yang mengandung `/`, `\`, `..`, kosong, atau lebih dari 255 karakter — nama berkas dipakai langsung sebagai kunci query database maupun nama objek MinIO, jadi tanpa validasi ini berpotensi dipakai menjangkau berkas di luar yang dimaksud.
  - [`backend/src/services/admin.service.js`](backend/src/services/admin.service.js) [+40] — `findAdminsForAttendance()` menerima parameter `scope` baru (`{mode: 'all'}` atau `{mode: 'self', selfId}`), menegakkan pembatasan di level query database untuk matriks kehadiran bulanan.
  - [`backend/src/services/attendance.service.js`](backend/src/services/attendance.service.js) [+97] — Fungsi service untuk datatable izin/cuti/absensi menerima `extraFind` sebagai parameter terpisah dari `params` milik klien; fungsi baru `findPresenceByImageName`/`findPermissionByAttachmentName` untuk lookup pemilik berkas (dipakai `files.controller.js` di atas).
  - [`backend/src/utils/resolveAdminFilter.js`](backend/src/utils/resolveAdminFilter.js) [DELETED] — Utilitas lama yang cuma memetakan `adminId` jadi filter pencarian, tanpa penegakan batasan apa pun, digantikan sepenuhnya oleh `resolveRequestOwnershipFilter()`.
  - Index database baru pada `Presence`/`Permission` (`{admin: 1, date: -1}`, `{status: 1}`) — query yang tadinya sesekali dipakai HRD kini jadi jalur yang dilewati **setiap** karyawan pada setiap muat halaman (karena semua orang sekarang query dirinya sendiri secara rutin), sehingga performanya perlu dijaga.
  - [`frontend/src/app/pages/activities/attendance/index.jsx`](frontend/src/app/pages/activities/attendance/index.jsx) [+90] — Antarmuka beradaptasi menurut `scope` dari response: judul & subjudul halaman berganti ("Absensi Saya" vs "Absensi Karyawan"), filter pencarian-per-nama disembunyikan saat *scope* `self` (karena tidak relevan lagi), dan **default aman**: bila response tidak menyertakan `scope` sama sekali (mis. backend versi lama atau request gagal), tampilan jatuh ke `self` — interpretasi paling sempit, bukan paling luas.
  - [`backend/test/integration/attendanceFileAccess.test.js`](backend/test/integration/attendanceFileAccess.test.js) [NEW, 200 baris], [`attendanceMonthlySummary.scope.test.js`](backend/test/integration/attendanceMonthlySummary.scope.test.js) [NEW, 159 baris], [`attendanceRequestList.scope.test.js`](backend/test/integration/attendanceRequestList.scope.test.js) [NEW, 266 baris] — 625 baris test baru, mencakup: akses foto check-in **dan** check-out, lampiran izin, 404-bukan-403 untuk berkas tak terdaftar, penolakan nama berkas berisi pemisah path, super admin tetap melihat semua tanpa privilege khusus, serta **percobaan klien mengirim `find.admin`/`find.adminId` mentah untuk memintas batasan — semuanya diabaikan server**.
- **Deskripsi Perubahan & Fungsi**:
  - Ini murni perbaikan privasi/kontrol akses, bukan fitur baru — sebelumnya modul absensi tidak punya batas antar-karyawan sama sekali di luar siapa yang boleh membuka modulnya. Sekarang berlaku prinsip *least privilege*: default setiap karyawan hanya melihat datanya sendiri (kehadiran, pengajuan izin/cuti, foto & lampiran terkait), dan melihat data orang lain memerlukan privilege eksplisit yang terpisah dari sekadar "bisa membuka menu Absensi".

---

## 📖 Penjelasan & Tutorial Singkat

### Kenapa perubahan ini penting

Sebelum hari ini, modul Absensi punya asumsi tersembunyi yang keliru: siapa pun yang boleh membuka menu Absensi otomatis dianggap boleh melihat data **semua** karyawan — termasuk foto selfie saat check-in/check-out dan lampiran pribadi seperti surat dokter untuk pengajuan izin sakit. Ini berarti karyawan biasa (bukan HRD) berpotensi melihat, atau bahkan menebak-tebak URL untuk membuka, data kehadiran dan dokumen pribadi rekan kerjanya sendiri — sesuatu yang seharusnya hanya untuk HRD/pihak berwenang.

### Apa yang berubah bagi pengguna

- **Karyawan biasa** (tanpa privilege `attendance.readSensitive`): saat membuka menu **Aktivitas → Absensi**, sekarang hanya melihat **data dirinya sendiri** — judul halaman berubah jadi "Absensi Saya", dan opsi pencarian/filter per-nama karyawan lain tidak lagi muncul karena memang tidak relevan.
- **HRD/pihak yang diberi hak `attendance.readSensitive`**: tidak ada perubahan pengalaman — tetap bisa melihat matriks kehadiran seluruh karyawan, daftar pengajuan izin/cuti semua orang, dan bisa mempersempit ke satu karyawan tertentu lewat pencarian, persis seperti sebelumnya.
- **Super admin**: tidak terpengaruh sama sekali — selalu melihat semua data tanpa perlu privilege tambahan apa pun (perilaku lama tetap dipertahankan untuk role ini).

### Langkah bagi Admin — Memberi Akses Lihat-Semua ke Staf HRD

1. Buka menu **Pengguna → Privilege**.
2. Cari/buat role untuk staf HRD, lalu berikan hak akses baru **`attendance.readSensitive`** (terpisah dari `attendance.list`/`attendance.read`/`attendance.update` yang mungkin sudah ada).
3. Staf dengan role tersebut sekarang bisa melihat data kehadiran & pengajuan seluruh karyawan seperti biasa; staf tanpa hak ini otomatis hanya melihat datanya sendiri — tidak perlu pengaturan tambahan apa pun per-karyawan.
4. **Catatan penting**: hak akses ini sebaiknya diberikan seminimal mungkin (hanya ke HRD/pihak yang memang berwenang) — karena mencakup akses ke data yang cukup sensitif: foto wajah karyawan saat presensi dan lampiran dokumen pribadi seperti surat sakit.
