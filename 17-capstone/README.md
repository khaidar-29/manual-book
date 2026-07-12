# Modul 17 — Capstone Review

Modul terakhir: mereview **seluruh Task Manager App**, merapikan detail UX, menyusun **README final** untuk GitHub, menyiapkan **demo 5–10 menit**, dan menilai diri sendiri sebelum memasukkan project ke portfolio.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 16 — Deploy](../16-deploy/README.md)

> Selamat datang di garis finis. Tidak ada fitur besar baru yang wajib — fokus pada **kualitas, kelengkapan, dan presentasi**.

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Memverifikasi checklist fitur Modul 01–16
- [ ] Merapikan 404, empty state, dan confirm dialog (Bootstrap)
- [ ] Menulis README.md project yang siap portfolio
- [ ] Menjalankan demo script 5–10 menit dengan alur jelas
- [ ] Menilai diri dengan rubrik self-assessment
- [ ] Menyusun langkah karier / wawancara berikutnya
- [ ] Commit final: `docs: capstone review dan readme final`

---

## Perayaan Dulu 🎉

Kamu telah membangun **satu aplikasi utuh** — bukan sekadar tutorial terpisah.

Dari `composer create-project` sampai email, API, testing, dan deploy: itu pola kerja mirip magang / junior Laravel di dunia nyata.

Stack yang kamu kuasai di project ini:

| Layer | Teknologi |
|---|---|
| Backend | Laravel + PHP |
| Database | MySQL |
| UI | Blade + Bootstrap 5 CDN |
| Auth | Manual (Auth facade) |
| API | JSON + Sanctum (Modul 13) |
| Quality | PHPUnit feature tests (Modul 14) |
| Email | Mailable + Mailtrap/SMTP (Modul 15) |
| Ops | Deploy production (Modul 16) |
| Git | `add` → `commit` → `push` ke `main` |

---

## Tabel Perjalanan Modul 01–17

| Modul | Topik | Milestone | Contoh commit |
|---|---|---|---|
| 00 | Project Overview | Pahami scope Task Manager | — |
| 01 | Setup | Laravel + MySQL jalan | `chore: initial laravel setup` |
| 02 | Git & GitHub | Repo remote + push | `docs: tambah readme project` |
| 03 | Routing & Controller | Route, controller, data ke view | `feat: tambah route dan controller` |
| 04 | Blade + Bootstrap | Layout CDN konsisten | `feat: tambah layout blade bootstrap cdn` |
| 05 | Database | Migration, model, seeder, list | `feat: tambah migration tasks dan halaman list` |
| 06 | CRUD Task | Create/edit/delete + validasi | `feat: crud task lengkap dengan validasi` |
| 07 | Authentication | Login/register manual + owner task | `feat: authentication login register manual` |
| 08 | Dashboard & Kategori | Stats, kategori, search/filter | `feat: dashboard kategori search filter readme` |
| 09 | Upload & Storage | Lampiran task + `storage:link` | `feat: upload lampiran task` |
| 10 | Relasi Lanjutan | Tag many-to-many, soft delete | `feat: tag many-to-many dan soft delete` |
| 11 | Middleware & Authorization | Role admin, Policy | `feat: role admin dan policy authorization` |
| 12 | Form Request & Refactor | Form Request + Service | `refactor: form request dan task service` |
| 13 | API Dasar | Endpoint JSON + Sanctum | `feat: api task dengan sanctum` |
| 14 | Testing | Feature test CRUD & auth | `test: feature test crud dan auth` |
| 15 | Mail & Notification | Email task via Mailtrap | `feat: email notifikasi task` |
| 16 | Deploy | Production checklist & live | `chore: setup deploy production` |
| 17 | Capstone Review | Polish, README final, demo | `docs: capstone review dan readme final` |

Setiap selesai modul: **hanya** `git add` → `git commit` → `git push` ke `main` — tanpa PR/merge/branch workflow di panduan ini.

