# 03 - System Architecture

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Architecture
**Terakhir Diperbarui:** [TANGGAL]
**Pemilik Dokumen:** [NAMA TIM / ROLE]

---

## Dokumen Terkait

| Dokumen | Path | Keterangan |
| --- | --- | --- |
| Project Overview | `docs/psd/01-project-overview.md` | Konteks bisnis, Module, dan tech stack yang dijelaskan di sini |
| Setup Guide | `docs/psd/02-setup-guide.md` | Cara menjalankan komponen yang digambarkan di sini |
| Coding Standard | `docs/psd/04-coding-standard.md` | Standar penulisan kode yang mengikuti struktur arsitektur ini |

---

## Cara Membaca Dokumen Ini

Dokumen ini menjelaskan **bagaimana project ini dibangun secara struktural** — komponen apa yang ada, bagaimana komponen itu terhubung, dan mengapa disusun seperti ini. Ini bukan penjelasan teori arsitektur software secara umum.

Baca dokumen ini setelah `docs/psd/01-project-overview.md`. Setelah membaca dokumen ini, developer diharapkan dapat menjawab:

- Komponen apa saja yang ada dan apa peran masing-masing dalam project ini?
- Ketika sebuah request masuk, komponen mana yang memprosesnya dan dalam urutan apa?
- Module mana saling berkomunikasi?
- Data disimpan di mana dan bagaimana cara mengaksesnya?
- Bagaimana autentikasi bekerja di project ini?

---

## Daftar Isi

