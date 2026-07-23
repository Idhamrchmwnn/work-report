# 📝 Daily Work Report - idham (2026-07-23)

---

## 📅 Laporan Harian - 23 Juli 2026

> **Catatan Metode:** Pada tanggal 23 Juli 2026 tidak ada *commit* baru yang tercatat di git. Seluruh pekerjaan hari ini masih berupa perubahan **belum di-commit** (_uncommitted / work in progress_) pada branch aktif `Feature/service`. Laporan ini disusun dari hasil analisis `git status` dan `git diff --stat` terhadap working tree (total **107 file** berubah: 45 file kode dengan 812 penambahan & 769 penghapusan baris, ditambah aset & ikon).

---

## 🌿 Branch: `Feature/service` — Layanan (Service), Ticketing, Versioning & Refactor Arsitektur

### 📌 Informasi Issue

- **Nomor Issue**: (belum ditautkan ke issue tracker)
- **Judul Issue**: Implementasi halaman Service, sistem Ticket, mekanisme Force-Update, serta refactor konfigurasi API & penyimpanan akun
- **Status Branch**: `Belum di-merge` (perubahan bahkan belum di-commit)

### 📅 Rincian Pekerjaan (Uncommitted)

#### 1. Sentralisasi Konfigurasi API

- **Komponen yang Berubah**:
  - `lib/config/api_config.dart` [NEW]
  - `lib/config/dio_provider.dart` [NEW]
- **Deskripsi Perubahan & Fungsi**:
  - Membuat kelas `ApiConfig` sebagai satu sumber kebenaran untuk seluruh endpoint (base URL `https://v2.ring.net.id/m-api/v1`, file server, invoice HTML/download). Menghapus URL yang tersebar (_hardcoded_) di banyak service.
  - `dio_provider.dart` menyediakan instance `Dio` terpusat untuk seluruh service.

#### 2. Fitur Ticket (Layanan Bantuan Pelanggan)

- **Komponen yang Berubah**:
  - `lib/models/ticket_model.dart` [NEW]
  - `lib/providers/ticket_provider.dart` [NEW]
  - `lib/services/ticket_service.dart` [NEW]
  - `lib/screens/ticket/screen_ticket_detail.dart` [NEW]
- **Deskripsi Perubahan & Fungsi**:
  - `TicketService` mengambil daftar tiket pelanggan dari endpoint `customerTicket` dengan header autentikasi, terintegrasi dengan `AuthEventService` untuk penanganan error terpusat.
  - Halaman detail tiket merender isi tiket berformat HTML (`flutter_html`) sehingga balasan/percakapan support tampil rapi.

#### 3. Mekanisme Cek Versi & Force-Update

- **Komponen yang Berubah**:
  - `lib/models/version_model.dart` [NEW]
  - `lib/providers/version_provider.dart` [NEW]
  - `lib/services/version_service.dart` [NEW]
  - `lib/services/version_preferences.dart` [NEW]
  - `lib/widgets/fullscreen_update_dialog.dart` [NEW]
- **Deskripsi Perubahan & Fungsi**:
  - `VersionService` membandingkan versi aplikasi terpasang (`package_info_plus`) dengan versi terbaru dari API, membawa `changeLog` dan flag `forceUpdate`.
  - `FullScreenUpdateDialog` (responsif) menampilkan pembaruan; jika `forceUpdate` aktif, dialog tidak dapat ditutup. Opsi "Ingatkan Nanti" / "Jangan Tampilkan Lagi" disimpan lewat `VersionPreferences` (SharedPreferences).

#### 4. Keamanan: Penyimpanan Akun Multi-User & Token Terenkripsi

- **Komponen yang Berubah**:
  - `lib/services/account_storage_service.dart` [NEW]
  - `lib/services/auth_event_service.dart` [NEW]
  - `lib/providers/auth_provider.dart`, `lib/models/auth_model.dart`, `lib/models/user_model.dart`, `lib/providers/user_provider.dart`, `lib/services/user_service.dart`
- **Deskripsi Perubahan & Fungsi**:
  - `AccountStorageService` mendukung penyimpanan banyak akun. Token autentikasi kini **hanya** disimpan di secure storage (Keychain iOS / EncryptedSharedPreferences+Keystore Android). Metadata non-sensitif tetap di SharedPreferences tanpa token, plus migrasi otomatis token plaintext lama ke secure storage.
  - `AuthEventService` menyediakan _event bus_ global: error `sessionExpired`/`unauthorized` memicu logout, sedangkan error umum hanya menampilkan dialog.

#### 5. Banner Carousel (Beranda)

- **Komponen yang Berubah**:
  - `lib/screens/home/banner/banner_model.dart` [NEW]
  - `lib/screens/home/banner/banner_provider.dart` [NEW]
  - `lib/screens/home/banner/banner_carousel_widget.dart` [NEW]
  - `lib/screens/home/screen_home.dart`
- **Deskripsi Perubahan & Fungsi**:
  - `BannerNotifier` (Riverpod) mengambil daftar banner dari API via `ApiConfig` dan menampilkannya sebagai carousel di beranda.

#### 6. Invoice & Ekspor PDF

- **Komponen yang Berubah**:
  - `lib/services/invoice_api_service.dart` [NEW]
  - `lib/services/pdf_service.dart` [NEW]
  - `lib/screens/transaction/invoice_detail_screen.dart`, `lib/screens/transaction/screen_alltransaction.dart` [NEW]
