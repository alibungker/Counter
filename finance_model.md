# 4.Financial Model — Queue Counter System
**Nama File:** finance_model.md  
**Versi:** 1.0  
**Tanggal:** 24 Oktober 2025  
**Disusun oleh:** Divisi TI – Bungker Corp

---

## 1) Tujuan
Model finansial ini dipakai untuk:
- Menghitung **biaya per tiket** (Cost per Ticket, CpT) per cabang/per layanan.
- Menilai **efisiensi SDM** (utilisasi loket, produktivitas per jam).
- Menghitung **ROI**, **Payback Period**, dan **TCO** (Total Cost of Ownership).
- Membuat **simulasi sensitivitas** (perubahan trafik, jam kerja, tarif SDM).

---

## 2) Taksonomi Biaya
### 2.1 CAPEX (Belanja Modal - disusutkan)
- Server (VPS/on-prem), TV Display, Mini PC/Thin Client untuk Display, Printer Thermal, Kiosk/Tablet.
- Instalasi/perangkat jaringan (switch, kabel, bracket).
- Lisensi permanen (jika ada).

**Metode Penyusutan:** Garis lurus (Straight-Line).  
\[ \text{Penyusutan Bulanan} = \frac{\text{CAPEX}}{\text{Usia Ekonomis (bulan)}} \]

Contoh usia ekonomis:
- Server & TV: 36 bulan
- Printer: 24 bulan
- Kiosk/Tablet: 24–36 bulan

### 2.2 OPEX (Biaya Operasional)
- Listrik, Internet, Kertas Thermal, Domain/SSL, Support & Maintenance, Sewa VPS.
- Lisensi berlangganan (jika ada, bulanan).

### 2.3 Biaya SDM
- Gaji/jam Teller & CS (jam kerja efektif).
- Tunjangan shift/operasional (opsional).

---

## 3) Definisi KPI
- **Cost per Ticket (CpT)**  
  \[ CpT = \frac{CAPEX_{bln} + OPEX_{bln} + SDM_{bln}}{\text{Jumlah Tiket per Bulan}} \]

- **Utilisasi Loket (U)**  
  \[ U = \frac{\text{Waktu Layanan Total}}{\text{Jam Kerja Efektif} \times \text{Jumlah Loket}} \]

- **Tickets per Staff Hour (TpSH)**  
  \[ TpSH = \frac{\text{Jumlah Tiket}}{\text{Jam Kerja Efektif} \times \text{Jumlah Loket}} \]

- **ROI (12 bulan)**  
  \[ ROI = \frac{\text{Penghematan} - \text{Investasi}}{\text{Investasi}} \]

- **Payback Period (bulan)**  
  \[ \text{Payback} = \frac{\text{Investasi Awal (CAPEX)}}{\text{Penghematan Bulanan}} \]

- **TCO (12/36 bulan)**  
  Total biaya keseluruhan selama horizon waktu (CAPEX tersusut + seluruh OPEX + SDM).

---

## 4) Input Asumsi (Contoh Nilai Default)
> **Catatan:** Ubah nilai sesuai kondisi cabang. Semua nilai contoh di bawah **bisa diedit** ketika diimpor ke spreadsheet.

| Parameter | Nilai Contoh | Satuan | Catatan |
|---|---:|---|---|
| Hari Operasional per Bulan | 26 | hari | |
| Jam Operasional per Hari | 8 | jam | |
| Jumlah Loket Aktif | 4 | loket | Teller+CS |
| Tiket per Hari (total) | 240 | tiket | ±30/tiket/loket/jam jika rata |
| Tarif SDM/jam (rata-rata) | 25,000 | Rp | campuran Teller/CS |
| OPEX Bulanan (non-SDM) | 1,200,000 | Rp | listrik+internet+kertas |
| Sewa VPS Bulanan | 350,000 | Rp | jika on-prem, nol |
| Domain/SSL Bulanan | 50,000 | Rp | bisa 0 (LE gratis) |
| CAPEX Server | 10,000,000 | Rp | jika on-prem |
| CAPEX TV + Bracket | 4,000,000 | Rp | 1 unit |
| CAPEX Mini PC Display | 3,500,000 | Rp | 1 unit |
| CAPEX Printer Thermal | 1,200,000 | Rp | 2 unit |
| CAPEX Kiosk/Tablet | 4,500,000 | Rp | 1 unit |
| Umur Ekonomis Server | 36 | bulan | |
| Umur Ekonomis TV | 36 | bulan | |
| Umur Ekonomis Mini PC | 36 | bulan | |
| Umur Ekonomis Printer | 24 | bulan | |
| Umur Ekonomis Kiosk | 24 | bulan | |

