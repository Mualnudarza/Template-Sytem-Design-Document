# <Project Documentation>

Repositori ini berisi kumpulan dokumentasi rekayasa perangkat lunak berbasis Markdown yang terstruktur dan saling terhubung untuk mendukung proses pengembangan sistem secara menyeluruh.

Dokumentasi dibagi menjadi tiga pilar utama:

| Pilar | Deskripsi |
|---|---|
| `A-MSRS` | Spesifikasi kebutuhan perangkat lunak (*Software Requirements Specification*) yang mendefinisikan **apa** yang harus dilakukan sistem. |
| `B-MSDD` | Dokumen desain perangkat lunak (*Software Design Description*) yang menjelaskan **bagaimana** sistem dibangun. |
| `C-MADR` | Catatan keputusan arsitektur dan teknis (*Markdown Any Decision Records*) yang menjelaskan **mengapa** keputusan diambil. |

---

# Struktur Dokumentasi

```tree
docs
│   README.md
│
├───A-MSRS
├───B-MSDD
└───C-MADR
```

---

# Traceability Triad

Dokumentasi ini menggunakan pendekatan *Traceability Triad* untuk menjaga keterlacakan antara kebutuhan, desain, dan keputusan teknis.

```text
Requirements (MSRS)
        ↓
Design (MSDD)
        ↓
Decisions (MADR)
```

---

# Aturan Penamaan File

| Jenis | Format |
|---|---|
| Requirement | `REQ-[AREA]-[NNN]` |
| Design View | `[NNN]-<title>` |
| Decision Record | `[NNNN]-<title>` |

Contoh:

```text
REQ-FUNC-001
001-payment-service
0001-use-postgresql
```

---

# Konvensi Dokumentasi

- Semua dokumen ditulis menggunakan format Markdown (`.md`)
- Gunakan bahasa yang konsisten dan formal
- Hindari informasi ambigu atau tidak dapat diverifikasi
- Seluruh diagram direkomendasikan menggunakan Mermaid atau PlantUML
- Semua artefak harus dapat dilacak antar dokumen

---

# Integrasi Antar Pilar

| Sumber | Tujuan |
|---|---|
| MSRS → MSDD | Requirement direalisasikan dalam desain |
| MSDD → MADR | Desain dipengaruhi keputusan arsitektur |
| MADR → MSRS | Keputusan dapat muncul karena kebutuhan tertentu |

---

# Status Dokumentasi

| Status | Deskripsi |
|---|---|
| `draft` | Masih dalam tahap penulisan |
| `proposed` | Menunggu validasi |
| `approved` | Disetujui untuk digunakan |
| `deprecated` | Tidak direkomendasikan lagi |
| `archived` | Tidak aktif namun tetap disimpan |

---

# Tujuan Repositori

- Menjaga konsistensi dokumentasi teknis
- Mendukung audit dan keterlacakan
- Mempermudah kolaborasi lintas tim
- Mendukung *Version Control System* (Git)
- Mempermudah integrasi dengan AI/LLM dan otomatisasi dokumentasi