# 3. Requirements

## 3.1 External Interfaces

### 3.1.1 User Interfaces
Sistem SEAP diakses melalui antarmuka berbasis web (dibangun menggunakan React.js).
*   **UI-01:** Antarmuka harus bersifat responsif (*mobile-friendly*) untuk mengakomodasi mahasiswa yang mengakses melalui perangkat seluler.
*   **UI-02:** Sistem wajib memiliki halaman Dasbor Admin terpisah untuk visualisasi data analitik dan manajemen entitas (Acara).
*   **UI-03:** Sistem harus mengimplementasikan status kosong (*empty states*) yang informatif ketika mahasiswa belum memiliki riwayat pendaftaran acara atau rekomendasi.

### 3.1.2 Hardware Interfaces
Sistem ini berbasis *cloud* dan tidak memiliki antarmuka perangkat keras kustom langsung di sisi klien. Persyaratan perangkat keras difokuskan pada *server host*:
*   **HW-01:** Lingkungan *deployment* (VPS/Cloud) harus mendukung virtualisasi tingkat OS untuk menjalankan instans Docker Engine.

### 3.1.3 Software Interfaces
*   **SW-01:** Seluruh interaksi dari *frontend* ke *microservices backend* harus melalui Nginx API Gateway.
*   **SW-02:** Komunikasi antar-layanan (*inter-service*) menggunakan protokol HTTP/REST dengan format *payload* JSON.
*   **SW-03:** Layanan analitik dan rekomendasi (Python) terhubung secara internal ke *database* PostgreSQL untuk mengekstraksi data mentah dari tabel `registrations` dan `feedbacks`.

---

## 3.2 Functional

Berikut adalah spesifikasi fungsional utama sistem:

- **ID:** REQ-FUNC-001-v1
- **Title:** Registrasi Akun Mahasiswa
- **Statement:** Sistem harus memungkinkan pengguna (Mahasiswa) untuk mendaftarkan akun baru dengan mengirimkan nama, email, dan kata sandi ke `/auth/register`.
- **Rationale:** Sebagai pintu masuk sistem agar aktivitas pengguna dapat dilacak untuk analisis dan pemberian rekomendasi.
- **Acceptance Criteria:** Sistem mengembalikan status `201 Created` dan menyimpan `password_hash` yang telah dienkripsi ke tabel `users`.
- **Verification Method:** Test
- **More Information:** Terkait dengan batasan keamanan pada REQ-SEC-001.

- **ID:** REQ-FUNC-002-v1
- **Title:** Pembuatan Acara Baru
- **Statement:** Sistem harus memungkinkan aktor dengan peran Admin untuk membuat acara baru melalui layanan Event Service dengan menyertakan judul, deskripsi, kategori, jadwal, dan lokasi.
- **Rationale:** Untuk menyediakan konten dan jadwal acara di dalam platform.
- **Acceptance Criteria:** Data berhasil masuk ke tabel `events` dengan `created_by` sesuai dengan ID Admin yang sedang masuk (*login*).
- **Verification Method:** Test
- **More Information:** -

- **ID:** REQ-FUNC-003-v1
- **Title:** Pendaftaran Acara oleh Mahasiswa
- **Statement:** Sistem harus memproses pendaftaran mahasiswa ke sebuah acara jika acara tersebut belum melewati batas waktu (jadwal `date` di masa depan).
- **Rationale:** Mencegah pendaftaran untuk acara yang sudah berlalu.
- **Acceptance Criteria:** Tabel `registrations` mencatat ID pengguna dan ID acara. Jika jadwal sudah lewat, sistem mengembalikan *error* `400 Bad Request`.
- **Verification Method:** Test
- **More Information:** -

- **ID:** REQ-FUNC-004-v1
- **Title:** Kalkulasi Analitik Acara (Cron Job)
- **Statement:** Sistem harus menghitung total peserta dan rata-rata *rating* setiap acara yang telah selesai secara otomatis atau melalui pemicu waktu (*cron job*).
- **Rationale:** Menyediakan metrik keberhasilan acara bagi Admin.
- **Acceptance Criteria:** Hasil perhitungan disimpan ke dalam tabel `analytics_logs` setiap siklus selesai.
- **Verification Method:** Analysis | Demonstration
- **More Information:** -

---

## 3.3 Quality of Service

### 3.3.1 Performance
- **ID:** REQ-PERF-001-v1
- **Title:** Latensi API Maksimal
- **Statement:** Sistem harus memberikan waktu respons kurang dari 500ms untuk semua permintaan HTTP GET (*read operations*), kecuali untuk *endpoint* Recommendation Service.
- **Rationale:** Menjaga kenyamanan navigasi pengguna di *frontend* (React.js).
- **Acceptance Criteria:** Pengujian beban dengan JMeter menunjukkan *P95 latency* < 500ms di bawah beban 100 *Concurrent Users*.
- **Verification Method:** Test

### 3.3.2 Security
- **ID:** REQ-SEC-001-v1
- **Title:** Autentikasi Berbasis JWT dan RBAC
- **Statement:** Sistem harus mengamankan seluruh *endpoint* manajemen (*create, update, delete*) menggunakan validasi token JWT dan Role-Based Access Control (RBAC).
- **Rationale:** Mencegah modifikasi data oleh aktor yang tidak memiliki izin (seperti Mahasiswa mengubah jadwal acara).
- **Acceptance Criteria:** Permintaan ke `/events` (POST) oleh pengguna dengan *role* 'Mahasiswa' akan ditolak dengan respons `403 Forbidden`.
- **Verification Method:** Test

