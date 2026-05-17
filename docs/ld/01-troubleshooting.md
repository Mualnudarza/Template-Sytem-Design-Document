# Troubleshooting Documentation Standard

## Overview

Dokumen ini mendefinisikan standar penulisan troubleshooting pada project agar seluruh proses investigasi, recovery, rollback, dan knowledge operational dapat terdokumentasi secara konsisten.

Dokumentasi troubleshooting berfungsi sebagai operational knowledge base yang digunakan oleh:

- Developer,
- DevOps,
- QA,
- Support Engineer,
- System Maintainer,
- Technical Lead.

Dokumen ini menjadi acuan utama dalam:

- dokumentasi incident,
- dokumentasi failure scenario,
- dokumentasi operational issue,
- dokumentasi deployment failure,
- dokumentasi integration problem,
- dokumentasi infrastructure problem,
- dokumentasi debugging process.

---

# Tujuan Dokumentasi Troubleshooting

Dokumentasi troubleshooting dibuat untuk membantu developer dan support team:

| Objective | Penjelasan |
| --- | --- |
| Identifikasi Masalah | Memahami gejala dan kondisi error |
| Root Cause Analysis | Menjelaskan penyebab utama masalah |
| Dependency Analysis | Memahami dependency yang terdampak |
| Recovery Process | Menjelaskan langkah pemulihan system |
| Rollback Process | Menjelaskan proses rollback yang aman |
| Verification | Memastikan system kembali normal |
| Prevention | Mengurangi kemungkinan issue berulang |
| Knowledge Transfer | Menyimpan operational knowledge project |

---

# Scope Troubleshooting

Dokumentasi troubleshooting mencakup:

- Setup issue,
- Environment issue,
- API issue,
- Database issue,
- Authentication issue,
- Authorization issue,
- Queue issue,
- Scheduler issue,
- Deployment issue,
- Infrastructure issue,
- Integration issue,
- Security issue,
- Performance issue,
- Data consistency issue,
- Service communication issue,
- External service failure,
- Cache issue,
- Storage issue,
- CI/CD issue.

---

# Troubleshooting Classification

Seluruh troubleshooting wajib memiliki kategori utama.

## Standard Classification

| Category | Penjelasan |
| --- | --- |
| Setup Issue | Masalah setup local/development |
| Database Issue | Masalah database, migration, query |
| API Issue | Masalah request/response API |
| Authentication Issue | Masalah login/token/session |
| Authorization Issue | Masalah permission/access control |
| Deployment Issue | Masalah deployment system |
| Queue Issue | Masalah async job/worker |
| Scheduler Issue | Masalah cron/scheduler |
| Integration Issue | Masalah third-party integration |
| Infrastructure Issue | Masalah server/network/container |
| Performance Issue | Masalah latency/resource usage |
| Security Issue | Masalah security vulnerability |
| Cache Issue | Masalah cache invalidation/state |
| Storage Issue | Masalah file/object storage |
| CI/CD Issue | Masalah pipeline automation |

---

# Mandatory Troubleshooting Structure

Seluruh troubleshooting wajib mengikuti struktur berikut.

---

# Problem Title

## Overview

Menjelaskan:

- konteks masalah,
- kondisi saat issue terjadi,
- area system yang terdampak.

### Penulisan

- singkat,
- teknis,
- langsung ke inti masalah.

---

## Error Symptoms

Menjelaskan:

- gejala yang muncul,
- perilaku system,
- indikasi failure,
- kondisi abnormal.

### Contoh

- API mengembalikan HTTP 500,
- login gagal,
- worker berhenti memproses queue,
- scheduler tidak berjalan,
- timeout pada integration service.

---

## Error Message

Berisi:

- error log,
- stack trace,
- response error,
- infrastructure error.

### Rules

- wajib menggunakan code block,
- tidak boleh dipotong,
- tidak boleh dimodifikasi,
- harus mempertahankan format asli.

### Example

```bash
SQLSTATE[08006] [7] could not connect to server:
Connection refused
```

---

## Impact

Menjelaskan dampak terhadap system.

