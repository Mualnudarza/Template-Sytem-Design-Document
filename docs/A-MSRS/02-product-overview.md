# 2. Product Overview

## 2.1 Product Perspective
**Smart Event Analytics Platform (SEAP)** adalah sebuah produk baru yang dirancang untuk mendigitalisasi dan memusatkan ekosistem manajemen acara (seperti *workshop*, seminar, dan kuliah tamu) di lingkungan kampus. Sebelumnya, pengelolaan acara seringkali terfragmentasi atau dilakukan secara manual. 

SEAP dibangun menggunakan pendekatan *Microservices Architecture* mandiri yang diakses melalui antarmuka web. Sistem ini berinteraksi dengan pengguna (Admin dan Mahasiswa) melalui sebuah *API Gateway* (Nginx) yang meneruskan permintaan ke tujuh layanan internal (Auth, User, Event, Registration, Feedback, Analytics, dan Recommendation). Meskipun pada rilis MVP sistem ini beroperasi dengan basis data pengguna bawaannya sendiri (PostgreSQL), arsitektur SEAP telah disiapkan agar kedepannya (*downstream*) dapat berintegrasi dengan sistem *Single Sign-On* (SSO) internal kampus. 

## 2.2 Product Functions
Sistem SEAP memfasilitasi fungsionalitas utama berikut, yang dikelompokkan berdasarkan kapabilitas domain operasionalnya:

*   **Manajemen Autentikasi & Pengguna:** Registrasi dan login pengguna dengan otorisasi berbasis peran (RBAC) menggunakan *JSON Web Token* (JWT).
*   **Manajemen Siklus Hidup Acara:** Memungkinkan Admin untuk membuat, memperbarui, dan mempublikasikan katalog acara kampus.
*   **Registrasi & Pelacakan Partisipasi:** Memungkinkan mahasiswa untuk mencari dan mendaftar acara, serta mencatat status keikutsertaan mereka.
*   **Pengumpulan Umpan Balik (Feedback):** Fasilitas bagi mahasiswa untuk memberikan *rating* dan komentar setelah acara selesai.
*   **Dasbor Analitik Acara:** Memproses dan menyajikan statistik acara secara agregat (seperti total peserta dan rata-rata *rating*) kepada Admin.
*   **Sistem Rekomendasi Cerdas:** Menyajikan daftar acara yang dipersonalisasi untuk setiap mahasiswa berdasarkan riwayat partisipasi, preferensi kategori, dan *rating* menggunakan algoritma *Collaborative Filtering* (dengan dukungan AHP opsional).

## 2.3 Product Constraints
Perancangan dan implementasi SEAP dibatasi oleh kondisi dan mandat teknis berikut:
*   **Arsitektur & Infrastruktur:** Sistem wajib dibangun menggunakan pendekatan *microservices* dan seluruh layanannya harus dikemas (di- *deploy*) menggunakan kontainer **Docker**.
*   **Teknologi Basis Data:** Sistem wajib menggunakan **PostgreSQL** sebagai basis data relasional utama untuk persistensi semua *service*.
*   **Tumpukan Teknologi Kecerdasan Buatan:** Layanan analitik dan rekomendasi (AI/ML) wajib menggunakan ekosistem **Python** (khususnya *library* Pandas dan Scikit-learn).
*   **Keamanan & Jaringan:** Semua komunikasi eksternal dari klien menuju *API Gateway* wajib dienkripsi menggunakan protokol **HTTPS**. Sistem wajib menerapkan pembatasan laju permintaan (*Rate Limiting*) untuk mencegah eksploitasi.

## 2.4 User Characteristics
Sistem ini dirancang untuk dua kelompok pengguna utama:

1.  **Admin (Panitia Acara / Staf Kampus):** 
    *   *Karakteristik:* Memiliki tingkat literasi komputer menengah. 
    *   *Tujuan:* Mengelola acara secara efisien, melacak jumlah pendaftar, dan melihat laporan analitik (keberhasilan acara) untuk pelaporan institusi. 
    *   *Kebutuhan:* Dasbor antarmuka yang informatif, navigasi yang jelas, dan kemudahan dalam mengekspor data laporan.
2.  **Mahasiswa (Pengguna Akhir):** 
    *   *Karakteristik:* *Digital native*, sangat terbiasa dengan aplikasi web modern dan media sosial. Mengakses platform dengan frekuensi tinggi saat awal semester atau saat ada pekan acara kampus.
    *   *Tujuan:* Mencari acara yang relevan dengan minat mereka, mendaftar dengan cepat, dan mendapatkan sertifikat/poin partisipasi.
    *   *Kebutuhan:* Antarmuka yang responsif (terutama di peramban seluler/ *mobile web*), pengalaman pendaftaran dengan *minim-klik*, dan rekomendasi yang akurat.

## 2.5 Assumptions and Dependencies
*   **Asumsi:**
    *   Pengguna mengakses platform menggunakan peramban web modern (seperti Chrome, Safari, Edge) dengan koneksi internet yang relatif stabil.
    *   Infrastruktur *server* (VPS/Cloud) yang disediakan oleh kampus memiliki spesifikasi komputasi yang memadai untuk menjalankan setidaknya 8 *container service* secara simultan beserta metrik *monitoring* (Prometheus + Grafana).
*   **Dependensi:**
    *   Kelancaran *routing* API sangat bergantung pada konfigurasi **Nginx** sebagai *API Gateway*.
    *   Fitur analitik bergantung pada ketersediaan dan integritas data historis di tabel `registrations` dan `feedbacks`. Jika data ini kosong/korup, kualitas *Recommendation Engine* akan menurun drastis (*Cold Start Problem*).

## 2.6 Apportioning of Requirements
Berikut adalah alokasi pemetaan fitur utama terhadap fase rilis produk:

| Fitur / Modul | Target Rilis | Keterangan |
| :--- | :--- | :--- |
| **Auth & Event Management** | v1.0.0 (MVP) | Kebutuhan dasar berjalannya sistem. |
| **Registration & Feedback Service** | v1.0.0 (MVP) | Kebutuhan interaksi pengguna. |
| **Basic Dashboard Analytics** | v1.0.0 (MVP) | Agregasi data peserta dan rata-rata *rating*. |
| **Collaborative Filtering Recommendation** | v1.0.0 (MVP) | Mesin rekomendasi tahap pertama. |
| **Integrasi SSO Kampus** | Ditangguhkan (v2.0) | Membutuhkan koordinasi dengan Biro IT Kampus. |
| **AI-Based Sentiment Analysis** | Ditangguhkan (v2.0) | Memerlukan NLP tingkat lanjut untuk analisis teks *feedback*. |
| **Aplikasi Mobile Native (iOS/Android)** | Ditangguhkan (v3.0) | MVP difokuskan pada antarmuka *Web Responsive*. |