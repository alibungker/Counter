# 3.Test Plan — Queue Counter System
**Nama File:** test_plan_counter.md  
**Versi:** 1.0  
**Tanggal:** 24 Oktober 2025  
**Disusun oleh:** Divisi TI – Bungker Corp

---

## 1. Tujuan & Ruang Lingkup
Mendefinisikan strategi pengujian untuk Sistem Antrian Teller/CS (Kiosk, Operator, Display, API, Realtime). Sasaran: memastikan fungsional, non-fungsional, keamanan, dan kinerja memenuhi **SRS** & **PRD**.

Ruang lingkup mencakup:
- Unit, Integration, End‑to‑End (E2E), UAT, Load/Stress, Security, Accessibility, dan DR (Disaster Recovery).

Referensi: `srs_counter.md`, `prd_counter.md`, `sdd_counter.md`, `DEPLOYMENT.md`.

---

## 2. Strategi & Lingkungan Uji
### 2.1 Level Pengujian
- **Unit**: Model, Service (QueueService/ReportService), Controller.
- **Integration**: API + DB + Redis + WebSockets.
- **E2E**: Kiosk → Operator → Display (realtime + TTS).
- **UAT**: Skenario bisnis di cabang (teller, cs, admin).
- **Non-Fungsional**: kinerja, keamanan, aksesibilitas, DR.

### 2.2 Lingkungan
- OS: Ubuntu 22.04 / AlmaLinux 8
- PHP 8.3, MySQL 8/MariaDB 10.5+, Redis 7
- Laravel 11, Laravel WebSockets (6001), Nginx
- Browser: Chrome/Edge terbaru (untuk TTS)
- Data seed: branch=1, services={A:Teller, B:CS}, counters=1..5

### 2.3 Alat
- **PHPUnit** (unit/integration), **Pest** opsional
- **Laravel Dusk** / **Playwright** (E2E UI)
- **Postman/Insomnia** (API tests, collection)
- **k6/JMeter** (load & stress)
- **OWASP ZAP** (security scan)
- **axe-core/Lighthouse** (accessibility/performance)
- **Bash/cron** (backup/DR drill)

---

## 3. Peran & Tanggung Jawab
| Peran | Tanggung Jawab |
|------|------------------|
| QA Lead | Strategi, planning, sign-off |
| QA Engineer | Test design & eksekusi, laporan bug |
| Developer | Unit test, perbaikan bug |
| DevOps | Provisioning env, monitoring, DR drill |
| Product Owner | UAT, penerimaan fitur |

---

## 4. Kriteria Masuk & Keluar
### 4.1 Entry Criteria
- Build berhasil, migration oke, `.env` terset.
- Endpoint health OK (`/api/health`).
- Postman collection tersedia.

### 4.2 Exit Criteria
- 100% **Critical** & **High** bug tertutup.
- >= 85% unit test coverage untuk service & controller kritikal.
- UAT semua **kriteria penerimaan SRS** terpenuhi.
- Load test lulus (lihat §9).

---

## 5. Perancangan Kasus Uji (Ringkas)
### 5.1 Unit Test (contoh)
| ID | Komponen | Kasus | Langkah Verifikasi | Hasil Diharapkan |
|----|----------|-------|--------------------|------------------|
| UT-01 | QueueService | Penomoran harian | INCR kunci tanggal baru | number_int = 1 |
| UT-02 | QueueService | Format kode | num=15, code=Teller(A) | "A-015" |
| UT-03 | Ticket | Status flow | waiting→called→done | Transisi valid |
| UT-04 | ReportService | WaitTime | calc dari timestamps | nilai akurat |
| UT-05 | ReportService | Cost/Ticket | input CAPEX/OPEX/SDM | nilai akurat |

### 5.2 Integration Test (contoh)
| ID | Endpoint | Kasus | Ekspektasi |
|----|----------|-------|------------|
| IT-01 | POST /api/tickets | Buat tiket Teller | 201, payload {id, code, status=waiting} |
| IT-02 | POST /api/counters/{id}/call-next | Ambil waiting tertua | 200, status=called |
| IT-03 | POST /api/calls/{id}/finish | Selesai layanan | 200, status=done |
| IT-04 | GET /api/now/{branch}/services | Ringkasan | 200, agregasi benar |
| IT-05 | WS event | Broadcast ticket.called | Display menerima < 500ms |

### 5.3 E2E (contoh skenario)
| ID | Skenario | Langkah | Hasil |
|----|----------|--------|-------|
| E2E-01 | Flow standar Teller | Kiosk ambil A‑001 → Operator Call Next loket 1 → Display update + TTS | UI update, TTS berbunyi |
| E2E-02 | Recall | Operator Recall A‑001 | Display ulang, TTS berbunyi |
| E2E-03 | Skip/No‑Show | Operator Skip → tiket berikutnya | Status sesuai, audit tercatat |
| E2E-04 | Transfer | A‑002 transfer ke CS | Service berubah, kembali waiting |
| E2E-05 | Multi-loket | Loket 1 & 2 memanggil paralel | Tidak ada race condition |

