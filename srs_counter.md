# Sistem Antrian Teller/CS (Queue Counter System)
**Dokumen**: Software Requirements Specification  
**Versi**: 1.0  
**Nama File**: srs_counter.md  
**Tanggal**: 24 Oktober 2025  
**Disusun oleh**: Divisi TI – Bungker Corp

---

## 1. Pendahuluan

### 1.1 Tujuan
Dokumen ini menjelaskan kebutuhan sistem antrian berbasis web yang digunakan untuk mengatur, memanggil, dan menampilkan nomor antrian pelanggan pada layanan **Teller** dan **Customer Service (CS)**.  
Sistem ini akan membantu:
- Mengatur distribusi pelanggan.
- Mengurangi waktu tunggu.
- Meningkatkan efisiensi pelayanan.
- Memberikan data statistik dan biaya operasional per tiket.

### 1.2 Ruang Lingkup
Sistem terdiri dari 3 antarmuka utama:
1. **Kiosk / Tiket Counter** – pelanggan mengambil tiket antrian.
2. **Operator (Teller / CS)** – petugas memanggil dan melayani pelanggan.
3. **Display Monitor (TV)** – menampilkan nomor antrian dan memutar suara panggilan.

Backend dikembangkan dengan **Laravel 11**, **MySQL/MariaDB**, **Redis**, dan **Laravel WebSockets** untuk komunikasi realtime.

### 1.3 Definisi & Singkatan
| Istilah | Definisi |
|----------|-----------|
| Teller | Petugas yang melayani transaksi keuangan. |
| CS | Customer Service. |
| Ticket | Nomor antrian yang diterbitkan dari kiosk. |
| Counter | Loket tempat pelayanan dilakukan. |
| TTS | Text-to-Speech, suara panggilan otomatis. |
| SLA | Service Level Agreement (target waktu layanan). |
| RBAC | Role Based Access Control. |

---

## 2. Deskripsi Sistem

### 2.1 Tujuan Bisnis
- Mengurangi waktu tunggu pelanggan dan meningkatkan efisiensi loket.
- Menyediakan laporan waktu pelayanan, biaya per tiket, dan utilisasi petugas.
- Menstandarkan sistem antrian di seluruh cabang.

### 2.2 Stakeholder
| Peran | Tujuan |
|--------|--------|
| Admin Cabang | Mengelola data layanan, loket, dan petugas. |
| Teller / CS | Memanggil dan melayani tiket antrian. |
| Pelanggan | Mengambil tiket antrian dan menunggu panggilan. |
| Manajemen | Memantau laporan efisiensi dan biaya. |

### 2.3 Lingkungan Operasional
- Browser modern (Chrome/Edge) untuk kiosk, operator, dan display.
- Server dengan PHP 8.3, MySQL 8+, Redis, dan WebSockets aktif.
- Koneksi LAN / Wi-Fi internal stabil.

---

## 3. Kebutuhan Fungsional

### 3.1 Modul Kiosk / Tiket
- F-01 Pengguna memilih jenis layanan (Teller / CS).
- F-02 Sistem mengeluarkan nomor antrian (mis. A-015).
- F-03 Nomor disimpan ke database dan dikirim via WebSocket.
- F-04 (Opsional) Cetak tiket di printer thermal.
- F-05 Reset nomor otomatis setiap hari.

### 3.2 Modul Operator Teller / CS
- F-06 Petugas login dengan role masing-masing.
- F-07 Klik “Call Next” untuk memanggil tiket berikutnya.
- F-08 Dapat melakukan Recall, Skip, Finish, Transfer, No-Show.
- F-09 Dapat melihat riwayat tiket yang telah dilayani.
- F-10 Semua aksi dicatat dalam log audit.

### 3.3 Modul Display / TV
- F-11 Menampilkan nomor tiket yang sedang dipanggil.
- F-12 Memutar suara otomatis: “Nomor antrian A-015 menuju Loket 2.”
- F-13 Menampilkan daftar 5 panggilan terakhir.
- F-14 Desain UI responsif dan kontras tinggi.

### 3.4 Modul Laporan & Monitoring
- F-15 Menampilkan total tiket, waktu tunggu rata-rata, no-show rate.
- F-16 Menghitung **biaya per tiket** berdasarkan konfigurasi biaya tetap/variabel.
- F-17 Ekspor laporan ke CSV/Excel.
- F-18 Grafik utilisasi per loket dan per layanan.

### 3.5 Modul Administrasi
- F-19 CRUD Cabang, Layanan, Loket, User.
- F-20 Pengaturan suara (template kalimat, bahasa, kecepatan).
- F-21 Pengaturan reset harian & parameter biaya.
- F-22 Role-based access (Admin, Teller, CS, Viewer).

---

## 4. Kebutuhan Non-Fungsional

| Kode | Deskripsi |
|------|------------|
| NF-01 | Update realtime (latency < 500ms dalam LAN). |
| NF-02 | Kapasitas: 10 loket/cabang, 1200 tiket/hari. |
| NF-03 | Tersedia fallback polling jika WebSocket gagal. |
| NF-04 | UI harus dapat diakses oleh pengguna difabel (font besar, kontras tinggi). |
| NF-05 | Sistem mencatat audit untuk setiap aksi operator. |
| NF-06 | Backup database harian otomatis. |
| NF-07 | Web dan API dilindungi HTTPS dan autentikasi JWT/session. |

---

## 5. Model Data

