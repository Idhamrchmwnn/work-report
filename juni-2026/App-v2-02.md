# 📝 Daily Work Report - Idham (2026-06-02)

---

## 📌 Informasi Issue
- **Nomor Issue**: #93
- **Judul Issue**: Perbaikan Privilege Key Tiket & Refactoring Komponen Shared

## 📅 Laporan Harian - 2 Juni 2026

### 🛠️ Pekerjaan Belum Di-commit / Work in Progress (WIP)

Tidak ada pekerjaan yang masih dalam proses. Semua perubahan telah di-commit.

---

### 📅 Rincian Commit

#### [[3a67a10]](3a67a10) - resolve #93

- **Komponen yang Berubah**:
  - `frontend/src/app/pages/network/sites/create.jsx`
  - `frontend/src/app/pages/network/sites/edit.jsx`
  - `frontend/src/app/pages/network/sites/editBatch.jsx`
  - `frontend/src/app/pages/network/sites/schema/createSchema.js`
  - `frontend/src/app/pages/tickets/customer/index.jsx`
  - `frontend/src/app/pages/tickets/dismantle/index.jsx`
  - `frontend/src/app/pages/tickets/installation/index.jsx`
  - `frontend/src/app/pages/tickets/partner/index.jsx`
  - `frontend/src/app/pages/tickets/survey/index.jsx`
  - `frontend/src/components/shared/SiteMap.jsx`
  - `frontend/src/components/shared/SiteTopologyMap.jsx`
  - `frontend/src/components/shared/form/Listbox.jsx`
  - `frontend/src/components/shared/table/ColumnFilter.jsx`

- **Deskripsi Perubahan & Fungsi**:
  - `tickets/customer/index.jsx`, `dismantle/index.jsx`, `installation/index.jsx`, `partner/index.jsx`, `survey/index.jsx` — Memperbaiki privilege key yang digunakan pada kondisi tampil tombol aksi di masing-masing halaman list tiket. Key lama `ticket.create` diganti menjadi privilege spesifik per jenis tiket (misal `ticketCustomer.create`, `ticketInstallation.create`, dst.) agar kontrol hak akses lebih granular dan tidak saling tumpang tindih antar modul tiket.
  - `SiteTopologyMap.jsx` — Refactoring struktur `createIcon` helper menjadi multi-line untuk keterbacaan, menambahkan dukungan tipe jaringan `DataAccess` dengan warna marker orange dan warna garis `#f97316`, serta perbaikan formatting JSX secara menyeluruh agar konsisten dengan standar penulisan kode proyek.
  - `SiteMap.jsx` — Penyesuaian minor pada komponen peta situs, kemungkinan sinkronisasi perubahan konfigurasi icon atau koordinat default dengan `SiteTopologyMap.jsx`.
  - `sites/create.jsx`, `sites/edit.jsx`, `sites/editBatch.jsx` — Perbaikan formatting JSX (multi-line props) pada field input koordinat dan label section Aesthetics agar konsisten dengan linter dan standar kode. Tidak ada perubahan fungsional.
  - `sites/schema/createSchema.js` — Penyesuaian aturan validasi Yup pada skema form sites, kemungkinan penambahan atau relaksasi validasi field koordinat terkait perubahan di halaman create/edit.
  - `Listbox.jsx` — Perbaikan indentasi dan format ternary operator pada blok render `selectedValue` dan `renderOption` di dalam ListboxButton dan ListboxOptions agar lebih mudah dibaca dan tidak memicu warning formatter.
  - `ColumnFilter.jsx` — Perbaikan format rendering label item filter dari satu baris menjadi multi-line pada komponen `ColumnFilter`, tidak ada perubahan logika atau fungsionalitas.
