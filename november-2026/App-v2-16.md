# 📝 Daily Work Report - Idham (2026-11-16)

---

## 📅 Laporan Harian - 16 November 2026

---

## 🌿 Branch: `issue-556` — Fondasi Mobile Frontend (Partner API, Sesi, PWA) & Cakupan Fitur

### 📌 Informasi Issue

- **Nomor Issue**: #556
- **Judul Issue**: Mobile Frontend: fondasi klien Partner API, sesi login, PWA, dan cakupan fitur di PRD
- **Status Branch**: `Belum di-commit`. Pekerjaan masih di working tree branch `issue-556`, di atas `master` terbaru (`resolve #554`, yang membawa desain awal `mobile-frontend`). Ada 60 berkas berubah/baru, sekitar +4.750 / −240 baris.

### 📅 Rincian Pekerjaan (belum di-commit)

- **Ringkasan**:
  - Desain awal `mobile-frontend` (PWA mobile-first untuk Mitra/POP dan Karyawan Internal) diberi fondasi teknis supaya modul-modulnya bisa dihubungkan ke **Partner API** (`/p-api/v1`).
    - Tampilan tidak diubah. Berkas desain awal dimasukkan ke `.prettierignore` supaya isinya tetap sama persis.
  - **Klien API** kini hanya boleh memanggil Partner API.
    - Interceptor menolak URL di luar `/p-api/v1`.
    - Satu-satunya pengecualian adalah klien publik tanpa token untuk nama dan logo aplikasi.
    - Semua galat diseragamkan menjadi `ApiError`.
  - **Sesi** disesuaikan dengan Partner API yang tidak punya refresh token:
    - 401 berarti sesi habis, dan pengguna kembali ke halaman login **sekali** dengan pesan "Sesi Anda telah berakhir".
    - Backend yang tidak terjangkau tidak dianggap sesi habis, jadi token dipertahankan.
    - Hanya token yang disimpan; profil pengguna selalu diambil ulang dari server.
    - Data contoh (`MOCK_USERS`) tidak lagi tergabung ke profil. Sebelumnya HP, email, dan statistik palsu bisa tampil.
    - Tujuan `?redirect=` setelah login dibatasi ke path internal (`safeRedirect`).
  - **PWA** mengikuti aturan PWA di `frontend` (AGENTS.md §9):
    - service worker kustom dengan mode `prompt`;
    - navigasi dan API selalu ke jaringan;
    - precache hanya shell, tanpa `index.html`;
    - cache aset tidak pernah menyimpan HTML;
    - pemeriksaan pembaruan berkala;
    - ikon PNG dan halaman offline.
  - **Infrastruktur daftar** untuk datatable Partner API: infinite scroll, dengan respons basi dibuang lewat `requestId`.
  - Modul kini punya ESLint, Prettier, dan Vitest. Ada **53 skenario test** logika murni.
  - **PRD** Bab 5 dan 6 diisi:
    - 19 modul dalam 2 fase, masing-masing dengan endpoint dan aksinya;
    - pola umum dan daftar di luar cakupan;
    - butir terbuka yang perlu persetujuan;
    - matriks menu per peran.
  - Modul didaftarkan ke orkestrator dev root (`npm run dev -- --with mobile`, port 4003), dan AGENTS.md mendapat bagian §2.M.

