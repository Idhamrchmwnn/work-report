# 📝 Daily Work Report - Idham (2026-11-23)

---

## 📅 Laporan Harian - 23 November 2026

---

## 🌿 Branch: `issue-580` — Mobile Frontend Modul G: Jaringan (Perangkat, Site & Node, Peta)

### 📌 Informasi Issue

- **Nomor Issue**: #580
- **Judul Issue**: Mobile Frontend: modul Jaringan (perangkat jaringan, site & node, peta kabel) + pengetatan endpoint jaringan di Partner API
- **Status Branch**: `Belum di-merge`.
  - Commit `resolve #580` (`c45c1f7e`) berada di atas `master` terbaru (`resolve #578`) dan sudah ada di `origin/issue-580`.
  - Commit ini menggabungkan empat commit `save #580` (`0a4776b8` 15:33, `a4e79d22` 16:32, `9af2fc4b` 16:52, `df98414a` 17:09). Rinciannya di bawah dikelompokkan per berkas.

### 📅 Rincian Commit

#### [c45c1f7e] - resolve #580 - 23 November 2026, 17:10

- **Ringkasan**:
  - **Halaman Jaringan** di mobile kini memakai data asli, bukan data contoh. Isinya tiga tab:
    - **Perangkat**: daftar dengan pencarian dan filter status, statistik (Total/Online/Offline/latensi), detail lima bagian seperti drawer web, serta tambah/ubah/hapus.
    - **Site & Node**: peta site, filter Tipe/Grup, kartu site berisi ringkasan node dan port terpakai/total, detail site, tambah/ubah site, serta tambah/kelola node.
    - **Peta**: marker site dan jalur kabel (hanya baca), detail node, detail kabel, dan sambungan per core.
  - **Partner API jaringan diperketat**:
    - Body perangkat jaringan memakai daftar-putih. Sebelumnya body diteruskan utuh, sehingga token mitra bisa menulis `status`, `partner`, `created_by`, dan field sistem lain.
    - POP hanya boleh site dalam cakupan pemanggil.
    - Admin kini ditolak 404 untuk node, kabel, dan perangkat milik mitra yang disembunyikan dari portal.
    - Kontak PIC site publik perusahaan tidak lagi dikirim ke mitra.
    - Query berbentuk array/objek dijawab 400, bukan 500.
  - **Yang tidak dibawa**: Trace/OTDR, hapus site/node, dan daftar akun broadband per POP. Partner API belum punya endpoint untuk ketiganya (dicatat di PRD).

