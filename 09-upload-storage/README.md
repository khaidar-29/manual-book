# Modul 09 — Upload & Storage

Modul kesembilan: menambahkan **upload lampiran** ke task di **Task Manager** — gambar atau PDF disimpan di disk `public`, ditampilkan di show/index, dan dibersihkan saat update/hapus.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 08 — Dashboard & Kategori](../08-project-akhir/README.md)

> Project ini masih **satu repo yang sama**: `task-manager`.  
> Stack tetap: **MySQL only**, **Bootstrap 5 CDN only** (tanpa npm, Vite, Tailwind, Laravel UI).  
> Auth manual dari Modul 07 sudah ada — jangan install Breeze/UI.

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Menjelaskan mengapa file upload penting di aplikasi nyata
- [ ] Menambah kolom `attachment` (nullable) ke tabel `tasks` via migration
- [ ] Membuat form create/edit dengan `enctype="multipart/form-data"`
- [ ] Validasi file: image/pdf, ukuran maksimal
- [ ] Menyimpan file dengan `Storage::disk('public')->store()`
- [ ] Membuat symlink `php artisan storage:link`
- [ ] Menampilkan link/preview lampiran di show & index
- [ ] Menghapus file lama saat update/destroy
- [ ] Menerapkan praktik keamanan upload (jangan percaya nama file dari client)
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
Migration: add_attachment_to_tasks_table
Kolom tasks.attachment (nullable string)
Update Task::$fillable
Update TaskController store/update/destroy
Update views tasks/create, edit, show, index
Symlink: public/storage → storage/app/public
```

---

## Mengapa File Upload Penting?

Di aplikasi nyata, user jarang hanya mengisi teks. Mereka sering butuh:

| Use case | Contoh |
|---|---|
| Bukti / dokumen | PDF invoice, surat tugas |
| Screenshot | Bug report dengan gambar |
| Lampiran referensi | Design mockup, foto barang |
| Identitas | KTP, foto profil |

Tanpa upload, Task Manager hanya menyimpan judul dan status. Dengan lampiran, task jadi **lebih kontekstual** — misalnya task "Perbaiki bug login" bisa dilampirkan screenshot error.

### Alur upload di Laravel (konsep)

```
1. User pilih file di form (input type="file")
2. Browser kirim multipart/form-data ke server
3. Controller validasi tipe & ukuran
4. Laravel simpan file ke storage/app/public/...
5. Path relatif disimpan di kolom database (bukan isi file-nya)
6. User akses file lewat URL /storage/... (via symlink)
```

**Penting:** Database menyimpan **path string**, bukan binary file. File fisik ada di disk.

---

## Konsep: Disk Storage

Laravel punya facade `Storage` dengan beberapa **disk** di `config/filesystems.php`:

| Disk | Lokasi fisik | Publik? |
|---|---|---|
| `local` | `storage/app/private` (Laravel 11+) / `storage/app` | Tidak |
| `public` | `storage/app/public` | Ya (setelah `storage:link`) |
| `s3` | Amazon S3 / object storage | Tergantung config |

Untuk modul ini kita pakai **`public`** — file bisa diakses browser tanpa auth khusus di level filesystem (proteksi tetap di route/controller jika perlu).

```php
use Illuminate\Support\Facades\Storage;

// Simpan file, return path relatif misal: "attachments/abc123.pdf"
$path = $request->file('attachment')->store('attachments', 'public');

// Hapus file
Storage::disk('public')->delete($path);

// Cek ada tidak
Storage::disk('public')->exists($path);

// URL publik (butuh symlink)
$url = Storage::disk('public')->url($path);
// atau: asset('storage/' . $path)
```

### Mengapa `store()` bukan nama asli file?

```php
// ❌ BERBAHAYA — percaya nama dari client
$filename = $request->file('attachment')->getClientOriginalName();
$request->file('attachment')->move(public_path('uploads'), $filename);
// Risiko: path traversal, overwrite file, nama aneh, XSS di filename

