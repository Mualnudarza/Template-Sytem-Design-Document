# 02 - Setup Guide

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Setup
**Terakhir Diperbarui:** [TANGGAL]
**Pemilik Dokumen:** [NAMA TIM / ROLE]

---

## Dokumen Terkait

| Dokumen | Path | Keterangan |
| --- | --- | --- |
| Project Overview | `docs/psd/01-project-overview.md` | Gambaran sistem, tech stack, dan daftar service yang di-setup |
| System Architecture | `docs/psd/03-system-architecture.md` | Arsitektur komponen yang dijalankan dalam setup ini |
| Coding Standard | `docs/psd/04-coding-standard.md` | Standar yang berlaku setelah setup selesai |

---

## Cara Membaca Dokumen Ini

Dokumen ini ditulis untuk dibaca dari atas ke bawah secara berurutan. Setiap langkah bergantung pada langkah sebelumnya. Jangan melewati langkah apapun kecuali secara eksplisit dinyatakan opsional atau kondisional.

**Simbol yang digunakan:**

| Simbol | Makna |
| --- | --- |
| `[WAJIB]` | Harus dilakukan tanpa pengecualian |
| `[KONDISIONAL]` | Hanya dilakukan jika kondisi yang disebutkan terpenuhi |
| `[OPSIONAL]` | Dapat dilewati; tidak mempengaruhi jalannya aplikasi |
| `[VERIFIKASI]` | Langkah pemeriksaan untuk memastikan langkah sebelumnya berhasil |

**Estimasi waktu setup pertama kali:** [N] menit (termasuk waktu download dependency)

---

## Daftar Isi