- **Komponen yang Berubah & Penjelasannya**:

  - **Backend**
    - `backend/src/utils/partnerApiNetworkDeviceInput.js` (baru)
      - `pickPartnerApiNetworkDeviceInput`: daftar-putih field form `partner-frontend`. Field lain dibuang diam-diam.
      - Objek pada field teks (mis. operator Mongo) ditolak, string kosong dibuang (nilai lama tidak terhapus), `snmp_port` dibatasi 1..65535, dan `snmp_version` harus `1`/`2c` (alias `v1`/`v2c` dinormalkan).
      - `custom_oid` hanya menerima sub-field skema.
      - Batas panjang teks dibuat longgar, cukup untuk menjaga ukuran payload. Data lama (mis. `sysDescr` Cisco yang panjang) tetap lolos saat form ubah web mengirimnya balik.
      - `partner` hanya diterima saat create oleh sesi admin.
    - `backend/src/controllers/partnerApiNetworkDevice.controller.js`
      - Cakupan satu data (detail/ubah/hapus): mitra hanya perangkat miliknya; admin semua perangkat kecuali milik mitra tersembunyi. Di luar cakupan dijawab 404. Sebelumnya hapus perangkat tak dikenal berakhir 500.
      - POP (`node`) diubah menjadi `_id` site yang boleh dipakai, dengan aturan sama seperti `site/pop-select`. POP di luar cakupan dan POP yang tidak ada dijawab sama ("POP tidak ditemukan"), supaya keberadaan site mitra lain tidak bisa ditebak.
      - Saat ubah, POP yang sama dengan POP terpasang tidak divalidasi ulang. Tanpa itu, perangkat yang POP lamanya sudah di luar cakupan tidak bisa diubah sama sekali.
      - Datatable: bentuk body diperiksa lebih dulu, dan `withDeleted` dari klien dibuang. `created_by` tidak lagi membawa flag `super`.
    - `backend/src/controllers/partnerApiMap.controller.js`
      - Helper `canReadMapDoc` dipakai di semua jalur satu-data peta (detail/laporan node, kabel, sambungan, topologi). Aturannya sama dengan daftar peta: mitra melihat miliknya atau publik; admin melihat semua kecuali milik mitra tersembunyi.
      - Kontak PIC site publik dibuang dari detail dan laporan node untuk mitra.
      - Query teks opsional divalidasi: array/objek dijawab 400.
    - `backend/src/controllers/partnerApiSite.controller.js`
      - Daftar site membawa `node_summary` (jumlah node + maks. 3 node dengan port terpakai/total), tetapi hanya untuk mitra dan admin pemegang `networkSite.read`. Tanpa syarat itu, daftar site menjadi jalan pintas membaca node tanpa hak akses.
      - `type-select`/`group-select` tanpa kata kunci kini langsung mengembalikan daftar, supaya dropdown filter langsung berisi pilihan.
    - `backend/src/services/locationPoint.service.js`
      - `findTypeSelect`/`findGroupSelect` mendapat opsi `listAll`: urut abjad, tanpa nilai kosong, maks. `SITE_SELECT_LIST_LIMIT` (50). Perilaku API utama tetap autocomplete.
      - `findSiteObjectIdForPartnerApi` (baru) hanya menerima `maps_id`/ObjectId, bukan nama site, karena nama tidak unik dan bisa menaut ke site mitra lain.
      - `stripForeignSitePic` (baru) memakai aturan yang sama dengan `hidePublicPic` di `site/read`.
      - `summarizeSiteNodes` (baru) memakai dua query untuk satu halaman (bukan satu per site), dengan aturan port sama dengan `decorateEquipments`.
    - `backend/src/services/networkDevice.service.js`
      - `findOneNetworkDevice` menerima opsi `siteSelect`/`createdBySelect`.
      - `findOneNetworkDeviceForPartnerApi` membuang PIC site publik untuk sesi mitra.
    - `backend/src/routes/partnerApi.route.js`
      - Swagger endpoint jaringan diperbarui: field yang diterima, `node_summary`, 400/404 baru, dan daftar tanpa kata kunci.
    - `backend/src/locales/{id,en}/translation.json`
      - Key `networkDevice.popNotFound`.
  - **Mobile frontend**
    - `mobile-frontend/src/pages/NetworkPage.jsx`
      - Data contoh diganti tiga tab: `DevicesTab`, `SitesTab`, dan `MapTab` (`pages/network/components/`).
    - `mobile-frontend/src/pages/network/DeviceDetailPage.jsx`, `DeviceFormPage.jsx` (baru)
      - Detail perangkat (community SNMP tampil + tombol salin) dan form tambah/ubah. Form memberi tahu bahwa field yang dikosongkan tidak menghapus nilai lama, sama dengan web.
    - `mobile-frontend/src/pages/network/SiteDetailPage.jsx`, `SiteFormPage.jsx`, `NodeFormPage.jsx` (baru)
      - Detail site + daftar node, form site, dan form node (nama, rasio, kondisi & catatan port).
      - Site publik bagi mitra hanya bisa dibaca.
    - `mobile-frontend/src/pages/network/components/NodeSheet.jsx`, `CableSheet.jsx`, `NodePortsSheet.jsx` (baru)
      - Detail node (info seperti drawer web, perangkat fiber, kabel terhubung, sambungan, topologi berupa daftar).
      - Detail kabel (panjang aktual = jarak × 1,025 + slack, seperti web; sambungan per core).
      - Port node hanya baca untuk pengguna tanpa hak kelola node.
    - `mobile-frontend/src/components/map/NetworkMap.jsx` (baru)
      - Peta jaringan hanya baca: marker site dan jalur kabel di canvas.
    - `mobile-frontend/src/components/SearchFilterSelect.jsx` (baru)
      - Filter dengan pilihan dari server plus pencarian, dipakai filter Tipe/Grup site.
    - `mobile-frontend/src/features/network/*` (baru, logika murni)
      - `deviceStatus`, `devicePayload`/`deviceSchema`, `sitePayload`/`siteSchema`, `nodePayload`, `siteAccess` (hak ubah site), `portSummary`, `mapStyle`, `listSort`, `cacheKeys`.
    - `mobile-frontend/src/services/networkApi.js` (baru), `endpoints.js`
      - Pemanggil endpoint `network-devices/*`, `site/*`, `node/*`, dan `map/*`.
    - `mobile-frontend/src/App.jsx`
      - Rute detail/form perangkat, site, dan node.
    - `mobile-frontend/src/data/mockData.js`
      - Data contoh jaringan dihapus.
    - `mobile-frontend/src/i18n/locales/{id,en}/translations.json`
      - Key modul Jaringan.
    - `mobile-frontend/PRD.md`, `README.md`
      - Baris modul G dirinci (apa yang dibawa/tidak dibawa).
      - Status modul 6 diubah menjadi "#580 — Selesai, menunggu uji perangkat & PR".
      - Struktur folder baru dicatat di README.
  - **Test**
    - `backend/test/integration/partnerApiNetworkDeviceInput.test.js` (baru)
      - 14 test: daftar-putih, operator Mongo, POP di luar cakupan, ubah dengan POP lama, cakupan admin/mitra, dan `withDeleted`.
    - `backend/test/integration/partnerApiMap.test.js`
      - +3 test: PIC site publik, admin 404 untuk data mitra tersembunyi, query array/objek 400.
    - `backend/test/integration/partnerApiSite.test.js`
      - +4 test: daftar tipe/grup tanpa kata kunci, API utama tetap autocomplete, `node_summary` untuk mitra, `node_summary` hanya untuk admin dengan hak akses.
    - `backend/test/integration/partnerApiNetworkDevice.test.js`
      - Disesuaikan, +1 test.
    - `mobile-frontend/src/features/network/network.test.js` (baru)
      - 22 test logika murni modul Jaringan.

