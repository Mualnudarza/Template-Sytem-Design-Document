# BAB 5: APPENDIXES

Bagian ini berisi materi pendukung opsional yang bersifat non-normatif. Lampiran berfungsi untuk memperjelas konteks desain, menyediakan referensi teknis detail, atau mendokumentasikan artefak pendukung tanpa memberatkan alur baca dokumen utama.

---

## 5.1 Kamus Data (Data Dictionary)

Bagian ini menyediakan definisi teknis lengkap mengenai elemen data yang digunakan dalam sistem, melengkapi model data yang telah digambarkan pada Information View (Bab 3.5).

---

**Perintah (Instructions)**

Rincikan seluruh entitas dan atribut data utama yang digunakan dalam sistem, termasuk nama field, tipe data, constraint, nilai default, dan deskripsi fungsionalnya. Jika terdapat aturan validasi atau transformasi data yang spesifik, dokumentasikan di sini. Informasi ini sangat krusial bagi Database Administrator (DBA), Backend Engineer, dan QA Engineer untuk menjaga integritas data di seluruh lapisan aplikasi.

**Catatan:** Kamus data ini harus selalu sinkron dengan skema basis data aktual di repositori. Jika terdapat perbedaan antara kamus data dan skema aktual, kamus data di sini bersifat as-designed dan skema di repositori bersifat as-built.

### Contoh (Example)

**Entitas: `ORDERS`**

| Nama Field | Tipe Data | Constraint | Default | Deskripsi |
| --- | --- | --- | --- | --- |
| `order_id` | UUID | PK, Not Null | `gen_random_uuid()` | Identitas unik global untuk setiap order. |
| `user_id` | UUID | FK → USERS.user_id, Not Null | — | Referensi ke pengguna yang membuat order. |
| `status` | ENUM | Not Null | `'PENDING'` | Status siklus hidup order. Nilai: `PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED, REFUNDED`. |
| `total_amount` | DECIMAL(18,2) | Not Null, Min: 0.01 | — | Total nilai moneter order dalam mata uang `{{mata_uang}}`. |
| `created_at` | TIMESTAMP WITH TIME ZONE | Not Null | `now()` | Waktu pembuatan order dalam UTC. |
| `updated_at` | TIMESTAMP WITH TIME ZONE | Not Null | `now()` | Waktu pembaruan terakhir record dalam UTC. |

---

## 5.2 Matriks Keterlacakan Desain ke Persyaratan

Bagian ini menyediakan pemetaan lintas-referensi antara Design View yang telah didefinisikan pada Bab 3 dengan ID persyaratan di MSRS, untuk memastikan seluruh persyaratan telah terpenuhi oleh setidaknya satu elemen desain.

---

**Perintah (Instructions)**

Buat tabel pemetaan yang menghubungkan setiap ID persyaratan dari MSRS dengan Design View yang mengimplementasikannya. Tabel ini digunakan oleh System Analyst dan QA Engineer untuk melakukan gap analysis — memastikan tidak ada persyaratan yang belum memiliki representasi desain. Cantumkan pula referensi ke keputusan arsitektur (ADR) yang relevan untuk persyaratan dengan implikasi desain yang signifikan.

**Catatan:** Persyaratan yang belum dipetakan ke Design View apapun harus ditandai dengan status `UNTRACED` dan menjadi item tindak lanjut yang wajib diselesaikan sebelum dokumen difinalisasi.

### Contoh (Example)

| REQ ID (MSRS) | Judul Persyaratan | Design View(s) | ADR Reference | Status |
| --- | --- | --- | --- | --- |
| `REQ-FUNC-001` | Login & Autentikasi | `DV-001-user-authentication-flow` | `ADR-002` | Traced |
| `REQ-FUNC-011` | Pembuatan Order | `DV-NNN-order-creation-sequence`, `DV-NNN-core-data-model` | `ADR-004` | Traced |
| `REQ-SEC-001` | Manajemen Token JWT | `DV-001-user-authentication-flow` | `ADR-001` | Traced |
| `REQ-AVAIL-001` | SLA Ketersediaan 99.9% | `DV-NNN-production-deployment-topology` | `ADR-009` | Traced |
| `REQ-PERF-001` | Latensi API < 200ms | `DV-NNN-production-deployment-topology` | `ADR-009` | Traced |
| `REQ-FUNC-NNN` | {{judul}} | — | — | UNTRACED |

