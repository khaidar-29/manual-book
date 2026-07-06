# Modul 06 — CRUD Task

Modul keenam: melengkapi **Task Manager** dengan operasi **Create, Read, Update, Delete** task — form, validasi, flash message, dan resource route.

**Estimasi waktu:** 2–3 hari  
**Prasyarat:** [Modul 05 — Database](../05-database/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Membuat resource controller dengan 7 method CRUD
- [ ] Mendaftarkan resource route dan memahami mapping URL-nya
- [ ] Membuat form create & edit dengan validasi Laravel
- [ ] Menggunakan `@csrf`, `@method`, `@error`, dan `old()`
- [ ] Menampilkan flash message sukses/error
- [ ] Menghapus data dengan konfirmasi JavaScript
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
TaskController → index, create, store, show, edit, update, destroy
resources/views/tasks/create.blade.php
resources/views/tasks/edit.blade.php
resources/views/tasks/show.blade.php
resources/views/tasks/index.blade.php   ← diupdate
Route::resource('tasks', TaskController::class)
```

---

## Apa itu CRUD?

**CRUD** = Create, Read, Update, Delete — empat operasi dasar manipulasi data.

| Operasi | HTTP Method | URL contoh | Controller method |
|---|---|---|---|
| **C**reate | POST | `/tasks` | `store()` |
| **R**ead (list) | GET | `/tasks` | `index()` |
| **R**ead (detail) | GET | `/tasks/1` | `show()` |
| **U**pdate | PUT/PATCH | `/tasks/1` | `update()` |
| **D**elete | DELETE | `/tasks/1` | `destroy()` |

Form create & edit pakai GET (tampilkan form), submit pakai POST/PUT/DELETE.

---

## Konsep: Resource Controller

Laravel punya konvensi **resource controller** — 7 method standar untuk CRUD:

```
index()   → tampilkan list
create()  → tampilkan form tambah
store()   → simpan data baru (POST)
show()    → tampilkan detail satu record
edit()    → tampilkan form edit
update()  → simpan perubahan (PUT/PATCH)
destroy() → hapus record (DELETE)
```

Buat dengan flag `--resource`:

```bash
php artisan make:controller TaskController --resource
```

---

## Konsep: Resource Route

Satu baris route mendaftarkan 7 route sekaligus:

```php
Route::resource('tasks', TaskController::class);
```

Cek dengan:

```bash
php artisan route:list --name=tasks
```

Output:

| Method | URI | Name | Action |
|---|---|---|---|
| GET | `/tasks` | tasks.index | index |
| GET | `/tasks/create` | tasks.create | create |
| POST | `/tasks` | tasks.store | store |
| GET | `/tasks/{task}` | tasks.show | show |
| GET | `/tasks/{task}/edit` | tasks.edit | edit |
| PUT/PATCH | `/tasks/{task}` | tasks.update | update |
| DELETE | `/tasks/{task}` | tasks.destroy | destroy |

**Route Model Binding:** Parameter `{task}` otomatis di-resolve ke model `Task` by id. Jika id tidak ada → 404.

---

## Konsep: CSRF Protection

**CSRF** (Cross-Site Request Forgery) = serangan memaksa user submit form tanpa sadar.

Laravel melindungi dengan **CSRF token** — wajib di setiap form POST/PUT/DELETE:

```blade
@csrf
```

Token disimpan di session. Tanpa token valid → error 419 Page Expired.

---

## Konsep: Method Spoofing

Browser HTML hanya mendukung GET dan POST. Untuk PUT dan DELETE, Laravel pakai **method spoofing**:

```blade
@method('PUT')     {{-- untuk update --}}
@method('DELETE')  {{-- untuk hapus --}}
```

Ini menambahkan hidden input `_method` yang Laravel baca saat routing.

---

## Konsep: Validasi Form

Laravel validasi di controller:

```php
$validated = $request->validate([
    'title' => 'required|string|min:3|max:255',
]);
```

Jika gagal → redirect back dengan error message. Di view tampilkan dengan `@error('title')`.

---

## Konsep: old() — Input Lama

Saat validasi gagal, user tidak perlu isi ulang form. `old()` mengambil nilai input sebelumnya:

```blade
<input value="{{ old('title') }}">
<input value="{{ old('title', $task->title) }}">  {{-- edit: fallback ke data task --}}
```

---

## Konsep: Flash Message

Pesan sekali tampil setelah redirect:

```php
return redirect()->route('tasks.index')
    ->with('success', 'Task berhasil ditambahkan!');
```

Di view (partial alert Modul 04):

```blade
@if(session('success'))
    <div class="alert alert-success">{{ session('success') }}</div>
@endif
```

Flash message hilang setelah refresh halaman berikutnya.

---

## Langkah 1: Resource Controller

Hapus atau overwrite `TaskController` lama. Buat ulang sebagai resource:

```bash
php artisan make:controller TaskController --resource
```

Edit `app/Http/Controllers/TaskController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    /**
     * GET /tasks — Tampilkan list task dengan pagination.
     */
    public function index()
    {
        $tasks = Task::latest()->paginate(10);

        return view('tasks.index', compact('tasks'));
    }

    /**
     * GET /tasks/create — Tampilkan form tambah task.
     */
    public function create()
    {
        return view('tasks.create');
    }

    /**
     * POST /tasks — Simpan task baru ke database.
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title'       => 'required|string|min:3|max:255',
            'description' => 'nullable|string|max:1000',
        ], [
            'title.required' => 'Judul task wajib diisi.',
            'title.min'      => 'Judul minimal 3 karakter.',
            'title.max'      => 'Judul maksimal 255 karakter.',
        ]);

        // Checkbox: jika tidak dicentang, tidak ada di request → false
        $validated['is_done'] = $request->boolean('is_done');

        Task::create($validated);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil ditambahkan!');
    }

    /**
     * GET /tasks/{task} — Tampilkan detail satu task.
     */
    public function show(Task $task)
    {
        return view('tasks.show', compact('task'));
    }

    /**
     * GET /tasks/{task}/edit — Tampilkan form edit task.
     */
    public function edit(Task $task)
    {
        return view('tasks.edit', compact('task'));
    }

    /**
     * PUT /tasks/{task} — Update task di database.
     */
    public function update(Request $request, Task $task)
    {
        $validated = $request->validate([
            'title'       => 'required|string|min:3|max:255',
            'description' => 'nullable|string|max:1000',
        ], [
            'title.required' => 'Judul task wajib diisi.',
            'title.min'      => 'Judul minimal 3 karakter.',
        ]);

        $validated['is_done'] = $request->boolean('is_done');

        $task->update($validated);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil diupdate!');
    }

    /**
     * DELETE /tasks/{task} — Hapus task dari database.
     */
    public function destroy(Task $task)
    {
        $task->delete();

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil dihapus!');
    }
}
```

**Penjelasan penting:**

| Baris | Penjelasan |
|---|---|
| `$request->validate([...])` | Validasi input, redirect back jika gagal |
| `$request->boolean('is_done')` | Checkbox: ada = true, tidak ada = false |
| `Task::create($validated)` | Insert ke DB (butuh `$fillable` di model) |
| `$task->update($validated)` | Update record yang sudah ada |
| `$task->delete()` | Hapus record |
| `->with('success', '...')` | Flash message ke session |
| `Task $task` | Route model binding — Laravel cari task by id |

---

## Langkah 2: Resource Route

Edit `routes/web.php`:

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Route;

Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');

// Ganti route GET /tasks tunggal dengan resource route
Route::resource('tasks', TaskController::class);
```

