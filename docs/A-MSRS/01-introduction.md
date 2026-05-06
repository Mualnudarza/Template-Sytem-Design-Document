# 1. Introduction

## 1.1 Document Purpose
Dokumen Markdown Software Requirements Specification (MSRS) ini mendefinisikan secara komprehensif seluruh kebutuhan fungsional dan non-fungsional untuk **Smart Event Analytics Platform (SEAP)**. Dokumen ini secara tegas berfokus pada *apa* yang harus dicapai oleh sistem, bukan *bagaimana* sistem tersebut dirancang atau diimplementasikan secara teknis. 

Audiens utama untuk dokumen ini meliputi tim *Product* untuk memastikan keselarasan fitur, tim *Engineering* (pengembang *Frontend* dan *Backend*) sebagai acuan target pengembangan, serta tim *Quality Assurance* (QA) sebagai basis penyusunan skenario pengujian dan verifikasi sistem.

## 1.2 Product Scope
**Smart Event Analytics Platform (SEAP) versi 1.0.0-MVP** adalah platform berbasis layanan web (*microservices*) yang bertujuan untuk memusatkan dan mendigitalisasi manajemen acara kampus, menganalisis data partisipasi mahasiswa, dan menyajikan rekomendasi acara yang dipersonalisasi. Sasaran utama produk ini adalah meningkatkan interaksi mahasiswa dengan kegiatan akademik maupun non-akademik melalui pengambilan keputusan berbasis data historis.

*   **Inclusions (Cakupan Masuk):** Autentikasi berbasis peran (Admin/Mahasiswa), manajemen siklus hidup acara (pembuatan, pendaftaran, *feedback*), dasbor analitik dasar, dan *Recommendation Engine* berbasis *Collaborative Filtering*.
*   **Exclusions (Cakupan Keluar):** Pada rilis MVP ini, sistem belum mencakup aplikasi *mobile native*, integrasi dengan sistem *Single Sign-On* (SSO) internal kampus secara langsung, serta analisis sentimen *feedback* berbasis AI tingkat lanjut.

## 1.3 Definitions, Acronyms, and Abbreviations

| Term / Acronym | Definition |
| :--- | :--- |
| **AHP** | *Analytic Hierarchy Process* - Metode pengambilan keputusan terstruktur yang digunakan sebagai algoritma opsional pembobotan pada mesin rekomendasi SEAP. |
| **API** | *Application Programming Interface* - Kumpulan definisi dan protokol untuk mengintegrasikan layanan antar *microservices*. |
| **JWT** | *JSON Web Token* - Standar terbuka (RFC 7519) yang mendefinisikan cara ringkas dan mandiri untuk mentransmisikan informasi otentikasi secara aman antar pihak. |
| **RBAC** | *Role-Based Access Control* - Pendekatan keamanan sistem yang membatasi akses ke *endpoint* API berdasarkan peran pengguna (misal: Admin vs. Mahasiswa). |
| **SEAP** | *Smart Event Analytics Platform* - Nama dari produk perangkat lunak yang dispesifikasikan dalam dokumen ini. |

## 1.4 References
*   **Informative:** IEEE Std 830-1998, *IEEE Recommended Practice for Software Requirements Specifications*.
*   **Normative:** SEAP UI/UX Prototype & Mockups (Tautan Figma internal tim).
*   **Normative:** MADR: `0002-select-postgresql-as-primary-db.md` (Keputusan arsitektur persistensi data yang memengaruhi batasan desain sistem).

## 1.5 Document Overview
MSRS ini disusun untuk memfasilitasi navigasi yang cepat dan terstruktur. Bagian **Product Overview** (Bab 2) memberikan pemahaman tentang konteks sistem, batasan operasional, dan karakteristik pengguna (Admin & Mahasiswa). Bagian **Requirements** (Bab 3) adalah inti dokumen yang menjabarkan secara detail spesifikasi antarmuka, fungsionalitas layanan (*Auth, Event, Analytics*, dll.), serta standar keamanan. Bagian **Verification** (Bab 4) menautkan setiap Kebutuhan (*Requirement*) dengan metode pengujiannya. Dokumen ini dikelola melalui sistem *version control* bersamaan dengan basis kode sistem.