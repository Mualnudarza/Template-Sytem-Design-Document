# 00 - Fundamental TSD

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Foundation
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

## Cara Membaca Dokumen Ini

Dokumen ini adalah panduan bagi siapapun yang **menulis atau memperbarui** dokumentasi teknis project. Baca dokumen ini sebelum membuat dokumen TSD baru atau mengubah dokumen TSD yang sudah ada.

Dokumen ini menjawab tiga pertanyaan:
- **Apa** yang harus didokumentasikan di TSD?
- **Bagaimana** cara menulisnya?
- **Kapan** dokumentasi harus diperbarui?

---

## Daftar Isi

1. [Tujuan TSD](#1-tujuan-tsd)
2. [Ruang Lingkup TSD](#2-ruang-lingkup-tsd)
3. [Hubungan PSD dengan TSD](#3-hubungan-psd-dengan-tsd)
4. [Technical Documentation Principle](#4-technical-documentation-principle)
5. [Dependency Documentation Principle](#5-dependency-documentation-principle)
6. [Flow Documentation Principle](#6-flow-documentation-principle)
7. [Database Documentation Principle](#7-database-documentation-principle)
8. [API Documentation Principle](#8-api-documentation-principle)
9. [Deployment Documentation Principle](#9-deployment-documentation-principle)
10. [Modularization Principle](#10-modularization-principle)
11. [Consistency Principle](#11-consistency-principle)
12. [Update Workflow](#12-update-workflow)
13. [Documentation Lifecycle](#13-documentation-lifecycle)
14. [Versioning Documentation](#14-versioning-documentation)
15. [Dependency Mapping Principle](#15-dependency-mapping-principle)

---

## 1. Tujuan TSD

### 1.1 Definisi

TSD (Technical System Documentation) adalah kumpulan dokumen teknis yang menjelaskan bagaimana project ini diimplementasikan di level detail. TSD menjawab pertanyaan “bagaimana tepatnya ini bekerja” — mulai dari skema Database, kontrak API, alur request, hingga prosedur deployment.

TSD berbeda dari PSD dalam cara ini:

|  | PSD | TSD |
| --- | --- | --- |
| Menjawab | Apa sistem ini dan mengapa dibangun | Bagaimana tepatnya ini diimplementasikan |
| Level detail | High-level, konseptual | Low-level, konkret |
| Pembaca utama | Semua developer | Developer yang mengerjakan implementasi |
| Contoh dokumen | Project Overview, Architecture | Database Schema, API Endpoint, Flow teknis |

### 1.2 Tujuan TSD dalam Project Ini

TSD project ini ada untuk:

1. **Menjadi referensi implementasi** — Developer yang mengerjakan fitur tidak perlu membaca kode orang lain dari nol; mereka membaca TSD terlebih dahulu untuk memahami konteks
2. **Mendokumentasikan kontrak teknis** — Skema Database dan kontrak API adalah perjanjian antar komponen; TSD menjaganya tetap terdokumentasi dan dapat dirujuk
3. **Mempercepat debugging** — Ketika ada bug, TSD menyediakan peta alur data yang dapat diikuti untuk menemukan titik masalah
4. **Memungkinkan deployment yang dapat diulang** — Prosedur deployment yang terdokumentasi tidak bergantung pada ingatan satu orang
5. **Menjadi fondasi test case** — Skenario di flow documentation langsung menjadi dasar test case QA

### 1.3 Yang Bukan Tujuan TSD

TSD **bukan**:
- Tutorial framework atau teknologi yang digunakan
- Pengganti kode sebagai sumber kebenaran implementasi
- Design document untuk fitur yang belum diimplementasikan
- Dokumentasi yang berdiri sendiri tanpa konteks dari PSD

---

## 2. Ruang Lingkup TSD

### 2.1 Yang Ada di TSD

| Topik | Dokumen TSD | Isi |
| --- | --- | --- |
| Database | `docs/tsd/01-database-design.md` | Skema tabel, relasi, index, query pattern |
| API | `docs/tsd/02-api-documentation.md` | Endpoint, request, response, validasi, error |
| Business Flow | `docs/tsd/03-business-flow/` | Alur teknis end-to-end per fitur |
| Deployment | `docs/tsd/deployment/` | Prosedur build, deploy, rollback per environment |
| Integration | `docs/tsd/integration/` | Kontrak integrasi layanan eksternal |

### 2.2 Yang Bukan Ada di TSD

Informasi berikut ada di PSD, bukan TSD. Jangan menduplikasi ini ke TSD.

| Informasi | Lokasi yang Benar |
| --- | --- |
| Gambaran sistem dan tujuan bisnis | `docs/psd/01-project-overview.md` |
| Arsitektur sistem high-level | `docs/psd/03-system-architecture.md` |
| Cara menjalankan project | `docs/psd/02-setup-guide.md` |
| Standar penulisan kode | `docs/psd/04-coding-standard.md` |
| Keputusan arsitektur dan alasannya | `docs/psd/03-system-architecture.md` |

### 2.3 Struktur Direktori TSD

```
docs/tsd/
├── 00-fundamental-tsd.md        # Dokumen ini: panduan menulis TSD
├── 01-database-design.md        # Desain dan skema Database
├── 02-api-documentation.md      # Dokumentasi seluruh endpoint API
├── 03-business-flow/            # Flow teknis per Module
│   ├── README.md                # Panduan menulis flow documentation
│   ├── [nama-module]/
│   │   ├── README.md
│   │   └── [nama-flow]-flow.md
│   └── [nama-module]/
│       └── ...
├── deployment/                  # Prosedur deployment per environment
│   ├── 01-deployment-overview.md
│   └── 02-[environment].md
└── integration/                 # Detail kontrak integrasi eksternal
    └── [nama-layanan].md
```

---

## 3. Hubungan PSD dengan TSD

### 3.1 Hierarki

PSD dan TSD membentuk hierarki dua lapis. PSD mendefinisikan **apa** yang dibangun; TSD mendefinisikan **bagaimana** implementasinya.

```
PSD (Level Tinggi)
  ├── Gambaran project
  ├── Arsitektur sistem
  ├── Standar kode
  └── Setup guide
          |
          | TSD mewarisi konteks dari PSD
          v
TSD (Level Teknis)
  ├── Skema Database (turunan dari arsitektur PSD)
  ├── Kontrak API (implementasi dari Module di PSD)
  ├── Flow teknis (detail dari Business Flow di PSD)
  └── Prosedur deployment (detail dari environment di PSD)
```

### 3.2 Aturan Referensi

TSD **wajib** merujuk ke PSD untuk konteks arsitektur:

```markdown
// BENAR — TSD merujuk ke PSD untuk konteks
Komponen ini mengikuti pola lapisan yang dijelaskan di
`docs/psd/03-system-architecture.md` Bagian 2.

// BENAR — PSD merujuk ke TSD untuk detail teknis
Untuk skema Database lengkap, lihat `docs/tsd/01-database-design.md`.

// SALAH — TSD menduplikasi konten PSD
Sistem menggunakan arsitektur berlapis dengan Controller, Service,
dan Repository... [lanjut menjelaskan ulang apa yang ada di PSD]
```

### 3.3 Sinkronisasi yang Wajib Dijaga

| Jika PSD ini berubah | Periksa konsistensi TSD ini |
| --- | --- |
| Daftar Module di `01-project-overview.md` | Direktori di `03-business-flow/` |
| Tech stack di `01-project-overview.md` | Versi di `deployment/` |
| Environment di `02-setup-guide.md` | Konfigurasi di `deployment/` |
| Arsitektur lapisan di `03-system-architecture.md` | Penjelasan komponen di seluruh TSD |
| Konvensi naming di `04-coding-standard.md` | Terminologi yang digunakan di seluruh TSD |

---

## 4. Technical Documentation Principle

### 4.1 Akurasi Lebih Penting dari Kelengkapan

Dokumentasi yang akurat namun tidak lengkap lebih berharga dari dokumentasi yang lengkap namun tidak akurat.

Dokumentasi yang salah lebih berbahaya dari tidak ada dokumentasi — developer akan mengikuti informasi yang salah dan menghasilkan bug atau keputusan yang salah.

Implikasinya:
- Jika tidak yakin sebuah informasi benar, jangan tulis. Atau tulis dengan keterangan eksplisit bahwa informasi perlu diverifikasi.
- Lebih baik dokumen yang singkat tapi akurat daripada dokumen yang panjang tapi ada bagian yang sudah tidak berlaku.

### 4.2 Tulis untuk Developer yang Sedang Debugging di Malam Hari

Setiap dokumen TSD harus dapat digunakan oleh developer on-call yang menangani insiden production jam 2 pagi. Ini berarti:

- Langkah-langkah harus eksplisit, bukan implisit
- Perintah terminal harus bisa langsung di-copy dan dijalankan
- Kondisi yang menyebabkan error harus disebutkan dengan jelas
- Tidak ada asumsi bahwa pembaca ingat detail yang dijelaskan di tempat lain

### 4.3 Tunjukkan, Jangan Hanya Ceritakan

Setiap klaim teknis harus didukung oleh contoh konkret.

| Deskripsi Saja (Kurang Baik) | Deskripsi + Contoh (Baik) |
| --- | --- |
| “Endpoint ini memerlukan autentikasi” | Tampilkan header `Authorization: Bearer [TOKEN]` dalam contoh request |
| “Query menggunakan index pada kolom user_id” | Tunjukkan DDL index dan contoh query yang memanfaatkannya |
| “Flow ini melibatkan transaksi Database” | Tunjukkan tabel mana yang terlibat dan dalam urutan apa |

### 4.4 Dokumen TSD Bukan Komentar Kode

TSD menjelaskan **perilaku dan alur** sistem, bukan detail sintaksis kode. Jangan salin-tempel implementasi lengkap ke dokumentasi. Gunakan kode snippet hanya ketika snippet tersebut menjelaskan sesuatu yang tidak bisa dijelaskan dengan teks.

```
TEPAT sebagai contoh di TSD:
  - Contoh request curl yang bisa langsung dijalankan
  - Contoh payload JSON dengan nilai yang representatif
  - Pola query SQL yang digunakan untuk membaca data
  - Potongan kode yang menunjukkan keputusan bisnis penting

TIDAK TEPAT untuk TSD (taruh di kode, bukan TSD):
  - Implementasi lengkap sebuah method (>20 baris)
  - Detail sintaksis ORM
  - Konfigurasi framework
```

---

## 5. Dependency Documentation Principle

### 5.1 Setiap Komponen Wajib Mendokumentasikan Dependency-nya

Dokumentasi yang tidak menyebutkan dependency tidak berguna saat debugging atau saat merencanakan perubahan. Ketika seseorang mengubah komponen A, mereka perlu tahu siapa saja yang bergantung pada A.

### 5.2 Format Dokumentasi Dependency

Setiap dokumen TSD yang mendeskripsikan sebuah komponen wajib menyertakan bagian dependency dengan format ini:

```markdown
### Dependency

| Tipe | Target | Operasi | Kritikal |
|---|---|---|---|
| Module Internal | `UserService` | `findById()`, `updateStatus()` | Ya |
| Database | Tabel `users` | READ | Ya |
| Database | Tabel `sessions` | READ, WRITE | Ya |
| Cache | Key `user:profile:{id}` | READ, WRITE | Tidak (fallback ke DB) |
| Queue | `email-notification` | PUBLISH | Tidak (asinkron) |
| External | Payment Gateway | Charge, Refund | Ya |
```

`Kritikal: Ya` berarti komponen ini tidak dapat berjalan jika dependency tersebut tidak tersedia.

### 5.3 Impact Analysis Sebelum Perubahan

Sebelum mengubah kontrak teknis sebuah komponen (API, Database, interface Service), developer harus memeriksa bagian dependency di seluruh dokumen TSD yang merujuk komponen tersebut. Daftar dokumen yang terdampak harus disebutkan dalam deskripsi Pull Request.

---

## 6. Flow Documentation Principle

### 6.1 Tujuan Flow Documentation

Flow documentation menjelaskan bagaimana proses bisnis tertentu diimplementasikan dari awal hingga akhir — dari request masuk hingga response keluar, termasuk semua percabangan, error, dan side effect.

### 6.2 Komponen Wajib dalam Setiap Flow

Setiap dokumen flow wajib memiliki komponen berikut:

| Komponen | Isi |
| --- | --- |
| Flow Overview | Satu paragraf yang menjelaskan apa yang dilakukan flow ini |
| Trigger | Apa yang memulai flow (HTTP request, event, jadwal) |
| Actor / Role | Siapa yang dapat memicu flow ini |
| Preconditions | Kondisi yang harus terpenuhi sebelum flow dimulai |
| Postconditions | Kondisi yang dijamin setelah flow sukses |
| Business Rules | Aturan bisnis yang di-enforce dalam flow ini |
| Main Flow | Langkah-langkah dalam skenario normal |
| Alternative Flow | Variasi yang menghasilkan sukses dengan jalur berbeda |
| Exception Flow | Skenario yang menghasilkan error |
| Database Interaction | Tabel yang diakses dan operasi yang dilakukan |
| Side Effect | Apa lagi yang terjadi selain response (Queue, log, notifikasi) |
| Dependency | Komponen yang dibutuhkan flow ini |

### 6.3 Cara Menulis Main Flow

Main Flow ditulis sebagai langkah bernomor yang menunjukkan komponen mana yang terlibat di setiap langkah:

```markdown
## Main Flow

Kondisi yang diasumsikan: pengguna mengirim email dan password yang valid,
akun aktif, rate limit belum terlampaui.

Langkah 1: Client mengirim request
  Input : POST /api/v1/auth/login
          Body: { email: "user@domain.com", password: "Password123" }

Langkah 2: RateLimitMiddleware memeriksa batas request
  Komponen : RateLimitMiddleware
  Aksi     : Periksa counter dari Cache; counter belum melebihi batas

Langkah 3: AuthController.login() menerima request
  Komponen : AuthController
  Aksi     : Parse body ke LoginDto; validasi format email dan password

Langkah 4: AuthService.login() menjalankan Business Flow
  Komponen : AuthService
  Aksi     : Cari user berdasarkan email (UserRepository.findByEmail)
             Verifikasi password dengan bcrypt.compare()
             Generate access token dan refresh token
             Simpan refresh token ke Database (SessionRepository.create)

Langkah 5: Response dikembalikan
  Output : HTTP 200 OK
           { status: "success", data: { accessToken, refreshToken, expiresIn } }
```

### 6.4 Cara Menulis Exception Flow

Exception Flow ditulis per skenario dengan format yang jelas:

```markdown
## Exception Flow

### Exc-1: Kredensial tidak valid

Kondisi: Email tidak ditemukan ATAU password tidak cocok
Titik terjadi: AuthService.login() setelah query Database
Alasan pesan sama untuk keduanya: Mencegah penyerang mengetahui apakah email terdaftar
Response: HTTP 401 { "code": "INVALID_CREDENTIALS", "message": "..." }

### Exc-2: Akun non-aktif

Kondisi: Email ditemukan, password benar, tapi isActive = false
Titik terjadi: AuthService.login() setelah verifikasi password
Response: HTTP 401 { "code": "ACCOUNT_INACTIVE", "message": "..." }
```

---

## 7. Database Documentation Principle

### 7.1 Setiap Tabel Harus Menjelaskan Business Meaning-nya

Dokumentasi Database bukan hanya DDL. Setiap kolom harus dijelaskan maknanya dalam konteks bisnis project ini, bukan hanya tipe datanya.

```markdown
// KURANG BAIK — hanya tipe data
| Kolom     | Tipe         | Constraint |
| is_active | BOOLEAN      | NOT NULL   |

// LEBIH BAIK — dengan business meaning
| Kolom     | Tipe    | Constraint | Business Meaning |
| is_active | BOOLEAN | NOT NULL   | Status aktif akun. False berarti pengguna tidak dapat login.
|           |         |            | Diset false saat admin menonaktifkan akun. Bukan penanda soft delete. |
```

### 7.2 Komponen Wajib Dokumentasi Tabel

| Komponen | Isi |
| --- | --- |
| Tujuan Tabel | Apa yang disimpan tabel ini dan untuk keperluan bisnis apa |
| Struktur Field | Kolom, tipe, constraint, dan business meaning |
| Index | Index yang ada dan query mana yang memanfaatkannya |
| Relasi | FK yang ada, arah relasi, dan semantik bisnis relasi tersebut |
| Business Rules | Aturan bisnis yang di-enforce di level Database |
| Transaction Impact | Tabel mana yang harus diubah dalam satu transaksi bersama tabel ini |
| Query Pattern | Pola query yang paling sering digunakan |

### 7.3 Dokumentasikan Mengapa, Bukan Hanya Apa

```markdown
// KURANG BAIK — hanya mendokumentasikan apa
Kolom deleted_at menyimpan waktu record dihapus.

// LEBIH BAIK — mendokumentasikan mengapa
Kolom deleted_at digunakan sebagai mekanisme soft delete. Record tidak dihapus
secara fisik untuk menjaga referential integrity (tabel lain mungkin merujuk
record ini) dan untuk memenuhi kebutuhan audit trail. Nilai NULL berarti record
aktif. Semua query normal harus menyertakan filter `WHERE deleted_at IS NULL`.
```

---

## 8. API Documentation Principle

### 8.1 Kontrak API Harus Dapat Langsung Digunakan

Dokumentasi API yang baik memungkinkan developer mengintegrasikan endpoint tanpa perlu bertanya atau membaca kode. Setiap endpoint harus memiliki:

- Contoh request yang dapat langsung di-copy dan dijalankan
- Contoh response sukses dengan nilai yang representatif
- Semua kemungkinan response error dengan kondisi pemicunya

### 8.2 Komponen Wajib Dokumentasi Endpoint

| Komponen | Isi |
| --- | --- |
| Tujuan Bisnis | Mengapa endpoint ini ada; apa yang bisa dilakukan pengguna dengannya |
| Authentication | Apakah memerlukan token; Role apa yang diperbolehkan |
| Request | Method, path, semua parameter (path, query, body), aturan validasi |
| Response Sukses | Status code, struktur body, contoh |
| Response Error | Semua kemungkinan error, kondisi pemicu, format |
| Business Rules | Aturan bisnis yang divalidasi dalam endpoint ini |
| Side Effect | Apa yang terjadi selain response (email terkirim, Queue dipublikasikan) |
| Dependency | Service dan Database yang diakses |

### 8.3 Contoh Request Wajib Dapat Dijalankan

```markdown
// BAIK — contoh request yang bisa langsung dijalankan
```bash
curl -X POST https://[BASE_URL]/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@domain.com",
    "password": "Password123"
  }'
```

// KURANG BAIK — terlalu abstrak
curl -X POST [URL] -d [BODY]

```

---

## 9. Deployment Documentation Principle

### 9.1 Prosedur Deployment Harus Dapat Diikuti Oleh Siapapun

Dokumentasi deployment ditulis dengan asumsi bahwa pembacanya belum pernah melakukan deployment project ini sebelumnya. Ini penting karena deployment sering dilakukan saat darurat atau saat engineer yang biasanya melakukan tidak tersedia.

### 9.2 Komponen Wajib Dokumentasi Deployment

| Komponen | Isi |
|---|---|
| Prasyarat | Apa yang harus tersedia sebelum deployment dimulai |
| Checklist Pre-Deployment | Yang harus diperiksa sebelum menjalankan perintah apapun |
| Langkah Build | Perintah eksak untuk membuat artefak deployment |
| Langkah Deploy | Perintah eksak untuk menjalankan deployment |
| Migration | Apakah ada migration Database; urutan eksekusinya |
| Verifikasi | Bagaimana memastikan deployment berhasil |
| Rollback | Langkah eksak untuk membatalkan deployment jika ada masalah |

### 9.3 Semua Perintah Harus Eksak dan Dapat Dijalankan

```markdown
// BAIK — perintah yang bisa langsung di-copy
```bash
# Langkah 1: Pastikan branch terbaru
git pull origin main

# Langkah 2: Build artefak
npm run build

# Langkah 3: Jalankan migration
npm run db:migrate

# Langkah 4: Restart service
pm2 restart [nama-service]

# Langkah 5: Verifikasi
curl https://[DOMAIN]/api/health
```

// KURANG BAIK — terlalu abstrak
Lakukan build, jalankan migration, kemudian restart server.

```

---

## 10. Modularization Principle

### 10.1 Satu Topik, Satu Dokumen

TSD project ini menggunakan pendekatan satu dokumen per topik teknis. Jangan menggabungkan dokumentasi Database, API, dan Flow ke dalam satu file besar.

| Topik | Dokumen |
|---|---|
| Seluruh skema Database | `docs/tsd/01-database-design.md` |
| Seluruh API | `docs/tsd/02-api-documentation.md` |
| Flow per Module | `docs/tsd/03-business-flow/[module]/[flow]-flow.md` |
| Deployment | `docs/tsd/deployment/` |

### 10.2 Kapan Memecah Dokumen

Dokumen yang sudah sangat panjang (lebih dari 500 baris yang berguna) boleh dipecah. Cara memecahnya:
```

// Sebelum pemecahan
docs/tsd/02-api-documentation.md (berisi seluruh API semua Module)

// Setelah pemecahan
docs/tsd/api/
├── 00-api-overview.md (format, versioning, error codes)
├── 01-auth-endpoints.md (endpoint Module Authentication)
└── 02-user-endpoints.md (endpoint Module User Management)

```

Ketika dokumen dipecah, file indeks (index) wajib dibuat dan seluruh referensi ke dokumen lama harus diperbarui.

### 10.3 Hindari Duplikasi

Informasi yang sama tidak boleh ada di dua dokumen TSD sekaligus. Ketika informasi yang sama dibutuhkan di dua tempat, gunakan referensi:

```markdown
// BENAR — referensi ke dokumen lain
Format response error yang digunakan endpoint ini mengikuti standar yang
dijelaskan di `docs/tsd/02-api-documentation.md` Bagian 7.

// SALAH — menduplikasi konten
Format response error:
{ "status": "error", "code": "...", "message": "..." }
[...lanjut menduplikasi deskripsi yang sudah ada di dokumen lain]
```

---

## 11. Consistency Principle

### 11.1 Terminologi yang Digunakan di TSD

TSD project ini menggunakan terminologi yang sama dengan PSD. Istilah berikut tidak boleh diganti dengan sinonimnya.

| Istilah Resmi | Jangan Diganti Dengan |
| --- | --- |
| Controller | Handler, Router, Action |
| Service | Manager, Helper, Logic, Provider |
| Repository | DAO, Store, DataAccess |
| Model | Entity, Schema, Record |
| Database | DB, Storage |
| Migration | Schema change, DB patch |
| Queue | Message Queue, Job Queue (boleh digunakan bersamaan untuk konteks berbeda) |
| Scheduler | Cron, Task Scheduler |
| Middleware | Interceptor (dalam konteks request pipeline) |
| Flow | Process, Workflow (kecuali dalam nama dokumen) |

### 11.2 Format yang Konsisten

| Elemen | Format |
| --- | --- |
| Tanggal | `YYYY-MM-DD` |
| Versi API | `v1`, `v2` (huruf kecil) |
| HTTP method | Huruf kapital: `GET`, `POST`, `PUT` |
| Nama tabel | `snake_case` dalam backtick: ``users`` |
| Nama kolom | `snake_case` dalam backtick: ``user_id`` |
| Path API | dalam backtick: ``GET /api/v1/users`` |
| Nama class | `PascalCase` dalam backtick: ``UserService`` |
| Nama method | `camelCase()` dalam backtick: ``findById()`` |
| Kode error | `SCREAMING_SNAKE_CASE` dalam backtick: ``USER_NOT_FOUND`` |
| Environment variable | `SCREAMING_SNAKE_CASE` dalam backtick: ``DATABASE_URL`` |

### 11.3 Header Metadata Dokumen

Setiap dokumen TSD wajib memiliki header metadata di baris pertama:

```markdown
# [Judul Dokumen]

**Versi Dokumen:** [X.X.X]
**Status:** [Draft | Active | Deprecated]
**Kategori:** [Database | API | Flow | Deployment | Integration]
**Terakhir Diperbarui:** [YYYY-MM-DD]
**Pemilik Dokumen:** [NAMA TIM / ROLE]
```

---

## 12. Update Workflow

### 12.1 Trigger Wajib Update TSD

Dokumentasi TSD harus diperbarui **dalam Pull Request yang sama** dengan perubahan kode yang memicunya. PR yang mengubah kontrak teknis tanpa memperbarui dokumentasi terkait tidak boleh di-merge.

| Perubahan Kode | TSD yang Harus Diperbarui |
| --- | --- |
| Endpoint baru atau perubahan kontrak API | `docs/tsd/02-api-documentation.md` |
| Kode error baru ditambahkan | Bagian error codes di `02-api-documentation.md` |
| Tabel baru atau perubahan kolom Database | `docs/tsd/01-database-design.md` |
| Migration baru | Bagian migration history di `01-database-design.md` |
| Perubahan Business Flow yang signifikan | Dokumen flow terkait di `03-business-flow/` |
| Integrasi eksternal baru atau berubah | `docs/tsd/integration/[nama-layanan].md` |
| Perubahan prosedur deployment | `docs/tsd/deployment/` |
| Bug diperbaiki dan solusinya perlu didokumentasikan | Troubleshooting reference |

### 12.2 Alur Update

```
Perubahan kode atau infrastruktur
        |
        v
Identifikasi dokumen TSD yang terdampak
(rujuk tabel trigger di atas)
        |
        v
Perbarui dokumen TSD dalam branch yang sama
        |
        v
Naikkan versi dokumen
(PATCH: koreksi, MINOR: tambah konten, MAJOR: perubahan breaking)
        |
        v
Perbarui field "Terakhir Diperbarui" di header
        |
        v
Tulis entri changelog di bagian akhir dokumen
        |
        v
PR mencakup kode + dokumentasi sekaligus
        |
        v
Reviewer memvalidasi konsistensi dokumentasi
        |
        v
Merge
```

### 12.3 Checklist Review Dokumentasi

Reviewer harus memvalidasi hal berikut sebelum menyetujui PR yang menyentuh TSD:

```
[ ] Terminologi konsisten dengan Bagian 11 dokumen ini
[ ] Header metadata sudah diperbarui
[ ] Contoh request/query dapat dijalankan
[ ] Tidak ada duplikasi dengan dokumen TSD lain
[ ] Konsisten dengan PSD yang relevan
[ ] Versi dokumen sudah dinaikkan
[ ] Changelog sudah diperbarui
[ ] Referensi silang masih valid
```

---

## 13. Documentation Lifecycle

### 13.1 Status Dokumen

Setiap dokumen TSD memiliki satu dari empat status. Status wajib tercantum di header metadata.

| Status | Artinya | Implikasi |
| --- | --- | --- |
| `Draft` | Sedang ditulis, belum divalidasi | Jangan dijadikan referensi; tandai dengan peringatan di atas dokumen |
| `Active` | Sudah divalidasi, mencerminkan kondisi sistem saat ini | Dokumen resmi yang digunakan tim |
| `Deprecated` | Sudah tidak berlaku karena sistem berubah | Harus menyebutkan dokumen penggantinya; tidak boleh dijadikan referensi |
| `Archived` | Sudah dipindahkan ke `docs/archive/`; hanya untuk referensi historis | Jangan digunakan untuk keputusan aktif |

### 13.2 Transisi Status

```
Draft    --> Active     : Setelah review dan persetujuan Tech Lead
Active   --> Deprecated : Ketika fitur/komponen yang didokumentasikan berubah total
                          atau dihapus; harus ada referensi ke dokumen pengganti
Deprecated --> Archived : Setelah dua sprint sejak Deprecated
```

### 13.3 Menandai Dokumen yang Sudah Tidak Akurat

Ketika ada bagian dokumen yang diketahui sudah tidak akurat namun belum sempat diperbarui, tandai secara eksplisit:

```markdown
> **PERHATIAN:** Bagian ini sudah tidak akurat sejak[TANGGAL] karena
>[ALASAN SINGKAT]. Sedang diperbarui di tiket[KODE TIKET].
> Untuk informasi terkini, hubungi Tech Lead atau periksa langsung di kode.
```

---

## 14. Versioning Documentation

### 14.1 Skema Versi

Semua dokumen TSD menggunakan `MAJOR.MINOR.PATCH`:

| Segmen | Kapan Dinaikkan | Contoh |
| --- | --- | --- |
| `MAJOR` | Perubahan breaking pada kontrak teknis: endpoint dihapus, tabel di-rename, flow berubah total | `1.0.0 → 2.0.0` |
| `MINOR` | Penambahan konten baru tanpa mengubah yang sudah ada: endpoint baru, tabel baru, section baru | `1.0.0 → 1.1.0` |
| `PATCH` | Koreksi, klarifikasi, perbaikan typo, penambahan contoh | `1.0.0 → 1.0.1` |

### 14.2 Changelog di Setiap Dokumen

Setiap dokumen TSD wajib memiliki bagian Changelog di akhir dokumen:

```markdown
## Changelog

| Versi | Tanggal | Perubahan | Penulis |
|---|---|---|---|
| 1.0.0 | YYYY-MM-DD | Initial release | [ROLE] |
| 1.1.0 | YYYY-MM-DD | Tambah dokumentasi endpoint /users/export | [ROLE] |
| 1.1.1 | YYYY-MM-DD | Perbaiki contoh request yang salah format | [ROLE] |
| 2.0.0 | YYYY-MM-DD | BREAKING: hapus endpoint v1, semua endpoint pindah ke v2 | [ROLE] |
```

### 14.3 Referensi Versi ke Versi Sistem

Ketika ada kenaikan versi `MAJOR` pada dokumen TSD karena breaking change pada sistem, cantumkan versi sistem yang menjadi konteks perubahan tersebut di kolom keterangan changelog.

---

## 15. Dependency Mapping Principle

### 15.1 Mengapa Dependency Harus Dipetakan

Dependency yang tidak terdokumentasi adalah penyebab utama perubahan yang tidak sengaja merusak komponen lain. Ketika developer A mengubah kontrak Service X tanpa tahu bahwa Module B bergantung padanya, Module B bisa rusak tanpa ada peringatan.

### 15.2 Tipe Dependency yang Harus Didokumentasikan

| Tipe | Contoh | Di Mana Didokumentasikan |
| --- | --- | --- |
| Service memanggil Service lain | `OrderService` memanggil `UserService.findById()` | Di bagian Dependency dokumen flow terkait |
| Endpoint bergantung pada tabel | Endpoint `GET /users` membaca tabel `users` | Di dokumentasi endpoint terkait |
| Tabel memiliki FK ke tabel lain | `sessions.user_id` → `users.id` | Di dokumentasi tabel terkait |
| Flow memicu Queue | Login flow mempublikasikan ke `email-notification` | Di bagian Side Effect dokumen flow terkait |
| Module bergantung pada integrasi eksternal | `PaymentService` bergantung pada Payment Gateway | Di dokumentasi integrasi dan flow terkait |

### 15.3 Format Peta Dependency

Untuk komponen yang memiliki banyak dependency, gunakan format peta ini:

```
[Nama Komponen yang Didokumentasikan]
    |
    ├── [Tipe: Service Internal]
    │   └── UserService.findById()           [KRITIKAL]
    |
    ├── [Tipe: Database]
    │   ├── Tabel `users`    READ            [KRITIKAL]
    │   └── Tabel `sessions` READ, WRITE     [KRITIKAL]
    |
    ├── [Tipe: Cache]
    │   └── Key `session:{id}` READ, WRITE   [TIDAK KRITIKAL — fallback ke DB]
    |
    └── [Tipe: External]
        └── Email Service PUBLISH            [TIDAK KRITIKAL — asinkron via Queue]
```

### 15.4 Update Peta Dependency Saat Ada Perubahan

Setiap kali dependency baru ditambahkan atau dihapus:

1. Perbarui bagian Dependency di dokumen TSD komponen yang bersangkutan
2. Perbarui bagian Dependency di dokumen TSD komponen yang bergantung padanya
3. Cantumkan dalam deskripsi Pull Request komponen mana saja yang terdampak