// ✅ AMAN — Laravel generate nama unik hash
$path = $request->file('attachment')->store('attachments', 'public');
// Hasil: attachments/XyZ9abCdEfGhIjKlMnOp.pdf
```

`store()` otomatis:
- Generate nama unik (hindari bentrok & overwrite)
- Ambil ekstensi dari tipe file yang terdeteksi
- Simpan di folder yang kamu tentukan

---

## Konsep: `enctype="multipart/form-data"`

Form HTML default mengirim data sebagai `application/x-www-form-urlencoded` — **tidak bisa** membawa file binary.

Wajib tambahkan:

```blade
<form action="..." method="POST" enctype="multipart/form-data">
```

Tanpa ini, `$request->file('attachment')` akan selalu `null`.

---

## Konsep: `storage:link`

File disimpan di `storage/app/public/attachments/...`  
Browser **tidak** boleh akses folder `storage/` secara langsung (di luar `public/`).

Solusi Laravel: **symlink**

```bash
php artisan storage:link
```

Ini membuat:

```
public/storage  →  storage/app/public
```

Sehingga URL:

```
http://localhost:8000/storage/attachments/xyz.pdf
```

mengarah ke file fisik:

```
storage/app/public/attachments/xyz.pdf
```

Jalankan **sekali** per environment (lokal & production). Di production sering lupa — gejala: gambar 404.

---

## Langkah 1: Migration Kolom `attachment`

Pastikan kamu berada di folder project Task Manager:

```bash
cd task-manager
php artisan make:migration add_attachment_to_tasks_table --table=tasks
```

Edit file migration yang baru dibuat di `database/migrations/`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->string('attachment')->nullable()->after('category_id');
        });
    }

    public function down(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->dropColumn('attachment');
        });
    }
};
```

**Penjelasan:**

| Bagian | Arti |
|---|---|
| `string('attachment')` | Path relatif file, max 255 karakter |
| `nullable()` | Task boleh tanpa lampiran |
| `after('category_id')` | Posisi kolom (opsional, kerapian MySQL) |

Jalankan migration (**MySQL**, bukan SQLite):

```bash
php artisan migrate
```

Cek di MySQL:

```sql
DESCRIBE tasks;
-- harus ada kolom attachment | varchar(255) | YES | NULL
```

---

## Langkah 2: Update Model Task

Edit `app/Models/Task.php` — tambahkan `attachment` ke `$fillable`:

```php
protected $fillable = [
    'title',
    'description',
    'is_done',
    'user_id',
    'category_id',
    'attachment',
];
```

Tanpa ini, mass assignment `Task::create([...])` / `$task->update([...])` akan mengabaikan atau error (tergantung config) pada field `attachment`.

### Helper opsional di model (disarankan)

```php
use Illuminate\Support\Facades\Storage;

// di dalam class Task:

public function attachmentUrl(): ?string
{
    if (!$this->attachment) {
        return null;
    }

    return asset('storage/' . $this->attachment);
}

public function isImageAttachment(): bool
{
    if (!$this->attachment) {
        return false;
    }

    $ext = strtolower(pathinfo($this->attachment, PATHINFO_EXTENSION));

    return in_array($ext, ['jpg', 'jpeg', 'png', 'gif', 'webp'], true);
}
```

Ini memudahkan Blade menampilkan preview vs link download.

---

## Langkah 3: Symlink Storage

```bash
php artisan storage:link
```

Output sukses mirip:

```
The [public/storage] link has been connected to [storage/app/public].
```

Verifikasi:

```bash
ls -la public/storage
# harus symlink ke ../storage/app/public
```

Jika sudah pernah di-link dan error "already exists", biasanya aman diabaikan — atau hapus symlink lama lalu buat ulang:

```bash
rm public/storage
php artisan storage:link
```

---

## Langkah 4: Validasi Upload

Aturan validasi yang dipakai di `store` dan `update`:

```php
'attachment' => [
    'nullable',
    'file',
    'mimes:jpg,jpeg,png,gif,webp,pdf',
    'max:2048', // kilobyte → 2 MB
],
```

**Penjelasan rule:**

| Rule | Arti |
|---|---|
| `nullable` | Boleh kosong |
| `file` | Harus file upload valid |
| `mimes:...` | Ekstensi yang diizinkan (dicek dari isi file, bukan hanya nama) |
| `max:2048` | Maksimal **2048 KB** = 2 MB |

> Catatan: `max` untuk file dihitung dalam **kilobyte**, bukan byte.

Alternatif lebih ketat untuk gambar saja:

```php
'attachment' => ['nullable', 'image', 'max:2048'],
```

Rule `image` = jpeg, png, bmp, gif, svg, webp. Karena modul ini juga izinkan PDF, pakai `mimes` + `file`.