Verifikasi:

```bash
php artisan route:list --name=tasks
```

Harus muncul 7 route (index, create, store, show, edit, update, destroy).

---

## Langkah 3: Update View Index

Overwrite `resources/views/tasks/index.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Daftar Task')

@section('content')
    <div class="d-flex justify-content-between align-items-center mb-4">
        <div>
            <h1 class="h2 fw-bold mb-1">Daftar Task</h1>
            <p class="text-muted mb-0 small">Kelola semua task kamu</p>
        </div>
        <a href="{{ route('tasks.create') }}" class="btn btn-primary">
            + Tambah Task
        </a>
    </div>

    @if($tasks->isEmpty())
        <div class="card shadow-sm">
            <div class="card-body text-center py-5">
                <p class="text-muted mb-3">Belum ada task.</p>
                <a href="{{ route('tasks.create') }}" class="btn btn-primary btn-sm">
                    Tambah Task Pertama
                </a>
            </div>
        </div>
    @else
        <div class="table-responsive">
            <table class="table table-striped table-hover bg-white shadow-sm">
                <thead class="table-light">
                    <tr>
                        <th style="width: 50px;">No</th>
                        <th>Judul</th>
                        <th style="width: 120px;">Status</th>
                        <th style="width: 180px;">Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach($tasks as $index => $task)
                        <tr>
                            <td>{{ $tasks->firstItem() + $index }}</td>
                            <td>
                                <a href="{{ route('tasks.show', $task) }}"
                                   class="text-decoration-none fw-semibold {{ $task->is_done ? 'text-muted text-decoration-line-through' : '' }}">
                                    {{ $task->title }}
                                </a>
                                @if($task->description)
                                    <br>
                                    <small class="text-muted">
                                        {{ Str::limit($task->description, 60) }}
                                    </small>
                                @endif
                            </td>
                            <td>
                                <span class="badge {{ $task->is_done ? 'bg-success' : 'bg-warning text-dark' }}">
                                    {{ $task->is_done ? 'Selesai' : 'Belum' }}
                                </span>
                            </td>
                            <td>
                                <a href="{{ route('tasks.edit', $task) }}"
                                   class="btn btn-warning btn-sm">Edit</a>

                                <form action="{{ route('tasks.destroy', $task) }}"
                                      method="POST"
                                      class="d-inline"
                                      onsubmit="return confirm('Yakin hapus task ini?')">
                                    @csrf
                                    @method('DELETE')
                                    <button type="submit" class="btn btn-danger btn-sm">Hapus</button>
                                </form>
                            </td>
                        </tr>
                    @endforeach
                </tbody>
            </table>
        </div>

        {{-- Pagination --}}
        <div class="mt-3 d-flex justify-content-center">
            {{ $tasks->links() }}
        </div>
    @endif
@endsection
```

