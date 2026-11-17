# 📝 Daily Work Report - Idham (2026-11-17)

---

## 📅 Laporan Harian - 17 November 2026

---

## 🌿 Branch: `issue-556` — Fondasi Mobile Frontend (lanjutan & penyelesaian)

### 📌 Informasi Issue

- **Nomor Issue**: #556
- **Judul Issue**: Mobile Frontend: fondasi klien Partner API, sesi login, PWA, dan cakupan fitur di PRD
- **Status Branch**: `Sudah di-merge`. Commit `resolve #556` (`996e966c`) di-merge ke `master` pada 17 November, 17:46.

### 📅 Rincian Commit

#### [996e966c] - resolve #556 - 17 November 2026, 11:23

- **Ringkasan**:
  - Pekerjaan kemarin (klien Partner API, sesi tanpa refresh token, PWA, infrastruktur daftar, tooling, PRD Bab 5–6, AGENTS §2.M) di-commit dan di-squash menjadi satu commit. Rinciannya ada di laporan 16 November.
  - Tambahan hari ini sebelum commit: stack disamakan dengan arahan PRD.
    - **React Context diganti Redux Toolkit.**
    - **Semua teks lewat i18n** (`id` dan `en`).
    - **Halaman dimuat per rute.**
  - Total 93 berkas, +6.819 / −746 baris (termasuk `package-lock.json`).

- **Komponen yang Berubah & Penjelasannya (tambahan hari ini)**:

  **State global (Redux Toolkit)**

  - `mobile-frontend/src/context/{AuthContext,ConfigContext,ThemeContext}.jsx` [DIHAPUS]
    - Diganti slice Redux berikut.
  - `mobile-frontend/src/features/authSlice.js` [NEW]
    - Menyimpan profil hasil `normalizeUser` dan status UI login.
    - Token **tidak** disimpan di Redux; token hanya ada di `utils/authStorage.js`.
    - Galat disimpan sebagai `{ key }` (diterjemahkan saat tampil) atau `{ message }` (pesan server apa adanya).
  - `mobile-frontend/src/features/{configSlice,themeSlice}.js` [NEW]
    - Info aplikasi (nama & logo) dan tema.
  - `mobile-frontend/src/store.js`, `hooks/useAuth.js`, `hooks/useConfig.js`, `hooks/useTheme.js` [NEW]
    - Halaman membaca state lewat hook, tidak langsung ke store.
    - `useAuth` menyusun teks profil terjemahan (`presentUser`).
  - `mobile-frontend/src/components/AppBootstrap.jsx` [NEW]
    - Efek awal yang dulu ada di provider Context: memulihkan sesi, memuat info aplikasi, menerapkan tema.
    - Dijalankan sekali per muat halaman, karena penanda sesi berakhir hanya boleh dipakai sekali.

  **i18n**

  - `mobile-frontend/src/i18n/config.js` [NEW]
    - Bahasa bawaan `id`, pilihan pengguna disimpan di `i18nextLng` (kunci yang sama dibaca `offline.html`).
    - **Sengaja tanpa deteksi bahasa browser**, supaya browser berbahasa Inggris tidak mengubah bahasa diam-diam.
  - `mobile-frontend/src/i18n/locales/{id,en}/translations.json` [NEW], 256 baris masing-masing.
  - `mobile-frontend/src/configs/menu.js` [NEW]
    - Konfigurasi menu samping dipindah dari `data/mockData.js` (konfigurasi, bukan data contoh), labelnya diganti key `nav.*`.
  - `mobile-frontend/src/pages/*.jsx` (Home, Login, Customers, Filters, Finance, Network, Organization, Products, Profile, Radius, Statistic)
    - Teks Indonesia langsung diganti `t()`, dan state dibaca lewat hook Redux.

  **Pemuatan halaman**

  - `mobile-frontend/src/utils/lazyPage.js` dan `components/ChunkLoadFailed.jsx` [NEW]
    - `React.lazy` yang tidak pernah melempar.
    - Bila berkas halaman gagal dimuat (jaringan putus, atau versi lama sudah dihapus setelah deploy), halaman diganti layar yang menawarkan muat ulang.
    - Tanpa error boundary berbentuk class (sesuai aturan §2.B.1).
  - `mobile-frontend/src/App.jsx`
    - Semua halaman dimuat per rute lewat `lazyPage`.

  **Dokumentasi**

  - `AGENTS.md` §2.M dan `mobile-frontend/README.md`, `PRD.md`
    - Stack diperbarui: Redux Toolkit + i18next.
    - Aturan baru: teks wajib lewat `t()` dengan key di `id` **dan** `en`, dijaga test cakupan terjemahan.

  **Test** (tambahan hari ini, 23 skenario; total test `mobile-frontend` saat commit: 76)

  - `src/features/authSlice.test.js` (13): login, pemulihan sesi, sesi berakhir, logout, bentuk galat.
  - `src/features/themeSlice.test.js` (5), `configSlice.test.js` (2).
  - `src/i18n/translations.coverage.test.js` (3)
    - Key yang dipakai kode harus ada di `id` dan `en`, termasuk properti `xxxKey: 'a.b'`.
    - Struktur `id` dan `en` harus sama.
    - `t()` tidak boleh diberi teks default.

