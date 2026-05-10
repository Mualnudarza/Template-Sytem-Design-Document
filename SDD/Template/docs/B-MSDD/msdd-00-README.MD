# MSDD — Markdown Software Design Description

Folder ini berisi dokumentasi desain perangkat lunak (*Software Design Description*) berbasis Markdown.

MSDD mendeskripsikan:

- Bagaimana sistem dibangun
- Struktur arsitektur
- Interaksi antar komponen
- Infrastruktur dan deployment
- Rationale desain teknis

---

# Struktur Dokumen

| File | Deskripsi |
|---|---|
| `msdd-01-introduction.md` | Pendahuluan dokumen desain |
| `msdd-02-design-overview.md` | Gambaran umum arsitektur |
| `msdd-03-design-views.md` | Detail representasi desain |
| `msdd-04-decisions.md` | Referensi keputusan desain |
| `msdd-05-appendixes.md` | Lampiran tambahan |

---


# Viewpoint yang Direkomendasikan

| Viewpoint | Deskripsi | Diagram Umum |
|---|---|---|
| Context | Sistem sebagai *black-box*. Menjelaskan batasan sistem, aktor lingkungan, dan layanan eksternal. | UML Use Case, C4 Context |
| Composition | Bagaimana sistem dirakit dan dipecah menjadi subsistem/komponen/modul utama. Menentukan strategi *buy-vs-build*. | UML Component, Hierarchical Decomposition |
| Logical | Struktur rancangan statis dari tipe dan implementasinya, enkapsulasi, serta abstraksi. | UML Class, UML Object |
| Physical | Topologi fisik dan infrastruktur tangibel tempat sistem berjalan. | Hardware Block, Network Topology, Rack Layout |
| Structure | Organisasi internal dari entitas yang lebih kompleks, mendokumentasikan antarmuka (*ports*) dan konektor (*connectors*). | UML Composite Structure, C4 Container |
| Dependency | Pemetaan grafis yang menunjukkan integrasi dan arah ketergantungan antar elemen (*uses, requires, provides*). Digunakan untuk analisis dampak perubahan. | Dependency Graph, UML Package |
| Information | Semantik persisten data, hubungan antar data, mekanisme pengelolaan/akses data, dan pemeliharaan integritas. | ERD, Logical Data Model |
| Interface | Spesifikasi kontrak kolaborasi yang terlihat oleh publik/sistem eksternal. | API Specs, IDL, Function Signatures |
| Interaction | Kolaborasi lintas entitas saat *runtime*; mencakup urutan pesan, sinkronisasi waktu, perambatan *error*, dan transisi logika. | UML Sequence, BPMN |
| Algorithm | Rincian logika internal (*time-space complexity*) dari suatu operasi kritikal (langkah, keputusan, iterasi, penanganan masalah deterministik/AI). | Pseudocode, Flowchart |
| State Dynamics | Rincian mengenai bagaimana komponen merespons stimulus tak terduga dan bagaimana sistem bertransisi antar-mode. | UML State Machine, Automata |
| Concurrency | Penanganan pemrosesan paralel, struktur antrean/utas (*threads*), mekanisme penguncian (*locking*), kontrol sinkronisasi, dan pencegahan *race-conditions*. | Concurrency Diagram, Thread Model |
| Patterns | Mengidentifikasi ide desain yang digunakan ulang (*Design Patterns* seperti MVC, *Microservices*, *Event-Driven*) yang memandu konsistensi arsitektur. | Pattern Diagram, Architecture Pattern |
| Deployment | Bagaimana entitas perangkat lunak dipetakan ke dalam lingkungan eksekusi fisik atau komputasi *cloud*. | UML Deployment, IaC Topology, CI/CD Pipelines |
| Resources | Bagaimana sistem mengelola utilitas sumber daya terbatas secara efisien (seperti alokasi memori, batas kuota koneksi basis data, prioritas *bandwidth*). | Resource Allocation Diagram, Capacity Model |

---

# Prinsip Penulisan

- Fokus pada “How”
- Setiap desain harus dapat ditelusuri ke requirement
- Gunakan diagram jika memungkinkan
- Hindari keputusan tanpa justifikasi
- Tautkan desain dengan MADR terkait