---

## Checklist Fitur Lengkap (Modul 01–16)

Centang semua yang relevan. Item opsional ditandai *(opsional)*.

### Fondasi (01–05)

- [ ] Project Laravel terinstall, jalan di local
- [ ] MySQL terhubung (`DB_CONNECTION=mysql`)
- [ ] Repo GitHub ada, history commit rapi
- [ ] Route & controller terstruktur
- [ ] Layout Blade + Bootstrap 5 CDN (tanpa Vite/Tailwind/Laravel UI)
- [ ] Migration & model Task (dan User)
- [ ] Seeder data demo

### CRUD & Auth (06–08)

- [ ] Resource CRUD task lengkap
- [ ] Validasi + flash message
- [ ] Login, register, logout manual
- [ ] Task terikat `user_id` (user hanya lihat miliknya)
- [ ] Dashboard statistik
- [ ] CRUD kategori
- [ ] Search / filter / pagination

### Intermediate (09–12)

- [ ] Upload lampiran / gambar task
- [ ] `php artisan storage:link` berfungsi
- [ ] Tag many-to-many *(atau relasi lanjut setara)*
- [ ] Soft delete *(jika diajarkan di modul 10)*
- [ ] Role admin + middleware
- [ ] Policy authorization (update/delete)
- [ ] Form Request class
- [ ] TaskService (atau service layer setara)

### API, Quality, Mail (13–15)

- [ ] API JSON untuk tasks (index/store/update/destroy minimal)
- [ ] Auth API Sanctum (token)
- [ ] Feature tests auth & CRUD (minimal beberapa case hijau)
- [ ] Email saat task dibuat (Mailtrap/SMTP)
- [ ] Template email terbaca di inbox testing
- [ ] Notification database *(opsional)*

### Deploy (16)

- [ ] `APP_ENV=production`, `APP_DEBUG=false` di server
- [ ] Document root ke `public/`
- [ ] `composer install --no-dev`
- [ ] `migrate --force`, `storage:link`, cache config/route/view
- [ ] `.env` tidak ter-commit
- [ ] HTTPS / rencana SSL jelas
- [ ] URL live atau dokumentasi deploy lengkap di README

### Non-fitur tapi wajib

- [ ] `.env.example` lengkap tanpa secret
- [ ] README project jelas (install steps)
- [ ] Tidak ada Bootstrap build via npm (CDN saja)
- [ ] Tidak ada SQLite sebagai DB utama pembelajaran

---

## Polish Tips (Wajib Direview)

Capstone = kesan profesional. Kerjakan item di bawah jika belum ada.

### 1. Halaman 404

Buat `resources/views/errors/404.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Halaman Tidak Ditemukan')

@section('content')
<div class="container py-5 text-center">
    <h1 class="display-4">404</h1>
    <p class="lead text-muted">Halaman yang kamu cari tidak ditemukan.</p>
    <a href="{{ route('dashboard') }}" class="btn btn-primary">Kembali ke Dashboard</a>
</div>
@endsection
```

Uji: buka URL acak seperti `/halaman-tidak-ada`.

### 2. Empty States (Bootstrap)

Jangan biarkan tabel kosong membosankan.

Contoh di `tasks/index.blade.php`:

```blade
@forelse ($tasks as $task)
    {{-- baris task --}}
@empty
    <div class="alert alert-light border text-center py-5">
        <h5 class="mb-2">Belum ada task</h5>
        <p class="text-muted mb-3">Buat task pertama untuk mulai produktif.</p>
        <a href="{{ route('tasks.create') }}" class="btn btn-primary">Tambah Task</a>
    </div>
@endforelse
```

Terapkan juga untuk:

- Daftar kategori kosong
- Hasil search tidak ketemu
- Notifikasi kosong *(jika ada)*

### 3. Confirm Dialog Hapus

Hindari hapus tanpa konfirmasi.