**Penjelasan Bootstrap:**

| Class | Fungsi |
|---|---|
| `table table-striped table-hover` | Tabel bergaris, highlight baris saat hover |
| `table-responsive` | Scroll horizontal di mobile |
| `btn btn-primary` | Tombol biru "Tambah" |
| `btn btn-warning btn-sm` | Tombol kuning kecil "Edit" |
| `btn btn-danger btn-sm` | Tombol merah kecil "Hapus" |
| `badge bg-success` | Label status hijau |
| `d-inline` | Form hapus sejajar dengan tombol edit |

**Penjelasan Blade:**

| Sintaks | Fungsi |
|---|---|
| `{{ $tasks->firstItem() + $index }}` | Nomor urut dengan pagination |
| `{{ Str::limit($task->description, 60) }}` | Potong deskripsi max 60 karakter |
| `@csrf` + `@method('DELETE')` | Wajib di form hapus |
| `onsubmit="return confirm(...)"` | Konfirmasi sebelum hapus |

---

## Langkah 4: Form Create

Buat `resources/views/tasks/create.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Tambah Task')

@section('content')
    <div class="mb-4">
        <a href="{{ route('tasks.index') }}" class="text-decoration-none small">
            ← Kembali ke Daftar Task
        </a>
    </div>

    <h1 class="h2 fw-bold mb-4">Tambah Task Baru</h1>

    <div class="card shadow-sm">
        <div class="card-body p-4">
            <form action="{{ route('tasks.store') }}" method="POST" novalidate>
                @csrf

                {{-- Judul --}}
                <div class="mb-3">
                    <label for="title" class="form-label fw-semibold">
                        Judul Task <span class="text-danger">*</span>
                    </label>
                    <input type="text"
                           name="title"
                           id="title"
                           value="{{ old('title') }}"
                           class="form-control @error('title') is-invalid @enderror"
                           placeholder="Contoh: Selesaikan laporan mingguan"
                           autofocus>
                    @error('title')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                {{-- Deskripsi --}}
                <div class="mb-3">
                    <label for="description" class="form-label fw-semibold">Deskripsi</label>
                    <textarea name="description"
                              id="description"
                              rows="4"
                              class="form-control @error('description') is-invalid @enderror"
                              placeholder="Detail task (opsional)">{{ old('description') }}</textarea>
                    @error('description')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                {{-- Status selesai --}}
                <div class="mb-4 form-check">
                    <input type="checkbox"
                           name="is_done"
                           value="1"
                           id="is_done"
                           class="form-check-input"
                           {{ old('is_done') ? 'checked' : '' }}>
                    <label for="is_done" class="form-check-label">
                        Tandai sudah selesai
                    </label>
                </div>

                {{-- Tombol --}}
                <div class="d-flex gap-2">
                    <button type="submit" class="btn btn-primary">
                        Simpan Task
                    </button>
                    <a href="{{ route('tasks.index') }}" class="btn btn-outline-secondary">
                        Batal
                    </a>
                </div>
            </form>
        </div>
    </div>
@endsection
```

