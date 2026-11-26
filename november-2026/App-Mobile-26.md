# 📝 Daily Work Report - Idham (2026-11-26)

---

## 📅 Laporan Harian - 26 November 2026

---

## 🌿 Branch: `issue-7` — Perbaikan Duplikasi & Dependency

### 📌 Informasi Issue

- **Nomor Issue**: #7
- **Judul Issue**: Perbaikan Duplikasi & Dependency
- **Status Branch**: `Sudah di-merge` (merge commit `04a6ff2` — "resolve #7", 26 Nov 2026 17:33)

### 📅 Rincian Commit

#### [15e2809] - flutter_html version #7 - 26 Nov 2026 17:22

- **Komponen yang Berubah**:
  - `pubspec.yaml`
  - `pubspec.lock`
- **Deskripsi Perubahan & Fungsi**:
  - Menambahkan `dependency_overrides` untuk mengunci package `html` di versi `0.15.6`.
  - `flutter_html 3.0.0` memakai fungsi internal `matches` dari `package:html` yang sudah dihapus di `html 0.15.7`, sehingga build gagal. Override ini menjaga build tetap berjalan sampai `flutter_html` merilis versi yang kompatibel (sudah diberi catatan di `pubspec.yaml` agar dihapus nanti).

#### [77719e8] - hapus dependency #7 - 26 Nov 2026 17:22

- **Komponen yang Berubah**:
  - `pubspec.yaml`
  - `pubspec.lock`
  - `ios/Podfile.lock`
  - `macos/Flutter/GeneratedPluginRegistrant.swift`
  - `windows/flutter/generated_plugin_registrant.cc`
  - `windows/flutter/generated_plugins.cmake`
- **Deskripsi Perubahan & Fungsi**:
  - Menghapus dependency yang tidak dipakai: `riverpod` (duplikat dari `flutter_riverpod`), `freezed_annotation`, `flutter_inappwebview`, `flutter_pdfview`, `jwt_decoder`, dan `connectivity_plus`.
  - Membersihkan komentar bawaan template Flutter di `pubspec.yaml` dan mengganti deskripsi aplikasi menjadi "Aplikasi mobile pelanggan RingNet."
  - Plugin registrant (macOS/Windows) dan `Podfile.lock` ikut diperbarui. Ukuran aplikasi dan waktu build berkurang karena plugin native yang tidak terpakai sudah tidak ikut dikompilasi.

#### [265065c] - refactor dio #7 - 26 Nov 2026 17:22

- **Komponen yang Berubah**:
  - `lib/config/dio_provider.dart`
  - `lib/services/auth_service.dart`
  - `lib/services/invoice_api_service.dart`
  - `test/dio_provider_test.dart` [NEW]
- **Deskripsi Perubahan & Fungsi**:
  - Menambahkan fungsi `createBaseDio()` di `dio_provider.dart` sebagai satu-satunya tempat konfigurasi dasar Dio (baseUrl, timeout, header default). `dioProvider` sekarang dibangun dari fungsi ini.
  - `AuthService` sebelumnya punya instance Dio sendiri dengan interceptor auth yang menduplikasi logika di `dioProvider` (inject token, validasi sesi, penanganan 401, pemetaan URL → nama fitur). Duplikasi itu dihapus (±180 baris): `AuthService` kini memakai `createBaseDio()` tanpa interceptor, karena login tidak butuh token dan logout mengirim token lama secara eksplisit. Aturan sesi dan 401 sekarang hanya ada di `dioProvider`.
  - Menghapus method yang tidak lagi terpakai di `AuthService`: `_handleTokenExpired`, `_getActiveToken`, `_handleUnauthorized`, `validateCurrentToken`, `_getFeatureFromUrl`.
  - Menambahkan komentar di `InvoiceApiService` yang menjelaskan alasan service ini tetap memakai Dio sendiri (timeout 120 detik untuk download PDF, mengikuti redirect, `validateStatus` khusus).
  - Menambahkan unit test `dio_provider_test.dart` untuk pemetaan path API ke nama fitur (`getFeatureFromUrl`).

---

## 🌿 Branch: `issue-5` — [ Update ] Refactor aplikasi

### 📌 Informasi Issue