1. [Architecture Overview](#1-architecture-overview)
2. [Struktur Komponen Sistem](#2-struktur-komponen-sistem)
3. [Backend Overview](#3-backend-overview)
4. [Frontend Overview](#4-frontend-overview)
5. [Database Overview](#5-database-overview)
6. [Authentication Overview](#6-authentication-overview)
7. [Request Flow Overview](#7-request-flow-overview)
8. [Module Interaction](#8-module-interaction)
9. [Dependency Overview](#9-dependency-overview)
10. [Integration Overview](#10-integration-overview)
11. [Scheduler dan Queue Overview](#11-scheduler-dan-queue-overview)
12. [Deployment Environment Overview](#12-deployment-environment-overview)
13. [Logging Overview](#13-logging-overview)
14. [Scalability Consideration](#14-scalability-consideration)
15. [Maintainability Consideration](#15-maintainability-consideration)

---

## 1. Architecture Overview

### 1.1 Bentuk Sistem

[NAMA PROJECT] dibangun sebagai **[NAMA POLA — contoh: Modular Monolith]**. Seluruh kode backend berjalan dalam satu deployment unit, namun di dalam dibagi ke dalam Module-Module yang memiliki batasan dan tanggung jawab yang tegas.

Keputusan menggunakan pola ini diambil karena:

- [ALASAN 1 — jelaskan dalam konteks project ini, contoh: Tim terdiri dari [N] developer; overhead infrastruktur microservices tidak proporsional untuk tahap ini]
- [ALASAN 2 — contoh: Domain bisnis masih aktif berkembang; batasan antar layanan belum cukup stabil untuk dipisah menjadi service mandiri]
- [ALASAN 3 — contoh: Seluruh Module memiliki siklus rilis yang sama; tidak ada kebutuhan untuk di-deploy secara independen]

### 1.2 Gambaran Komponen yang Berjalan

Ketika project ini dijalankan, inilah seluruh komponen yang aktif:

```
+---------------------------------------------------------------+
|                     CLIENT                                    |
|   Browser / Mobile App / API Consumer Eksternal               |
+-------------------------------+-------------------------------+
                                |
                          HTTP / HTTPS
                                |
+-------------------------------v-------------------------------+
|                     BACKEND API                               |
|                   [NAMA PROJECT] Backend                      |
|   Framework: [FRAMEWORK — contoh: NestJS / Express]           |
|   Runtime   : [RUNTIME — contoh: Node.js v[VERSI]]            |
|                                                               |
|   +-------------------+   +-------------------+              |
|   |  Middleware Stack  |   |  Module-Module     |              |
|   |  - Auth            |   |  - [MOD-01]        |              |
|   |  - Logger          |   |  - [MOD-02]        |              |
|   |  - Rate Limit      |   |  - [MOD-03]        |              |
|   +-------------------+   +-------------------+              |
+--------+----------+-----------+----------+--------------------+
         |          |           |          |
         v          v           v          v
+--------+--+  +----+----+  +---+----+  +-+----------+
|  DATABASE  |  |  CACHE  |  |  QUEUE |  | SCHEDULER  |
| [DB NAME]  |  | [CACHE] |  | [QUEUE]|  | (In-process|
| [VERSI]    |  | [VERSI] |  | [VERSI]|  |  atau      |
|            |  |         |  |        |  |  terpisah) |
+------------+  +---------+  +--------+  +------------+
```

### 1.3 Keputusan Teknis Fundamental

Berikut keputusan teknis yang memengaruhi cara seluruh project bekerja. Developer baru perlu memahami ini sebelum menyentuh kode.

| Keputusan | Yang Dipilih | Konsekuensi yang Perlu Dipahami |
| --- | --- | --- |
| Pola arsitektur | [Modular Monolith / dll] | [Jelaskan dampaknya pada cara developer bekerja — contoh: Semua Module di-deploy sekaligus; tidak ada deployment parsial per Module] |
| Akses Database | [ORM — contoh: TypeORM / Prisma / Sequelize] | [Dampak — contoh: Semua query Database harus melalui Repository; tidak boleh query langsung dari Service atau Controller] |
| Autentikasi | [Mekanisme — contoh: JWT stateless] | [Dampak — contoh: Backend tidak menyimpan session di memori; backend dapat di-scale horizontal tanpa shared session store] |
| Komunikasi asinkron | [Queue — contoh: Bull + Redis] | [Dampak — contoh: Operasi yang tidak perlu hasil langsung (email, notifikasi) selalu dimasukkan ke Queue, bukan diproses langsung di request] |

---

## 2. Struktur Komponen Sistem

### 2.1 Lapisan Sistem

Project ini menggunakan struktur berlapis di mana setiap lapisan memiliki satu tanggung jawab. Komunikasi hanya boleh bergerak satu arah: dari lapisan atas ke bawah.

```
+---------------------------------------------+
|              LAPISAN CLIENT                 |
|  Browser · Mobile App · API Consumer        |
+---------------------------------------------+
                     |
                HTTP Request
                     |
+---------------------------------------------+
|          LAPISAN MIDDLEWARE                  |
|  Autentikasi · Logging · Rate Limiting       |
|  (Diproses sebelum request sampai ke Module)|
+---------------------------------------------+
                     |
+---------------------------------------------+
|           LAPISAN CONTROLLER                 |
|  Menerima request · Validasi input           |
|  Delegasikan ke Service yang tepat           |
+---------------------------------------------+
                     |
+---------------------------------------------+
|             LAPISAN SERVICE                  |
|  Jalankan Business Flow · Aturan bisnis      |
|  Orkestrasi Repository · Kelola transaksi    |
+---------------------------------------------+
                     |
+---------------------------------------------+
|           LAPISAN REPOSITORY                 |
|  Satu-satunya jalur ke Database              |
|  Tidak ada Business Flow di sini             |
+---------------------------------------------+
                     |
+---------------------------------------------+
|             LAPISAN DATA                     |
|  Database · Cache · Queue · Object Storage   |
+---------------------------------------------+
```

### 2.2 Aturan Komunikasi Antar Lapisan

Aturan ini berlaku untuk semua Module tanpa pengecualian:

| Dari | Boleh Memanggil | Tidak Boleh Memanggil |
| --- | --- | --- |
| Controller | Service | Repository, Database langsung |
| Service | Repository, Service lain (lintas Module) | Controller, Database langsung |
| Repository | Database (via ORM) | Service, Controller |
| Model | — | Tidak memanggil apapun; hanya struktur data |

Jika ditemukan kode yang melanggar aturan ini, itu adalah bug arsitektur yang harus diperbaiki, bukan pengecualian.

### 2.3 Komponen Shared

Komponen berikut digunakan oleh seluruh Module dan tidak milik satu Module tertentu:

| Komponen | Lokasi | Fungsi dalam Project |
| --- | --- | --- |
| Auth Middleware | `src/shared/middleware/` | Memverifikasi JWT token di setiap request yang memerlukan autentikasi |
| Logger Middleware | `src/shared/middleware/` | Mencatat setiap request masuk dan response keluar |
| Rate Limit Middleware | `src/shared/middleware/` | Membatasi jumlah request per user atau per IP |
| Roles Guard | `src/shared/guards/` | Memeriksa Role user sebelum request diproses Controller |
| Exception Filter | `src/shared/filters/` | Menangkap semua error dan mengubahnya menjadi format response yang konsisten |
| Utils | `src/shared/utils/` | Fungsi bantu yang digunakan lintas Module (format tanggal, hash, dll) |
| Constants | `src/shared/constants/` | Nilai konstanta yang digunakan di banyak tempat (kode error, default value) |

---

## 3. Backend Overview

### 3.1 Struktur Direktori Backend

[Jelaskan dengan penggambaran struktur proyek dari sisi Backend]

```
src/
├── main.[ext]                     # Entry point: inisialisasi aplikasi dan start server
├── app.module.[ext]               # Root Module: mendaftarkan semua Module bisnis
│
├── modules/                       # Semua Module bisnis berada di sini
│   ├── [modul-authentication]/    # Contoh: autentikasi pengguna
│   ├── [modul-user]/              # Contoh: manajemen data pengguna
│   └── [modul-lainnya]/           # Module bisnis lain sesuai domain
│
├── shared/                        # Komponen yang dipakai lintas Module
│   ├── middleware/                # Auth, Logger, Rate Limit
│   ├── guards/                    # Roles Guard
│   ├── filters/                   # Global Exception Filter
│   ├── utils/                     # Fungsi bantu tanpa side effect
│   └── constants/                 # Konstanta global project
│
└── config/                        # Konfigurasi koneksi Database, Cache, dan environment
    ├── database.config.[ext]
    ├── cache.config.[ext]
    └── app.config.[ext]
```

### 3.2 Struktur Internal Setiap Module

Setiap Module mengikuti struktur yang seragam. Konsistensi ini memungkinkan developer memahami satu Module dan langsung produktif di Module lain.

```
[nama-module]/
├── [nama-module].module.[ext]       # Mendaftarkan Controller, Service, Repository ke framework
├── [nama-module].controller.[ext]   # Mendefinisikan route dan menangani HTTP request
├── [nama-module].service.[ext]      # Seluruh Business Flow dan aturan bisnis Module ini
├── [nama-module].repository.[ext]   # Semua query ke Database untuk entitas Module ini
├── [nama-module].model.[ext]        # Definisi struktur data dan mapping ke tabel Database
├── dto/                             # Data Transfer Object: kontrak dan validasi input
│   ├── create-[nama-module].dto.[ext]
│   └── update-[nama-module].dto.[ext]
└── tests/                           # Test untuk Service dan Controller Module ini
    ├── [nama-module].service.spec.[ext]
    └── [nama-module].controller.spec.[ext]
```

### 3.3 Tanggung Jawab Setiap File dalam Module

**Controller** — Hanya menangani hal yang berkaitan dengan HTTP. Ekstrak data dari request, validasi format input lewat DTO, panggil Service, kembalikan response. Tidak ada aturan bisnis di Controller.

**Service** — Tempat Business Flow project ini berada. Memeriksa aturan bisnis, memanggil Repository, mengelola transaksi Database, memutuskan kapan harus publish ke Queue.

**Repository** — Satu-satunya tempat query Database ditulis. Tidak ada Business Flow di Repository. Service memanggil Repository; Repository tidak memanggil Service.

**Model** — Mendefinisikan struktur tabel Database dan relasinya. Model tidak memiliki method bisnis; hanya berisi definisi properti dan dekorator ORM.

**DTO** — Mendefinisikan kontrak input yang diterima endpoint. Berisi aturan validasi format (email valid, field wajib ada, panjang maksimum). Validasi bisnis (email unik, stok tersedia) tetap di Service.

---

## 4. Frontend Overview

`[KONDISIONAL]` Bagian ini hanya relevan jika project memiliki komponen frontend. Hapus bagian ini jika project adalah API-only.

### 4.1 Deskripsi Frontend

Frontend project ini dibangun dengan **[FRAMEWORK — contoh: Next.js / React / Vue]** dan berjalan di browser. Frontend berkomunikasi dengan backend **hanya** melalui REST API yang terdokumentasi di `docs/tsd/02-api-documentation.md`. Frontend tidak pernah mengakses Database secara langsung.

### 4.2 Cara Frontend Berinteraksi dengan Backend

```
Aksi pengguna di browser
        |
        v
Komponen UI (Page / View)
        |
        v
API Client Layer
  - Tambahkan token ke header: Authorization: Bearer [TOKEN]
  - Format request sesuai kontrak API
        |
        | HTTP Request
        v
Backend API
        |
        | HTTP Response
        v
State Management (simpan data ke state global jika perlu)
        |
        v
Komponen UI update tampilannya
```

### 4.3 Konfigurasi URL Backend

Frontend tahu ke mana harus mengirim request melalui environment variable. Nilai ini dikonfigurasi di file `.env` direktori frontend:

```bash
[VARIABLE — contoh: VITE_API_BASE_URL atau NEXT_PUBLIC_API_URL]=[URL BACKEND]
```

Nilai per environment ada di `docs/psd/02-setup-guide.md` Bagian 13.

---

## 5. Database Overview

### 5.1 Database yang Digunakan

Project ini menggunakan **[NAMA DATABASE — contoh: PostgreSQL v15]** sebagai penyimpanan data persisten. Seluruh data bisnis yang perlu bertahan setelah restart tersimpan di sini.

Project ini juga menggunakan **[NAMA CACHE — contoh: Redis]** untuk menyimpan data sementara yang membutuhkan akses cepat: [sebutkan apa yang disimpan di cache — contoh: session token aktif, counter rate limiting, hasil query yang sering dibaca].

### 5.2 Bagaimana Kode Mengakses Database

Tidak ada query Database yang boleh ditulis di luar lapisan Repository. Alur akses data selalu mengikuti jalur berikut:

```
Service
  |
  | Memanggil method Repository
  v
Repository
  |
  | Query via ORM ([NAMA ORM])
  v
Connection Pool ([MIN]-[MAX] koneksi)
  |
  v
Database Server
  |
  v
Data dikembalikan sebagai Model
```

Penggunaan raw query diperbolehkan hanya untuk kasus tertentu yang ORM tidak dapat menangani secara efisien, dan harus menggunakan parameterized query tanpa pengecualian.

### 5.3 Tabel Utama dan Module yang Mengelolanya

| Tabel | Dikelola oleh Module | Fungsi |
| --- | --- | --- |
| `[NAMA TABEL — contoh: users]` | `[Nama Module]` | [Fungsi bisnis tabel ini] |
| `[NAMA TABEL — contoh: sessions]` | `[Nama Module]` | [Fungsi] |
| `[NAMA TABEL]` | `[Nama Module]` | [Fungsi] |
| `[NAMA TABEL]` | `[Nama Module]` | [Fungsi] |
| `[NAMA TABEL]` | `[Nama Module]` | [Fungsi] |

Detail lengkap struktur tabel, relasi, dan index ada di `docs/tsd/01-database-design.md`.

### 5.4 Aturan Kepemilikan Data

Setiap tabel Database hanya boleh ditulis oleh Repository dari Module yang memilikinya. Module lain yang membutuhkan data dari tabel tersebut harus memanggil Service pemiliknya, bukan langsung mengakses Repository-nya.

```
Contoh:
  OrderService membutuhkan data user
  → BENAR  : OrderService memanggil UserService.findById()
  → SALAH  : OrderService memanggil UserRepository.findById() langsung
```

---

## 6. Authentication Overview

### 6.1 Mekanisme Autentikasi

Project ini menggunakan **[MEKANISME — contoh: JWT (JSON Web Token) dengan strategi dual token]**:

- **Access Token** — Token berumur pendek ([DURASI — contoh: 15 menit]) yang dikirimkan di setiap request terautentikasi melalui header `Authorization: Bearer [TOKEN]`.
- **Refresh Token** — Token berumur panjang ([DURASI — contoh: 7 hari]) yang disimpan di Database. Digunakan untuk mendapatkan access token baru tanpa login ulang.

### 6.2 Alur Login hingga Request Terautentikasi

```
ALUR LOGIN:

Pengguna mengirim email + password
        |
        v
AuthService memverifikasi kredensial ke Database
        |
        v
Jika valid:
  - Generate access token (JWT, exp: [DURASI])
  - Generate refresh token (random string)
  - Simpan refresh token ke tabel [NAMA TABEL — contoh: sessions]
        |
        v
Client menerima kedua token
Client menyimpan token di penyimpanan yang aman

---

ALUR REQUEST SETELAH LOGIN:

Client mengirim request
  + Header: Authorization: Bearer [ACCESS_TOKEN]
        |
        v
Auth Middleware memeriksa token
  - Apakah token ada?
  - Apakah signature valid?
  - Apakah token belum expire?
        |
        +-- Tidak valid --> Response 401 UNAUTHORIZED
        |
        +-- Valid --> Sisipkan data user ke request context
                          (userId, email, role)
        |
        v
Roles Guard memeriksa Role user
  (hanya jika endpoint memerlukan Role tertentu)
        |
        +-- Role tidak cukup --> Response 403 FORBIDDEN
        |
        +-- Role sesuai --> Lanjut ke Controller
```

### 6.3 Alur Refresh Token

```
Access token kadaluwarsa
        |
Client mengirim POST /api/v[N]/auth/refresh
  + Body: { refreshToken }
        |
        v
AuthService memvalidasi refresh token ke Database
  - Apakah refresh token ada di tabel [sessions]?
  - Apakah belum kadaluwarsa?
        |
        +-- Tidak valid --> Response 401; client harus login ulang
        |
        +-- Valid --> Generate access token baru
                      Kembalikan ke client
```

### 6.4 Di Mana Token Disimpan

| Token | Disimpan Di | Mengapa |
| --- | --- | --- |
| Access Token | Memori client (tidak persisten) | Token berumur pendek; tidak perlu persisten |
| Refresh Token | Database — tabel `[NAMA TABEL]` | Perlu bisa direvoke; harus persisten |

Access token tidak disimpan di Database. Backend memverifikasi access token dengan memvalidasi signature JWT menggunakan `JWT_SECRET`, tanpa perlu query ke Database. Ini yang membuat backend dapat di-scale horizontal.

---

## 7. Request Flow Overview

### 7.1 Alur Request Normal (Happy Path)

Berikut adalah jalur lengkap yang ditempuh sebuah request dari masuk hingga response dikembalikan:

```
CLIENT
  | POST /api/v1/[resource]
  | Header: Authorization: Bearer [TOKEN]
  | Body: { ...data }
  v
HTTP SERVER
  | Terima koneksi, parse HTTP headers dan body
  v
ROUTER
  | Cocokkan URL + method ke Controller yang terdaftar
  v
MIDDLEWARE PIPELINE (dijalankan berurutan)
  | 1. Logger Middleware
  |    → Catat: method, path, IP, timestamp, generate Request ID
  | 2. Auth Middleware
  |    → Verifikasi token, sisipkan user context ke request
  |    → Jika gagal: kembalikan 401
  | 3. Rate Limit Middleware
  |    → Periksa batas request user/IP
  |    → Jika melebihi: kembalikan 429
  v
ROLES GUARD
  | Periksa apakah Role user memenuhi syarat untuk endpoint ini
  | Jika tidak: kembalikan 403
  v
CONTROLLER
  | Ekstrak parameter dari request (body, params, query)
  | Validasi format input via DTO
  | Jika validasi gagal: kembalikan 422
  | Panggil method Service yang sesuai
  v
SERVICE
  | Periksa aturan bisnis
  | Panggil Repository untuk baca/tulis data
  | Kelola transaksi Database jika perlu
  | Publish ke Queue jika ada side effect asinkron
  v
REPOSITORY
  | Eksekusi query ke Database via ORM
  v
DATABASE
  | Kembalikan hasil query
  v
REPOSITORY → SERVICE → CONTROLLER
  | Data mengalir kembali ke atas
  v
RESPONSE INTERCEPTOR
  | Format response ke struktur standar
  v
LOGGER MIDDLEWARE (response)
  | Catat: status code, durasi pemrosesan
  v
CLIENT
  | HTTP Response: { status, data }
```

### 7.2 Alur Error

Ketika terjadi error di titik manapun dalam pipeline di atas:

```
Error dilempar di Service atau Repository
        |
        v
Error menggelembung ke atas (bubble up)
        |
        v
Global Exception Filter menangkap
        |
        +-- Custom exception (4xx):
        |     Format ke response error standar
        |     { status: "error", code: "...", message: "...", timestamp: "..." }
        |
        +-- Unexpected error (5xx):
              Catat full stack trace ke logging system
              Kembalikan response error generic
              { status: "error", code: "INTERNAL_SERVER_ERROR" }
```

---

## 8. Module Interaction

### 8.1 Peta Interaksi Antar Module

Diagram berikut menunjukkan Module mana yang memanggil Service dari Module lain. Panah menunjukkan arah pemanggilan.

```
[NAMA MODULE 1]
    |
    | Memanggil Service
    v
[NAMA MODULE 2]      [NAMA MODULE 3]
    |                     |
    | Mengakses DB         | Mengakses DB
    v                     v
Database             Database

---

[NAMA MODULE 4]
    |
    +-- Memanggil [NAMA MODULE 2].findById()
    +-- Memanggil [NAMA MODULE 3].getStatus()
    |
    v
Database (tabel milik Module 4 sendiri)
```

### 8.2 Aturan Interaksi

| Aturan | Penjelasan |
| --- | --- |
| Service memanggil Service (diperbolehkan) | Ketika Module A membutuhkan data dari domain Module B, Module A boleh memanggil Service Module B |
| Repository memanggil Repository (dilarang) | Module A tidak boleh query ke tabel yang dikelola Module B; harus lewat Service Module B |
| Circular dependency (dilarang) | Jika Module A memanggil Module B, maka Module B tidak boleh memanggil balik Module A |

### 8.3 Ketergantungan Antar Module

| Module | Bergantung Pada | Dibutuhkan Oleh |
| --- | --- | --- |
| `[MOD-01 — contoh: Authentication]` | `[MOD-02 — contoh: User]` | Hampir semua Module (untuk validasi token) |
| `[MOD-02 — contoh: User]` | Tidak ada | `[MOD-01]`, `[MOD-03]`, `[MOD-04]` |
| `[MOD-03]` | `[MOD-02]` | `[MOD-04]` |
| `[MOD-04]` | `[MOD-02]`, `[MOD-03]` | Tidak ada |
| `[MOD-05]` | `[MOD-02]` | Tidak ada |

Dependency graph ini harus diperbarui setiap kali ada Module baru atau dependency baru ditambahkan.

---

## 9. Dependency Overview

### 9.1 Service yang Dibutuhkan Sistem

Berikut adalah seluruh service eksternal yang harus berjalan agar project ini dapat beroperasi:

```
[NAMA PROJECT] Backend
        |
        ├── [Database — contoh: PostgreSQL]
        |     Wajib. Seluruh data persisten tersimpan di sini.
        |     Jika mati: sistem tidak dapat berfungsi sama sekali.
        |
        ├── [Cache — contoh: Redis]
        |     Wajib (di environment ini). Menyimpan [apa yang disimpan].
        |     Jika mati: [dampak konkret — contoh: autentikasi tidak berjalan karena session store tidak tersedia]
        |
        ├── [Queue — contoh: Bull/Redis]
        |     Tidak wajib untuk fungsi utama.
        |     Jika mati: [dampak — contoh: email tidak terkirim, namun API tetap merespons]
        |
        └── [Service lain jika ada]
              [Wajib/Tidak wajib]. [Dampak jika mati]
```

### 9.2 Dependency Kritis vs Non-Kritis

| Service | Kritis | Jika Tidak Tersedia |
| --- | --- | --- |
| [Database] | Ya | Seluruh API mengembalikan error; tidak ada yang bisa diproses |
| [Cache] | [Ya/Tidak] | [Dampak konkret] |
| [Queue] | Tidak | [Dampak konkret — contoh: Job asinkron tertunda; request API tetap diproses] |
| [Integrasi Email] | Tidak | [Dampak — contoh: Email tidak terkirim; Business Flow utama tetap berjalan] |

### 9.3 Library yang Paling Berpengaruh

Library berikut sangat tertanam di seluruh kode sehingga penggantiannya akan membutuhkan perubahan besar:

| Library | Versi | Dampak jika Diganti |
| --- | --- | --- |
| `[ORM — contoh: TypeORM]` | `[VERSI]` | Seluruh Repository harus ditulis ulang |
| `[Framework — contoh: NestJS]` | `[VERSI]` | Seluruh struktur Module, Controller, Service perlu diubah |
| `[Validation — contoh: class-validator]` | `[VERSI]` | Seluruh DTO perlu ditulis ulang |
| `[JWT library]` | `[VERSI]` | AuthService dan Middleware perlu ditulis ulang |

---

## 10. Integration Overview

### 10.1 Layanan Eksternal yang Digunakan

Project ini memanggil atau menerima panggilan dari layanan eksternal berikut:

| Layanan | Arah | Kapan Dipanggil | Kritis | Jika Gagal |
| --- | --- | --- | --- | --- |
| `[Nama Layanan — contoh: Email Service]` | Project → Layanan | [Contoh: Saat user baru dibuat, saat password direset] | Tidak | [Contoh: Email tidak terkirim; job dijadwalkan ulang via Queue] |
| `[Nama Layanan — contoh: Payment Gateway]` | Project → Layanan | [Contoh: Saat user melakukan pembayaran] | Ya | [Contoh: Transaksi gagal; pengguna diinformasikan untuk coba lagi] |
| `[Nama Layanan]` | Layanan → Project (Webhook) | [Contoh: Saat status pembayaran berubah di sisi gateway] | [Ya/Tidak] | [Dampak] |

### 10.2 Pola Integrasi yang Digunakan

**Integrasi Sinkron** — Project langsung menunggu respons dari layanan eksternal sebelum melanjutkan:

```
Digunakan untuk: [Contoh: verifikasi pembayaran real-time]

Service
  | Panggil API eksternal
  v
Adapter ([Nama Layanan]Adapter)
  | HTTP request ke layanan eksternal
  v
Layanan Eksternal
  | Response
  v
Adapter mengembalikan data ke Service
  | Jika gagal: lempar exception; Business Flow tidak dilanjutkan
```

**Integrasi Asinkron via Queue** — Project mempublikasikan job; Queue Worker memprosesnya di latar belakang:

```
Digunakan untuk: [Contoh: pengiriman email, notifikasi push]

Service
  | Publish job ke Queue: { type: "SEND_EMAIL", payload: {...} }
  | Tidak menunggu hasil
  v
Response dikembalikan ke client (request selesai)

[Di latar belakang]
Queue Worker mengambil job
  | Panggil Adapter layanan eksternal
  v
Layanan Eksternal
  | Jika gagal: retry [N] kali dengan jeda exponential
  | Jika semua retry gagal: pindahkan ke Dead Letter Queue
```

### 10.3 Adapter Pattern

Seluruh komunikasi dengan layanan eksternal dibungkus dalam Adapter class. Service tidak pernah memanggil HTTP client atau SDK eksternal secara langsung. Ini memastikan:

- Ketika layanan eksternal berubah, hanya Adapter yang perlu dimodifikasi
- Adapter dapat di-mock saat menulis unit test
- Lokasi semua panggilan ke layanan eksternal dapat ditemukan dengan mudah

---

## 11. Scheduler dan Queue Overview

`[KONDISIONAL]` Hapus bagian ini jika project tidak menggunakan Scheduler atau Queue.

### 11.1 Scheduler

Scheduler menjalankan job secara otomatis berdasarkan jadwal yang dikonfigurasi. Job Scheduler berjalan di dalam proses backend yang sama (bukan proses terpisah).

**Job terjadwal dalam project ini:**

| Nama Job | Jadwal | Fungsi |
| --- | --- | --- |
| `[NAMA JOB]` | `[Jadwal — contoh: Setiap hari pukul 02:00]` | [Fungsi konkret — contoh: Menghapus session yang sudah expire dari tabel sessions] |
| `[NAMA JOB]` | `[Jadwal]` | [Fungsi] |

Ketika Scheduler gagal mengeksekusi job, [jelaskan apa yang terjadi — contoh: error dicatat di log; tidak ada retry otomatis; job berikutnya dijadwalkan sesuai cron berikutnya].

### 11.2 Queue

Queue menangani job asinkron yang tidak perlu dieksekusi dalam satu siklus request. Queue Worker berjalan sebagai proses terpisah.

**Antrian yang ada dalam project ini:**

```
Queue: [NAMA QUEUE 1 — contoh: email-notification]
  Consumer: [NamaJobHandler]
  Dipublikasikan oleh: [Contoh: UserService saat user baru dibuat]
  Berisi job: [Contoh: SendWelcomeEmail, SendPasswordResetEmail]
  Retry policy: [Contoh: Maksimal 3 kali, jeda 60 detik antar percobaan]
  Jika semua retry gagal: [Contoh: Pindah ke queue email-notification-failed]

Queue: [NAMA QUEUE 2]
  Consumer: [NamaJobHandler]
  Dipublikasikan oleh: [...]
  Berisi job: [...]
  Retry policy: [...]
  Jika semua retry gagal: [...]
```

### 11.3 Alur Queue dari Publikasi hingga Eksekusi

```
Service mempublikasikan job
  | queue.add('SEND_WELCOME_EMAIL', { userId: '...' })
  v
Redis (penyimpanan antrian)
  | Job tersimpan di antrian
  v
Queue Worker mengambil job (polling)
  |
  v
Job Handler memproses
  | [NamaJobHandler].handle({ userId })
  |   → Ambil data user dari Database
  |   → Panggil EmailAdapter.sendWelcomeEmail()
  |   → Catat log sukses
  |
  +-- Sukses: hapus job dari antrian
  |
  +-- Gagal: increment retry counter
              Jadwalkan ulang dengan delay [N] detik
              Jika retry habis: pindah ke dead letter queue
```

---

## 12. Deployment Environment Overview

### 12.1 Environment yang Ada

Project dijalankan di empat environment yang memiliki konfigurasi berbeda:

```
local          development         staging           production
  |                 |                  |                  |
Mesin          Server shared        Server identik     Server live
developer      tim development      dengan production  pengguna nyata
  |                 |                  |                  |
Docker         Auto-deploy          Semi-auto deploy   Manual deploy
Compose        dari branch          dengan approval    dengan approval
               [develop]            QA                 Tech Lead
```

### 12.2 Perbedaan Konfigurasi per Environment

| Aspek | local | development | staging | production |
| --- | --- | --- | --- | --- |
| Database | Docker container lokal | Instance dedicated | Instance dedicated | Instance prod (High Availability) |
| Cache | Docker container lokal | Instance dedicated | Instance dedicated | Instance prod (Cluster) |
| Integrasi Eksternal | Sandbox / Mock | Sandbox | Sandbox | Production |
| Log Level | DEBUG | DEBUG | INFO | WARNING |
| Data | Seed dummy | Tidak stabil, dapat direset | Data anonim | Data riil |

### 12.3 Alur Deploy ke Production

```
Developer membuat Pull Request
        |
        v
CI Pipeline berjalan otomatis:
  - Unit test
  - Integration test
  - Linter check
  - Type check
  - Build check
        |
        +-- Gagal → PR tidak dapat di-merge
        |
        +-- Sukses → PR siap di-review
        |
        v
Code review dan approval Tech Lead
        |
        v
Merge ke [BRANCH DEV]
        |
        v
Auto-deploy ke environment development
        |
        v
QA testing di development
        |
        v
Merge ke [BRANCH STAGING]
        |
        v
Auto-deploy ke staging
        |
        v
UAT (User Acceptance Testing) di staging
        |
        v
Approval Tech Lead + Product Owner
        |
        v
Manual deploy ke production
```

Detail prosedur deployment ada di `docs/tsd/deployment/`.

---

## 13. Logging Overview

### 13.1 Cara Logging Bekerja di Project Ini

Project ini menggunakan **structured logging** — setiap entri log ditulis dalam format JSON dengan field yang konsisten. Ini memungkinkan log dicari dan difilter secara efisien di log aggregator.

```json
{
  "timestamp": "2025-01-15T08:30:00.000Z",
  "level": "INFO",
  "context": "UserController",
  "message": "User berhasil dibuat",
  "requestId": "uuid-request-id",
  "userId": "uuid-user-id",
  "duration": 142
}
```

### 13.2 Apa yang Dicatat

| Kejadian | Level Log | Yang Dicatat |
| --- | --- | --- |
| Setiap request masuk | INFO | Method, path, IP klien, Request ID |
| Setiap response keluar | INFO | Status code, durasi (ms) |
| Operasi bisnis penting | INFO | Nama operasi, ID entitas yang terlibat |
| Pemanggilan integrasi eksternal | INFO | Nama layanan, endpoint, status, durasi |
| Error yang dikenal (4xx) | WARN | Tipe error, pesan, konteks |
| Error tidak terduga (5xx) | ERROR | Stack trace lengkap, context request |
| Eksekusi job Scheduler | INFO | Nama job, waktu mulai, durasi, status |
| Pemrosesan job Queue | INFO | Nama job, job ID, status, durasi |

### 13.3 Yang Tidak Boleh Dicatat

- Password atau hash password
- Secret key, API key, token autentikasi
- Data kartu kredit atau data keuangan sensitif
- Data pribadi pengguna secara lengkap (cukup ID-nya)

### 13.4 Akses Log per Environment

| Environment | Cara Akses Log |
| --- | --- |
| local | Output terminal langsung atau file log di `[PATH — contoh: storage/logs/]` |
| development | [NAMA TOOL — contoh: Grafana Loki / Datadog / CloudWatch] di [URL] |
| staging | [NAMA TOOL] di [URL] |
| production | [NAMA TOOL] di [URL]; akses terbatas untuk on-call engineer dan Tech Lead |

---

## 14. Scalability Consideration

### 14.1 Apa yang Sudah Disiapkan untuk Scaling

Keputusan teknis berikut sudah dibuat sejak awal dengan mempertimbangkan scaling di masa depan:

**Backend stateless**
Backend tidak menyimpan state sesi di memori proses. Session token disimpan di Database. Ini berarti backend dapat di-scale horizontal (menjalankan lebih dari satu instance) tanpa konfigurasi tambahan — setiap instance dapat melayani request apapun karena tidak bergantung pada state lokal.

**Cache terpusat**
Rate limiting counter dan session data disimpan di [CACHE], bukan di memori backend. Ketika ada lebih dari satu instance backend, semua instance berbagi state yang sama melalui cache terpusat.

**Pemrosesan asinkron via Queue**
Operasi yang tidak perlu hasil langsung (email, notifikasi, laporan besar) diproses melalui Queue. Queue Worker dapat di-scale secara independen dari backend API jika volume job meningkat.

### 14.2 Bottleneck yang Diketahui

| Komponen | Potensi Bottleneck | Kapan Menjadi Masalah | Solusi yang Diantisipasi |
| --- | --- | --- | --- |
| Database | Semua write ke satu instance | Ketika write load sangat tinggi | Read replica untuk query read-heavy |
| [Tabel yang tumbuh cepat — contoh: audit_logs] | Tabel tumbuh terus, tidak ada archival | Setelah [N] bulan dengan [N] pengguna aktif | Partitioning atau archival berkala |
| Queue Worker | Satu worker untuk semua antrian | Ketika volume job meningkat signifikan | Tambah worker instance; pisah antrian per prioritas |

### 14.3 Yang Belum Disiapkan (dan Mengapa)

| Hal | Alasan Belum Dilakukan | Kapan Perlu Dilakukan |
| --- | --- | --- |
| [Contoh: Caching query Database] | [Contoh: Volume data saat ini belum membutuhkan; menambah kompleksitas tanpa manfaat nyata] | [Contoh: Ketika p95 response time API list melebihi 500ms] |
| [Contoh: CDN untuk static asset] | [Alasan] | [Trigger] |

---

## 15. Maintainability Consideration

### 15.1 Keputusan yang Menjaga Maintainability

**Konsistensi struktur Module**
Setiap Module memiliki struktur direktori yang identik. Developer yang sudah memahami satu Module dapat langsung produktif di Module lain karena polanya sama persis.

**Satu tabel, satu Module pemilik**
Setiap tabel Database hanya ditulis oleh satu Module. Ini berarti ketika ada bug data, selalu ada Module yang jelas bertanggung jawab.

**Tidak ada business flow di luar Service**
Seluruh aturan bisnis berada di Service. Controller hanya routing; Repository hanya query. Ini berarti untuk memahami bagaimana sebuah fitur bekerja, developer hanya perlu membaca satu file: Service-nya.

**Error handling terpusat**
Seluruh error diformat oleh satu Global Exception Filter. Format response error selalu konsisten tanpa perlu setiap Controller menghandle error secara berbeda.

### 15.2 Indikator Arsitektur Mulai Terdegradasi

Berikut adalah tanda-tanda yang menunjukkan arsitektur project mulai tidak sehat dan perlu perhatian:

| Tanda | Artinya | Tindakan |
| --- | --- | --- |
| Service dengan lebih dari [N — contoh: 20] method publik | Module terlalu besar; tanggung jawab terlalu banyak | Pecah menjadi sub-Module atau Service yang lebih spesifik |
| Controller yang berisi kondisional bisnis | Business Flow bocor ke lapisan yang salah | Pindahkan logika ke Service |
| Repository yang memanggil Service lain | Circular dependency mulai terbentuk | Review batas Module; ekstrak ke Service shared |
| Test yang sangat sulit ditulis | Coupling terlalu tinggi | Review dependency; pertimbangkan dependency injection |
| Migration yang mengubah nama kolom yang sudah ada | Tidak ada backward compatibility | Selalu additive-only; hapus kolom lama setelah deployment selesai |

### 15.3 Cara Menambah Feature Tanpa Merusak yang Sudah Ada

Urutan yang benar saat menambahkan feature baru:

```
1. Tentukan Module mana yang bertanggung jawab
   (jika tidak ada yang cocok, buat Module baru)
        |
        v
2. Buat migration Database jika diperlukan tabel baru
        |
        v
3. Buat atau perbarui Model
        |
        v
4. Buat atau perbarui Repository
        |
        v
5. Implementasi Business Flow di Service
        |
        v
6. Buat DTO untuk validasi input
        |
        v
7. Buat atau perbarui Controller
        |
        v
8. Tulis unit test untuk Service
9. Tulis unit test untuk Controller
10. Tulis integration test untuk alur lengkap
```