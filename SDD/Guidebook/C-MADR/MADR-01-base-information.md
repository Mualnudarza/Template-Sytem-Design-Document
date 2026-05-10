# Markdown Any Decision Records (MADR)

> "Markdown Any Decision Records" (MADR) adalah pendekatan dokumentasi keputusan berbasis Markdown yang digunakan untuk mencatat keputusan penting secara terstruktur, terlacak, dan terversi di dalam repositori proyek.
> 

MADR digunakan untuk mendokumentasikan *mengapa* suatu keputusan teknis, arsitektural, operasional, atau proses bisnis diambil. Dokumen ini melengkapi MSRS (*apa yang dibutuhkan sistem*) dan MSDD (*bagaimana sistem dibangun*) dengan menjelaskan rasionalitas di balik keputusan tersebut.

---

---

# 1. Introduction

## 1.1 Purpose

Panduan ini menjelaskan standar penggunaan MADR (*Markdown Any Decision Records*) sebagai mekanisme dokumentasi keputusan pada proyek perangkat lunak.

Tujuan utama MADR adalah:

- Mendokumentasikan keputusan penting secara konsisten dan terversi.
- Menyimpan konteks dan alasan teknis di balik keputusan.
- Mengurangi hilangnya pengetahuan (*knowledge loss*) akibat pergantian anggota tim.
- Mendukung keterlacakan (*traceability*) antara kebutuhan, desain, implementasi, dan keputusan.
- Menjadi referensi historis ketika terjadi perubahan arsitektur atau evaluasi teknis.

Dokumen MADR berfokus pada pertanyaan:

> "Mengapa keputusan ini diambil?"
> 

---

## 1.2 Scope

Panduan ini berlaku untuk seluruh keputusan yang memiliki dampak terhadap:

- Arsitektur perangkat lunak
- Desain teknis
- Integrasi sistem
- Pemilihan teknologi
- Keamanan dan kepatuhan
- Infrastruktur dan deployment
- Proses operasional
- AI/ML governance
- Standar pengembangan

Keputusan dapat bersifat:

- Strategis
- Taktis
- Teknis
- Operasional
- Organisasional

---

## 1.3 Relationship with MSRS and MSDD

MADR merupakan bagian dari *Traceability Triad* bersama MSRS dan MSDD.

| Artefact | Focus | Primary Question |
| --- | --- | --- |
| MSRS | Requirements | Apa yang harus dilakukan sistem? |
| MSDD | Design | Bagaimana sistem dibangun? |
| MADR | Decisions | Mengapa keputusan tersebut dipilih? |

Hubungan antar artefak:

```
Requirements (MSRS)
        ↓
Design (MSDD)
        ↓
Decision Rationale (MADR)
```

MADR wajib memiliki referensi silang (*cross-reference*) terhadap:

- Requirement ID dari MSRS
- Design View dari MSDD
- Proof of Concept (POC)
- Benchmark
- Risk Assessment
- Hasil evaluasi teknis

---

## 1.4 Benefits

Penggunaan MADR memberikan manfaat berikut:

- Meningkatkan transparansi keputusan teknis.
- Mempermudah onboarding anggota baru.
- Mengurangi pengulangan diskusi teknis yang sama.
- Mendukung audit dan kepatuhan.
- Mempermudah analisis dampak perubahan.
- Menjadi dokumentasi historis evolusi sistem.
- Mendukung praktik *Architecture as Code*.

---

# 2. MADR Structure

## 2.1 File Naming Convention

Setiap MADR harus menggunakan pola penamaan berikut:

```
NNNN-title-with-dashes.md
```

Contoh:

```
0001-use-postgresql.md
0002-adopt-event-driven-architecture.md
0003-use-openid-connect.md
```

Aturan:

| Element | Description |
| --- | --- |
| NNNN | Nomor berurutan |
| title | Judul singkat menggunakan lowercase dan dash |
| .md | Format Markdown |

---

## 2.2 Repository Structure

Direktori standar MADR:

```
docs/
└── decisions/
    ├── 0001-use-postgresql.md
    ├── 0002-use-kafka.md
    └── 0003-use-redis-cache.md
```

Untuk proyek besar, kategori dapat dipisahkan berdasarkan domain:

```
docs/
└── decisions/
    ├── backend/
    ├── frontend/
    ├── infrastructure/
    ├── ai-ml/
    └── security/
```

---

## 2.3 Decision Lifecycle

Siklus hidup keputusan:

```
Proposed
    ↓
Accepted
    ↓
Implemented
    ↓
Deprecated / Superseded
```

Status keputusan harus dicantumkan secara eksplisit pada dokumen MADR.

---

# 3. MADR Template

## 3.1 Mandatory Sections

