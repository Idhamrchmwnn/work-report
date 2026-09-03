# 📝 Daily Work Report - Idham (2026-09-03)

---

## 📅 Laporan Harian - 3 September 2026

---

## 🌿 Branch: `issue-1` — [ Update ] Security Refactor

### 📌 Informasi Issue

- **Nomor Issue**: #1
- **Judul Issue**: [ Update ] Security Refactor
- **Status Branch**: `Sudah di-merge` (PR #2, merged 15:47 WIB)

### 📅 Rincian Commit

#### [7495961] - resolve #1 - Kamis, 3 September 2026 15:24 WIB

- **Komponen yang Berubah**:
  - **Baru**
    - `lib/common/app_logger.dart` [NEW]
  - **Dihapus (dead code)**
    - `lib/screens/service/new.dart`
    - `lib/screens/service/new2.dart`
    - `lib/widgets/backup.dart`
  - **Keamanan & penyimpanan akun**
    - `lib/services/account_storage_service.dart`
    - `lib/services/auth_service.dart`
    - `lib/services/auth_event_service.dart`
    - `lib/providers/auth_provider.dart`
    - `pubspec.yaml`, `pubspec.lock`
    - `android/app/build.gradle.kts`
  - **Providers lain (bersih-bersih logging & minor fix)**
    - `lib/providers/payment_provider.dart`, `service_provider.dart`, `transaction_provider.dart`, `user_provider.dart`, `version_provider.dart`
  - **Screens — auth**
    - `comingsoon.dart`, `registration_flow.dart`, `screen_alamat.dart`, `screen_identitas.dart`, `screen_login.dart`, `screen_produk.dart`, `screen_verifikasi.dart`, `splash_screen.dart`
  - **Screens — home/profile/service/ticket/transaction**
    - `home/banner/banner_carousel_widget.dart`, `home/banner/banner_provider.dart`, `home/screen_home.dart`
    - `profile/change_password_screen.dart`, `profile/help_center_screen.dart`, `profile/privacy_policy_screen.dart`, `profile/screen_profile.dart`
    - `service/screen_service.dart`
    - `ticket/screen_ticket.dart`, `ticket/screen_ticket_detail.dart`
    - `transaction/invoice_detail_screen.dart`, `transaction/payment_method_screen.dart`, `transaction/screen_alltransaction.dart`, `transaction/screen_transaction.dart`
    - `transaction/payment_success/payment_success_button.dart`, `payment_success_details.dart`, `payment_success_screen.dart`
    - `transaction/widgets/credit_card_external_browser_screen.dart`, `credit_card_webview_screen.dart`
  - **Services lain**
    - `invoice_api_service.dart`, `payment_service.dart`, `pdf_service.dart`, `service_service.dart`, `ticket_service.dart`, `transaction_service.dart`, `user_service.dart`, `version_preferences.dart`, `version_service.dart`
  - **Model & util**
    - `models/auth_model.dart`, `models/payment_model.dart`, `models/ticket_model.dart`, `models/transaction_model.dart`, `utils/location_utils.dart`, `widgets/fullscreen_update_dialog.dart`, `widgets/location_picker_screen.dart`
  - **Config/build lain**
    - `.gitignore`, `lib/main.dart`, `lib/config/dio_provider.dart`, `test/widget_test.dart`
    - `linux/flutter/generated_plugin_registrant.cc`, `linux/flutter/generated_plugins.cmake`, `macos/Flutter/GeneratedPluginRegistrant.swift`, `windows/flutter/generated_plugin_registrant.cc`, `windows/flutter/generated_plugins.cmake`

  > Total: 68 file diubah — **+1.092 / -5.590 baris**.

- **Deskripsi Perubahan & Fungsi**:
  - Menambahkan dependency `flutter_secure_storage: ^9.2.2` dan menaikkan `minSdk` Android menjadi minimal 23 (dibutuhkan oleh `EncryptedSharedPreferences`/`androidx.security-crypto`). Token akun yang sebelumnya tersimpan di `SharedPreferences` (plaintext) kini dimigrasikan otomatis ke secure storage saat aplikasi dibuka pertama kali setelah update (`account_storage_service.dart`, dikontrol flag migrasi agar hanya berjalan sekali).
  - Menambahkan `lib/common/app_logger.dart`: wrapper logging terpusat (`logDebug`, `logWarning`, `logError`, `reportFatalError`) berbasis package `logger`. Semua log otomatis **nonaktif di release build** (`kReleaseMode`) — sebelumnya `print()` dipakai di puluhan tempat dan tetap aktif di production, berisiko membanjiri logcat/Console dan membocorkan data. Seluruh pemanggilan `print()` di services, providers, dan screens diganti ke `logDebug`/`logError`.
  - Menghapus tiga file yang sudah tidak dipakai lagi: `screens/service/new.dart` (1.300 baris), `screens/service/new2.dart` (2.081 baris), dan `widgets/backup.dart` (1.037 baris) — total ±4.400 baris kode usang dibersihkan dari repo.
  - Menaikkan versi aplikasi dari `1.1.9+19` ke `1.2.0+20` di `pubspec.yaml`.
  - Perubahan lain di seluruh screens/providers bersifat pendukung refactor ini (penyesuaian pemanggilan service yang API-nya berubah, penghapusan `print()`, rapikan import).

---

## 🌿 Branch: `issue-3` — [Fix Bug] Gagal Logout

### 📌 Informasi Issue

- **Nomor Issue**: #3
- **Judul Issue**: [Fix Bug] Gagal Logout
- **Status Branch**: `Sudah di-merge` (PR #4, merged 16:07 WIB)

### 📅 Rincian Commit

#### [ad5387a] - resolve #3 - Kamis, 3 September 2026 16:05 WIB

- **Komponen yang Berubah**:
  - `lib/services/auth_service.dart`

- **Deskripsi Perubahan & Fungsi**:
  - Root cause: `logout()` memanggil `_getActiveToken()` (satu kali baca secure storage) lalu langsung memanggil `loadSavedAccounts()` lagi (baca secure storage kedua, berurutan per akun tersimpan). Pada beberapa perangkat Android, dua kali baca `EncryptedSharedPreferences` berurutan ini membuat UI macet saat logout — pola bug yang sama dengan kasus "macet saat tambah akun" yang pernah terjadi sebelumnya.
  - Perbaikan: `logout()` dan `logoutAll()` sekarang hanya memanggil `loadSavedAccounts()` **satu kali**, lalu token aktif diambil dari `accountsList.activeAccount` yang sudah dimuat — tidak ada lagi baca ganda ke secure storage.
  - `_tryServerLogout()` dibungkus dalam closure `async` dengan `try/catch` di dalamnya (bukan di luar `_dio.post(...)` yang tidak di-`await`), sehingga error dari request logout ke server tertangkap dengan benar dan tidak lolos menjadi *unhandled Future error*.

---

## 🌿 Branch: `main` — Hotfix langsung (tanpa issue)

### 📌 Informasi Issue

- **Nomor Issue**: – (tidak ada issue/PR, commit langsung ke `main`)
- **Judul Issue**: Perbaikan teks halaman "Coming Soon"
- **Status Branch**: `Sudah di-merge` (langsung di `main`)

### 📅 Rincian Commit

#### [f179e03] - urgent fix - Kamis, 3 September 2026 16:25 WIB

- **Komponen yang Berubah**:
  - `lib/screens/auth/comingsoon.dart`

- **Deskripsi Perubahan & Fungsi**:
  - String `operationalHours` salah menggunakan escape karakter `\S` alih-alih `\n`, membuat jam operasional Sabtu menyatu dalam satu baris dengan jam Senin–Jumat di halaman "Coming Soon". Diperbaiki menjadi line break yang benar.
  - Nomor WhatsApp admin di kartu kontak sebelumnya di-hardcode (`+62 8783-9988-767`) dan tidak sinkron dengan konstanta `adminWhatsApp` yang sudah ada di file yang sama. Diganti menjadi interpolasi `'+$adminWhatsApp'` agar satu sumber data.
  - ⚠️ Catatan: commit ini masuk langsung ke `main` tanpa branch issue/PR — perlu dicek apakah ini sesuai kebijakan review tim.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Issue | Judul                          | Dampak Utama                                                                                                   |
| ----- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| #1    | Security Refactor               | Token akun disimpan terenkripsi (secure storage, bukan plaintext); log debug otomatis mati di release build; ±4.400 baris kode usang dibersihkan; versi app naik ke 1.2.0+20 |
| #3    | Fix Bug: Gagal Logout           | Logout tidak lagi berpotensi macet di perangkat Android akibat baca ganda secure storage                          |
| –     | Hotfix teks "Coming Soon"       | Jam operasional Sabtu & nomor WhatsApp admin tampil benar di halaman pendaftaran                                  |

### Kemampuan Baru Pengguna/Admin

- Tidak ada kemampuan baru yang terlihat langsung oleh pengguna — seluruh pekerjaan hari ini bersifat **hardening keamanan dan perbaikan stabilitas** di balik layar.

### Bug Fix / Solusi Masalah

- **Gagal logout (issue #3)**: UI tidak lagi macet saat pengguna melakukan logout, terutama di perangkat Android yang sebelumnya terdampak pola baca-ganda secure storage.
- **Teks salah tampil**: jam operasional Sabtu dan nomor WhatsApp admin di halaman "Coming Soon" kini akurat.

### Menu/Fitur Baru

- Tidak ada menu/fitur baru hari ini.

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur**: Perubahan inti hari ini adalah migrasi penyimpanan token akun dari `SharedPreferences` (plaintext) ke `flutter_secure_storage` (terenkripsi via Android Keystore/iOS Keychain). Migrasi berjalan **otomatis dan sekali saja** saat pengguna membuka aplikasi versi baru — tidak perlu login ulang secara manual, dan pengguna tidak akan melihat perubahan apa pun di UI.
- **Langkah Penggunaan (Tutorial — untuk QA/verifikasi)**:
  1. Install versi lama, login dengan 1–2 akun agar token tersimpan di `SharedPreferences`.
  2. Update ke build hari ini (1.2.0+20), buka aplikasi — pastikan sesi tetap aktif tanpa perlu login ulang (tanda migrasi berhasil).
  3. Lakukan logout dari salah satu akun berulang kali (termasuk cepat berturut-turut) — pastikan UI tidak freeze dan proses selesai normal (verifikasi fix issue #3).
  4. Buka halaman "Coming Soon" (registrasi), cek jam operasional Sabtu tampil di baris terpisah dan nomor WhatsApp admin sesuai konstanta `adminWhatsApp`.
