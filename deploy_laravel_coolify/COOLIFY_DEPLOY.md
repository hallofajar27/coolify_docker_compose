# Panduan Deploy Coolify (Nixpacks + External PostgreSQL - Produksi)

## Ringkasan Arsitektur
- **Runtime**: PHP 8.4 + Node.js 22 (Nixpacks)
- **Web Server**: Nginx + PHP-FPM (Supervisor multi-process)
- **Database**: PostgreSQL Eksternal (`192.168.0.103:25432`)
- **Cache, Session, Queue**: Driver `database` (Queue worker otomatis via Supervisor)
- **Migrasi**: Dinonaktifkan (database sudah siap)

---

## 1. Konfigurasi Aplikasi di Coolify Dashboard
- **Build Pack**: `Nixpacks`
- **Port**: `80` (atau sesuaikan dengan env `PORT`)
- **Health Check Path**: `/up`

---

## 2. Environment Variables
Wajib pastikan `APP_KEY` diset di dashboard Coolify (jangan digenerate ulang setiap container start):

```dotenv
APP_NAME="SIRVP Semarang BRIN"
APP_ENV=production
APP_KEY=base64:...
APP_DEBUG=false
APP_URL=https://sirvp-staging.hallofajar.my.id

# Database PostgreSQL Eksternal
DB_CONNECTION=pgsql
DB_HOST=***
DB_PORT=***
DB_DATABASE=webgis_sirvp_brin
DB_USERNAME=***
DB_PASSWORD=***

# Driver Standalone (tanpa Redis)
SESSION_DRIVER=database
CACHE_STORE=database
QUEUE_CONNECTION=database
```

---

## 3. Persistent Storage (Volumes)
Tambahkan di tab **Storages** Coolify:
- **Mount Path**: `/app/storage`

---

## 4. Alur Start Container Produksi (`start.sh` via Supervisor)
1. Setup direktori storage framework & permission `www-data` (`775`).
2. `php artisan storage:link --force`.
3. Optimasi cache: `config:cache`, `route:cache`, `view:cache`.
4. Render template Nginx (`/etc/nginx.conf`).
5. Menjalankan Supervisor (`supervisord`):
   - **Worker Nginx**: HTTP server performa tinggi.
   - **Worker PHP-FPM**: Eksekutor PHP dinamis (max 50 workers).
   - **Worker Laravel**: Pemroses antrian `queue:work` latar belakang.
