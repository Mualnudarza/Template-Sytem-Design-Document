# MSRS — Markdown Software Requirements Specification

Folder ini berisi spesifikasi kebutuhan perangkat lunak (*Software Requirements Specification*) berbasis Markdown.

MSRS mendefinisikan:

- Apa yang harus dilakukan sistem
- Kebutuhan fungsional dan non-fungsional
- Batasan sistem
- Integrasi eksternal
- Kepatuhan dan validasi

---

# Struktur Dokumen

| File | Deskripsi |
|---|---|
| `msrs-01-introduction.md` | Pendahuluan dan ruang lingkup sistem |
| `msrs-02-product-overview.md` | Gambaran umum produk |
| `msrs-03-requirements.md` | Seluruh kebutuhan sistem |
| `msrs-04-verification.md` | Matriks verifikasi kebutuhan |
| `msrs-05-appendixes.md` | Lampiran tambahan |


---

# Aturan Penulisan

- Gunakan format ID:
  
```text
REQ-[AREA]-[NNN]
```

Contoh:

```text
REQ-FUNC-001
REQ-SEC-002
REQ-PERF-010
```

---

# Kategori Requirement

| Area | Deskripsi |
|---|---|
| FUNC | Functional Requirements |
| INT | External Interfaces |
| PERF | Performance |
| SEC | Security |
| REL | Reliability |
| AVAIL | Availability |
| OBS | Observability |
| COMP | Compliance |
| INST | Installation |
| BUILD | Build and Delivery |
| DIST | Distribution |
| MAINT | Maintainability |
| REUSE | Reusability |
| PORT | Portability |
| COST | Cost Constraints |
| DEAD | Deadline and Milestones |
| POC | Proof of Concept |
| CM | Change Management |
| ML | AI/ML Requirements |
---

# Prinsip Penulisan

- Requirement harus dapat diuji
- Requirement harus jelas dan tidak ambigu
- Fokus pada “What”, bukan “How”
- Gunakan kata “shall” untuk kebutuhan wajib
- Hindari detail implementasi teknis