```blade
<form action="{{ route('tasks.destroy', $task) }}" method="POST"
      onsubmit="return confirm('Yakin hapus task ini?')">
    @csrf
    @method('DELETE')
    <button type="submit" class="btn btn-sm btn-outline-danger">Hapus</button>
</form>
```

Atau Bootstrap Modal jika ingin lebih rapi — yang penting ada konfirmasi.

### 4. Flash Message Konsisten

Pastikan layout menampilkan success/error:

```blade
@if (session('success'))
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        {{ session('success') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif

@if (session('error'))
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        {{ session('error') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif
```

### 5. Validasi Error di Form

Semua form penting punya `@error` + `old()`.

### 6. Navbar State

- `@guest` → Login / Register
- `@auth` → Dashboard, Tasks, Categories, Logout, nama user
- Link aktif (opsional) dengan `request()->routeIs(...)`

### 7. Responsive Cek

Buka Chrome DevTools → mode mobile. Pastikan:

- Navbar collapse Bootstrap berfungsi
- Tabel tidak overflow parah (bisa `table-responsive`)
- Form nyaman di layar kecil

### 8. Konsistensi Bahasa UI

Pilih Indonesia **atau** campuran yang konsisten. Hindari tombol "Submit" di satu halaman dan "Simpan" di halaman lain tanpa alasan.

### 9. Seed Data Demo untuk Presentasi

Siapkan seeder yang langsung siap demo:

```text
Admin: admin@taskmanager.test / password
User:  demo@taskmanager.test / password
Beberapa task, kategori, tag
```

### 10. Matikan Debug di Production

Double-check server:

```env
APP_DEBUG=false
APP_ENV=production
```

---

## Template README.md Final untuk GitHub Project

Ganti `README.md` di root **project Task Manager** (bukan repo panduan `manual-app`) dengan versi lengkap seperti di bawah. Sesuaikan nama user, URL, dan fitur yang benar-benar kamu buat.

````markdown
# Task Manager

Aplikasi manajemen tugas berbasis **Laravel** + **MySQL** + **Bootstrap 5 (CDN)**.

Dibangun bertahap sebagai project pembelajaran full-stack: CRUD, authentication manual, dashboard, upload, authorization, API, testing, email, hingga deploy production.

## Fitur

- Register, login, logout (session auth manual)
- CRUD task per user
- Kategori task + search/filter/pagination
- Dashboard statistik
- Upload lampiran task
- Tag (many-to-many) & soft delete
- Role admin + Policy authorization
- Form Request & Service layer
- REST API + Laravel Sanctum
- Feature tests (PHPUnit)
- Email notifikasi saat task dibuat
- Siap deploy (document root `public/`)

## Tech Stack

| Layer | Teknologi |
|---|---|
| Framework | Laravel |
| Database | MySQL |
| Frontend | Blade + Bootstrap 5 CDN |
| Auth Web | Laravel Auth facade (manual) |
| Auth API | Laravel Sanctum |
| Email | SMTP (Mailtrap untuk development) |

**Tidak memakai:** Vite build untuk UI, Tailwind, Laravel Breeze/UI/Jetstream sebagai starter auth.

## Requirement

- PHP 8.2+ (sesuaikan versi Laravel)
- Composer
- MySQL
- Extensi PHP umum Laravel (openssl, pdo, mbstring, tokenizer, xml, ctype, json, bcmath, fileinfo)

## Instalasi (Local)

```bash
git clone https://github.com/USERNAME/task-manager.git
cd task-manager
composer install
cp .env.example .env
php artisan key:generate
```

Edit `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager
DB_USERNAME=root
DB_PASSWORD=

MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
```

```bash
php artisan migrate --seed
php artisan storage:link
php artisan serve
```

Buka: http://127.0.0.1:8000

### Akun demo (jika di-seed)

