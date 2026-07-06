# Panduan Belajar Laravel

Belajar Laravel dengan **satu project utuh** — **Task Manager App** — dikerjakan bertahap modul demi modul.

Setiap selesai satu modul:

```bash
git add .
git commit -m "pesan commit"
git push
```

Tidak perlu buat branch atau merge Pull Request.

---

## Project: Task Manager App

Aplikasi manajemen task sederhana. Di akhir pembelajaran, app kamu punya:

- Halaman publik (beranda, tentang)
- CRUD task lengkap
- Login & register (manual, tanpa package auth)
- Task per user
- Dashboard & kategori

Gambaran lengkap: [00 — Project Overview](./00-project-overview/README.md)

---

## Modul Pembelajaran

Kerjakan **berurutan**. Jangan loncat modul.

| # | Modul | Apa yang ditambahkan | Estimasi |
|---|---|---|---|
| 00 | [Project Overview](./00-project-overview/README.md) | Gambaran app akhir & milestone | 30 menit |
| 01 | [Setup](./01-setup/README.md) | Install Laravel + MySQL | 1 hari |
| 02 | [Git & GitHub](./02-git-github/README.md) | Push project ke GitHub | 1 hari |
| 03 | [Routing & Controller](./03-routing-controller/README.md) | Route, controller, kirim data | 1–2 hari |
| 04 | [Blade + Bootstrap](./04-blade-template/README.md) | Layout & styling via CDN | 1–2 hari |
| 05 | [Database](./05-database/README.md) | Migration, model, seeder | 2 hari |
| 06 | [CRUD Task](./06-crud-task/README.md) | Tambah, edit, hapus, validasi | 2–3 hari |
| 07 | [Authentication](./07-authentication/README.md) | Login & register manual | 2 hari |
| 08 | [Project Akhir](./08-project-akhir/README.md) | Dashboard, kategori, search | 2–3 hari |

**Total estimasi:** ±2–3 minggu

---

## Alur Project per Modul

```
Modul 01   Project Laravel + MySQL jalan
    ↓ git init → commit → push
Modul 02   Repo GitHub terhubung
    ↓ add → commit → push
Modul 03   + Route & controller
    ↓ add → commit → push
Modul 04   + Layout Blade + Bootstrap CDN
    ↓ add → commit → push
Modul 05   + Database & list task
    ↓ add → commit → push
Modul 06   + CRUD task lengkap
    ↓ add → commit → push
Modul 07   + Login & register
    ↓ add → commit → push
Modul 08   + Dashboard & kategori
    ↓ add → commit → push → app siap demo 🎉
```

---

## Tech Stack

| Layer | Teknologi |
|---|---|
| Backend | Laravel 12, PHP 8.2+ |
| Database | MySQL 8 |
| Frontend | Blade + Bootstrap 5 (CDN) |
| Auth | Manual (Auth facade bawaan Laravel) |
| Version Control | Git + GitHub |

> **Tidak pakai npm/Vite/Tailwind/Laravel UI.** Bootstrap cukup via CDN.

---

## Prasyarat Software

| Software | Versi |
|---|---|
| PHP | 8.2+ |
| Composer | 2.x |
| MySQL | 8.x |
| Git | Terbaru |
| Akun GitHub | Untuk push project |

```bash
php -v && composer -V && mysql --version && git --version
```

---

## Konvensi Commit

```
chore: initial laravel setup
docs: tambah readme project
feat: tambah route dan controller halaman tentang
feat: tambah layout blade dengan bootstrap cdn
feat: tambah migration tasks dan halaman list
feat: crud task lengkap dengan validasi
feat: authentication login register manual
feat: dashboard kategori dan search filter
```

---

## Workflow Git (Setiap Modul)

```bash
git add .
git commit -m "feat: deskripsi perubahan"
git push
```

Itu saja. Push langsung ke branch `main`.

---

## Referensi

- [Laravel Documentation](https://laravel.com/docs)
- [Bootstrap 5 Documentation](https://getbootstrap.com/docs/5.3)
- [Git Documentation](https://git-scm.com/doc)

---

## Troubleshooting Cepat

| Masalah | Solusi |
|---|---|
| `could not find driver` | Install extension `pdo_mysql` |
| `SQLSTATE[HY000] [1045] Access denied` | Cek kredensial MySQL di `.env` |
| `No application encryption key` | `php artisan key:generate` |
| Git push ditolak | Cek remote URL & token GitHub |
| Port 8000 sudah dipakai | `php artisan serve --port=8080` |