---

## 5) Perhitungan Dasar (Workbook)
### 5.1 Jam Kerja Efektif & Volume Bulanan
\[ \text{Jam/Bulan} = \text{Hari Operasional} \times \text{Jam/Hari} \]
\[ \text{Tiket/Bulan} = \text{Tiket/Hari} \times \text{Hari Operasional} \]

Dengan contoh input:  
- Jam/Bulan = 26 × 8 = **208 jam**  
- Tiket/Bulan = 240 × 26 = **6,240 tiket**

### 5.2 Biaya SDM Bulanan
\[ SDM_{bln} = \text{Tarif/jam} \times \text{Jam/Bulan} \times \text{Jumlah Loket} \]
= 25,000 × 208 × 4 = **Rp20,800,000**

### 5.3 CAPEX → Penyusutan Bulanan
| Item | CAPEX | Umur (bln) | Penyusutan/bln |
|---|---:|---:|---:|
| Server | 10,000,000 | 36 | 277,778 |
| TV + Bracket | 4,000,000 | 36 | 111,111 |
| Mini PC Display | 3,500,000 | 36 | 97,222 |
| Printer Thermal (2x) | 1,200,000 | 24 | 50,000 |
| Kiosk/Tablet | 4,500,000 | 24 | 187,500 |
| **Total** | **23,200,000** |  | **723,611** |

### 5.4 OPEX Bulanan
OPEX non‑SDM = 1,200,000 + 350,000 + 50,000 = **Rp1,600,000**

### 5.5 Cost per Ticket (CpT)
\[ CpT = \frac{CAPEX_{bln} + OPEX_{bln} + SDM_{bln}}{Tiket/Bulan} \]
= (723,611 + 1,600,000 + 20,800,000) / 6,240  
= **Rp3,73 / tiket × 1000 = Rp3.73 ribu** *(dibulatkan)*

> Interpretasi: Dengan asumsi beban kerja dan tarif di atas, biaya per tiket ≈ **Rp3.7k**.

### 5.6 Utilisasi & Produktivitas
- **Tickets per Staff Hour (TpSH)** = 6,240 / (208 × 4) = **7.5 tiket/jam**  
- Jika waktu layanan rata-rata **6 menit/tiket**, kapasitas teoritis per loket = 10 tiket/jam → **Utilisasi ≈ 75%**.

---

## 6) Analisis ROI & Payback
### 6.1 Baseline Manual (Tanpa Sistem)
Misal biaya per tiket manual (estimasi internal): **Rp5.0k**/tiket.  
Selisih efisiensi = 5.0k − 3.7k = **Rp1.3k/tiket** hemat.

**Penghematan Bulanan** = 1.3k × 6,240 ≈ **Rp8,112,000**.

### 6.2 Investasi & Payback
- **Investasi Awal (CAPEX)** = Rp23,200,000.  
- **Payback Period** = 23,200,000 / 8,112,000 ≈ **2.86 bulan**.

### 6.3 ROI 12 Bulan
- **Total Penghematan 12 bln** ≈ 8,112,000 × 12 = **Rp97,344,000**.  
- **TCO 12 bln** = (CAPEX disusut 12 bln: 723,611 × 12) + (OPEX: 1,600,000 × 12) + (SDM: 20,800,000 × 12)  
- TCO 12 bln ≈ 8,683,332 + 19,200,000 + 249,600,000 = **Rp277,483,332**.

Jika dibanding baseline manual, ROI dihitung terhadap **penghematan relatif** terhadap proses manual (gunakan model yang konsisten di organisasi).