- **Deskripsi Perubahan & Fungsi**:
  - `InvoiceApiService` menyatukan operasi invoice (preview, download, manajemen cache) dengan penanganan izin penyimpanan.
  - `PdfService` menghasilkan dokumen PDF dari data transaksi/pembayaran/user dan mendukung *share* (`share_plus`).

#### 7. Ubah Kata Sandi & Refactor Autentikasi

- **Komponen yang Berubah**:
  - `lib/screens/profile/change_password_screen.dart` [NEW]
  - `lib/screens/auth/screen_login.dart`, `lib/screens/auth/splash_screen.dart`, `lib/screens/profile/screen_profile.dart`, `lib/screens/profile/help_center_screen.dart`
- **Deskripsi Perubahan & Fungsi**:
  - Menambah halaman ganti kata sandi dari menu profil serta penyesuaian alur login/splash mengikuti mekanisme penyimpanan akun & error handling baru.

#### 8. Refactor Layanan Transaksi & Pembayaran

- **Komponen yang Berubah**:
  - `lib/services/transaction_service.dart`, `lib/services/payment_service.dart`, `lib/providers/transaction_provider.dart`, `lib/providers/payment_provider.dart`, `lib/models/transaction_model.dart`, `lib/models/payment_model.dart`
  - `lib/screens/transaction/payment_method_screen.dart`, `.../credit_card_webview_screen.dart`, `.../credit_card_external_browser_screen.dart`, `.../payment_success/*`
- **Deskripsi Perubahan & Fungsi**:
  - Menyelaraskan seluruh service transaksi/pembayaran agar memakai `ApiConfig` dan `Dio` terpusat, memperbaiki alur pembayaran kartu kredit (WebView / browser eksternal) dan halaman sukses.

#### 9. Rebranding Aset & Ikon Aplikasi (GoRingNet)

- **Komponen yang Berubah**:
  - `assets/icon/goringnet_logo.png` [NEW], `assets/fonts/Roboto-Regular.ttf` [NEW], `assets/fonts/Roboto-Bold.ttf` [NEW]
  - Seluruh ikon aplikasi iOS (`Icon-App-*`) & Android (`ic_launcher.png` semua densitas)
  - `android/app/build.gradle.kts`, `android/settings.gradle.kts`, `android/app/src/main/AndroidManifest.xml`, `MainActivity.kt`
- **Deskripsi Perubahan & Fungsi**:
  - Mengganti ikon peluncur aplikasi iOS & Android ke logo GoRingNet, menambahkan font Roboto, serta menyesuaikan konfigurasi build Android.

---

## 📢 Ringkasan Dampak Perubahan & Fungsionalitas Baru

| Area | Judul | Dampak Utama |
| ---- | ----- | ------------ |
| Config | Sentralisasi API | Endpoint terpusat, mudah pindah environment |
| Ticket | Layanan bantuan | Pelanggan bisa melihat & membaca detail tiket |
| Versioning | Force-update | Paksa pengguna memakai versi terbaru |
| Keamanan | Token terenkripsi | Token pindah ke secure storage + multi-akun |
| Beranda | Banner carousel | Promo/banner dinamis di beranda |
| Invoice | Ekspor PDF | Preview, unduh, dan bagikan invoice PDF |
| Branding | Ikon GoRingNet | Identitas visual aplikasi diperbarui |

### Kemampuan Baru Pengguna/Admin

- Pengguna dapat menyimpan **beberapa akun** dan berpindah antar-akun; token tersimpan aman di Keychain/Keystore.
- Pengguna dapat melihat daftar & detail **tiket support** langsung dari aplikasi.
- Pengguna dapat **mengganti kata sandi** dari menu profil.
- Pengguna dapat **mengunduh dan membagikan invoice** dalam bentuk PDF.
- Aplikasi otomatis **memeriksa versi** dan menampilkan dialog pembaruan (opsional/wajib).

### Bug Fix / Solusi Masalah

- Token yang sebelumnya tersimpan plaintext di SharedPreferences kini **dimigrasikan otomatis** ke penyimpanan terenkripsi.
- Error sesi/otorisasi ditangani terpusat lewat `AuthEventService` (logout otomatis saat sesi kedaluwarsa) sehingga alur error lebih konsisten.

### Menu/Fitur Baru

- Menu **Tiket** (detail tiket support).
- Menu **Ganti Kata Sandi** pada Profil.
- **Banner carousel** pada beranda.
- Dialog **Update Aplikasi** (full screen).

---

## 📖 Informasi & Tutorial Singkat Fitur Utama

- **Penjelasan Fitur (Force-Update):** Saat aplikasi dibuka, `VersionService` membaca versi terpasang via `package_info_plus` lalu membandingkannya dengan data versi dari API. Bila versi server lebih baru, `FullScreenUpdateDialog` muncul menampilkan `changeLog`. Jika flag `forceUpdate` `true`, dialog tidak bisa ditutup hingga pengguna memperbarui aplikasi.
- **Langkah Penggunaan (Tutorial):**
  1. Buka aplikasi seperti biasa.
  2. Jika tersedia versi baru, dialog pembaruan akan tampil otomatis.
  3. Untuk pembaruan opsional, pilih **"Ingatkan Nanti"** atau **"Jangan Tampilkan Lagi"** (preferensi disimpan per versi).
  4. Untuk pembaruan wajib, tekan tombol perbarui untuk membuka halaman toko aplikasi.

---

> ⚠️ **Rekomendasi Tindak Lanjut:** Seluruh pekerjaan di atas belum di-commit. Disarankan segera melakukan `git commit` (dan mengaitkannya dengan nomor issue terkait) agar riwayat pekerjaan tanggal 23 Juli 2026 tercatat resmi di git.