| Role | Email | Password |
|---|---|---|
| Demo | demo@taskmanager.test | password |
| Admin | admin@taskmanager.test | password |

## Menjalankan Test

```bash
php artisan test
```

## API (ringkas)

Autentikasi: Sanctum token (endpoint login/token sesuai implementasi).

Contoh header:

```http
Authorization: Bearer {token}
Accept: application/json
```

Endpoint umum:

| Method | URI | Keterangan |
|---|---|---|
| GET | `/api/tasks` | List task user |
| POST | `/api/tasks` | Buat task |
| GET | `/api/tasks/{id}` | Detail |
| PUT | `/api/tasks/{id}` | Update |
| DELETE | `/api/tasks/{id}` | Hapus |

## Deploy (Production)

1. Clone/upload kode ke server
2. Document root mengarah ke `public/`
3. `composer install --no-dev --optimize-autoloader`
4. Siapkan `.env` production (`APP_ENV=production`, `APP_DEBUG=false`, MySQL prod)
5. `php artisan key:generate` (jika perlu)
6. `php artisan migrate --force`
7. `php artisan storage:link`
8. `php artisan config:cache && php artisan route:cache && php artisan view:cache`
9. Pastikan permission `storage/` dan `bootstrap/cache/`
10. Aktifkan HTTPS

**Jangan commit file `.env`.**

## Struktur Singkat

```
app/Http/Controllers/...
app/Http/Requests/...
app/Services/...
app/Models/...
app/Mail/...
app/Policies/...
resources/views/...
routes/web.php
routes/api.php
```

## Screenshot

<!-- Tambahkan screenshot dashboard & list task di sini -->

## Lisensi

MIT (atau sesuai pilihanmu)

## Catatan Pembelajaran

Project ini mengikuti jalur modul Task Manager (setup → git → CRUD → auth → dashboard → upload → relasi → policy → refactor → API → test → mail → deploy → capstone).
````

Paste screenshot nyata ke README — recruiter sering scroll visual dulu.

---

## Demo Script (5–10 Menit)

Gunakan script ini saat presentasi magang, kelas, atau interview teknis ringan.

### Persiapan (sebelum presentasi)

1. Browser sudah login siap (atau siap register cepat)
2. Database ter-seed
3. Mailtrap tab terbuka (opsional)
4. Postman/Insomnia siap untuk 30 detik API demo
5. Mode desktop + satu kali cek mobile

### Menit 0:00–1:00 — Opening

> "Ini Task Manager yang saya bangun dengan Laravel dan MySQL. UI memakai Bootstrap CDN tanpa kompleksitas frontend build. Saya akan demo alur utama dari register sampai email dan API."

Tunjukkan struktur singkat: web auth, CRUD, dashboard, API.

### Menit 1:00–3:00 — Auth & Isolasi Data

1. Logout jika perlu
2. Register user baru **atau** login demo
3. Tunjukkan navbar `@auth`
4. Buat 1 task
5. (Opsional cepat) sebut: user lain tidak bisa lihat task ini — buktikan jika waktu cukup

### Menit 3:00–5:30 — Fitur Inti

1. Dashboard: angka statistik
2. Kategori: assign kategori ke task
3. Search / filter
4. Edit task + flash success
5. Upload lampiran (buka file/gambar)
6. Tag / soft delete (sebut singkat)
7. (Jika admin) satu aksi khusus admin **atau** Policy: user tidak bisa edit task orang lain → 403

### Menit 5:30–7:00 — Quality & Email

1. Jalankan `php artisan test` (atau tunjukkan hasil hijau di terminal yang sudah dijalankan)
2. Buat task baru → buka Mailtrap → tunjukkan email `TaskCreated`

### Menit 7:00–8:30 — API

1. Ambil token Sanctum
2. `GET /api/tasks` → JSON
3. `POST /api/tasks` → data muncul di web juga

### Menit 8:30–10:00 — Deploy & Closing