---

## 7) Sensitivitas (What‑If) — Tabel Skenario
> Ubah satu variabel untuk melihat dampaknya pada **CpT**.

### 7.1 Sensitivitas Trafik
| Tiket/Hari | Tiket/Bulan | CpT (Rp) |
|---:|---:|---:|
| 160 | 4,160 | 5,59k |
| 200 | 5,200 | 4,46k |
| 240 *(default)* | 6,240 | 3,73k |
| 300 | 7,800 | 3,00k |

### 7.2 Sensitivitas Tarif SDM
| Tarif/jam (Rp) | SDM/bln (Rp) | CpT (Rp) |
|---:|---:|---:|
| 20,000 | 16,640,000 | 3,07k |
| 25,000 *(default)* | 20,800,000 | 3,73k |
| 30,000 | 24,960,000 | 4,39k |

### 7.3 Sensitivitas Jumlah Loket
*(dengan tiket/hari tetap, jam tetap)*  
| Loket | SDM/bln (Rp) | TpSH | CpT (Rp) |
|---:|---:|---:|---:|
| 3 | 15,600,000 | 10.1 | 3,00k |
| 4 *(default)* | 20,800,000 | 7.5 | 3,73k |
| 5 | 26,000,000 | 6.0 | 4,47k |

> Insight: kelebihan loket menurunkan produktivitas per staf & menaikkan CpT bila trafik tetap.

---

## 8) Template Input untuk Aplikasi (Konfigurasi di Admin)
Simpan parameter ini di tabel **settings** / modul laporan:
- `cost.capex.server`, `cost.capex.tv`, `cost.capex.pc_display`, `cost.capex.printer`, `cost.capex.kiosk`
- `life.server`, `life.tv`, `life.pc_display`, `life.printer`, `life.kiosk` (bulan)
- `opex.listrik_internet`, `opex.kertas`, `opex.vps`, `opex.domain_ssl`
- `sdm.tarif_per_jam`, `sdm.jumlah_loket`, `operasional.hari_per_bulan`, `operasional.jam_per_hari`
- `trafik.tiket_per_hari`

> Perhitungan live di dashboard: **CpT**, **U**, **TpSH**, tren biaya 30/90 hari, dan rekomendasi alokasi loket.

---

## 9) Rekomendasi Finansial
1. **Skalakan jumlah loket dinamis** terhadap trafik harian puncak untuk menahan CpT.  
2. **Gabungkan loket** di jam sepi untuk menjaga utilisasi 70–85%.  
3. **Migrasi ke VPS hemat** bila on-prem tidak diperlukan, atau sebaliknya untuk cabang dengan jaringan terbatas.  
4. **Pantau harga kertas thermal** dan gunakan mode cetak hemat.  
5. **Terapkan SLA** untuk meminimalkan no-show (no-show meningkatkan CpT tanpa nilai).

---

## 10) Lampiran — Rumus Ringkas
- Jam/Bulan = Hari × Jam/Hari  
- Tiket/Bulan = Tiket/Hari × Hari  
- SDM/bln = Tarif/jam × Jam/Bulan × Loket  
- Depresiasi/bln per item = CAPEX / Umur  
- CAPEX/bln = Σ Depresiasi/bln item  
- CpT = (CAPEX/bln + OPEX/bln + SDM/bln) / Tiket/Bulan  
- TpSH = Tiket/Bulan / (Jam/Bulan × Loket)  
- Utilisasi ≈ (TpSH / KapasitasTeoritisPerJam) × 100%

---

## 11) Catatan Implementasi
- Kotak “Asumsi Biaya” di **Admin > Laporan** harus dapat diubah oleh peran **Admin/Manajer Keuangan**.  
- Tampilkan **CpT** di laporan harian & bulanan, serta bandingkan terhadap baseline manual.  
- Sediakan **ekspor CSV** raw kalkulasi (asumsi + hasil) untuk audit.

---

**Disusun oleh:**  
_M. Ali Murtaza_  
Divisi TI – Bungker Corp