---

## 🌿 Branch: `issue-559` — Mobile Frontend Modul B: Pelanggan

### 📌 Informasi Issue

- **Nomor Issue**: #559
- **Judul Issue**: Mobile Frontend: modul Pelanggan (Reguler, Mitra, Bisnis) terhubung ke Partner API
- **Status Branch**: `Belum di-merge`. Empat commit `save #559`, sudah ada di `origin/issue-559` (di atas `resolve #556`). Salinan hasil rebase keempat commit ini juga menjadi dasar branch `issue-561`.

### 📅 Rincian Commit

#### [f3ba4581] - save #559 - 17 November 2026, 13:09

- **Ringkasan**:
  - Modul Pelanggan dihubungkan ke Partner API untuk tiga jenis: **Reguler** (hanya admin), **Mitra/POP**, dan **Bisnis**.
  - Daftar, detail bertab, form tambah/ubah, ubah status, dan hapus (hanya dari detail, dengan konfirmasi).
  - Data contoh pelanggan dihapus dari `data/mockData.js`.
  - Ketergantungan baru: `react-hook-form`, `@hookform/resolvers`, `yup`.

- **Komponen yang Berubah & Penjelasannya**:

  **Logika murni (`features/customers/`)**

  - `customerKinds.js` [NEW]
    - Satu tempat untuk perbedaan tiap jenis: endpoint, kunci id (`customer_id` vs `partner_id`), peran yang boleh, dan kemampuan (mis. dokumen). Halaman tidak perlu menyebar `if (kind === ...)`.
    - `requiresOwnerPicker`: admin yang menambah pelanggan mitra **atau** bisnis wajib memilih mitra pemilik. Tanpa itu backend menempelkan record ke mitra portal pertama yang ditemukan, jadi pemiliknya salah.
  - `customerPayload.js` [NEW]
    - Konversi record ↔ nilai form.
    - Telepon ditampilkan dengan prefiks `+62` tetap, dan dikirim ke server sebagai `62812…`.
    - **KTP/NPWP yang tersamar dari server tidak pernah dikirim balik**, karena akan menimpa nomor asli.
    - Payload ubah hanya berisi field yang berubah; field yang dikosongkan dikirim sebagai `remove-data`.
    - Galat field dari server dipetakan ke kolom form.
  - `customerSchema.js` [NEW]
    - Skema Yup: KTP wajib untuk pelanggan, NPWP wajib untuk bisnis, koordinat dalam batas, dan pemilik wajib saat admin menambah.
  - `customerStatus.js` [NEW]
    - Pelanggan yang berhenti berlangganan dan blacklist hanya bisa dibaca; pasif tetap bisa diubah.

  **Service & endpoint**

  - `services/customerApi.js` [NEW]
    - Daftar, detail, tambah (multipart untuk pelanggan supaya dokumen ikut; JSON untuk bisnis), ubah, ubah status, hapus.
    - Dokumen: daftar, unduh sebagai Blob (endpoint butuh Bearer, jadi tidak bisa `<a href>`), unggah, hapus.
    - Akun PPPoE milik pelanggan untuk tab Layanan.
  - `services/endpoints.js`
    - Path `customers/*`, `customer-partners/*`, `business/*`, dan dokumen.
  - `utils/apiClient.js`
    - Penyesuaian untuk unggahan multipart.

  **Halaman**

  - `pages/CustomersPage.jsx`
    - Tab per jenis sesuai peran (mitra tidak melihat tab Reguler), pencarian, filter status, dan daftar kartu.
  - `pages/customers/CustomerDetailPage.jsx` [NEW]
    - Tab Info, Layanan (akun PPPoE), Dokumen (hanya untuk jenis yang punya dokumen), dan Tagihan.
    - Aksi ubah status dan hapus dengan konfirmasi.
  - `pages/customers/CustomerFormPage.jsx` [NEW]
    - Form tambah/ubah dengan React Hook Form + Yup. Pemilih mitra muncul untuk admin, dan dokumen bisa diambil dari kamera/galeri.
  - `pages/customers/components/CustomerDocumentsTab.jsx`, `CustomerServicesTab.jsx`, `InfoRow.jsx` [NEW]

  **Komponen umum**

  - `components/BottomSheet.jsx`, `ConfirmDialog.jsx`, `ListState.jsx` [NEW]
    - Panel bawah, pop up konfirmasi, dan keadaan memuat/kosong/galat daftar.
  - `components/PartnerPicker.jsx` [NEW]
    - Pemilih mitra dengan pencarian dan gulir tak terbatas (`/partners/list`).
  - `components/form/fields.jsx`, `CoordinateField.jsx`, `FileField.jsx` [NEW]
    - Input teks, pilihan, telepon, koordinat, dan berkas bergaya desain awal.
  - `configs/customerTypes.js` [NEW]
    - Pilihan jenis pelanggan.

  **Test**

  - `src/features/customers/customers.test.js` [NEW], 21 skenario untuk `customerKinds`, `customerStatus`, `customerPayload`, dan `customerSchema` (telepon, nilai tersamar, payload tambah/ubah, FormData, pemetaan galat, validasi per jenis).