### Contoh

| Impact Area | Penjelasan |
| --- | --- |
| API | Request gagal diproses |
| Database | Write operation gagal |
| Queue | Async process tertunda |
| User | User tidak dapat login |
| Integration | Sinkronisasi external service gagal |

---

## Root Cause

Menjelaskan penyebab utama issue.

### Root Cause Wajib Menjelaskan

- dependency yang terlibat,
- hubungan antar service,
- hubungan antar module,
- kondisi system yang memicu issue,
- failure point utama.

### Hindari

- asumsi,
- penjelasan ambigu,
- penjelasan terlalu umum.

### Contoh Buruk

> Database error karena koneksi bermasalah.
> 

### Contoh Benar

> PostgreSQL container tidak berhasil start karena volume corruption pada persistent storage sehingga application service gagal membuat database connection saat bootstrap initialization.
> 

---

## Possible Cause

Menjelaskan kemungkinan penyebab lain yang dapat menghasilkan issue serupa.

### Tujuan

- mempercepat investigasi,
- membantu identifikasi false-positive,
- membantu support engineer.

---

## Related Component

Menjelaskan component yang berkaitan langsung dengan issue.

### Contoh

| Component | Description |
| --- | --- |
| Auth Service | Authentication processing |
| API Gateway | Request routing |
| Queue Worker | Async processing |
| PostgreSQL | Main relational database |

---

## Related Module

Menjelaskan module project yang terdampak.

### Contoh

| Module | Impact |
| --- | --- |
| Authentication Module | Login failure |
| Payment Module | Transaction validation gagal |
| Notification Module | Email tidak terkirim |

---

## Dependency Impact

Menjelaskan dependency yang terdampak akibat issue.

### Contoh

| Dependency | Impact |
| --- | --- |
| Redis | Session invalidation gagal |
| RabbitMQ | Queue processing berhenti |
| S3 Storage | File upload gagal |

---

## Environment Impact

Menjelaskan environment yang terdampak.

### Environment Standard

Environment

---

Local

---

Development

---

Staging

---

Production

---

### Contoh

| Environment | Impact |
| --- | --- |
| Local | Tidak terdampak |
| Staging | Intermittent issue |
| Production | Full service outage |

---

## Validation Step

Menjelaskan langkah validasi awal untuk memastikan issue benar terjadi.

### Contoh

```bash
docker ps
docker logs app-service
curl http://localhost:8080/health
```

### Tujuan

- memastikan reproducibility,
- memastikan issue valid,
- menghindari false alarm.

---

## Investigation Step

Menjelaskan langkah investigasi teknis.

### Investigation Wajib Meliputi

- log analysis,
- service validation,
- dependency validation,
- infrastructure validation,
- database validation,
- integration validation.

### Contoh

1. Validasi status container.
2. Validasi database connectivity.
3. Validasi queue connectivity.
4. Analisis application log.
5. Analisis reverse proxy log.
6. Validasi environment variable.

---

## Recovery Step

Menjelaskan langkah recovery system.

### Recovery Rules

Recovery wajib:

- aman,
- berurutan,
- dapat direplikasi,
- mempertimbangkan dependency system.

### Recovery Structure

| Step | Action | Risk |
| --- | --- | --- |
| 1 | Restart queue worker | Low |
| 2 | Rebuild cache | Medium |
| 3 | Re-run migration | High |

### Wajib Menjelaskan

- prerequisite,
- dependency,
- risiko,
- service interruption jika ada.

---

## Rollback Step

Wajib ada jika issue berkaitan dengan:

- deployment,
- migration,
- integration,
- infrastructure change,
- release update.

### Rollback Wajib Menjelaskan

- kondisi rollback,
- trigger rollback,
- dependency rollback,
- langkah rollback,
- verification rollback.

### Contoh

```bash
docker compose down
git checkout previous-stable-tag
docker compose up -d
```

---

## Verification Step

Menjelaskan cara memastikan recovery berhasil.

### Verification Harus Meliputi

- service validation,
- endpoint validation,
- queue validation,
- database validation,
- monitoring validation.