---

## Langkah 5: Update TaskController — `store`

Edit method `store` di `app/Http/Controllers/TaskController.php`:

```php
use Illuminate\Support\Facades\Storage;
use Illuminate\Http\Request;

public function store(Request $request)
{
    $validated = $request->validate([
        'title'       => ['required', 'string', 'max:255'],
        'description' => ['nullable', 'string'],
        'is_done'     => ['sometimes', 'boolean'],
        'category_id' => ['nullable', 'exists:categories,id'],
        'attachment'  => [
            'nullable',
            'file',
            'mimes:jpg,jpeg,png,gif,webp,pdf',
            'max:2048',
        ],
    ]);

    // Pastikan kategori milik user yang login (jika ada)
    if (!empty($validated['category_id'])) {
        $ownsCategory = auth()->user()
            ->categories()
            ->where('id', $validated['category_id'])
            ->exists();

        if (!$ownsCategory) {
            abort(403, 'Kategori tidak valid.');
        }
    }

    $validated['is_done'] = $request->boolean('is_done');
    $validated['user_id'] = auth()->id();

    if ($request->hasFile('attachment')) {
        $validated['attachment'] = $request
            ->file('attachment')
            ->store('attachments', 'public');
    }

    Task::create($validated);

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task berhasil dibuat!');
}
```

**Alur singkat:**

1. Validasi input + file  
2. Cek ownership kategori  
3. Jika ada file → `store('attachments', 'public')` → dapat path  
4. Simpan path ke DB bersama data task lain  

---

## Langkah 6: Update TaskController — `update`

Saat update, ada tiga skenario lampiran:

1. User **tidak** upload file baru → biarkan lampiran lama  
2. User upload file baru → hapus file lama, simpan yang baru  
3. (Opsional latihan) User centang "hapus lampiran" → hapus file + set null  

```php
public function update(Request $request, Task $task)
{
    $this->authorizeTask($task); // dari Modul 07/08 — ownership check

    $validated = $request->validate([
        'title'       => ['required', 'string', 'max:255'],
        'description' => ['nullable', 'string'],
        'is_done'     => ['sometimes', 'boolean'],
        'category_id' => ['nullable', 'exists:categories,id'],
        'attachment'  => [
            'nullable',
            'file',
            'mimes:jpg,jpeg,png,gif,webp,pdf',
            'max:2048',
        ],
    ]);

    if (!empty($validated['category_id'])) {
        $ownsCategory = auth()->user()
            ->categories()
            ->where('id', $validated['category_id'])
            ->exists();

        if (!$ownsCategory) {
            abort(403, 'Kategori tidak valid.');
        }
    }

    $validated['is_done'] = $request->boolean('is_done');

    if ($request->hasFile('attachment')) {
        // Hapus file lama jika ada
        if ($task->attachment) {
            Storage::disk('public')->delete($task->attachment);
        }

        $validated['attachment'] = $request
            ->file('attachment')
            ->store('attachments', 'public');
    }

    $task->update($validated);

    return redirect()
        ->route('tasks.show', $task)
        ->with('success', 'Task berhasil diupdate!');
}
```

**Mengapa hapus file lama?**  
Jika tidak dihapus, file orphan menumpuk di disk — boros storage dan berpotensi bocor data lama.

---

## Langkah 7: Update TaskController — `destroy`

Saat task dihapus, hapus juga file fisiknya:

```php
public function destroy(Task $task)
{
    $this->authorizeTask($task);

    if ($task->attachment) {
        Storage::disk('public')->delete($task->attachment);
    }

    $task->delete();

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task berhasil dihapus!');
}
```

Urutan disarankan: **hapus file dulu** (atau setelah), lalu hapus row DB. Jika delete DB gagal, file masih bisa di-cleanup manual; yang lebih parah adalah DB terhapus tapi path masih menunjuk file yang sudah hilang — untuk modul ini pola di atas sudah cukup.

---

## Langkah 8: Form Create — `enctype` + Input File

Edit `resources/views/tasks/create.blade.php`.

Pastikan tag `<form>` punya `enctype`:

```blade
<form action="{{ route('tasks.store') }}" method="POST" enctype="multipart/form-data">
    @csrf

    {{-- field title, description, category, is_done seperti Modul 06–08 --}}

    <div class="mb-3">
        <label for="attachment" class="form-label">Lampiran (opsional)</label>
        <input
            type="file"
            name="attachment"
            id="attachment"
            class="form-control @error('attachment') is-invalid @enderror"
            accept=".jpg,.jpeg,.png,.gif,.webp,.pdf,image/*,application/pdf"
        >
        <div class="form-text">Gambar atau PDF, maksimal 2 MB.</div>
        @error('attachment')
            <div class="invalid-feedback">{{ $message }}</div>
        @enderror
    </div>

    <button type="submit" class="btn btn-primary">Simpan Task</button>
</form>
```

**Catatan `accept`:**  
Atribut `accept` hanya membantu UX di browser (filter file picker). **Bukan** keamanan — validasi server (`mimes`, `max`) tetap wajib.

---

## Langkah 9: Form Edit

Edit `resources/views/tasks/edit.blade.php`:

```blade
<form action="{{ route('tasks.update', $task) }}" method="POST" enctype="multipart/form-data">
    @csrf
    @method('PUT')

    {{-- field lain tetap --}}

    <div class="mb-3">
        <label for="attachment" class="form-label">Lampiran</label>

        @if ($task->attachment)
            <div class="mb-2">
                <span class="text-muted">Lampiran saat ini:</span>
                <a href="{{ asset('storage/' . $task->attachment) }}" target="_blank" rel="noopener">
                    Lihat file
                </a>
            </div>
        @endif

        <input
            type="file"
            name="attachment"
            id="attachment"
            class="form-control @error('attachment') is-invalid @enderror"
            accept=".jpg,.jpeg,.png,.gif,.webp,.pdf,image/*,application/pdf"
        >
        <div class="form-text">Kosongkan jika tidak ingin mengganti. Maksimal 2 MB.</div>
        @error('attachment')
            <div class="invalid-feedback">{{ $message }}</div>
        @enderror
    </div>

    <button type="submit" class="btn btn-primary">Update Task</button>
</form>
```

---

## Langkah 10: Tampilkan di Show

Edit `resources/views/tasks/show.blade.php` — tambahkan section lampiran:

```blade
<div class="mb-3">
    <strong>Lampiran:</strong>

    @if ($task->attachment)
        @php
            $url = asset('storage/' . $task->attachment);
            $ext = strtolower(pathinfo($task->attachment, PATHINFO_EXTENSION));
            $isImage = in_array($ext, ['jpg', 'jpeg', 'png', 'gif', 'webp'], true);
        @endphp

        @if ($isImage)
            <div class="mt-2">
                <img
                    src="{{ $url }}"
                    alt="Lampiran {{ $task->title }}"
                    class="img-fluid rounded border"
                    style="max-height: 320px;"
                >
            </div>
        @endif

        <div class="mt-2">
            <a href="{{ $url }}" class="btn btn-sm btn-outline-secondary" target="_blank" rel="noopener">
                Buka / unduh lampiran
            </a>
            <span class="text-muted small ms-2">{{ strtoupper($ext) }}</span>
        </div>
    @else
        <span class="text-muted">Tidak ada lampiran</span>
    @endif
</div>
```

---

## Langkah 11: Tampilkan di Index (ringkas)

Di tabel/list `resources/views/tasks/index.blade.php`, tambahkan indikator lampiran:

```blade
<td>
    {{ $task->title }}
    @if ($task->attachment)
        <a href="{{ asset('storage/' . $task->attachment) }}"
           target="_blank"
           rel="noopener"
           class="ms-1"
           title="Ada lampiran">
            📎
        </a>
    @endif
</td>
```

Atau versi Bootstrap tanpa emoji (lebih rapi):

```blade
@if ($task->attachment)
    <a href="{{ asset('storage/' . $task->attachment) }}"
       class="badge text-bg-secondary text-decoration-none"
       target="_blank"
       rel="noopener">
        File
    </a>
@endif
```

---

## Langkah 12: Pastikan `.gitignore` Tidak Mengabaikan Struktur

Laravel biasanya meng-ignore isi storage kecuali `.gitignore` di dalam folder. Pastikan struktur tetap:

```
storage/app/public/
  .gitignore   ← biasanya berisi * dan !.gitignore
```

**Jangan commit file upload user** ke Git. Yang di-commit hanya kode + migration. File lampiran hidup di environment masing-masing.

Di root `.gitignore` biasanya sudah ada:

```
/storage/*.key
/public/storage
```

Symlink `public/storage` tidak perlu di-commit (dibuat ulang dengan `storage:link`).

---

## Keamanan Upload — Checklist Wajib

| Praktik | Mengapa |
|---|---|
| Validasi `mimes` / `image` di server | Client bisa tipu `accept` / Content-Type |
| Batasi `max` size | Cegah disk penuh & timeout |
| Pakai `store()` (nama hash) | Hindari path traversal & overwrite |
| Jangan simpan di `public/` langsung dengan nama asli | Sulit dikontrol; pakai disk `public` + symlink |
| Hapus file saat update/destroy | Cegah orphan & data bocor |
| Jangan tampilkan path storage mentah ke user tanpa kontrol | Cukup URL `/storage/...` |
| Pertimbangkan auth pada file sensitif | Untuk dokumen rahasia, jangan expose publik — butuh route download terproteksi (latihan lanjut) |

### Anti-pola yang sering muncul

```php
// ❌ Jangan
$name = $_FILES['attachment']['name'];
move_uploaded_file(..., public_path($name));

// ❌ Jangan percaya extension dari client saja tanpa validasi Laravel
$ext = pathinfo($request->file('attachment')->getClientOriginalName(), PATHINFO_EXTENSION);

// ✅ Pakai
$path = $request->file('attachment')->store('attachments', 'public');
```

---

## Batas Upload di PHP / Web Server (penting!)

Validasi Laravel `max:2048` **tidak akan tercapai** jika PHP menolak lebih dulu.

Cek `php.ini`:

```ini
upload_max_filesize = 2M
post_max_size = 8M
```

`post_max_size` harus **lebih besar** dari `upload_max_filesize`.

Di nginx, bisa ada:

```nginx
client_max_body_size 2M;
```

Jika terlalu kecil → error **413 Request Entity Too Large** sebelum Laravel jalan.

---

## Uji Manual End-to-End

1. Login ke Task Manager  
2. Buat task **tanpa** lampiran → sukses, kolom `attachment` null  
3. Buat task dengan **PNG < 2 MB** → file muncul di `storage/app/public/attachments/`  
4. Buka show → preview gambar tampil  
5. Buka URL `/storage/attachments/....png` → file terbuka  
6. Edit task, ganti dengan **PDF** → file lama terhapus, PDF baru tersimpan  
7. Hapus task → file fisik ikut hilang  
8. Coba upload `.exe` atau file 5 MB → validasi gagal, pesan error tampil  

---

## Latihan

### Latihan 1: Batas 1 MB + Pesan Custom

Ubah validasi menjadi `max:1024` dan tambahkan pesan custom:

```php
$request->validate([
    // ...
    'attachment' => ['nullable', 'file', 'mimes:jpg,jpeg,png,gif,webp,pdf', 'max:1024'],
], [
    'attachment.max'   => 'Ukuran lampiran maksimal 1 MB.',
    'attachment.mimes' => 'Lampiran harus berupa gambar atau PDF.',
]);
```

### Latihan 2: Checkbox Hapus Lampiran

Di form edit, tambahkan:

```blade
@if ($task->attachment)
    <div class="form-check mb-3">
        <input class="form-check-input" type="checkbox" name="remove_attachment" value="1" id="remove_attachment">
        <label class="form-check-label" for="remove_attachment">Hapus lampiran saat ini</label>
    </div>
@endif
```

Di `update()`:

```php
if ($request->boolean('remove_attachment') && $task->attachment) {
    Storage::disk('public')->delete($task->attachment);
    $validated['attachment'] = null;
}
```

Pastikan logika ini **tidak bentrok** dengan upload file baru (prioritaskan: jika ada file baru, abaikan remove — atau sebaliknya, dokumentasikan pilihanmu).

### Latihan 3: Folder per User

Simpan file di subfolder user:

```php
$dir = 'attachments/' . auth()->id();
$validated['attachment'] = $request->file('attachment')->store($dir, 'public');
```

### Latihan 4: Tampilkan Ukuran File

Di show, tampilkan ukuran:

```php
$size = Storage::disk('public')->size($task->attachment);
// format ke KB/MB di Blade
```

### Latihan 5: Validasi Dimensi Gambar

Untuk gambar, batasi dimensi:

```php
'attachment' => ['nullable', 'file', 'mimes:jpg,jpeg,png', 'max:2048', 'dimensions:max_width=2000,max_height=2000'],
```

### Latihan 6 (Bonus): Route Download Terproteksi

Buat route `GET /tasks/{task}/attachment` yang:

1. Cek ownership (`authorizeTask`)  
2. `return Storage::disk('public')->download($task->attachment)`  

Ini lebih aman untuk file sensitif daripada URL publik langsung.

### Latihan 7 (Bonus): Multiple Attachments

Desain ulang: tabel `task_attachments` (hasMany) agar satu task bisa banyak file. (Persiapan mental untuk Modul 10 tentang relasi.)

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| `$request->file()` selalu null | Form tanpa `enctype` | Tambah `enctype="multipart/form-data"` |
| Gambar 404 di browser | Belum `storage:link` | Jalankan `php artisan storage:link` |
| Symlink ada tapi 404 | Path DB salah / file tidak tersimpan | Cek `storage/app/public/attachments` & nilai kolom DB |
| Error permission denied | Folder storage tidak writable | `chmod -R 775 storage bootstrap/cache` (sesuaikan user web server) |
| **413 Request Entity Too Large** | Batas nginx/Apache/PHP | Naikkan `client_max_body_size` / `upload_max_filesize` / `post_max_size` |
| Validasi `mimes` gagal padahal PDF | File corrupt / ekstensi palsu | Pastikan file asli valid; cek `fileinfo` PHP extension aktif |
| `League\Flysystem\UnableToCreateDirectory` | Permission / path disk salah | Cek `config/filesystems.php` disk `public` |
| File lama tidak hilang saat update | Lupa `Storage::delete` | Hapus file lama sebelum store baru |
| `SQLSTATE` column attachment not found | Migration belum dijalankan | `php artisan migrate` |
| Mass assignment error / field kosong | `attachment` belum di `$fillable` | Tambahkan ke model Task |
| Upload sukses di lokal, gagal di server | `public/storage` belum di-link di server | Jalankan `storage:link` di production |
| Preview PDF jadi `<img>` rusak | PDF ditampilkan sebagai image | Cek ekstensi; hanya image yang pakai `<img>` |

### Debug cepat

```bash
php artisan storage:link
ls -la public/storage
ls -la storage/app/public/attachments
php artisan tinker
>>> Storage::disk('public')->files('attachments');
```

---

## Checklist Selesai

Centang sebelum lanjut ke Modul 10:

- [ ] Migration `attachment` nullable di tabel `tasks` (MySQL)
- [ ] `attachment` ada di `$fillable` model Task
- [ ] `php artisan storage:link` sudah dijalankan
- [ ] Form create & edit memakai `enctype="multipart/form-data"`
- [ ] Validasi `mimes` + `max` di store & update
- [ ] File disimpan dengan `store(..., 'public')` (bukan nama client mentah)
- [ ] File lama dihapus saat update (jika diganti)
- [ ] File dihapus saat destroy
- [ ] Show menampilkan preview gambar / link PDF
- [ ] Index menampilkan indikator lampiran
- [ ] Upload file ilegal / terlalu besar ditolak dengan pesan jelas
- [ ] **TIDAK** memakai SQLite, npm, Vite, Tailwind, atau Laravel UI
- [ ] Commit & push dengan pesan yang ditentukan

---

## Git — Commit & Push

Di folder project `task-manager`:

```bash
git add .
git commit -m "feat: upload lampiran task"
git push
```

Jangan buat branch, jangan buat Pull Request, jangan merge. Langsung commit di branch kerja utama kamu (biasanya `main`).

---

## Ringkasan Modul 09

| Topik | Yang dipelajari |
|---|---|
| Upload | `multipart/form-data`, `hasFile`, `file()` |
| Validasi | `file`, `mimes`, `max`, `image`, `dimensions` |
| Storage | Disk `public`, `store()`, `delete()`, `url()` |
| Symlink | `php artisan storage:link` |
| Keamanan | Jangan percaya filename client; hapus orphan file |
| UI | Preview gambar, link PDF, badge di index |
| Ops | Permission, 413, `php.ini` upload limits |

---

**Modul sebelumnya:** [08 — Dashboard & Kategori](../08-project-akhir/README.md)  
**Modul berikutnya:** [10 — Relasi Lanjutan](../10-relasi-lanjutan/README.md)