#### [fbf2a774] - save #559 - 17 November 2026, 14:45

- **Ringkasan**:
  - **Pemilih lokasi di peta**, setara `LocationPickerInput` di web.
    - Cari nama tempat atau tempel koordinat.
    - Geser peta di bawah pin tengah, atau pakai GPS.
  - Ketergantungan baru: `leaflet`, `react-leaflet`.

- **Komponen yang Berubah & Penjelasannya**:

  - `components/map/MapPickerSheet.jsx` [NEW]
    - Layar penuh dengan kolom cari dan pin tengah.
    - Titik hasil cari/GPS/tempelan disimpan pasti, karena pusat peta setelah `flyTo` bergeser di desimal ke-6.
    - Esc menutup, dan gulir halaman di belakang dikunci selama layar terbuka.
  - `components/map/CenterPin.jsx`, `MapPreview.jsx`, `mapTiles.js` [NEW]
    - Pin, pratinjau peta kecil di form, dan tile Google lewat HTTPS.
  - `services/placeSearch.js` [NEW]
    - Pencarian tempat lewat OpenStreetMap Nominatim, tanpa token dan tanpa data pelanggan.
    - Pemanggil wajib debounce dan membatalkan permintaan lama (kebijakan Nominatim maks 1 permintaan/detik).
  - `utils/coordinate.js`, `geolocation.js`, `scheduleMapResize.js` [NEW]
    - Parsing koordinat, GPS, dan `invalidateSize` setelah animasi buka (pola Bab 9 untuk Leaflet di panel).
  - `components/form/CoordinateField.jsx`, `fields.jsx`, `fieldStyles.js`, `PartnerPicker.jsx`
    - Kolom koordinat membuka pemilih peta. Gaya input disatukan di `fieldStyles.js`.
  - `AGENTS.md` §2.M butir 2, `PRD.md`, `README.md`
    - Pengecualian kedua dari aturan "hanya Partner API": peta (tile Google + Nominatim), dimuat lazy, tanpa token/data pelanggan.
  - **Test**: `src/utils/coordinate.test.js` [NEW], 5 skenario.