---

## 🌿 Branch: `issue-582` — Mobile Frontend Modul H: Organisasi (Mitra, Karyawan, Admin, Hak Akses)

### 📌 Informasi Issue

- **Nomor Issue**: #582
- **Judul Issue**: Mobile Frontend: modul Organisasi (daftar & detail mitra, karyawan, admin, hak akses — khusus admin, hanya baca)
- **Status Branch**: `Masih dikerjakan` (baru commit `save`).
  - `c09c9eaa` sudah ada di `origin/issue-582`, ditumpuk di atas `resolve #580`.

### 📅 Rincian Commit

#### [c09c9eaa] - save #582 - 23 November 2026, 17:30

- **Ringkasan**:
  - **Halaman Organisasi** (khusus admin, hanya baca) kini memakai data asli dengan empat tab: Mitra, Karyawan, Admin, dan Hak Akses.
  - **Detail mitra (halaman baru)**:
    - Kontak (telepon/WhatsApp/email), alamat + peta, informasi mitra, dan petugas pendamping.
    - Dokumen legalitas bisa dilihat dan diunduh, tetapi tidak diunggah; unggah tetap oleh mitra atau lewat web.
  - **"Role" diganti istilahnya menjadi Hak Akses** (koleksi `privileges`), supaya tidak tertukar dengan `user.role` (`partner`/`admin`).
    - Daftar izin di dalam hak akses dan role super tidak dikirim Partner API, jadi tidak ditampilkan.
    - Akun yang hak aksesnya `null` diberi keterangan.
  - **Pemakaian ulang dokumen**: logika lihat/unduh berkas ber-Bearer yang dibuat di #576 dipindah ke hook bersama `useDocumentFiles`. Halaman Dokumen Mitra (profil) dan detail mitra kini memakai hook yang sama.

