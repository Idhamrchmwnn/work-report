# 📝 Daily Work Report - Idham (2026-09-23)

---

## 📅 Laporan Harian - 23 September 2026

---

## 🌿 Branch: `issue-333` — Perbaikan Akses Halaman Profil Akun (403 untuk Diri Sendiri)

### 📌 Informasi Issue

- **Nomor Issue**: #333 (berdiri sendiri)
- **Judul Issue**: [Fix Bug] Profil Akun
- **Status Branch**: `resolve #333`, sudah di-push ke `origin/issue-333`, satu commit bersih.
- **⚠️ Penting — branch ini dibangun dari titik yang BERBEDA dari `issue-330`** (dilaporkan 22 September, juga soal pembatasan akses data absensi): dikonfirmasi lewat `git merge-base --is-ancestor`, komit `issue-330` **bukan** leluhur dari komit hari ini. Kedua branch sama-sama mengubah berkas inti yang sama (`attendance.controller.js`, `attendance.route.js`, `attendance.service.js`, model absensi, `privilege.json`, `files.controller.js`) dengan pendekatan yang **berbeda** untuk masalah yang mirip — lihat bagian "Catatan Risiko" di bawah, ini kemungkinan besar akan tabrakan saat digabung.

### 🧭 Latar Belakang

Halaman **Profil Akun** (`/profile`) menampilkan tab riwayat Absensi, Pengajuan Izin, dan Pengajuan Cuti milik pengguna yang sedang login — dimaksudkan bisa dibuka **siapa saja yang login**, tanpa privilege khusus, karena isinya data diri sendiri. Bug-nya: tab-tab ini ternyata memanggil endpoint yang sama dengan yang dipakai HRD untuk melihat data **semua** karyawan (`/attendance/list`, `/attendance/permission/list`, `/attendance/permit/list`, `/admin/read/:id`) — endpoint-endpoint itu digerbang privilege (`attendance.list`, `admin.read`, dst.) yang **tidak dimiliki mayoritas karyawan biasa**. Akibatnya, karyawan tanpa privilege tersebut mendapat error 403 saat sekadar membuka halaman profilnya sendiri.

### 📅 Rincian Perubahan

#### [cbb9fc13] - resolve #333 - 23 September 2026, 18:34:56 WIB

- **Komponen yang Berubah**:
  - [`backend/src/middlewares/privilegeSelf.middleware.js`](backend/src/middlewares/privilegeSelf.middleware.js) [NEW] — Middleware generator `checkPrivilegeOrSelf(key, getTargetId, getSelfId)`: meloloskan request bila target data yang diminta adalah milik pemanggil sendiri (dibandingkan sebagai string, karena `admin_id` bisa tersimpan sebagai angka pada dokumen lama), **atau** pemanggil punya privilege `key`. Kalau tidak keduanya, 403. Komentar di kode menjelaskan kenapa `checkPrivilege` biasa tidak cukup: middleware itu jalan sebelum controller dan tidak pernah tahu **data siapa** yang sebenarnya diminta.
  - [`backend/src/routes/admin.route.js`](backend/src/routes/admin.route.js) — `GET /admin/read/:id` dan `GET /admin/read/points/:id` (dipakai halaman profil untuk identitas & grafik poin) diubah dari `checkPrivilege('admin.read')` (wajib privilege untuk siapa pun) menjadi `checkPrivilegeOrSelf('admin.read', ...)`.
  - [`backend/src/routes/attendance.route.js`](backend/src/routes/attendance.route.js) [+142] — **3 endpoint baru khusus diri-sendiri**, tanpa privilege apa pun selain login (`protectedAdmin` saja): `POST /attendance/self/list`, `POST /attendance/self/permission/list`, `POST /attendance/self/permit/list`. Didokumentasikan eksplisit di Swagger: parameter `find` yang dikirim klien **selalu diabaikan** — cakupannya ditentukan server, tidak ada cara menggesernya ke data karyawan lain. Endpoint lama (`/attendance/list` dkk., butuh `attendance.list`) tidak diubah — tetap jalur terpisah untuk HRD melihat semua karyawan.
  - [`backend/src/controllers/attendance.controller.js`](backend/src/controllers/attendance.controller.js) [+69] — Handler baru `getSelfAttendanceList`, `getSelfPermissionList`, `getSelfPaidLeaveList`, masing-masing memanggil fungsi datatable yang sama dengan versi admin tapi dengan filter tambahan `selfOnlyFilter(req) = { admin: req.user._id }` yang **tidak bisa ditimpa** oleh body request.
  - [`backend/src/controllers/files.controller.js`](backend/src/controllers/files.controller.js) [+20] — `getPermissionFile` (lampiran pengajuan izin, mis. surat dokter): gerbang privilege dipindah dari route ke dalam controller, supaya pemilik lampiran tetap bisa membukanya sendiri dari halaman profil tanpa privilege apa pun, sementara yang bukan pemilik tetap butuh `attendance.read`. Lampiran yang tidak terdaftar dibalas 404 (bukan 403) supaya keberadaannya tidak ikut terkonfirmasi.
  - [`backend/src/services/attendance.service.js`](backend/src/services/attendance.service.js) [+73] — Fungsi datatable menerima filter tambahan sebagai parameter terpisah dari body klien (pola yang sama seperti dijelaskan di laporan 22 September, dikembangkan secara independen di branch ini).
  - [`backend/src/utils/generate-permissions.js`](backend/src/utils/generate-permissions.js) [+19] — Generator daftar privilege (`npm run gp`) disesuaikan supaya route yang memakai `checkPrivilegeOrSelf` tetap terdeteksi dan tercatat sebagai privilege yang valid (sebelumnya generator ini kemungkinan hanya mengenali `checkPrivilege` biasa).
  - [`frontend/src/app/pages/users/components/UserAttendanceTabs.jsx`](frontend/src/app/pages/users/components/UserAttendanceTabs.jsx) [+83/-x] — Komponen tab Absensi/Izin/Cuti yang dipakai bersama di dua tempat: halaman detail karyawan (admin, `isSelf=false`, pakai endpoint lama) dan halaman Profil sendiri (`isSelf=true`, pakai 3 endpoint baru di atas).
  - [`frontend/src/app/pages/profile/index.jsx`](frontend/src/app/pages/profile/index.jsx) — Merender `UserAttendanceTabs` dengan `isSelf`.
  - [`AGENTS.md`](AGENTS.md) [+13/-x] — Panduan internal diperbarui mendokumentasikan pola `checkPrivilegeOrSelf` untuk kasus endpoint self-service serupa di masa depan.
  - [`backend/test/integration/attendanceSelfList.test.js`](backend/test/integration/attendanceSelfList.test.js) [NEW, 242 baris], [`attendancePermissionFileAccess.test.js`](backend/test/integration/attendancePermissionFileAccess.test.js) [NEW, 115 baris], [`backend/test/unit/privilegeSelf.middleware.test.js`](backend/test/unit/privilegeSelf.middleware.test.js) [NEW, 84 baris] — Menguji: karyawan tanpa privilege apa pun tetap bisa mengambil daftar absensi/izin/cuti miliknya sendiri, `find` yang dikirim klien tidak bisa menggeser cakupan, endpoint lama tetap butuh privilege untuk melihat orang lain, akses lampiran izin sendiri vs milik orang lain, dan unit test middleware untuk perbandingan tipe data id (string vs angka).
