# 01 - Project Overview

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Overview
**Terakhir Diperbarui:** [TANGGAL]
**Pemilik Dokumen:** [NAMA TIM / ROLE]

---

## Dokumen Terkait

| Dokumen | Path | Keterangan |
| --- | --- | --- |
| Fundamental PSD | `docs/psd/00-fundamental-psd.md` | Standar dan aturan penulisan dokumentasi project |
| Setup Guide | `docs/psd/02-setup-guide.md` | Cara menjalankan project dari nol |
| System Architecture | `docs/psd/03-system-architecture.md` | Arsitektur dan struktur teknis project |
| Coding Standard | `docs/psd/04-coding-standard.md` | Standar penulisan kode project ini |

---

## Cara Membaca Dokumen Ini

Dokumen ini adalah titik masuk pertama untuk memahami project. Baca seluruhnya dari atas ke bawah sebelum membuka dokumen lain. Dokumen ini tidak menjelaskan cara kerja teknologi yang digunakan; dokumen ini menjelaskan **apa** dan **mengapa** project ini dibangun, serta **bagaimana** project ini diorganisasi.

Setelah membaca dokumen ini, developer baru diharapkan mampu menjawab:

- Project ini menyelesaikan masalah apa?
- Siapa yang menggunakan project ini dan dalam kapasitas apa?
- Module apa saja yang ada dan apa tanggung jawab masing-masing?
- Ke mana harus pergi untuk membaca detail tentang sesuatu yang spesifik?

---

## Daftar Isi

