# Modul 01 — Setup & Instalasi

Membuat project **Task Manager** dan menjalankannya di local.

**Estimasi waktu:** 1 hari  
**Prasyarat:** [Modul 00 — Project Overview](../00-project-overview/README.md)

---

## Tujuan Modul

- [ ] Software terinstall (PHP, Composer, Node, Git)
- [ ] Project `task-manager` dibuat
- [ ] Database & `.env` dikonfigurasi
- [ ] Server jalan di browser
- [ ] Git di-init + commit pertama

---

## Langkah 1: Cek Software

```bash
php -v          # Minimal 8.2
composer -V
node -v
npm -v
git --version
```

### Install cepat

**macOS:** `brew install php composer node git mysql`  
**Windows:** [Laragon](https://laragon.org)  
**Linux:** `sudo apt install php8.2 php8.2-sqlite3 php8.2-mbstring composer nodejs npm git`

---

## Langkah 2: Buat Project Laravel

```bash
cd ~/projects   # atau folder kerja kamu

composer create-project laravel/laravel task-manager
cd task-manager
```

---

## Langkah 3: Konfigurasi Environment

```bash
cp .env.example .env
php artisan key:generate
```

Edit `.env`:

```env
APP_NAME="Task Manager"
APP_URL=http://localhost:8000

# SQLite (paling mudah untuk belajar)
DB_CONNECTION=sqlite
```

Buat file database:

```bash
touch database/database.sqlite
```

> **Alternatif MySQL:** set `DB_CONNECTION=mysql`, buat database `task_manager`, isi `DB_USERNAME` dan `DB_PASSWORD`.

---

## Langkah 4: Migration & Server

```bash
php artisan migrate
```

Buka **dua terminal**:

```bash
# Terminal 1
php artisan serve

# Terminal 2
npm install && npm run dev
```

Buka **http://localhost:8000** — halaman welcome Laravel harus tampil.

---

## Langkah 5: Kenalan Struktur Folder

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

## Langkah 6: Git Init + Commit Pertama

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
| `could not find driver` | Install `pdo_sqlite` atau `pdo_mysql` |
| `No application encryption key` | `php artisan key:generate` |
| `Vite manifest not found` | `npm install && npm run dev` |

---

## Checklist Selesai

- [ ] Project `task-manager` dibuat
- [ ] `.env` dikonfigurasi, migrate berhasil
- [ ] `php artisan serve` + `npm run dev` jalan
- [ ] Halaman welcome tampil di browser
- [ ] `git init` + commit pertama selesai

---

**Modul sebelumnya:** [00 — Project Overview](../00-project-overview/README.md)  
**Modul berikutnya:** [02 — Git & GitHub](../02-git-github/README.md)
