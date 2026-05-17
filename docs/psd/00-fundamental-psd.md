# 00 - Fundamental PSD (Product/System Documentation)

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Foundation
**Terakhir Diperbarui:** [TANGGAL]
**Pemilik Dokumen:** [NAMA TIM / ROLE]

---

## Daftar Isi

1. [Tujuan PSD](#1-tujuan-psd)
2. [Ruang Lingkup PSD](#2-ruang-lingkup-psd)
3. [Target Pembaca](#3-target-pembaca)
4. [Struktur Dokumentasi PSD](#4-struktur-dokumentasi-psd)
5. [Hubungan PSD dengan TSD](#5-hubungan-psd-dengan-tsd)
6. [Prinsip Maintainability](#6-prinsip-maintainability)
7. [Prinsip Scalability](#7-prinsip-scalability)
8. [Prinsip Consistency](#8-prinsip-consistency)
9. [Aturan Penulisan Dokumentasi](#9-aturan-penulisan-dokumentasi)
10. [Aturan Heading](#10-aturan-heading)
11. [Aturan Naming](#11-aturan-naming)
12. [Aturan Struktur Project](#12-aturan-struktur-project)
13. [Aturan Penggunaan Diagram](#13-aturan-penggunaan-diagram)
14. [Aturan Update Dokumentasi](#14-aturan-update-dokumentasi)
15. [Workflow Update Dokumentasi](#15-workflow-update-dokumentasi)
16. [Dependency Antar Dokumen](#16-dependency-antar-dokumen)
17. [Onboarding Philosophy](#17-onboarding-philosophy)
18. [Dokumentasi Lifecycle](#18-dokumentasi-lifecycle)
19. [Versioning Documentation](#19-versioning-documentation)

---

## 1. Tujuan PSD

PSD (Product/System Documentation) adalah lapisan dokumentasi utama yang menjelaskan gambaran sistem secara menyeluruh dari perspektif teknis dan bisnis tingkat tinggi. PSD berfungsi sebagai fondasi pemahaman sistem yang harus dikuasai oleh setiap anggota tim sebelum menyentuh kode atau infrastruktur.

### 1.1 Tujuan Utama

PSD bertujuan untuk:

1. **Menjamin keberlangsungan project** - Memastikan project tidak bergantung pada pengetahuan personal developer tertentu, sehingga knowledge dapat diwariskan secara struktural.
2. **Mempercepat onboarding** - Membantu developer baru memahami sistem dalam waktu singkat tanpa harus membaca seluruh codebase atau bertanya kepada developer lama.
3. **Menjaga konsistensi implementasi** - Menyediakan standar yang jelas sehingga seluruh anggota tim mengikuti pola yang sama dalam pengembangan.
4. **Mendukung maintainability jangka panjang** - Mendokumentasikan keputusan arsitektur, standar coding, dan Business Flow agar system tetap mudah dipelihara seiring pertumbuhan tim dan fitur.
5. **Menjadi referensi tunggal** - Berfungsi sebagai sumber kebenaran (single source of truth) untuk pertanyaan-pertanyaan tingkat tinggi mengenai sistem.

### 1.2 Pernyataan Tujuan

> PSD hadir untuk menjawab pertanyaan: *“Apa sistem ini, bagaimana cara kerjanya secara garis besar, bagaimana cara menjalankannya, dan bagaimana cara berkontribusi dengan benar?”*
> 

---

## 2. Ruang Lingkup PSD

### 2.1 Cakupan PSD

PSD mencakup topik-topik berikut:

| Topik | Keterangan |
| --- | --- |
| Gambaran Sistem | Deskripsi sistem, tujuan bisnis, dan konteks domain |
| Struktur Project | Penjelasan direktori, modul, dan tanggung jawab tiap komponen |
| Setup Project | Langkah instalasi, konfigurasi Environment, dan cara menjalankan project |
| Arsitektur Sistem | Pola arsitektur, alur data, hubungan antar komponen, dan keputusan desain |
| Standar Coding | Konvensi penamaan, struktur kode, pola yang digunakan, dan aturan tim |
| Onboarding Developer | Panduan lengkap bagi developer baru untuk memulai kontribusi |

### 2.2 Yang Tidak Dicakup PSD

PSD secara eksplisit **tidak** membahas topik berikut. Topik-topik ini didelegasikan kepada dokumen TSD (Technical/Service Documentation) atau Log Documentation:

| Topik | Delegasi Ke |
| --- | --- |
| Query SQL kompleks dan optimasi database | TSD - Database Documentation |
| Detail implementasi API (request/response schema, validasi field) | TSD - API Documentation |
| Detail skema database tingkat rendah (index, constraint, trigger) | TSD - Database Schema Documentation |
| Konfigurasi infrastruktur deployment (server, container, CI/CD pipeline) | TSD - Deployment Documentation |
| Debugging mendalam dan troubleshooting spesifik | Log Documentation |
| Algoritma atau logika bisnis kompleks spesifik per modul | TSD - Module Documentation |

---

## 3. Target Pembaca

PSD ditulis untuk memenuhi kebutuhan beberapa kelompok pembaca dengan tingkat keterlibatan yang berbeda.

### 3.1 Pembaca Utama

| Kelompok Pembaca | Kebutuhan |
| --- | --- |
| Developer Baru | Memahami sistem, menjalankan project, dan mulai berkontribusi |
| Developer Senior | Memahami keputusan arsitektur dan memastikan konsistensi standar |
| Tech Lead | Mereview dan menjaga kualitas dokumentasi serta standar development |
| Software Architect | Memvalidasi kesesuaian implementasi dengan desain arsitektur |

### 3.2 Pembaca Sekunder

| Kelompok Pembaca | Kebutuhan |
| --- | --- |
| Product Manager | Memahami batasan teknis dan gambaran sistem secara non-teknis |
| QA Engineer | Memahami alur sistem untuk keperluan pengujian |
| DevOps Engineer | Memahami kebutuhan sistem sebelum merancang infrastruktur |

### 3.3 Asumsi Pengetahuan Pembaca

Dokumentasi PSD mengasumsikan pembaca memiliki:

- Pemahaman dasar tentang pengembangan perangkat lunak
- Kemampuan membaca kode pada bahasa pemrograman yang digunakan project
- Pemahaman dasar tentang konsep API, Database, dan arsitektur aplikasi
- Kemampuan mengoperasikan terminal dan perintah dasar Git

---

## 4. Struktur Dokumentasi PSD

### 4.1 Hierarki File PSD

Setiap project menggunakan struktur file PSD yang terstandarisasi sebagai berikut:

```
docs/
└── psd/
    ├── 00-fundamental-psd.md         # Fondasi dan aturan dokumentasi PSD
    ├── 01-project-overview.md        # Gambaran umum project dan konteks bisnis
    ├── 02-setup-guide.md             # Panduan instalasi dan konfigurasi
    ├── 03-architecture.md            # Arsitektur sistem dan keputusan desain
    ├── 04-coding-standard.md         # Standar dan konvensi penulisan kode
    └── 05-onboarding-guide.md        # Panduan lengkap developer baru
```

### 4.2 Deskripsi Tiap File

| File | Tujuan | Pembaca Utama |
| --- | --- | --- |
| `00-fundamental-psd.md` | Mendefinisikan aturan, filosofi, dan standar dokumentasi PSD itu sendiri | Tech Lead, Architect |
| `01-project-overview.md` | Menjelaskan konteks bisnis, tujuan sistem, dan gambaran fungsionalitas | Semua |
| `02-setup-guide.md` | Panduan step-by-step untuk menjalankan project di environment lokal | Developer Baru |
| `03-architecture.md` | Menjelaskan pola arsitektur, komponen utama, dan alur data sistem | Developer, Architect |
| `04-coding-standard.md` | Mendefinisikan konvensi, pola, dan standar kode yang wajib diikuti | Developer |
| `05-onboarding-guide.md` | Panduan komprehensif dari setup hingga kontribusi pertama | Developer Baru |

### 4.3 Urutan Pembacaan yang Direkomendasikan

Untuk developer baru, urutan pembacaan dokumen PSD yang direkomendasikan adalah:

```
00-fundamental-psd.md
        |
        v
01-project-overview.md
        |
        v
02-setup-guide.md
        |
        v
03-architecture.md
        |
        v
04-coding-standard.md
        |
        v
05-onboarding-guide.md
```

---

## 5. Hubungan PSD dengan TSD

### 5.1 Definisi Lapisan Dokumentasi

Sistem dokumentasi project terbagi ke dalam dua lapisan utama yang saling melengkapi:

| Lapisan | Singkatan | Fokus | Tingkat Detail |
| --- | --- | --- | --- |
| Product/System Documentation | PSD | Gambaran sistem, arsitektur, setup, standar | High-level |
| Technical/Service Documentation | TSD | Detail implementasi, API, Database, deployment | Low-level |

### 5.2 Pembagian Tanggung Jawab

```
PSD (High-Level)
├── Apa yang dilakukan sistem?
├── Bagaimana cara menjalankan sistem?
├── Bagaimana arsitektur sistem secara umum?
├── Apa standar pengembangan yang berlaku?
└── Bagaimana cara berkontribusi?

TSD (Low-Level)
├── Bagaimana detail implementasi tiap Service?
├── Apa saja endpoint API dan kontraknya?
├── Bagaimana skema Database secara lengkap?
├── Bagaimana konfigurasi deployment?
└── Bagaimana cara debug masalah spesifik?
```

### 5.3 Aturan Referensi Silang

- PSD **boleh merujuk** ke TSD untuk topik yang memerlukan penjelasan lebih mendalam.
- TSD **wajib merujuk balik** ke PSD untuk konteks arsitektur dan standar yang berlaku.
- PSD **tidak boleh menduplikasi** informasi yang sudah ada di TSD.
- Setiap referensi silang harus menggunakan path relatif yang konsisten.

Contoh referensi dari PSD ke TSD:

```markdown
Untuk detail skema Database, lihat: `docs/tsd/database/schema.md`
Untuk dokumentasi API lengkap, lihat: `docs/tsd/api/endpoints.md`
```

---

## 6. Prinsip Maintainability

### 6.1 Definisi

Maintainability dalam konteks dokumentasi berarti dokumentasi harus mudah diperbarui, dipahami, dan divalidasi kebenarannya seiring perubahan sistem.

### 6.2 Aturan Maintainability

**Aturan 1: Satu Sumber Kebenaran**
Setiap informasi hanya boleh ada di satu lokasi. Informasi yang sama di dua tempat berbeda akan menjadi tidak konsisten seiring waktu. Gunakan referensi silang, bukan duplikasi.

**Aturan 2: Dokumentasi Dekat dengan Kode**
Perubahan kode yang berdampak pada arsitektur atau standar harus segera diikuti dengan pembaruan dokumentasi pada sprint atau pull request yang sama.

**Aturan 3: Gunakan Format yang Bertahan Lama**
Gunakan format Markdown murni tanpa dependensi pada tool eksternal yang dapat berubah. Diagram menggunakan format teks (Mermaid atau ASCII) sehingga dapat dirender di berbagai platform.

**Aturan 4: Hindari Referensi yang Mudah Kadaluwarsa**
Hindari menyebutkan nama orang, tanggal spesifik implementasi, atau detail yang sering berubah di dalam penjelasan utama. Tempatkan informasi tersebut di bagian changelog atau catatan versi.

**Aturan 5: Dokumentasi Harus Dapat Divalidasi**
Setiap klaim teknis dalam dokumentasi harus dapat diverifikasi langsung dari codebase atau konfigurasi sistem. Hindari pernyataan yang tidak dapat divalidasi.

---

## 7. Prinsip Scalability

### 7.1 Definisi

Scalability dalam konteks dokumentasi berarti struktur dokumentasi harus tetap berfungsi dengan baik ketika project tumbuh dalam hal fitur, komponen, anggota tim, dan kompleksitas.

### 7.2 Aturan Scalability

**Aturan 1: Struktur Modular**
Setiap dokumen membahas satu topik spesifik. Ketika sebuah topik berkembang terlalu besar untuk satu file, pecah menjadi sub-dokumen dalam direktori tersendiri.

Contoh ekspansi modular:

```
# Sebelum ekspansi
docs/psd/03-architecture.md

# Setelah ekspansi (ketika konten terlalu besar)
docs/psd/architecture/
├── 03-architecture-overview.md
├── 03a-backend-architecture.md
├── 03b-frontend-architecture.md
└── 03c-integration-architecture.md
```

**Aturan 2: Naming yang Prediktif**
Gunakan konvensi penamaan file yang memungkinkan penambahan dokumen baru tanpa merusak urutan atau konteks yang sudah ada.

**Aturan 3: Indeks Dokumen**
Setiap direktori dokumentasi yang berisi lebih dari satu file wajib memiliki file `README.md` atau indeks yang menavigasi ke seluruh dokumen di dalamnya.

**Aturan 4: Abstraksi yang Tepat**
PSD harus tetap berada di level abstraksi yang benar. Ketika detail teknis semakin banyak, pindahkan ke TSD dan sisakan hanya ringkasan di PSD dengan referensi yang jelas.

---

## 8. Prinsip Consistency

### 8.1 Definisi

Consistency berarti seluruh dokumen dalam ekosistem PSD menggunakan terminologi, format, dan struktur yang seragam, sehingga pembaca tidak perlu menyesuaikan diri setiap membuka dokumen baru.

### 8.2 Terminologi Standar

Berikut adalah daftar terminologi yang wajib digunakan secara konsisten di seluruh dokumentasi. Penggunaan sinonim atau variasi tidak diperbolehkan.

| Istilah Resmi | Dilarang Diganti Dengan |
| --- | --- |
| Controller | Handler, Router, Endpoint |
| Service | Manager, Helper, Logic, Usecase |
| Repository | DAO, Store, Persistence, DataAccess |
| Model | Entity, Schema, Object, Struct |
| Module | Feature, Domain, Package, Layer |
| Database | DB, Storage, Data Store |
| API | Endpoint, Interface, Service (dalam konteks ini) |
| Environment | Env, Config, Setting |
| Deployment | Deploy, Release, Ship |
| Business Flow | Business Logic, Use Case Flow, Process Flow |

### 8.3 Konsistensi Format

Seluruh dokumen PSD wajib menggunakan:

- Format tanggal: `YYYY-MM-DD` (contoh: `2025-01-15`)
- Format versi: `MAJOR.MINOR.PATCH` (contoh: `1.2.3`)
- Code block dengan label bahasa eksplisit (contoh: ````bash`, ````json`, ````yaml`)
- Tabel untuk data komparatif atau daftar yang memiliki lebih dari satu atribut

### 8.4 Konsistensi Antar Dokumen

Setiap dokumen dalam PSD harus sinkron satu sama lain. Aturan sinkronisasi:

| Dokumen Sumber | Harus Sinkron Dengan |
| --- | --- |
| `03-architecture.md` | `04-coding-standard.md` (pola yang dijelaskan harus konsisten) |
| `02-setup-guide.md` | `03-architecture.md` (komponen yang di-setup harus sesuai arsitektur) |
| `01-project-overview.md` | `02-setup-guide.md` (teknologi yang disebutkan harus ada di setup) |
| `04-coding-standard.md` | `05-onboarding-guide.md` (standar harus diajarkan di onboarding) |

---

## 9. Aturan Penulisan Dokumentasi

### 9.1 Bahasa

- Gunakan Bahasa Indonesia formal teknis.
- Kalimat harus jelas, padat, dan tidak ambigu.
- Hindari bahasa santai, slang, atau ungkapan tidak formal.
- Gunakan kalimat aktif lebih sering daripada kalimat pasif.
- Satu paragraf maksimal membahas satu ide utama.

### 9.2 Gaya Penulisan

| Aspek | Aturan |
| --- | --- |
| Panjang kalimat | Maksimal 25 kata per kalimat |
| Panjang paragraf | Maksimal 5 kalimat per paragraf |
| Panjang dokumen | Tidak dibatasi, namun wajib memiliki daftar isi |
| Placeholder | Gunakan format `[KETERANGAN]` yang deskriptif, bukan lorem ipsum |
| Asumsi | Nyatakan asumsi secara eksplisit, jangan biarkan tersirat |

### 9.3 Elemen yang Dilarang

- Emoji dalam konten teknis
- Bahasa hiperbola atau marketing
- Klaim tanpa dasar yang dapat diverifikasi
- Placeholder `TBD`, `TODO`, atau `coming soon` tanpa tiket atau estimasi
- Referensi ke nama orang spesifik sebagai pemilik permanen suatu komponen

### 9.4 Elemen yang Diwajibkan

Setiap dokumen PSD wajib memiliki header metadata di bagian atas:

```markdown
# [Judul Dokumen]

**Versi Dokumen:** [X.X.X]
**Status:** [Draft | Active | Deprecated]
**Kategori:** [Foundation | Architecture | Setup | Standard | Onboarding]
**Terakhir Diperbarui:** [YYYY-MM-DD]
**Pemilik Dokumen:** [NAMA TIM / ROLE]
```

---

## 10. Aturan Heading

### 10.1 Hierarki Heading

Struktur heading yang wajib diikuti:

```
# Judul Dokumen (H1)         - Hanya satu per dokumen, di baris pertama
## Bagian Utama (H2)         - Topik-topik besar dalam dokumen
### Sub-Bagian (H3)          - Penjelasan lebih lanjut dari H2
#### Detail (H4)             - Rincian spesifik, digunakan secukupnya
```

### 10.2 Aturan Penggunaan Heading

- H1 hanya digunakan satu kali per dokumen sebagai judul utama.
- H2 digunakan untuk topik utama yang tercantum dalam daftar isi.
- H3 digunakan untuk sub-topik di dalam H2.
- H4 digunakan hanya jika benar-benar diperlukan untuk kedalaman tertentu.
- Tidak boleh ada heading H5 atau lebih dalam.
- Tidak boleh ada dua heading berturut-turut tanpa konten di antaranya.
- Heading tidak diakhiri dengan tanda titik atau tanda baca lainnya.

### 10.3 Format Penomoran Heading

Gunakan penomoran eksplisit pada H2 dan H3 untuk memudahkan referensi:

```markdown
## 1. Tujuan Sistem
### 1.1 Latar Belakang
### 1.2 Permasalahan yang Diselesaikan

## 2. Arsitektur
### 2.1 Komponen Utama
### 2.2 Alur Data
```

---

## 11. Aturan Naming

### 11.1 Penamaan File Dokumentasi

| Aturan | Contoh Benar | Contoh Salah |
| --- | --- | --- |
| Gunakan huruf kecil semua | `02-setup-guide.md` | `02-Setup-Guide.md` |
| Gunakan tanda hubung sebagai pemisah | `coding-standard.md` | `coding_standard.md` |
| Awali dengan nomor urut dua digit | `03-architecture.md` | `3-architecture.md` |
| Nama file deskriptif dan spesifik | `01-project-overview.md` | `01-doc.md` |
| Ekstensi selalu `.md` | `setup-guide.md` | `setup-guide.txt` |

### 11.2 Penamaan Direktori

| Aturan | Contoh Benar | Contoh Salah |
| --- | --- | --- |
| Huruf kecil semua | `docs/psd/` | `docs/PSD/` |
| Tanda hubung sebagai pemisah | `api-reference/` | `api_reference/` |
| Nama singkat dan deskriptif | `architecture/` | `arch-docs-v2/` |

### 11.3 Penamaan Komponen dalam Dokumentasi

Ketika menyebutkan komponen kode dalam dokumentasi, gunakan format berikut:

| Komponen | Format Penulisan | Contoh |
| --- | --- | --- |
| Class atau Struct | PascalCase dalam backtick | `UserController` |
| Method atau Function | camelCase dalam backtick | `getUserById()` |
| File | nama file dalam backtick | `user.service.ts` |
| Direktori | path dalam backtick | `src/modules/user/` |
| Environment Variable | UPPER_SNAKE_CASE dalam backtick | `DATABASE_URL` |
| Command | dalam code block | `npm install` |

---

## 12. Aturan Struktur Project

### 12.1 Prinsip Struktur Direktori

Dokumentasi PSD wajib menjelaskan struktur direktori project mengikuti prinsip berikut:

**Prinsip 1: Grup Berdasarkan Modul, Bukan Tipe File**
Struktur yang direkomendasikan mengelompokkan file berdasarkan Module atau domain, bukan berdasarkan tipe file (controller, service, model dalam satu folder masing-masing).

```
# Direkomendasikan - Grup berdasarkan Module
src/
└── modules/
    └── user/
        ├── user.controller.ts
        ├── user.service.ts
        ├── user.repository.ts
        └── user.model.ts

# Tidak direkomendasikan - Grup berdasarkan tipe
src/
├── controllers/
│   └── user.controller.ts
├── services/
│   └── user.service.ts
└── models/
    └── user.model.ts
```

**Prinsip 2: Konsistensi Antar Module**
Setiap Module harus memiliki struktur internal yang identik. Perbedaan struktur antar Module harus didokumentasikan dengan alasan yang jelas.

**Prinsip 3: Depth yang Terkontrol**
Kedalaman direktori tidak boleh lebih dari 5 level dari root project untuk mencegah kompleksitas navigasi yang berlebihan.

### 12.2 Dokumentasi Struktur Project

Setiap dokumen PSD yang menampilkan struktur direktori wajib menggunakan format tree dengan keterangan:

```
src/
├── modules/               # Kumpulan Module bisnis utama
│   ├── user/              # Module pengelolaan pengguna
│   └── product/           # Module pengelolaan produk
├── shared/                # Komponen yang digunakan lintas Module
│   ├── middleware/        # Middleware global
│   └── utils/             # Fungsi utilitas umum
└── config/                # Konfigurasi Environment dan aplikasi
```

---

## 13. Aturan Penggunaan Diagram

### 13.1 Kapan Diagram Digunakan

Diagram digunakan ketika penjelasan teks tidak cukup untuk menyampaikan hubungan atau alur yang kompleks. Berikut panduan penggunaan:

| Situasi | Gunakan Diagram | Cukup Teks |
| --- | --- | --- |
| Alur data antar komponen | Ya |  |
| Hierarki komponen sistem | Ya |  |
| Sequence interaksi antar layanan | Ya |  |
| Deskripsi satu komponen tunggal |  | Ya |
| Penjelasan parameter atau konfigurasi |  | Ya |
| Daftar fitur atau kemampuan sistem |  | Ya |

### 13.2 Format Diagram yang Diperbolehkan

| Format | Digunakan Untuk | Alat |
| --- | --- | --- |
| Mermaid | Flowchart, Sequence Diagram, Entity Diagram | Mermaid syntax dalam code block |
| ASCII Art | Diagram sederhana yang tidak butuh tool khusus | Teks biasa |

Contoh penggunaan Mermaid dalam dokumen:

```markdown
    ```mermaid
    graph TD
        A[Client] --> B[Controller]
        B --> C[Service]
        C --> D[Repository]
        D --> E[(Database)]
    ```
```

### 13.3 Aturan Diagram

- Setiap diagram wajib memiliki judul dan keterangan singkat.
- Diagram harus dapat dirender di GitHub atau platform Markdown standar.
- Jangan menggunakan screenshot diagram dari tool eksternal seperti Miro atau Figma sebagai gambar statis, karena tidak dapat diperbarui dengan mudah.
- Jika menggunakan file gambar (`.png`, `.svg`), simpan di direktori `docs/assets/` dan sertakan source file yang dapat diedit.

---

## 14. Aturan Update Dokumentasi

### 14.1 Trigger Wajib Update Dokumentasi

Pembaruan dokumentasi PSD wajib dilakukan ketika terjadi perubahan berikut:

| Perubahan | Dokumen yang Harus Diperbarui |
| --- | --- |
| Penambahan Module baru | `01-project-overview.md`, `03-architecture.md` |
| Perubahan arsitektur sistem | `03-architecture.md`, `04-coding-standard.md` |
| Perubahan requirement Environment | `02-setup-guide.md` |
| Perubahan konvensi atau standar kode | `04-coding-standard.md`, `05-onboarding-guide.md` |
| Penambahan dependensi mayor | `01-project-overview.md`, `02-setup-guide.md` |
| Perubahan struktur direktori | `03-architecture.md` |
| Perubahan Business Flow utama | `01-project-overview.md`, `03-architecture.md` |

### 14.2 Aturan Pembaruan

- Pembaruan dokumentasi **harus dilakukan di pull request yang sama** dengan perubahan kode yang memicunya.
- Pull request yang mengubah arsitektur atau standar **tidak akan disetujui** tanpa pembaruan dokumentasi yang sesuai.
- Versi dokumen wajib dinaikkan setiap kali terjadi pembaruan konten.
- Changelog di bagian akhir dokumen wajib diperbarui dengan deskripsi singkat perubahan.

### 14.3 Kriteria Dokumen Tidak Perlu Diperbarui

Pembaruan dokumentasi PSD **tidak diperlukan** untuk:

- Perbaikan bug yang tidak mengubah arsitektur atau alur sistem
- Penambahan unit test
- Refactoring internal yang tidak mengubah kontrak komponen
- Perubahan konfigurasi yang bersifat sementara

---

## 15. Workflow Update Dokumentasi

### 15.1 Alur Standar Pembaruan Dokumentasi

```
1. Developer melakukan perubahan kode atau arsitektur
        |
        v
2. Developer mengidentifikasi dokumen yang terdampak
   (rujuk tabel trigger di Bagian 14.1)
        |
        v
3. Developer memperbarui dokumen yang terdampak
        |
        v
4. Developer menaikkan versi dokumen
   dan memperbarui changelog
        |
        v
5. Reviewer memvalidasi konsistensi antar dokumen
   (rujuk tabel sinkronisasi di Bagian 8.4)
        |
        v
6. Dokumentasi disetujui bersama kode
        |
        v
7. Merge ke branch utama
```

### 15.2 Checklist Review Dokumentasi

Reviewer dokumentasi wajib memvalidasi hal berikut sebelum menyetujui:

```
[ ] Terminologi konsisten dengan daftar istilah resmi (Bagian 8.2)
[ ] Format heading sesuai aturan (Bagian 10)
[ ] Struktur direktori yang disebutkan sesuai kondisi aktual
[ ] Referensi silang antar dokumen masih valid
[ ] Tidak ada duplikasi informasi dengan dokumen lain
[ ] Versi dokumen telah dinaikkan
[ ] Changelog telah diperbarui
[ ] Header metadata dokumen telah diperbarui
[ ] Diagram (jika ada) masih akurat dengan kondisi sistem
```

### 15.3 Penanggung Jawab Dokumentasi

| Peran | Tanggung Jawab |
| --- | --- |
| Developer | Memperbarui dokumen yang terdampak oleh perubahannya |
| Tech Lead | Mereview dan memastikan kualitas serta konsistensi dokumentasi |
| Team Lead / Engineering Manager | Memastikan budaya dokumentasi dijalankan dalam tim |

---

## 16. Dependency Antar Dokumen

### 16.1 Peta Dependency

Berikut adalah peta dependency antar dokumen PSD. Dokumen di sebelah kiri memiliki ketergantungan terhadap dokumen di sebelah kanan:

| Dokumen | Bergantung Pada | Keterangan |
| --- | --- | --- |
| `02-setup-guide.md` | `01-project-overview.md` | Setup harus konsisten dengan teknologi yang dideklarasikan di overview |
| `03-architecture.md` | `01-project-overview.md` | Arsitektur harus mendukung tujuan bisnis yang dijelaskan di overview |
| `04-coding-standard.md` | `03-architecture.md` | Standar kode harus mencerminkan pola arsitektur yang dipilih |
| `05-onboarding-guide.md` | `02-setup-guide.md`, `04-coding-standard.md` | Onboarding menggabungkan setup dan standar sebagai panduan terpadu |
| TSD (semua dokumen) | `03-architecture.md` | Semua TSD harus merujuk pada konteks arsitektur yang didefinisikan di PSD |

### 16.2 Dampak Perubahan

Ketika sebuah dokumen diperbarui, dokumen berikut harus diperiksa konsistensinya:

```
Jika 01-project-overview.md diubah:
  -> Periksa 02-setup-guide.md
  -> Periksa 03-architecture.md

Jika 03-architecture.md diubah:
  -> Periksa 04-coding-standard.md
  -> Periksa 05-onboarding-guide.md
  -> Periksa semua TSD yang mereferensikan arsitektur

Jika 04-coding-standard.md diubah:
  -> Periksa 05-onboarding-guide.md
```

---

## 17. Onboarding Philosophy

### 17.1 Prinsip Dasar Onboarding

Dokumentasi PSD didesain berdasarkan filosofi bahwa:

> *Seorang developer baru harus mampu memahami sistem, menjalankan project di environment lokal, dan membuat kontribusi pertamanya hanya dengan membaca dokumentasi PSD, tanpa perlu bertanya kepada siapa pun.*
> 

### 17.2 Standar Onboarding yang Dapat Diukur

Dokumentasi PSD dianggap berhasil jika developer baru dapat mencapai milestone berikut:

| Milestone | Target Waktu | Dokumen yang Menunjang |
| --- | --- | --- |
| Memahami tujuan sistem dan konteks bisnis | 1 jam pertama | `01-project-overview.md` |
| Menjalankan project di environment lokal | 2 jam pertama | `02-setup-guide.md` |
| Memahami arsitektur dan komponen utama | Hari pertama | `03-architecture.md` |
| Memahami standar kode yang berlaku | Hari pertama | `04-coding-standard.md` |
| Siap membuat kontribusi pertama | Akhir hari kedua | `05-onboarding-guide.md` |

### 17.3 Tanggung Jawab Dokumentasi terhadap Onboarding

- Dokumentasi bertanggung jawab atas **efisiensi** onboarding, bukan tim yang melakukan mentoring.
- Pertanyaan yang sering diajukan developer baru adalah indikasi dokumentasi yang perlu diperbaiki.
- Setiap developer baru yang selesai onboarding wajib memberikan feedback tertulis tentang bagian dokumentasi yang kurang jelas.

---

## 18. Dokumentasi Lifecycle

### 18.1 Status Dokumen

Setiap dokumen PSD memiliki siklus hidup dengan status yang terdefinisi:

| Status | Deskripsi | Aksi yang Diperlukan |
| --- | --- | --- |
| `Draft` | Dokumen sedang dalam proses penulisan, belum divalidasi | Tidak dapat dijadikan referensi resmi |
| `Active` | Dokumen sudah divalidasi, akurat, dan dapat digunakan | Dokumen utama yang dirujuk tim |
| `Deprecated` | Dokumen tidak lagi relevan namun disimpan untuk referensi historis | Harus ada dokumen pengganti yang ditunjuk |
| `Archived` | Dokumen dari versi sistem yang sudah tidak aktif | Dipindahkan ke direktori `docs/archive/` |

### 18.2 Transisi Status

```
Draft --> Active       : Setelah review dan persetujuan Tech Lead
Active --> Deprecated  : Ketika ada dokumen pengganti yang Active
Deprecated --> Archived: Setelah 2 sprint sejak status Deprecated
```

### 18.3 Retensi Dokumen

- Dokumen `Archived` disimpan minimal selama **1 tahun** setelah diarsipkan.
- Dokumen `Deprecated` harus menyertakan referensi ke dokumen penggantinya.
- Tidak ada dokumen yang boleh dihapus permanen tanpa persetujuan Tech Lead.

---

## 19. Versioning Documentation

### 19.1 Skema Versioning

Dokumentasi PSD menggunakan skema versioning `MAJOR.MINOR.PATCH`:

| Segmen | Kapan Dinaikkan | Contoh Perubahan |
| --- | --- | --- |
| `MAJOR` | Perubahan struktural besar atau perubahan arah sistem yang signifikan | Pergantian arsitektur utama, restrukturisasi seluruh dokumentasi |
| `MINOR` | Penambahan konten baru atau perubahan yang memperluas dokumentasi | Penambahan section baru, penambahan diagram, perubahan standar |
| `PATCH` | Koreksi minor, perbaikan typo, klarifikasi kalimat | Perbaikan ejaan, klarifikasi kalimat ambigu, pembaruan referensi |

### 19.2 Changelog Format

Setiap dokumen PSD wajib menyertakan bagian Changelog di akhir dokumen dengan format berikut:

```markdown
---

## Changelog

| Versi | Tanggal | Perubahan | Penulis |
|---|---|---|---|
| 1.0.0 | YYYY-MM-DD | Initial release | [NAMA / ROLE] |
| 1.1.0 | YYYY-MM-DD | Penambahan section [X] | [NAMA / ROLE] |
| 1.1.1 | YYYY-MM-DD | Koreksi typo pada section [Y] | [NAMA / ROLE] |
```

### 19.3 Sinkronisasi Versi dengan Sistem

Versi dokumentasi PSD tidak harus sinkron dengan versi aplikasi. Namun, ketika terjadi rilis mayor aplikasi, tim wajib melakukan audit dokumentasi untuk memastikan seluruh dokumen PSD masih akurat dan memperbarui versi dokumen yang terdampak.

### 19.4 Tagging di Version Control

Setiap rilis dokumentasi yang signifikan (kenaikan `MAJOR` atau `MINOR`) harus di-tag di repository dengan konvensi:

```
docs/psd/v[MAJOR].[MINOR]
```

Contoh: `docs/psd/v1.2` menandai kondisi dokumentasi PSD pada versi minor ke-2.

---

## Changelog

| Versi | Tanggal | Perubahan | Penulis |
| --- | --- | --- | --- |
| 1.0.0 | [TANGGAL] | Initial release - Penulisan seluruh fundamental PSD | [NAMA / ROLE] |