### 5.1 Entitas & Atribut
| Entitas | Atribut Utama |
|----------|---------------|
| Branch | id, name |
| Service | id, branch_id, code, name, is_priority |
| Counter | id, branch_id, name, service_id |
| Ticket | id, branch_id, service_id, number_int, code, status, called_at, served_by_counter_id, served_by_user_id, created_at |
| Call | id, ticket_id, counter_id, action, created_at |
| User | id, name, role, counter_id |

### 5.2 Status Tiket
`waiting → called → serving → done`  
atau: `waiting → called → noshow / skipped → done`

---

## 6. Alur Proses

### 6.1 Alur Utama (BPMN)
```mermaid
flowchart LR
  A[Kiosk: Ambil Tiket] --> B[DB: Simpan Ticket waiting]
  B --> C[Operator: Call Next]
  C --> D[DB: Update Status = called]
  D --> E[WebSocket: Broadcast ticket.called]
  E --> F[Display: Update UI + TTS]
  F --> G[Operator: Finish/Skip/Transfer]
  G --> H[DB: Update Status Final]
```

### 6.2 Sequence Diagram
```mermaid
sequenceDiagram
  participant User as Pelanggan
  participant Kiosk
  participant API
  participant Operator
  participant Display

  User->>Kiosk: Pilih Layanan
  Kiosk->>API: POST /tickets
  API->>Kiosk: Kembalikan nomor (A-015)
  Operator->>API: POST /counters/{id}/call-next
  API->>Display: Broadcast ticket.called
  Display->>Display: Render dan ucapkan TTS
```

---

## 7. API Endpoint Ringkas

| Endpoint | Method | Deskripsi |
|-----------|--------|-----------|
| `/api/tickets` | POST | Buat tiket baru |
| `/api/counters/{id}/call-next` | POST | Ambil tiket berikutnya |
| `/api/calls/{id}/recall` | POST | Panggil ulang |
| `/api/calls/{id}/finish` | POST | Selesaikan tiket |
| `/api/calls/{id}/noshow` | POST | Tandai tidak hadir |
| `/api/calls/{id}/skip` | POST | Lewati tiket |
| `/api/reports` | GET | Ambil laporan harian |

---

## 8. Kebutuhan Keamanan

- Autentikasi berbasis JWT / Session Laravel.  
- HTTPS wajib di semua endpoint.  
- Role-based authorization:  
  - Admin: semua fitur  
  - Teller/CS: akses loket masing-masing  
  - Viewer: hanya laporan  
- Audit log semua aksi penting (call, skip, transfer, finish).

---

## 9. Aspek Finansial & Analitik

### 9.1 Perhitungan Biaya per Tiket
BiayaPerTiket = (BiayaTetap + BiayaVariabel + BiayaSDM) / JumlahTiketBulanan

| Komponen | Contoh |
|-----------|--------|
| Biaya Tetap | Server, Printer, TV |
| Biaya Variabel | Kertas, Listrik, Bandwidth |
| Biaya SDM | Gaji per jam × jam operasional |
| Jumlah Tiket | Total tiket dalam periode laporan |

### 9.2 KPI Operasional
- Average Wait Time (≤ 7 menit)
- Average Service Time
- Tickets per Staff Hour
- Utilisasi Loket (%)
- No-show Rate (%)
- Cost per Ticket (Rp)

---

## 10. Kriteria Penerimaan

| No | Kriteria | Status |
|----|-----------|--------|
| 1 | Tiket dapat diambil dengan format A-XXX | ✅ |
| 2 | Operator memanggil tiket dan tampil di display < 0.5s | ✅ |
| 3 | Suara panggilan terdengar jelas (id-ID) | ✅ |
| 4 | Laporan menampilkan total tiket dan waktu tunggu | ✅ |
| 5 | Ekspor CSV berjalan normal | ✅ |

---

## 11. Rencana Implementasi

### Tahap 1 – MVP
- Modul Kiosk, Operator, Display, WebSocket, DB schema, API dasar.  
- TTS menggunakan Web Speech API.

### Tahap 2 – Laporan & Finansial
- Laporan harian, biaya per tiket, SLA chart.

### Tahap 3 – Fitur Lanjutan
- TTS server-side (Polly/gTTS).  
- Multi-branch dashboard.  
- Digital signage/iklan saat idle.

---

## 12. Risiko & Mitigasi

| Risiko | Dampak | Mitigasi |
|---------|---------|----------|
| Koneksi LAN putus | Display tidak realtime | Aktifkan polling setiap 3 detik |
| Browser tidak mendukung TTS | Tidak ada suara panggilan | Gunakan TTS server-side |
| Server overload | Delay panggilan | Gunakan Redis untuk cache nomor |
| Operator salah input | Data tidak valid | Audit trail & undo terbatas |

---

## 13. Lampiran

### 13.1 Data Dictionary
| Field | Deskripsi |
|--------|------------|
| `tickets.status` | waiting, called, serving, done, noshow, skipped |
| `services.code` | A=Teller, B=CS |
| `calls.action` | call, recall, transfer, finish, noshow, skip |

### 13.2 Template Kalimat TTS
> "Nomor antrian **{CODE}**, menuju **Loket {N}**."

### 13.3 Konfigurasi Default
- Reset harian: 00:00
- Bahasa: id-ID
- Suara: Web Speech API (Chrome)
- Jumlah loket: 5
- Jumlah layanan: 2 (Teller, CS)

---

## 14. Penutup
Dokumen ini menjadi dasar pengembangan dan pengujian sistem antrian teller/CS di lingkungan Bungker Corp dan instansi pengguna.  
Semua pengembangan dan perubahan fitur wajib mengikuti spesifikasi dalam dokumen ini.

---
**Disusun oleh:**  
_M. Ali Murtaza_  
Divisi TI – Bungker Corp
