# Modul 01 — Setup & Instalasi

Modul pertama: menyiapkan environment dan menjalankan project Laravel pertama kali.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** Tidak ada

---

## Tujuan Pembelajaran

Setelah menyelesaikan modul ini, kamu sudah bisa:

- [ ] Mengecek dan menginstall software yang dibutuhkan
- [ ] Membuat project Laravel baru
- [ ] Mengkonfigurasi file `.env`
- [ ] Menjalankan development server
- [ ] Memahami struktur folder dasar Laravel

---

## Langkah 1: Cek Software

Jalankan perintah berikut di terminal:

```bash
php -v          # Minimal PHP 8.2
composer -V     # Minimal Composer 2.x
node -v         # Minimal Node.js 18
npm -v
mysql --version # Atau gunakan SQLite untuk latihan
git --version
```

### Extension PHP yang wajib aktif

```bash
php -m | grep -E 'pdo|mbstring|openssl|tokenizer|xml|ctype|json|bcmath|fileinfo'
```

Jika ada yang belum terinstall, install dulu sebelum lanjut.

### Install cepat (macOS)

```bash
brew install php composer node mysql
brew services start mysql
```

### Install cepat (Windows)

Download **Laragon** dari [laragon.org](https://laragon.org) — sudah include PHP, MySQL, Composer, dan Node.js.

---

## Langkah 2: Buat Project Laravel

```bash
# Pindah ke folder kerja kamu
cd ~/projects

# Buat project baru
composer create-project laravel/laravel belajar-laravel

# Masuk ke folder project
cd belajar-laravel
```

Tunggu sampai proses download selesai. Folder `belajar-laravel` adalah project kamu untuk semua modul berikutnya.

---

## Langkah 3: Konfigurasi Environment

```bash
cp .env.example .env
php artisan key:generate
```

Buka file `.env` dan sesuaikan:

```env
APP_NAME="Belajar Laravel"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

# Opsi A: SQLite (paling mudah untuk latihan)
DB_CONNECTION=sqlite
# DB_DATABASE akan otomatis pakai database/database.sqlite

# Opsi B: MySQL
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=belajar_laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

### Jika pakai SQLite

```bash
touch database/database.sqlite
```

### Jika pakai MySQL

```bash
mysql -u root -p
```

```sql
CREATE DATABASE belajar_laravel;
EXIT;
```

---

## Langkah 4: Jalankan Migration

```bash
php artisan migrate
```

Jika berhasil, akan muncul pesan migration selesai. Ini artinya koneksi database sudah benar.

---

## Langkah 5: Jalankan Development Server

Buka **dua terminal**:

**Terminal 1 — Laravel server:**
```bash
php artisan serve
```

**Terminal 2 — Vite (asset CSS/JS):**
```bash
npm install
npm run dev
```

Buka browser: **http://localhost:8000**

Jika muncul halaman welcome Laravel, setup kamu sudah berhasil.

---

## Langkah 6: Kenalan Struktur Folder

Buka project di editor (VS Code / Cursor) dan perhatikan folder berikut:

```
belajar-laravel/
├── app/Http/Controllers/   ← Logika aplikasi
├── app/Models/             ← Model database
├── routes/web.php          ← Definisi URL
├── resources/views/        ← File tampilan HTML
├── database/migrations/    ← Struktur tabel database
├── .env                    ← Konfigurasi (JANGAN di-commit!)
└── artisan                 ← Command-line tool
```

Cukup hafalkan 5 folder di atas dulu. Detailnya akan dipelajari di modul berikutnya.

---

## Latihan

Kerjakan latihan berikut untuk memastikan setup berjalan:

1. Jalankan `php artisan serve` dan buka `http://localhost:8000`
2. Jalankan `php artisan route:list` — lihat daftar route yang ada
3. Jalankan `php artisan tinker` lalu ketik `app()->version()` — cek versi Laravel
4. Ubah `APP_NAME` di `.env`, refresh browser, lihat apakah nama berubah

---

## Troubleshooting

| Error | Solusi |
|---|---|
| `could not find driver` | Install extension `pdo_sqlite` atau `pdo_mysql` |
| `No application encryption key` | Jalankan `php artisan key:generate` |
| `SQLSTATE[HY000] [1045] Access denied` | Cek `DB_USERNAME` dan `DB_PASSWORD` di `.env` |
| `Vite manifest not found` | Jalankan `npm install && npm run dev` |
| Port 8000 sudah dipakai | `php artisan serve --port=8080` |

---

## Checklist Selesai

- [ ] PHP, Composer, Node.js terinstall
- [ ] Project Laravel berhasil dibuat
- [ ] File `.env` sudah dikonfigurasi
- [ ] `php artisan migrate` berhasil
- [ ] `php artisan serve` + `npm run dev` berjalan
- [ ] Halaman welcome Laravel tampil di browser

---

**Modul berikutnya:** [02 — MVC Dasar](../02-mvc-dasar/README.md)
