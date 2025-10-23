# User Guide — Queue Counter System
**Nama File:** user_guide.md  
**Versi:** 1.0  
**Tanggal:** 24 Oktober 2025  
**Disusun oleh:** Divisi TI – Bungker Corp  

---

## 1. Tujuan
Panduan ini menjelaskan cara menggunakan Sistem Antrian Teller/CS untuk **Pelanggan**, **Operator (Teller/CS)**, dan **Admin Cabang**.  
Sistem ini berbasis web dan dapat dijalankan di browser tanpa instalasi tambahan.

---

## 2. Peran & Akses
| Peran | Modul yang Diakses | Deskripsi |
|-------|--------------------|------------|
| **Pelanggan** | Kiosk | Mengambil tiket antrian |
| **Teller / CS** | Operator | Memanggil dan melayani pelanggan |
| **Admin Cabang** | Admin Panel | Mengelola user, layanan, loket, laporan |
| **Manajer** | Laporan | Melihat statistik & biaya per tiket |

---

## 3. Login dan Navigasi
### 3.1 Login
1. Buka URL aplikasi (misal): `https://counter.example.com/login`
2. Masukkan username dan password yang diberikan oleh admin.
3. Klik **Masuk**.  
   Jika berhasil, Anda akan diarahkan ke dashboard sesuai peran.

> 💡 **Catatan:** Setiap role memiliki tampilan berbeda — Teller/CS hanya melihat loketnya, sedangkan Admin bisa mengelola seluruh cabang.

### 3.2 Navigasi Menu Utama
| Menu | Fungsi |
|------|---------|
| **Kiosk** | Antarmuka pelanggan untuk mengambil tiket |
| **Operator** | Antarmuka Teller/CS untuk memanggil tiket |
| **Display** | Layar monitor antrian |
| **Admin** | Pengaturan cabang, layanan, loket, dan user |
| **Laporan** | Statistik harian, waktu tunggu, biaya per tiket |

---

## 4. Modul Kiosk (Ambil Tiket)

### 4.1 Akses
- URL: `https://counter.example.com/kiosk`
- Gunakan pada perangkat Kiosk, tablet, atau PC dengan printer thermal (opsional).

### 4.2 Langkah
1. Pilih jenis layanan: **Teller** atau **Customer Service**.  
2. Sistem menampilkan nomor antrian (misal **A-015**).  
3. (Opsional) Tiket tercetak otomatis melalui printer thermal.
4. Tunggu panggilan nomor di layar Display.

### 4.3 Tampilan
```
+----------------------------------------+
| PILIH LAYANAN                         |
| [ Teller ]     [ Customer Service ]   |
|----------------------------------------|
| Nomor Anda: A-015                     |
| Harap menunggu panggilan di layar TV. |
+----------------------------------------+
```

> 📌 Tiket akan otomatis reset setiap hari pukul 00:00.

---

## 5. Modul Operator (Teller / CS)

### 5.1 Akses
- URL: `https://counter.example.com/operator`
- Login menggunakan akun Teller/CS.

### 5.2 Tampilan
```
+-------------------------------------------+
| LOKET 2 - Teller                          |
|-------------------------------------------|
| Tiket Sekarang: A-015                     |
|-------------------------------------------|
| [ Call Next ] [ Recall ] [ Finish ]       |
| [ Skip ] [ Transfer ] [ No-Show ]         |
|-------------------------------------------|
| Riwayat Hari Ini:                         |
| A-012 Done | A-013 Done | A-014 Done     |
+-------------------------------------------+
```

### 5.3 Fungsi Tombol
| Tombol | Fungsi |
|--------|---------|
| **Call Next** | Memanggil tiket berikutnya di antrian |
| **Recall** | Memanggil ulang tiket terakhir |
| **Finish** | Menandai tiket selesai dilayani |
| **Skip** | Melewati tiket, lanjut ke berikutnya |
| **Transfer** | Memindahkan tiket ke layanan lain |
| **No-Show** | Menandai pelanggan tidak hadir |

### 5.4 Catatan Operasional
- Semua aktivitas direkam di sistem audit log.  
- Jika Display tidak berubah, pastikan WebSocket aktif (`php artisan websockets:serve`).  
- Hanya **1 tiket aktif per loket** pada satu waktu.

---

## 6. Modul Display (Layar TV)

### 6.1 Akses
- URL: `https://counter.example.com/display`
- Bisa dibuka di TV dengan browser (mode fullscreen).

