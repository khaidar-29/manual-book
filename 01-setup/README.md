# Modul 01 — Setup & Instalasi

Modul pertama: menyiapkan environment, membuat project **Task Manager**, dan menjalankannya dengan **MySQL**.

**Estimasi waktu:** 1 hari  
**Prasyarat:** [Modul 00 — Project Overview](../00-project-overview/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Mengecek dan menginstall software yang dibutuhkan
- [ ] Membuat database MySQL `task_manager`
- [ ] Membuat project Laravel `task-manager`
- [ ] Mengkonfigurasi file `.env` untuk MySQL
- [ ] Menjalankan migration bawaan Laravel
- [ ] Membuka aplikasi di browser
- [ ] Git init + commit pertama

---

## Apa yang Akan Dibuat?

Di modul ini belum ada fitur aplikasi. Targetnya:

1. Laravel terinstall
2. MySQL terhubung
3. Halaman welcome Laravel tampil di `http://localhost:8000`
4. Git siap dipakai

---

## Langkah 1: Cek Software

Buka terminal dan jalankan:

```bash
php -v
composer -V
mysql --version
git --version
```

### Versi minimum

| Software | Versi |
|---|---|
| PHP | 8.2 atau lebih baru |
| Composer | 2.x |
| MySQL | 8.x |
| Git | versi terbaru |

### Extension PHP wajib

Laravel + MySQL butuh extension `pdo_mysql`:

```bash
php -m | grep pdo_mysql
```

Jika kosong, install extension sesuai OS kamu.

---

## Langkah 2: Install Software

### macOS (Homebrew)

```bash
brew install php composer mysql git
brew services start mysql
```

### Windows (Laragon — disarankan)

1. Download [Laragon](https://laragon.org)
2. Install dan jalankan Laragon
3. Klik **Start All** — PHP, MySQL, Composer sudah siap

### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install php8.2 php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip php8.2-bcmath
sudo apt install composer mysql-server git
sudo systemctl start mysql
```

---

## Langkah 3: Buat Database MySQL

Login ke MySQL:

```bash
mysql -u root -p
```

Jalankan perintah SQL:

```sql
CREATE DATABASE task_manager CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
SHOW DATABASES;
EXIT;
```

**Penjelasan:**
- `task_manager` — nama database yang dipakai project
- `utf8mb4` — charset yang mendukung emoji dan karakter unicode
- Pastikan database `task_manager` muncul di `SHOW DATABASES`

---

## Langkah 4: Buat Project Laravel

Pilih folder kerja kamu, lalu buat project:

```bash
cd ~/projects

composer create-project laravel/laravel task-manager

cd task-manager
```

Tunggu sampai proses download selesai (bisa beberapa menit).

**Apa yang terjadi?**
- Composer mengunduh Laravel dan semua dependency PHP
- Folder `task-manager/` berisi project lengkap
- File `.env` otomatis dibuat dari `.env.example`

---

## Langkah 5: Konfigurasi Environment

### Generate application key

```bash
cp .env.example .env
php artisan key:generate
```

Key ini dipakai Laravel untuk enkripsi session, cookie, dll.

### Edit file `.env`

Buka `.env` dengan editor, ubah bagian berikut:

```env
APP_NAME="Task Manager"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager
DB_USERNAME=root
DB_PASSWORD=password_mysql_kamu
```

**Penting:**
- `DB_DATABASE` harus sama dengan database yang sudah dibuat
- `DB_PASSWORD` isi password MySQL kamu (kosongkan jika root tanpa password)
- **Jangan commit file `.env`** ke GitHub

---

## Langkah 6: Jalankan Migration

Migration bawaan Laravel membuat tabel `users`, `cache`, dan `jobs`:

```bash
php artisan migrate
```

Output yang diharapkan:

```
INFO  Running migrations.
  0001_01_01_000000_create_users_table .......... DONE
  0001_01_01_000001_create_cache_table .......... DONE
  0001_01_01_000002_create_jobs_table ........... DONE
```

Jika error `Access denied`, cek ulang `DB_USERNAME` dan `DB_PASSWORD` di `.env`.

---

## Langkah 7: Jalankan Development Server

```bash
php artisan serve
```

Buka browser: **http://localhost:8000**

Halaman **welcome Laravel** harus tampil. Jika iya, setup berhasil.

> **Catatan styling:** Manual ini pakai **Bootstrap 5 via CDN**. Kamu **tidak perlu** menjalankan `npm install` atau `npm run dev`.

---

## Langkah 8: Kenalan Struktur Folder

```
task-manager/
├── app/
│   ├── Http/Controllers/     ← Controller (logika request)
│   └── Models/               ← Model (representasi tabel DB)
├── bootstrap/                ← Bootstrap framework
├── config/                   ← Konfigurasi aplikasi
├── database/
│   ├── migrations/           ← File struktur tabel
│   └── seeders/              ← Data dummy
├── public/                   ← Entry point web (index.php)
├── resources/
│   └── views/                ← File Blade (HTML)
├── routes/
│   └── web.php               ← Definisi URL
├── storage/                  ← Log, cache, upload
├── .env                      ← Konfigurasi environment
└── artisan                   ← Command-line tool
```

### Folder yang paling sering kamu sentuh

| Folder/File | Fungsi |
|---|---|
| `routes/web.php` | Mendefinisikan URL aplikasi |
| `app/Http/Controllers/` | Logika bisnis |
| `resources/views/` | Tampilan HTML |
| `database/migrations/` | Struktur tabel database |
| `app/Models/` | Interaksi dengan database |
| `.env` | Konfigurasi (database, app name, dll.) |

---

## Langkah 9: Eksplorasi Artisan

Coba perintah berikut:

```bash
# Lihat semua route
php artisan route:list

# REPL interaktif — test kode PHP
php artisan tinker
>>> app()->version()
>>> exit

# Lihat bantuan artisan
php artisan help
```

---

## Langkah 10: Git Init + Commit Pertama

```bash
git init
git add .
git commit -m "chore: initial laravel setup"
```

Cek status:

```bash
git status
git log --oneline
```

Harus muncul 1 commit dan working tree clean.

---

## Latihan Modul 01

1. Jalankan `php artisan route:list` — catat berapa route bawaan Laravel
2. Buka `php artisan tinker`, ketik `DB::connection()->getDatabaseName()` — pastikan return `task_manager`
3. Ubah `APP_NAME` di `.env` menjadi `"Task Manager v1"`, refresh browser
4. Coba `php artisan serve --port=8080` jika port 8000 sibuk
5. Buka folder `database/migrations/` — baca isi migration `create_users_table`

---

## Troubleshooting

| Error | Penyebab | Solusi |
|---|---|---|
| `could not find driver` | Extension `pdo_mysql` belum aktif | Install `php-mysql` / `php8.2-mysql` |
| `SQLSTATE[HY000] [1045] Access denied` | Username/password salah | Cek `.env` |
| `Unknown database 'task_manager'` | Database belum dibuat | Buat via MySQL |
| `No application encryption key` | Belum generate key | `php artisan key:generate` |
| Port 8000 already in use | Port dipakai app lain | `php artisan serve --port=8080` |
| `composer: command not found` | Composer belum terinstall | Install Composer |

---

## Checklist Selesai

- [ ] PHP 8.2+, Composer, MySQL, Git terinstall
- [ ] Database `task_manager` dibuat di MySQL
- [ ] Project `task-manager` dibuat via Composer
- [ ] `.env` dikonfigurasi untuk MySQL
- [ ] `php artisan migrate` berhasil
- [ ] `php artisan serve` jalan, welcome page tampil
- [ ] Paham struktur folder utama
- [ ] `git init` + commit pertama selesai

---

## Git — Push (Opsional di Modul Ini)

Jika repo GitHub sudah siap (Modul 02), push sekarang:

```bash
git add .
git commit -m "chore: initial laravel setup"
git push
```

---

**Modul sebelumnya:** [00 — Project Overview](../00-project-overview/README.md)  
**Modul berikutnya:** [02 — Git & GitHub](../02-git-github/README.md)