- **Deskripsi Perubahan & Fungsi**:
  - Menutup bug di mana karyawan tanpa privilege administratif (mayoritas staf) mendapat error saat membuka halaman profilnya sendiri, karena tab riwayat di halaman itu ternyata memanggil endpoint yang sama dengan yang dipakai HRD untuk melihat data semua orang.
  - Pendekatannya **berbeda** dari perbaikan serupa 22 September: di sana endpoint lama dimodifikasi untuk secara kondisional membatasi cakupan berdasarkan privilege baru (`attendance.readSensitive`); di sini endpoint lama **tidak disentuh sama sekali** — dibuatkan endpoint baru yang terpisah, khusus untuk kebutuhan self-service, tanpa privilege sama sekali.

---

## ⚠️ Catatan Risiko: Dua Pendekatan Berbeda untuk Masalah yang Mirip di Dua Branch Terpisah

- `issue-330` (22 September) dan `issue-333` (hari ini) sama-sama menyentuh inti modul Absensi untuk alasan yang berkaitan (kontrol akses data antar-karyawan), tapi dikerjakan di branch yang **tidak beririsan** — komentar di `privilegeSelf.middleware.js` hari ini bahkan menyebut "issue-330" secara eksplisit, namun kode privilege `attendance.readSensitive` dan `resolveRequestOwnershipFilter()` dari issue-330 **tidak ada** di branch ini; `resolveAdminFilter.js` (yang dihapus di issue-330) di branch ini **masih ada dan masih dipakai**.
- **Potensi tabrakan saat digabung**: kedua branch mengubah baris yang sama atau berdekatan di `attendance.controller.js`, `attendance.route.js`, `attendance.service.js`, ketiga model absensi, `privilege.json`, dan `files.controller.js` — Git kemungkinan besar akan menandai konflik langsung pada berkas-berkas ini, dan lebih penting lagi: **kedua solusi perlu direkonsiliasi secara desain**, bukan sekadar digabung otomatis, supaya tidak berakhir dengan dua mekanisme kontrol akses yang tumpang tindih (`checkPrivilegeOrSelf` generik vs `attendance.readSensitive` + endpoint `/self/*` khusus) untuk masalah yang sama.
- Disarankan salah satu branch di-rebase ke atas yang lain sesegera mungkin — sebelum keduanya direview terpisah — supaya penggabungan desainnya dilakukan sadar oleh yang memahami kedua konteks, bukan ditemukan mendadak sebagai konflik generik saat merge.