1. [Dependency Project](about:blank#2-dependency-project)
2. [Software Requirement](about:blank#3-software-requirement)
3. [Clone Repository](about:blank#4-clone-repository)
4. [Install Dependency](about:blank#5-install-dependency)
5. [Setup Environment](about:blank#6-setup-environment)
6. [Struktur Environment Project](about:blank#7-struktur-environment-project)
7. [Setup Configuration](about:blank#8-setup-configuration)
8. [Setup Database](about:blank#9-setup-database)
9. [Migration](about:blank#10-migration)
10. [Seed Data](about:blank#11-seed-data)
11. [Menjalankan Backend](about:blank#12-menjalankan-backend)
12. [Menjalankan Frontend](about:blank#13-menjalankan-frontend)
13. [Menjalankan Scheduler dan Queue](about:blank#14-menjalankan-scheduler-dan-queue)
14. [Validasi Setup Berhasil](about:blank#15-validasi-setup-berhasil)
15. [Common Command](about:blank#16-common-command)
16. [Common Setup Issue](about:blank#17-common-setup-issue)

---

## 1. Dependency Project

### 1.1 Service yang Harus Berjalan

Project ini membutuhkan service berikut untuk dapat berjalan. Seluruh service ini dapat dijalankan via Docker Compose yang sudah dikonfigurasi dalam repository.

| Service | Versi | Peran dalam Project | Dijalankan Via |
| --- | --- | --- | --- |
| [Database — contoh: PostgreSQL] | [VERSI] | Penyimpanan data persisten seluruh sistem | Docker Compose |
| [Cache — contoh: Redis] | [VERSI] | [Peran konkret — contoh: Menyimpan session token, rate limit counter, cache query] | Docker Compose |
| [Queue — contoh: Redis / RabbitMQ] | [VERSI] | [Peran — contoh: Antrian job asinkron untuk email dan notifikasi] | Docker Compose |
| [Service lain jika ada] | [VERSI] | [Peran] | [Docker Compose / Manual] |

### 1.2 Mengapa Setiap Service Ada

Bagian ini menjelaskan mengapa setiap service di atas dibutuhkan project ini — bukan penjelasan tentang teknologinya secara umum.

**[Database — contoh: PostgreSQL]**
Menyimpan seluruh data persisten project: [sebutkan entitas utama — contoh: data user, data karyawan, data kehadiran, data approval]. Tanpa service ini, aplikasi tidak dapat membaca atau menyimpan data apapun.

**[Cache — contoh: Redis]**
Menyimpan [sebutkan konkret apa yang di-cache — contoh: refresh token sesi pengguna dan counter rate limiting]. Tanpa service ini, autentikasi dan rate limiting tidak berfungsi. Cache juga digunakan untuk [penggunaan kedua jika ada].

**[Queue — contoh: Bull/Redis atau RabbitMQ]**
Mengantrikan [sebutkan jenis job — contoh: pengiriman email notifikasi, pemrosesan laporan yang memakan waktu]. Tanpa service ini, aksi-aksi asinkron tidak akan diproses meskipun API tetap merespons.

---

## 2. Software Requirement

### 2.1 Software yang Harus Terinstal

`[WAJIB]` Seluruh software berikut harus terinstal di mesin lokal sebelum memulai setup.

| Software | Versi Minimum | Cara Verifikasi | Unduh |
| --- | --- | --- | --- |
| [Runtime — contoh: Node.js] | [VERSI] | `[perintah --version]` | [URL resmi] |
| [Package manager — contoh: pnpm / npm / yarn] | [VERSI] | `[perintah --version]` | [URL resmi] |
| Git | 2.30 | `git --version` | https://git-scm.com |
| Docker | [VERSI] | `docker --version` | https://docs.docker.com/get-docker |
| Docker Compose | [VERSI] | `docker compose version` | Termasuk dalam Docker Desktop |

### 2.2 Software Opsional yang Direkomendasikan

`[OPSIONAL]` Software berikut tidak wajib, namun sangat membantu produktivitas selama development.

| Software | Fungsi dalam Project |
| --- | --- |
| [IDE — contoh: VS Code] | [Alasan spesifik — contoh: Extension ESLint dan Prettier sudah dikonfigurasi dalam repository di `.vscode/`] |
| [DB Client — contoh: TablePlus / DBeaver] | Inspeksi data Database secara visual saat debugging |
| [API Client — contoh: Postman / Insomnia] | Koleksi request API tersedia di direktori [PATH KOLEKSI] |

### 2.3 Verifikasi Software Terinstal

Sebelum melanjutkan, jalankan perintah berikut untuk memastikan seluruh software wajib sudah terinstal dengan versi yang benar:

```bash
# Verifikasi runtime
[PERINTAH VERSION — contoh: node --version]
# Output yang diharapkan: v[VERSI] atau lebih baru

# Verifikasi package manager
[PERINTAH VERSION — contoh: pnpm --version]
# Output yang diharapkan: [VERSI] atau lebih baru

# Verifikasi Git
git --version
# Output yang diharapkan: git version 2.30.x atau lebih baru

# Verifikasi Docker
docker --version
# Output yang diharapkan: Docker version [VERSI] atau lebih baru

docker compose version
# Output yang diharapkan: Docker Compose version v[VERSI] atau lebih baru
```

---

## 3. Clone Repository

### 3.1 Akses Repository

`[WAJIB]` Pastikan akun di [NAMA PLATFORM — contoh: GitHub / GitLab] sudah memiliki akses ke repository sebelum melakukan clone.

**Setup SSH Key (Direkomendasikan):**

```bash
# Buat SSH key jika belum ada
ssh-keygen -t ed25519 -C "[email anda]"

# Tampilkan public key untuk didaftarkan ke platform
cat ~/.ssh/id_ed25519.pub
# Daftarkan output ini ke:
# [NAMA PLATFORM]: Settings → SSH Keys → Add new key

# Verifikasi koneksi SSH
ssh -T git@[HOST PLATFORM — contoh: github.com]
# Output yang diharapkan: berhasil terautentikasi
```

### 3.2 Clone dan Masuk ke Direktori

```bash
# Clone repository
git clone [SSH URL REPOSITORY]
# atau jika menggunakan HTTPS:
# git clone [HTTPS URL REPOSITORY]

# Masuk ke direktori project
cd [NAMA DIREKTORI REPOSITORY]

# Pastikan berada di branch yang tepat
git branch
# Output yang diharapkan: * [NAMA BRANCH UTAMA — contoh: main atau develop]
```

### 3.3 Branch yang Digunakan untuk Development

Developer baru wajib mengerjakan task dari branch berikut:

```bash
# Ambil perubahan terbaru dari remote
git pull origin [BRANCH UTAMA — contoh: develop]

# Buat branch baru untuk task yang akan dikerjakan
git checkout -b feature/[nama-task]
```

Detail branching strategy ada di `docs/psd/04-coding-standard.md` Bagian Git Workflow.

---

## 4. Install Dependency

### 4.1 Install Dependency Backend

`[WAJIB]` Jalankan perintah berikut dari direktori root project (atau direktori backend jika monorepo):

```bash
cd [PATH BACKEND — contoh: . atau ./backend]

[PERINTAH INSTALL — contoh: pnpm install atau npm install]
```

**Yang terjadi saat perintah ini dijalankan:**
Package manager membaca file `[NAMA FILE — contoh: package.json]` dan mengunduh seluruh library yang dibutuhkan ke direktori `[DIREKTORI — contoh: node_modules/]`. File lock `[NAMA FILE LOCK — contoh: pnpm-lock.yaml]` memastikan versi library yang diinstal identik di semua mesin.

`[VERIFIKASI]` Instalasi berhasil jika tidak ada pesan `error` yang muncul. Pesan `warning` dapat diabaikan kecuali berkaitan dengan versi atau keamanan.

### 4.2 Install Dependency Frontend

`[KONDISIONAL]` Lakukan langkah ini hanya jika project memiliki frontend yang dikelola di repository yang sama.

```bash
cd [PATH FRONTEND — contoh: ./frontend]

[PERINTAH INSTALL]

# Kembali ke root setelah selesai
cd [PATH KEMBALI — contoh: ../]
```

### 4.3 Catatan Penting

- Jangan gunakan `sudo` saat menginstal dependency project. Jika muncul error permission, perbaiki ownership direktori.
- Jangan hapus atau edit file lock secara manual. File lock menjamin konsistensi versi library antar developer.
- Jika instalasi gagal di tengah jalan, hapus direktori `[DIREKTORI DEPENDENCY]` dan ulangi dari awal.

---

## 5. Setup Environment

### 5.1 Membuat File Environment

`[WAJIB]` File `.env` menyimpan seluruh konfigurasi yang berbeda antar environment (lokal, staging, production). File ini tidak di-commit ke repository karena menyimpan nilai yang sensitif.

```bash
# Salin file template ke file aktif
cp .env.example .env
```

File `.env.example` berisi seluruh variable yang dibutuhkan beserta keterangan. Buka file `.env` yang baru dibuat dan isi nilai yang diperlukan:

```bash
# Buka file dengan editor
[PERINTAH BUKA EDITOR — contoh: code .env atau nano .env]
```

### 5.2 Cara Mendapatkan Nilai yang Diperlukan

Tidak semua nilai di `.env` dapat diisi sendiri. Berikut panduan mendapatkan nilai per kategori:

| Kategori Variable | Cara Mendapatkan |
| --- | --- |
| Konfigurasi Database lokal | Tentukan sendiri; nilai harus sama dengan yang dikonfigurasi di `docker-compose.yml` |
| Secret key aplikasi | Generate menggunakan perintah di Bagian 8.1 |
| API key layanan eksternal | Minta kepada Tech Lead atau ambil dari sistem manajemen secret tim di [LOKASI — contoh: Vault, 1Password] |
| URL layanan eksternal mode sandbox | Lihat di dokumentasi masing-masing layanan atau minta Tech Lead |

### 5.3 File Environment untuk Struktur Monorepo

`[KONDISIONAL]` Jika project menggunakan monorepo dengan backend dan frontend terpisah:

```bash
cp [PATH BACKEND]/.env.example [PATH BACKEND]/.env
cp [PATH FRONTEND]/.env.example [PATH FRONTEND]/.env
```

---

## 6. Struktur Environment Project

### 6.1 Penjelasan Seluruh Variable

Berikut adalah seluruh variable environment yang digunakan project ini, dikelompokkan berdasarkan fungsi.

**Konfigurasi Aplikasi:**

| Variable | Tipe | Contoh Nilai | Keterangan |
| --- | --- | --- | --- |
| `APP_NAME` | `string` | `[NAMA APLIKASI]` | Nama aplikasi yang muncul di log dan notifikasi |
| `APP_ENV` | `enum` | `local` | Nilai valid: `local`, `development`, `staging`, `production` |
| `APP_PORT` | `integer` | `[PORT — contoh: 3000]` | Port HTTP server backend |
| `APP_DEBUG` | `boolean` | `true` | Aktifkan hanya di environment `local`; nonaktifkan di staging dan production |
| `APP_URL` | `string` | `http://localhost:[PORT]` | Base URL aplikasi tanpa trailing slash |
| `APP_KEY` | `string` | `[32 karakter hex]` | Secret key enkripsi; generate menggunakan perintah di Bagian 8.1 |

**Konfigurasi Database:**

| Variable | Tipe | Contoh Nilai | Keterangan |
| --- | --- | --- | --- |
| `DATABASE_HOST` | `string` | `localhost` | Host Database; `localhost` untuk Docker di mesin lokal |
| `DATABASE_PORT` | `integer` | `[PORT — contoh: 5432]` | Port Database; sesuaikan dengan konfigurasi di `docker-compose.yml` |
| `DATABASE_NAME` | `string` | `[NAMA]_local` | Nama Database; gunakan suffix `_local` untuk environment lokal |
| `DATABASE_USER` | `string` | `[USERNAME]` | Username akses Database; sesuaikan dengan `docker-compose.yml` |
| `DATABASE_PASSWORD` | `string` | `[PASSWORD]` | Password Database; sesuaikan dengan `docker-compose.yml` |
| `DATABASE_SSL` | `boolean` | `false` | `false` untuk lokal dan dev; `true` untuk staging dan production |

**Konfigurasi Cache:**

| Variable | Tipe | Contoh Nilai | Keterangan |
| --- | --- | --- | --- |
| `REDIS_HOST` | `string` | `localhost` | Host Redis |
| `REDIS_PORT` | `integer` | `6379` | Port Redis; default 6379 |
| `REDIS_PASSWORD` | `string` | *(kosong)* | Kosongkan untuk lokal; isi untuk staging dan production |
| `REDIS_DB` | `integer` | `0` | Index database Redis yang digunakan; default 0 |

**Konfigurasi Authentication:**

| Variable | Tipe | Contoh Nilai | Keterangan |
| --- | --- | --- | --- |
| `JWT_SECRET` | `string` | `[64 karakter hex]` | Secret key JWT; generate menggunakan perintah di Bagian 8.1 |
| `JWT_EXPIRES_IN` | `string` | `15m` | Durasi access token; format: `[N]s`, `[N]m`, `[N]h`, `[N]d` |
| `JWT_REFRESH_EXPIRES_IN` | `string` | `7d` | Durasi refresh token |

**Konfigurasi Queue:**

`[KONDISIONAL]` Isi bagian ini jika project menggunakan Queue.

| Variable | Tipe | Contoh Nilai | Keterangan |
| --- | --- | --- | --- |
| `QUEUE_DRIVER` | `string` | `redis` | Driver queue yang digunakan |
| `QUEUE_DEFAULT` | `string` | `default` | Nama queue default |

**Konfigurasi Integrasi Eksternal:**

`[KONDISIONAL]` Isi sesuai integrasi yang digunakan project ini. Nilai aktual diperoleh dari Tech Lead.

| Variable | Tipe | Cara Mendapatkan | Keterangan |
| --- | --- | --- | --- |
| `[NAMA_LAYANAN]_API_KEY` | `string` | Minta ke Tech Lead | API key untuk [nama layanan] |
| `[NAMA_LAYANAN]_BASE_URL` | `string` | Dokumentasi layanan | Base URL API [nama layanan] |
| `[NAMA_LAYANAN]_WEBHOOK_SECRET` | `string` | Minta ke Tech Lead | Secret validasi webhook dari [nama layanan] |

---

## 7. Setup Configuration

### 7.1 Generate Secret Key

`[WAJIB]` Generate nilai untuk `APP_KEY` dan `JWT_SECRET`. Setiap instalasi project harus memiliki nilai yang unik.

```bash
# Generate APP_KEY (32 byte hex)
openssl rand -hex 32
# Salin output ke APP_KEY di .env

# Generate JWT_SECRET (64 byte hex; lebih panjang untuk JWT)
openssl rand -hex 64
# Salin output ke JWT_SECRET di .env
```

Atau menggunakan perintah bawaan project jika tersedia:

```bash
[PERINTAH GENERATE KEY — contoh: npm run generate:key]
```

### 7.2 Konfigurasi Spesifik Project

`[KONDISIONAL]` Jika project memerlukan langkah konfigurasi tambahan selain environment variable, jalankan perintah berikut:

```bash
# Contoh: membuat direktori storage yang dibutuhkan aplikasi
mkdir -p [DIREKTORI YANG DIPERLUKAN — contoh: storage/logs storage/uploads]

# Contoh: menjalankan bootstrap konfigurasi framework
[PERINTAH BOOTSTRAP — contoh: npm run config:init]
```

---

## 8. Setup Database

### 8.1 Menjalankan Database via Docker Compose

`[WAJIB]` Docker Compose menjalankan seluruh service infrastruktur (Database, Cache, Queue) dalam container terisolasi.

```bash
# Pastikan Docker sudah berjalan
docker info
# Output yang diharapkan: informasi Docker tanpa error koneksi

# Jalankan seluruh service infrastruktur di background
docker compose up -d
```

`[VERIFIKASI]` Pastikan seluruh container berjalan:

```bash
docker compose ps
```

Output yang diharapkan: seluruh service berstatus `running` atau `healthy`. Tidak ada yang berstatus `exited` atau `restarting`.

**Apa yang dijalankan oleh `docker-compose.yml` project ini:**

| Service | Container Name | Port Lokal | Keterangan |
| --- | --- | --- | --- |
| [Database](about:blank#database) | `[NAMA CONTAINER]` | `[PORT HOST]:[PORT CONTAINER]` | [Deskripsi singkat] |
| [Cache] | `[NAMA CONTAINER]` | `[PORT HOST]:[PORT CONTAINER]` | [Deskripsi singkat] |
| [Queue] | `[NAMA CONTAINER]` | `[PORT HOST]:[PORT CONTAINER]` | [Deskripsi singkat] |

### 8.2 Menjalankan Database Tanpa Docker

`[KONDISIONAL]` Gunakan metode ini hanya jika Docker tidak tersedia di mesin lokal.

```bash
# Masuk ke CLI Database sebagai superuser
[PERINTAH LOGIN SUPERUSER — contoh: sudo -u postgres psql]

# Buat Database dan user untuk project ini
[PERINTAH CREATE DATABASE — contoh: CREATE DATABASE nama_database_local;]
[PERINTAH CREATE USER — contoh: CREATE USER username WITH PASSWORD 'password';]
[PERINTAH GRANT — contoh: GRANT ALL PRIVILEGES ON DATABASE nama_database_local TO username;]
[PERINTAH EXIT — contoh: \q]
```

Setelah Database dibuat, perbarui variable berikut di `.env`:

```bash
DATABASE_HOST=localhost
DATABASE_PORT=[PORT]
DATABASE_NAME=[NAMA DATABASE YANG BARU DIBUAT]
DATABASE_USER=[USERNAME YANG BARU DIBUAT]
DATABASE_PASSWORD=[PASSWORD YANG DITETAPKAN]
```

### 8.3 Verifikasi Koneksi Database

`[VERIFIKASI]` Uji koneksi ke Database sebelum melanjutkan:

```bash
[PERINTAH UJI KONEKSI — contoh: npm run db:check atau psql -h localhost -U username -d nama_database -c "SELECT 1"]
# Output yang diharapkan: berhasil terhubung tanpa error
```

---

## 9. Migration

### 9.1 Apa yang Dilakukan Migration

Migration membuat seluruh struktur tabel Database yang dibutuhkan aplikasi. Setiap file migration mencerminkan satu perubahan skema yang terdokumentasi. Detail struktur yang dibuat ada di `docs/tsd/01-database-design.md`.

### 9.2 Menjalankan Migration

`[WAJIB]` Jalankan setelah Database berhasil terkoneksi:

```bash
[PERINTAH MIGRATION — contoh: npm run db:migrate atau npx typeorm migration:run]
```

Output yang diharapkan: daftar file migration yang dieksekusi satu per satu, masing-masing dengan konfirmasi berhasil. Tidak ada baris `error` atau `failed`.

`[VERIFIKASI]` Periksa status migration:

```bash
[PERINTAH STATUS MIGRATION — contoh: npm run db:migrate:status]
# Output yang diharapkan: seluruh migration berstatus "executed" atau "applied"
# Tidak ada migration dengan status "pending"
```

### 9.3 Rollback Migration

`[KONDISIONAL]` Gunakan hanya di environment `local` saat perlu membatalkan migration terakhir:

```bash
# Rollback satu batch terakhir
[PERINTAH ROLLBACK — contoh: npm run db:rollback]

# Reset seluruh migration (hapus semua tabel dan jalankan ulang dari awal)
[PERINTAH RESET — contoh: npm run db:fresh]
```

> **Peringatan:** Perintah reset akan menghapus seluruh data di Database lokal. Jangan jalankan di environment `staging` atau `production`.
> 

---

## 10. Seed Data

### 10.1 Apa yang Dilakukan Seeder

Seeder mengisi Database dengan data awal yang dibutuhkan agar aplikasi dapat digunakan selama development. Data ini mencakup dua jenis:

| Jenis Seed | Isi | Harus Dijalankan |
| --- | --- | --- |
| Data master | Data referensi yang dibutuhkan sistem untuk berjalan (Role, konfigurasi default, tipe status) | `[WAJIB]` |
| Data dummy | Contoh data untuk testing fitur (user contoh, transaksi contoh) | `[OPSIONAL]` |

### 10.2 Menjalankan Seeder

```bash
# Jalankan seluruh seeder (data master + data dummy)
[PERINTAH SEED — contoh: npm run db:seed]

# Jalankan hanya data master (tanpa data dummy)
[PERINTAH SEED MASTER — contoh: npm run db:seed:master atau npm run db:seed -- --only=master]

# Reset Database, jalankan ulang migration, dan seed (sering digunakan saat development)
[PERINTAH FRESH SEED — contoh: npm run db:fresh:seed]
```

### 10.3 Akun Default Setelah Seed

Setelah seed berhasil, akun berikut tersedia untuk digunakan di environment `local`:

| Role | Email / Username | Password | Catatan |
| --- | --- | --- | --- |
| `[ROLE_1 — contoh: SUPER_ADMIN]` | `[EMAIL]` | `[PASSWORD]` | Akses penuh ke seluruh fitur |
| `[ROLE_2 — contoh: ADMIN]` | `[EMAIL]` | `[PASSWORD]` | [Akses spesifik] |
| `[ROLE_3 — contoh: USER]` | `[EMAIL]` | `[PASSWORD]` | Akses standar pengguna |

> **Penting:** Password di atas hanya berlaku di environment `local`. Jangan gunakan akun ini di staging atau production.
> 

---

## 11. Menjalankan Backend

### 11.1 Mode Development

`[WAJIB]` Jalankan backend dalam mode development yang mendukung hot-reload (restart otomatis saat file berubah):

```bash
# Pastikan berada di direktori backend
cd [PATH BACKEND — contoh: . atau ./backend]

# Jalankan
[PERINTAH START DEV — contoh: npm run start:dev atau pnpm dev]
```

Output yang diharapkan saat startup berhasil:

```
[Contoh output startup yang menandakan aplikasi siap]
[Contoh: Application listening on port 3000]
[Contoh: Database connection established]
[Contoh: [Module] initialized]
```

Jika output menyebutkan error koneksi Database atau service tidak ditemukan, periksa kembali apakah container Docker sudah berjalan (`docker compose ps`).

### 11.2 Port dan URL Backend

Backend berjalan di port yang dikonfigurasi melalui `APP_PORT` di `.env`. Default port project ini adalah `[PORT DEFAULT]`.

| URL | Keterangan |
| --- | --- |
| `http://localhost:[PORT]` | Root aplikasi |
| `http://localhost:[PORT]/api/v1` | Base URL seluruh endpoint API |
| `http://localhost:[PORT]/api/health` | Health check endpoint |

### 11.3 Mengubah Port Backend

Jika port default sudah digunakan proses lain, ubah nilai `APP_PORT` di `.env` ke port yang kosong, kemudian restart backend.

```bash
# Cek proses yang menggunakan port tertentu
lsof -i :[PORT]
```

---

## 12. Menjalankan Frontend

`[KONDISIONAL]` Lakukan hanya jika project memiliki frontend yang dikelola di repository ini.

### 12.1 Mode Development

```bash
# Masuk ke direktori frontend
cd [PATH FRONTEND — contoh: ./frontend atau ./client]

# Jalankan development server
[PERINTAH START FRONTEND — contoh: npm run dev atau pnpm dev]
```

Output yang diharapkan: URL akses frontend, biasanya `http://localhost:[PORT FRONTEND]`.

### 12.2 Konfigurasi URL Backend

Frontend menggunakan variable environment untuk mengetahui URL backend. Pastikan nilai berikut di `.env` frontend sudah diisi dengan benar:

```bash
# Path: [PATH FRONTEND]/.env

[VARIABLE API URL — contoh: VITE_API_BASE_URL=http://localhost:[PORT BACKEND]/api/v1]
# atau
[VARIABLE API URL — contoh: NEXT_PUBLIC_API_URL=http://localhost:[PORT BACKEND]/api/v1]
```

---

## 13. Menjalankan Scheduler dan Queue

`[KONDISIONAL]` Lakukan hanya jika project menggunakan Scheduler atau Queue Worker.

### 13.1 Scheduler

Scheduler menjalankan job terjadwal secara otomatis. Di environment `local`, Scheduler tidak harus dijalankan kecuali sedang mengerjakan fitur yang berkaitan.

**Job terjadwal yang ada dalam project ini:**

| Nama Job | Jadwal | Fungsi |
| --- | --- | --- |
| `[NAMA JOB — contoh: CleanExpiredSessions]` | `[CRON — contoh: Setiap hari pukul 00:00]` | [Fungsi konkret — contoh: Menghapus sesi yang sudah expire dari tabel sessions] |
| `[NAMA JOB]` | `[JADWAL]` | [Fungsi] |

```bash
# Jalankan Scheduler (berjalan di background)
[PERINTAH SCHEDULER — contoh: npm run scheduler atau npm run start:scheduler]
```

### 13.2 Queue Worker

Queue Worker memproses job asinkron dari antrian. Di environment `local`, Queue Worker tidak wajib dijalankan kecuali sedang mengerjakan fitur yang mengirim job ke antrian.

**Queue yang ada dalam project ini:**

| Nama Queue | Jenis Job yang Diproses |
| --- | --- |
| `[NAMA QUEUE — contoh: email-notification]` | [Contoh: Pengiriman email selamat datang, notifikasi approval] |
| `[NAMA QUEUE]` | [Jenis job] |

```bash
# Jalankan Queue Worker
[PERINTAH QUEUE — contoh: npm run queue:work atau npm run start:queue]

# Jalankan Queue Worker untuk antrian tertentu
[PERINTAH QUEUE SPESIFIK — contoh: npm run queue:work -- --queue=email-notification]
```

> Queue Worker harus dijalankan di terminal terpisah dari backend.
> 

---

## 14. Validasi Setup Berhasil

`[VERIFIKASI]` Jalankan seluruh pemeriksaan berikut setelah menyelesaikan semua langkah setup. Semua item harus lulus sebelum mulai development.

### Pemeriksaan 1: Service Infrastruktur Berjalan

```bash
docker compose ps
```

Yang diharapkan: Seluruh container berstatus `running` atau `healthy`.

---

### Pemeriksaan 2: Koneksi Database

```bash
[PERINTAH CEK KONEKSI — contoh: npm run db:check]
```

Yang diharapkan: Konfirmasi koneksi berhasil tanpa error.

---

### Pemeriksaan 3: Migration Sudah Diterapkan

```bash
[PERINTAH STATUS MIGRATION]
```

Yang diharapkan: Tidak ada migration dengan status `pending`.

---

### Pemeriksaan 4: Backend Merespons

```bash
curl http://localhost:[PORT]/api/health
```

Yang diharapkan:

```json
{
  "status": "ok",
  "database": "connected",
  "timestamp": "[WAKTU SAAT INI]"
}
```

---

### Pemeriksaan 5: Login dengan Akun Default Berhasil

```bash
curl -X POST http://localhost:[PORT]/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "[EMAIL DEFAULT]", "password": "[PASSWORD DEFAULT]"}'
```

Yang diharapkan: Response HTTP 200 dengan `accessToken` dan `refreshToken` dalam body.

---

### Pemeriksaan 6: Frontend Dapat Diakses

`[KONDISIONAL]` Hanya jika project memiliki frontend.

Buka browser dan akses `http://localhost:[PORT FRONTEND]`. Yang diharapkan: halaman utama tampil tanpa error kritis di browser console.

---

### Pemeriksaan 7: Test Suite Lulus

```bash
[PERINTAH TEST — contoh: npm run test atau pnpm test]
```

Yang diharapkan: Seluruh test lulus. Tidak ada test dengan status `failed`.

---

## 15. Common Command

Referensi cepat perintah yang paling sering digunakan selama development.

### Manajemen Service Docker

```bash
# Jalankan seluruh service infrastruktur
docker compose up -d

# Hentikan seluruh service
docker compose down

# Hentikan dan hapus data volume (reset penuh)
docker compose down -v

# Lihat status container
docker compose ps

# Lihat log container tertentu
docker compose logs -f [NAMA SERVICE — contoh: postgres atau redis]
```

### Menjalankan Aplikasi

```bash
# Backend (mode development)
[PERINTAH START DEV]

# Frontend (mode development)
[PERINTAH START FRONTEND DEV]

# Backend dan Frontend sekaligus (jika tersedia)
[PERINTAH START ALL — contoh: npm run dev:all]

# Queue Worker
[PERINTAH QUEUE WORK]

# Scheduler
[PERINTAH SCHEDULER]
```

### Database

```bash
# Jalankan migration
[PERINTAH MIGRATE]

# Lihat status migration
[PERINTAH MIGRATE STATUS]

# Rollback migration terakhir
[PERINTAH ROLLBACK]

# Reset Database dan jalankan ulang (hapus semua data)
[PERINTAH FRESH]

# Reset Database, migration ulang, dan seed
[PERINTAH FRESH SEED]

# Buat file migration baru
[PERINTAH BUAT MIGRATION — contoh: npm run migration:create -- --name=create_orders_table]
```

### Testing dan Kualitas Kode

```bash
# Jalankan seluruh test
[PERINTAH TEST]

# Jalankan test dengan coverage report
[PERINTAH TEST COVERAGE]

# Jalankan test dalam mode watch
[PERINTAH TEST WATCH]

# Jalankan linter
[PERINTAH LINT]

# Perbaiki masalah linting otomatis
[PERINTAH LINT FIX]

# Jalankan type check
[PERINTAH TYPE CHECK]
```

---

## 16. Common Setup Issue

### Issue 1: Container Docker tidak mau berjalan

**Gejala:**

```
docker compose ps
```

Menampilkan container dengan status `exited` atau `restarting`.

**Diagnosis:**

```bash
# Lihat log container yang bermasalah
docker compose logs [NAMA SERVICE]
```

**Kemungkinan penyebab dan solusinya:**

**Port sudah digunakan proses lain:**

```bash
# Cek proses yang menggunakan port tersebut
lsof -i :[PORT — contoh: 5432]
# Hentikan proses tersebut, atau ubah port di docker-compose.yml
```

**Volume dari sesi sebelumnya korup:**

```bash
# Hapus volume dan buat ulang
docker compose down -v
docker compose up -d
```

---

### Issue 2: Error `Connection refused` ke Database

**Gejala:** Backend menampilkan error koneksi ke Database saat startup.

**Diagnosis bertahap:**

```bash
# Langkah 1: Pastikan container Database berjalan
docker compose ps
# Jika tidak running, jalankan: docker compose up -d

# Langkah 2: Pastikan nilai DATABASE_HOST, PORT, USER, PASSWORD di .env
# sesuai dengan konfigurasi di docker-compose.yml
cat .env | grep DATABASE
cat docker-compose.yml | grep -A 10 [NAMA SERVICE DATABASE]

# Langkah 3: Uji koneksi langsung ke Database
[PERINTAH KONEKSI CLI — contoh: psql -h localhost -U username -d nama_database]
```

---

### Issue 3: Error `Missing required environment variable`

**Gejala:** Aplikasi gagal startup dengan pesan variable environment tidak ditemukan.

**Solusi:**

```bash
# Pastikan file .env ada
ls -la .env

# Jika tidak ada, buat dari template
cp .env.example .env

# Bandingkan .env dengan .env.example untuk menemukan variable yang belum ada
diff .env.example .env
```

Variable yang muncul di `.env.example` namun tidak ada di `.env` adalah yang perlu ditambahkan.

---

### Issue 4: Migration gagal dengan error tabel sudah ada

**Gejala:** Perintah migration menghasilkan error `relation already exists` atau `table already exists`.

**Penyebab:** Migration pernah dijalankan sebagian dan gagal di tengah jalan.

**Solusi di environment `local`:**

```bash
# Reset penuh: hapus semua tabel dan jalankan ulang dari awal
[PERINTAH FRESH SEED]
```

> **Catatan:** Seluruh data di Database lokal akan hilang. Jangan gunakan di staging atau production.
> 

---

### Issue 5: `pnpm install` atau `npm install` gagal karena permission

**Gejala:**

```
EACCES: permission denied
```

**Solusi:**

```bash
# Perbaiki ownership direktori (jangan gunakan sudo untuk install)
sudo chown -R $USER:$GROUP ~/.npm
sudo chown -R $USER:$GROUP ./node_modules

# Hapus node_modules dan instal ulang
rm -rf node_modules
[PERINTAH INSTALL]
```

---

### Issue 6: Port backend sudah digunakan

**Gejala:**

```
Error: listen EADDRINUSE: address already in use :[PORT]
```

**Solusi:**

```bash
# Temukan proses yang menggunakan port tersebut
lsof -i :[PORT]

# Hentikan proses (ganti PID dengan angka dari output di atas)
kill -9 [PID]

# Atau ubah APP_PORT di .env ke port lain yang kosong
```

---

### Mendapatkan Bantuan

Jika masalah tidak ada di daftar ini:

1. Baca pesan error secara lengkap — baris pertama biasanya paling informatif.
2. Periksa apakah ada issue terkait di repository project.
3. Hubungi Tech Lead di [CHANNEL KOMUNIKASI TIM] dengan menyertakan:
    - Langkah yang sedang dikerjakan
    - Perintah yang dijalankan
    - Output error lengkap
    - Output dari `docker compose ps` dan `node --version` (atau runtime yang digunakan)
    - Sistem operasi dan versi