- **ID:** REQ-SEC-002-v1
- **Title:** Pembatasan Laju API (Rate Limiting)
- **Statement:** Nginx API Gateway harus membatasi maksimal 100 permintaan per menit untuk setiap alamat IP klien.
- **Rationale:** Mencegah serangan *Denial of Service* (DoS) dan penyalahgunaan *endpoint* registrasi.
- **Acceptance Criteria:** Permintaan ke-101 dalam rentang satu menit mengembalikan status `429 Too Many Requests`.
- **Verification Method:** Test

### 3.3.3 Reliability
- **ID:** REQ-REL-001-v1
- **Title:** Degradasi Rapi pada Mesin Rekomendasi
- **Statement:** Jika Recommendation Service gagal memberikan hasil (waktu tunggu habis > 2 detik) atau *user* mengalami *Cold Start*, API Gateway harus mengembalikan daftar "Acara Terpopuler" yang diambil dari Event Service (berdasarkan jumlah pendaftar tertinggi).
- **Rationale:** Memastikan UI tidak kosong meskipun model AI/ML gagal merespons.
- **Acceptance Criteria:** Klien menerima kode status `200 OK` dengan *payload* *fallback* data dari Event Service.
- **Verification Method:** Test

### 3.3.4 Availability
- **ID:** REQ-AVAIL-001-v1
- **Title:** Target Ketersediaan Platform
- **Statement:** Sistem harus menargetkan *uptime* keseluruhan sebesar 99.5% per bulan, di luar jadwal pemeliharaan terencana.
- **Rationale:** Platform sering diakses pada malam hari untuk pendaftaran.
- **Acceptance Criteria:** Laporan dari Prometheus/Grafana mengonfirmasi *uptime* di atas 99.5%.
- **Verification Method:** Analysis

### 3.3.5 Observability
- **ID:** REQ-OBS-001-v1
- **Title:** Sentralisasi Log Layanan
- **Statement:** Sistem harus mengirimkan log terstruktur dari seluruh *microservices* (Node.js & Python) ke ELK Stack terpusat.
- **Rationale:** Memfasilitasi *debugging* cepat jika terjadi insiden lintas-layanan (misal: *timeout* dari API Gateway ke Recommendation Service).
- **Acceptance Criteria:** Log dapat dicari dan difilter di antarmuka Kibana berdasarkan `service_name` dan `level` (*Info/Error*).
- **Verification Method:** Demonstration

---

## 3.4 Compliance
*   **COMP-01:** Sistem harus mematuhi kebijakan privasi institusi, memastikan *password_hash* tidak pernah terekspos di log aplikasi (REQ-OBS-001).

---

## 3.5 Design and Implementation

### 3.5.1 Installation
*   **INST-01:** Sistem wajib dijalankan (*deploy*) menggunakan profil Docker Compose yang mencakup *API Gateway, 7 Microservices, dan PostgreSQL*.

### 3.5.2 Build and Delivery
*   **BUILD-01:** Proses pengemasan citra (*image*) Docker harus diotomatisasi menggunakan *pipeline* CI/CD (GitHub Actions/GitLab CI) yang berjalan secara otomatis pada *branch* `main`.

### 3.5.3 Distribution
*   Sistem MVP ini akan di-*deploy* pada server tunggal (VPS) dengan arsitektur monolitik virtual (*containerized*) sebelum nantinya ditingkatkan skala (*scale-out*) jika pengguna melebihi target metrik kinerja.

### 3.5.10 Change Management
*   **CM-01:** Skema modifikasi struktur tabel (contoh: penambahan kolom pada `events`) harus menggunakan *migration scripts* (contoh: Knex.js atau Alembic) untuk memastikan *rollback* dapat dilakukan dengan aman.

---

## 3.6 AI/ML

Kebutuhan khusus untuk **Recommendation Engine**.

### 3.6.1 Model Specification
- **ID:** REQ-ML-001-v1
- **Title:** Akurasi Model Rekomendasi
- **Statement:** Algoritma *Collaborative Filtering* (berbasis Pandas dan Scikit-learn) harus mengklasifikasikan acara berdasarkan histori partisipasi, preferensi kategori, dan *rating* mahasiswa sebelumnya.
- **Rationale:** Menghasilkan rekomendasi yang personal untuk setiap mahasiswa.
- **Acceptance Criteria:** Model mengembalikan *Top 5* acara yang relevan dengan skor probabilitas tertinggi (atau pembobotan AHP tertinggi jika diaktifkan).
- **Verification Method:** Analysis

### 3.6.2 Data Management
*   **DATA-01:** Pemodelan AI hanya boleh membaca secara langsung dari basis data `registrations` dan `feedbacks` (operasi *Read-Only*). AI tidak boleh memiliki hak untuk mengubah atau menghapus riwayat interaksi (*insert/update/delete*).

### 3.6.3 Guardrails
*   **GUARD-01:** Model harus mengabaikan anomali *rating*, seperti skenario di mana seorang mahasiswa memberikan *rating* pada acara yang menurut sistem `registrations` belum pernah ia ikuti atau acara yang belum diselenggarakan.

### 3.6.6 Model Lifecycle and Operations
*   **LIFECYCLE-01:** Matriks *Collaborative Filtering* dihitung ulang secara *batch* satu kali sehari untuk mengurangi beban komputasi langsung pada jam sibuk kampus. Fitur *Real-time recommendation* akan ditangguhkan ke versi masa depan (v2.0).