- **Nomor Issue**: #5
- **Judul Issue**: [ Update ] Refactor aplikasi
- **Status Branch**: `Sudah di-merge` (merge commit `a998998` — "resolve #5", 26 Nov 2026 16:59)

### 📅 Rincian Commit

#### [cef7b5e] - resolve #5 - 26 Nov 2026 16:48 (ditulis 19 Okt 2026, di-commit ulang & di-merge hari ini)

- **Komponen yang Berubah** (73 file, utama):
  - `.vscode/launch.json` [NEW]
  - `lib/utils/image_type_utils.dart` [NEW]
  - `lib/config/api_config.dart`
  - `lib/config/dio_provider.dart`
  - `lib/main.dart`
  - `android/app/build.gradle.kts`
  - `lib/services/*` (auth, account_storage, invoice_api, payment, pdf, transaction, version)
  - `lib/screens/*` (auth, home, profile, service, ticket, transaction)
  - `lib/widgets/*`, `lib/providers/*`, `lib/models/*`
  - `pubspec.yaml`, `pubspec.lock`, `README.md`
- **Deskripsi Perubahan & Fungsi**:
  - **Pemilihan environment saat build**: `ApiConfig` sekarang membaca `--dart-define=ENV=development`. Default-nya production (`v2.ring.net.id`). Mode development memakai `dev-srv.dekadata.net` dan `srv.dekadata.net`. Sebelumnya URL dev/prod diganti manual dengan comment/uncomment.
  - **Launch config VS Code**: tersedia dua profil, "RingNet (Production)" dan "RingNet (Development)".
  - **Header auth**: header `Authorization: Bearer ...` yang ganda dihapus. Backend hanya menerima token mentah di header `authorization`.
  - **Signing Android**: konfigurasi `release` hanya dibuat jika `android/key.properties` ada. Tanpa file itu, build jatuh ke debug signing dan memberi peringatan, sehingga project tetap bisa di-build di mesin lain atau di CI.
  - **Logging**: environment aktif dan base URL dicatat saat aplikasi start.
  - Refactor dan perapian di banyak screen, service, dan widget, plus utilitas baru `image_type_utils.dart`.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul                             | Dampak Utama                                                                                                     |
| ----- | --------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| #7    | Perbaikan Duplikasi & Dependency  | Interceptor auth tidak lagi terduplikasi, 6 dependency tak terpakai dihapus, build `flutter_html` diperbaiki     |
| #5    | [ Update ] Refactor aplikasi      | Environment dev/prod dipilih saat build, header auth diseragamkan, build Android bisa jalan tanpa keystore        |

### Kemampuan Baru Pengguna/Admin

- Developer bisa menjalankan aplikasi ke server development cukup dengan `--dart-define=ENV=development` atau profil "RingNet (Development)" di VS Code, tanpa mengubah kode.
- Project bisa di-build (termasuk `flutter run --release`) di mesin tanpa keystore produksi.

### Bug Fix / Solusi Masalah

- Build gagal karena `flutter_html 3.0.0` tidak kompatibel dengan `html 0.15.7`: diperbaiki dengan override ke `html 0.15.6`.
- Penanganan sesi dan 401 yang terduplikasi di `AuthService` dan `dioProvider` (bisa memicu perilaku logout/popup ganda) sekarang terpusat di `dioProvider`.
- Header `Authorization` ganda (Bearer + token mentah) diseragamkan menjadi satu format.

### Menu/Fitur Baru

- Tidak ada menu baru untuk pengguna akhir. Perubahan hari ini berupa refactor, perbaikan dependency, dan tooling developer.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Konfigurasi environment berbasis `--dart-define`. `ApiConfig.environmentName` dibaca saat compile; `baseUrl` dan `invoiceDownloadBaseUrl` otomatis menyesuaikan. Semua request API (kecuali login/logout dan download invoice) melewati satu Dio bersama dari `dioProvider`, yang menangani token dan sesi.
- **Langkah Penggunaan (Tutorial)**:
  1. Production (default): `flutter run` atau `flutter build apk --release`.
  2. Development: `flutter run --dart-define=ENV=development`, atau pilih profil **RingNet (Development)** di panel Run & Debug VS Code.
  3. Cek log saat start: `🌐 Environment: development → https://dev-srv.dekadata.net/m-api/v1`.
  4. Jalankan unit test baru: `flutter test test/dio_provider_test.dart`.
