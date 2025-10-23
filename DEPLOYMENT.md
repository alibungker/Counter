# 2.Deployment Guide (2.DEPLOYMENT.md)
**Nama Sistem:** Sistem Antrian Teller/CS (Queue Counter System)  
**Versi:** 1.0  
**Tanggal:** 24 Oktober 2025  
**Disusun oleh:** Divisi TI – Bungker Corp  

---

## 1. Tujuan
Dokumen ini menjelaskan langkah-langkah instalasi, konfigurasi, dan pengoperasian sistem **Queue Counter System** berbasis Laravel 11.  
Tujuannya agar tim DevOps dan administrator dapat:
- Melakukan instalasi dan konfigurasi dari nol.
- Menjalankan aplikasi di lingkungan production.
- Menjaga kestabilan dan keamanan sistem antrian.

---

## 2. Persyaratan Sistem

### 2.1 Spesifikasi Server Minimum
| Komponen | Spesifikasi |
|-----------|-------------|
| OS | Ubuntu 22.04 LTS / AlmaLinux 8 |
| CPU | 2 Core |
| RAM | 4 GB |
| Storage | 40 GB SSD |
| Network | 100 Mbps LAN |
| Web Server | Nginx / Apache |
| Database | MySQL 8 / MariaDB 10.5+ |
| Cache | Redis 7 |
| PHP | 8.3 |
| Node.js | 20+ |
| Composer | 2.7+ |

### 2.2 Paket Pendukung
```bash
sudo apt install -y nginx php8.3-fpm php8.3-mysql php8.3-redis php8.3-mbstring php8.3-xml php8.3-curl php8.3-zip redis-server mysql-server
```

---

## 3. Persiapan Lingkungan

### 3.1 Clone Repository
```bash
cd /var/www/
git clone https://github.com/bungkercorp/queue-counter.git
cd queue-counter
```

### 3.2 Instalasi Dependensi
```bash
composer install
npm install && npm run build
```

### 3.3 Atur File `.env`
Salin dari contoh:
```bash
cp .env.example .env
php artisan key:generate
```

Edit file `.env` dan sesuaikan konfigurasi berikut:

```dotenv
APP_NAME="Queue Counter"
APP_ENV=production
APP_URL=https://counter.example.com
APP_DEBUG=false

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=queue_counter
DB_USERNAME=queue_user
DB_PASSWORD=strongpassword

BROADCAST_DRIVER=pusher
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=file

PUSHER_APP_ID=local
PUSHER_APP_KEY=local
PUSHER_APP_SECRET=local
PUSHER_HOST=127.0.0.1
PUSHER_PORT=6001
PUSHER_SCHEME=http
```

---

## 4. Setup Database

### 4.1 Buat Database
```bash
mysql -u root -p
CREATE DATABASE queue_counter CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'queue_user'@'localhost' IDENTIFIED BY 'strongpassword';
GRANT ALL PRIVILEGES ON queue_counter.* TO 'queue_user'@'localhost';
FLUSH PRIVILEGES;
```

### 4.2 Jalankan Migrasi dan Seeder
```bash
php artisan migrate --seed
```
Seeder awal akan membuat:
- Admin user (`admin@example.com`)
- Default branch dan layanan (Teller & CS)

---

## 5. Jalankan Layanan Realtime (WebSocket & Queue)

### 5.1 Jalankan Queue Worker
```bash
php artisan queue:work
```

### 5.2 Jalankan WebSocket Server
```bash
php artisan websockets:serve
```

### 5.3 Supervisor Setup
Buat file `/etc/supervisor/conf.d/queue-counter.conf`:

```
[program:queue-counter-worker]
command=php /var/www/queue-counter/artisan queue:work --sleep=3 --tries=3
autostart=true
autorestart=true
stderr_logfile=/var/log/queue-counter-worker.err.log
stdout_logfile=/var/log/queue-counter-worker.out.log

[program:queue-counter-websockets]
command=php /var/www/queue-counter/artisan websockets:serve
autostart=true
autorestart=true
stderr_logfile=/var/log/queue-counter-websockets.err.log
stdout_logfile=/var/log/queue-counter-websockets.out.log
```

Aktifkan:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start all
```

---

## 6. Konfigurasi Nginx

### 6.1 Lokasi File
`/etc/nginx/sites-available/queue-counter.conf`

### 6.2 Contoh Konfigurasi
```nginx
server {
    listen 80;
    server_name counter.example.com;
    root /var/www/queue-counter/public;

    index index.php index.html;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }

    access_log /var/log/nginx/queue-counter.access.log;
    error_log /var/log/nginx/queue-counter.error.log;
}
```

Aktifkan:
```bash
sudo ln -s /etc/nginx/sites-available/queue-counter.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 7. SSL Configuration (Opsional)
Gunakan **Let’s Encrypt**:
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d counter.example.com
```

Atur auto-renew:
```bash
sudo systemctl enable certbot.timer
```

---

## 8. Pengujian Sistem

### 8.1 Akses Web
- `https://counter.example.com/kiosk` → Ambil tiket.  
- `https://counter.example.com/operator` → Halaman teller/CS.  
- `https://counter.example.com/display` → Layar TV.

### 8.2 Verifikasi Realtime
1. Buka halaman **Display** di browser.  
2. Dari halaman **Operator**, klik **Call Next**.  
3. Display harus berubah **kurang dari 1 detik** dan suara TTS aktif.

---

## 9. Backup & Maintenance

### 9.1 Backup Database Harian
Tambahkan di `crontab -e`:
```
0 2 * * * mysqldump -uqueue_user -pstrongpassword queue_counter > /var/backups/queue_counter_$(date +\%F).sql
```

### 9.2 Log Rotation
Tambahkan ke `/etc/logrotate.d/queue-counter`:
```
/var/log/nginx/queue-counter*.log {
    daily
    missingok
    rotate 14
    compress
    notifempty
}
```

---

## 10. Monitoring & Health Check

### 10.1 Health Endpoint
`GET /api/health`  
Contoh respons:
```json
{
  "status": "OK",
  "db": "connected",
  "redis": "connected",
  "queue": "running"
}
```

### 10.2 Uptime Monitoring
Gunakan salah satu:
- Uptime Kuma
- Grafana Loki + Promtail

---

## 11. Troubleshooting Umum

| Masalah | Penyebab | Solusi |
|----------|-----------|--------|
| Display tidak realtime | WebSocket tidak berjalan | Jalankan `php artisan websockets:serve` |
| Tidak bisa ambil tiket | Redis tidak aktif | `sudo systemctl restart redis-server` |
| Suara tidak keluar | Browser tidak support TTS | Gunakan Chrome terbaru atau aktifkan server-side TTS |
| Tidak bisa login | Cache session rusak | Jalankan `php artisan cache:clear && php artisan config:clear` |

---

## 12. Checklist Sebelum Go-Live

| No | Item | Status |
|----|-------|--------|
| 1 | HTTPS aktif dan valid | ✅ |
| 2 | Supervisor untuk queue & websocket aktif | ✅ |
| 3 | Backup otomatis berjalan | ✅ |
| 4 | Health check berfungsi | ✅ |
| 5 | Laporan harian berjalan normal | ✅ |

---

## 13. Revisi & Versi
| Versi | Tanggal | Perubahan | Penulis |
|--------|----------|-----------|----------|
| 1.0 | 24/10/2025 | Draft awal panduan deployment | M. Ali Murtaza |

---

**Disusun oleh:**  
_M. Ali Murtaza_  
Divisi TI – Bungker Corp  
