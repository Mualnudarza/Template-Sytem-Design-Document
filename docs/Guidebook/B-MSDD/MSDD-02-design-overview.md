# BAB 2: DESIGN OVERVIEW

Bagian ini mendeskripsikan arah pendekatan arsitektur dan sifat sistem secara makro. Bab ini mendefinisikan siapa pemangku kepentingan utama beserta kekhawatiran teknisnya, serta mengidentifikasi sudut pandang (viewpoints) arsitektur yang digunakan untuk mendeskripsikan sistem secara komprehensif.

---

## 2.1 Stakeholder Concerns

Bagian ini mengidentifikasi pemangku kepentingan utama sistem beserta kekhawatiran desain spesifik yang harus dijawab oleh arsitektur yang dibangun.

---

**Perintah (Instructions)**

Identifikasi seluruh kelas pemangku kepentingan yang memiliki kepentingan terhadap desain sistem, termasuk pengguna akhir, operator, tim pengembang, tim QA, manajemen, regulator, dan tim keamanan. Untuk setiap kelompok, jelaskan kekhawatiran desain spesifik mereka — misalnya pengguna membutuhkan ketersediaan tinggi, operator membutuhkan kemudahan deployment, manajemen membutuhkan keamanan data. Tunjukkan secara eksplisit bagaimana desain yang diajukan menjawab setiap kekhawatiran tersebut. Gunakan tabel untuk kejelasan struktur.

**Catatan:** Kekhawatiran pemangku kepentingan yang terdaftar di sini harus dapat dipetakan kembali ke persyaratan non-fungsional (Quality of Service) pada MSRS sub-bab 3.3. Jika terdapat kekhawatiran baru yang belum tercakup dalam MSRS, catat sebagai gap dan eskalasikan ke Product Owner.

### Contoh (Example)

| Pemangku Kepentingan | Peran | Kekhawatiran Utama | Jawaban Desain |
| --- | --- | --- | --- |
| End User | Konsumen layanan | Ketersediaan tinggi & respons cepat | Implementasi load balancer dan caching layer; target latensi < 200ms |
| Software Engineer | Implementor | Struktur kode yang modular & testable | Penerapan pola Clean Architecture dengan dependency injection |
| DevOps / SRE | Operator produksi | Kemudahan deployment & observabilitas | Containerisasi via Docker, pipeline CI/CD otomatis, dan integrasi APM |
| QA Engineer | Verifikator kualitas | Keterlacakan desain ke pengujian | Setiap Design View mereferensikan REQ-ID dari MSRS |
| Security Team | Pelindung aset data | Keamanan data sensitif & otentikasi | Enkripsi AES-256 at rest, TLS 1.3 in transit, implementasi OAuth2/OIDC |
| Product Owner | Pemilik produk | Kecepatan delivery & adaptabilitas | Arsitektur modular yang mendukung iterasi fitur tanpa breaking change |
| Compliance Officer | Penjaga regulasi | Kepatuhan terhadap regulasi data | Audit log terpusat, data retention policy, mekanisme anonimisasi PII |

---

## 2.2 Selected Viewpoints

Bagian ini mengidentifikasi sudut pandang (viewpoints) arsitektur yang dipilih untuk mendeskripsikan sistem dalam dokumen ini, beserta tujuan dan kekhawatiran yang dijawab oleh masing-masing viewpoint.

---

**Perintah (Instructions)**

Dari daftar 15 viewpoint arsitektur standar yang tersedia, pilih dan dokumentasikan hanya viewpoint yang paling relevan untuk sistem yang dirancang. Tim tidak diwajibkan menggunakan seluruh 15 viewpoint; pilih yang secara signifikan menjawab Stakeholder Concerns yang telah diidentifikasi pada sub-bab 2.1. Untuk setiap viewpoint yang dipilih, cantumkan nama, tujuan, jenis diagram yang digunakan, dan stakeholder yang dilayani. Viewpoint yang tidak digunakan harus dinyatakan secara eksplisit beserta alasannya untuk menghindari ambiguitas.

**Catatan:** Setiap viewpoint yang dipilih di sini wajib memiliki minimal satu Design View yang sesuai pada Bab 3. Konsistensi antara daftar viewpoint dan isi Bab 3 merupakan persyaratan integritas dokumen ini.

### Contoh (Example)

Berikut adalah viewpoint yang dipilih untuk sistem `{{nama_sistem}}`. Pemilihan didasarkan pada kompleksitas arsitektur, profil risiko, dan kekhawatiran pemangku kepentingan yang telah diidentifikasi.

| # | Viewpoint | Tujuan | Jenis Diagram | Stakeholder Utama |
| --- | --- | --- | --- | --- |
| 1 | **Context** | Menjelaskan batas sistem, aktor eksternal, dan layanan yang dikonsumsi/disediakan | C4 Context, UML Use Case | Product Owner, Architect |
| 2 | **Composition** | Mendeskripsikan bagaimana sistem dipecah menjadi subsistem dan modul utama | UML Component, Hierarchical Decomposition | Software Engineer, Architect |
| 3 | **Logical** | Mendefinisikan abstraksi statis seperti kelas, antarmuka, dan relasi | UML Class, UML Object | Software Engineer, QA Engineer |
| 4 | **Dependency** | Memetakan ketergantungan antar elemen desain untuk analisis dampak perubahan | Dependency Graph, UML Package | Software Engineer, DevOps |
| 5 | **Information** | Mendefinisikan semantik data persisten, relasi antar data, dan manajemen integritas | ERD, Logical Data Model | DBA, Backend Engineer |
| 6 | **Interface** | Menetapkan kontrak kolaborasi yang terlihat publik dan oleh sistem eksternal | API Specification, Function Signatures | Frontend Dev, Integration Specialist |
| 7 | **Interaction** | Mendeskripsikan kolaborasi lintas entitas saat runtime dan urutan pertukaran pesan | UML Sequence, BPMN | Software Engineer, QA Engineer |
| 8 | **Deployment** | Mendefinisikan pemetaan komponen perangkat lunak ke lingkungan eksekusi fisik/cloud | UML Deployment, IaC Topology, CI/CD Pipeline | DevOps/SRE, Cloud Architect |
| 9 | **State Dynamics** | Mendokumentasikan transisi status komponen sebagai respons terhadap stimulus eksternal | UML State Machine | Software Engineer, QA Engineer |

**Viewpoint yang tidak digunakan pada dokumen ini:**

| Viewpoint | Alasan Tidak Digunakan |
| --- | --- |
| Physical | Infrastruktur fisik dikelola sepenuhnya oleh tim Cloud Provider; tidak dalam kendali langsung tim Engineering. |
| Structure | Kompleksitas port dan konektor internal tidak cukup signifikan untuk memerlukan dokumentasi terpisah; sudah tercakup dalam Composition dan Interface. |
| Algorithm | Tidak terdapat algoritma kritikal non-deterministik pada versi sistem ini di luar cakupan standard library. |
| Concurrency | Model concurrency mengikuti standar framework yang digunakan; tidak memerlukan desain khusus. |
| Patterns | Pola arsitektur yang digunakan didokumentasikan secara inline pada setiap Design View yang relevan. |
| Resources | Manajemen sumber daya didelegasikan ke platform Kubernetes; tidak memerlukan desain eksplisit. |