**Penjelasan form:**

| Elemen | Penjelasan |
|---|---|
| `action="{{ route('tasks.store') }}"` | Submit ke POST /tasks |
| `method="POST"` | Method HTTP untuk create |
| `@csrf` | Token keamanan wajib |
| `class="form-control"` | Style input Bootstrap |
| `@error('title') is-invalid @enderror` | Border merah jika ada error |
| `{{ old('title') }}` | Isi ulang input setelah validasi gagal |
| `form-check` | Style checkbox Bootstrap |

---

## Langkah 5: Form Edit

Buat `resources/views/tasks/edit.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Edit Task')

@section('content')
    <div class="mb-4">
        <a href="{{ route('tasks.index') }}" class="text-decoration-none small">
            ← Kembali ke Daftar Task
        </a>
    </div>

    <h1 class="h2 fw-bold mb-4">Edit Task</h1>

    <div class="card shadow-sm">
        <div class="card-body p-4">
            <form action="{{ route('tasks.update', $task) }}" method="POST" novalidate>
                @csrf
                @method('PUT')

                <div class="mb-3">
                    <label for="title" class="form-label fw-semibold">
                        Judul Task <span class="text-danger">*</span>
                    </label>
                    <input type="text"
                           name="title"
                           id="title"
                           value="{{ old('title', $task->title) }}"
                           class="form-control @error('title') is-invalid @enderror"
                           autofocus>
                    @error('title')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-3">
                    <label for="description" class="form-label fw-semibold">Deskripsi</label>
                    <textarea name="description"
                              id="description"
                              rows="4"
                              class="form-control @error('description') is-invalid @enderror">{{ old('description', $task->description) }}</textarea>
                    @error('description')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-4 form-check">
                    <input type="checkbox"
                           name="is_done"
                           value="1"
                           id="is_done"
                           class="form-check-input"
                           {{ old('is_done', $task->is_done) ? 'checked' : '' }}>
                    <label for="is_done" class="form-check-label">
                        Tandai sudah selesai
                    </label>
                </div>

                <div class="d-flex gap-2">
                    <button type="submit" class="btn btn-primary">
                        Update Task
                    </button>
                    <a href="{{ route('tasks.show', $task) }}" class="btn btn-outline-secondary">
                        Batal
                    </a>
                </div>
            </form>
        </div>
    </div>
@endsection
```

**Perbedaan create vs edit:**

| Aspek | Create | Edit |
|---|---|---|
| Action | `route('tasks.store')` | `route('tasks.update', $task)` |
| Method spoofing | Tidak perlu | `@method('PUT')` |
| Value input | `old('title')` | `old('title', $task->title)` |
| Checkbox | `old('is_done')` | `old('is_done', $task->is_done)` |
| Tombol submit | "Simpan Task" | "Update Task" |

---

## Langkah 6: View Show (Detail)

Buat `resources/views/tasks/show.blade.php`:

```blade
@extends('layouts.app')

@section('title', $task->title)

@section('content')
    <div class="mb-4">
        <a href="{{ route('tasks.index') }}" class="text-decoration-none small">
            ← Kembali ke Daftar Task
        </a>
    </div>

    <div class="card shadow-sm">
        <div class="card-body p-4">
            <div class="d-flex justify-content-between align-items-start mb-3">
                <h1 class="h2 fw-bold mb-0 {{ $task->is_done ? 'text-decoration-line-through text-muted' : '' }}">
                    {{ $task->title }}
                </h1>
                <span class="badge fs-6 {{ $task->is_done ? 'bg-success' : 'bg-warning text-dark' }}">
                    {{ $task->is_done ? 'Selesai' : 'Belum Selesai' }}
                </span>
            </div>

            @if($task->description)
                <div class="mb-4">
                    <h2 class="h6 text-muted fw-semibold">Deskripsi</h2>
                    <p class="mb-0">{{ $task->description }}</p>
                </div>
            @else
                <p class="text-muted fst-italic mb-4">Tidak ada deskripsi.</p>
            @endif

            <div class="row text-muted small mb-4">
                <div class="col-md-6">
                    <strong>Dibuat:</strong> {{ $task->created_at->format('d M Y, H:i') }}
                </div>
                <div class="col-md-6">
                    <strong>Diupdate:</strong> {{ $task->updated_at->format('d M Y, H:i') }}
                </div>
            </div>

            <div class="d-flex gap-2">
                <a href="{{ route('tasks.edit', $task) }}" class="btn btn-warning">
                    Edit Task
                </a>
                <form action="{{ route('tasks.destroy', $task) }}"
                      method="POST"
                      class="d-inline"
                      onsubmit="return confirm('Yakin hapus task ini?')">
                    @csrf
                    @method('DELETE')
                    <button type="submit" class="btn btn-danger">Hapus</button>
                </form>
            </div>
        </div>
    </div>
@endsection
```

---

## Langkah 7: Pastikan Partial Alert Berfungsi

Pastikan `resources/views/partials/alert.blade.php` (dari Modul 04) menangani flash message:

```blade
@if(session('success'))
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        {{ session('success') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif

@if(session('error'))
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        {{ session('error') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif
```

Partial ini sudah di-include di `layouts/app.blade.php` via `@include('partials.alert')`.

---

## Langkah 8: Uji CRUD Lengkap

Jalankan server:

```bash
php artisan serve
```

### Checklist uji manual

| # | URL / Aksi | Hasil yang diharapkan |
|---|---|---|
| 1 | Buka `/tasks` | List task + tombol "Tambah Task" |
| 2 | Klik "Tambah Task" | Form create tampil |
| 3 | Submit form kosong | Error validasi di field judul |
| 4 | Submit judul "ab" (2 karakter) | Error "minimal 3 karakter" |
| 5 | Submit form valid | Redirect ke index + alert hijau |
| 6 | Klik judul task | Halaman detail (show) |
| 7 | Klik "Edit" | Form terisi data lama |
| 8 | Update judul | Redirect + alert "berhasil diupdate" |
| 9 | Centang "sudah selesai" + update | Badge berubah hijau |
| 10 | Klik "Hapus" → Cancel | Task tidak terhapus |
| 11 | Klik "Hapus" → OK | Task hilang + alert hijau |
| 12 | Buka `/tasks/999` | Halaman 404 Not Found |
| 13 | Refresh setelah alert | Alert hilang (flash message) |

---

## Alur Request Lengkap (Create)

```
1. User klik "Tambah Task"
   → GET /tasks/create
   → TaskController@create
   → View tasks/create.blade.php

2. User isi form, klik "Simpan"
   → POST /tasks (dengan @csrf)
   → TaskController@store
   → Validasi input
   → Jika gagal: redirect back + errors + old input
   → Jika sukses: Task::create() → redirect /tasks + flash success

3. Browser redirect ke /tasks
   → GET /tasks
   → TaskController@index
   → View tasks/index + alert hijau
```

---

## Latihan Modul 06

### Latihan 1: Validasi Deskripsi Wajib

Ubah validasi: deskripsi wajib diisi minimal 10 karakter. Tampilkan pesan error custom dalam Bahasa Indonesia.

### Latihan 2: Halaman 404 Custom

Buat view `resources/views/errors/404.blade.php` dengan Bootstrap card. Tampilkan pesan "Task tidak ditemukan" saat akses `/tasks/999`.

### Latihan 3: Search Box di Index

Tambah form GET di atas tabel:

```blade
<form method="GET" class="mb-3">
    <input type="text" name="search" value="{{ request('search') }}"
           placeholder="Cari task..." class="form-control form-control-sm">
</form>
```