- **Komponen yang Berubah & Penjelasannya**:

  - `mobile-frontend/src/pages/OrganizationPage.jsx`
    - Data contoh diganti tab `PartnersTab`, `StaffTab` (dipakai untuk Karyawan dan Admin), dan `PrivilegesTab`.
  - `mobile-frontend/src/pages/organization/components/OrganizationList.jsx` (baru)
    - Daftar berhalaman bersama (pencarian, halaman di URL, cache). Halaman di URL yang melewati halaman terakhir dipindah ke halaman akhir.
  - `mobile-frontend/src/pages/organization/components/StaffTab.jsx`, `StaffSheet.jsx` (baru)
    - Administrator (`super: true`) dan karyawan (`super: false`) ada di satu koleksi, tetapi endpoint dan hak aksesnya terpisah, jadi tabnya juga terpisah. Detail ditampilkan di panel bawah.
  - `mobile-frontend/src/pages/organization/components/PartnersTab.jsx`, `PrivilegesTab.jsx` (baru)
    - Daftar mitra (status aktif) dan daftar hak akses (nama + keterangan).
  - `mobile-frontend/src/pages/organization/PartnerDetailPage.jsx` (baru)
    - Detail mitra. Data terbaru dari detail juga memperbarui kartu di daftar.
  - `mobile-frontend/src/features/organization/organizationView.js`, `listSort.js` (baru)
    - Penyusun tampilan murni: status & alamat mitra, petugas pendamping, nama hak akses.
    - Urutan daftar tetap dan selalu berakhir di kolom unik, supaya data tidak dobel atau hilang antarhalaman.
  - `mobile-frontend/src/hooks/useDocumentFiles.js` (baru)
    - Lihat/unduh berkas ber-Bearer beserta pratinjau, dipakai profil dan detail mitra.
  - `mobile-frontend/src/pages/profile/DocumentsPage.jsx`, `components/DocumentCard.jsx`, `components/DocumentPreview.jsx` (baru)
    - Dirapikan agar memakai `useDocumentFiles`. `DocumentCard` kini bisa tampil dalam mode hanya-lihat.
  - `mobile-frontend/src/services/accountApi.js`
    - Pemanggil dokumen menerima `partnerId`, yang wajib untuk sesi admin; server memverifikasi bahwa berkas memang milik mitra itu.
  - `mobile-frontend/src/services/organizationApi.js` (baru), `endpoints.js`
    - Detail mitra, `admins/*`, `employees/*`, `privileges/*`. Id di path adalah id publik (`admin_id`/`privilege_id`).
  - `mobile-frontend/src/App.jsx`
    - Rute detail mitra.
  - `mobile-frontend/src/data/mockData.js`
    - Data contoh organisasi dihapus.
  - `mobile-frontend/src/i18n/locales/{id,en}/translations.json`
    - Key modul Organisasi.
  - `mobile-frontend/PRD.md`, `README.md`
    - Baris modul H dirinci, status modul 7 diubah menjadi "#582 — Selesai, menunggu uji perangkat & PR", dan struktur folder dicatat.
  - `mobile-frontend/src/features/organization/organization.test.js` (baru)
    - 12 test penyusun tampilan dan urutan daftar.

---

## 📊 Ringkasan Hari Ini

| Issue | Commit | Pekerjaan |
|---|---|---|
| #580 | `0a4776b8`, `a4e79d22`, `9af2fc4b`, `df98414a` save; `c45c1f7e` resolve #580 | Modul Jaringan di mobile (Perangkat, Site & Node, Peta hanya baca); Partner API jaringan diperketat (daftar-putih body perangkat, POP dalam cakupan, 404 untuk data mitra tersembunyi, PIC site publik tidak dikirim ke mitra, `node_summary` berhak akses); +44 test |
| #582 | `c09c9eaa` save | Modul Organisasi di mobile (Mitra + detail & dokumen hanya lihat, Karyawan, Admin, Hak Akses), hook bersama `useDocumentFiles`; +12 test |

> Catatan merge: `resolve #576` (dikerjakan 20 November) sudah di-merge ke `master` pada 21 November, 15:21–15:28, termasuk penyesuaian dari Dedy S.N Putra.
>
> Catatan branch: `issue-582` ditumpuk di atas `resolve #580`, jadi #580 sebaiknya di-merge lebih dulu. Simulasi merge (`git merge-tree`) `issue-582` ke `master` tidak menunjukkan konflik.
