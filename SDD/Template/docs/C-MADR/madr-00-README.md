# MADR — Markdown Any Decision Records

Folder ini berisi dokumentasi keputusan teknis dan arsitektural menggunakan pendekatan MADR (*Markdown Any Decision Records*).

MADR digunakan untuk mencatat:

- Keputusan arsitektur
- Pemilihan teknologi
- Pertimbangan desain
- Trade-off teknis
- Dampak keputusan jangka panjang

---

# Tujuan MADR

- Mendokumentasikan alasan keputusan
- Menghindari kehilangan konteks teknis
- Mendukung audit dan review arsitektur
- Mempermudah onboarding tim baru
- Mendukung keterlacakan desain

---

# Struktur File

```text
0000-template.md
0001-use-postgresql.md
0002-use-kafka.md
0003-adopt-microservices.md
```

---

# Format Penamaan

```text
[NNNN]-<title>.md
```

Contoh:

```text
0001-use-postgresql.md
0002-use-redis-cache.md
0003-adopt-clean-architecture.md
```

---

# Prinsip Penulisan

- Fokus pada “Why”
- Setiap keputusan harus memiliki konteks
- Jelaskan alternatif yang dipertimbangkan
- Dokumentasikan trade-off
- Hindari keputusan tanpa justifikasi

---

# Integrasi dengan MSDD dan MSRS

| Relasi | Deskripsi |
|---|---|
| MSRS → MADR | Requirement dapat memicu keputusan |
| MSDD → MADR | Desain dipengaruhi keputusan |
| MADR → MSDD | Keputusan menghasilkan perubahan desain |

---

# Status Keputusan

| Status | Deskripsi |
|---|---|
| Proposed | Sedang dipertimbangkan |
| Accepted | Disetujui |
| Deprecated | Tidak direkomendasikan |
| Superseded | Digantikan keputusan lain |
| Rejected | Ditolak |