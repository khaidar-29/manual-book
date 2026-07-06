# Modul 02 — Git & GitHub

Modul kedua: memahami Git dan menghubungkan project **Task Manager** ke GitHub.

**Estimasi waktu:** 1 hari  
**Prasyarat:** [Modul 01 — Setup](../01-setup/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Memahami konsep Git (working directory, staging, commit, push)
- [ ] Mengkonfigurasi identitas Git
- [ ] Membuat repository di GitHub
- [ ] Menghubungkan project lokal ke GitHub
- [ ] Push perubahan ke GitHub
- [ ] Memahami workflow `add → commit → push` yang dipakai setiap modul

---

## Mengapa Git Penting?

Git mencatat **setiap perubahan** kode kamu. Manfaatnya:

- Bisa kembali ke versi sebelumnya jika ada kesalahan
- Riwayat kerja terdokumentasi otomatis
- Kode bisa di-backup ke GitHub (cloud)
- Kolaborasi tim jadi lebih mudah

---

## Konsep Git

```
┌─────────────────┐     git add      ┌─────────────┐     git commit     ┌─────────────┐
│ Working         │  ──────────────► │  Staging    │  ────────────────► │  Local Repo │
│ Directory       │                  │  Area       │                    │  (.git)     │
│ (file kamu)     │                  │  (siap commit)                   │             │
└─────────────────┘                  └─────────────┘                    └──────┬──────┘
                                                                                  │
                                                                             git push
                                                                                  │
                                                                                  ▼
                                                                         ┌─────────────┐
                                                                         │  GitHub     │
                                                                         │  (remote)   │
                                                                         └─────────────┘
```

### Istilah penting

| Istilah | Arti |
|---|---|
| **Repository (repo)** | Folder project + seluruh riwayat perubahan |
| **Commit** | Snapshot (foto) kode pada satu titik waktu + pesan |
| **Push** | Kirim commit dari komputer ke GitHub |
| **Pull** | Ambil update terbaru dari GitHub |
| **Remote** | Versi repo di server (GitHub) |
| **Origin** | Nama default remote (GitHub) |

---

## Langkah 1: Konfigurasi Git (Sekali Saja)

Set identitas kamu — dipakai di setiap commit:

```bash
git config --global user.name "Nama Kamu"
git config --global user.email "email@example.com"
```

Verifikasi:

```bash
git config --global user.name
git config --global user.email
```

> Email sebaiknya sama dengan email akun GitHub.

---

## Langkah 2: Buat Repository di GitHub

1. Login ke [github.com](https://github.com)
2. Klik tombol **+** → **New repository**
3. Isi form:
   - **Repository name:** `task-manager`
   - **Description:** `Aplikasi manajemen task — belajar Laravel`
   - **Visibility:** Public atau Private
   - **Jangan centang** "Add a README file" (project lokal sudah ada)
4. Klik **Create repository**

GitHub akan menampilkan instruksi push — kita kerjakan di langkah berikutnya.

---

## Langkah 3: Hubungkan Project ke GitHub

Masuk ke folder project Laravel:

```bash
cd ~/projects/task-manager
```

Hubungkan ke remote GitHub (ganti `USERNAME`):

```bash
git remote add origin https://github.com/USERNAME/task-manager.git
git branch -M main
git push -u origin main
```

### Jika diminta login GitHub

GitHub **tidak menerima password akun** untuk push via HTTPS. Pakai **Personal Access Token (PAT)** sebagai password.

**Cara buat PAT:**
1. GitHub → **Settings** → **Developer settings**
2. **Personal access tokens** → **Tokens (classic)**
3. **Generate new token (classic)**
4. Centang scope **`repo`**
5. Generate → **copy token** (hanya muncul sekali!)
6. Saat `git push` diminta password → paste token

---

## Langkah 4: Verifikasi Push Berhasil

1. Refresh halaman repo di GitHub
2. Semua file project harus sudah tampil
3. Di terminal:

```bash
git remote -v
git log --oneline
git status
```

`git status` harus: `nothing to commit, working tree clean`

---

## Langkah 5: Workflow Git — Setiap Modul

**Ini workflow yang dipakai di modul 03–08.** Tidak perlu buat branch atau merge Pull Request.

```bash
# 1. Kerjakan coding sesuai modul...

# 2. Cek perubahan
git status
git diff

# 3. Stage semua perubahan
git add .

# 4. Commit dengan pesan jelas
git commit -m "feat: tambah halaman tentang"

# 5. Push ke GitHub
git push
```

Selesai. Langsung push ke branch `main`.

---

## Konvensi Commit Message

Format: **`tipe: deskripsi singkat`**

| Tipe | Kapan dipakai | Contoh |
|---|---|---|
| `feat` | Fitur baru | `feat: tambah halaman tentang` |
| `fix` | Perbaikan bug | `fix: perbaiki validasi form task` |
| `docs` | Dokumentasi | `docs: tambah readme project` |
| `chore` | Setup/konfigurasi | `chore: initial laravel setup` |
| `refactor` | Ubah struktur tanpa ubah fungsi | `refactor: pisahkan logic ke controller` |

**Tips menulis commit message:**
- Pakai bahasa Indonesia atau Inggris — konsisten saja
- Deskripsikan **apa** yang berubah, bukan **bagaimana**
- Contoh buruk: `update file`
- Contoh bagus: `feat: tambah migration tabel tasks`

---

## Perintah Git yang Sering Dipakai

```bash
git status                    # File apa saja yang berubah?
git diff                      # Detail perubahan baris per baris
git add .                     # Stage semua perubahan
git add app/Models/Task.php   # Stage satu file saja
git commit -m "pesan"         # Simpan snapshot
git push                      # Kirim ke GitHub
git pull                      # Ambil update dari GitHub
git log --oneline             # Riwayat commit (ringkas)
git log --oneline -10         # 10 commit terakhir
```

---

## Latihan Modul 02

### Latihan 1: Push project Modul 01

Jika belum push di Modul 01:

```bash
git add .
git commit -m "chore: initial laravel setup"
git push -u origin main
```

### Latihan 2: Tambah README project

Buat file `README.md` di root project `task-manager/`:

```markdown
# Task Manager

Aplikasi manajemen task — project belajar Laravel.

## Fitur (Progress)
- [x] Setup Laravel + MySQL
- [ ] Routing & Controller
- [ ] Blade + Bootstrap
- [ ] Database & CRUD
- [ ] Authentication
- [ ] Dashboard & Kategori

## Tech Stack
- Laravel 12
- MySQL 8
- Blade + Bootstrap 5 (CDN)

## Cara Menjalankan
1. Clone repo
2. `composer install`
3. Copy `.env.example` ke `.env`, sesuaikan database
4. `php artisan key:generate`
5. `php artisan migrate`
6. `php artisan serve`
```

Commit & push:

```bash
git add README.md
git commit -m "docs: tambah readme project task manager"
git push
```

### Latihan 3: Lihat riwayat di GitHub

1. Buka repo di GitHub
2. Klik tab **Commits**
3. Pastikan 2 commit tampil: initial setup + readme

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| `remote origin already exists` | `git remote set-url origin URL_BARU` |
| `failed to push — rejected` | `git pull origin main` dulu, lalu `git push` lagi |
| `could not read Username` | Pakai Personal Access Token sebagai password |
| `.env` ikut ke-commit | Pastikan `.env` ada di `.gitignore`. Jalankan `git rm --cached .env` |
| `git push` minta login terus | Simpan credential: `git config --global credential.helper store` |

---

## Checklist Selesai

- [ ] Git config (name & email) sudah di-set
- [ ] Repo GitHub `task-manager` dibuat
- [ ] Remote origin terhubung
- [ ] Project ter-push ke GitHub
- [ ] README project ditambahkan dan di-push
- [ ] Paham workflow: `git add .` → `commit` → `push`
- [ ] Paham konvensi commit message

---

## Git — Commit & Push Modul 02

```bash
git add .
git commit -m "docs: setup git github dan tambah readme project"
git push
```

---

**Modul sebelumnya:** [01 — Setup](../01-setup/README.md)  
**Modul berikutnya:** [03 — Routing & Controller](../03-routing-controller/README.md)
