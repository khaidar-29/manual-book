# Modul 02 — Git & GitHub

Memahami Git dan menghubungkan project **Task Manager** ke GitHub.

**Estimasi waktu:** 1 hari  
**Prasyarat:** [Modul 01 — Setup](../01-setup/README.md)

---

## Tujuan Modul

- [ ] Paham konsep Git (commit, branch, push)
- [ ] Repo GitHub dibuat & terhubung
- [ ] Project di-push ke GitHub
- [ ] Paham workflow commit per modul

---

## Konsep Git

```
Working Directory  →  git add  →  Staging  →  git commit  →  Local Repo
                                                                  ↓
                                                            git push
                                                                  ↓
                                                            GitHub (Remote)
```

| Istilah | Arti |
|---|---|
| **Repository (repo)** | Folder project + riwayat perubahan |
| **Commit** | Snapshot perubahan dengan pesan |
| **Branch** | Cabang kerja terpisah |
| **Push** | Kirim commit ke GitHub |
| **Pull** | Ambil update dari GitHub |

---

## Langkah 1: Konfigurasi Git (sekali saja)

```bash
git config --global user.name "Nama Kamu"
git config --global user.email "email@example.com"
```

Pastikan email sama dengan akun GitHub.

---

## Langkah 2: Buat Repo di GitHub

1. Login ke [github.com](https://github.com)
2. Klik **New repository**
3. Nama: `task-manager`
4. Visibility: **Public** atau **Private**
5. **Jangan** centang "Add README" (project lokal sudah ada)
6. Klik **Create repository**

---

## Langkah 3: Hubungkan & Push

Di folder project `task-manager`:

```bash
git remote add origin https://github.com/USERNAME/task-manager.git
git branch -M main
git push -u origin main
```

Ganti `USERNAME` dengan username GitHub kamu.

Jika diminta login, gunakan **Personal Access Token** sebagai password (bukan password akun).

### Buat Personal Access Token

1. GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Centang scope `repo`
4. Copy token, simpan — dipakai saat push

---

## Langkah 4: Verifikasi

Refresh halaman repo di GitHub — semua file project harus sudah tampil.

```bash
git remote -v        # Cek remote URL
git log --oneline    # Lihat riwayat commit
git status           # Harus "nothing to commit, working tree clean"
```

---

## Langkah 5: Workflow Branch

Setiap modul baru, kerjakan di branch terpisah:

```bash
# Buat branch untuk modul 03
git checkout main
git pull origin main
git checkout -b modul/03-routing-controller

# ... coding ...

git add .
git commit -m "feat: tambah route dan controller halaman tentang"
git push -u origin modul/03-routing-controller
```

Setelah selesai, buat **Pull Request** di GitHub → merge ke `main`.

> Untuk pembelajaran solo, boleh push langsung ke `main`. Tapi latihan pakai branch + PR sangat disarankan.

---

## Konvensi Commit Message

Format: `tipe: deskripsi singkat`

| Tipe | Kapan dipakai | Contoh |
|---|---|---|
| `feat` | Fitur baru | `feat: tambah halaman tentang` |
| `fix` | Perbaikan bug | `fix: perbaiki validasi form task` |
| `docs` | Dokumentasi | `docs: update readme` |
| `chore` | Setup/config | `chore: initial laravel setup` |
| `refactor` | Ubah struktur kode | `refactor: pisah logic ke service` |

---

## Perintah Git yang Sering Dipakai

```bash
git status                  # Lihat file berubah
git diff                    # Lihat detail perubahan
git add .                   # Stage semua perubahan
git add app/Models/Task.php # Stage file tertentu
git commit -m "pesan"       # Commit
git push                    # Push ke GitHub
git pull                    # Ambil update dari GitHub
git log --oneline -10       # 10 commit terakhir
git checkout -b nama-branch # Buat & pindah branch
git checkout main           # Pindah ke branch main
git stash                   # Simpan perubahan sementara
git stash pop               # Kembalikan perubahan
```

---

## Latihan Modul 02

1. Buat repo GitHub `task-manager`
2. Push project dari Modul 01 ke GitHub
3. Buat branch `modul/02-git-github`
4. Buat file `README.md` di root project dengan isi:

```markdown
# Task Manager

Aplikasi manajemen task — project belajar Laravel.

## Tech Stack
- Laravel 12
- Blade + Tailwind CSS
- SQLite / MySQL
```

5. Commit & push:

```bash
git add README.md
git commit -m "docs: tambah readme project"
git push -u origin modul/02-git-github
```

6. Buat Pull Request di GitHub, merge ke `main`
7. Di local: `git checkout main && git pull origin main`

---

## Template Git di Setiap Modul Berikutnya

Di modul 03–08, ulangi pola ini di akhir setiap modul:

```bash
git checkout main
git pull origin main
git checkout -b modul/XX-nama-modul

# ... kerjakan latihan modul ...

git add .
git commit -m "feat: deskripsi perubahan modul ini"
git push -u origin modul/XX-nama-modul

# Buat PR di GitHub → merge → pull main
git checkout main && git pull origin main
```

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| `remote origin already exists` | `git remote set-url origin URL_BARU` |
| Push ditolak (rejected) | `git pull origin main` dulu, lalu push lagi |
| `could not read Username` | Pakai Personal Access Token, bukan password |
| Commit ke branch salah | `git checkout main` lalu buat branch baru |
| `.env` ikut ke-commit | `git rm --cached .env` — pastikan ada di `.gitignore` |

---

## Checklist Selesai

- [ ] Git config (name & email) sudah di-set
- [ ] Repo GitHub `task-manager` dibuat
- [ ] Project ter-push ke GitHub
- [ ] Bisa buat branch, commit, push
- [ ] Paham konvensi commit message
- [ ] (Opsional) Pull Request pertama berhasil di-merge

---

**Modul sebelumnya:** [01 — Setup](../01-setup/README.md)  
**Modul berikutnya:** [03 — Routing & Controller](../03-routing-controller/README.md)