### 5.4 UAT — Kriteria Penerimaan (mapping SRS §12)
| ID | Kriteria | Skenario Uji | Lulus |
|----|----------|--------------|------|
| UAT-01 | Reset nomor harian | Jam 00:00, nomor ulang ke 001 | ☐ |
| UAT-02 | Realtime < 0.5s | Call Next → Display | ☐ |
| UAT-03 | TTS id‑ID jelas | Panggilan di Chrome | ☐ |
| UAT-04 | Laporan harian | CSV akurat | ☐ |
| UAT-05 | Audit trail | Aksi operator tercatat | ☐ |

---

## 6. Data Uji
- Branch: `{"id":1,"name":"Cabang Utama"}`  
- Services: `A=Teller`, `B=CS`  
- Counters: Loket 1..5 (A), Loket 6..7 (B)  
- Tiket dummy: 50 waiting per layanan  
- Parameter finansial (contoh): CAPEX=Rp25jt/36bln, OPEX=Rp500rb/bln, SDM Rp25rb/jam

---

## 7. Jadwal Pengujian (Contoh)
| Minggu | Aktivitas |
|--------|-----------|
| 1 | Unit & Integration Test |
| 2 | E2E + Regression |
| 3 | UAT + Perbaikan |
| 4 | Load/Security/Accessibility + Go‑Live Readiness |

---

## 8. Manajemen Defect
- Severity: **Critical**, High, Medium, Low.  
- SLA perbaikan: Critical < 24 jam, High < 48 jam.  
- Alur: Repro → Log & bukti → Assign dev → Fix → Verify → Close.  
- Tool: GitHub Issues / Jira, label: `bug`, `priority:high`, `area:display`, dll.

---

## 9. Pengujian Kinerja
### 9.1 Target
- Latensi display setelah **Call Next**: **< 500 ms** (LAN).
- Throughput: minimal **1200 tiket/hari** (puncak 5 req/detik pada /tickets).
- Resource: CPU < 70%, RAM < 75%.

### 9.2 Skenario k6 (pseudocode)
```js
import http from 'k6/http';
import { sleep, check } from 'k6';

export let options = { stages: [
  { duration: '2m', target: 50 },   // ramp-up
  { duration: '5m', target: 50 },   // steady
  { duration: '1m', target: 0 },    // ramp-down
]};

export default function () {
  const res = http.post('https://counter.example.com/api/tickets', JSON.stringify({branch_id:1, service_id:1}), { headers: { 'Content-Type': 'application/json' } });
  check(res, { '201': (r) => r.status === 201 });
  sleep(1);
}
```

### 9.3 WS Latency Check
- Catat timestamp pada **broadcast** & **terima** event di display → hitung delta.
- Lulus jika p95 < 500 ms.

---

## 10. Pengujian Keamanan
- Auth & RBAC: pastikan role akses sesuai (admin/teller/cs/viewer).
- CSRF/XSS/SQLi: gunakan **OWASP ZAP** & fuzzing sederhana.
- Rate-limit: brute-force `/login` & spam `/api/tickets` → blok sesuai konfigurasi.
- Transport security: HTTPS aktif (no mixed content).
- Secret management: `.env` tidak terekspos publik.

---

## 11. Pengujian Aksesibilitas
- Kontras tinggi WCAG 2.1 AA (display & kiosk).
- Ukuran font minimal 18px untuk nomor antrian.
- TTS bahasa `id-ID` aktif; subtitle teks di layar (opsional).

---

## 12. Disaster Recovery (DR) & Backup Test
- **Backup DB harian** tersedia & valid (restore test ke staging).
- Simulasi **mati WebSocket** → display fallback polling 3 detik.
- Simulasi **server DB down** → aplikasi tampilkan pesan maintenance; pemulihan **RTO ≤ 30 menit**.
- Uji **restore** dari dump terbaru → verifikasi integritas data.

---

## 13. Artefak Uji & Pelaporan
- Postman collection: `/tests/postman/queue-counter.postman_collection.json`
- k6 script: `/tests/perf/k6_tickets.js`
- Laporan test: `/reports/test-summary-YYYYMMDD.md` (berisi pass/fail, bug list, rekomendasi)

---

## 14. Risiko & Mitigasi
| Risiko | Dampak | Mitigasi |
|-------|--------|----------|
| Browser blokir TTS | Suara tidak keluar | Trigger interaksi user; fallback TTS server-side |
| Redis down | Penomoran gagal | Systemd restarter + alert |
| WS latency tinggi | Display lambat | Optimasi jaringan LAN, kompres payload |
| Data seed salah | Tes bias | Skrip seeding konsisten & idempotent |

---

## 15. Persetujuan
| Peran | Nama | Tanggal | Tanda Tangan |
|------|------|---------|--------------|
| QA Lead |  |  |  |
| Product Owner |  |  |  |
| Tech Lead |  |  |  |

---

**Disusun oleh:**  
_M. Ali Murtaza_  
Divisi TI – Bungker Corp
