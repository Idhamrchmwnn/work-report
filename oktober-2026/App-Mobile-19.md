# 📝 Daily Work Report - Idham (2026-10-19)

---

## 📅 Laporan Harian - 19 Oktober 2026

---

## 🌿 Branch: `issue-5` — [ Update ] Refactor aplikasi

### 📌 Informasi Issue

- **Nomor Issue**: #5
- **Judul Issue**: [ Update ] Refactor aplikasi
- **Status Branch**: `Belum di-merge` (issue masih OPEN, belum ada PR)

### 📅 Rincian Commit

#### [7188e34] - save #5 - Senin, 19 Oktober 2026 18:05 WIB

- **Komponen yang Berubah**:
  - **Baru**
    - `lib/utils/image_type_utils.dart` [NEW]
    - `.vscode/launch.json` [NEW]
  - **Konfigurasi environment & API**
    - `lib/config/api_config.dart`, `lib/config/dio_provider.dart`, `lib/main.dart`
  - **Build Android & dependency**
    - `android/app/build.gradle.kts`, `android/settings.gradle.kts`, `android/gradle/wrapper/gradle-wrapper.properties`
    - `pubspec.yaml`, `pubspec.lock`
    - `linux/flutter/generated_plugin_registrant.cc`, `linux/flutter/generated_plugins.cmake`, `macos/Flutter/GeneratedPluginRegistrant.swift`, `windows/flutter/generated_plugins.cmake`
  - **Dokumentasi**
    - `README.md`
  - **Services**
    - `account_storage_service.dart`, `auth_event_service.dart`, `auth_service.dart`, `invoice_api_service.dart`, `payment_service.dart`, `pdf_service.dart`, `service_service.dart`, `transaction_service.dart`, `version_service.dart`
  - **Model, provider & util**
    - `models/auth_model.dart`, `models/payment_model.dart`, `models/service_model.dart`, `models/ticket_model.dart`
    - `providers/auth_provider.dart`, `providers/payment_provider.dart`, `providers/ticket_provider.dart`, `providers/user_provider.dart`
    - `utils/location_utils.dart`, `common/constant.dart`
  - **Screens — auth**
    - `comingsoon.dart`, `registration_flow.dart`, `screen_alamat.dart`, `screen_identitas.dart`, `screen_login.dart`, `screen_produk.dart`, `screen_register.dart`, `screen_verifikasi.dart`, `splash_screen.dart`
  - **Screens — home/profile/service/ticket/transaction**
    - `home/banner/banner_carousel_widget.dart`, `home/home_screen.dart`, `home/screen_home.dart`
    - `profile/change_password_screen.dart`, `profile/help_center_screen.dart`, `profile/privacy_policy_screen.dart`, `profile/screen_profile.dart`, `profile/terms_conditions_screen.dart`
    - `service/screen_allservice.dart`, `service/screen_service.dart`, `service/screen_service_history.dart`, `service/screen_subscription_detail.dart`
    - `ticket/screen_ticket.dart`, `ticket/screen_ticket_detail.dart`
    - `transaction/invoice_detail_screen.dart`, `transaction/payment_method_screen.dart`, `transaction/screen_alltransaction.dart`, `transaction/screen_transaction.dart`
    - `transaction/payment_success/payment_success_button.dart`, `payment_success_details.dart`, `payment_success_header.dart`, `payment_success_screen.dart`
    - `transaction/widgets/credit_card_external_browser_screen.dart`, `credit_card_webview_screen.dart`
  - **Widgets**
    - `bottom_nav_bar.dart`, `fullscreen_update_dialog.dart`, `location_picker_screen.dart`, `location_picker_widget.dart`, `step_progress_indicator.dart`, `update_dialog.dart`

  > Total: 73 file diubah — **+1.160 / -979 baris**.