#### [e6f981aa] - save #559 - 17 November 2026, 15:53

- **Ringkasan**:
  - **Alamat & area diisi otomatis** dari titik peta/GPS (reverse geocoding).
    - Field hanya diisi bila masih kosong atau masih berisi isian otomatis sebelumnya.
    - Isian pengguna, termasuk alamat lama saat mode ubah, tidak pernah ditimpa.
  - **Pengaturan Tema & Bahasa** di halaman Akun dan menu.
  - Detail pelanggan dirapikan.

- **Komponen yang Berubah & Penjelasannya**:

  - `features/customers/addressAutofill.js` [NEW]
    - `planAutofill` mengembalikan field yang diisi dan field yang dibiarkan.
  - `features/customers/useAddressAutofill.js` [NEW]
    - Hook yang memanggil reverse geocoding sekali per titik terpilih, bukan per gerakan peta.
  - `services/placeSearch.js`
    - `reverseGeocode` dan `toAddressFields`: urutan bagian alamat Indonesia, nilai ganda dibuang, `area` = kabupaten/kota.
  - `components/form/CoordinateField.jsx`
    - Memicu isi otomatis setelah titik dipilih.
  - `pages/customers/components/CustomerLocationCard.jsx` [NEW]
    - Kartu lokasi di detail pelanggan dengan pratinjau peta.
  - `pages/customers/CustomerDetailPage.jsx`, `CustomerFormPage.jsx`
    - Tata letak detail dirapikan, form memakai isi otomatis.
  - `components/SettingSelect.jsx`, `SegmentedControl.jsx` [NEW], `hooks/useLanguage.js` [NEW], `hooks/useTheme.js`
    - Dropdown ringkas untuk Tema dan Bahasa.
  - `components/MobileShell.jsx`, `pages/ProfilePage.jsx`, `pages/LoginPage.jsx`
    - Pilihan tema & bahasa di menu dan halaman Akun.
  - `i18n/locales/{id,en}/translations.json`
    - Key baru dan perapian teks.
  - **Test**:
    - `customers.test.js`: +3 skenario `addressAutofill` (field kosong diisi, isian pengguna tidak ditimpa, hasil kosong tidak menghapus).
    - `coordinate.test.js`: +1.

#### [19e43b37] - save #559 - 17 November 2026, 16:55

- **Ringkasan**:
  - **Daftar berhalaman seperti versi web**, menggantikan gulir tak terbatas di halaman daftar.
    - Nomor halaman dan pilihan jumlah per halaman 10/20/50/100 (bawaan 20).
    - Pencarian, filter, jumlah, dan halaman disimpan di URL.
  - **Cache daftar di memori**: kembali dari detail menampilkan halaman yang sama tanpa memuat ulang.
  - **Toast** (Sonner, gaya sama dengan web), **penjaga perubahan belum disimpan** di form, dan tautan telepon/WhatsApp di detail.
  - Ketergantungan baru: `sonner`.