Di controller:

```php
$query = Task::latest();
if ($request->filled('search')) {
    $query->where('title', 'like', '%' . $request->search . '%');
}
$tasks = $query->paginate(10)->withQueryString();
```

### Latihan 4: Form Request Terpisah

Pisahkan validasi ke class terpisah:

```bash
php artisan make:request StoreTaskRequest
php artisan make:request UpdateTaskRequest
```

Pindahkan rules ke `StoreTaskRequest`, pakai di controller:

```php
public function store(StoreTaskRequest $request)
{
    $validated = $request->validated();
    // ...
}
```

### Latihan 5: Toggle Selesai dari Index

Tambah route dan method baru:

```php
// routes/web.php
Route::patch('/tasks/{task}/toggle', [TaskController::class, 'toggle'])->name('tasks.toggle');
```

```php
public function toggle(Task $task)
{
    $task->update(['is_done' => !$task->is_done]);
    return back()->with('success', 'Status task diupdate!');
}
```

Tombol di index:

```blade
<form action="{{ route('tasks.toggle', $task) }}" method="POST" class="d-inline">
    @csrf
    @method('PATCH')
    <button class="btn btn-sm btn-outline-success">✓</button>
</form>
```

### Latihan 6 (Bonus): Soft Delete

Pelajari `$table->softDeletes()` di migration dan `use SoftDeletes` di model. Task yang dihapus tidak benar-benar hilang dari DB.

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| 419 Page Expired | CSRF token tidak valid/expired | Pastikan `@csrf` ada di form; refresh halaman |
| `MethodNotAllowedHttpException` | Route method salah | Form edit butuh `@method('PUT')`; hapus butuh `@method('DELETE')` |
| Validasi tidak muncul | `@error` tidak ditulis | Tambahkan `@error('field')` di view |
| Input kosong setelah error | Tidak pakai `old()` | Tambahkan `value="{{ old('field') }}"` |
| Checkbox selalu false | Tidak pakai `boolean()` | Pakai `$request->boolean('is_done')` di controller |
| `Add [title] to fillable` | Model `$fillable` kurang | Tambahkan kolom ke `$fillable` |
| Pagination link rusak | Bootstrap CSS pagination | Tambahkan view custom atau pakai `->withQueryString()` |
| Flash message tidak muncul | Partial alert belum ada | Pastikan `@include('partials.alert')` di layout |
| Hapus tidak jalan | Form method POST tanpa DELETE | Tambahkan `@method('DELETE')` |
| Route model binding 404 | ID task tidak ada | Normal — Laravel return 404 otomatis |

---

## Checklist Selesai

Centang sebelum lanjut ke Modul 07:

- [ ] 7 route resource terdaftar (`php artisan route:list --name=tasks`)
- [ ] Form create berfungsi dengan validasi
- [ ] Form edit menampilkan data lama
- [ ] Halaman show menampilkan detail task
- [ ] Hapus task dengan konfirmasi berfungsi
- [ ] Flash message sukses muncul setelah create/update/delete
- [ ] Error validasi tampil di form (border merah + pesan)
- [ ] `old()` mengisi ulang input setelah validasi gagal
- [ ] Pagination di halaman index
- [ ] Semua view pakai `@extends('layouts.app')` + Bootstrap CDN

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: crud task lengkap dengan validasi dan flash message"
git push
```

---

## Ringkasan Modul 06

| Topik | Yang dipelajari |
|---|---|
| Resource Controller | 7 method standar CRUD |
| Resource Route | `Route::resource()` mapping URL |
| CSRF | `@csrf` di setiap form POST |
| Method Spoofing | `@method('PUT')`, `@method('DELETE')` |
| Validasi | `$request->validate()`, custom messages |
| old() | Input ulang setelah validasi gagal |
| @error | Tampilkan error per field |
| Flash Message | `->with('success', '...')` |
| Route Model Binding | `Task $task` otomatis resolve by id |
| Bootstrap Form | `form-control`, `form-check`, `is-invalid` |

---

**Modul sebelumnya:** [05 — Database](../05-database/README.md)  
**Modul berikutnya:** [07 — Authentication](../07-authentication/README.md)