1. Buka URL production (jika ada) **atau** jelaskan checklist deploy + screenshot
2. Sebut README, `.env` tidak di-commit, `APP_DEBUG=false`
3. Closing:

> "Project ini mencakup full cycle: fitur, kualitas (test), integrasi email, dan deploy. Next step saya: [isi: concurrency, CI, Redis queue, dll.]."

### Tips demo

- Jangan debug live terlalu lama — punya backup tab yang sudah login
- Jika internet CDN lambat, bilang "Bootstrap dari CDN" dan lanjut
- Satu bug kecil? Acknowledge, jangan panik: "ini edge case, saya catat"

---

## Self-Assessment Rubric

Nilai diri 1–5 per kategori. Jujur — ini untuk perbaikan, bukan pamer.

| Kategori | 1 | 3 | 5 | Nilaimu |
|---|---|---|---|---|
| **CRUD & Validasi** | Sering error, validasi lemah | CRUD jalan, validasi dasar | Solid, UX form rapi, edge case terpikir | |
| **Auth & Authorization** | Hanya login | Login + filter user | Policy/role jelas, 403 benar | |
| **Arsitektur** | Semua di controller gemuk | Ada Form Request | Form Request + Service rapi | |
| **Database & Relasi** | Satu tabel saja | belongsTo/hasMany | M2M/tag, soft delete paham | |
| **API** | Belum ada | CRUD JSON | Sanctum + konsisten error JSON | |
| **Testing** | Tidak ada test | Beberapa feature test | Coverage alur kritis hijau | |
| **Email / Integrasi** | Belum | Kirim sukses di Mailtrap | Template rapi + error handling sadar | |
| **Deploy & Ops** | Hanya local | Checklist tertulis | Live / pernah deploy sukses | |
| **UI/UX Bootstrap** | Berantakan | Cukup rapi | Empty state, 404, confirm, responsif | |
| **Dokumentasi & Git** | README minim | README bisa diikuti | README portfolio-grade + commit jelas | |

**Interpretasi skor total (max 50):**

| Skor | Artinya |
|---|---|
| 40–50 | Siap masuk portfolio kuat; siap cerita di interview |
| 30–39 | Bagus — perkuat 2 kategori terlemah minggu ini |
| 20–29 | Fondasi ada — ulangi modul yang nilainya ≤2 |
| <20 | Fokus selesaikan checklist fitur dulu sebelum polish |

Tulis 3 tindakan perbaikan konkret:

1. ...
2. ...
3. ...

---

## Career Next Steps

### Portfolio

1. Repo publik dengan README final (template di atas)
2. URL live (meski subdomain murah)
3. 3–5 screenshot di README
4. Pin repo di GitHub profile
5. Tulis case study singkat (1 halaman): masalah → solusi → tech → hasil

### Topik Interview yang Siap Kamu Ceritakan

Latih jawaban 1–2 menit untuk:

| Topik | Poin yang bisa kamu sebut dari project |
|---|---|
| MVC Laravel | Route → Controller → Service → Model → Blade |
| Eloquent & relasi | User–Task, Task–Category, Tag M2M |
| Auth | Session vs token Sanctum |
| Authorization | Policy vs middleware role |
| Validasi | Form Request vs validate di controller |
| Security | CSRF, hash password, `APP_DEBUG`, `.env`, mass assignment |
| Testing | Feature test HTTP, assert database |
| Email | Mailable, SMTP, mengapa Mailtrap di dev |
| Deploy | `public/` document root, config cache, migrate `--force` |
| Trade-off | Mengapa Bootstrap CDN, bukan SPA |

### Skill Lanjutan (setelah capstone)

Urutan saran:

1. CI GitHub Actions (`php artisan test` otomatis)
2. Queue + Supervisor untuk email production
3. Redis cache/session
4. Docker sederhana untuk onboarding tim
5. Filament / admin panel (opsional) — setelah paham manual
6. TypeScript + API Laravel sebagai backend terpisah

