# Modul 00 — Project Overview

Gambaran lengkap **Task Manager App** — project yang akan kamu bangun dari awal sampai selesai.

**Estimasi waktu:** 30 menit (baca & pahami)  
**Prasyarat:** Tidak ada

---

## Apa itu Task Manager App?

Task Manager adalah aplikasi web untuk mengelola daftar pekerjaan (task/to-do). User bisa mendaftar, login, lalu membuat dan mengelola task miliknya sendiri.

Project ini **bukan** kumpulan latihan terpisah. Kamu membangun **satu aplikasi** yang terus dilengkapi setiap modul.

---

## Fitur Akhir (Setelah Modul 08)

### Halaman Publik (tanpa login)
- **Beranda** — pengenalan aplikasi
- **Tentang** — informasi project

### Authentication
- **Register** — daftar akun baru
- **Login** — masuk ke aplikasi
- **Logout** — keluar dari aplikasi

### Task Management (per user, butuh login)
- Lihat daftar task
- Tambah task baru
- Edit task
- Hapus task
- Tandai selesai / belum selesai

### Dashboard
- Total task
- Jumlah task selesai vs belum
- Task terbaru
- Ringkasan per kategori

### Kategori
- Buat kategori (Work, Personal, Study, dll.)
- Assign kategori ke task
- Filter task berdasarkan kategori

---

## Milestone per Modul

| Modul | Apa yang ditambahkan | Hasil di browser |
|---|---|---|
| 01 | Install Laravel + MySQL | Halaman welcome Laravel |
| 02 | Push ke GitHub | Project ada di GitHub |
| 03 | Route + Controller | `/` dan `/tentang` jalan |
| 04 | Layout Blade + Bootstrap CDN | Halaman rapi dengan navbar |
| 05 | Migration + Model + Seeder | `/tasks` tampil data dari DB |
| 06 | CRUD lengkap | Bisa tambah/edit/hapus task |
| 07 | Login + Register manual | Harus login untuk akses tasks |
| 08 | Dashboard + Kategori + Search | App siap demo |

---

## Struktur Database Akhir

### Tabel `users` (bawaan Laravel)

| Kolom | Tipe | Keterangan |
|---|---|---|
| id | BIGINT | Primary key |
| name | VARCHAR | Nama user |
| email | VARCHAR | Email (unik) |
| password | VARCHAR | Password (di-hash) |
| created_at | TIMESTAMP | Waktu dibuat |
| updated_at | TIMESTAMP | Waktu diupdate |

### Tabel `tasks`

| Kolom | Tipe | Keterangan |
|---|---|---|
| id | BIGINT | Primary key |
| user_id | BIGINT | FK ke users |
| category_id | BIGINT (nullable) | FK ke categories |
| title | VARCHAR | Judul task |
| description | TEXT (nullable) | Deskripsi |
| is_done | BOOLEAN | Status selesai |
| created_at | TIMESTAMP | Waktu dibuat |
| updated_at | TIMESTAMP | Waktu diupdate |

### Tabel `categories`

| Kolom | Tipe | Keterangan |
|---|---|---|
| id | BIGINT | Primary key |
| user_id | BIGINT | FK ke users |
| name | VARCHAR | Nama kategori |
| color | VARCHAR | Warna badge |
| created_at | TIMESTAMP | Waktu dibuat |
| updated_at | TIMESTAMP | Waktu diupdate |

> Tabel `users` sudah ada bawaan Laravel. Tabel `tasks` dibuat Modul 05, kolom `user_id` Modul 07, tabel `categories` Modul 08.

---

## Tech Stack

| Layer | Teknologi | Catatan |
|---|---|---|
| Backend | Laravel 12, PHP 8.2+ | Framework utama |
| Database | MySQL 8 | Satu-satunya database |
| Frontend | Blade + Bootstrap 5 CDN | **Tidak pakai npm/Vite** |
| Auth | Manual (Auth facade) | **Tidak pakai Laravel UI/Breeze** |
| Version Control | Git + GitHub | Push langsung ke main |

---

## Nama Project

Saat install di Modul 01:

```bash
composer create-project laravel/laravel task-manager
```

Folder `task-manager/` dipakai untuk **semua modul**. Jangan buat project baru.

---

## Cara Menggunakan Manual Ini

1. Baca modul ini sampai paham gambaran akhir
2. Kerjakan modul **01 → 08** berurutan
3. Setiap selesai modul → `git add .` → `git commit` → `git push`
4. Centang checklist di akhir setiap modul
5. Jika error, baca bagian Troubleshooting modul tersebut

---

## Struktur Folder Manual

```
manual-app/                  ← repo manual (dokumentasi)
├── README.md
├── 00-project-overview/
├── 01-setup/
├── ...
└── 08-project-akhir/

task-manager/                ← project Laravel kamu (dibuat di Modul 01)
├── app/
├── routes/
├── resources/views/
└── database/
```

---

**Modul berikutnya:** [01 — Setup](../01-setup/README.md)