- **Deskripsi Perubahan & Fungsi**:
  - **Pemilihan environment saat build (production/development)**: `ApiConfig` sekarang membaca `--dart-define=ENV=...` (default `production`). Base URL API dan URL download invoice otomatis menyesuaikan:
    - Production → `https://v2.ring.net.id/m-api/v1` & `https://srv.ring.net.id`
    - Development → `https://dev-srv.dekadata.net/m-api/v1` & `https://srv.dekadata.net`

    Sebelumnya URL dev/prod diganti manual dengan comment/uncomment kode, dan enum `Environment` + `setEnvironment()` yang ada tidak pernah benar-benar mengubah URL (semua cabang mengembalikan URL yang sama). Kode mati tersebut dihapus. Environment aktif dicetak ke log saat app start (`main.dart`). Ditambahkan juga `.vscode/launch.json` berisi dua konfigurasi run: **RingNet (Production)** dan **RingNet (Development)**.
  - **Header otorisasi dirapikan**: `authHeaders()` dan `downloadHeaders()` sebelumnya mengirim dua header sekaligus (`Authorization: Bearer <token>` dan `authorization: <token>`). Sekarang hanya `authorization: <token>` (token mentah), sesuai format yang diterima backend dan konsisten dengan interceptor di `dioProvider`.
  - **Signing Android lebih aman untuk mesin lain/CI**: `build.gradle.kts` hanya membuat signing config `release` jika `android/key.properties` ada. Jika tidak ada, release build jatuh ke debug signing dengan peringatan (bisa di-run, tapi **tidak bisa di-upload ke Play Store**). Sebelumnya project gagal sync/build total tanpa file keystore.
  - **Upgrade toolchain Android**: Android Gradle Plugin `8.7.0 → 8.13.0`, Gradle wrapper `8.10.2 → 8.13`.
  - **Upgrade dependency**: `geolocator ^9 → ^14`, `geocoding ^2 → ^5`, `share_plus ^7 → ^12`, `package_info_plus ^8 → ^9`. Package `open_file` dihapus (sudah tidak dipakai). Kode yang terdampak breaking change ikut disesuaikan:
    - Geolocator: `desiredAccuracy`/`timeLimit` → `LocationSettings(...)` (`location_utils.dart`, `location_picker_screen.dart`).
    - Geocoding: fungsi top-level `locationFromAddress`/`placemarkFromCoordinates` → instance `Geocoding()` (`location_utils.dart`, `location_picker_screen.dart`, `location_picker_widget.dart`).
    - Share Plus: `Share.shareXFiles(...)` → `SharePlus.instance.share(ShareParams(...))` (`pdf_service.dart`).
    - QR code: `foregroundColor` → `eyeStyle` + `dataModuleStyle` (`payment_success_details.dart`).
  - **Util baru `image_type_utils.dart`**: fungsi `isImageSvg()` yang sebelumnya diduplikasi di `screen_home.dart` dan `screen_profile.dart` digabung menjadi satu. Versi baru memakai Dio khusus dengan timeout 5 detik dan **cache hasil per URL**, sehingga `FutureBuilder` tidak lagi mengirim HEAD request baru setiap kali widget rebuild (sebelumnya avatar profil bisa memicu request berulang).
  - **`version_service.dart`**: Dio untuk cek versi sekarang memakai timeout dan default header dari `ApiConfig` (sebelumnya `Dio()` polos tanpa timeout — berpotensi menggantung lama di splash screen saat jaringan buruk).
  - **Migrasi API Flutter yang deprecated** (tanpa perubahan perilaku yang disengaja):
    - `WillPopScope` → `PopScope` di semua layar (registrasi, splash, profile, payment success, webview/external browser kartu kredit, dialog update).
    - `Color.withOpacity()` → `Color.withValues(alpha: ...)` di seluruh UI.
    - `Radio` dengan `groupValue`/`onChanged` per item → `RadioGroup` (filter & sort di `screen_produk.dart`).
    - `DropdownButtonFormField.value` → `initialValue` (`screen_identitas.dart`).
  - **Perbaikan keamanan `BuildContext` setelah `await`**: ditambahkan pengecekan `mounted`/`context.mounted` sebelum memakai `context` setelah operasi async (`screen_profile.dart`, `screen_service.dart`, `help_center_screen.dart`, `payment_method_screen.dart`, `splash_screen.dart`) — mencegah crash/warning "widget has been unmounted" jika user keluar dari layar saat proses masih berjalan.
  - **Lint cleanup**: konstanta `SCREAMING_CASE` di `account_storage_service.dart` diganti lowerCamelCase (nilai key penyimpanan **tidak berubah**, jadi data akun yang sudah tersimpan tetap terbaca), `super.key`/super parameters, `final` untuk field yang tidak di-reassign, hapus `.toList()` yang tidak perlu di spread, hapus import tak terpakai, hapus method `@deprecated generateAndOpenPdf`, dan format ulang kode (`dart format`).
  - **README** ditulis ulang: cara setup, pemilihan environment, kebutuhan `key.properties` untuk build rilis, perintah build rilis dengan `--obfuscate --split-debug-info`, dan perintah pengecekan (`flutter analyze`, `flutter test`).

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul                         | Dampak Utama                                                                                                                                                                                             |
| ----- | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| #5    | [ Update ] Refactor aplikasi  | Ganti server dev/prod cukup lewat flag build; dependency & toolchain Android ter-update; API deprecated Flutter dibersihkan; project bisa di-build tanpa keystore produksi; request HEAD avatar tidak lagi berulang |

