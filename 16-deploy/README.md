# Modul 16 — Deploy

Modul keenam belas: men-deploy **Task Manager** ke lingkungan **production** — shared hosting (cPanel) dan/atau VPS sederhana. Fokus pada checklist aman, document root `public/`, MySQL production, optimasi Laravel, dan error deploy yang sering muncul.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 15 — Mail & Notification](../15-mail-notification/README.md)

> **Stack tetap sama:** MySQL only, Bootstrap 5 CDN (tidak perlu build Vite di server), tanpa Tailwind / Laravel UI. Deploy lebih sederhana karena asset CSS/JS dari CDN.

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Menyiapkan checklist production (`.env`, debug, key, MySQL)
- [ ] Memahami dua jalur deploy: **shared hosting (cPanel)** dan **VPS**
- [ ] Upload / `git clone` kode ke server
- [ ] Menjalankan `composer install --no-dev`, migrate, `storage:link`
- [ ] Mengatur permission storage & bootstrap/cache
- [ ] Mengarahkan document root ke folder `public/`
- [ ] Mengoptimasi dengan `config:cache`, `route:cache`, `view:cache`
- [ ] Mengenali error deploy umum dan cara mengatasinya
- [ ] Commit dokumentasi: `chore: setup deploy production`

**Yang ditambahkan ke project:**

```
README.md (root project)     ← bagian Deploy / Production checklist
(Opsional) DEPLOY.md         ← catatan langkah server
.env.example                 ← pastikan lengkap tanpa secret
```

Kode fitur app biasanya **tidak** berubah besar di modul ini — yang berubah adalah cara menjalankan di production.

---

## Mengapa Deploy Penting?

Local (`php artisan serve`) hanya untuk development. Production membutuhkan:

| Aspek | Local | Production |
|---|---|---|
| `APP_ENV` | `local` | `production` |
| `APP_DEBUG` | `true` | `false` |
| Database | MySQL local | MySQL hosting/VPS |
| URL | `http://127.0.0.1:8000` | `https://domain.com` |
| Error detail | Tampil di browser | Disembunyikan (log saja) |
| Performance | Belum di-cache | Config/route/view di-cache |

Jika `APP_DEBUG=true` di production, stack trace & secret bisa bocor ke publik.

---

## Konsep: Document Root = `public/`

Struktur Laravel:

```
task-manager/
├── app/
├── bootstrap/
├── config/
├── database/
├── public/          ← HANYA folder ini yang boleh diakses web
│   ├── index.php
│   └── ...
├── resources/
├── routes/
├── storage/
├── .env             ← JANGAN bisa diakses dari browser
└── vendor/
```

Jika document root mengarah ke folder project (bukan `public/`), file `.env` berisiko bisa diunduh. **Ini bahaya.**

Aturan emas:

> Web server (Apache/Nginx) harus serve **`public/index.php`**, bukan root project.

---

## Production Checklist (Wajib)

Sebelum / saat deploy, pastikan `.env` production:

```env
APP_NAME="Task Manager"
APP_ENV=production
APP_KEY=base64:....   # hasil php artisan key:generate
APP_DEBUG=false
APP_URL=https://domain-kamu.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_database_prod
DB_USERNAME=user_database_prod
DB_PASSWORD=password_kuat_prod

SESSION_DRIVER=database
QUEUE_CONNECTION=database
CACHE_STORE=database

MAIL_MAILER=smtp
MAIL_HOST=...
MAIL_PORT=587
MAIL_USERNAME=...
MAIL_PASSWORD=...
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="noreply@domain-kamu.com"
MAIL_FROM_NAME="${APP_NAME}"
```

### Checklist keamanan & konfigurasi

- [ ] `APP_ENV=production`
- [ ] `APP_DEBUG=false`
- [ ] `APP_KEY` terisi (jangan kosong)
- [ ] `APP_URL` pakai `https://` jika SSL aktif
- [ ] Database **MySQL** (bukan SQLite)
- [ ] Password DB kuat & unik
- [ ] `.env` **tidak** di-commit ke Git
- [ ] `.env.example` ada untuk dokumentasi
- [ ] Mail production (bukan Mailtrap) jika ingin email nyata — atau tetap Mailtrap untuk staging

### Generate APP_KEY di server

```bash
php artisan key:generate
```

Jangan copy `APP_KEY` dari laptop ke production jika session/cookie harus terisolasi — boleh sama hanya jika kamu sengaja migrasi encrypted data.

---

## Persiapan di Local Sebelum Deploy

1. Semua fitur Modul 01–15 sudah jalan di local
2. Test suite Modul 14 hijau (ideal)
3. Repo GitHub up to date di `main`
4. Pastikan `.gitignore` berisi:

```gitignore
/.env
/vendor
/node_modules
/public/storage
/storage/*.key
```

5. Update README project dengan instruksi install & deploy (lihat template di Modul 17; di modul ini minimal tambah checklist)

Commit yang diminta modul ini:

```bash
git add .
git commit -m "chore: setup deploy production"
git push
```

Isi commit: dokumentasi deploy + checklist di README project (bukan secret).

---

## Opsi A — Shared Hosting (cPanel)

Cocok jika kamu punya hosting murah dengan cPanel + MySQL + PHP 8.2+.

### A1. Cek persyaratan hosting

| Requirement | Minimal |
|---|---|
| PHP | 8.2+ (sesuaikan versi Laravel kamu) |
| Extensions | OpenSSL, PDO, Mbstring, Tokenizer, XML, Ctype, JSON, BCMath, Fileinfo |
| Database | MySQL / MariaDB |
| Composer | Idealnya ada SSH + Composer; jika tidak, upload `vendor/` (kurang ideal) |
| SSH | Sangat membantu; tanpa SSH lebih ribet |

### A2. Buat database MySQL di cPanel

1. cPanel → **MySQL Databases**
2. Buat database, user, password
3. Assign user ke database (ALL PRIVILEGES)
4. Catat: host biasanya `localhost`, nama DB sering berprefix `username_`

### A3. Upload kode

**Cara 1 — Git (jika SSH tersedia):**

```bash
cd ~/
git clone https://github.com/USERNAME/task-manager.git
cd task-manager
composer install --no-dev --optimize-autoloader
```

**Cara 2 — Upload ZIP via File Manager:**

1. Di local: pastikan `vendor/` tidak wajib di-zip jika nanti `composer install` di server
2. Zip project (kecualikan `.env`, `node_modules`)
3. Upload ke home directory / folder app
4. Extract

### A4. Susun document root ke `public/`

Beberapa pola umum:

**Pola recommended:**

```
/home/user/task-manager/          ← kode Laravel
/home/user/public_html/           ← document root domain
```

Opsi:

1. **Symlink** (jika diizinkan):

```bash
# backup public_html lama dulu!
rm -rf ~/public_html
ln -s ~/task-manager/public ~/public_html
```

2. Atau pindahkan isi `public/` ke `public_html` dan edit `index.php` path ke `../task-manager/...` (lebih error-prone)

3. Atau di cPanel **Domains** → set document root ke `task-manager/public`

Pastikan file `public/.htaccess` ada (Laravel sudah sediakan).

### A5. Buat `.env` di server

Via File Manager atau SSH:

```bash
cp .env.example .env
nano .env
```

Isi nilai production (MySQL cPanel, `APP_DEBUG=false`, dll).

```bash
php artisan key:generate
```

### A6. Composer, migrate, storage

```bash
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan storage:link
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

`--force` wajib di production karena Laravel menolak migrate interaktif saat `APP_ENV=production`.

### A7. Permission

```bash
chmod -R 775 storage bootstrap/cache
# sesuaikan owner ke user web server jika perlu
chown -R $USER:www-data storage bootstrap/cache
```

Di shared hosting, `chown` kadang tidak diizinkan — cukup pastikan folder writable oleh PHP user.

### A8. Uji

- Buka `https://domain-kamu.com`
- Register / login
- Buat task, upload (jika Modul 09), cek email jika SMTP production sudah diisi

---

## Opsi B — VPS Sederhana (Ubuntu)

Cocok jika kamu punya VPS (DigitalOcean, Contabo, AWS Lightsail, dll.) dan akses SSH root/sudo.

> Outline di bawah adalah **langkah praktis ringkas**, bukan hardening enterprise penuh.

### B1. Persiapan server

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx mysql-server redis-tools unzip git
```

Install PHP 8.2 + extensions (contoh Ubuntu):

```bash
sudo apt install -y php8.2-fpm php8.2-mysql php8.2-xml php8.2-mbstring \
  php8.2-curl php8.2-zip php8.2-bcmath php8.2-gd php8.2-cli
```

Install Composer:

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

### B2. Buat database MySQL

```bash
sudo mysql
```

```sql
CREATE DATABASE task_manager CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'tm_user'@'localhost' IDENTIFIED BY 'password_kuat';
GRANT ALL PRIVILEGES ON task_manager.* TO 'tm_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### B3. Clone project

```bash
sudo mkdir -p /var/www
cd /var/www
sudo git clone https://github.com/USERNAME/task-manager.git
cd task-manager
sudo chown -R $USER:www-data .
```

```bash
composer install --no-dev --optimize-autoloader
cp .env.example .env
nano .env
php artisan key:generate
php artisan migrate --force
php artisan storage:link
```