1. [Identitas Project](about:blank#1-identitas-project)
2. [Tujuan Project](about:blank#2-tujuan-project)
3. [Deskripsi Sistem](about:blank#3-deskripsi-sistem)
4. [Role Pengguna](about:blank#7-role-pengguna)
5. [Module Utama](about:blank#8-module-utama)
6. [Feature Utama](about:blank#9-feature-utama)
7. [High-Level Flow System](about:blank#10-high-level-flow-system)
8. [Dependency Utama](about:blank#11-dependency-utama)
9. [Integrasi Utama](about:blank#12-integrasi-utama)
10. [Environment Overview](about:blank#13-environment-overview)
11. [Repository Overview](about:blank#14-repository-overview)
12. [Struktur Dokumentasi](about:blank#15-struktur-dokumentasi)
13. [Batasan Sistem](about:blank#16-batasan-sistem)
14. [Asumsi Sistem](about:blank#17-asumsi-sistem)
15. [Glossary Project](about:blank#18-glossary-project)

---

## 1. Identitas Project

### 1.1 Informasi Project

| Atribut | Nilai |
| --- | --- |
| Nama Project | [NAMA PROJECT] |
| Kode Project | [KODE SINGKAT — contoh: HRM-CORE, INV-SYS, ECOM-API] |
| Versi Saat Ini | [X.X.X] |
| Status | [Active Development / Maintenance / Deprecated] |
| Tanggal Mulai | [YYYY-MM-DD] |
| Repository | [URL REPOSITORY] |
| Project Board | [URL — Jira, Linear, Trello, atau yang digunakan tim] |
| Dokumentasi Root | `docs/` |

### 1.2 Tim Project

| Peran | Nama | Kontak | Tanggung Jawab |
| --- | --- | --- | --- |
| Product Owner | [NAMA] | [KONTAK] | Penentu prioritas dan arah bisnis project |
| Tech Lead | [NAMA] | [KONTAK] | Penjaga konsistensi teknis dan arsitektur |
| Backend Engineer | [NAMA / TIM] | [KONTAK] | Implementasi Service, Repository, API |
| Frontend Engineer | [NAMA / TIM] | [KONTAK] | Implementasi antarmuka pengguna |
| QA Engineer | [NAMA / TIM] | [KONTAK] | Pengujian fungsional dan non-fungsional |
| DevOps Engineer | [NAMA / TIM] | [KONTAK] | Deployment, infrastruktur, monitoring |

---

## 2. Tujuan Project

### 2.1 Tujuan Bisnis

[NAMA PROJECT] dibangun untuk [DESKRIPSI SATU KALIMAT TUJUAN BISNIS UTAMA].

Project ini menjawab kebutuhan bisnis berikut:

1. **[TUJUAN BISNIS 1]** — [Jelaskan dalam satu kalimat mengapa tujuan ini penting bagi organisasi atau pengguna]
2. **[TUJUAN BISNIS 2]** — [Jelaskan dalam satu kalimat mengapa tujuan ini penting]
3. **[TUJUAN BISNIS 3]** — [Jelaskan dalam satu kalimat mengapa tujuan ini penting]

### 2.2 Tujuan Teknis

Dari perspektif engineering, project ini dibangun dengan tujuan teknis berikut:

1. **[TUJUAN TEKNIS 1]** — [Contoh: Menyediakan REST API yang dapat dikonsumsi oleh aplikasi mobile dan web secara bersamaan]
2. **[TUJUAN TEKNIS 2]** — [Contoh: Memisahkan domain bisnis ke dalam Module yang mandiri agar dapat dikerjakan tim secara paralel]
3. **[TUJUAN TEKNIS 3]** — [Contoh: Mendukung skalabilitas horizontal sejak awal karena estimasi pengguna tumbuh 10x dalam 18 bulan pertama]

### 2.3 Ukuran Keberhasilan

Project ini dianggap berhasil memenuhi tujuannya ketika:

| Indikator | Target | Cara Mengukur |
| --- | --- | --- |
| [INDIKATOR 1 — contoh: Waktu proses approval] | [TARGET — contoh: < 24 jam] | [CARA — contoh: Timestamp dari submission hingga approval di Database] |
| [INDIKATOR 2] | [TARGET] | [CARA] |
| [INDIKATOR 3] | [TARGET] | [CARA] |

---

## 3. Deskripsi Sistem

### 3.1 Gambaran Sistem

[NAMA PROJECT] adalah [JENIS SISTEM — contoh: web application, REST API, platform SaaS] yang berjalan di [PLATFORM — contoh: cloud server, on-premise, hybrid]. Sistem ini digunakan oleh [DESKRIPSI SINGKAT PENGGUNA] untuk [AKTIVITAS UTAMA YANG DAPAT DILAKUKAN PENGGUNA].

Secara teknis, sistem terdiri dari:

```
[NAMA SISTEM]
├── Backend API         — Menerima request, menjalankan Business Flow, dan menyimpan data
├── Frontend Web        — [Deskripsi jika ada; hapus baris ini jika project API-only]
├── Database            — Penyimpanan data persisten seluruh sistem
├── Cache               — Penyimpanan sementara untuk data yang sering diakses
└── [Komponen lain]     — [Deskripsi]
```

### 3.2 Konteks Bisnis

Project ini beroperasi dalam domain [NAMA DOMAIN BISNIS — contoh: human resource management, e-commerce, logistics, education].

Sebelum project ini ada, [DESKRIPSI KONDISI BISNIS SEBELUMNYA — contoh: proses dilakukan manual menggunakan spreadsheet, atau menggunakan sistem lama yang tidak dapat discale]. Hal ini menyebabkan [DESKRIPSI MASALAH YANG TIMBUL]. Project ini dibangun untuk menggantikan atau melengkapi proses tersebut.

### 3.3 Posisi Sistem dalam Ekosistem

[Jelaskan apakah sistem ini berdiri sendiri atau merupakan bagian dari ekosistem yang lebih besar. Jika ada sistem lain yang berinteraksi dengannya, sebutkan dan jelaskan hubungannya.]

```
[Gambaran posisi sistem jika ada sistem lain di sekitarnya]

Contoh:
  [Sistem ERP Perusahaan] <--sync data--> [NAMA PROJECT] <--API--> [Aplikasi Mobile]
                                                |
                                          [Database]
```

---

## 4. Role Pengguna

### 4.1 Daftar Role

Sistem mengenal Role berikut. Role menentukan endpoint API mana yang dapat diakses dan data mana yang dapat dibaca atau diubah.

| Role | Deskripsi dalam Konteks Bisnis | Hak Akses Utama |
| --- | --- | --- |
| `[ROLE_1 — contoh: SUPER_ADMIN]` | [Siapa yang memiliki role ini dalam konteks organisasi] | [Akses penuh ke seluruh fitur dan data] |
| `[ROLE_2 — contoh: ADMIN]` | [Deskripsi] | [Dapat mengelola user dan konfigurasi, tidak dapat akses billing] |
| `[ROLE_3 — contoh: MANAGER]` | [Deskripsi] | [Dapat melihat dan menyetujui data tim yang dipimpinnya] |
| `[ROLE_4 — contoh: USER]` | [Deskripsi] | [Dapat mengelola data miliknya sendiri saja] |

### 4.2 Matriks Akses per Module

Tabel berikut menunjukkan akses tingkat tinggi tiap Role terhadap Module utama. Detail per endpoint ada di `docs/tsd/02-api-documentation.md`.

| Module | [ROLE_1] | [ROLE_2] | [ROLE_3] | [ROLE_4] |
| --- | --- | --- | --- | --- |
| [MODULE A] | Penuh | Penuh | Baca | Tidak Ada |
| [MODULE B] | Penuh | Penuh | Baca (Tim Sendiri) | Baca (Data Sendiri) |
| [MODULE C] | Penuh | Tidak Ada | Penuh | Tidak Ada |
| [MODULE D] | Penuh | Baca | Baca | Tidak Ada |

Keterangan: `Penuh` = Create, Read, Update, Delete sesuai scope. `Baca` = Read Only. `Tidak Ada` = Tidak dapat mengakses.

---

## 5. Module Utama

### 5.1 Daftar Module

Project dibagi ke dalam Module berikut. Setiap Module adalah unit domain bisnis yang mandiri dengan Controller, Service, Repository, dan Model tersendiri.

| Kode | Nama Module | Tanggung Jawab | Status |
| --- | --- | --- | --- |
| `MOD-01` | [NAMA MODULE — contoh: Authentication] | [Tanggung jawab dalam satu kalimat — contoh: Mengelola login, logout, dan manajemen token sesi] | Active |
| `MOD-02` | [NAMA MODULE] | [Tanggung jawab] | Active |
| `MOD-03` | [NAMA MODULE] | [Tanggung jawab] | Active |
| `MOD-04` | [NAMA MODULE] | [Tanggung jawab] | In Development |
| `MOD-05` | [NAMA MODULE] | [Tanggung jawab] | Planned |

### 5.2 Deskripsi Tiap Module

### MOD-01: [NAMA MODULE]

**Tanggung Jawab:**
[Jelaskan dalam 2-3 kalimat apa yang menjadi tanggung jawab Module ini. Apa yang dikelola? Apa yang dihasilkan? Apa yang tidak boleh dilakukan Module ini?]

**Entitas Data yang Dikelola:**
[Nama entitas utama yang disimpan oleh Module ini — contoh: User, Session]

**Bergantung Pada:**
[Module lain yang dipanggil oleh Module ini, jika ada. Contoh: Bergantung pada MOD-02 untuk validasi data karyawan]

**Yang Bergantung Padanya:**
[Module lain yang membutuhkan Module ini. Contoh: Seluruh Module bergantung pada Module ini untuk autentikasi]

**Business Flow Terkait:**
[Daftar flow bisnis yang diproses dalam Module ini — contoh: Login, Logout, Refresh Token]

---

### MOD-02: [NAMA MODULE]

**Tanggung Jawab:**
[Deskripsi]

**Entitas Data yang Dikelola:**
[Nama entitas]

**Bergantung Pada:**
[Dependency Module]

**Yang Bergantung Padanya:**
[Module yang butuh ini]

**Business Flow Terkait:**
[Flow dalam Module ini]

---

### MOD-03: [NAMA MODULE]

**Tanggung Jawab:**
[Deskripsi]

**Entitas Data yang Dikelola:**
[Nama entitas]

**Bergantung Pada:**
[Dependency Module]

**Yang Bergantung Padanya:**
[Module yang butuh ini]

**Business Flow Terkait:**
[Flow dalam Module ini]

---

## 6. Feature Utama

### 6.1 Daftar Feature

| Kode | Nama Feature | Module | Deskripsi Singkat | Status |
| --- | --- | --- | --- | --- |
| `FEAT-01` | [NAMA FEATURE] | [KODE MODULE] | [Satu kalimat yang menjelaskan apa yang bisa dilakukan pengguna dengan feature ini] | Active |
| `FEAT-02` | [NAMA FEATURE] | [KODE MODULE] | [Deskripsi] | Active |
| `FEAT-03` | [NAMA FEATURE] | [KODE MODULE] | [Deskripsi] | Active |
| `FEAT-04` | [NAMA FEATURE] | [KODE MODULE] | [Deskripsi] | Active |
| `FEAT-05` | [NAMA FEATURE] | [KODE MODULE] | [Deskripsi] | In Development |
| `FEAT-06` | [NAMA FEATURE] | [KODE MODULE] | [Deskripsi] | Planned |

### 6.2 Feature yang Secara Eksplisit Tidak Diimplementasi

Berikut adalah feature yang secara sadar tidak diimplementasi dalam versi ini, beserta alasannya:

| Feature | Alasan Tidak Diimplementasi | Alternatif |
| --- | --- | --- |
| [NAMA FEATURE] | [Alasan bisnis atau teknis yang jelas] | [Cara pengguna menyelesaikan kebutuhan yang sama tanpa feature ini] |
| [NAMA FEATURE] | [Alasan] | [Alternatif] |

---

## 7. High-Level Flow System

### 7.1 Cara Membaca Bagian Ini

Bagian ini menjelaskan bagaimana data dan permintaan bergerak melalui sistem secara umum. Ini bukan diagram arsitektur teknis — itu ada di `docs/psd/03-system-architecture.md`. Ini adalah gambaran alur dari sudut pandang pengguna dan proses bisnis.

### 7.2 Alur Umum Request ke Sistem

Setiap permintaan yang masuk ke sistem melewati jalur berikut:

```
Pengguna (Browser / Mobile / API Consumer)
              |
              | HTTP Request + Bearer Token
              v
API Gateway / Load Balancer
  (validasi awal, rate limiting, routing)
              |
              v
Middleware Pipeline
  (verifikasi token, pencatatan log, pembatasan rate per user)
              |
              v
Controller
  (terima request, validasi format input, delegasikan ke Service)
              |
              v
Service
  (jalankan Business Flow, atur transaksi, panggil Repository)
              |
              v
Repository
  (baca atau tulis data ke Database)
              |
              v
Database
              |
              v (data kembali ke atas melalui setiap lapisan)
Pengguna menerima Response
```

### 7.3 Alur Bisnis Utama

### Flow 1: [NAMA FLOW BISNIS UTAMA 1 — contoh: Onboarding Karyawan Baru]

**Konteks bisnis:** [Siapa yang melakukan ini dan dalam kondisi apa]

```
[AKTOR] melakukan [AKSI]
              |
              v
[LANGKAH BISNIS 1 — contoh: Admin mengisi form data karyawan]
              |
              v
[LANGKAH BISNIS 2 — contoh: Sistem memvalidasi data dan membuat akun]
              |
              v
[LANGKAH BISNIS 3 — contoh: Sistem mengirim email undangan login ke karyawan]
              |
              v
[HASIL AKHIR — contoh: Karyawan dapat login dan mengakses sistem]
```

**Module yang terlibat:** [Daftar Module — contoh: MOD-01 Authentication, MOD-02 Employee]
**Untuk detail teknis:** `docs/tsd/03-business-flow/[nama-module]/[nama-flow].md`

---

### Flow 2: [NAMA FLOW BISNIS UTAMA 2]

**Konteks bisnis:** [Siapa dan dalam kondisi apa]

```
[AKTOR]
  |
  v
[LANGKAH 1]
  |
  v
[LANGKAH 2]
  |
  v
[HASIL AKHIR]
```

**Module yang terlibat:** [Daftar Module]
**Untuk detail teknis:** `docs/tsd/03-business-flow/[nama-module]/[nama-flow].md`

---

### Flow 3: [NAMA FLOW BISNIS UTAMA 3]

**Konteks bisnis:** [Siapa dan dalam kondisi apa]

```
[AKTOR]
  |
  v
[LANGKAH 1]
  |
  v
[LANGKAH 2]
  |
  v
[HASIL AKHIR]
```

**Module yang terlibat:** [Daftar Module]
**Untuk detail teknis:** `docs/tsd/03-business-flow/[nama-module]/[nama-flow].md`

---

## 8. Dependency Utama

### 8.1 Tech Stack

| Lapisan | Teknologi | Versi | Peran dalam Project |
| --- | --- | --- | --- |
| Runtime | [contoh: Node.js] | [VERSI] | Menjalankan kode backend |
| Framework | [contoh: NestJS] | [VERSI] | Struktur aplikasi: routing, DI, Module |
| Database | [contoh: PostgreSQL] | [VERSI] | Penyimpanan data persisten |
| Cache | [contoh: Redis] | [VERSI] | Session, rate limit counter, cache query |
| ORM | [contoh: TypeORM] | [VERSI] | Abstraksi akses Database |
| Queue | [contoh: Bull + Redis] | [VERSI] | Pemrosesan job asinkron |
| Frontend | [contoh: Next.js] | [VERSI] | Antarmuka pengguna web |
| Package Manager | [contoh: pnpm] | [VERSI] | Manajemen dependency |
| Container | [contoh: Docker] | [VERSI] | Isolasi Environment untuk development |

### 8.2 Library Kritikal

Library yang jika dihapus atau diganti akan membutuhkan perubahan signifikan di seluruh codebase:

| Library | Versi | Fungsi dalam Project | Digunakan Di |
| --- | --- | --- | --- |
| [NAMA LIBRARY] | [VERSI] | [Fungsi konkret — contoh: Validasi DTO input di seluruh Controller] | Seluruh Module |
| [NAMA LIBRARY] | [VERSI] | [Fungsi] | [Lokasi penggunaan] |
| [NAMA LIBRARY] | [VERSI] | [Fungsi] | [Lokasi penggunaan] |
| [NAMA LIBRARY] | [VERSI] | [Fungsi] | [Lokasi penggunaan] |

### 8.3 Keputusan Teknologi yang Perlu Dipahami

Beberapa keputusan pemilihan teknologi memiliki implikasi signifikan pada cara project ini bekerja:

| Teknologi | Mengapa Dipilih | Konsekuensi yang Perlu Dipahami |
| --- | --- | --- |
| [TEKNOLOGI] | [Alasan pemilihan yang konkret, bukan hanya “populer”] | [Dampak pada cara developer bekerja — contoh: Semua query Database harus melalui Repository karena menggunakan Active Record pattern] |
| [TEKNOLOGI] | [Alasan] | [Konsekuensi] |
| [TEKNOLOGI] | [Alasan] | [Konsekuensi] |

---

## 9. Integrasi Utama

### 9.1 Daftar Integrasi

Project ini berkomunikasi dengan layanan eksternal berikut:

| Kode | Layanan Eksternal | Tipe Komunikasi | Tujuan | Kritikal |
| --- | --- | --- | --- | --- |
| `EXT-01` | [NAMA LAYANAN — contoh: SendGrid] | [REST API / Webhook / SDK] | [Tujuan konkret — contoh: Pengiriman email transaksional] | Ya / Tidak |
| `EXT-02` | [NAMA LAYANAN] | [Tipe] | [Tujuan] | Ya / Tidak |
| `EXT-03` | [NAMA LAYANAN] | [Tipe] | [Tujuan] | Ya / Tidak |

Keterangan `Kritikal: Ya` berarti kegagalan layanan ini menyebabkan Business Flow utama sistem tidak dapat berjalan.

### 9.2 Deskripsi Tiap Integrasi

### EXT-01: [NAMA LAYANAN]

**Tujuan:** [Mengapa project ini membutuhkan layanan ini — dalam konteks bisnis]

**Arah komunikasi:**
- Project ini memanggil layanan ini ketika: [kondisi/event — contoh: pengguna baru berhasil dibuat, password di-reset]
- Layanan ini memanggil project ini ketika: [kondisi — atau tulis “Tidak ada; komunikasi satu arah”]

**Module yang menggunakan:** [Daftar Module — contoh: MOD-01 Authentication, MOD-02 User Management]

**Behavior saat layanan tidak tersedia:** [Contoh: Email tidak terkirim, namun proses bisnis utama tetap berlanjut. Job dijadwalkan ulang di Queue untuk dicoba kembali 3x sebelum dianggap gagal]

**Detail teknis:** `docs/tsd/integration/[nama-layanan].md`

---

### EXT-02: [NAMA LAYANAN]

**Tujuan:** [Tujuan dalam konteks bisnis]

**Arah komunikasi:**
- Project ini memanggil ketika: [kondisi]
- Layanan memanggil project ketika: [kondisi atau “Tidak ada”]

**Module yang menggunakan:** [Daftar Module]

**Behavior saat tidak tersedia:** [Deskripsi]

**Detail teknis:** `docs/tsd/integration/[nama-layanan].md`

---

## 10. Environment Overview

### 10.1 Environment yang Tersedia

| Environment | Tujuan | URL | Siapa yang Menggunakannya |
| --- | --- | --- | --- |
| `local` | Development di mesin developer | `http://localhost:[PORT]` | Developer saat mengerjakan fitur |
| `development` | Shared server untuk integrasi fitur tim | `https://[URL-DEV]` | Tim developer saat integrasi antar fitur |
| `staging` | Server pre-production, identik dengan production | `https://[URL-STAGING]` | QA Engineer, UAT, demo internal |
| `production` | Server live yang melayani pengguna nyata | `https://[URL-PRODUCTION]` | Pengguna akhir |

### 10.2 Perbedaan Konfigurasi per Environment

| Aspek | local | development | staging | production |
| --- | --- | --- | --- | --- |
| Database | Docker container lokal | Instance dedicated dev | Instance dedicated staging | Instance dedicated prod (HA) |
| Integrasi Eksternal | Mode sandbox / mock | Mode sandbox | Mode sandbox | Mode production |
| Log Level | DEBUG | DEBUG | INFO | WARNING |
| Data | Seed data dummy | Data non-stabil, dapat direset | Data anonim dari prod | Data riil pengguna |
| Deployment | Manual oleh developer | Otomatis saat merge ke `develop` | Semi-otomatis, perlu approval | Manual dengan approval Tech Lead |

### 10.3 Cara Menjalankan Project Secara Lokal

Untuk menjalankan project di environment `local`, ikuti panduan lengkap di:

**`docs/psd/02-setup-guide.md`**

---

## 11. Repository Overview

### 11.1 Struktur Direktori

```
[NAMA-REPOSITORY]/
├── src/                           # Seluruh source code aplikasi
│   ├── modules/                   # Module bisnis (satu direktori per Module)
│   │   ├── [modul-a]/             # Module [NAMA]
│   │   ├── [modul-b]/             # Module [NAMA]
│   │   └── [modul-n]/             # Module [NAMA]
│   ├── shared/                    # Komponen yang digunakan lintas Module
│   │   ├── middleware/            # Middleware global (autentikasi, logging, rate limit)
│   │   ├── guards/                # Guard otorisasi berbasis Role
│   │   ├── filters/               # Global exception handler
│   │   ├── utils/                 # Fungsi utilitas tanpa state
│   │   └── constants/             # Konstanta global sistem
│   └── config/                    # Konfigurasi koneksi Database, Cache, dan Environment
│
├── test/                          # Test suite
│   ├── unit/                      # Unit test per Module
│   ├── integration/               # Integration test
│   └── e2e/                       # End-to-end test
│
├── docs/                          # Seluruh dokumentasi project
│   ├── psd/                       # Product/System Documentation (dokumen ini)
│   ├── tsd/                       # Technical/Service Documentation
│   └── ld/                        # Log Documentation
│
├── scripts/                       # Script utilitas (migration, seeder, tooling)
├── .env.example                   # Template konfigurasi Environment
├── docker-compose.yml             # Konfigurasi service untuk development lokal
└── README.md                      # Titik masuk repository
```

### 11.2 Struktur Internal Tiap Module

Setiap Module di dalam `src/modules/` mengikuti struktur yang seragam:

```
[nama-module]/
├── [nama-module].module.[ext]       # Deklarasi dan registrasi Module
├── [nama-module].controller.[ext]   # Routing HTTP dan response
├── [nama-module].service.[ext]      # Business Flow dan aturan bisnis
├── [nama-module].repository.[ext]   # Akses Database
├── [nama-module].model.[ext]        # Struktur data entitas
├── dto/                             # Kontrak dan validasi input
└── tests/                           # Unit test Module ini
```

Konsistensi struktur ini berlaku untuk seluruh Module tanpa pengecualian. Penyimpangan harus didokumentasikan dengan alasan yang jelas.

### 11.3 Branch Strategy

| Branch | Tujuan | Deploy Ke |
| --- | --- | --- |
| `main` | Kode yang siap production | Production (manual) |
| `develop` | Integrasi fitur yang sedang dikerjakan | Development (otomatis) |
| `feature/[nama]` | Pengerjaan satu fitur atau task | Tidak ada |
| `fix/[nama]` | Perbaikan bug | Tidak ada |
| `hotfix/[nama]` | Perbaikan darurat di production | Production (manual, approval ketat) |

---

## 12. Struktur Dokumentasi

### 12.1 Hierarki Dokumentasi

```
docs/
├── psd/                             # Product/System Documentation (high-level)
│   ├── 00-fundamental-psd.md        # Standar dan panduan penulisan dokumentasi
│   ├── 01-project-overview.md       # Dokumen ini: gambaran project
│   ├── 02-setup-guide.md            # Cara menjalankan project
│   ├── 03-system-architecture.md    # Arsitektur dan struktur sistem
│   ├── 04-coding-standard.md        # Standar penulisan kode
│   └── 05-onboarding-guide.md       # Panduan developer baru
│
├── tsd/                             # Technical/Service Documentation (low-level)
│   ├── 00-fundamental-tsd.md        # Standar dan panduan penulisan TSD
│   ├── 01-database-design.md        # Desain Database, tabel, relasi
│   ├── 02-api-documentation.md      # Kontrak seluruh endpoint API
│   └── 03-business-flow/            # Dokumentasi flow bisnis per Module
│       ├── README.md                # Panduan menulis business flow documentation
│       ├── authentication/          # Flow Module Authentication
│       └── [nama-module]/           # Flow Module lainnya
└── ld/ 
    ├── 01-troubleshooting.md        # Dokumentasi troubleshooting project
    └── 02-changelog.md              # Dokumentasi perubahan semua dokumen docs
```

### 12.2 Cara Menggunakan Dokumentasi

| Pertanyaan | Dokumen yang Menjawab |
| --- | --- |
| Project ini untuk apa? | `docs/psd/01-project-overview.md` (dokumen ini) |
| Bagaimana cara menjalankan project? | `docs/psd/02-setup-guide.md` |
| Bagaimana arsitektur sistem bekerja? | `docs/psd/03-system-architecture.md` |
| Apa standar penulisan kode yang berlaku? | `docs/psd/04-coding-standard.md` |
| Apa saja tabel Database dan strukturnya? | `docs/tsd/01-database-design.md` |
| Apa kontrak endpoint API tertentu? | `docs/tsd/02-api-documentation.md` |
| Bagaimana alur teknis Business Flow X? | `docs/tsd/03-business-flow/[module]/[flow].md` |
| Bagaimana tata cara deployment proyek ini? | `docs/tsd/04-deployment-guide.md` |

### 12.3 Siapa yang Bertanggung Jawab atas Dokumentasi

| Dokumen | Penulis | Reviewer |
| --- | --- | --- |
| PSD (semua file) | [Developer yang membuat perubahan] | [Tech Lead] |
| TSD Database Design | [Developer yang membuat migration] | [Tech Lead] |
| TSD API Documentation | [Developer yang mengimplementasi endpoint] | [Tech Lead] |
| TSD Business Flow | [Developer yang mengimplementasi flow] | [Tech Lead + QA] |

---

## 13. Batasan Sistem

### 13.1 Batasan Fungsional

Batasan fungsional adalah hal yang secara sengaja tidak dilakukan sistem, meskipun mungkin terlihat relevan:

| No | Batasan | Alasan |
| --- | --- | --- |
| 1 | [BATASAN FUNGSIONAL — contoh: Sistem tidak menangani penggajian] | [Alasan bisnis — contoh: Penggajian ditangani oleh sistem ERP terpisah yang sudah ada] |
| 2 | [BATASAN] | [ALASAN] |
| 3 | [BATASAN] | [ALASAN] |

### 13.2 Batasan Non-Fungsional

Batasan teknis sistem yang perlu dipahami developer dan integrator:

| Aspek | Batasan | Keterangan |
| --- | --- | --- |
| Throughput | [N] request/detik | Berdasarkan kapasitas infrastruktur deployment saat ini |
| Ukuran file upload | Maksimal [N] MB | Dibatasi di level Middleware dan konfigurasi server |
| Retensi data | [N] tahun | Kebijakan retensi data yang berlaku |
| Bahasa antarmuka | [BAHASA] | Sistem saat ini hanya mendukung bahasa ini |
| Browser yang didukung | [DAFTAR] | Versi lebih lama tidak dijamin kompatibel |
| Timezone | [TIMEZONE] | Seluruh timestamp menggunakan timezone ini |

---

## 14. Asumsi Sistem

### 14.1 Asumsi Bisnis

Asumsi bisnis adalah kondisi yang dianggap benar saat sistem dirancang. Jika asumsi ini berubah, bagian sistem mungkin perlu direvisi.

| No | Asumsi | Bagian Sistem yang Terdampak jika Asumsi Berubah |
| --- | --- | --- |
| 1 | [ASUMSI BISNIS — contoh: Satu karyawan hanya bekerja di satu departemen pada satu waktu] | [DAMPAK — contoh: Skema Database tabel employees, validasi di EmployeeService] |
| 2 | [ASUMSI] | [DAMPAK] |
| 3 | [ASUMSI] | [DAMPAK] |

### 14.2 Asumsi Teknis

| No | Asumsi | Dampak jika Berubah |
| --- | --- | --- |
| 1 | [ASUMSI TEKNIS — contoh: Server berjalan di lingkungan Linux berbasis Debian] | [DAMPAK — contoh: Script shell dalam deployment mungkin tidak kompatibel] |
| 2 | [ASUMSI — contoh: Koneksi Database selalu tersedia; tidak ada offline mode] | [DAMPAK — contoh: Seluruh operasi yang butuh data akan gagal; tidak ada local cache fallback] |
| 3 | [ASUMSI] | [DAMPAK] |

### 14.3 Asumsi Operasional

| No | Asumsi | Dampak jika Berubah |
| --- | --- | --- |
| 1 | [ASUMSI OPERASIONAL — contoh: Tim on-call tersedia 24/7 untuk menangani insiden production] | [DAMPAK] |
| 2 | [ASUMSI — contoh: Backup Database dilakukan setiap hari oleh tim infrastruktur] | [DAMPAK — contoh: Recovery point objective (RPO) berubah jika frekuensi backup dikurangi] |

---

## 15. Glossary Project

Istilah spesifik yang digunakan dalam project ini dan memiliki makna khusus dalam konteks domain bisnis ini. Istilah teknis umum (Controller, Service, Repository) mengikuti definisi standar di `docs/psd/00-fundamental-psd.md`.

| Istilah | Definisi | Konteks Penggunaan |
| --- | --- | --- |
| [ISTILAH DOMAIN 1 — contoh: Karyawan Tetap] | [Definisi dalam konteks bisnis project ini] | [Digunakan di Module mana, tabel mana, field mana] |
| [ISTILAH DOMAIN 2 — contoh: Shift Kerja] | [Definisi] | [Konteks] |
| [ISTILAH DOMAIN 3] | [Definisi] | [Konteks] |
| [ISTILAH DOMAIN 4] | [Definisi] | [Konteks] |
| [ISTILAH DOMAIN 5] | [Definisi] | [Konteks] |
| [ISTILAH DOMAIN 6] | [Definisi] | [Konteks] |
| [ISTILAH DOMAIN 7] | [Definisi] | [Konteks] |
| [ISTILAH DOMAIN 8] | [Definisi] | [Konteks] |