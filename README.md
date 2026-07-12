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

Aplikasi manajemen task yang terus dilengkapi dari dasar sampai siap production.

**Fase 1 (Modul 00–08):** CRUD, auth manual, dashboard, kategori  
**Fase 2 (Modul 09–12):** Upload, relasi lanjut, authorization, refactor  
**Fase 3 (Modul 13–15):** API, testing, email  
**Fase 4 (Modul 16–17):** Deploy & capstone

Gambaran awal: [00 — Project Overview](./00-project-overview/README.md)

> Folder `contoh-project/` (jika ada) adalah referensi lokal dan **tidak ikut di-push** ke GitHub.

---

## Modul Pembelajaran

Kerjakan **berurutan**. Jangan loncat modul.

### Fase 1 — Dasar (±2–3 minggu)

| # | Modul | Apa yang ditambahkan | Estimasi |
|---|---|---|---|
| 00 | [Project Overview](./00-project-overview/README.md) | Gambaran app & milestone | 30 menit |
| 01 | [Setup](./01-setup/README.md) | Install Laravel + MySQL | 1 hari |
| 02 | [Git & GitHub](./02-git-github/README.md) | Push project ke GitHub | 1 hari |
| 03 | [Routing & Controller](./03-routing-controller/README.md) | Route, controller, kirim data | 1–2 hari |
| 04 | [Blade + Bootstrap](./04-blade-template/README.md) | Layout & styling via CDN | 1–2 hari |
| 05 | [Database](./05-database/README.md) | Migration, model, seeder | 2 hari |
| 06 | [CRUD Task](./06-crud-task/README.md) | Tambah, edit, hapus, validasi | 2–3 hari |
| 07 | [Authentication](./07-authentication/README.md) | Login & register manual | 2 hari |
| 08 | [Dashboard & Kategori](./08-project-akhir/README.md) | Dashboard, kategori, search | 2–3 hari |

### Fase 2 — Intermediate (±1–1,5 minggu)

| # | Modul | Apa yang ditambahkan | Estimasi |
|---|---|---|---|
| 09 | [Upload & Storage](./09-upload-storage/README.md) | Upload lampiran task | 1–2 hari |
| 10 | [Relasi Lanjutan](./10-relasi-lanjutan/README.md) | Tag many-to-many, soft delete | 1–2 hari |
| 11 | [Middleware & Authorization](./11-middleware-authorization/README.md) | Role admin, Policy | 1–2 hari |
| 12 | [Form Request & Refactor](./12-form-request-refactor/README.md) | Form Request, Service class | 1–2 hari |

### Fase 3 — API & Quality (±1–1,5 minggu)

| # | Modul | Apa yang ditambahkan | Estimasi |
|---|---|---|---|
| 13 | [API Dasar](./13-api-dasar/README.md) | Endpoint JSON + Sanctum | 2 hari |
| 14 | [Testing](./14-testing/README.md) | Feature test CRUD & auth | 2 hari |
| 15 | [Mail & Notification](./15-mail-notification/README.md) | Email task (Mailtrap) | 1–2 hari |

### Fase 4 — Deploy & Capstone (±3–5 hari)

| # | Modul | Apa yang ditambahkan | Estimasi |
|---|---|---|---|
| 16 | [Deploy](./16-deploy/README.md) | Deploy production | 1–2 hari |
| 17 | [Capstone Review](./17-capstone/README.md) | Polish, checklist, demo | 1–2 hari |

**Total estimasi lengkap:** ±5–7 minggu

---

## Alur Project

```
Fase 1
01 Setup → 02 Git → 03 Route → 04 Blade → 05 DB → 06 CRUD → 07 Auth → 08 Dashboard
                                                                              ↓
Fase 2
09 Upload → 10 Relasi lanjut → 11 Middleware/Policy → 12 Refactor
                                                              ↓
Fase 3
13 API → 14 Testing → 15 Mail
                            ↓
Fase 4
16 Deploy → 17 Capstone → app siap production 🎉
```

Setiap modul: `git add .` → `git commit` → `git push`

---

## Tech Stack

| Layer | Teknologi |
|---|---|
| Backend | Laravel 12, PHP 8.2+ |
| Database | MySQL 8 |
| Frontend | Blade + Bootstrap 5 (CDN) |
| Auth Web | Manual (Auth facade) |
| Auth API | Laravel Sanctum (Modul 13) |
| Testing | PHPUnit / Pest bawaan Laravel (Modul 14) |
| Mail | Laravel Mail + Mailtrap (Modul 15) |
| Version Control | Git + GitHub |

> **Tidak pakai npm/Vite/Tailwind/Laravel UI.** Bootstrap cukup via CDN.

---

## Prasyarat Software

| Software | Versi | Kapan dibutuhkan |
|---|---|---|
| PHP | 8.2+ | Dari awal |
| Composer | 2.x | Dari awal |
| MySQL | 8.x | Dari awal |
| Git | Terbaru | Dari awal |
| Akun GitHub | — | Modul 02 |
| Postman / Insomnia | Opsional | Modul 13 |
| Akun Mailtrap | Gratis | Modul 15 |

```bash
php -v && composer -V && mysql --version && git --version
```

---

## Konvensi Commit

```
chore: initial laravel setup
feat: tambah route dan controller halaman tentang
feat: tambah layout blade dengan bootstrap cdn
feat: tambah migration tasks dan halaman list
feat: crud task lengkap dengan validasi
feat: authentication login register manual
feat: dashboard kategori dan search filter
feat: upload lampiran task
feat: tag many-to-many dan soft delete
feat: role admin dan policy authorization
refactor: form request dan service class
feat: api tasks dengan sanctum
test: feature test auth dan crud task
feat: email notifikasi task
chore: setup deploy production
docs: capstone review dan readme final
```

---

## Workflow Git (Setiap Modul)

```bash
git add .
git commit -m "feat: deskripsi perubahan"
git push
```

Push langsung ke branch `main`. Tidak perlu branch/PR/merge.

---

## Referensi

- [Laravel Documentation](https://laravel.com/docs)
- [Bootstrap 5 Documentation](https://getbootstrap.com/docs/5.3)
- [Git Documentation](https://git-scm.com/doc)
- [Laravel Sanctum](https://laravel.com/docs/sanctum)
- [Mailtrap](https://mailtrap.io)

---

## Troubleshooting Cepat

| Masalah | Solusi |
|---|---|
| `could not find driver` | Install extension `pdo_mysql` |
| `SQLSTATE[HY000] [1045] Access denied` | Cek kredensial MySQL di `.env` |
| `No application encryption key` | `php artisan key:generate` |
| Git push ditolak | Cek remote URL & token GitHub |
| Port 8000 sudah dipakai | `php artisan serve --port=8080` |
| Upload gagal permission | `chmod -R 775 storage` + `php artisan storage:link` |