### Contoh

```bash
curl http://localhost:8080/health
```

```bash
docker ps
```

---

## Prevention Recommendation

Menjelaskan tindakan pencegahan jangka panjang.

### Contoh

- menambahkan health check,
- menambahkan retry mechanism,
- memperbaiki monitoring,
- menambahkan timeout validation,
- memperbaiki deployment validation.

---

## Monitoring Recommendation

Menjelaskan monitoring yang perlu ditambahkan.

### Monitoring Area

| Area | Monitoring |
| --- | --- |
| API | Response time |
| Database | Connection pool |
| Queue | Pending job |
| Infrastructure | CPU & memory |
| Cache | Hit/miss ratio |

---

## Logging Reference

Menjelaskan lokasi log dan identifier penting.

### Wajib Menjelaskan

- lokasi log,
- service log,
- log format,
- trace id,
- correlation id,
- request id.

### Contoh

| Service | Log Location |
| --- | --- |
| API Service | `/var/log/app/api.log` |
| Queue Worker | `/var/log/app/worker.log` |
| Nginx | `/var/log/nginx/error.log` |

---

## Related Documentation

Troubleshooting wajib memiliki referensi ke dokumentasi lain.

### Minimal Relation

| Documentation | Tujuan |
| --- | --- |
| Architecture Documentation | Memahami system structure |
| Deployment Guide | Memahami deployment flow |
| API Documentation | Memahami API behavior |
| Database Documentation | Memahami schema & relation |
| Business Flow | Memahami operational flow |

---

## Notes

Digunakan untuk:

- catatan tambahan,
- limitation,
- operational warning,
- dependency khusus,
- environment exception.

---

# Troubleshooting Writing Rules

## Penulisan Judul

Gunakan format:

```
[Kategori] - [Nama Masalah]
```

### Contoh

```
[Database Issue] - PostgreSQL Connection Refused
```

```
[Deployment Issue] - Container Failed During Production Release
```

---

## Penulisan Bahasa

Gunakan:

- bahasa formal teknis,
- jelas,
- spesifik,
- operational-oriented.

Hindari:

- opini personal,
- asumsi,
- bahasa informal,
- penjelasan generik.

---

# Troubleshooting Documentation Workflow

## Standard Workflow

```
Issue Detected
    ↓
Validation
    ↓
Investigation
    ↓
Root Cause Analysis
    ↓
Recovery
    ↓
Verification
    ↓
Documentation Update
    ↓
Monitoring Improvement
```

---

# Troubleshooting Maintainability Rule

Dokumentasi troubleshooting wajib:

- diperbarui setelah incident selesai,
- diperbarui setelah root cause ditemukan,
- diperbarui setelah recovery berubah,
- diperbarui setelah architecture berubah.

---

# Troubleshooting Documentation Quality Standard

Dokumentasi troubleshooting dianggap valid jika:

| Validation | Status |
| --- | --- |
| Root cause jelas | Mandatory |
| Recovery dapat direplikasi | Mandatory |
| Dependency impact dijelaskan | Mandatory |
| Verification tersedia | Mandatory |
| Rollback tersedia jika diperlukan | Mandatory |
| Related documentation tersedia | Mandatory |
| Logging reference tersedia | Mandatory |

---

# Operational Knowledge Rule

Dokumentasi troubleshooting merupakan bagian dari:

- operational knowledge,
- production knowledge,
- deployment knowledge,
- maintenance knowledge.

Dokumentasi ini wajib diperlakukan sebagai:

- source of truth operational issue,
- debugging reference,
- recovery handbook,
- onboarding material support engineer.

---

# Final Notes

Dokumentasi troubleshooting bukan dokumentasi bug biasa.

Dokumentasi troubleshooting adalah:

- engineering operational handbook,
- production recovery guide,
- debugging knowledge base,
- maintainable support documentation.

Tujuan utamanya adalah:

- mempercepat investigasi,
- mempercepat recovery,
- mengurangi downtime,
- menjaga knowledge project,
- mengurangi dependency terhadap developer sebelumnya.