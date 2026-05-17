# 04 - Deployment Guide

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Deployment
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

Dokumen ini menjelaskan seluruh proses deployment project dari build hingga verifikasi. Baca berdasarkan kebutuhan:

- **Pertama kali deploy ke environment baru** — Baca dari Bagian 1 hingga 5 secara berurutan
- **Menjalankan deployment rutin** — Langsung ke Bagian 5 (Deployment Process) dan Bagian 14 (Deployment Checklist)
- **Menangani deployment yang gagal** — Langsung ke Bagian 6 (Rollback Process)
- **Menambah migration ke deployment** — Bagian 7 (Migration Deployment)
- **Investigasi masalah pasca-deploy** — Bagian 10 (Logging) dan Bagian 11 (Monitoring)
- **Memahami backup dan recovery** — Bagian 13 (Backup Strategy)

**Asumsi:** Dokumen ini mengasumsikan pembaca sudah memahami arsitektur sistem dari `docs/psd/03-system-architecture.md` dan memiliki akses yang sesuai ke environment target.

---

## Daftar Isi

1. [Deployment Overview](#1-deployment-overview)
2. [Environment Overview](#2-environment-overview)
3. [Configuration Overview](#3-configuration-overview)
4. [Build Process](#4-build-process)
5. [Deployment Process](#5-deployment-process)
6. [Rollback Process](#6-rollback-process)
7. [Migration Deployment](#7-migration-deployment)
8. [Scheduler Deployment](#8-scheduler-deployment)
9. [Queue Deployment](#9-queue-deployment)
10. [Restart Service](#10-restart-service)
11. [Logging Location](#11-logging-location)
12. [Monitoring Overview](#12-monitoring-overview)
13. [Verification Deployment](#13-verification-deployment)
14. [Backup Strategy](#14-backup-strategy)
15. [Deployment Checklist](#15-deployment-checklist)
16. [Dependency Deployment](#16-dependency-deployment)

---

## 1. Deployment Overview

### 1.1 Gambaran Proses Deployment

Project ini mengikuti alur deployment satu arah: dari kode di repository menuju environment production. Setiap perubahan kode harus melewati seluruh tahap secara berurutan; tidak ada mekanisme deploy langsung ke production melewati staging.

```
Branch feature/fix
        |
        v
Pull Request ke branch develop
        |
        v
CI Pipeline (lint, test, build check)
        |
        +-- Gagal → PR ditolak; developer memperbaiki
        |
        +-- Sukses → Code review Tech Lead
                        |
                        v
                   Merge ke develop
                        |
                        v
              Auto-deploy ke Development
                        |
                        v
                   QA Testing
                        |
                        v
                Merge ke staging
                        |
                        v
              Auto-deploy ke Staging
                        |
                        v
        User Acceptance Testing (UAT) + Approval
                        |
                        v
                Merge ke main
                        |
                        v
          Manual deploy ke Production (Tech Lead)
```

### 1.2 Komponen yang Di-deploy

Setiap deployment mencakup komponen berikut. Seluruh komponen di-deploy dalam satu unit kecuali dinyatakan sebaliknya.

| Komponen | Mekanisme Deploy | Dijalankan Terpisah |
| --- | --- | --- |
| Backend API | [MEKANISME — contoh: Docker container / PM2 / systemd] | Tidak |
| Scheduler | [MEKANISME — contoh: Berjalan dalam proses yang sama / container terpisah] | [Ya / Tidak] |
| Queue Worker | [MEKANISME — contoh: Container terpisah / proses terpisah via PM2] | Ya |
| Migration | Script yang dijalankan satu kali sebelum service dimulai | Ya |
| Database | Tidak di-deploy; sudah berjalan di infrastruktur | — |
| Cache | Tidak di-deploy; sudah berjalan di infrastruktur | — |

### 1.3 Siapa yang Berwenang Melakukan Deployment

| Environment | Siapa yang Boleh Deploy | Mekanisme |
| --- | --- | --- |
| development | CI/CD pipeline (otomatis) | Auto-deploy saat merge ke `develop` |
| staging | CI/CD pipeline (otomatis) | Auto-deploy saat merge ke `staging` |
| production | Tech Lead (manual, approval Product Owner) | Manual trigger setelah approval |

Hotfix ke production memerlukan persetujuan Tech Lead dan Product Owner, dan mengikuti prosedur di Bagian 5.4.

---

## 2. Environment Overview

### 2.1 Daftar Environment

Project ini memiliki tiga environment aktif dengan tujuan yang berbeda.

| Environment | Branch | URL | Tujuan |
| --- | --- | --- | --- |
| development | `develop` | [URL DEV] | Testing fitur oleh tim QA; integrasi antar fitur |
| staging | `staging` | [URL STAGING] | UAT oleh stakeholder; mirror mendekati production |
| production | `main` | [URL PRODUCTION] | Pengguna akhir; environment aktif |

### 2.2 Perbedaan Konfigurasi Antar Environment

| Aspek | development | staging | production |
| --- | --- | --- | --- |
| Database | [NAMA DB DEV] | [NAMA DB STAGING] | [NAMA DB PROD] |
| Log Level | `DEBUG` | `INFO` | `WARN` |
| Email pengiriman | Ditangkap Mailtrap / ditolak | Dikirim ke akun test | Dikirim ke alamat nyata |
| Rate limiting | Longgar atau dinonaktifkan | Sama dengan production | Aktif sesuai konfigurasi |
| Queue processing | Aktif | Aktif | Aktif |
| Scheduler | Aktif | Aktif | Aktif |
| Data | Data dummy / seeder | Data dummy mendekati realistis | Data produksi nyata |

### 2.3 Infrastruktur per Environment

| Komponen | development | staging | production |
| --- | --- | --- | --- |
| Server | [SPESIFIKASI — contoh: 2 vCPU, 4GB RAM] | [SPESIFIKASI] | [SPESIFIKASI] |
| Database | [HOST:PORT] | [HOST:PORT] | [HOST:PORT] |
| Cache | [HOST:PORT] | [HOST:PORT] | [HOST:PORT] |
| Queue | [HOST:PORT] | [HOST:PORT] | [HOST:PORT] |
| Object Storage | [OPSIONAL / NAMA LAYANAN] | [NAMA LAYANAN] | [NAMA LAYANAN] |
| Domain | [DOMAIN DEV] | [DOMAIN STAGING] | [DOMAIN PRODUCTION] |
| SSL | Self-signed / Let’s Encrypt | Let’s Encrypt | [PROVIDER CERT] |

### 2.4 Akses ke Environment

Akses server dilakukan melalui SSH dengan key yang sudah terdaftar. Tidak ada akses password langsung ke server.

```bash
# Akses ke development server
ssh [USER]@[HOST DEV] -i [PATH KEY]

# Akses ke staging server
ssh [USER]@[HOST STAGING] -i [PATH KEY]

# Akses ke production server
# Hanya Tech Lead yang memiliki key ini
ssh [USER]@[HOST PRODUCTION] -i [PATH KEY]
```

Untuk mendapatkan akses, hubungi Tech Lead atau DevOps Engineer di [CHANNEL KOMUNIKASI].

---

## 3. Configuration Overview

### 3.1 Sumber Konfigurasi

Konfigurasi project berasal dari environment variables. Tidak ada nilai konfigurasi yang di-hardcode dalam kode.

| Environment | Sumber Konfigurasi |
| --- | --- |
| local | File `.env` di direktori root (tidak di-commit ke repository) |
| development | [MEKANISME — contoh: Secret Manager / CI/CD environment variables / file `.env` di server] |
| staging | [MEKANISME] |
| production | [MEKANISME] |

### 3.2 Variabel Konfigurasi Wajib

Seluruh variabel berikut harus terdefinisi sebelum aplikasi dapat berjalan. Aplikasi akan gagal startup jika ada yang kosong.

**Aplikasi:**

| Variabel | Contoh Nilai | Keterangan |
| --- | --- | --- |
| `APP_ENV` | `production` | Identifier environment aktif |
| `APP_PORT` | `3000` | Port yang didengarkan aplikasi |
| `APP_URL` | `https://[DOMAIN]` | URL publik aplikasi; digunakan untuk generate link |
| `APP_SECRET` | `[STRING ACAK PANJANG]` | Secret untuk enkripsi internal |

**Database:**

| Variabel | Contoh Nilai | Keterangan |
| --- | --- | --- |
| `DATABASE_HOST` | `[HOST]` | Host Database |
| `DATABASE_PORT` | `5432` | Port Database |
| `DATABASE_NAME` | `[NAMA DB]` | Nama Database |
| `DATABASE_USER` | `[USER]` | Username Database |
| `DATABASE_PASSWORD` | `[PASSWORD]` | Password Database; wajib kuat di production |

**Cache:**

| Variabel | Contoh Nilai | Keterangan |
| --- | --- | --- |
| `REDIS_HOST` | `[HOST]` | Host Redis |
| `REDIS_PORT` | `6379` | Port Redis |
| `REDIS_PASSWORD` | `[PASSWORD]` | Password Redis; wajib di production |

**Authentication:**

| Variabel | Contoh Nilai | Keterangan |
| --- | --- | --- |
| `JWT_SECRET` | `[STRING ACAK PANJANG]` | Secret untuk signing JWT access token |
| `JWT_EXPIRES_IN` | `15m` | Masa berlaku access token |
| `JWT_REFRESH_SECRET` | `[STRING ACAK LAIN]` | Secret untuk signing JWT refresh token |
| `JWT_REFRESH_EXPIRES_IN` | `7d` | Masa berlaku refresh token |

**Integrasi Eksternal:**

| Variabel | Contoh Nilai | Keterangan |
| --- | --- | --- |
| `[NAMA LAYANAN]_API_KEY` | `[KEY]` | API key layanan eksternal |
| `[NAMA LAYANAN]_BASE_URL` | `[URL]` | Base URL layanan eksternal |

### 3.3 Cara Memvalidasi Konfigurasi

Jalankan perintah berikut untuk memverifikasi seluruh variabel konfigurasi sudah terdefinisi sebelum deployment:

```bash
# Verifikasi dari dalam direktori project
[PERINTAH VALIDASI ENV — contoh: node -e "require('./src/config/env.validation')" atau npm run config:check]
```

Output yang diharapkan: konfirmasi bahwa seluruh variabel wajib sudah terdefinisi. Jika ada yang kurang, aplikasi menampilkan daftar variabel yang hilang dan menolak berjalan.

---

## 4. Build Process

### 4.1 Gambaran Build

Sebelum di-deploy, source code harus melalui proses build untuk menghasilkan artifact yang siap dijalankan di server. Build dilakukan oleh CI/CD pipeline secara otomatis; build manual hanya dilakukan untuk debugging.

### 4.2 Langkah Build

Build dilakukan dalam urutan berikut:

```bash
# [1] Install dependency (menggunakan versi yang terkunci di file lock)
[PERINTAH INSTALL — contoh: pnpm install --frozen-lockfile]

# [2] Jalankan type check (hanya jika menggunakan TypeScript)
[PERINTAH TYPE CHECK — contoh: pnpm run type-check]

# [3] Jalankan linter
[PERINTAH LINT — contoh: pnpm run lint]

# [4] Jalankan test suite
[PERINTAH TEST — contoh: pnpm run test]

# [5] Build artifact produksi
[PERINTAH BUILD — contoh: pnpm run build]
```

Jika salah satu langkah gagal, proses build berhenti dan deployment tidak dilanjutkan.

### 4.3 Output Build

| Artifact | Lokasi | Keterangan |
| --- | --- | --- |
| Build utama | `[DIREKTORI — contoh: dist/]` | File yang siap dijalankan di production |
| Docker image | `[REGISTRY]/[IMAGE-NAME]:[TAG]` | Image yang di-push ke container registry |

Tag image mengikuti format: `[NAMA-IMAGE]:[VERSI-SEMVER]-[SHORT-COMMIT-HASH]`

Contoh: `project-api:1.4.2-a3f8c91`

### 4.4 Build di CI/CD Pipeline

CI/CD pipeline menjalankan build secara otomatis setiap ada push ke branch `develop`, `staging`, dan `main`. Konfigurasi pipeline ada di [PATH FILE CI — contoh: `.github/workflows/ci.yml` atau `.gitlab-ci.yml`].

Untuk melihat riwayat build: [URL CI/CD PLATFORM].

---

## 5. Deployment Process

### 5.1 Deployment ke Development

Deployment ke development berjalan otomatis setelah merge ke branch `develop` dan CI pipeline sukses.

```
Merge ke develop
      |
      v
CI Pipeline berjalan (±[N] menit)
      |
      v
Image baru di-push ke registry
      |
      v
Server development pull image terbaru
      |
      v
Container lama dihentikan; container baru dijalankan
      |
      v
Migration otomatis dijalankan (jika dikonfigurasi)
      |
      v
Health check memverifikasi service berjalan
```

Jika ada kegagalan, Tech Lead mendapat notifikasi di [CHANNEL NOTIFIKASI].

### 5.2 Deployment ke Staging

Deployment ke staging berjalan otomatis setelah merge ke branch `staging` dan CI pipeline sukses. Prosesnya identik dengan development. Data di staging tidak di-reset saat deployment kecuali ada instruksi eksplisit dari Tech Lead.

### 5.3 Deployment ke Production

Deployment ke production dilakukan secara manual oleh Tech Lead setelah mendapat approval dari Product Owner. Tidak ada auto-deploy ke production.

**Prasyarat sebelum deploy production:**

```
[ ] UAT di staging sudah selesai dan disetujui Product Owner
[ ] Tidak ada open bug blocker dari QA
[ ] Tech Lead sudah mereview seluruh perubahan
[ ] Deployment checklist di Bagian 15 sudah dilengkapi
[ ] Backup Database production sudah dikonfirmasi (Bagian 14)
[ ] Jendela deployment disepakati: hari kerja, jam [JAM DEPLOYMENT PRODUCTION]
[ ] Tim on-call tersedia selama [N] jam setelah deployment
```

**Langkah deployment production:**

```bash
# [1] Masuk ke server production
ssh [USER]@[HOST PRODUCTION] -i [PATH KEY]

# [2] Masuk ke direktori deployment
cd [PATH DIREKTORI DEPLOYMENT]

# [3] Pull image terbaru
[PERINTAH PULL IMAGE — contoh: docker pull [REGISTRY]/[IMAGE]:[TAG]]

# [4] Jalankan migration (SEBELUM mengganti container yang berjalan)
[PERINTAH MIGRATION — contoh: docker run --rm --env-file .env [IMAGE]:[TAG] [PERINTAH MIGRATE]]

# [5] Ganti container yang berjalan dengan yang baru
[PERINTAH DEPLOY — contoh: docker compose up -d --no-deps api]

# [6] Verifikasi service berjalan
[PERINTAH HEALTH CHECK — contoh: curl -f http://localhost:[PORT]/health]

# [7] Verifikasi log tidak ada error kritis
[PERINTAH LIHAT LOG — contoh: docker logs --tail 50 [CONTAINER NAME]]
```

### 5.4 Hotfix ke Production

Hotfix adalah perubahan darurat langsung ke production untuk memperbaiki bug kritis yang tidak dapat menunggu siklus deployment normal.

**Kapan hotfix diizinkan:**
- Bug menyebabkan fitur kritikal tidak berfungsi sama sekali
- Kerentanan keamanan yang sudah dieksploitasi atau sangat berisiko
- Data corruption yang aktif terjadi

**Prosedur hotfix:**

```bash
# [1] Buat branch hotfix dari main (bukan dari develop)
git checkout main
git pull origin main
git checkout -b hotfix/[deskripsi-singkat]

# [2] Implementasi perbaikan, commit, dan push
git commit -m "hotfix: [deskripsi perbaikan]"
git push origin hotfix/[deskripsi-singkat]

# [3] Buat PR ke main; minta approval Tech Lead
# [4] Setelah merge, deploy ke production menggunakan prosedur Bagian 5.3
# [5] Setelah production stabil, merge hotfix ke develop juga
git checkout develop
git merge hotfix/[deskripsi-singkat]
git push origin develop
```

---

## 6. Rollback Process

### 6.1 Kapan Rollback Dilakukan

Rollback dilakukan jika setelah deployment ditemukan kondisi berikut:

- Health check gagal dan service tidak bisa dijalankan
- Error rate melonjak di atas threshold monitoring (lihat Bagian 12)
- Bug kritis ditemukan yang tidak ada di versi sebelumnya
- Performa menurun signifikan dibanding baseline sebelum deploy

### 6.2 Prosedur Rollback Aplikasi

Rollback aplikasi dilakukan dengan menjalankan kembali image versi sebelumnya. Tag image versi sebelumnya dapat dilihat di container registry atau catatan deployment.

```bash
# [1] Identifikasi tag image versi sebelumnya
# Lihat di: [URL CONTAINER REGISTRY] atau catatan deployment

# [2] Masuk ke server
ssh [USER]@[HOST TARGET] -i [PATH KEY]

# [3] Ganti ke image versi sebelumnya
[PERINTAH — contoh: docker compose up -d --no-deps api [REGISTRY]/[IMAGE]:[TAG-LAMA]]

# [4] Verifikasi service berjalan dengan image lama
[PERINTAH HEALTH CHECK]

# [5] Verifikasi log tidak ada error baru
[PERINTAH LIHAT LOG]
```

### 6.3 Prosedur Rollback Migration Database

> **PENTING:** Rollback migration Database adalah operasi berisiko tinggi. Jangan lakukan tanpa koordinasi dengan Tech Lead dan pastikan backup tersedia.
> 

Rollback migration hanya dilakukan jika migration yang baru dijalankan terbukti menyebabkan masalah dan belum ada data baru yang masuk ke kolom/tabel yang terpengaruh.

```bash
# [1] Pastikan backup Database terbaru tersedia dan dapat di-restore
# [2] Jalankan rollback migration
[PERINTAH ROLLBACK MIGRATION — contoh: npm run migration:revert atau node migrate down]

# [3] Verifikasi skema Database kembali ke kondisi sebelumnya
[PERINTAH CEK SKEMA — contoh: psql -c "\d [NAMA TABEL]"]
```

Jika data sudah masuk ke skema yang baru dan tidak dapat di-rollback dengan aman, opsi terbaik adalah forward fix: buat migration baru yang memperbaiki masalah, bukan rollback.

### 6.4 Rollback Konfigurasi

Jika masalah disebabkan oleh perubahan konfigurasi (environment variable), rollback dilakukan dengan mengembalikan nilai variabel ke nilai sebelumnya dan me-restart service.

```bash
# [1] Perbarui nilai environment variable ke nilai sebelumnya
# [MEKANISME SESUAI ENV — contoh: update di Secret Manager atau edit file .env]

# [2] Restart service
# Lihat prosedur restart di Bagian 10
```

---

## 7. Migration Deployment

### 7.1 Posisi Migration dalam Alur Deployment

Migration Database dijalankan **setelah** image baru tersedia tetapi **sebelum** container aplikasi baru dijalankan. Urutan ini penting untuk menjaga konsistensi antara skema Database dan kode aplikasi.

```
Image baru tersedia di registry
        |
        v
Jalankan migration (container sementara)
        |
        +-- Gagal → STOP; jangan lanjutkan; jalankan rollback migration
        |
        +-- Sukses → Jalankan container aplikasi baru
                         |
                         v
                  Verifikasi aplikasi
```

### 7.2 Menjalankan Migration

```bash
# Jalankan migration menggunakan image yang sama dengan yang akan di-deploy
# Ini memastikan migration dan kode berjalan di versi yang sama
[PERINTAH — contoh: docker run --rm \
  --env-file /path/to/.env \
  [REGISTRY]/[IMAGE]:[TAG] \
  [PERINTAH MIGRATE — contoh: node dist/scripts/migrate.js up]]
```

### 7.3 Memverifikasi Status Migration

Setelah migration selesai, verifikasi bahwa seluruh migration sudah teraplikasikan:

```bash
[PERINTAH CEK STATUS — contoh: docker run --rm \
  --env-file /path/to/.env \
  [REGISTRY]/[IMAGE]:[TAG] \
  [PERINTAH STATUS — contoh: node dist/scripts/migrate.js status]]
```

Output yang diharapkan: semua migration dalam status `applied`. Jika ada yang `pending`, jalankan ulang perintah migration.

### 7.4 Aturan Penulisan Migration untuk Deployment

Migration yang masuk ke production harus mengikuti aturan berikut agar aman dijalankan tanpa downtime:

| Aturan | Alasan |
| --- | --- |
| Hanya additive — tambah kolom baru, jangan hapus atau rename | Kode lama yang masih berjalan tidak rusak saat migration sedang dijalankan |
| Kolom baru harus nullable atau memiliki default value | Migration bisa selesai tanpa mengisi seluruh baris yang sudah ada |
| Tidak ada data transformation yang besar dalam satu migration | Migration yang berjalan lama mengunci tabel dan menyebabkan timeout |
| Hapus kolom di migration terpisah, beberapa sprint setelah kolom tidak digunakan | Memberikan waktu untuk verifikasi bahwa tidak ada kode yang masih membaca kolom lama |

---

## 8. Scheduler Deployment

### 8.1 Gambaran Scheduler

Scheduler menjalankan job terjadwal secara periodik. Daftar job scheduler dan jadwalnya:

| Nama Job | Jadwal (cron) | Fungsi | Module |
| --- | --- | --- | --- |
| [NAMA JOB 1] | `[CRON EXPRESSION]` | [Fungsi konkret — contoh: Mengirim ringkasan harian ke supervisor] | [NAMA MODULE] |
| [NAMA JOB 2] | `[CRON EXPRESSION]` | [Fungsi] | [NAMA MODULE] |
| [NAMA JOB 3] | `[CRON EXPRESSION]` | [Fungsi] | [NAMA MODULE] |

Timezone yang digunakan scheduler: `[TIMEZONE — contoh: Asia/Jakarta]`. Pastikan environment variable timezone server konsisten.

### 8.2 Cara Scheduler Berjalan

Scheduler berjalan sebagai bagian dari proses backend utama. Tidak ada proses terpisah yang perlu di-deploy untuk Scheduler.

```
[CARA SCHEDULER DIJALANKAN — sesuaikan dengan implementasi]

Contoh jika in-process:
Backend API start
      |
      v
Scheduler module diinisialisasi
      |
      v
Job terdaftar sesuai cron expression
      |
      v
Job berjalan otomatis sesuai jadwal
```

### 8.3 Perhatian Multi-Instance

Jika backend berjalan lebih dari satu instance (horizontal scaling), pastikan Scheduler **tidak berjalan di setiap instance secara bersamaan**, karena akan menyebabkan job dijalankan duplikat.

Mekanisme yang digunakan project ini untuk mencegah duplikasi Scheduler: [MEKANISME — contoh: Distributed lock via Redis / Scheduler hanya berjalan di instance pertama / Scheduler dijalankan di container terpisah].

### 8.4 Verifikasi Scheduler Berjalan

```bash
# Verifikasi dari log bahwa Scheduler berhasil diregistrasi
[PERINTAH CEK LOG SCHEDULER — contoh: grep "Scheduler registered" /var/log/app.log]

# Output yang diharapkan:
# [timestamp] INFO Scheduler: Job [NAMA JOB] registered with schedule [JADWAL]
```

---

## 9. Queue Deployment

### 9.1 Gambaran Queue

Queue memproses job asinkron yang tidak perlu hasil langsung. Daftar queue yang aktif:

| Nama Queue | Job yang Diproses | Prioritas | Retry Maksimal |
| --- | --- | --- | --- |
| `[NAMA QUEUE 1]` | [Jenis job — contoh: pengiriman email] | [HIGH/NORMAL/LOW] | [N] kali |
| `[NAMA QUEUE 2]` | [Jenis job] | [PRIORITAS] | [N] kali |
| `[NAMA QUEUE 3]` | [Jenis job] | [PRIORITAS] | [N] kali |

### 9.2 Cara Queue Worker Berjalan

```
[CARA QUEUE WORKER DIJALANKAN — sesuaikan dengan implementasi]

Contoh jika proses terpisah:
Backend API menerima request
      |
      v
Job dipublikasikan ke Redis Queue
      |
      v
Queue Worker (proses terpisah) mengambil job
      |
      v
Job diproses
      |
      +-- Sukses → Job ditandai selesai; log ditulis
      |
      +-- Gagal → Retry (maksimal [N] kali) → Jika tetap gagal → Dead Letter Queue
```

### 9.3 Deployment Queue Worker

Queue Worker di-deploy sebagai [MEKANISME — contoh: container terpisah / proses PM2 terpisah / bagian dari container yang sama].

```bash
# Menjalankan Queue Worker (jika proses terpisah)
[PERINTAH — contoh: docker compose up -d worker]

# Atau via PM2
[PERINTAH — contoh: pm2 start ecosystem.config.js --only queue-worker]
```

### 9.4 Monitoring Queue

Pantau kondisi queue untuk memastikan tidak ada penumpukan job yang tidak diproses:

```bash
# Cek jumlah job di tiap queue
[PERINTAH — contoh: redis-cli llen [NAMA QUEUE]]

# Cek job yang gagal (dead letter queue)
[PERINTAH — contoh: redis-cli llen [NAMA QUEUE]:failed]
```

Threshold yang perlu diperhatikan:
- Job pending melebihi [N] = investigasi apakah worker berjalan
- Job failed lebih dari [N] dalam [N] jam = investigasi penyebab kegagalan

---

## 10. Restart Service

### 10.1 Kapan Restart Diperlukan

Service perlu di-restart dalam kondisi berikut:

- Perubahan environment variable (konfigurasi tidak reload otomatis)
- Memory leak terdeteksi (penggunaan memori naik terus tanpa turun)
- Service tidak merespons health check meskipun proses masih berjalan
- Setelah rollback image

### 10.2 Prosedur Restart

**Restart tanpa downtime (rolling restart):**

```bash
# Jika menggunakan Docker Compose
docker compose restart [NAMA SERVICE — contoh: api]

# Jika menggunakan PM2
pm2 reload [NAMA APP]

# Jika menggunakan systemd
sudo systemctl restart [NAMA SERVICE]
```

**Restart dengan downtime (hard restart — hanya jika rolling restart tidak berhasil):**

```bash
# Stop service
[PERINTAH STOP]

# Tunggu hingga semua request selesai (grace period)
sleep [N]

# Start service
[PERINTAH START]
```

### 10.3 Urutan Restart Saat Restart Penuh

Jika seluruh stack perlu di-restart (misalnya setelah maintenance server), ikuti urutan ini:

```
1. Jalankan Database (tunggu hingga ready)
2. Jalankan Cache/Redis (tunggu hingga ready)
3. Jalankan Queue broker (tunggu hingga ready)
4. Jalankan Backend API
5. Jalankan Queue Worker
6. Verifikasi health check semua komponen
```

Jangan jalankan Backend API sebelum Database dan Cache siap; aplikasi akan gagal startup karena tidak bisa membuat koneksi awal.

---

## 11. Logging Location

### 11.1 Format Log

Seluruh log ditulis dalam format JSON struktural. Setiap entri log memiliki field standar:

```json
{
  "timestamp": "2025-01-15T08:30:00.000Z",
  "level": "INFO",
  "context": "[NamaKomponen]",
  "message": "[Pesan log]",
  "requestId": "[UUID request]",
  "userId": "[UUID user jika tersedia]",
  "duration": 142
}
```

### 11.2 Lokasi Log per Environment

| Environment | Lokasi Log | Cara Akses |
| --- | --- | --- |
| local | `[PATH — contoh: storage/logs/app.log]` atau output terminal | `tail -f [PATH LOG]` |
| development | [NAMA PLATFORM — contoh: Grafana Loki / Datadog / CloudWatch] | [URL AKSES] |
| staging | [NAMA PLATFORM] | [URL AKSES] |
| production | [NAMA PLATFORM]; akses terbatas | [URL AKSES]; hubungi Tech Lead untuk akses |

### 11.3 Level Log dan Artinya

| Level | Kapan Digunakan | Tindakan yang Diperlukan |
| --- | --- | --- |
| `DEBUG` | Informasi detail untuk debugging lokal | Tidak ada; hanya aktif di local |
| `INFO` | Operasi normal yang berjalan sesuai rencana | Tidak ada |
| `WARN` | Kondisi yang tidak diharapkan tapi sistem masih berjalan | Investigasi jika berulang |
| `ERROR` | Error yang menyebabkan operasi gagal; termasuk stack trace | Investigasi segera |
| `FATAL` | Error yang menyebabkan service tidak bisa berjalan | Tindakan darurat; eskalasi |

### 11.4 Cara Mencari Log

```bash
# Development / staging — via log platform
# Filter berdasarkan level ERROR dalam 1 jam terakhir:
[QUERY LOG PLATFORM — contoh: {app="project-api"} |= "ERROR" | json | line_format "{{.message}}"]

# Langsung dari server (production)
# Log 50 baris terakhir
docker logs --tail 50 [NAMA CONTAINER]

# Log real-time
docker logs -f [NAMA CONTAINER]

# Cari request berdasarkan requestId
docker logs [NAMA CONTAINER] 2>&1 | grep [REQUEST_ID]
```

### 11.5 Retensi Log

| Environment | Retensi | Keterangan |
| --- | --- | --- |
| development | [N] hari | Log lama dihapus otomatis |
| staging | [N] hari | Log lama dihapus otomatis |
| production | [N] hari | Sesuai kebijakan compliance yang berlaku |

---

## 12. Monitoring Overview

### 12.1 Komponen yang Dimonitor

| Komponen | Metrik yang Dipantau | Tool |
| --- | --- | --- |
| Backend API | Response time, error rate, request per detik | [TOOL — contoh: Prometheus + Grafana] |
| Database | Query time, connection pool, storage usage | [TOOL] |
| Cache | Hit rate, memory usage, connection count | [TOOL] |
| Queue | Job pending, job failed, processing time | [TOOL / Dashboard Queue] |
| Server | CPU, RAM, disk usage | [TOOL] |

### 12.2 Threshold dan Alert

Alert otomatis dikirim ke [CHANNEL ALERT — contoh: Slack #alerts atau PagerDuty] ketika threshold berikut tercapai:

| Metrik | Threshold Warning | Threshold Critical | Tindakan |
| --- | --- | --- | --- |
| Error rate API | > [N]% dalam [N] menit | > [N]% dalam [N] menit | Investigasi log; pertimbangkan rollback |
| P95 response time | > [N] ms | > [N] ms | Investigasi slow query atau bottleneck |
| CPU server | > [N]% selama [N] menit | > [N]% selama [N] menit | Cek proses yang membebani |
| RAM server | > [N]% | > [N]% | Investigasi memory leak |
| Disk server | > [N]% | > [N]% | Bersihkan log lama atau perluas storage |
| Queue pending | > [N] job | > [N] job | Verifikasi worker berjalan |
| Database connection | > [N]% pool terpakai | Pool habis | Investigasi koneksi yang tidak dilepas |

### 12.3 Akses Dashboard Monitoring

| Dashboard | URL | Akses |
| --- | --- | --- |
| Grafana (API metrics) | [URL] | Tim engineering |
| Log platform | [URL] | Tim engineering |
| Queue dashboard | [URL] | Tim engineering |
| Uptime monitor | [URL] | Seluruh tim |

### 12.4 Prosedur Respons Insiden

```
Alert masuk
      |
      v
On-call engineer menerima alert
      |
      v
Periksa monitoring dashboard — identifikasi komponen yang bermasalah
      |
      v
Periksa log di sekitar waktu alert terjadi
      |
      v
Tentukan severity:
  - Critical (layanan down) → Eskalasi ke Tech Lead segera
  - High (performa degradasi) → Investigasi dan perbaiki dalam [N] jam
  - Medium (warning, belum kritis) → Investigasi dalam [N] jam kerja
      |
      v
Dokumentasikan temuan dan tindakan di [SISTEM TIKET / CHANNEL INSIDEN]
```

---

## 13. Verification Deployment

### 13.1 Verifikasi Wajib Setelah Setiap Deployment

Setelah setiap deployment selesai, jalankan pemeriksaan berikut sebelum menyatakan deployment sukses.

**Pemeriksaan 1: Health Check API**

```bash
curl -f [URL ENVIRONMENT]/health
# Output yang diharapkan: HTTP 200
# Body contoh: {"status": "ok", "database": "connected", "cache": "connected"}
```

**Pemeriksaan 2: Verifikasi Versi**

```bash
curl [URL ENVIRONMENT]/version
# Output yang diharapkan: versi aplikasi sesuai yang di-deploy
# Body contoh: {"version": "[VERSI]", "commit": "[COMMIT HASH]"}
```

**Pemeriksaan 3: Endpoint Autentikasi**

```bash
# Coba login dengan akun test
curl -X POST [URL ENVIRONMENT]/api/[VERSI]/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "[EMAIL TEST]", "password": "[PASSWORD TEST]"}'
# Output yang diharapkan: HTTP 200 dengan accessToken dan refreshToken
```

**Pemeriksaan 4: Log Tidak Ada Error Fatal**

```bash
# Lihat log 2 menit setelah deployment
[PERINTAH LOG — contoh: docker logs --tail 100 [CONTAINER] | grep -i "error\|fatal"]
# Output yang diharapkan: tidak ada ERROR atau FATAL dalam log startup
```

**Pemeriksaan 5: Migration Sudah Teraplikasikan**

```bash
[PERINTAH CEK STATUS MIGRATION]
# Output yang diharapkan: semua migration berstatus applied/executed
```

**Pemeriksaan 6: Queue Worker Aktif**

```bash
[PERINTAH CEK QUEUE WORKER — contoh: docker ps | grep worker]
# Output yang diharapkan: container worker dalam status running
```

### 13.2 Smoke Test Setelah Deployment Production

Untuk deployment production, lakukan smoke test tambahan dengan menjalankan alur kritikal secara manual:

| Alur | Cara Verifikasi | Hasil yang Diharapkan |
| --- | --- | --- |
| Login pengguna | Masuk menggunakan akun test production | Berhasil masuk, token diterima |
| [FITUR KRITIKAL 1] | [Cara verifikasi] | [Hasil yang diharapkan] |
| [FITUR KRITIKAL 2] | [Cara verifikasi] | [Hasil yang diharapkan] |
| Job Queue | Trigger aksi yang menghasilkan job; cek log worker | Job berhasil diproses |

Smoke test dilakukan oleh [ROLE — contoh: QA Engineer atau Tech Lead] dalam [N] menit setelah deployment.

---

## 14. Backup Strategy

### 14.1 Backup Database

Backup Database production dilakukan secara otomatis oleh [SIAPA/APA — contoh: tim infrastruktur / script cron / layanan managed Database].

| Aspek | Detail |
| --- | --- |
| Frekuensi | [FREKUENSI — contoh: Setiap hari pukul 02.00 WIB] |
| Retensi | [RETENSI — contoh: 30 hari terakhir] |
| Lokasi penyimpanan | [LOKASI — contoh: Object Storage / S3-compatible di [REGION]] |
| Format | [FORMAT — contoh: pg_dump dalam format compressed] |
| Notifikasi gagal | Dikirim ke [CHANNEL] jika backup gagal |

### 14.2 Memverifikasi Backup Tersedia

Sebelum setiap deployment production, verifikasi bahwa backup terbaru tersedia dan dapat di-restore:

```bash
# Cek daftar backup yang tersedia
[PERINTAH CEK BACKUP — contoh: aws s3 ls s3://[BUCKET]/backups/ --recursive | tail -5]

# Verifikasi backup terbaru tidak corrupt (opsional tapi direkomendasikan)
[PERINTAH VERIFIKASI — contoh: pg_restore --list [NAMA FILE BACKUP] > /dev/null && echo "OK"]
```

### 14.3 Prosedur Restore Database

Restore Database hanya dilakukan dalam kondisi darurat atau setelah konfirmasi dari Tech Lead dan Product Owner.

> **PERINGATAN:** Restore Database menimpa seluruh data sejak waktu backup. Seluruh transaksi setelah waktu backup akan hilang permanen.
> 

```bash
# [1] Hentikan aplikasi untuk mencegah write baru
[PERINTAH HENTIKAN BACKEND]

# [2] Unduh backup dari storage
[PERINTAH UNDUH BACKUP — contoh: aws s3 cp s3://[BUCKET]/backups/[NAMA FILE] /tmp/]

# [3] Restore ke Database
[PERINTAH RESTORE — contoh: pg_restore -h [HOST] -U [USER] -d [NAMA DB] /tmp/[NAMA FILE]]

# [4] Verifikasi data
[QUERY VERIFIKASI — contoh: psql -c "SELECT COUNT(*) FROM users;" atau cek tabel kritis]

# [5] Jalankan kembali aplikasi
[PERINTAH START BACKEND]
```

### 14.4 Recovery Point Objective (RPO) dan Recovery Time Objective (RTO)

| Metrik | Target | Keterangan |
| --- | --- | --- |
| RPO (maksimal kehilangan data) | [DURASI — contoh: 24 jam] | Data yang masuk sejak backup terakhir dapat hilang jika terjadi bencana |
| RTO (waktu pemulihan) | [DURASI — contoh: 2 jam] | Waktu dari keputusan restore hingga sistem kembali operasional |

---

## 15. Deployment Checklist

### 15.1 Checklist Pre-Deployment

Checklist ini wajib dilengkapi sebelum setiap deployment production. Checklist staging dapat melewati beberapa item yang ditandai `[PROD ONLY]`.

**Kode dan Testing:**

```
[ ] Seluruh test lulus di CI pipeline
[ ] Tidak ada test yang di-skip tanpa alasan yang terdokumentasi
[ ] Code review sudah selesai dan disetujui Tech Lead
[ ] Linting tidak ada error
[ ] Type check tidak ada error (jika TypeScript)
```

**Database:**

```
[ ] Migration sudah diuji di staging
[ ] Migration bersifat backward compatible (additive only)
[ ] [PROD ONLY] Backup production terbaru sudah diverifikasi tersedia
[ ] Estimasi waktu run migration diketahui (untuk migration besar)
```

**Konfigurasi:**

```
[ ] Seluruh environment variable untuk environment target sudah terdefinisi
[ ] Tidak ada API key development yang terbawa ke production
[ ] Konfigurasi integrasi eksternal mengarah ke endpoint yang benar
```

**Koordinasi:**

```
[ ] [PROD ONLY] Approval Product Owner sudah didapat
[ ] [PROD ONLY] Jendela deployment sudah disepakati
[ ] [PROD ONLY] Tim on-call sudah diberitahu
[ ] [PROD ONLY] Komunikasi ke pengguna sudah disiapkan jika ada downtime
```

### 15.2 Checklist Post-Deployment

```
[ ] Health check endpoint mengembalikan HTTP 200
[ ] Versi aplikasi sesuai yang di-deploy
[ ] Login berhasil
[ ] Tidak ada ERROR atau FATAL dalam log startup
[ ] Seluruh migration berstatus applied
[ ] Queue Worker berjalan
[ ] Scheduler terdaftar dan berjalan
[ ] [PROD ONLY] Smoke test alur kritikal selesai
[ ] [PROD ONLY] Monitoring tidak menunjukkan anomali dalam 15 menit pertama
[ ] [PROD ONLY] Catatan deployment ditulis di [SISTEM — contoh: Notion / Confluence / tiket Jira]
```

---

## 16. Dependency Deployment

### 16.1 Peta Dependency Deployment

Setiap komponen yang di-deploy memiliki dependency yang harus sudah berjalan sebelum komponen tersebut dijalankan.

```
Infrastruktur (Database, Cache, Queue broker)
        |
        | Harus berjalan sebelum
        v
Migration (container sementara)
        |
        | Harus selesai sebelum
        v
Backend API
        |
        | Harus berjalan sebelum
        v
Queue Worker
        |
        | Berjalan bersamaan atau setelah
        v
Scheduler (jika proses terpisah)
```

### 16.2 Dependency Eksternal

Project ini bergantung pada layanan eksternal berikut. Jika layanan ini tidak dapat dijangkau, bagian sistem yang bergantung padanya akan terdampak.

| Layanan Eksternal | Digunakan Oleh | Dampak Jika Tidak Tersedia | Cara Verifikasi |
| --- | --- | --- | --- |
| [NAMA LAYANAN 1 — contoh: Email Service] | [Module yang menggunakannya] | [Dampak — contoh: Email tidak terkirim; job masuk dead letter queue] | `curl [URL HEALTH CHECK LAYANAN]` |
| [NAMA LAYANAN 2] | [Module] | [Dampak] | [Perintah verifikasi] |
| [NAMA LAYANAN 3] | [Module] | [Dampak] | [Perintah verifikasi] |

### 16.3 Verifikasi Dependency Sebelum Deployment Production

```bash
# Verifikasi konektivitas ke Database
[PERINTAH — contoh: pg_isready -h [HOST] -p [PORT] -U [USER]]

# Verifikasi konektivitas ke Cache
[PERINTAH — contoh: redis-cli -h [HOST] -p [PORT] ping]

# Verifikasi konektivitas ke layanan eksternal
[PERINTAH — contoh: curl -f https://[LAYANAN]/health]
```

Semua dependency harus dapat dijangkau sebelum deployment dilanjutkan. Deployment ke environment yang tidak terhubung ke dependency kritikal akan menyebabkan startup gagal.

---