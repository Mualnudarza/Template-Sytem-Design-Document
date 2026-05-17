# Business Flow Documentation

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Flow
**Terakhir Diperbarui:** [TANGGAL]
**Pemilik Dokumen:** [NAMA TIM / ROLE]

---

## Dokumen Terkait

| Dokumen | Path | Keterangan |
| --- | --- | --- |
| Database Design | `docs/tsd/01-database-design.md` | Skema Database dan migration yang dijalankan saat deployment |
| API Documentation | `docs/tsd/02-api-documentation.md` | Endpoint yang diverifikasi setelah deployment selesai |
| Business Flow | `docs/tsd/03-business-flow.md` | Penjelasan modul hingga fitur yang berada pada project |
| Deployment Guide | `docs/tsd/04-deployment-guide.md` | Dokumentasi serta arahan untuk deployment project |

---

## Daftar Isi

1. [Tujuan Business Flow Documentation](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
2. [Ruang Lingkup](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
3. [Cara Membaca Flow Documentation](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
4. [Struktur Folder](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
5. [Standar Modularisasi](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
6. [Naming Convention](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
7. [Flow Relationship Principle](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
8. [Dependency Explanation Principle](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
9. [Diagram Usage Principle](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
10. [Code Snippet Usage Principle](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
11. [Business Rule Explanation Principle](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
12. [Daftar Module](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
13. [Daftar Flow Utama](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
14. [Relationship Antar Module](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
15. [Flow Ownership](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
16. [Update Workflow Dokumentasi](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)
17. [Versioning Flow Documentation](https://www.notion.so/03-business-flow-363039c050b8804aae22dd1db423db4c?pvs=21)

---

## 1. Tujuan Business Flow Documentation

### 1.1 Definisi

Business Flow Documentation adalah kumpulan dokumen yang menjelaskan bagaimana setiap proses dalam project ini bekerja dari awal hingga akhir — mulai dari siapa yang memicu proses, komponen mana yang terlibat, data apa yang bergerak, kondisi apa yang bisa terjadi, hingga hasil akhir yang dijamin sistem.

Direktori ini adalah bagian dari TSD. Setiap dokumen di sini mewarisi konteks bisnis dari PSD dan menjelaskan implementasinya di level teknis yang cukup untuk memahami flow tanpa harus membaca kode.

### 1.2 Pertanyaan yang Dijawab Dokumentasi Ini

Developer yang membaca dokumentasi ini harus mampu menjawab:

- Alur mana yang bertanggung jawab untuk proses bisnis X?
- Komponen mana yang terlibat dalam flow Y dan dalam urutan apa?
- Apa yang terjadi jika kondisi Z tidak terpenuhi?
- Data apa yang dibaca dan ditulis ke Database dalam flow ini?
- Job apa yang dipublikasikan ke Queue setelah flow ini selesai?
- Aturan bisnis apa yang di-enforce dalam flow ini?
- Flow ini bergantung pada Service atau Module mana?

### 1.3 Yang Bukan Tujuan Dokumentasi Ini

Dokumentasi ini **bukan**:

- Tutorial cara mengimplementasikan flow menggunakan framework tertentu
- Dokumentasi API endpoint (ada di `docs/tsd/02-api-documentation.md`)
- Dokumentasi skema Database (ada di `docs/tsd/01-database-design.md`)
- Panduan belajar arsitektur software secara umum
- Spesifikasi untuk fitur yang belum diimplementasikan

---

## 2. Ruang Lingkup

### 2.1 Yang Didokumentasikan

Setiap proses yang memiliki Business Rule, melibatkan lebih dari satu komponen, atau memiliki kemungkinan error yang perlu dipahami harus didokumentasikan sebagai flow.

| Kategori | Contoh | Didokumentasikan |
| --- | --- | --- |
| Flow yang dipicu API | Login, create user, submit laporan | Ya |
| Flow yang dipicu Scheduler | Kirim ringkasan harian, ekspirasi token | Ya |
| Flow yang dipicu Queue | Proses email, generate laporan besar | Ya |
| Flow validasi dan otorisasi | Verifikasi Role sebelum aksi | Ya |
| Logika internal Service sederhana | Format string, konversi tipe data | Tidak |

### 2.2 Batasan Dokumentasi

Flow documentation tidak menduplikasi informasi yang sudah ada di dokumen lain:

| Informasi | Lokasi yang Benar |
| --- | --- |
| Detail kontrak request/response endpoint | `docs/tsd/02-api-documentation.md` |
| Skema kolom dan relasi tabel Database | `docs/tsd/01-database-design.md` |
| Gambaran Module dan tech stack | `docs/psd/01-project-overview.md` |
| Aturan lapisan Controller/Service/Repository | `docs/psd/03-system-architecture.md` |

Flow documentation merujuk ke dokumen-dokumen di atas, tidak menduplikasi isinya.

---

## 3. Cara Membaca Flow Documentation

### 3.1 Urutan Baca untuk Developer Baru

Developer baru disarankan membaca dalam urutan berikut sebelum mengerjakan fitur apapun:

```
1. docs/psd/01-project-overview.md
   → Pahami Module apa saja yang ada dan bisnis yang dijalankan

2. docs/psd/03-system-architecture.md
   → Pahami lapisan sistem dan cara komponen berkomunikasi

3. docs/tsd/03-business-flow/README.md (dokumen ini)
   → Pahami peta seluruh flow yang ada

4. docs/tsd/03-business-flow/[nama-module]/README.md
   → Pahami scope Module yang akan dikerjakan

5. docs/tsd/03-business-flow/[nama-module]/[nama-flow]-flow.md
   → Pahami flow spesifik yang relevan dengan task
```

### 3.2 Membaca Satu Dokumen Flow

Setiap dokumen flow mengikuti struktur yang konsisten. Baca berdasarkan kebutuhan:

| Kebutuhan | Bagian yang Dibaca |
| --- | --- |
| Memahami flow secara cepat | Overview, Trigger, Main Flow |
| Menulis test case | Business Rules, Main Flow, Alternative Flow, Exception Flow |
| Debugging error production | Exception Flow, Failure Scenario, Logging Impact |
| Mengubah flow yang sudah ada | Dependency Component, Related Flow, Business Rules |
| Memahami dampak ke sistem lain | Data Movement, Database Interaction, External Integration |

### 3.3 Konvensi yang Digunakan dalam Flow

Seluruh dokumen flow menggunakan konvensi berikut secara konsisten:

| Konvensi | Makna |
| --- | --- |
| `Komponen: [NamaKelas]` | Layer atau class yang menjalankan langkah ini |
| `Aksi: [deskripsi]` | Yang dilakukan komponen di langkah ini |
| `Input: [data]` | Data yang masuk ke langkah ini |
| `Output: [data]` | Data yang keluar dari langkah ini |
| `[KRITIKAL]` | Dependency yang jika tidak tersedia menyebabkan flow gagal total |
| `[TIDAK KRITIKAL]` | Dependency yang jika tidak tersedia menyebabkan degradasi parsial, bukan kegagalan total |
| `Exc-[N]` | Nomor identifikasi Exception Flow |
| `Alt-[N]` | Nomor identifikasi Alternative Flow |

---

## 4. Struktur Folder

### 4.1 Hierarki Direktori

Business flow diorganisasi dalam hierarki tiga level: direktori root → Module → Flow.

```
docs/tsd/03-business-flow/
│
├── README.md                          # Dokumen ini: peta dan standar seluruh flow
│
├── _templates/                        # Template reusable untuk penulisan flow baru
│   ├── module-readme-template.md      # Template README per Module
│   └── flow-template.md               # Template dokumen flow
│
├── [nama-module]/                     # Satu direktori per Module bisnis
│   ├── README.md                      # Overview Module, scope, dan daftar flow
│   ├── [nama-flow]-flow.md            # Satu file per flow
│   └── [nama-flow]-flow.md
│
├── [nama-module]/
│   ├── README.md
│   └── [nama-flow]-flow.md
│
└── [nama-module]/
    ├── README.md
    └── [nama-flow]-flow.md
```

### 4.2 Contoh Struktur Nyata

```
docs/tsd/03-business-flow/
│
├── README.md
├── _templates/
│   ├── module-readme-template.md
│   └── flow-template.md
│
├── authentication/
│   ├── README.md
│   ├── login-flow.md
│   ├── refresh-token-flow.md
│   └── logout-flow.md
│
├── user-management/
│   ├── README.md
│   ├── create-user-flow.md
│   ├── update-user-flow.md
│   └── deactivate-user-flow.md
│
└── [nama-module]/
    ├── README.md
    └── [nama-flow]-flow.md
```

### 4.3 Aturan Satu File Satu Flow

Satu dokumen flow hanya boleh menjelaskan satu proses. Dokumen yang menggabungkan beberapa flow dalam satu file tidak diperbolehkan karena menyulitkan navigasi, referensi silang, dan maintenance.

Jika sebuah flow melibatkan sub-proses yang kompleks, sub-proses tersebut dapat dijadikan flow terpisah dan dirujuk dari flow utama menggunakan bagian Related Flow.

---

## 5. Standar Modularisasi

### 5.1 Satu Module — Satu Direktori

Setiap Module bisnis memiliki satu direktori yang berisi seluruh flow-nya. Batas direktori mengikuti batas Module yang didefinisikan dalam `docs/psd/01-project-overview.md`. Tidak boleh ada flow milik Module A yang disimpan di direktori Module B.

### 5.2 Flow Lintas Module

Jika sebuah flow melibatkan dua Module atau lebih secara signifikan, flow didokumentasikan di Module yang menjadi initiator (yang memulai flow). Module lain yang terlibat disebutkan sebagai dependency di bagian Dependency Component.

Contoh: Flow "Create Order" yang memanggil `UserService` dan `InventoryService` didokumentasikan di direktori `order-management/`, bukan di `user-management/` atau `inventory/`.

### 5.3 README Wajib per Module

Setiap direktori Module wajib memiliki `README.md` yang menjelaskan scope Module, daftar flow, dan dependency antar flow. README Module adalah titik masuk navigasi untuk Module tersebut. Developer yang baru pertama kali menyentuh Module harus membaca README Module sebelum membaca flow individual.

### 5.4 Granularitas Flow yang Tepat

| Terlalu Kasar (Hindari) | Tepat |
| --- | --- |
| `authentication-flow.md` (berisi login, logout, refresh semua dalam satu file) | `login-flow.md`, `logout-flow.md`, `refresh-token-flow.md` |
| `user-flow.md` (berisi create, update, delete) | `create-user-flow.md`, `update-user-flow.md`, `deactivate-user-flow.md` |

| Terlalu Halus (Hindari) | Tepat |
| --- | --- |
| `validate-email-format-flow.md` (hanya validasi format) | Bagian Validation Flow dalam `create-user-flow.md` |
| `hash-password-flow.md` (satu operasi internal) | Disebutkan sebagai langkah dalam flow yang relevan |

---

## 6. Naming Convention

### 6.1 Nama Direktori Module

Gunakan `lowercase-kebab-case` yang mencerminkan nama domain bisnis, bukan nama teknis.

| Benar | Salah |
| --- | --- |
| `authentication/` | `auth/`, `Auth/`, `authentication_module/` |
| `user-management/` | `users/`, `UserManagement/`, `user_management/` |
| `order-management/` | `orders/`, `OrderMgmt/` |
| `inventory/` | `Inventory/`, `inv/` |

### 6.2 Nama File Flow

Format wajib: `[nama-proses]-flow.md`

Nama proses menggunakan `lowercase-kebab-case` yang mendeskripsikan aksi, bukan entitas.

| Benar | Salah |
| --- | --- |
| `login-flow.md` | `Login.md`, `auth.md`, `login.flow.md` |
| `create-user-flow.md` | `create_user.md`, `CreateUser.md`, `user-create-flow.md` |
| `refresh-token-flow.md` | `token.md`, `refresh.md` |
| `export-monthly-report-flow.md` | `report-flow.md`, `monthly.md` |

### 6.3 Nama Bagian dalam Dokumen Flow

Gunakan heading yang konsisten sesuai template. Jangan mengganti nama bagian meskipun tampak sinonim.

| Nama Bagian Resmi | Jangan Diganti Dengan |
| --- | --- |
| `## Main Flow` | `## Happy Path`, `## Normal Flow`, `## Alur Utama` |
| `## Exception Flow` | `## Error Flow`, `## Alur Error`, `## Failure Flow` |
| `## Business Rules` | `## Aturan Bisnis`, `## Rules`, `## Constraints` |
| `## Dependency Component` | `## Dependencies`, `## Requires`, `## Komponen Terkait` |

---

## 7. Flow Relationship Principle

### 7.1 Setiap Flow Harus Mendeklarasikan Relasinya

Flow tidak berdiri sendiri. Setiap flow memiliki flow lain yang mendahuluinya (prerequisite), mengikutinya (continuation), atau berhubungan padanya (related). Relasi ini wajib dideklarasikan di bagian Related Flow agar developer dapat menelusuri keterhubungan proses bisnis.

### 7.2 Tipe Relasi Antar Flow

| Tipe Relasi | Keterangan | Cara Mendokumentasikan |
| --- | --- | --- |
| **Prerequisite** | Flow ini tidak dapat berjalan sebelum flow lain selesai | Sebutkan di bagian Preconditions dan Related Flow |
| **Triggered By** | Flow ini dipicu secara otomatis oleh flow lain | Sebutkan trigger di bagian Trigger |
| **Triggers** | Flow ini memicu flow lain setelah selesai | Sebutkan di bagian Postconditions atau Side Effect |
| **Shares Data** | Flow ini membaca data yang ditulis flow lain | Sebutkan di bagian Data Movement |
| **Kompetitor** | Flow ini dan flow lain tidak boleh berjalan bersamaan (mutex) | Sebutkan di bagian Technical Notes |

### 7.3 Contoh Relasi

```
Login Flow
    ├── Prerequisite   : (tidak ada; ini flow pertama dalam sesi)
    ├── Triggered By   : (tidak ada; dipicu langsung oleh client)
    ├── Triggers       : Tidak ada flow otomatis setelah login
    └── Shares Data    : Refresh Token Flow membaca session yang dibuat Login Flow
```

### 7.4 Cara Merujuk Flow Lain

Gunakan path relatif dari root direktori `03-business-flow/`:

```markdown
Lihat: `authentication/refresh-token-flow.md`
Lihat: `user-management/create-user-flow.md`
```

Jangan merujuk dengan nama saja tanpa path karena nama flow bisa ambigu antar Module.

---

## 8. Dependency Explanation Principle

### 8.1 Dependency Wajib Dideklarasikan Secara Eksplisit

Setiap flow wajib mendaftar seluruh dependency-nya — Service yang dipanggil, tabel yang diakses, cache key yang digunakan, Queue yang dipublikasikan, dan integrasi eksternal yang dihubungi. Dependency yang tidak terdokumentasi adalah risiko: developer yang mengubah komponen tidak tahu dampaknya ke flow lain.

### 8.2 Format Tabel Dependency

Gunakan format tabel standar berikut di bagian Dependency Component setiap flow:

```markdown
| Tipe | Komponen | Operasi | Kritikal | Keterangan |
|---|---|---|---|---|
| Service | `UserService` | `findByEmail()` | Ya | Memverifikasi eksistensi user |
| Service | `SessionService` | `create()` | Ya | Menyimpan refresh token baru |
| Database | Tabel `users` | READ | Ya | Membaca data user untuk autentikasi |
| Database | Tabel `sessions` | WRITE | Ya | Menyimpan session baru |
| Cache | Key `rate-limit:{ip}` | READ, WRITE | Tidak | Counter rate limiting; jika Redis down, request diloloskan |
| Queue | `email-notification` | PUBLISH | Tidak | Mengirim email notifikasi login; asinkron |
```

### 8.3 Penjelasan Tingkat Kritikal

| Kritikal | Artinya |
| --- | --- |
| `Ya` | Flow gagal total jika dependency ini tidak tersedia |
| `Tidak` | Flow tetap berjalan meskipun degradasi; ada fallback atau operasi bersifat opsional |

---

## 9. Diagram Usage Principle

### 9.1 Kapan Menggunakan Diagram

Diagram digunakan hanya ketika teks saja tidak cukup untuk menyampaikan alur, terutama untuk:

- Flow yang memiliki banyak percabangan kondisi
- Flow yang melibatkan lebih dari tiga komponen secara bersamaan
- State transition yang kompleks
- Sequence antar sistem yang perlu diklarifikasi urutannya

Untuk flow yang linier dan sederhana, text flow sudah cukup. Jangan menambah diagram hanya untuk estetika.

### 9.2 Format Diagram yang Digunakan

Diagram dalam dokumentasi ini menggunakan format text-based agar dapat dibaca langsung di file Markdown tanpa tool tambahan.

**Format text flow (untuk flow linier):**

```
Request masuk
      |
      v
Middleware memvalidasi token
      |
      +-- Token tidak valid → HTTP 401
      |
      +-- Token valid
              |
              v
        Controller menerima request
              |
              v
        Service menjalankan Business Flow
              |
              v
        Repository query Database
              |
              v
        Response dikembalikan → HTTP 200
```

**Format sequence text (untuk interaksi antar komponen):**

```
Client          Controller       Service          Repository       Database
  |                  |               |                |               |
  |-- POST /login -->|               |                |               |
  |                  |-- login() --->|                |               |
  |                  |               |-- findByEmail()-->|            |
  |                  |               |                |-- SELECT ---->|
  |                  |               |                |<-- user data--|
  |                  |               |<-- user -------|               |
  |                  |               |-- create() --->|               |
  |                  |               |                |-- INSERT ---->|
  |                  |<-- tokens ----|                |               |
  |<-- HTTP 200 -----|               |                |               |
```

### 9.3 Batasan Diagram

- Diagram tidak menggantikan penjelasan teks; diagram melengkapi teks
- Setiap diagram harus memiliki keterangan singkat di bawahnya yang menjelaskan apa yang digambarkan
- Jangan menggambar ulang struktur yang sudah ada di `docs/psd/03-system-architecture.md`

---

## 10. Code Snippet Usage Principle

### 10.1 Kapan Code Snippet Diperbolehkan

Code snippet dalam dokumentasi flow hanya digunakan ketika penjelasan teks tidak cukup untuk menyampaikan kontrak teknis, validasi, atau aturan bisnis yang spesifik.

| Diperbolehkan | Tidak Diperbolehkan |
| --- | --- |
| Contoh payload JSON request/response | Implementasi lengkap method (>20 baris) |
| Pola SQL query yang digunakan | Konfigurasi ORM |
| Pseudo-code kondisi bisnis yang kompleks | Detail sintaksis framework |
| Contoh struktur data yang dikirim ke Queue | Implementasi lengkap Service |

### 10.2 Aturan Code Snippet

- Snippet **harus singkat** — maksimal 20 baris per blok
- Snippet **harus diberi konteks** — sertakan komentar atau penjelasan sebelum snippet mengapa snippet ini perlu ada
- Snippet **bukan implementasi nyata** — boleh menggunakan pseudo-code selama lebih mudah dipahami
- Snippet **tidak boleh menjadi fokus utama** — jika developer perlu membaca implementasi, arahkan ke kode; bukan dokumentasi

### 10.3 Format Code Snippet

Selalu gunakan code block dengan identifier bahasa:

```markdown
```json
{
  "email": "user@domain.com",
  "password": "Password123"
}
```

```sql
-- Query yang digunakan untuk mencari user berdasarkan email
SELECT id, email, password_hash, is_active
FROM users
WHERE email = $1 AND deleted_at IS NULL
```
```

---

## 11. Business Rule Explanation Principle

### 11.1 Business Rule Harus Eksplisit

Business Rule adalah aturan yang di-enforce oleh sistem dan menentukan apakah sebuah operasi boleh dilanjutkan atau tidak. Business Rule tidak boleh tersirat; harus disebutkan secara eksplisit dalam dokumen flow agar developer yang mengubah kode tahu batasan yang tidak boleh dilanggar.

### 11.2 Format Business Rule

Setiap Business Rule ditulis dengan format bernomor dan harus menyebutkan di mana rule tersebut di-enforce:

```markdown
## Business Rules

**BR-1: Email harus unik di seluruh sistem**
Tidak ada dua akun yang boleh memiliki email yang sama, terlepas dari status akun (aktif, nonaktif, atau dihapus). Rule ini di-enforce di `UserService.create()` sebelum INSERT ke Database. Database juga memiliki UNIQUE constraint pada kolom `email` di tabel `users` sebagai lapisan kedua.

**BR-2: Password minimal 8 karakter dengan kombinasi huruf dan angka**
Validasi dilakukan di DTO sebelum request sampai ke Service. Pesan error yang dikembalikan tidak menyebutkan aturan spesifik mana yang dilanggar untuk mencegah brute force yang terarah.

**BR-3: Akun yang dinonaktifkan tidak dapat login**
Meskipun kredensial benar, akun dengan `is_active = false` mendapat respons HTTP 401 yang identik dengan respons kredensial salah. Ini untuk mencegah penyerang mengetahui status akun.
```

### 11.3 Hubungan Business Rule dengan Exception Flow

Setiap Business Rule yang dapat dilanggar harus memiliki Exception Flow yang menjelaskan apa yang terjadi ketika rule tersebut tidak terpenuhi. Nomor Exception Flow harus merujuk nomor Business Rule yang terkait.

---

## 12. Daftar Module

Daftar Module yang ada dalam sistem dan direktori dokumentasi flow-nya. Daftar ini harus selalu sinkron dengan daftar Module di `docs/psd/01-project-overview.md`.

| Module | Direktori | Status | Jumlah Flow |
| --- | --- | --- | --- |
| [NAMA MODULE 1 — contoh: Authentication] | `authentication/` | [Active / In Progress] | [N] |
| [NAMA MODULE 2 — contoh: User Management] | `user-management/` | [Active / In Progress] | [N] |
| [NAMA MODULE 3] | `[nama-direktori]/` | [Active / In Progress] | [N] |
| [NAMA MODULE 4] | `[nama-direktori]/` | [Active / In Progress] | [N] |
| [NAMA MODULE 5] | `[nama-direktori]/` | [Active / In Progress] | [N] |

> Jika Module baru ditambahkan ke sistem, direktori flow-nya harus dibuat bersamaan dengan implementasi Module menggunakan template di `_templates/module-readme-template.md`.
> 

---

## 13. Daftar Flow Utama

Referensi cepat seluruh flow yang terdokumentasi dalam sistem. Diurutkan per Module.

### Module: [NAMA MODULE 1 — contoh: Authentication]

| Flow | File | Trigger | Keterangan Singkat |
| --- | --- | --- | --- |
| [NAMA FLOW — contoh: Login] | `authentication/login-flow.md` | `POST /api/v1/auth/login` | Autentikasi user dan penerbitan token |
| [NAMA FLOW] | `authentication/refresh-token-flow.md` | `POST /api/v1/auth/refresh` | Memperbarui access token menggunakan refresh token |
| [NAMA FLOW] | `authentication/logout-flow.md` | `POST /api/v1/auth/logout` | Mencabut refresh token dan mengakhiri sesi |

### Module: [NAMA MODULE 2 — contoh: User Management]

| Flow | File | Trigger | Keterangan Singkat |
| --- | --- | --- | --- |
| [NAMA FLOW] | `user-management/create-user-flow.md` | `POST /api/v1/users` | Membuat akun pengguna baru |
| [NAMA FLOW] | `user-management/update-user-flow.md` | `PATCH /api/v1/users/:id` | Memperbarui data profil pengguna |
| [NAMA FLOW] | `user-management/deactivate-user-flow.md` | `DELETE /api/v1/users/:id` | Menonaktifkan akun pengguna |

### Module: [NAMA MODULE 3]

| Flow | File | Trigger | Keterangan Singkat |
| --- | --- | --- | --- |
| [NAMA FLOW] | `[direktori]/[nama]-flow.md` | [TRIGGER] | [KETERANGAN] |
| [NAMA FLOW] | `[direktori]/[nama]-flow.md` | [TRIGGER] | [KETERANGAN] |

---

## 14. Relationship Antar Module

### 14.1 Peta Dependency Antar Module

Berikut adalah peta ketergantungan antar Module. Panah menunjukkan arah dependency: Module A → Module B berarti Module A bergantung pada Module B.

```
[NAMA MODULE 1 — contoh: Authentication]
    |
    └── bergantung pada → [NAMA MODULE 2 — contoh: User Management]
                                |
                                └── bergantung pada → [NAMA MODULE 3]
                                                            |
                                                            └── bergantung pada → [NAMA MODULE LAIN]

[NAMA MODULE INDEPENDEN]
    └── (tidak bergantung pada Module lain)
```

### 14.2 Tabel Dependency Antar Module

| Module | Bergantung Pada | Jenis Dependency | Keterangan |
| --- | --- | --- | --- |
| [MODULE A] | [MODULE B] | Service call | [MODULE A] memanggil [NamaService] dari [MODULE B] untuk [tujuan] |
| [MODULE A] | [MODULE C] | Data read | [MODULE A] membaca tabel yang dimiliki [MODULE C] via Repository resmi |
| [MODULE B] | [MODULE D] | Event/Queue | [MODULE B] mempublikasikan event ke Queue yang dikonsumsi [MODULE D] |

### 14.3 Aturan Dependency Antar Module

Ketika sebuah Module membutuhkan data atau fungsi dari Module lain, komunikasi harus melalui Service yang diekspor — tidak langsung ke Repository Module lain. Ini menjaga batas domain tetap terjaga dan mencegah coupling pada level Database.

---

## 15. Flow Ownership

### 15.1 Kepemilikan Flow

Setiap flow memiliki satu Module sebagai pemilik. Pemilik Module bertanggung jawab atas:

- Akurasi dokumentasi flow
- Memperbarui dokumentasi saat implementasi berubah
- Memvalidasi konsistensi flow dengan API Documentation dan Database Design

### 15.2 Tabel Kepemilikan

| Module | Pemilik Dokumentasi | Reviewer |
| --- | --- | --- |
| [NAMA MODULE 1] | [ROLE — contoh: Backend Engineer — tim A] | Tech Lead |
| [NAMA MODULE 2] | [ROLE] | Tech Lead |
| [NAMA MODULE 3] | [ROLE] | Tech Lead |

### 15.3 Kepemilikan Bersama (Co-ownership)

Flow yang melibatkan lebih dari satu Module secara signifikan tetap dimiliki oleh Module initiator. Module yang terlibat sebagai dependency bertanggung jawab memperbarui dokumentasi mereka sendiri jika kontrak mereka berubah dan berdampak pada flow yang merujuk mereka.

---

## 16. Update Workflow Dokumentasi

### 16.1 Kapan Dokumentasi Flow Harus Diperbarui

Dokumentasi flow harus diperbarui dalam Pull Request yang sama dengan perubahan kode yang memicunya.

| Perubahan Kode | Dokumen yang Harus Diperbarui |
| --- | --- |
| Business Rule baru atau berubah | Bagian Business Rules di flow yang relevan |
| Langkah baru ditambahkan ke flow | Bagian Main Flow; periksa juga Alternative dan Exception Flow |
| Endpoint trigger berubah | Bagian Trigger; sinkronkan dengan `02-api-documentation.md` |
| Tabel baru terlibat | Bagian Database Interaction; sinkronkan dengan `01-database-design.md` |
| Dependency Service baru ditambahkan | Bagian Dependency Component |
| Integrasi eksternal baru | Bagian External Integration |
| Flow baru ditambahkan | Tambah file flow baru; perbarui README Module; perbarui Bagian 13 README ini |
| Module baru ditambahkan | Buat direktori Module; buat README Module; tambah ke Bagian 12 README ini |

### 16.2 Alur Update

```
Perubahan implementasi (kode, aturan bisnis, atau infrastruktur)
        |
        v
Identifikasi flow yang terdampak
(gunakan Bagian 13 dan 14 README ini sebagai referensi)
        |
        v
Perbarui dokumen flow dalam branch yang sama
        |
        v
Naikkan versi dokumen flow (lihat Bagian 17)
        |
        v
Perbarui README Module jika ada flow yang ditambah atau dihapus
        |
        v
Perbarui README ini jika ada Module atau flow utama yang berubah
        |
        v
PR mencakup kode + dokumentasi dalam satu commit
        |
        v
Tech Lead memvalidasi konsistensi saat review
        |
        v
Merge
```

### 16.3 Checklist Review Dokumentasi Flow

Reviewer harus memvalidasi hal berikut sebelum menyetujui PR yang menyentuh dokumentasi flow:

```
[ ] Nama file dan direktori mengikuti naming convention (Bagian 6)
[ ] Seluruh bagian wajib template diisi (tidak ada yang dikosongkan tanpa alasan)
[ ] Business Rules bernomor dan menyebutkan di mana rule di-enforce
[ ] Exception Flow merujuk Business Rule yang dilanggar
[ ] Dependency Component lengkap dengan status kritikal
[ ] Related Flow merujuk path yang benar dan file yang ada
[ ] Tidak ada duplikasi dengan API Documentation atau Database Design
[ ] README Module diperbarui jika ada flow baru
[ ] README root ini diperbarui jika ada Module atau flow utama yang berubah
[ ] Versi dokumen sudah dinaikkan
```

---

## 17. Versioning Flow Documentation

### 17.1 Skema Versi

Setiap dokumen flow menggunakan `MAJOR.MINOR.PATCH`:

| Segmen | Kapan Dinaikkan | Contoh |
| --- | --- | --- |
| `MAJOR` | Flow berubah secara fundamental: trigger berubah, Business Rule utama dihapus, komponen utama diganti | `1.0.0 → 2.0.0` |
| `MINOR` | Penambahan Alternative Flow, Exception Flow baru, Dependency baru ditambahkan | `1.0.0 → 1.1.0` |
| `PATCH` | Koreksi teks, klarifikasi langkah, perbaikan typo, penambahan contoh | `1.0.0 → 1.0.1` |

### 17.2 Header Metadata Wajib

Setiap dokumen flow wajib memiliki header metadata di baris pertama:

```markdown
# [Nama Flow]

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Flow
**Module:** [Nama Module]
**Terakhir Diperbarui:** [YYYY-MM-DD]
**Pemilik Dokumen:** [NAMA TIM / ROLE]
```

### 17.3 Changelog di Setiap Dokumen Flow

Setiap dokumen flow wajib memiliki bagian Changelog di akhir:

```markdown
## Changelog

| Versi | Tanggal | Perubahan | Penulis |
|---|---|---|---|
| 1.0.0 | YYYY-MM-DD | Initial release | [ROLE] |
| 1.1.0 | YYYY-MM-DD | Tambah Exception Flow Exc-3: rate limit terlampaui | [ROLE] |
| 2.0.0 | YYYY-MM-DD | BREAKING: Flow dipecah; refresh token flow dipindah ke file terpisah | [ROLE] |
```

---