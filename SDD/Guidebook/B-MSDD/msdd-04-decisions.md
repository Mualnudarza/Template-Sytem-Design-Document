# BAB 4: DECISIONS

Bagian ini mendokumentasikan keputusan arsitektural atau desain teknis yang signifikan, berpotensi memberikan dampak jangka panjang, dan tidak dapat dibalik dengan mudah. Setiap keputusan harus mencantumkan konteks permasalahan, opsi yang dipertimbangkan, pilihan akhir, dan justifikasinya.

---

## 4.0 Standar dan Aturan Penulisan Keputusan Arsitektur

Bagian ini merupakan referensi teknis wajib untuk menjamin konsistensi format rekaman keputusan (Architectural Decision Record). **Bagian ini bersifat informatif untuk fase penyusunan dan wajib dihapus dari versi final setelah seluruh keputusan selesai didokumentasikan.**

---

**Perintah (Instructions)**

Keputusan arsitektur dapat didokumentasikan secara inline menggunakan template di bawah ini, atau dihubungkan (cross-linked) langsung ke file MADR terpisah di direktori `docs/A-MADR/`. Disarankan menggunakan file MADR terpisah untuk keputusan yang memiliki dampak lintas tim atau bersifat permanen. Untuk keputusan yang lebih terlokalisasi, dokumentasi inline diperbolehkan.

**Template Wajib Rekaman Keputusan (Inline MADR):**

```
- ID: [NNNN]-{judul-singkat}
  Contoh: 0001-use-postgresql-for-core-db

- Title: [Judul singkat dan deskriptif dari keputusan arsitektur]

- Status: [Proposed | Accepted | Deprecated | Superseded by NNNN]

- Context:
  [Deskripsi kondisi saat ini dan rumusan permasalahan teknis yang memerlukan keputusan.
  Jelaskan "mengapa" keputusan ini perlu diambil pada saat ini.]

- Options:
  [Daftar opsi alternatif yang dipertimbangkan sebelum mengambil keputusan, beserta
  kelebihan dan kekurangan singkat setiap opsi.]

- Outcome:
  Opsi yang dipilih: "{{nama_opsi}}", karena "{{justifikasi_teknis_atau_bisnis}}".
  Konsekuensi yang diterima: "{{konsekuensi_yang_disadari_dan_diterima}}".

- More Information:
  [Tautan ke hasil riset, POC, benchmark, jurnal teknis, atau dokumen pendukung lainnya.]
```

**Catatan:** Keputusan yang bersifat “Superseded” harus tetap dipertahankan dalam dokumen untuk menjaga riwayat arsitektur. Jangan menghapus keputusan lama; cukup ubah statusnya menjadi `Superseded by NNNN`.

### Contoh (Example)

---

**ID:** `0001-use-postgresql-for-primary-database`

**Title:** Penggunaan PostgreSQL sebagai Basis Data Utama

**Status:** Accepted

**Context:**

Sistem memerlukan basis data relasional yang andal untuk menyimpan entitas transaksional inti (order, payment, user). Sistem memiliki kebutuhan konsistensi data yang tinggi (ACID compliance), dukungan untuk transaksi kompleks lintas tabel, serta kemampuan untuk melakukan query analitik sesekali tanpa infrastruktur tambahan.

**Options:**

| Opsi | Kelebihan | Kekurangan |
| --- | --- | --- |
| PostgreSQL | ACID compliant, fitur JSON native, ekosistem mature, open-source | Horizontal write-scaling terbatas dibanding NoSQL |
| MySQL | Familiar bagi tim, performa read tinggi | Fitur transaksi lebih terbatas, JSON support lebih lemah |
| MongoDB | Skema fleksibel, horizontal scaling mudah | Tidak ACID penuh lintas dokumen pada versi lama |

**Outcome:**

Opsi yang dipilih: “PostgreSQL”, karena kebutuhan konsistensi transaksional sistem ini lebih diprioritaskan daripada kebutuhan skala tulis horizontal, dan fitur JSONB PostgreSQL sudah cukup untuk menangani kebutuhan data semi-terstruktur tanpa menambah kompleksitas infrastruktur.

Konsekuensi yang diterima: Skalabilitas tulis horizontal akan memerlukan pola sharding manual atau migrasi ke distributed database jika volume transaksi melebihi 10.000 TPS.

**More Information:**
- Hasil benchmark internal: `docs/research/db-benchmark-2024.md`
- REQ Reference: `REQ-PERF-001`, `REQ-REL-001`

---

**ID:** `ADR-[NNN]-{judul}`

**Title:** `{{judul_keputusan}}`

**Status:** `{{status}}`

**Context:** `{{konteks_permasalahan}}`

**Options:** `{{daftar_opsi}}`

**Outcome:** `{{pilihan_dan_justifikasi}}`

**More Information:** `{{tautan_pendukung}}`