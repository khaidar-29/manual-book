# Modul 01 — Setup & Instalasi

Membuat project **Task Manager** dan menjalankannya di local.

**Estimasi waktu:** 1 hari  
**Prasyarat:** [Modul 00 — Project Overview](../00-project-overview/README.md)

---

## Tujuan Modul

- [ ] Software terinstall (PHP, Composer, MySQL, Git)
- [ ] Project `task-manager` dibuat
- [ ] Database MySQL & `.env` dikonfigurasi
- [ ] Server jalan di browser
- [ ] Git di-init + commit pertama

---

## Langkah 1: Cek Software

```bash
php -v          # Minimal 8.2
composer -V
mysql --version
git --version
```

### Install cepat

**macOS:**
```bash
brew install php composer mysql git
brew services start mysql
```

**Windows:** [Laragon](https://laragon.org) — sudah include PHP, MySQL, Composer

**Linux:**
```bash
sudo apt install php8.2 php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl composer mysql-server git
```

### Extension PHP wajib

```bash
php -m | grep pdo_mysql
```

---

## Langkah 2: Buat Database MySQL

```bash
mysql -u root -p
```

```sql
CREATE DATABASE task_manager;
EXIT;
```

---

## Langkah 3: Buat Project Laravel

```bash
cd ~/projects   # atau folder kerja kamu

composer create-project laravel/laravel task-manager
cd task-manager
```

---

## Langkah 4: Konfigurasi Environment

```bash
cp .env.example .env
php artisan key:generate
```

Edit `.env`:

```env
APP_NAME="Task Manager"
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager
DB_USERNAME=root
DB_PASSWORD=        # isi password MySQL kamu
```

---

## Langkah 5: Migration & Server

```bash
php artisan migrate
php artisan serve
```

Buka **http://localhost:8000** — halaman welcome Laravel harus tampil.

> **Catatan:** Manual ini pakai **Bootstrap 5 via CDN** untuk styling. Tidak perlu `npm run dev` sampai Modul 07 (authentication).

---

## Langkah 6: Kenalan Struktur Folder

```
task-manager/
├── app/Http/Controllers/   ← Logika aplikasi
├── app/Models/             ← Model database
├── routes/web.php          ← Definisi URL
├── resources/views/        ← Tampilan HTML (Blade)
├── database/migrations/    ← Struktur tabel
├── .env                    ← Konfigurasi (JANGAN di-commit!)
└── artisan                 ← CLI tool
```

---

## Langkah 7: Git Init + Commit Pertama

```bash
git init
git add .
git commit -m "chore: initial laravel setup"
```

> `.env` sudah otomatis di-ignore oleh `.gitignore` bawaan Laravel. Jangan pernah commit file `.env`.

---

## Latihan

1. Jalankan `php artisan route:list` — lihat route bawaan Laravel
2. Jalankan `php artisan tinker`, ketik `app()->version()` — cek versi
3. Ubah `APP_NAME` di `.env`, refresh browser
4. Pastikan `git status` bersih setelah commit

---

## Troubleshooting

| Error | Solusi |
|---|---|
| `could not find driver` | Install extension `pdo_mysql` |
| `SQLSTATE[HY000] [1045] Access denied` | Cek `DB_USERNAME` & `DB_PASSWORD` di `.env` |
| `Unknown database 'task_manager'` | Buat database di MySQL dulu |
| `No application encryption key` | `php artisan key:generate` |

---

## Checklist Selesai

- [ ] Project `task-manager` dibuat
- [ ] Database MySQL `task_manager` dibuat
- [ ] `.env` dikonfigurasi, migrate berhasil
- [ ] `php artisan serve` jalan
- [ ] Halaman welcome tampil di browser
- [ ] `git init` + commit pertama selesai

---

**Modul sebelumnya:** [00 — Project Overview](../00-project-overview/README.md)  
**Modul berikutnya:** [02 — Git & GitHub](../02-git-github/README.md)