### Soft skill presentasi

- Rekam demo 5 menit → upload unlisted YouTube → link di README
- Minta feedback teman: "apakah README bisa diikuti tanpa tanya?"

---

## Latihan Modul 17

### Latihan 1: Audit Checklist

Cetak / salin checklist fitur 01–16. Tandai ❌ yang belum. Perbaiki minimal 3 item ❌ sebelum commit final.

### Latihan 2: Polish Sprint (2 jam)

Wajib selesai:

- [ ] 404 page
- [ ] Empty state list task
- [ ] Confirm hapus
- [ ] README final terisi lengkap

### Latihan 3: Dry-run Demo

Timer 8 menit. Rekam layar. Tonton ulang — catat bagian yang gagap / terlalu lama.

### Latihan 4: Peer Review

Minta teman clone repo hanya dari README (tanpa bantuan chat). Catat di mana mereka stuck — perbaiki dokumentasi.

### Latihan 5: Rubrik + Rencana 7 Hari

Isi rubrik. Pilih 1 kategori terlemah. Buat rencana 7 hari (30–60 menit/hari) untuk menaikkan skor kategori itu.

---

## Checklist Modul 17

### Review

- [ ] Checklist fitur 01–16 sudah diaudit
- [ ] Bug blocker untuk demo sudah ditutup
- [ ] Seeder demo siap

### Polish

- [ ] Halaman 404
- [ ] Empty states
- [ ] Confirm dialog delete
- [ ] Flash message & validasi konsisten
- [ ] Cek tampilan mobile dasar

### Dokumentasi & Demo

- [ ] README.md project final (template lengkap)
- [ ] Screenshot / URL live (jika ada)
- [ ] Demo script sudah dilatih sekali
- [ ] Self-assessment rubrik terisi

### Git

- [ ] Commit: `docs: capstone review dan readme final`
- [ ] Push ke `main`

---

## Git — Commit Final & Push

```bash
git add .
git commit -m "docs: capstone review dan readme final"
git push
```

Ini commit penutup jalur pembelajaran Task Manager di panduan ini.

Tetap pada workflow sederhana:

```text
git add .
git commit -m "..."
git push
```

Tidak perlu Pull Request, merge, atau branch feature untuk mengikuti panduan ini.

---

## Ringkasan Arsitektur Akhir (Gambaran)

```text
Browser (Bootstrap CDN)
    ↓
routes/web.php  ──→ Controllers ──→ Form Requests
                       ↓
                   TaskService / Policies
                       ↓
                   Eloquent Models  ←→  MySQL
                       ↓
                   Mail / Notifications
                       ↓
                   SMTP (Mailtrap / production)

API clients
    ↓
routes/api.php ──→ Sanctum ──→ API Controllers ──→ JSON
```

Kamu tidak harus hafal semua file — yang penting paham **alur request** dan bisa menjelaskan trade-off.

---

## Apa yang Sudah Kamu Kuasai (Capstone)

- Membangun aplikasi Laravel end-to-end
- Autentikasi & autorisasi praktis
- Desain fitur bertahap (incremental delivery)
- API + testing + email sebagai pelengkap production-ready
- Deploy & kesadaran keamanan dasar
- Dokumentasi project untuk manusia lain (dan recruiter)

---

## Selamat — Kamu Selesai!

Task Manager bukan lagi "ikut tutorial": itu **project milikmu**.

Langkah berikutnya bukan menambah library demi gengsi, melainkan:

1. Pakai app-nya sendiri seminggu
2. Catat gesekan UX
3. Perbaiki 2–3 hal yang paling mengganggu
4. Masukkan ke portfolio + cerita di interview

Terima kasih sudah mengikuti seluruh modul.

---

**Modul sebelumnya:** [16 — Deploy](../16-deploy/README.md)  
**Kembali ke:** [Panduan Utama](../README.md)
