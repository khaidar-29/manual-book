# Panduan Belajar Laravel

Belajar Laravel dengan **satu project utuh** — **Task Manager App** — dikerjakan bertahap modul demi modul. Setiap selesai satu tahap, commit dan push ke GitHub.

---

## Project: Task Manager App

Aplikasi manajemen task sederhana. Di akhir pembelajaran, app kamu punya:

- Halaman publik (beranda, tentang)
- CRUD task lengkap
- Login & register
- Task per user
- Dashboard & kategori

Gambaran lengkap: [00 — Project Overview](./00-project-overview/README.md)

---

## Modul Pembelajaran

Kerjakan **berurutan**. Jangan loncat modul.

| # | Modul | Apa yang ditambahkan | Estimasi |
|---|---|---|---|
| 00 | [Project Overview](./00-project-overview/README.md) | Gambaran app akhir & milestone | 30 menit |
| 01 | [Setup](./01-setup/README.md) | Install Laravel, project jalan | 1 hari |
| 02 | [Git & GitHub](./02-git-github/README.md) | Version control, push ke GitHub | 1 hari |
| 03 | [Routing & Controller](./03-routing-controller/README.md) | Route, controller, kirim data | 1–2 hari |
| 04 | [Blade Template](./04-blade-template/README.md) | Layout, navbar, sintaks Blade | 1–2 hari |
| 05 | [Database](./05-database/README.md) | Migration, model, seeder, list task | 2 hari |
| 06 | [CRUD Task](./06-crud-task/README.md) | Tambah, edit, hapus, validasi | 2–3 hari |
| 07 | [Authentication](./07-authentication/README.md) | Login, register, proteksi route | 2 hari |
| 08 | [Project Akhir](./08-project-akhir/README.md) | Dashboard, kategori, search | 2–3 hari |

**Total estimasi:** ±2–3 minggu

---

## Alur Project per Modul

```
Modul 01   Project Laravel kosong ✅
    ↓ git init + commit
Modul 02   Repo GitHub + push pertama
    ↓
Modul 03   + Route & controller (halaman tentang)
    ↓ commit & push
Modul 04   + Layout Blade & navbar
    ↓ commit & push
Modul 05   + Tabel tasks, list dari database
    ↓ commit & push
Modul 06   + CRUD task lengkap
    ↓ commit & push
Modul 07   + Login, task milik user
    ↓ commit & push
Modul 08   + Dashboard, kategori, search
    ↓ commit & push → app siap demo 🎉
```

---

## Prasyarat Software

| Software | Versi |
|---|---|
| PHP | 8.2+ |
| Composer | 2.x |
| Node.js & NPM | 18+ |
| Git | Terbaru |
| MySQL / SQLite | Untuk database |
| Akun GitHub | Untuk push project |

Cek versi:

```bash
php -v && composer -V && node -v && git --version
```

---

## Konvensi Commit

Gunakan format ini di setiap modul:

```
feat: tambah halaman tentang
feat: buat layout blade dan navbar
feat: tampilkan list task dari database
feat: crud task lengkap dengan validasi
feat: authentication dengan laravel breeze
feat: dashboard dan kategori task
```

---

## Referensi

- [Laravel Documentation](https://laravel.com/docs)
- [Git Documentation](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com)

---

## Troubleshooting Cepat

| Masalah | Solusi |
|---|---|
| `could not find driver` | Install extension `pdo_sqlite` atau `pdo_mysql` |
| `No application encryption key` | `php artisan key:generate` |
| `Vite manifest not found` | `npm install && npm run dev` |
| Git push ditolak | Cek remote URL & login GitHub |
| Port 8000 sudah dipakai | `php artisan serve --port=8080` |

Detail troubleshooting ada di masing-masing modul.
