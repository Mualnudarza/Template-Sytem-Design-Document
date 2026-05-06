# 5. Appendixes

Bagian ini memuat materi pendukung operasional dan teknis yang tidak mengikat secara langsung (*non-normative*), namun sangat krusial untuk membantu tim pengembang dalam memahami model data dan konteks implementasi Smart Event Analytics Platform (SEAP).

## 5.1 Kamus Data (Data Dictionary)

Berikut adalah rincian skema tabel pada basis data PostgreSQL yang digunakan oleh *Microservices* SEAP:

### Tabel `users` (User Service & Auth Service)
Menyimpan data identitas dan kredensial pengguna.
| Kolom | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `id` | UUID | *Primary Key* |
| `name` | VARCHAR(100) | Nama lengkap pengguna |
| `email` | VARCHAR(100) | Alamat surel (Unik) |
| `password_hash` | TEXT | Kata sandi yang telah dienkripsi (bcrypt) |
| `role` | VARCHAR(20) | Peran sistem (`admin` atau `mahasiswa`) |
| `created_at` | TIMESTAMP | Waktu pendaftaran akun |

### Tabel `events` (Event Service)
Menyimpan entitas acara yang dikelola oleh Admin.
| Kolom | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `id` | UUID | *Primary Key* |
| `title` | VARCHAR(255) | Judul acara |
| `description` | TEXT | Deskripsi lengkap acara |
| `category` | VARCHAR(50) | Kategori acara (misal: Teknologi, Bisnis, Seni) |
| `date` | TIMESTAMP | Jadwal pelaksanaan acara |
| `location` | VARCHAR(255) | Tempat pelaksanaan atau tautan daring |
| `created_by` | UUID | *Foreign Key* ke tabel `users` (Admin pembuat) |
| `created_at` | TIMESTAMP | Waktu acara dibuat ke sistem |

### Tabel `registrations` (Registration Service)
Menyimpan relasi (pendaftaran) antara mahasiswa dan acara.
| Kolom | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `id` | UUID | *Primary Key* |
| `user_id` | UUID | *Foreign Key* ke tabel `users` |
| `event_id` | UUID | *Foreign Key* ke tabel `events` |
| `status` | VARCHAR(20) | Status partisipasi (`registered`, `attended`, `cancelled`) |
| `registered_at` | TIMESTAMP | Waktu mahasiswa melakukan klik daftar |

### Tabel `feedbacks` (Feedback Service)
Menyimpan ulasan dari mahasiswa pasca-acara (komponen vital untuk *Recommendation Engine*).
| Kolom | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `id` | UUID | *Primary Key* |
| `user_id` | UUID | *Foreign Key* ke tabel `users` |
| `event_id` | UUID | *Foreign Key* ke tabel `events` |
| `rating` | INT | Skala penilaian 1 hingga 5 |
| `comment` | TEXT | Ulasan naratif opsional |
| `created_at` | TIMESTAMP | Waktu ulasan dikirim |

### Tabel `analytics_logs` (Analytics Service)
Menyimpan hasil komputasi *cron job* untuk dasbor Admin.
| Kolom | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `id` | UUID | *Primary Key* |
| `event_id` | UUID | *Foreign Key* ke tabel `events` |
| `total_participants` | INT | Agregat jumlah `registrations` berstatus `attended` |
| `avg_rating` | FLOAT | Rata-rata dari tabel `feedbacks` untuk acara tersebut |
| `calculated_at` | TIMESTAMP | Waktu terakhir log ini dihitung ulang |

---

## 5.2 Standar Respons Kode API (API Status Codes)

Semua layanan di belakang *API Gateway* wajib mematuhi standar respons HTTP berikut untuk menjaga konsistensi *Frontend*:

*   **200 OK:** Permintaan berhasil dieksekusi (terutama *GET* dan *PUT*).
*   **201 Created:** Sumber daya baru berhasil dibuat (terutama *POST* pada registrasi atau pembuatan acara).
*   **400 Bad Request:** *Payload* JSON tidak valid atau struktur tidak sesuai.
*   **401 Unauthorized:** Token JWT tidak ada, kedaluwarsa, atau tidak valid.
*   **403 Forbidden:** Token JWT valid, tetapi *role* tidak memiliki izin (misal: Mahasiswa memanggil fungsi khusus Admin).
*   **404 Not Found:** ID sumber daya tidak ditemukan di basis data.
*   **409 Conflict:** Terjadi duplikasi data (misal: *email* sudah digunakan, mahasiswa mendaftar acara yang sama 2x).
*   **500 Internal Server Error:** Terjadi kegagalan *backend* atau koneksi basis data terputus.

---

## 5.3 Contoh Format Data Rekomendasi (Sample ML Input)

Sebagai gambaran bagi *Data Engineer*, berikut adalah contoh struktur matriks *user-item* virtual (direpresentasikan dalam Pandas DataFrame) yang ditarik secara periodik dari pangkalan data untuk melatih model *Collaborative Filtering*:
```json
[
  {
    "user_id": "u-101",
    "event_id": "e-001",
    "category": "Tech",
    "rating": 5
  },
  {
    "user_id": "u-101",
    "event_id": "e-002",
    "category": "Business",
    "rating": 3
  },
  {
    "user_id": "u-102",
    "event_id": "e-001",
    "category": "Tech",
    "rating": 4
  }
]