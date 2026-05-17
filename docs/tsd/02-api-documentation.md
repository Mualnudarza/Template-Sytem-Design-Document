# 02 - API Documentation

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** API
**Terakhir Diperbarui:** [TANGGAL]
**Pemilik Dokumen:** [NAMA TIM / ROLE]
**Versi API:** v1

---

## Dokumen Terkait

| Dokumen | Path | Keterangan |
| --- | --- | --- |
| Fundamental TSD | `docs/tsd/00-fundamental-tsd.md` | Standar penulisan dokumentasi teknis |
| Database Design | `docs/tsd/01-database-design.md` | Tabel yang dibaca dan ditulis oleh endpoint ini |
| Business Flow | `docs/tsd/03-business-flow/` | Alur teknis lengkap per fitur |
| Project Overview | `docs/psd/01-project-overview.md` | Daftar Module dan Role yang berlaku |
| System Architecture | `docs/psd/03-system-architecture.md` | Arsitektur request flow dan autentikasi |

---

## Cara Membaca Dokumen Ini

Dokumen ini adalah referensi teknis untuk seluruh endpoint API project ini. Baca dari atas ke bawah untuk memahami konteks, lalu gunakan sebagai referensi endpoint saat mengerjakan fitur atau integrasi.

- **Mengintegrasikan API dari client** — Baca Bagian 1-4 (overview, auth, response, error), lalu langsung ke endpoint yang dibutuhkan
- **Mengerjakan fitur backend** — Baca Bagian 5 (request flow) dan Bagian 6 (middleware), lalu baca endpoint yang relevan
- **Menangani bug atau error** — Baca Bagian 7 (error codes) lalu cek detail endpoint terkait
- **QA membuat test case** — Setiap endpoint menyertakan skenario sukses, alternatif, dan error yang bisa langsung dijadikan test case

---

## Daftar Isi