Berikut adalah struktur minimum yang wajib dimiliki setiap MADR.

```markdown
# [Decision Title]

## Status
Accepted | Proposed | Deprecated | Superseded

## Context

Penjelasan masalah, kondisi saat ini, batasan, dan kebutuhan yang memicu keputusan.

## Decision

Keputusan yang dipilih.

## Considered Options

- Opsi A
- Opsi B
- Opsi C

## Consequences

### Positive

- Dampak positif keputusan

### Negative

- Risiko atau trade-off

## Alternatives Rejected

Penjelasan mengapa opsi lain tidak dipilih.

## References

- MSRS Requirement ID
- MSDD Design View
- POC
- Benchmark
- External References
```

---

## 3.2 Recommended Additional Sections

Bagian tambahan yang direkomendasikan:

| Section | Purpose |
| --- | --- |
| Security Impact | Analisis keamanan |
| Cost Impact | Dampak biaya |
| Operational Impact | Dampak operasional |
| Migration Plan | Strategi migrasi |
| Rollback Plan | Strategi rollback |
| Compliance Impact | Dampak regulasi |
| AI/ML Considerations | Risiko model dan data |
| Monitoring Strategy | Observability dan alerting |

---

# 4. Writing Guidelines

## 4.1 Good Decision Characteristics

MADR yang baik memiliki karakteristik berikut:

| Characteristic | Description |
| --- | --- |
| Specific | Fokus pada satu keputusan utama |
| Traceable | Memiliki referensi silang |
| Justified | Memiliki alasan yang jelas |
| Versioned | Terversi di Git |
| Actionable | Dapat dipahami dan diterapkan |
| Concise | Tidak bertele-tele |
| Historical | Menjelaskan konteks waktu keputusan dibuat |

---

## 4.2 What Should Be Recorded

Keputusan berikut direkomendasikan untuk dicatat:

- Pemilihan database
- Pemilihan framework
- Pemilihan arsitektur
- Strategi deployment
- Integrasi pihak ketiga
- Strategi keamanan
- Penggunaan AI/ML
- Strategi observability
- Penggunaan message broker
- Standar coding
- Strategi caching
- API versioning
- Pemilihan cloud provider

---

## 4.3 What Should Not Be Recorded

Hal berikut tidak direkomendasikan menjadi MADR:

- Diskusi sementara tanpa keputusan
- Catatan rapat umum
- Detail implementasi kecil
- Tugas harian developer
- Bug fixing minor
- Konfigurasi sementara

---

# 5. Traceability Integration

MADR harus terhubung dengan artefak lain di dalam repositori.

Contoh integrasi:

| Artifact | Example |
| --- | --- |
| MSRS | `REQ-SEC-001` |
| MSDD | `Viewpoint: Deployment` |
| Source Code | `src/auth/` |
| CI/CD | `.github/workflows/` |
| POC | `research/poc-kafka.md` |
| Threat Model | `security/threat-model.md` |

Contoh referensi di MADR:

```markdown
## References

- REQ-PERF-002
- REQ-SEC-004
- MSDD Viewpoint: Deployment
- docs/poc/kafka-benchmark.md
```

---

# 6. Examples

## Example MADR

```markdown
# Use PostgreSQL for Core Transaction Database

## Status

Accepted

## Context

Sistem membutuhkan database relasional yang mendukung transaksi ACID,
replikasi, dan integritas data tinggi.

## Decision

Menggunakan PostgreSQL sebagai database utama.

## Considered Options

- PostgreSQL
- MySQL
- MongoDB

## Consequences

### Positive

- Dukungan ACID kuat
- Ekosistem matang
- Dukungan indexing lengkap

### Negative

- Membutuhkan tuning performa
- Replikasi lebih kompleks dibanding managed NoSQL

## Alternatives Rejected

MongoDB tidak dipilih karena kebutuhan transaksi relasional sangat tinggi.

## References

- REQ-REL-001
- REQ-PERF-003
- MSDD Deployment View
```

---

# 7. Appendix

## Recommended Tooling

| Tool | Purpose |
| --- | --- |
| Markdownlint | Konsistensi format Markdown |
| Vale | Style guide linting |
| MkDocs | Publikasi dokumentasi |
| GitHub Actions | Otomatisasi validasi |
| Mermaid | Diagram visual |
| ADR Tools | Otomatisasi ADR |

---

## Recommended Practices

- Simpan MADR di repositori yang sama dengan kode sumber.
- Gunakan pull request untuk review keputusan.
- Jangan menghapus keputusan lama.
- Gunakan status `Superseded` untuk keputusan yang digantikan.
- Hubungkan MADR dengan issue tracker dan roadmap proyek.
- Pastikan setiap keputusan memiliki justifikasi teknis maupun bisnis.