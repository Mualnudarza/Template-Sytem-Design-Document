# 4. Verification

Bagian ini mendeskripsikan bagaimana setiap kebutuhan (*requirements*) yang telah didefinisikan pada Bab 3 akan diverifikasi untuk memberikan bukti objektif atas kepatuhan sistem. Dokumen ini menjadi acuan utama bagi tim *Quality Assurance* (QA) dan *Engineering* dalam mengeksekusi integrasi berkelanjutan (CI/CD).

## 4.1 Lingkungan & Alat Verifikasi
Verifikasi akan dieksekusi menggunakan lingkungan dan tumpukan alat (*tool stack*) berikut:
*   **Lingkungan Pengujian:** `Staging Environment` (Replika dari `Production` dengan penyamarkan/anonimisasi data).
*   **Unit & Integration Testing (Backend):** Jest / Supertest (untuk layanan Node.js) dan PyTest (untuk layanan Python/Analytics).
*   **Load / Performance Testing:** k6 atau Apache JMeter.
*   **End-to-End (E2E) UI Testing:** Cypress.
*   **AI Model Validation:** Scikit-learn metrics (RMSE, Precision@K) untuk mengukur performa *Recommendation Engine*.

## 4.2 Matriks Verifikasi

Berikut adalah matriks keterlacakan pengujian untuk iterasi rilis MVP (v1.0.0). Status dan bukti akan diperbarui secara otomatis atau manual seiring berjalannya siklus rilis.

| Requirement ID | Verification Method | Test/Artifact Link | Status | Evidence |
| :--- | :--- | :--- | :--- | :--- |
| **REQ-INT-001-v1** | Test \| Inspection | `tests/ui/responsive.spec.js` | WIP | *Pending CI pipeline run* |
| **REQ-INT-002-v1** | Demonstration | `docs/testing/manual-hw-demo.md` | Passed | `reports/demo-hw-01.pdf` |
| **REQ-INT-003-v1** | Test | `tests/api/gateway-routing.spec.js` | Passed | `reports/api-gateway.html` |
| **REQ-FUNC-001-v1** | Test | `tests/api/auth-service.spec.js` | Passed | `reports/auth-coverage.xml` |
| **REQ-FUNC-002-v1** | Test | `tests/api/event-creation.spec.js` | Passed | `reports/event-coverage.xml` |
| **REQ-FUNC-003-v1** | Test | `tests/api/registration.spec.js` | WIP | *Pending edge-case fixes* |
| **REQ-FUNC-004-v1** | Test | `tests/api/feedback.spec.js` | Passed | `reports/feedback-coverage.xml` |
| **REQ-FUNC-005-v1** | Test \| Analysis | `tests/jobs/analytics-cron.spec.py` | Passed | `reports/analytics-job.html` |
| **REQ-PERF-001-v1** | Test | `tests/load/k6-read-latency.js` | WIP | *Awaiting Staging deploy* |
| **REQ-SEC-001-v1** | Test \| Inspection | `tests/security/jwt-validation.spec.js` | Passed | `reports/sec-jwt.html` |
| **REQ-REL-001-v1** | Test | `tests/db/transaction-rollback.spec.js` | Passed | `reports/db-acid.html` |
| **REQ-COMP-001-v1** | Test \| Inspection | `docs/compliance/data-exposure-audit.md` | WIP | *Pending review by DPO* |
| **REQ-INST-001-v1** | Demonstration | `deploy/docker-compose.staging.yml` | Passed | `reports/docker-build-logs.txt` |
| **REQ-ML-001-v1** | Test \| Analysis | `tests/ai/hybrid-ahp-cf.spec.py` | WIP | *AHP weights calibration* |
| **REQ-ML-002-v1** | Test \| Demonstration| `tests/ai/cold-start-fallback.spec.py` | Passed | `reports/ai-coldstart.html` |
| **REQ-ML-003-v1** | Inspection | `docs/architecture/cron-retrain-model.md` | Passed | `reports/cron-logs-sample.txt` |

## 4.3 Kriteria Penerimaan Rilis (*Release Sign-Off*)
Sebuah rilis dinyatakan siap (*Ready for Production*) apabila memenuhi kondisi berikut:
1.  **100%** kebutuhan berlabel `REQ-FUNC` dan `REQ-SEC` berstatus **Passed**.
2.  Metrik performa (k6 *Load Test*) pada `REQ-PERF-001-v1` minimal mencapai 95% tingkat kesuksesan respons di bawah target latensi.
3.  Model Rekomendasi (`REQ-ML-001-v1`) memiliki akurasi *Precision@K* yang lebih baik daripada algoritma *Random Recommendation* pada skenario *backtesting* data historis (evaluasi *offline*).