---

## 5.3 Sampel Payload dan Kontrak API Tambahan

Bagian ini menyediakan contoh data nyata dalam format JSON yang merepresentasikan input dan output sistem untuk skenario tambahan di luar yang telah didokumentasikan pada Interface View.

---

**Perintah (Instructions)**

Sajikan contoh payload untuk endpoint atau skenario yang paling sering digunakan dalam pengujian integrasi dan pengembangan klien. Sertakan minimal satu contoh untuk kondisi sukses (happy path) dan satu contoh untuk kondisi kegagalan umum (error path) per endpoint kritikal. Batasi dokumentasi pada skenario yang tidak tercakup dalam Interface View di Bab 3.6.

**Catatan:** Seluruh contoh payload harus menggunakan data fiktif (synthetic data). Dilarang keras menggunakan data produksi nyata dalam dokumentasi teknis.

### Contoh (Example)

**Endpoint: `GET /api/v1/orders/{orderId}`**

**Response (200 OK):**

```json
{
  "status": "success",
  "data": {
    "order_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "status": "CONFIRMED",
    "total_amount": 250000.00,
    "items": [
      {
        "product_code": "PRD-001",
        "name": "Contoh Produk A",
        "quantity": 2,
        "unit_price": 125000.00
      }
    ],
    "created_at": "2024-05-20T10:00:00Z"
  },
  "metadata": {
    "request_id": "req-9900xyz",
    "timestamp": "2024-05-20T10:05:00Z"
  }
}
```

**Response (404 Not Found):**

```json
{
  "status": "error",
  "error": {
    "code": "ORDER_NOT_FOUND",
    "message": "Order with the specified ID does not exist or is not accessible.",
    "request_id": "req-9901abc"
  }
}
```

---

## 5.4 Glosarium Teknis dan Referensi Pendukung

Bagian ini mendaftarkan istilah-istilah teknis spesifik, referensi pustaka pihak ketiga, standar industri, atau dokumen penelitian yang menjadi inspirasi arsitektur namun tidak tercantum dalam glosarium utama (Bab 1.3).

---

**Perintah (Instructions)**

Daftarkan istilah teknis sangat spesifik yang digunakan hanya dalam konteks arsitektur ini, referensi ke paper penelitian, standar teknis, atau wiki internal yang menginformasikan keputusan desain. Bagian ini berfungsi sebagai pusat pengetahuan bagi anggota tim baru dalam proses onboarding. Pastikan setiap tautan referensi masih aktif dan dapat diakses oleh seluruh anggota tim.

**Catatan:** Entri pada glosarium ini bersifat informatif dan tidak normatif. Untuk istilah yang memiliki implikasi teknis kritis terhadap implementasi, tambahkan catatan mengenai cara penggunaan yang tepat dalam konteks sistem ini.

### Contoh (Example)

| Istilah / Referensi | Tautan | Penjelasan dalam Konteks Sistem Ini |
| --- | --- | --- |
| Idempotency Key | [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) | Diimplementasikan pada endpoint `POST /orders` untuk mencegah duplikasi order akibat network retry. Header: `X-Idempotency-Key`. |
| PKCE (Proof Key for Code Exchange) | [RFC 7636](https://datatracker.ietf.org/doc/html/rfc7636) | Ekstensi OAuth2 yang diimplementasikan pada Auth Service untuk mengamankan alur autentikasi pada klien publik (SPA/Mobile). |
| Blue-Green Deployment | [Martin Fowler — BlueGreenDeployment](https://martinfowler.com/bliki/BlueGreenDeployment.html) | Strategi rilis yang digunakan oleh pipeline CI/CD sistem ini untuk memastikan zero-downtime deployment. |
| Circuit Breaker Pattern | [Release It! — Michael Nygard](https://pragprog.com/titles/mnee2/) | Pola yang diimplementasikan pada koneksi ke Payment Gateway eksternal untuk mencegah kegagalan berantai (cascading failure). |
| The Traceability Triad | Dokumen Internal MSDD v1.0 | Prinsip keterlacakan tiga arah: MSRS (What) → MSDD (How) → MADR (Why) yang menjadi landasan sistem dokumentasi ini. |