Permission:

```bash
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

### B4. Nginx server block

Contoh `/etc/nginx/sites-available/task-manager`:

```nginx
server {
    listen 80;
    server_name domain-kamu.com www.domain-kamu.com;
    root /var/www/task-manager/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Aktifkan:

```bash
sudo ln -s /etc/nginx/sites-available/task-manager /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### B5. HTTPS dengan Let's Encrypt

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d domain-kamu.com -d www.domain-kamu.com
```

Lalu pastikan `.env`:

```env
APP_URL=https://domain-kamu.com
```

```bash
php artisan config:cache
```

### B6. Optimasi Laravel

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache   # opsional
```

Setelah ubah `.env` atau config, jalankan lagi `config:cache`.

### B7. (Opsional) Queue worker & scheduler

Jika pakai queue database:

```bash
php artisan queue:work --daemon
```

Lebih baik pakai Supervisor. Cron untuk scheduler:

```bash
crontab -e
```

```cron
* * * * * cd /var/www/task-manager && php artisan schedule:run >> /dev/null 2>&1
```

---

## Ringkasan Perintah Deploy (Copy-Paste)

Urutan umum setelah kode ada di server:

```bash
composer install --no-dev --optimize-autoloader
cp .env.example .env          # hanya jika belum ada
# edit .env → production values
php artisan key:generate      # jika APP_KEY kosong
php artisan migrate --force
php artisan storage:link
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

Update kode berikutnya (VPS/git):

```bash
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## Mengapa Bootstrap CDN Mempermudah Deploy?

Karena UI memakai CDN:

- **Tidak perlu** `npm install` / `npm run build` di server
- Tidak perlu Node.js di production
- Deploy = PHP + Composer + MySQL saja

Ini selaras aturan panduan: tanpa Vite/Tailwind di stack pembelajaran.

---

## Keamanan Deploy

### Jangan pernah commit `.env`

```bash
git status
# Pastikan .env tidak muncul sebagai file yang akan di-commit
```

Jika `.env` pernah ter-push:

1. Rotate semua secret (DB password, mail password, APP_KEY)
2. Hapus dari history (advanced) atau minimal hapus dari commit berikutnya + force-ignore
3. Anggap secret sudah bocor

### HTTPS

- Pakai Let's Encrypt / SSL cPanel
- Redirect HTTP → HTTPS
- Set `APP_URL` ke `https://...`
- Untuk cookie aman (opsional di `config/session.php`): `secure => true` di production

### APP_DEBUG=false

Dengan debug false:

- User lihat halaman error generik
- Detail error di `storage/logs/laravel.log`
- Jangan share log mentah jika berisi data sensitif

### Permission

- Folder `storage/` dan `bootstrap/cache/` writable
- File lain tidak perlu 777 sembarangan
- Hindari `chmod -R 777` permanen

### File upload (Modul 09)

- Validasi mime/size tetap aktif
- `storage:link` wajib agar file publik terlayani
- Jangan simpan file upload di luar disk yang dikontrol

---

## Error Deploy yang Sering Muncul

### 500 Internal Server Error

| Cek | Perintah / tindakan |
|---|---|
| Log Laravel | `storage/logs/laravel.log` |
| Permission | `chmod -R 775 storage bootstrap/cache` |
| APP_KEY kosong | `php artisan key:generate` |
| Config cache usang | `php artisan config:clear` lalu `config:cache` |
| Document root salah | Harus ke `public/` |

### 404 di semua route kecuali `/`

- `try_files` Nginx belum benar
- `.htaccess` Apache hilang / `AllowOverride` off
- Document root bukan `public`

### Error database connection

- Host/user/password salah
- User MySQL belum di-grant
- Di shared hosting, host kadang bukan `127.0.0.1` (cek panel)

### `The stream or file could not be opened ... storage/logs`

Permission storage. Perbaiki owner/chmod.

### CSS Bootstrap tidak muncul

- Koneksi internet user (CDN)
- Atau Content-Security-Policy memblokir CDN — longgarkan / allow Bootstrap CDN

### `Please provide a valid cache path`

Jalankan:

```bash
php artisan view:clear
mkdir -p storage/framework/{cache,sessions,views}
chmod -R 775 storage
```

### Migrate ditolak di production

Pakai:

```bash
php artisan migrate --force
```

### `storage:link` gagal / gambar 404

```bash
php artisan storage:link
ls -la public/storage
```

Pastikan symlink mengarah ke `storage/app/public`.

### Route cache error setelah ubah route closure

`route:cache` tidak mendukung closure route. Ubah ke controller class, atau jangan cache route sampai semua route pakai controller.

---

## Dokumentasi di README Project

Tambahkan section singkat di `README.md` root Task Manager (contoh):

```markdown
## Deploy (Production)

### Checklist
- APP_ENV=production, APP_DEBUG=false
- MySQL production + APP_KEY
- Document root → public/
- composer install --no-dev --optimize-autoloader
- php artisan migrate --force
- php artisan storage:link
- php artisan config:cache && php artisan route:cache && php artisan view:cache

### Catatan
- Jangan commit file .env
- Aktifkan HTTPS
- Bootstrap via CDN — tidak perlu npm di server
```

Detail lengkap bisa merujuk ke modul ini di repo panduan, atau file `DEPLOY.md` di project.

---

## Staging vs Production

Praktik bagus (opsional tapi direkomendasikan):

| Lingkungan | Tujuan | Debug | Mail |
|---|---|---|---|
| Local | Development | `true` | Mailtrap / log |
| Staging | Uji di server mirip prod | `false` atau terbatas | Mailtrap |
| Production | User nyata | `false` | SMTP sungguhan |

Jangan uji eksperimen langsung di production tanpa backup DB.

---

## Backup Sebelum Deploy Ulang

Sebelum `migrate` atau `git pull` besar:

```bash
# contoh dump MySQL
mysqldump -u tm_user -p task_manager > backup_$(date +%F).sql
```

Simpan juga salinan `.env` di tempat aman (bukan di Git).

---

## Latihan Modul 16

### Latihan 1: Checklist Tertulis

Buat section **Deploy** di README project yang memuat:

- Requirement server
- Langkah install
- Perintah artisan production
- Peringatan keamanan `.env` & HTTPS

### Latihan 2: Deploy ke Satu Lingkungan

Pilih **salah satu**:

- Shared hosting cPanel, **atau**
- VPS Nginx

Dokumentasikan URL live (boleh subdomain).

### Latihan 3: Simulasi Error

Sengaja (di staging):

1. Set `APP_DEBUG=false` lalu buat error — pastikan detail tidak bocor
2. Salahkan document root sebentar — amati bahayanya, kembalikan ke `public/`
3. Hapus symlink storage — amati 404 file upload, lalu perbaiki

### Latihan 4: Smoke Test Production

Setelah deploy, uji:

| Fitur | OK? |
|---|---|
| Register / login | |
| CRUD task | |
| Dashboard & filter | |
| Upload (jika ada) | |
| API (jika ada) dengan token | |
| Halaman 404 custom | |
| HTTPS padlock | |

### Latihan 5: Redeploy dari Git

Ubah teks kecil di README → commit → `git push` → di server `git pull` + cache ulang. Pastikan proses update terukur.

---

## Checklist Modul 16

### Persiapan

- [ ] Kode di `main` sudah lengkap Modul 01–15
- [ ] `.gitignore` mengabaikan `.env` & `vendor`
- [ ] README project berisi langkah deploy
- [ ] Database MySQL production siap

### Server

- [ ] Kode ter-upload / di-clone
- [ ] `composer install --no-dev --optimize-autoloader`
- [ ] `.env` production benar (`APP_DEBUG=false`)
- [ ] `migrate --force` sukses
- [ ] `storage:link` sukses
- [ ] Permission `storage` & `bootstrap/cache` OK
- [ ] Document root = `public/`
- [ ] `config:cache` / `route:cache` / `view:cache`

### Keamanan

- [ ] `.env` tidak di Git
- [ ] HTTPS aktif (atau rencana SSL jelas)
- [ ] Debug mati
- [ ] Password DB kuat

### Git

- [ ] Commit: `chore: setup deploy production`
- [ ] Push ke `main`

---

## Git — Commit & Push

```bash
git add .
git commit -m "chore: setup deploy production"
git push
```

Commit ini untuk **dokumentasi/checklist deploy** di project — bukan untuk menambahkan secret.

Alur Git tetap sederhana: hanya `main`, tanpa PR/merge/branch workflow.

---

## Apa yang Sudah Kamu Kuasai

- Checklist production Laravel
- Deploy shared hosting & outline VPS
- Document root `public/`
- Composer production + migrate force
- Optimasi cache Laravel
- Keamanan dasar: `.env`, debug, HTTPS
- Troubleshooting error deploy umum

---

## Modul Selanjutnya

Modul 17 (Capstone) merangkum seluruh perjalanan, checklist fitur 01–16, polish UI, demo script, dan README final untuk portfolio.

---

**Modul sebelumnya:** [15 — Mail & Notification](../15-mail-notification/README.md)  
**Modul berikutnya:** [17 — Capstone Review](../17-capstone/README.md)  
**Kembali ke:** [Panduan Utama](../README.md)