1. [API Overview](#1-api-overview)
2. [Authentication Overview](#2-authentication-overview)
3. [Response Structure](#3-response-structure)
4. [Error Response](#4-error-response)
5. [Request Flow](#5-request-flow)
6. [Middleware Terkait](#6-middleware-terkait)
7. [Referensi Kode Error](#7-referensi-kode-error)
8. [Matriks Akses Endpoint](#8-matriks-akses-endpoint)
9. [Module: Authentication](#9-module-authentication)
10. [Module: User Management](#10-module-user-management)
11. [Module: [Nama Module]](#11-module-nama-module)
12. [Integration Impact](#12-integration-impact)

---

## 1. API Overview

### 1.1 Tentang API Project Ini

API project ini adalah REST API yang menjadi satu-satunya jalur komunikasi antara client (browser, aplikasi mobile, atau sistem eksternal) dengan backend. Seluruh data yang dibaca dan ditulis melalui sistem mengalir lewat API ini.

**Karakteristik API project ini:**

- **Stateless** — backend tidak menyimpan state sesi di memori; setiap request membawa token autentikasi sendiri
- **Format JSON** — seluruh request body dan response menggunakan `application/json`
- **Versioned** — versi ada di URL path (`/api/v1/`); perubahan breaking dilakukan dengan menaikkan versi
- **Response konsisten** — seluruh endpoint menggunakan format response yang sama baik untuk sukses maupun error

### 1.2 URL dan Konfigurasi

| Parameter | Nilai |
| --- | --- |
| Versi API aktif | `v1` |
| Base URL production | `https://[DOMAIN]/api/v1` |
| Base URL staging | `https://[DOMAIN-STAGING]/api/v1` |
| Base URL local | `http://localhost:[PORT]/api/v1` |
| Format request | `application/json` |
| Format response | `application/json` |
| Timestamp | ISO 8601 dalam UTC (`2025-01-15T08:30:00.000Z`) |

### 1.3 Konvensi URL

| Pola | Digunakan Untuk | Contoh |
| --- | --- | --- |
| `GET /[resource]` | Mengambil koleksi | `GET /api/v1/users` |
| `GET /[resource]/[id]` | Mengambil satu item | `GET /api/v1/users/[id]` |
| `POST /[resource]` | Membuat item baru | `POST /api/v1/users` |
| `PATCH /[resource]/[id]` | Memperbarui sebagian | `PATCH /api/v1/users/[id]` |
| `PUT /[resource]/[id]` | Mengganti penuh | `PUT /api/v1/users/[id]` |
| `DELETE /api/v1/[resource]/[id]` | Menghapus item | `DELETE /api/v1/users/[id]` |
| `GET /[resource]/[id]/[sub]` | Sub-resource | `GET /api/v1/users/[id]/orders` |

### 1.4 Endpoint Publik (Tidak Butuh Token)

Endpoint berikut dapat dipanggil tanpa token autentikasi:

| Endpoint | Keterangan |
| --- | --- |
| `POST /api/v1/auth/login` | Login untuk mendapatkan token |
| `POST /api/v1/auth/refresh` | Memperbarui access token |
| `GET /api/v1/health` | Health check untuk monitoring |

Seluruh endpoint lain memerlukan Bearer token yang valid.

---

## 2. Authentication Overview

### 2.1 Mekanisme Token

Project ini menggunakan **JWT dengan strategi dual token**:

| Token | Durasi | Disimpan Di | Fungsi |
| --- | --- | --- | --- |
| Access Token | `[DURASI — contoh: 15 menit]` | Memori client | Dikirim di setiap request; diverifikasi tanpa query Database |
| Refresh Token | `[DURASI — contoh: 7 hari]` | Database (tabel `sessions`) | Digunakan untuk mendapatkan access token baru |

### 2.2 Cara Menggunakan Token

Sertakan access token di header setiap request yang memerlukan autentikasi:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

### 2.3 Isi JWT (Access Token)

```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@domain.com",
  "role": "USER",
  "iat": 1705300200,
  "exp": 1705301100
}
```

| Field | Tipe | Keterangan |
| --- | --- | --- |
| `sub` | UUID | ID pengguna; digunakan sebagai identifier di seluruh sistem |
| `email` | string | Email saat token diterbitkan |
| `role` | string | Role aktif: nilai dari enum Role project ini |
| `iat` | unix timestamp | Waktu token diterbitkan |
| `exp` | unix timestamp | Waktu token kadaluwarsa |

### 2.4 Alur Refresh Token

```
Access token kadaluwarsa
  → Client mendapat 401 TOKEN_EXPIRED

Client memanggil POST /api/v1/auth/refresh
  { "refreshToken": "..." }

AuthService memvalidasi refresh token ke tabel sessions
  → Valid: terbitkan access token baru
  → Tidak valid: 401 REFRESH_TOKEN_INVALID (client harus login ulang)
```

### 2.5 Role dan Otorisasi

Setelah token divalidasi, Guard memeriksa Role pengguna:

| Role | Deskripsi Akses |
| --- | --- |
| `[ROLE_1 — contoh: SUPER_ADMIN]` | [Hak akses] |
| `[ROLE_2 — contoh: ADMIN]` | [Hak akses] |
| `[ROLE_3 — contoh: USER]` | [Hak akses] |

Dokumentasi setiap endpoint menyebutkan Role mana yang diperbolehkan. Penanda yang digunakan dalam dokumen ini:

- `PUBLIC` — tidak butuh token
- `Authenticated` — butuh token valid, Role apapun
- `ADMIN` — hanya Role ADMIN
- `Owner atau ADMIN` — pemilik resource atau ADMIN

---

## 3. Response Structure

### 3.1 Response Sukses — Satu Item

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Nama Pengguna",
    "email": "pengguna@domain.com",
    "role": "USER",
    "createdAt": "2025-01-15T08:30:00.000Z"
  }
}
```

### 3.2 Response Sukses — Koleksi dengan Pagination

```json
{
  "status": "success",
  "data": [
    { "id": "uuid-1", "name": "..." },
    { "id": "uuid-2", "name": "..." }
  ],
  "meta": {
    "total": 150,
    "page": 1,
    "limit": 20,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

### 3.3 Response Sukses — Tanpa Data

```json
{
  "status": "success",
  "message": "Operasi berhasil dilakukan"
}
```

### 3.4 Parameter Pagination

Endpoint koleksi mendukung parameter berikut:

| Parameter | Tipe | Default | Maks | Keterangan |
| --- | --- | --- | --- | --- |
| `page` | integer | `1` | — | Nomor halaman; dimulai dari 1 |
| `limit` | integer | `20` | `100` | Jumlah item per halaman |
| `sortBy` | string | `createdAt` | — | Field untuk sorting |
| `sortOrder` | `ASC` atau `DESC` | `DESC` | — | Urutan sorting |

---

## 4. Error Response

### 4.1 Format Error

Semua error menggunakan format yang sama:

```json
{
  "status": "error",
  "code": "KODE_ERROR_BISNIS",
  "message": "Pesan yang menjelaskan apa yang salah",
  "timestamp": "2025-01-15T08:30:00.000Z"
}
```

### 4.2 Format Error Validasi (422)

Untuk error validasi input, response menyertakan detail field yang bermasalah:

```json
{
  "status": "error",
  "code": "VALIDATION_ERROR",
  "message": "Terdapat kesalahan pada data yang dikirimkan",
  "errors": [
    { "field": "email", "message": "Format email tidak valid" },
    { "field": "password", "message": "Password minimal 8 karakter" }
  ],
  "timestamp": "2025-01-15T08:30:00.000Z"
}
```

### 4.3 HTTP Status Code yang Digunakan

| Status | Kapan Digunakan |
| --- | --- |
| `200 OK` | Request berhasil; data dikembalikan |
| `201 Created` | Resource baru berhasil dibuat |
| `204 No Content` | Berhasil; tidak ada data yang dikembalikan |
| `400 Bad Request` | Request tidak bisa diparsing (JSON malformed) |
| `401 Unauthorized` | Token tidak ada, tidak valid, atau expired |
| `403 Forbidden` | Token valid tapi Role tidak cukup |
| `404 Not Found` | Resource tidak ditemukan |
| `409 Conflict` | Melanggar constraint unik (data sudah ada) |
| `422 Unprocessable Entity` | Input valid tapi melanggar aturan bisnis atau validasi |
| `429 Too Many Requests` | Rate limit terlampaui |
| `500 Internal Server Error` | Error tidak terduga di server |

---

## 5. Request Flow

### 5.1 Jalur Request dari Masuk hingga Response

Setiap request yang masuk ke API melewati komponen berikut secara berurutan:

```
Client mengirim HTTP Request
        |
        v
HTTP Server (terima dan parse request)
        |
        v
Router (cocokkan URL + method ke Controller)
        |
        v
Logger Middleware
  → Catat: method, path, IP, timestamp, generate Request ID
        |
        v
Auth Middleware
  → Jika ada token: verifikasi; sisipkan user context ke request
  → Jika tidak ada token: lanjutkan (Guard nanti yang menolak)
        |
        v
Rate Limit Middleware
  → Periksa batas request user/IP
  → Jika melebihi: return 429
        |
        v
Roles Guard
  → Periksa Role user terhadap konfigurasi endpoint
  → Jika Role tidak cukup: return 403
        |
        v
Controller
  → Ekstrak data dari request
  → Validasi input via DTO
  → Jika validasi gagal: return 422
  → Delegasikan ke Service
        |
        v
Service
  → Jalankan Business Flow
  → Periksa aturan bisnis
  → Akses Database via Repository
  → Publish ke Queue jika perlu
        |
        v
Repository → Database
        |
        v
Response mengalir kembali ke atas
        |
        v
Logger Middleware (response)
  → Catat: status code, durasi (ms)
        |
        v
Client menerima HTTP Response
```

### 5.2 Request ID

Setiap request mendapat ID unik (`X-Request-ID`) yang:
- Disertakan di setiap entri log untuk korelasi
- Dikembalikan ke client via header `X-Request-ID`
- Digunakan untuk tracing saat debugging atau support

Jika client mengirim header `X-Request-ID`, nilai tersebut digunakan. Jika tidak, sistem generate UUID baru.

---

## 6. Middleware Terkait

### 6.1 Middleware yang Berjalan untuk Setiap Request

| Middleware | Urutan | Yang Dilakukan | Jika Gagal |
| --- | --- | --- | --- |
| `LoggerMiddleware` | 1 | Catat request masuk; generate Request ID | Tidak pernah gagal; logging error tidak memblokir request |
| `AuthMiddleware` | 2 | Verifikasi token JWT; sisipkan user context | Return 401 jika token ada tapi tidak valid; lanjutkan jika tidak ada token |
| `RateLimitMiddleware` | 3 | Periksa batas request dari Cache | Return 429 jika batas terlampaui |

**Catatan AuthMiddleware:** Middleware ini tidak memblokir request yang tidak membawa token. Yang memblokir adalah Roles Guard di level Controller. Ini memungkinkan endpoint publik berjalan tanpa token.

### 6.2 Header Request yang Diproses

| Header | Keterangan |
| --- | --- |
| `Authorization: Bearer [TOKEN]` | Wajib untuk endpoint terautentikasi |
| `Content-Type: application/json` | Wajib untuk request dengan body |
| `X-Request-ID` | Opsional; digunakan untuk distributed tracing |

### 6.3 Header Response yang Dikembalikan

| Header | Keterangan |
| --- | --- |
| `Content-Type: application/json; charset=utf-8` | Selalu ada |
| `X-Request-ID` | ID unik request; gunakan ini saat laporan masalah ke tim |
| `X-Response-Time` | Durasi pemrosesan dalam milidetik |
| `X-RateLimit-Limit` | Batas maksimum request dalam window |
| `X-RateLimit-Remaining` | Sisa request dalam window saat ini |
| `X-RateLimit-Reset` | Unix timestamp saat window reset |

---

## 7. Referensi Kode Error

### 7.1 Error Autentikasi dan Otorisasi

| Kode Error | HTTP | Kondisi Pemicu | Tindakan Client |
| --- | --- | --- | --- |
| `TOKEN_MISSING` | 401 | Header `Authorization` tidak ada | Tambahkan header Authorization |
| `TOKEN_INVALID` | 401 | Token tidak dapat diverifikasi (signature salah) | Minta pengguna login ulang |
| `TOKEN_EXPIRED` | 401 | Access token sudah melewati masa berlaku | Panggil `POST /auth/refresh` |
| `REFRESH_TOKEN_INVALID` | 401 | Refresh token tidak ada di Database | Minta pengguna login ulang |
| `REFRESH_TOKEN_EXPIRED` | 401 | Refresh token sudah kadaluwarsa | Minta pengguna login ulang |
| `INVALID_CREDENTIALS` | 401 | Email tidak ditemukan atau password salah | Pesan sengaja sama untuk mencegah user enumeration |
| `ACCOUNT_INACTIVE` | 401 | Akun dinonaktifkan admin | Informasikan pengguna untuk menghubungi admin |
| `INSUFFICIENT_PERMISSION` | 403 | Role tidak cukup untuk endpoint ini | Jangan ekspos detail role yang dibutuhkan ke pengguna |

### 7.2 Error Validasi dan Resource

| Kode Error | HTTP | Kondisi Pemicu |
| --- | --- | --- |
| `VALIDATION_ERROR` | 422 | Format input tidak valid; lihat field `errors` |
| `USER_NOT_FOUND` | 404 | User dengan ID tersebut tidak ada |
| `EMAIL_ALREADY_REGISTERED` | 409 | Email sudah digunakan akun lain |
| `CANNOT_DELETE_SELF` | 422 | Admin mencoba menghapus akun sendiri |
| `[KODE_ERROR_BISNIS]` | [STATUS] | [Kondisi pemicu] |

### 7.3 Error Sistem

| Kode Error | HTTP | Keterangan |
| --- | --- | --- |
| `RATE_LIMIT_EXCEEDED` | 429 | Tunggu sesuai nilai header `Retry-After` |
| `INTERNAL_SERVER_ERROR` | 500 | Catat `X-Request-ID` dan laporkan ke tim teknis |

---

## 8. Matriks Akses Endpoint

Ringkasan akses endpoint per Role. Detail ada di masing-masing section endpoint.

| Endpoint | Method | PUBLIC | USER | ADMIN | Catatan |
| --- | --- | --- | --- | --- | --- |
| `/auth/login` | POST | Ya | Ya | Ya |  |
| `/auth/refresh` | POST | Ya | Ya | Ya |  |
| `/auth/logout` | POST | — | Ya | Ya |  |
| `/auth/me` | GET | — | Ya | Ya |  |
| `/users` | GET | — | — | Ya |  |
| `/users/:id` | GET | — | Owner | Ya | USER hanya bisa lihat diri sendiri |
| `/users` | POST | — | — | Ya |  |
| `/users/:id` | PATCH | — | Owner (field terbatas) | Ya | USER tidak bisa ubah role dan isActive |
| `/users/:id` | DELETE | — | — | Ya | ADMIN tidak bisa hapus diri sendiri |
| `/health` | GET | Ya | Ya | Ya | Monitoring endpoint |
| `[endpoint lain]` | `[method]` | — | — | — |  |

---

## 9. Module: Authentication

### 9.1 Informasi Module

| Parameter | Nilai |
| --- | --- |
| Base path | `/api/v1/auth` |
| Controller | `AuthController` |
| Service | `AuthService` |
| Repository | `UserRepository`, `SessionRepository` |
| Tabel | `users`, `sessions` |
| Integrasi eksternal | `[Nama Email Service]` (notifikasi login via Queue) |

### 9.2 Dependency Module

```
AuthController
    |
    v
AuthService
    |
    ├── UserRepository     → Tabel users (READ)
    ├── SessionRepository  → Tabel sessions (READ, WRITE, DELETE)
    └── EmailAdapter       → [Email Service] via Queue (PUBLISH, non-kritikal)
```

---

### Endpoint: POST /api/v1/auth/login

**Tujuan:** Mengautentikasi pengguna dengan email dan password; mengembalikan token untuk sesi berikutnya.

**Authentication:** PUBLIC

**Validation:**

| Field | Tipe | Required | Aturan |
| --- | --- | --- | --- |
| `email` | string | Ya | Format email valid; di-lowercase dan di-trim |
| `password` | string | Ya | Tidak boleh kosong |

**Request:**

```bash
curl -X POST http://localhost:[PORT]/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@domain.com",
    "password": "Password123"
  }'
```

**Request Flow:**

```
1. RateLimitMiddleware periksa batas request per IP
   → Gagal: 429 RATE_LIMIT_EXCEEDED

2. LoginDto validasi format email dan keberadaan password
   → Gagal: 422 VALIDATION_ERROR

3. AuthService.login()
   → Cari user by email di UserRepository
   → Verifikasi password dengan bcrypt
   → Periksa is_active user
   → Generate access token (JWT)
   → Generate refresh token (random string)
   → Simpan refresh token ke SessionRepository

4. Side effect (asinkron via Queue)
   → Publish job: kirim email notifikasi login baru
   → Request tidak menunggu hasil ini

5. Return response
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "d3RoaXMtaXMtYS1zZWN1cmUtcmFuZG9t...",
    "expiresIn": 900,
    "tokenType": "Bearer",
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Nama Pengguna",
      "email": "admin@domain.com",
      "role": "ADMIN"
    }
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 422 | `VALIDATION_ERROR` | Format email tidak valid atau password kosong |
| 401 | `INVALID_CREDENTIALS` | Email tidak terdaftar atau password salah |
| 401 | `ACCOUNT_INACTIVE` | Akun dinonaktifkan admin |
| 429 | `RATE_LIMIT_EXCEEDED` | Terlalu banyak percobaan dari IP ini |

**Business Rules:**
- Pesan error untuk email tidak ditemukan dan password salah sengaja dibuat identik (`INVALID_CREDENTIALS`) untuk mencegah penyerang mengetahui apakah email terdaftar
- Akun dengan `is_active = false` tidak dapat login meski kredensial benar
- Setiap login sukses membuat satu record baru di tabel `sessions`

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `users` | READ by email |
| Database | `sessions` | WRITE (simpan refresh token) |
| Queue | `[NAMA_QUEUE]` | PUBLISH (notifikasi email, non-kritikal) |

---

### Endpoint: POST /api/v1/auth/refresh

**Tujuan:** Mendapatkan access token baru menggunakan refresh token yang masih valid, tanpa login ulang.

**Authentication:** PUBLIC

**Validation:**

| Field | Tipe | Required | Aturan |
| --- | --- | --- | --- |
| `refreshToken` | string | Ya | Tidak boleh kosong |

**Request:**

```bash
curl -X POST http://localhost:[PORT]/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "d3RoaXMtaXMtYS1zZWN1cmUtcmFuZG9t..."
  }'
```

**Request Flow:**

```
1. RefreshTokenDto validasi keberadaan refreshToken

2. AuthService.refresh()
   → Cari refresh token di SessionRepository
   → Periksa apakah belum expire (expires_at > NOW())
   → Ambil data user dari sessions.user_id
   → Periksa is_active user
   → Generate access token baru
   → [Opsional] Rotate refresh token

3. Return response
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 900,
    "tokenType": "Bearer"
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 422 | `VALIDATION_ERROR` | Field refreshToken kosong |
| 401 | `REFRESH_TOKEN_INVALID` | Token tidak ditemukan di Database |
| 401 | `REFRESH_TOKEN_EXPIRED` | Token sudah melewati masa berlaku |
| 401 | `ACCOUNT_INACTIVE` | Akun dinonaktifkan setelah token diterbitkan |

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `sessions` | READ (validasi token) |
| Database | `users` | READ (cek status akun) |

---

### Endpoint: POST /api/v1/auth/logout

**Tujuan:** Menginvalidasi sesi aktif dengan menghapus refresh token dari Database.

**Authentication:** Required (Authenticated)

**Request (opsional):**

Jika body tidak dikirim, **semua sesi aktif** pengguna dihapus (logout dari semua perangkat).
Jika body dikirim, hanya sesi yang bersangkutan yang dihapus (logout satu perangkat).

```bash
# Logout semua perangkat
curl -X POST http://localhost:[PORT]/api/v1/auth/logout \
  -H "Authorization: Bearer [ACCESS_TOKEN]"

# Logout satu perangkat
curl -X POST http://localhost:[PORT]/api/v1/auth/logout \
  -H "Authorization: Bearer [ACCESS_TOKEN]" \
  -H "Content-Type: application/json" \
  -d '{ "refreshToken": "d3RoaXMtaXMtYS1zZWN1cmUtcmFuZG9t..." }'
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "message": "Berhasil logout dari sistem"
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 401 | `TOKEN_INVALID` | Access token tidak valid |
| 401 | `TOKEN_EXPIRED` | Access token sudah kadaluwarsa |

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `sessions` | DELETE (hapus berdasarkan userId atau refreshToken) |

---

### Endpoint: GET /api/v1/auth/me

**Tujuan:** Mengambil data pengguna yang sedang terautentikasi berdasarkan token aktif.

**Authentication:** Required (Authenticated)

**Request:**

```bash
curl -X GET http://localhost:[PORT]/api/v1/auth/me \
  -H "Authorization: Bearer [ACCESS_TOKEN]"
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Nama Pengguna",
    "email": "pengguna@domain.com",
    "role": "USER",
    "isActive": true,
    "createdAt": "2025-01-10T08:00:00.000Z",
    "updatedAt": "2025-01-15T09:00:00.000Z"
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 401 | `TOKEN_INVALID` | Token tidak valid |
| 401 | `TOKEN_EXPIRED` | Token expired |
| 404 | `USER_NOT_FOUND` | User di JWT sudah dihapus dari sistem |

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `users` | READ by ID dari JWT payload |

---

## 10. Module: User Management

### 10.1 Informasi Module

| Parameter | Nilai |
| --- | --- |
| Base path | `/api/v1/users` |
| Controller | `UserController` |
| Service | `UserService` |
| Repository | `UserRepository` |
| Tabel | `users` |
| Integrasi eksternal | `[Nama Email Service]` (email selamat datang via Queue) |

### 10.2 Dependency Module

```
UserController
    |
    v
UserService
    |
    ├── UserRepository     → Tabel users (READ, WRITE, UPDATE, SOFT DELETE)
    └── EmailAdapter       → [Email Service] via Queue (PUBLISH, non-kritikal)
```

---

### Endpoint: GET /api/v1/users

**Tujuan:** Mengambil daftar seluruh pengguna sistem dengan pagination, sorting, dan filter.

**Authentication:** Required

**Authorization:** ADMIN

**Query Parameters:**

| Parameter | Tipe | Default | Keterangan |
| --- | --- | --- | --- |
| `page` | integer | `1` | Nomor halaman |
| `limit` | integer | `20` | Item per halaman (maks 100) |
| `search` | string | — | Filter pada `name` dan `email` (ILIKE) |
| `role` | string | — | Filter berdasarkan role |
| `isActive` | boolean | — | Filter status aktif |
| `sortBy` | string | `createdAt` | Field untuk sorting |
| `sortOrder` | `ASC` / `DESC` | `DESC` | Urutan sorting |

**Request:**

```bash
curl -X GET "http://localhost:[PORT]/api/v1/users?page=1&limit=20&search=john" \
  -H "Authorization: Bearer [ADMIN_TOKEN]"
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "John Doe",
      "email": "john@domain.com",
      "role": "USER",
      "isActive": true,
      "createdAt": "2025-01-10T08:00:00.000Z"
    }
  ],
  "meta": {
    "total": 45,
    "page": 1,
    "limit": 20,
    "totalPages": 3,
    "hasNextPage": true,
    "hasPreviousPage": false
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 401 | `TOKEN_INVALID` | Token tidak valid |
| 403 | `INSUFFICIENT_PERMISSION` | Bukan ADMIN |
| 422 | `VALIDATION_ERROR` | Query param tidak valid (page < 1, limit > 100) |

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `users` | READ (SELECT dengan filter, sorting, pagination) |

---

### Endpoint: GET /api/v1/users/:id

**Tujuan:** Mengambil detail satu pengguna berdasarkan ID.

**Authentication:** Required

**Authorization:** Owner atau ADMIN (USER hanya bisa lihat profil sendiri)

**Path Parameters:**

| Parameter | Tipe | Keterangan |
| --- | --- | --- |
| `id` | UUID | ID pengguna yang diminta |

**Request:**

```bash
curl -X GET http://localhost:[PORT]/api/v1/users/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer [TOKEN]"
```

**Request Flow:**

```
1. Validasi format UUID di path parameter

2. UserService.findById()
   → Otorisasi bisnis:
     - Jika bukan ADMIN dan request.user.sub ≠ id: 403
   → UserRepository.findById(id)
   → Jika tidak ditemukan: 404 USER_NOT_FOUND
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "John Doe",
    "email": "john@domain.com",
    "role": "USER",
    "isActive": true,
    "createdAt": "2025-01-10T08:00:00.000Z",
    "updatedAt": "2025-01-15T09:30:00.000Z"
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 403 | `INSUFFICIENT_PERMISSION` | USER mencoba lihat profil orang lain |
| 404 | `USER_NOT_FOUND` | User tidak ditemukan |
| 422 | `VALIDATION_ERROR` | ID bukan format UUID valid |

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `users` | READ by ID |

---

### Endpoint: POST /api/v1/users

**Tujuan:** Membuat akun pengguna baru. Hanya ADMIN yang dapat melakukan ini.

**Authentication:** Required

**Authorization:** ADMIN

**Validation:**

| Field | Tipe | Required | Aturan |
| --- | --- | --- | --- |
| `name` | string | Ya | Trim; maks 255 karakter |
| `email` | string | Ya | Format email valid; di-lowercase; harus unik |
| `password` | string | Ya | Min 8 karakter; harus ada huruf kapital dan angka |
| `role` | string | Tidak | Nilai valid: `[DAFTAR ROLE]`; default: `USER` |

**Request:**

```bash
curl -X POST http://localhost:[PORT]/api/v1/users \
  -H "Authorization: Bearer [ADMIN_TOKEN]" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@domain.com",
    "password": "Password123",
    "role": "USER"
  }'
```

**Request Flow:**

```
1. CreateUserDto validasi semua field

2. UserService.createUser()
   → Cek email unik: UserRepository.findByEmail(email)
   → Jika sudah ada: 409 EMAIL_ALREADY_REGISTERED
   → Hash password dengan bcrypt
   → UserRepository.create({ ...data, password: hashedPassword })
   → Publish job ke Queue: kirim email selamat datang

3. Return user yang baru dibuat (tanpa field password)
```

**Response Sukses (201):**

```json
{
  "status": "success",
  "data": {
    "id": "772a0622-g41d-63f6-c938-668877662222",
    "name": "John Doe",
    "email": "john@domain.com",
    "role": "USER",
    "isActive": true,
    "createdAt": "2025-01-15T10:00:00.000Z",
    "updatedAt": "2025-01-15T10:00:00.000Z"
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 403 | `INSUFFICIENT_PERMISSION` | Bukan ADMIN |
| 409 | `EMAIL_ALREADY_REGISTERED` | Email sudah digunakan |
| 422 | `VALIDATION_ERROR` | Validasi field gagal |

**Business Rules:**
- Password tidak pernah dikembalikan dalam response
- Field `isActive` selalu `true` untuk user yang baru dibuat
- Email di-lowercase sebelum disimpan

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `users` | READ (cek email) + WRITE (simpan user baru) |
| Queue | `[NAMA_QUEUE]` | PUBLISH (email selamat datang, non-kritikal) |

---

### Endpoint: PATCH /api/v1/users/:id

**Tujuan:** Memperbarui sebagian data pengguna. Hanya field yang dikirim yang diubah.

**Authentication:** Required

**Authorization:** Owner (field terbatas) atau ADMIN (semua field)

**Validation:**

| Field | Tipe | Required | Aturan | Hanya ADMIN |
| --- | --- | --- | --- | --- |
| `name` | string | Tidak | Trim; maks 255 karakter | Tidak |
| `email` | string | Tidak | Format email valid; lowercase; unik | Tidak |
| `role` | string | Tidak | Nilai valid sesuai enum | Ya |
| `isActive` | boolean | Tidak | — | Ya |

**Request:**

```bash
curl -X PATCH http://localhost:[PORT]/api/v1/users/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer [TOKEN]" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe Updated",
    "email": "john.new@domain.com"
  }'
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "John Doe Updated",
    "email": "john.new@domain.com",
    "role": "USER",
    "isActive": true,
    "createdAt": "2025-01-10T08:00:00.000Z",
    "updatedAt": "2025-01-15T11:00:00.000Z"
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 403 | `INSUFFICIENT_PERMISSION` | USER mencoba update profil orang lain atau field yang hanya untuk ADMIN |
| 404 | `USER_NOT_FOUND` | User tidak ditemukan |
| 409 | `EMAIL_ALREADY_REGISTERED` | Email baru sudah dipakai pengguna lain |
| 422 | `VALIDATION_ERROR` | Format field tidak valid |

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `users` | READ (validasi + cek email unik) + WRITE (update) |

---

### Endpoint: DELETE /api/v1/users/:id

**Tujuan:** Menonaktifkan akun pengguna (soft delete). Record tidak dihapus dari Database.

**Authentication:** Required

**Authorization:** ADMIN

**Request:**

```bash
curl -X DELETE http://localhost:[PORT]/api/v1/users/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer [ADMIN_TOKEN]"
```

**Request Flow:**

```
1. UserService.deactivate()
   → Cari user: UserRepository.findById(id)
   → Jika tidak ditemukan: 404 USER_NOT_FOUND
   → Cek apakah ADMIN menghapus dirinya sendiri
   → Jika ya: 422 CANNOT_DELETE_SELF
   → Dalam satu transaksi:
     - UPDATE users SET deleted_at = NOW(), is_active = false WHERE id = ?
     - DELETE FROM sessions WHERE user_id = ?

2. Return response sukses
```

**Response Sukses (200):**

```json
{
  "status": "success",
  "message": "Akun pengguna berhasil dinonaktifkan"
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| 403 | `INSUFFICIENT_PERMISSION` | Bukan ADMIN |
| 404 | `USER_NOT_FOUND` | User tidak ditemukan |
| 422 | `CANNOT_DELETE_SELF` | ADMIN mencoba menghapus akun sendiri |

**Business Rules:**
- Operasi ini adalah soft delete; data tidak hilang dari Database
- Seluruh sesi aktif user yang di-delete dihapus secara bersamaan dalam satu transaksi
- ADMIN tidak dapat menghapus akun sendiri untuk mencegah sistem tanpa ADMIN aktif

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `users` | READ + WRITE (soft delete) |
| Database | `sessions` | DELETE (hapus semua sesi aktif user) |

---

## 11. Module: [Nama Module]

> Gunakan section ini sebagai template untuk setiap Module baru yang ditambahkan ke project. Duplikasi seluruh struktur dari Bagian 10 (informasi module, dependency module, dan satu subsection per endpoint).
> 

### 11.1 Informasi Module

| Parameter | Nilai |
| --- | --- |
| Base path | `/api/v1/[resource]` |
| Controller | `[NamaController]` |
| Service | `[NamaService]` |
| Repository | `[NamaRepository]` |
| Tabel | `[nama_tabel]` |
| Integrasi eksternal | [Nama layanan jika ada, atau “Tidak ada”] |

### 11.2 Dependency Module

```
[NamaController]
    |
    v
[NamaService]
    |
    ├── [NamaRepository]       → Tabel [nama_tabel] ([OPERASI])
    └── [NamaAdapter] (jika ada) → [Layanan eksternal] ([OPERASI])
```

---

### Endpoint: [METHOD] /api/v1/[resource]

**Tujuan:** [Satu kalimat yang menjelaskan tujuan bisnis endpoint ini]

**Authentication:** [Required / PUBLIC]

**Authorization:** [PUBLIC / Authenticated / ADMIN / Owner atau ADMIN]

**Validation:**

| Field | Tipe | Required | Aturan |
| --- | --- | --- | --- |
| `[field]` | [tipe] | [Ya/Tidak] | [Aturan validasi] |

**Request:**

```bash
curl -X [METHOD] http://localhost:[PORT]/api/v1/[resource] \
  -H "Authorization: Bearer [TOKEN]" \
  -H "Content-Type: application/json" \
  -d '{
    "[field]": "[nilai contoh]"
  }'
```

**Request Flow:**

```
1. [Langkah 1 — komponen dan aksi]
2. [Langkah 2]
3. [Langkah N — termasuk kondisi error]
```

**Response Sukses ([STATUS]):**

```json
{
  "status": "success",
  "data": {
    "[field]": "[nilai contoh]"
  }
}
```

**Response Error:**

| HTTP | Kode Error | Kondisi |
| --- | --- | --- |
| [STATUS] | `[KODE_ERROR]` | [Kondisi pemicu] |

**Business Rules:**
- [Aturan bisnis yang berlaku untuk endpoint ini]

**Dependency:**

| Tipe | Target | Operasi |
| --- | --- | --- |
| Database | `[nama_tabel]` | [READ / WRITE / DELETE] |
| Queue | `[nama_queue]` | [PUBLISH] (jika ada) |
| External | `[Nama Layanan]` | [Operasi] (jika ada) |

---

## 12. Integration Impact

### 12.1 Endpoint yang Memiliki Side Effect ke Sistem Eksternal

| Endpoint | Side Effect | Layanan | Kritikal |
| --- | --- | --- | --- |
| `POST /auth/login` | Kirim email notifikasi login baru | `[Email Service]` via Queue | Tidak |
| `POST /users` | Kirim email selamat datang | `[Email Service]` via Queue | Tidak |
| `[Endpoint]` | [Side effect] | [Layanan] | [Ya/Tidak] |

`Kritikal: Tidak` berarti kegagalan side effect tidak memengaruhi status response endpoint — proses bisnis utama tetap berjalan.

### 12.2 Endpoint yang Bergantung pada Layanan Eksternal Secara Sinkron

| Endpoint | Layanan | Kapan Dipanggil | Jika Layanan Gagal |
| --- | --- | --- | --- |
| `[Endpoint]` | `[Nama Layanan]` | [Kondisi/event] | [Dampak — contoh: return 503 atau 422] |

### 12.3 Dampak Perubahan API ke Consumer

Ketika mengubah kontrak API (struktur request/response, kode error, HTTP status), perlu memperhatikan:

- **Mobile app** — perubahan breaking memerlukan update app yang didistribusikan; tidak bisa rollback cepat
- **Sistem integrasi eksternal** — perlu koordinasi dengan tim pemilik sistem tersebut
- **Frontend web** — dapat diupdate bersamaan dengan backend jika dalam satu repository

Prinsip perubahan API yang aman:
- Tambah field baru di response: aman (backward compatible)
- Ubah tipe data atau hapus field di response: breaking change; naikkan versi API
- Tambah field opsional di request: aman
- Jadikan field opsional menjadi required: breaking change; naikkan versi API

---