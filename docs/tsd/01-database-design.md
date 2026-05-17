# 01 - Database Design

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Database
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

Dokumen ini menjelaskan seluruh struktur Database project ini. Baca bagian yang relevan dengan kebutuhan:

- **Mengerjakan fitur baru** — Baca Bagian 1-4 untuk gambaran, lalu baca detail tabel yang relevan
- **Menulis query** — Langsung ke detail tabel; fokus pada kolom, index, dan query pattern
- **Menangani bug data** — Baca Bagian 10 (dependency), lalu detail tabel yang terlibat
- **Membuat migration** — Baca Bagian 11 (migration), lalu baca aturan di Bagian 12 (konsistensi data)
- **Pertama kali memahami Database project** — Baca dari atas ke bawah; mulai dari Bagian 2 (ERD)

---

## Daftar Isi

1. [Database Overview](#1-database-overview)
2. [ERD Overview](#2-erd-overview)
3. [Daftar Tabel dan Fungsinya](#3-daftar-tabel-dan-fungsinya)
4. [Relasi Antar Tabel](#4-relasi-antar-tabel)
5. [Audit Field](#5-audit-field)
6. [Soft Delete](#6-soft-delete)
7. [Detail Tabel: `users`](#7-detail-tabel-users)
8. [Detail Tabel: `sessions`](#8-detail-tabel-sessions)
9. [Detail Tabel: `[dan seterusnya...]`](#9-detail-tabel-nama_tabel)
10. [Dependency Antar Tabel](#10-dependency-antar-tabel)
11. [Migration Overview](#11-migration-overview)
12. [Data Consistency Consideration](#12-data-consistency-consideration)

---

## 1. Database Overview

### 1.1 Database yang Digunakan

| Komponen | Teknologi | Versi | Tujuan |
| --- | --- | --- | --- |
| Database Utama | [NAMA — contoh: PostgreSQL] | [VERSI] | Penyimpanan data persisten seluruh sistem |
| Cache | [NAMA — contoh: Redis] | [VERSI] | Penyimpanan sementara: [sebutkan konkret — contoh: session token, rate limit counter] |
| ORM | [NAMA — contoh: TypeORM] | [VERSI] | Abstraksi akses Database dari kode |

### 1.2 Cara Kode Mengakses Database

Seluruh akses Database di project ini harus melalui lapisan Repository. Tidak ada query Database di Service atau Controller.

```
Service memanggil Repository
    |
    v
Repository menulis query via ORM
    |
    v
Connection Pool ([MIN]-[MAX] koneksi per proses)
    |
    v
Database Server
```

Pelanggaran terhadap aturan ini (query di luar Repository) adalah bug arsitektur. Lihat `docs/psd/04-coding-standard.md` Bagian 8 untuk detail.

### 1.3 Ringkasan Database

| Parameter | Nilai |
| --- | --- |
| Total tabel | [JUMLAH] |
| Database encoding | UTF-8 |
| Timezone | UTC (seluruh timestamp disimpan UTC) |
| Primary key type | UUID (menggunakan `gen_random_uuid()`) |
| Strategi penghapusan | Soft delete (field `deleted_at`) untuk tabel bisnis |

### 1.4 Konfigurasi Koneksi per Environment

| Parameter | local | development | staging | production |
| --- | --- | --- | --- | --- |
| Host | `localhost` | `[HOSTNAME]` | `[HOSTNAME]` | `[HOSTNAME]` |
| Database name | `[APP]_local` | `[APP]_dev` | `[APP]_staging` | `[APP]_prod` |
| Pool min | `2` | `5` | `10` | `20` |
| Pool max | `10` | `20` | `50` | `100` |
| SSL | Tidak | Tidak | Ya | Ya |

Environment variable yang digunakan: `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_NAME`, `DATABASE_USER`, `DATABASE_PASSWORD`, `DATABASE_SSL`. Lihat `docs/psd/02-setup-guide.md` Bagian 7 untuk detail nilai per environment.

---

## 2. ERD Overview

### 2.1 Diagram Relasi Antar Tabel

```
+------------------+          +------------------+
|     users        |          |    sessions      |
+------------------+          +------------------+
| PK  id           |1        N| PK  id           |
| name             +----------+ FK  user_id      |
| email (UNIQUE)   |          | refresh_token    |
| password         |          | expires_at       |
| role             |          | ip_address       |
| is_active        |          | created_at       |
| created_at       |          | updated_at       |
| updated_at       |          +------------------+
| deleted_at       |
+--------+---------+
         |
         | 1
         |
         | N
+--------+---------+          +------------------+
|   [TABEL_A]      |          |   [TABEL_B]      |
+------------------+          +------------------+
| PK  id           |1        N| PK  id           |
| FK  user_id      +----------+ FK  [tabel_a]_id |
| [field bisnis]   |          | [field bisnis]   |
| status           |          | status           |
| created_at       |          | created_at       |
| updated_at       |          | updated_at       |
| deleted_at       |          | deleted_at       |
+------------------+          +------------------+

+------------------+
|   audit_logs     |
+------------------+
| PK  id           |  ← Tidak ada FK; menyimpan snapshot data
| table_name       |
| record_id        |
| action           |
| old_values       |
| new_values       |
| performed_by     |
| performed_at     |
+------------------+
```

### 2.2 Legenda Notasi

| Simbol | Makna |
| --- | --- |
| `1 ---- N` | Satu record di kiri memiliki banyak record di kanan |
| `PK` | Primary key |
| `FK` | Foreign key |
| `UNIQUE` | Kolom dengan constraint UNIQUE |

### 2.3 Flow Data: Contoh Registrasi hingga Login

```
Aksi: Admin membuat user baru
  INSERT INTO users → { id, name, email, password_hash, role, is_active }

Aksi: User login
  SELECT FROM users WHERE email = ?       → verifikasi kredensial
  INSERT INTO sessions                    → simpan refresh token baru
  [JWT access token dibuat di memori]     → tidak disimpan ke Database

Aksi: User logout
  DELETE FROM sessions WHERE user_id = ? → hapus sesi aktif
```

---

## 3. Daftar Tabel dan Fungsinya

### 3.1 Master List

| Tabel | Module Pemilik | Fungsi | Repository |
| --- | --- | --- | --- |
| `users` | Authentication / User Management | Menyimpan identitas dan kredensial pengguna | `UserRepository` |
| `sessions` | Authentication | Menyimpan refresh token sesi aktif | `SessionRepository` |
| `audit_logs` | System | Mencatat jejak seluruh perubahan data penting | `AuditRepository` |
| `[nama_tabel]` | [Module] | [Fungsi] | `[NamaRepository]` |
| `[nama_tabel]` | [Module] | [Fungsi] | `[NamaRepository]` |

### 3.2 Pengelompokan Tabel per Module

```
Module: Authentication & User Management
  ├── users          → Akun dan identitas pengguna
  └── sessions       → Sesi aktif dan refresh token

Module: [Nama Module Bisnis]
  ├── [tabel_utama]  → [Deskripsi singkat]
  └── [tabel_detail] → [Deskripsi singkat]

Module: System
  └── audit_logs     → Audit trail perubahan data
```

---

## 4. Relasi Antar Tabel

### 4.1 Daftar Relasi

| Tabel Induk | Tabel Anak | Kolom FK | Tipe Relasi | ON DELETE |
| --- | --- | --- | --- | --- |
| `users` | `sessions` | `sessions.user_id` | One-to-Many | CASCADE |
| `users` | `[tabel_bisnis]` | `[tabel].user_id` | One-to-Many | RESTRICT |
| `[tabel_A]` | `[tabel_B]` | `[tabel_B].[tabel_a]_id` | One-to-Many | RESTRICT |

### 4.2 Penjelasan Kebijakan ON DELETE

| Kebijakan | Kapan Digunakan di Project Ini | Contoh |
| --- | --- | --- |
| `CASCADE` | Record anak tidak bermakna tanpa induknya | Hapus user → hapus semua sessions |
| `RESTRICT` | Record anak masih bermakna meski induk dihapus; lindungi data bisnis | Hapus user yang masih punya order aktif diblokir |

### 4.3 Catatan Soft Delete dan Cascade

Project ini menggunakan soft delete (field `deleted_at`) untuk tabel bisnis. Soft delete tidak memicu CASCADE karena record tidak dihapus secara fisik. Penghapusan cascade pada soft delete harus dilakukan manual di Service dalam satu transaksi.

Contoh: Ketika admin menonaktifkan user, Service wajib:
1. `UPDATE users SET deleted_at = NOW() WHERE id = ?`
2. `DELETE FROM sessions WHERE user_id = ?` (dilakukan manual dalam transaksi yang sama)

---

## 5. Audit Field

### 5.1 Field Standar

Setiap tabel bisnis di project ini wajib memiliki field berikut:

| Field | Tipe | Default | Diisi Oleh | Keterangan |
| --- | --- | --- | --- | --- |
| `created_at` | `TIMESTAMPTZ` | `NOW()` | Database otomatis | Waktu record dibuat; tidak boleh diubah |
| `updated_at` | `TIMESTAMPTZ` | `NOW()` | ORM otomatis | Diperbarui otomatis setiap kali record di-UPDATE |
| `deleted_at` | `TIMESTAMPTZ NULL` | `NULL` | Service secara manual | Waktu soft delete; `NULL` berarti record aktif |
| `created_by` | `UUID NULL` | `NULL` | Service (dari request context) | ID user yang membuat record |

### 5.2 Field Audit Opsional yang Direkomendasikan

| Field | Tipe | Keterangan |
| --- | --- | --- |
| `updated_by` | `UUID NULL` | ID user yang terakhir mengubah record |
| `deleted_by` | `UUID NULL` | ID user yang melakukan soft delete |

### 5.3 Tabel yang Dikecualikan dari Audit Field

| Tabel | Alasan Dikecualikan |
| --- | --- |
| `sessions` | Sesi dihapus secara fisik (hard delete); soft delete tidak relevan |
| `audit_logs` | Tabel append-only; tidak pernah diubah atau dihapus |

---

## 6. Soft Delete

### 6.1 Cara Kerja di Project Ini

Project ini tidak menghapus record secara fisik dari tabel bisnis. Record yang “dihapus” memiliki nilai non-null pada kolom `deleted_at`.

```sql
-- Record aktif
SELECT * FROM [tabel] WHERE deleted_at IS NULL;

-- Melakukan soft delete
UPDATE [tabel]
SET deleted_at = NOW(), deleted_by = '[user_id]'
WHERE id = '[record_id]' AND deleted_at IS NULL;

-- Memulihkan record yang ter-soft-delete
UPDATE [tabel]
SET deleted_at = NULL, deleted_by = NULL
WHERE id = '[record_id]';
```

### 6.2 Aturan Query

Semua query normal wajib menyertakan filter `WHERE deleted_at IS NULL`. ORM dikonfigurasi untuk menerapkan filter ini secara otomatis.

Jika ada query yang perlu mengakses record yang sudah di-soft-delete (misalnya untuk keperluan admin atau audit), filter ini harus dinonaktifkan secara eksplisit dengan komentar yang menjelaskan alasannya.

### 6.3 Alasan Menggunakan Soft Delete

Project ini menggunakan soft delete karena:
- Tabel lain mungkin memiliki FK yang merujuk ke record tersebut; hard delete akan merusak referential integrity
- Data yang terhapus perlu dapat dipulihkan jika ada kesalahan
- Jejak audit memerlukan data historis

---

## 7. Detail Tabel: `users`

### 7.1 Informasi Tabel

| Parameter | Nilai |
| --- | --- |
| Nama tabel | `users` |
| Module pemilik | Authentication, User Management |
| Repository | `UserRepository` |
| Strategi penghapusan | Soft delete |
| Tingkat risiko | Tinggi — di-referensi oleh hampir semua tabel lain |

### 7.2 Tujuan Tabel

Tabel `users` menyimpan identitas, kredensial, dan status akun seluruh pengguna sistem. Hampir semua tabel bisnis lain memiliki FK yang menunjuk ke tabel ini melalui kolom `user_id`. Perubahan struktur tabel ini berdampak pada seluruh sistem.

### 7.3 Struktur Field

| Kolom | Tipe | Constraint | Default | Business Meaning |
| --- | --- | --- | --- | --- |
| `id` | `UUID` | PK, NOT NULL | `gen_random_uuid()` | Identifier unik pengguna yang tidak pernah berubah. Digunakan sebagai referensi di seluruh sistem. |
| `name` | `VARCHAR(255)` | NOT NULL | — | Nama lengkap pengguna untuk ditampilkan di antarmuka. |
| `email` | `VARCHAR(255)` | NOT NULL, UNIQUE | — | Alamat email untuk login dan komunikasi. Disimpan dalam huruf kecil. Harus unik di seluruh sistem. |
| `password` | `VARCHAR(255)` | NOT NULL | — | Hash bcrypt dari password pengguna. **Tidak pernah dikembalikan di response API.** Kolom ini dikecualikan dari query SELECT normal via ORM (`select: false`). |
| `role` | `VARCHAR(20)` | NOT NULL, CHECK | `'USER'` | Role akses pengguna yang menentukan endpoint apa yang dapat diakses. Nilai valid didefinisikan di CHECK constraint. |
| `is_active` | `BOOLEAN` | NOT NULL | `TRUE` | Pengguna dengan nilai `false` tidak dapat login. Diset `false` saat admin menonaktifkan akun, terlepas dari nilai `deleted_at`. |
| `created_at` | `TIMESTAMPTZ` | NOT NULL | `NOW()` | Waktu akun dibuat. Tidak dapat diubah setelah dibuat. |
| `updated_at` | `TIMESTAMPTZ` | NOT NULL | `NOW()` | Waktu terakhir data akun diubah. Diperbarui otomatis. |
| `deleted_at` | `TIMESTAMPTZ` | NULLABLE | `NULL` | `NULL` = akun aktif. Terisi = akun di-soft-delete. |
| `created_by` | `UUID` | NULLABLE | `NULL` | ID admin yang membuat akun ini. `NULL` untuk akun yang dibuat via seed atau cara lain tanpa request context. |

### 7.4 Constraint

```sql
CONSTRAINT users_pkey PRIMARY KEY (id)
CONSTRAINT uq_users_email UNIQUE (email)
CONSTRAINT chk_users_role CHECK (role IN ('SUPER_ADMIN', 'ADMIN', 'USER'))
-- Sesuaikan nilai CHECK dengan Role yang ada di project ini
```

### 7.5 Index

| Nama Index | Kolom | Tujuan |
| --- | --- | --- |
| `users_pkey` | `id` | Lookup by primary key |
| `uq_users_email` | `email` | Constraint UNIQUE; digunakan saat login |
| `idx_users_role` | `role` | Filter pengguna berdasarkan role |
| `idx_users_is_active_deleted_at` | `is_active`, `deleted_at` | Query pengguna aktif (kondisi `deleted_at IS NULL`) |

### 7.6 Relasi

| Tipe | Tabel | Kolom FK | ON DELETE | Keterangan |
| --- | --- | --- | --- | --- |
| HAS MANY | `sessions` | `sessions.user_id` | CASCADE | Satu user memiliki banyak sesi aktif |
| HAS MANY | `[tabel_bisnis]` | `[tabel].user_id` | RESTRICT | Satu user memiliki banyak [entitas bisnis] |

### 7.7 Business Rules yang Di-enforce di Database

- Nilai `role` dibatasi via CHECK constraint; nilai di luar daftar akan ditolak langsung oleh Database
- `email` UNIQUE memastikan tidak ada dua akun dengan email yang sama, bahkan dalam kondisi race condition
- `password` dikecualikan dari SELECT default via ORM; harus di-select secara eksplisit hanya saat dibutuhkan (proses login)

### 7.8 Transaction Impact

Ketika user di-soft-delete, operasi berikut harus terjadi dalam satu transaksi:

```sql
BEGIN;
  UPDATE users SET deleted_at = NOW(), is_active = FALSE WHERE id = ?;
  DELETE FROM sessions WHERE user_id = ?;
COMMIT;
```

Jika salah satu gagal, keduanya di-rollback. Ini memastikan tidak ada user yang sudah di-delete namun masih memiliki sesi aktif.

### 7.9 Query Pattern Umum

```sql
-- Login: ambil user by email (harus eksplisit memilih kolom password)
SELECT id, email, password, role, is_active
FROM users
WHERE email = $1 AND deleted_at IS NULL;

-- Lookup by ID (paling sering digunakan)
SELECT id, name, email, role, is_active, created_at, updated_at
FROM users
WHERE id = $1 AND deleted_at IS NULL;

-- List dengan filter dan pagination
SELECT id, name, email, role, is_active, created_at
FROM users
WHERE deleted_at IS NULL
  AND ($1::text IS NULL OR name ILIKE '%' || $1 || '%' OR email ILIKE '%' || $1 || '%')
  AND ($2::varchar IS NULL OR role = $2)
ORDER BY created_at DESC
LIMIT $3 OFFSET $4;
```

---

## 8. Detail Tabel: `sessions`

### 8.1 Informasi Tabel

| Parameter | Nilai |
| --- | --- |
| Nama tabel | `sessions` |
| Module pemilik | Authentication |
| Repository | `SessionRepository` |
| Strategi penghapusan | Hard delete (tidak menggunakan soft delete) |
| Tingkat risiko | Tinggi — diakses setiap request terautentikasi |

### 8.2 Tujuan Tabel

Tabel `sessions` menyimpan refresh token yang aktif untuk setiap pengguna yang sedang login. Keberadaan record di tabel ini adalah bukti sesi valid. Saat pengguna logout atau sesi di-revoke, record dihapus secara fisik.

Access token (JWT) tidak disimpan di Database — backend memverifikasinya langsung dari signature tanpa perlu query.

### 8.3 Struktur Field

| Kolom | Tipe | Constraint | Default | Business Meaning |
| --- | --- | --- | --- | --- |
| `id` | `UUID` | PK, NOT NULL | `gen_random_uuid()` | Identifier unik sesi. |
| `user_id` | `UUID` | NOT NULL, FK → `users.id` | — | Pengguna pemilik sesi ini. |
| `refresh_token` | `VARCHAR(512)` | NOT NULL, UNIQUE | — | Nilai refresh token. Di-hash sebelum disimpan; nilai aslinya tidak dapat direkonstruksi dari Database. |
| `expires_at` | `TIMESTAMPTZ` | NOT NULL | — | Waktu refresh token kadaluwarsa. Digunakan untuk validasi; sesi dengan `expires_at < NOW()` tidak valid. |
| `ip_address` | `VARCHAR(45)` | NULLABLE | `NULL` | IP address saat sesi dibuat; untuk keperluan audit keamanan. |
| `device_info` | `VARCHAR(500)` | NULLABLE | `NULL` | Informasi browser atau device; untuk tampilan “sesi aktif” di UI jika ada. |
| `created_at` | `TIMESTAMPTZ` | NOT NULL | `NOW()` | Waktu sesi dibuat (waktu login). |
| `updated_at` | `TIMESTAMPTZ` | NOT NULL | `NOW()` | Waktu sesi terakhir diperbarui (saat token rotation). |

### 8.4 Constraint

```sql
CONSTRAINT sessions_pkey PRIMARY KEY (id)
CONSTRAINT uq_sessions_refresh_token UNIQUE (refresh_token)
CONSTRAINT fk_sessions_user_id
  FOREIGN KEY (user_id) REFERENCES users(id)
  ON DELETE CASCADE ON UPDATE CASCADE
```

### 8.5 Index

| Nama Index | Kolom | Tujuan |
| --- | --- | --- |
| `sessions_pkey` | `id` | Lookup by primary key |
| `uq_sessions_refresh_token` | `refresh_token` | Lookup sesi saat validasi token refresh |
| `idx_sessions_user_id` | `user_id` | Hapus semua sesi user saat logout dari semua perangkat |
| `idx_sessions_expires_at` | `expires_at` | Cleanup job: hapus sesi kadaluwarsa secara berkala |

### 8.6 Business Rules

- Satu pengguna boleh memiliki lebih dari satu sesi aktif (login dari beberapa perangkat)
- Sesi dengan `expires_at < NOW()` sudah tidak valid meskipun record masih ada; Scheduler membersihkannya secara berkala
- Logout satu perangkat menghapus satu record; logout semua perangkat menghapus semua record dengan `user_id` yang sama
- ON DELETE CASCADE berarti ketika user dihapus secara fisik dari Database, seluruh sesinya ikut terhapus otomatis. Namun di project ini user menggunakan soft delete, sehingga cascade tidak terpicu; penghapusan sesi dilakukan manual di Service.

### 8.7 Query Pattern Umum

```sql
-- Validasi refresh token saat endpoint /auth/refresh dipanggil
SELECT s.id, s.user_id, s.expires_at, u.is_active, u.role
FROM sessions s
JOIN users u ON u.id = s.user_id
WHERE s.refresh_token = $1
  AND s.expires_at > NOW()
  AND u.deleted_at IS NULL;

-- Logout satu perangkat
DELETE FROM sessions
WHERE refresh_token = $1 AND user_id = $2;

-- Logout semua perangkat
DELETE FROM sessions WHERE user_id = $1;

-- Cleanup job (dijalankan Scheduler)
DELETE FROM sessions WHERE expires_at < NOW();
```

---

## 9. Detail Tabel: `[nama_tabel]`

> Gunakan section ini sebagai template untuk setiap tabel bisnis yang ditambahkan ke project. Duplikasi section ini satu kali per tabel baru.
> 

### 9.1 Informasi Tabel

| Parameter | Nilai |
| --- | --- |
| Nama tabel | `[nama_tabel]` |
| Module pemilik | [Nama Module] |
| Repository | `[NamaRepository]` |
| Strategi penghapusan | [Soft delete / Hard delete] |
| Tingkat risiko | [Rendah / Sedang / Tinggi] |

### 9.2 Tujuan Tabel

[Jelaskan dalam 2-3 kalimat apa yang disimpan tabel ini dan mengapa tabel ini perlu ada. Hubungkan dengan kebutuhan bisnis konkret di project ini.]

### 9.3 Struktur Field

| Kolom | Tipe | Constraint | Default | Business Meaning |
| --- | --- | --- | --- | --- |
| `id` | `UUID` | PK, NOT NULL | `gen_random_uuid()` | Identifier unik record |
| `user_id` | `UUID` | NOT NULL, FK → `users.id` | — | [Jelaskan makna bisnis relasi ini] |
| `[nama_kolom]` | `[TIPE]` | [CONSTRAINT] | [DEFAULT] | [Business meaning — bukan sekadar tipe data] |
| `status` | `VARCHAR(50)` | NOT NULL, CHECK | `'[DEFAULT]'` | Status record: [daftar nilai valid dan maknanya] |
| `created_at` | `TIMESTAMPTZ` | NOT NULL | `NOW()` | Waktu record dibuat |
| `updated_at` | `TIMESTAMPTZ` | NOT NULL | `NOW()` | Waktu record terakhir diperbarui |
| `deleted_at` | `TIMESTAMPTZ` | NULLABLE | `NULL` | `NULL` = aktif. Terisi = di-soft-delete |

### 9.4 Constraint

```sql
CONSTRAINT [nama_tabel]_pkey PRIMARY KEY (id)
CONSTRAINT fk_[nama_tabel]_user_id
  FOREIGN KEY (user_id) REFERENCES users(id)
  ON DELETE RESTRICT ON UPDATE CASCADE
CONSTRAINT chk_[nama_tabel]_status
  CHECK (status IN ('[NILAI1]', '[NILAI2]', '[NILAI3]'))
```

### 9.5 Index

| Nama Index | Kolom | Tujuan |
| --- | --- | --- |
| `[nama_tabel]_pkey` | `id` | Lookup by primary key |
| `idx_[nama_tabel]_user_id` | `user_id` | Query record milik user tertentu |
| `idx_[nama_tabel]_status` | `status` | Filter by status |
| `idx_[nama_tabel]_created_at` | `created_at DESC` | Sorting dan range query |

### 9.6 Business Rules

- [Aturan bisnis yang di-enforce di level Database via constraint — contoh: status hanya boleh bernilai X, Y, atau Z]
- [Aturan lain]

### 9.7 Transaction Impact

[Jelaskan apakah operasi pada tabel ini harus dilakukan dalam transaksi bersama tabel lain. Jika ya, sebutkan tabel mana dan mengapa atomisitas diperlukan.]

### 9.8 Query Pattern Umum

```sql
-- Query paling umum: ambil record aktif milik user
SELECT [kolom yang diperlukan]
FROM [nama_tabel]
WHERE user_id = $1 AND deleted_at IS NULL
ORDER BY created_at DESC
LIMIT $2 OFFSET $3;

-- [Query lain yang sering digunakan]
```

---

## 10. Dependency Antar Tabel

### 10.1 Peta Dependency

Diagram berikut menunjukkan arah dependency antar tabel. Panah berarti “bergantung pada” (punya FK yang menunjuk ke).

```
users (TABEL INTI — semua tabel lain bergantung padanya)
    |
    |← sessions          (CASCADE: sesi tidak bermakna tanpa user)
    |← [tabel_bisnis_A]  (RESTRICT: data bisnis dilindungi)
    |     |
    |     |← [tabel_bisnis_B]  (CASCADE: detail tidak bermakna tanpa induk)
    |
    |← [tabel_bisnis_C]  (RESTRICT)
    |
    |← audit_logs.performed_by  (SET NULL: log tetap ada meski user dihapus)
```

### 10.2 Analisis Dampak Perubahan Tabel

| Jika tabel ini berubah | Tabel yang terdampak | Komponen kode yang terdampak |
| --- | --- | --- |
| `users` (skema kolom) | `sessions`, semua tabel dengan `user_id` | `UserRepository`, `AuthService`, JWT payload, semua Guard |
| `sessions` (skema kolom) | — | `SessionRepository`, `AuthService.refresh()`, `AuthService.logout()` |
| `[tabel_A]` (kolom status) | Tabel lain yang JOIN ke `[tabel_A]` | `[NamaRepository]`, semua query yang filter status |
| `audit_logs` (skema kolom) | — | `AuditRepository`, semua Service yang menulis audit |

### 10.3 Aturan Kepemilikan Data

Setiap tabel hanya boleh ditulis oleh Repository dari Module yang memilikinya. Module lain tidak boleh mengakses Repository dari Module lain secara langsung.

```
BENAR:
  OrderService membutuhkan data user
  → OrderService memanggil UserService.findById()
  → UserService memanggil UserRepository.findById()

SALAH:
  OrderService memanggil UserRepository.findById() langsung
```

---

## 11. Migration Overview

### 11.1 Cara Membuat Migration

Migration adalah satu-satunya cara yang diizinkan untuk mengubah skema Database. Tidak ada perubahan skema yang boleh dilakukan langsung ke Database tanpa file migration.

```bash
# Membuat file migration baru
[PERINTAH — contoh: npm run migration:create -- --name=create_orders_table]

# Menjalankan migration
[PERINTAH — contoh: npm run db:migrate]

# Melihat status migration
[PERINTAH — contoh: npm run db:migrate:status]

# Rollback migration terakhir (hanya di environment local)
[PERINTAH — contoh: npm run db:rollback]
```

### 11.2 Konvensi Nama File Migration

```
Format: [TIMESTAMP]_[deskripsi_snake_case].[ext]

Contoh:
  1705300200000_create_users_table.ts
  1705387200000_add_is_active_to_users.ts
  1705473600000_create_sessions_table.ts
  1705560000000_add_index_users_email.ts
```

### 11.3 Jenis Migration dan Tingkat Risikonya

| Jenis | Contoh | Risiko | Perlu Review |
| --- | --- | --- | --- |
| Tambah tabel baru | `CREATE TABLE orders (...)` | Rendah | Tidak |
| Tambah kolom NULLABLE | `ADD COLUMN phone VARCHAR(20) NULL` | Rendah | Tidak |
| Tambah index | `CREATE INDEX CONCURRENTLY ...` | Sedang | Tidak (gunakan CONCURRENTLY) |
| Tambah kolom NOT NULL tanpa default | `ADD COLUMN code VARCHAR(10) NOT NULL` | Tinggi | Ya |
| Hapus atau rename kolom | `DROP COLUMN deprecated_field` | Tinggi | Ya |
| Migration data | `UPDATE users SET role = 'USER' WHERE role IS NULL` | Tinggi | Ya |
| Ubah tipe data kolom | `ALTER COLUMN amount TYPE DECIMAL(15,2)` | Tinggi | Ya |

### 11.4 Prosedur Migration ke Production

```
1. Backup Database production (WAJIB sebelum migration apapun)

2. Jalankan migration di staging terlebih dahulu
   Validasi hasil: tidak ada error, data masih konsisten

3. Saat deploy ke production:
   - Jalankan migration sebelum restart service
   - Verifikasi migration sukses sebelum melanjutkan

4. Jika ada masalah:
   - Jalankan rollback migration
   - Kembalikan versi aplikasi
   - Investigasi masalah
```

---

## 12. Data Consistency Consideration

### 12.1 Operasi yang Memerlukan Transaksi

Beberapa operasi di project ini menyentuh lebih dari satu tabel dan harus dilakukan dalam satu transaksi agar atomik.

| Operasi Bisnis | Tabel yang Terlibat | Alasan Atomisitas |
| --- | --- | --- |
| Soft delete user | `users` + `sessions` | Tidak boleh ada user non-aktif dengan sesi yang masih valid |
| [Operasi Bisnis B] | `[tabel_1]` + `[tabel_2]` | [Mengapa harus atomik] |
| [Operasi Bisnis C] | `[tabel_1]` + `[tabel_2]` + `[tabel_3]` | [Alasan] |

### 12.2 Cara Menulis Transaksi di Project Ini

Transaksi dikelola di lapisan Service, bukan Repository. Repository hanya menerima parameter entity manager opsional untuk mendukung transaksi.

```tsx
// Di Service — cara yang benar
async deactivateUser(userId: string): Promise<void> {
  await this.dataSource.transaction(async (entityManager) => {
    await entityManager.update(User, userId, {
      isActive: false,
      deletedAt: new Date(),
    });
    await entityManager.delete(Session, { userId });
  });
}
```

### 12.3 Konsistensi yang Dijaga di Level Database

| Mekanisme | Contoh di Project Ini |
| --- | --- |
| Foreign Key | `sessions.user_id` tidak bisa berisi ID user yang tidak ada |
| UNIQUE constraint | Tidak bisa ada dua user dengan email yang sama |
| CHECK constraint | Nilai `role` di tabel `users` tidak bisa berisi nilai selain yang terdaftar |
| NOT NULL | Kolom wajib tidak bisa dilewati; bug di kode tidak bisa menghasilkan data tanpa field penting |

### 12.4 Tabel yang Paling Berisiko

| Tabel | Risiko | Yang Harus Diperhatikan |
| --- | --- | --- |
| `users` | Tinggi — semua tabel lain bergantung padanya | Perubahan skema harus dianalisis dampaknya ke seluruh sistem |
| `sessions` | Tinggi — diakses setiap request terautentikasi | Tabel tumbuh cepat; perlu cleanup berkala untuk menghindari bloat |
| `[tabel_transaksi_finansial]` | Kritis — data keuangan | Setiap operasi harus dalam transaksi; audit log wajib |
| `audit_logs` | Sedang — tumbuh terus | Tidak pernah dihapus; pertimbangkan partitioning setelah [N] juta baris |