# Modul 00 — Project Overview

Gambaran **Task Manager App** — project yang akan kamu bangun dari nol sampai selesai.

**Estimasi waktu:** 30 menit (baca & pahami)  
**Prasyarat:** Tidak ada

---

## Apa itu Task Manager App?

Aplikasi web sederhana untuk mengelola daftar task (to-do). User bisa login, membuat task, menandai selesai, dan mengelompokkan task berdasarkan kategori.

---

## Fitur Akhir (Modul 08)

```
Task Manager App
│
├── 🌐 Halaman Publik
│   ├── Beranda
│   └── Tentang
│
├── 🔐 Authentication
│   ├── Register
│   ├── Login
│   └── Logout
│
├── 📋 Task Management (per user)
│   ├── Lihat daftar task
│   ├── Tambah task
│   ├── Edit task
│   ├── Hapus task
│   └── Tandai selesai
│
├── 📊 Dashboard
│   ├── Total task
│   ├── Task selesai vs belum
│   └── Task per kategori
│
└── 🏷️ Kategori
    ├── Work, Personal, Study, dll.
    └── Filter task by kategori
```

---

## Milestone per Modul

| Modul | State project | Bisa diakses di browser |
|---|---|---|
| 01 | Laravel fresh install | Halaman welcome |
| 02 | Repo di GitHub | (sama, sudah di GitHub) |
| 03 | Route + controller | `/`, `/tentang` |
| 04 | Layout Blade | Halaman dengan navbar |
| 05 | Database | `/tasks` — list dari DB |
| 06 | CRUD | Tambah, edit, hapus task |
| 07 | Auth | Harus login untuk akses tasks |
| 08 | Polish | Dashboard, kategori, search |

---

## Struktur Database Akhir

```
users
├── id
├── name
├── email
├── password
└── timestamps

categories
├── id
├── name
├── user_id (FK)
└── timestamps

tasks
├── id
├── title
├── description
├── is_done
├── category_id (FK, nullable)
├── user_id (FK)
└── timestamps
```

> Tabel `users` sudah ada bawaan Laravel. Tabel `categories` dan kolom relasi ditambahkan di modul 07–08.

---

## Tech Stack

| Layer | Teknologi |
|---|---|
| Backend | Laravel 12, PHP 8.2+ |
| Database | SQLite (dev) / MySQL (production) |
| Frontend | Blade + Tailwind CSS |
| Auth | Laravel Breeze |
| Version Control | Git + GitHub |

---

## Nama Project

Saat install di Modul 01, gunakan nama:

```bash
composer create-project laravel/laravel task-manager
```

Folder `task-manager/` adalah project kamu untuk **semua modul**. Jangan buat project baru di modul berikutnya.

---

## Cara Menggunakan Manual Ini

1. Baca modul overview ini
2. Kerjakan modul 01 → 08 **berurutan**
3. Setiap selesai modul → **commit & push** ke GitHub (Modul 02 ajarkan caranya)
4. Centang checklist di akhir setiap modul
5. Jika stuck, baca bagian Troubleshooting di modul tersebut

---

**Modul berikutnya:** [01 — Setup](../01-setup/README.md)