- **Komponen yang Berubah & Penjelasannya**:

  - `hooks/usePagedList.js` [NEW]
    - Daftar datatable per halaman.
    - Pindah halaman menampilkan skeleton (bukan isi halaman lain), sedangkan muat ulang senyap mempertahankan isi.
    - Penyegaran gagal mempertahankan data lama.
  - `hooks/listCache.js` [NEW]
    - Cache per sesi tab (tidak ke storage).
    - Detail/form menyegarkan cache lewat `patchCachedListItem`, `removeCachedListItem`, dan `invalidateListCache`.
  - `components/Pagination.jsx`, `utils/pagination.js` [NEW]
    - Nomor halaman ringkas dengan elipsis.
  - `hooks/useInfiniteList.js`
    - Kini hanya dipakai panel pilihan (pemilih mitra), dengan dukungan cache.
  - `hooks/useUnsavedChangesGuard.js` [NEW]
    - Menahan navigasi di aplikasi (termasuk tombol/gestur kembali HP) dan muat ulang/tutup tab saat form punya perubahan.
  - `components/Toaster.jsx`, `utils/toast.js` [NEW], `index.css`
    - Toast warna penuh, tombol tutup di kanan atas, posisi di atas navigasi bawah.
  - `utils/phone.js` [NEW]
    - Format tampilan `+62 812-3456-7890`, serta tautan `tel:` dan WhatsApp.
  - `pages/CustomersPage.jsx`
    - Dropdown status + jumlah per halaman, dan paginasi.
  - `pages/customers/CustomerDetailPage.jsx`, `CustomerFormPage.jsx`, `components/CustomerDocumentsTab.jsx`, `InfoRow.jsx`
    - Cache daftar disegarkan setelah ubah/hapus, toast setelah aksi, dan penjaga form.
  - `App.jsx`, `MobileShell.jsx`, `ConfirmDialog.jsx`, `SettingSelect.jsx`
    - Toaster dipasang; `SegmentedControl` dihapus karena digantikan `SettingSelect`.
  - **Test**:
    - `src/hooks/listCache.test.js` (2) [NEW]
    - `src/utils/pagination.test.js` (3) [NEW]
    - `src/utils/phone.test.js` (2) [NEW]

---

## 🌿 Branch: `issue-561` — Mobile Frontend Modul C: Layanan RADIUS

### 📌 Informasi Issue

- **Nomor Issue**: #561
- **Judul Issue**: Mobile Frontend: modul Layanan RADIUS (akun PPPoE, sesi online, profil) terhubung ke Partner API
- **Status Branch**: `Belum di-merge`. Commit `save #561` sudah ada di `origin/issue-561`.
  - Branch ini dibangun di atas salinan hasil rebase commit #556 dan #559 (`bf1fde77`, `b84a9243`…`70e9f6c2`) di atas `resolve #558`.

### 📅 Rincian Commit

#### [a6a225c3] - save #561 - 17 November 2026, 18:13

- **Ringkasan**:
  - Modul Layanan RADIUS dihubungkan ke Partner API. Tiga tab di `RadiusPage`:
    - **Akun PPPoE**
    - **Sesi Online**
    - **Profil** (hanya admin)
  - **Akun PPPoE**:
    - daftar berhalaman dengan statistik total/aktif/nonaktif/online;
    - detail bertab Info, Pemakaian, Sesi, dan Log;
    - form tambah/ubah;
    - ubah status, hapus, dan putus sesi.
  - **Profil bandwidth**: daftar, detail dengan penghitung akun, serta form tambah/ubah.
  - Tab Layanan di detail pelanggan kini menautkan ke detail akun.
  - Data contoh RADIUS dihapus dari `data/mockData.js`.