### 6.2 Fungsi
- Menampilkan **nomor antrian yang sedang dipanggil**.  
- Memutar suara otomatis menggunakan **Text-to-Speech (TTS)**:  
  > “Nomor antrian A-015 menuju Loket 2.”

### 6.3 Tampilan
```
+-----------------------------------------------+
| NOMOR DIPANGGIL       | LOKET                |
|-----------------------------------------------|
| A-015                 | 2                    |
|-----------------------------------------------|
| A-014 | Loket 1 | 10:12                      |
| A-013 | Loket 2 | 10:10                      |
|-----------------------------------------------|
| CABANG UTAMA — SISTEM ANTRIAN BUNGKER CORP   |
+-----------------------------------------------+
```

> 💡 Jika suara tidak keluar, aktifkan izin **microphone/speaker** pada browser.

---

## 7. Modul Admin (Pengaturan)

### 7.1 Akses
- URL: `https://counter.example.com/admin`
- Role: **Admin Cabang**

### 7.2 Fitur
| Menu | Deskripsi |
|------|------------|
| **Cabang** | Tambah/edit cabang (kode, alamat) |
| **Layanan** | Tambah layanan Teller/CS baru |
| **Loket** | Tentukan jumlah dan nama loket |
| **User** | Tambah Teller, CS, Admin baru |
| **Suara & Bahasa** | Atur template TTS & kecepatan |
| **Reset Harian** | Atur jam reset otomatis (default 00:00) |
| **Parameter Biaya** | Input CAPEX, OPEX, SDM untuk laporan finansial |

---

## 8. Modul Laporan & Analitik

### 8.1 Fitur Laporan
- Jumlah tiket per hari/per loket.
- Waktu tunggu rata-rata.
- Durasi layanan rata-rata.
- No-show rate (% tiket tidak hadir).
- Biaya per tiket (Cost per Ticket, CpT).
- Utilisasi loket (%).

### 8.2 Ekspor Laporan
Klik **Ekspor CSV / Excel** → file akan berisi:
| Kolom | Keterangan |
|-------|-------------|
| Branch | Nama cabang |
| Service | Teller / CS |
| Total Tickets | Jumlah tiket |
| Avg Wait Time | Waktu tunggu rata-rata |
| Avg Service Time | Waktu layanan |
| Cost per Ticket | Hasil kalkulasi finansial |

---

## 9. Troubleshooting Umum
| Masalah | Penyebab | Solusi |
|----------|-----------|--------|
| Nomor tidak muncul di display | WebSocket mati | Jalankan `php artisan websockets:serve` |
| Suara panggilan tidak terdengar | Browser blokir TTS | Aktifkan suara & reload halaman |
| Tiket tidak tercetak | Printer offline / koneksi USB | Restart printer & cek driver |
| Tidak bisa login | Cache rusak | Jalankan `php artisan config:clear` |
| Nomor tidak reset harian | Cron job belum aktif | Pastikan `queue:work` dan `schedule:run` berjalan |

---

## 10. Tips Operasional
- Gunakan TV layar besar (43"+) agar nomor mudah terbaca.  
- Gunakan **font kontras tinggi (putih di latar biru/hitam)**.  
- Pastikan koneksi LAN stabil antara Kiosk, Operator, dan Display.  
- Jalankan **Supervisor** untuk menjaga service tetap aktif:
  ```bash
  sudo supervisorctl status
  ```

---

## 11. Panduan Admin untuk Backup & Restore
### 11.1 Backup Harian (otomatis)
File SQL tersimpan di `/var/backups/queue_counter_YYYY-MM-DD.sql`.

### 11.2 Restore Manual
```bash
mysql -uqueue_user -p queue_counter < /var/backups/queue_counter_2025-10-24.sql
```

### 11.3 Verifikasi
Setelah restore, cek data cabang & layanan di halaman Admin.

---

## 12. Panduan Keamanan
- Gunakan HTTPS untuk semua akses.  
- Ganti password admin secara berkala.  
- Batasi akses Admin hanya untuk IP internal (opsional).  
- Nonaktifkan debug di `.env`: `APP_DEBUG=false`.  
- Pastikan backup disimpan di lokasi aman (off-site / NAS).

---

## 13. Kontak & Dukungan Teknis
**Tim TI Bungker Corp**  
Jl. Pocut Baren, Laksana, Banda Aceh 24415  
📧 support@bungker.co.id  
🌐 [www.bungker.co.id](https://www.bungker.co.id)

---

**Disusun oleh:**  
_M. Ali Murtaza_  
Divisi TI – Bungker Corp