- **Komponen yang Berubah & Penjelasannya**:

  **Klien API & layanan**

  - `mobile-frontend/src/configs/server.config.js` [NEW]
    - Satu sumber alamat backend: `P_API_URL` (`/p-api/v1`) dan `PUBLIC_API_URL` (`/api/v1`, hanya untuk nama dan logo aplikasi).

  - `mobile-frontend/src/utils/apiClient.js` [NEW]
    - Satu-satunya klien HTTP untuk data aplikasi, dengan `baseURL` Partner API.
    - Interceptor request:
      - menolak URL di luar prefix (`apiBoundary.js`);
      - memasang Bearer token yang dibaca per request.
    - Interceptor response:
      - 401 menandai sesi berakhir lalu mengalihkan ke `/login?redirect=…` **sekali** (penjaga `hasRedirected`), walau beberapa request gagal bersamaan;
      - semua galat di-reject sebagai `ApiError`.

  - `mobile-frontend/src/utils/apiBoundary.js` [NEW]
    - `isAllowedApiUrl`: path relatif diterima, sedangkan URL absolut hanya bila berada di bawah `baseURL` Partner API.

  - `mobile-frontend/src/utils/apiError.js` [NEW] dan `errors.js` [NEW]
    - `ApiError` (status, pesan backend, `isNetworkError`) dan `toApiError` dari galat axios.
    - `getErrorMessage` memakai pesan backend apa adanya. Galat jaringan dan galat internal memakai pesan umum Bahasa Indonesia.

  - `mobile-frontend/src/utils/publicClient.js` [NEW]
    - Klien tanpa token untuk Core API publik, dengan daftar-putih dua path: `configuration/company-information` dan `file/app-logo`.

  - `mobile-frontend/src/services/endpoints.js` [NEW]
    - Seluruh path Partner API yang dipakai aplikasi ada di satu tempat, agar mudah dicocokkan dengan `partnerApi.route.js`. Parameter path di-encode di sini.

  - `mobile-frontend/src/services/authApi.js` [NEW] dan `partnerApi.js` [NEW]
    - `login`, `logout`, `getProfile`, dan `listPartners` (sumber pemilih mitra untuk admin).

  - `mobile-frontend/src/lib/api.js`
    - Kini hanya meneruskan ekspor dari `src/utils`, supaya impor lama tetap bekerja.
    - Logika lama dihapus: klien axios dengan header Authorization statis, `setAuthSession`, dan `isTokenValid` lokal.

  **Sesi & autentikasi**

  - `mobile-frontend/src/context/AuthContext.jsx`
    - Memakai `authApi`, `authStorage`, `jwt`, dan `normalizeUser`. Interceptor 401 lokal dihapus, karena kini ditangani `apiClient`.
    - Pemulihan sesi saat aplikasi dibuka:
      - token tidak valid dibersihkan;
      - galat jaringan menampilkan pesan, tetapi token tetap disimpan;
      - galat lain membersihkan sesi.
    - Saat login, profil lengkap diambil dari `/partners/profile` lalu digabung dengan data login. Peran dari respons login dipakai karena DTO mitra tidak membawa `role`.
    - Pesan "Sesi Anda telah berakhir" tampil sekali setelah dialihkan (`sessionNotice`).

  - `mobile-frontend/src/utils/authStorage.js` [NEW] dan `storage.js` [NEW]
    - Hanya token (`authToken`, `refreshToken`) yang disimpan.
    - Akses Web Storage dibungkus try/catch, supaya aplikasi tetap jalan saat storage diblokir (mode privat).

  - `mobile-frontend/src/utils/jwt.js` [NEW]
    - `isTokenValid` (cek `exp` di sisi klien) untuk menghindari request yang pasti 401.

  - `mobile-frontend/src/utils/normalizeUser.js` [NEW]
    - Memetakan profil mitra dan admin ke bentuk `user` yang dibaca halaman.
    - Field kosong diisi `-`, **tidak pernah** diisi data contoh.

  - `mobile-frontend/src/utils/safeRedirect.js` [NEW] dan `sessionNotice.js` [NEW]
    - `safeRedirect` menolak URL absolut, `//host`, dan skema lain. Dipakai di `App.jsx` dan `pages/LoginPage.jsx`.
    - `sessionNotice` menyimpan penanda "sesi berakhir" di sessionStorage supaya bertahan melewati pemuatan ulang.

  **Daftar & pemuatan data**

  - `mobile-frontend/src/utils/dataTable.js` [NEW]
    - `buildListParams` menyusun body datatable `{ pageIndex, pageSize, globalFilter, columnFilters, sorting }`, dengan `pageSize` maksimal 100.
    - `parseListResponse` menormalkan respons `{ list, totalDocs, totalPage, … }`.

  - `mobile-frontend/src/hooks/useInfiniteList.js` [NEW] dan `infiniteListReducer.js` [NEW]
    - Infinite scroll untuk daftar Partner API.
    - Reducer murni dengan `requestId`: respons pencarian lama atau respons sebelum reset dibuang.

  - `mobile-frontend/src/hooks/useApiResource.js`, `useDebounce.js`, `useOnline.js` [NEW]
    - Memuat satu sumber data (membatalkan permintaan basi), debounce input pencarian, dan status online/offline.

  **PWA**

  - `mobile-frontend/src/pwa/sw.js` [NEW]
    - Service worker kustom (`injectManifest`).
    - Navigasi dan seluruh `/p-api/*` dan `/api/*` selalu ke jaringan; offline menampilkan `offline.html`.
    - Versi baru menunggu `SKIP_WAITING` dari tombol "Muat ulang".
    - Saat dipasang/aktif, entri cache yang terkontaminasi HTML dibersihkan.

  - `mobile-frontend/src/pwa/assetCache.js`, `shellEntries.js`, `updateScheduler.js` [NEW]
    - Disalin dari `frontend/src/pwa` beserta test-nya:
      - `rejectHtmlPlugin` dan `purgeHtmlEntries`;
      - `selectShellEntries`, yang memilih aset yang dirujuk `index.html` ditambah `offline.html`;
      - pemeriksaan pembaruan berkala (1 jam), dengan jeda minimal 5 menit.

  - `mobile-frontend/src/components/PwaUpdatePrompt.jsx` [NEW]
    - Mendaftarkan service worker dan menampilkan tawaran "Muat ulang" saat versi baru tersedia. Dipasang di `App.jsx`.

  - `mobile-frontend/vite.config.js`
    - PWA diganti ke `strategies: 'injectManifest'` dan `registerType: 'prompt'` (sebelumnya `autoUpdate`).
    - Manifest memakai nama dari `VITE_APP_NAME` dan ikon PNG 192/512/maskable.
    - `manifestTransforms` membatasi precache ke shell.
    - Plugin `noCdnCacheInDev` mengirim `CDN-Cache-Control: no-store` saat dev.
    - Port dibaca dari `VITE_PORT`, ditambah alias `@` dan `dedupe` React.

  - `mobile-frontend/public/images/pwa/*.png` [NEW], `public/offline.html` [NEW], dan `index.html`
    - Ikon PWA, halaman offline, dan `apple-touch-icon`.

  **Tooling & dokumentasi**

  - `mobile-frontend/package.json`
    - Skrip `lint`, `lint:fix`, `format`, `test`, `test:watch`.
    - Dependensi Workbox, `vite-plugin-pwa` v2, ESLint, Prettier, dan Vitest.
  - `mobile-frontend/eslint.config.js`, `prettier.config.js`, `.prettierignore`, `jsconfig.json`, `vitest.config.js` [NEW]
    - ESLint melarang `console.log`.
    - `.prettierignore` melindungi berkas desain awal.
    - Vitest berjalan di environment `node`.
  - `mobile-frontend/.env.example` dan `.gitignore`
    - Penjelasan `VITE_API_URL`, nama cadangan aplikasi, dan `VITE_PORT`; `dev-dist` diabaikan.
  - `mobile-frontend/README.md` [NEW]
    - Cara menjalankan, daftar perintah, dan struktur folder.
  - `mobile-frontend/PRD.md`
    - Status menjadi "Draf — Bab 5 & 6 terisi, menunggu persetujuan".
    - Pengecualian tunggal "Zero New Backend API" dicatat: nama dan logo aplikasi diambil dari Core API publik.
    - Keterbatasan logout dicatat: `refreshToken` di body GET dibuang browser.
    - Stack disesuaikan dengan desain awal: React Context, teks Indonesia langsung, tanpa Redux/i18n.
    - Bab 5:
      - prinsip cakupan;
      - 19 modul (Fase 1: autentikasi, beranda, pelanggan mitra/bisnis, akun PPPoE, sesi, paket, tagihan, rekening, profil, dokumen; Fase 2: penerbitan tagihan, jaringan, site/node, peta, pelanggan reguler, profil bandwidth, daftar mitra) beserta endpoint dan aksinya;
      - pola umum;
      - di luar cakupan;
      - butir terbuka.
    - Bab 6: matriks menu per peran, struktur navigasi, konteks mitra untuk admin (pemilih `partner_id`), serta beranda dan aksi cepat per peran.
  - `AGENTS.md`
    - Mobile Frontend ditambahkan ke overview ekosistem dan tabel port.
    - Bagian baru §2.M dengan 9 aturan wajib: desain mengikuti tampilan awal, hanya Partner API, aturan sesi, pola daftar, pemilih mitra untuk admin, aturan §2.B yang berlaku, PWA, penghapusan data contoh, dan logging.
    - Catatan runner test `mobile-frontend` di Bab 6.
  - `tools/dev/services.config.mjs`
    - Service opsional `mobile` (port 4003, `VITE_PORT`) di orkestrator `npm run dev`.

  **Test** (Vitest, 53 skenario)

  - `src/utils/apiBoundary.test.js` (4): path relatif, URL di bawah Partner API, serta penolakan Core API dan host lain.
  - `src/utils/apiError.test.js` (4): galat backend, jaringan, dan tanpa respons.
  - `src/utils/dataTable.test.js` (6): body datatable, batas `pageSize`, dan normalisasi respons.
  - `src/utils/jwt.test.js` (3), `safeRedirect.test.js` (3), `normalizeUser.test.js` (7): mitra tanpa `role`, admin, dan field kosong tidak diisi data contoh.
  - `src/hooks/infiniteListReducer.test.js` (5): tambah halaman, reset, dan respons basi dibuang.
  - `src/pwa/assetCache.test.js` (11), `shellEntries.test.js` (6), `updateScheduler.test.js` (4).

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #556 | _belum di-commit_ | Fondasi `mobile-frontend`: klien Partner API berpagar, sesi tanpa refresh token, PWA sesuai standar `frontend`, infrastruktur daftar, tooling dan 53 test, cakupan fitur & matriks menu di PRD, aturan §2.M di AGENTS.md |

> Catatan merge: `resolve #545` (Kinerja Marketing, dikerjakan 13 November) sudah di-merge ke `master` pada 14 November.