- **Komponen yang Berubah & Penjelasannya**:

  **Logika murni (`features/radius/`)**

  - `accountPayload.js` [NEW]
    - Aturan yang sama dengan `partner-frontend/src/utils/radiusAccount.js`.
    - Kontrak backend yang **tidak simetris**:
      - **Buat** menormalkan nilai form sendiri, jadi nilai dikirim apa adanya.
      - **Ubah** menyimpan body mentah, jadi batas dinormalkan di klien (×1000 untuk waktu, ×1.000.000 untuk kuota) dan angka dikirim sebagai **string**. `cleanFormData` membuang angka `0`, sedangkan `'0'` lolos.
    - Kolom yang dikosongkan saat ubah tidak menghapus nilai tersimpan, kecuali IP statis (`null`).
    - Respons ubah 200 yang berisi galat validasi tetap dikenali (`extractUpdateError`).
    - Sesi mitra tidak pernah mengirim `profile` (server memasang profil paket turunannya).
  - `profilePayload.js` [NEW]
    - Endpoint profil membaca dan menulis satuan database mentah (ms/byte), jadi form mengonversi ke dua arah.
    - Ubah hanya mengirim yang berubah: dikosongkan = `remove-data`, kecepatan dikosongkan = `speed_limit: ''` (server menghapus ketiga field kecepatan bersama).
  - `speedLimit.js` [NEW]
    - Port `mikrotikBurst.js` web supaya string yang dikirim identik.
    - Kecepatan yang tidak diatur = **field tidak dikirim, bukan `0K/0K`** (aturan Bab 9).
  - `accountSchema.js`, `profileSchema.js` [NEW]
    - Kata sandi wajib saat tambah, opsional saat ubah.
    - Username tanpa spasi, IP statis butuh pool dan IPv4 sah, angka tidak negatif.
  - `accountStatus.js`, `accountDisplay.js`, `usage.js`, `cacheKeys.js` [NEW]
    - Status (isolir > nonaktif > aktif), label mode tagihan/jenis pelanggan/pool, laporan bulanan, persen pemakaian, byte dan durasi.

  **Service & endpoint**

  - `services/radiusApi.js` [NEW]
    - Akun (daftar, statistik, detail, tambah, ubah, status, hapus, log), sesi online, profil, serta pilihan paket dan IP Pool.
    - `disconnectSession` sukses **hanya** bila server menjawab 200; 500 berarti radiusd tidak memutus sesi.
  - `services/endpoints.js`
    - Path `radius/*` tambahan.

  **Halaman (`pages/radius/`)**

  - `pages/RadiusPage.jsx`
    - Tiga tab memakai komponen `AccountsTab`, `SessionsTab`, dan `ProfilesTab` [NEW].
  - `AccountDetailPage.jsx` [NEW], dengan `AccountInfoTab`, `AccountUsageTab`, `AccountHistoryTabs` [NEW].
    - Status koneksi disegarkan berkala tanpa skeleton.
  - `AccountFormPage.jsx` [NEW]
    - Pemilih pelanggan, paket, profil (admin), IP Pool + IP statis, kecepatan, batas waktu/kuota, dan kata sandi acak.
  - `ProfileDetailPage.jsx`, `ProfileFormPage.jsx` [NEW], `components/SpeedFields.jsx` [NEW]
  - `components/useDisconnect.jsx` [NEW]
    - Putus sesi dengan konfirmasi; 404/409 dari server (teks Inggris mentah) diganti pesan i18n.
    - Setelah sukses, data dimuat ulang beberapa kali karena Accounting-Stop datang menyusul.

  **Komponen & hook umum**

  - `components/CustomerPicker.jsx` [NEW]
    - Sesi mitra memilih dari pelanggan miliknya.
    - Admin memilih jenis (Reguler/Mitra) lalu mencari. Mitra pemilik dikembalikan untuk pilihan IP Pool.
    - Pelanggan bisnis tidak muncul karena memakai layanan dedicated, bukan PPPoE.
  - `components/form/SearchSelectField.jsx`, `PasswordField.jsx` [NEW], `fields.jsx`
    - Pilihan dengan pencarian, kata sandi dengan tombol lihat/acak, dan input bersatuan.
  - `components/CollapsibleSection.jsx`, `InfoCard.jsx` [NEW]
  - `components/InfoRow.jsx`, `components/LocationCard.jsx`
    - Dipindah dari `pages/customers/components/` supaya dipakai bersama modul Pelanggan dan RADIUS.
  - `hooks/useAutoRefresh.js` [NEW]
    - Pola web: tick berikutnya dijadwalkan setelah permintaan selesai, dilewati saat tab tersembunyi, dan langsung menyegarkan saat tab terlihat lagi.
  - `hooks/useListParams.js` [NEW], `hooks/usePagedList.js`
    - State daftar di URL dipisah menjadi hook sendiri, dipakai daftar pelanggan dan RADIUS.
  - `utils/money.js`, `password.js`, `time.js` [NEW]
  - `pages/customers/components/CustomerServicesTab.jsx`, `CustomerDetailPage.jsx`
    - Kartu akun PPPoE menautkan ke detail akun.
  - `App.jsx`
    - Rute `/radius/accounts/*` dan `/radius/profiles/*`.
  - `i18n/locales/{id,en}/translations.json`
    - Key `radius.*`, sekitar 190 baris masing-masing.

  **Dokumentasi**

  - `AGENTS.md` §2.M butir 4
    - Daftar halaman memakai `usePagedList` + `Pagination` + `useListParams` dengan cache `listCache`, dan detail/form wajib menyegarkan cache.
    - Payload akun PPPoE wajib lewat `accountPayload.js`, dan payload profil lewat `profilePayload.js`.
  - `mobile-frontend/PRD.md`
    - Daftar memakai paginasi seperti web.
    - Modul B (#559) dan C (#561) ditandai "Selesai, menunggu PR".
  - `mobile-frontend/README.md`
    - Struktur folder dan daftar komponen/hook diperbarui.

  **Test**

  - `src/features/radius/radius.test.js` [NEW], 22 skenario:
    - `speedLimit` (3): tidak diatur = string kosong, format burst, dan parsing.
    - `accountPayload` (9): buat apa adanya, IP statis bersama pool, normalisasi satuan saat ubah, kata sandi hanya bila diisi, melepas IP statis, mitra tanpa profil, galat dalam respons 200.
    - `accountSchema` (2), `profilePayload` (3).
    - Tampilan akun (5): urutan status, label, laporan bulanan, persen pemakaian, dan kata sandi acak.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #556 | `996e966c` resolve #556 | Penyelesaian fondasi `mobile-frontend`: Context → Redux Toolkit, semua teks lewat i18n (`id`/`en`), halaman dimuat per rute; 23 test tambahan (sudah di-merge) |
| #559 | `f3ba4581`, `fbf2a774`, `e6f981aa`, `19e43b37` save #559 | Modul Pelanggan (Reguler/Mitra/Bisnis): daftar berhalaman dengan cache, detail bertab, form + dokumen, pemilih lokasi di peta dengan isi alamat otomatis, toast, penjaga form, pengaturan tema & bahasa; 31 test |
| #561 | `a6a225c3` save #561 | Modul Layanan RADIUS: akun PPPoE (daftar, detail, form, status, putus sesi), sesi online, profil bandwidth, payload sesuai kontrak Partner API; 22 test |

> Catatan merge: `resolve #556` sudah di-merge ke `master` pada 17 November, 17:46.
>
> Catatan branch: `issue-561` memuat salinan hasil rebase #556 dan #559, bukan commit yang ada di `master`. Sebelum PR, `issue-561` perlu di-rebase ke `master` terbaru supaya #556 tidak masuk dua kali. Pilihan PR yang lain: gabungkan #559 lebih dulu, atau ajukan #561 sekaligus membawa #559.