### Kemampuan Baru Pengguna/Admin

- Tidak ada perubahan yang terlihat langsung oleh pengguna akhir — pekerjaan hari ini bersifat **refactor dan maintenance**.
- Untuk developer: bisa menjalankan aplikasi ke server **development** tanpa mengubah kode, cukup `flutter run --dart-define=ENV=development` atau pilih konfigurasi **RingNet (Development)** di VS Code.

### Bug Fix / Solusi Masalah

- Konfigurasi environment lama tidak berfungsi (semua environment mengarah ke URL production) — sekarang benar-benar memilih server sesuai flag.
- Project gagal sync/build di mesin yang tidak punya `android/key.properties` — sekarang tetap bisa build dengan debug signing.
- Cek tipe gambar SVG avatar mengirim HEAD request baru setiap rebuild — sekarang di-cache per URL.
- Cek versi aplikasi tidak punya timeout — sekarang mengikuti timeout standar `ApiConfig`.
- Potensi crash saat `context` dipakai setelah layar ditutup (async gap) — ditambahkan guard `mounted`.

### Menu/Fitur Baru

- Tidak ada menu/fitur baru hari ini.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Fitur utama hari ini adalah **pemilihan environment saat build**. Nilai `ENV` dibaca sebagai konstanta compile-time (`String.fromEnvironment`), jadi build production tidak membawa logika runtime untuk pindah server. Jika flag tidak diberikan, aplikasi **selalu** memakai server production — aman untuk build rilis.
- **Langkah Penggunaan (Tutorial — untuk developer/QA)**:
  1. Jalankan `flutter pub get` (dependency berubah hari ini).
  2. Mode development: `flutter run --dart-define=ENV=development`, atau di VS Code buka panel **Run & Debug** → pilih **RingNet (Development)**.
  3. Cek log saat app start — harus muncul `🌐 Environment: development → https://dev-srv.dekadata.net/m-api/v1`.
  4. Mode production: `flutter run` biasa (atau **RingNet (Production)**) — log harus menunjukkan `production → https://v2.ring.net.id/m-api/v1`.
  5. Build rilis Android: siapkan `android/key.properties`, lalu `flutter build appbundle --release --obfuscate --split-debug-info=build/symbols`. Simpan folder `build/symbols` untuk membaca stack trace rilis.
  6. Regression test yang disarankan karena ada upgrade dependency: pilih lokasi di peta saat registrasi (geolocator/geocoding), share PDF invoice (share_plus), tampilan QR di halaman sukses pembayaran, tombol back di halaman pembayaran kartu kredit & payment success, serta login/ganti akun (pastikan akun tersimpan tetap terbaca).
