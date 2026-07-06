# Modul 06 — CRUD Task

Melengkapi **Task Manager** dengan Create, Read, Update, Delete task.

**Estimasi waktu:** 2–3 hari  
**Prasyarat:** [Modul 05 — Database](../05-database/README.md)

---

## Tujuan Modul

- [ ] Resource controller 7 method CRUD
- [ ] Form tambah & edit dengan validasi
- [ ] Hapus dengan konfirmasi
- [ ] Flash message sukses/error
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
TaskController → index, create, store, show, edit, update, destroy
resources/views/tasks/create.blade.php
resources/views/tasks/edit.blade.php
resources/views/tasks/show.blade.php
Route::resource('tasks', TaskController::class)
```

---

## Langkah 1: Resource Controller

```bash
# Hapus TaskController lama, buat ulang:
php artisan make:controller TaskController --resource
```

**`app/Http/Controllers/TaskController.php`:**

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function index()
    {
        $tasks = Task::latest()->paginate(10);
        return view('tasks.index', compact('tasks'));
    }

    public function create()
    {
        return view('tasks.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title'       => 'required|string|min:3|max:255',
            'description' => 'nullable|string',
        ]);

        $validated['is_done'] = $request->boolean('is_done');

        Task::create($validated);

        return redirect()->route('tasks.index')
            ->with('success', 'Task berhasil ditambahkan!');
    }

    public function show(Task $task)
    {
        return view('tasks.show', compact('task'));
    }

    public function edit(Task $task)
    {
        return view('tasks.edit', compact('task'));
    }

    public function update(Request $request, Task $task)
    {
        $validated = $request->validate([
            'title'       => 'required|string|min:3|max:255',
            'description' => 'nullable|string',
        ]);

        $validated['is_done'] = $request->boolean('is_done');

        $task->update($validated);

        return redirect()->route('tasks.index')
            ->with('success', 'Task berhasil diupdate!');
    }

    public function destroy(Task $task)
    {
        $task->delete();

        return redirect()->route('tasks.index')
            ->with('success', 'Task berhasil dihapus!');
    }
}
```

---

## Langkah 2: Resource Route

**`routes/web.php`:**

```php
Route::resource('tasks', TaskController::class);
```

```bash
php artisan route:list --name=tasks
```

---

## Langkah 3: Update View Index

**`resources/views/tasks/index.blade.php`:**

```blade
@extends('layouts.app')

@section('title', 'Daftar Task')

@section('content')
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h1 class="h2 fw-bold mb-0">Daftar Task</h1>
        <a href="{{ route('tasks.create') }}" class="btn btn-primary btn-sm">+ Tambah Task</a>
    </div>

    @if($tasks->isEmpty())
        <p class="text-muted">Belum ada task.
            <a href="{{ route('tasks.create') }}">Tambah sekarang</a>
        </p>
    @else
        <div class="table-responsive">
            <table class="table table-striped table-hover bg-white">
                <thead class="table-light">
                    <tr>
                        <th>No</th>
                        <th>Judul</th>
                        <th>Status</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach($tasks as $index => $task)
                        <tr>
                            <td>{{ $tasks->firstItem() + $index }}</td>
                            <td>
                                <a href="{{ route('tasks.show', $task) }}">{{ $task->title }}</a>
                            </td>
                            <td>
                                <span class="badge {{ $task->is_done ? 'bg-success' : 'bg-warning text-dark' }}">
                                    {{ $task->is_done ? 'Selesai' : 'Belum' }}
                                </span>
                            </td>
                            <td>
                                <a href="{{ route('tasks.edit', $task) }}" class="btn btn-warning btn-sm">Edit</a>
                                <form action="{{ route('tasks.destroy', $task) }}" method="POST" class="d-inline"
                                      onsubmit="return confirm('Yakin hapus?')">
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
        <div class="mt-3">{{ $tasks->links() }}</div>
    @endif
@endsection
```

---

## Langkah 4: Form Create

**`resources/views/tasks/create.blade.php`:**

```blade
@extends('layouts.app')

@section('title', 'Tambah Task')

@section('content')
    <h1 class="h2 fw-bold mb-4">Tambah Task</h1>

    <div class="card shadow-sm">
        <div class="card-body">
            <form action="{{ route('tasks.store') }}" method="POST">
                @csrf

                <div class="mb-3">
                    <label class="form-label">Judul <span class="text-danger">*</span></label>
                    <input type="text" name="title" value="{{ old('title') }}"
                           class="form-control @error('title') is-invalid @enderror">
                    @error('title')<div class="invalid-feedback">{{ $message }}</div>@enderror
                </div>

                <div class="mb-3">
                    <label class="form-label">Deskripsi</label>
                    <textarea name="description" rows="3" class="form-control">{{ old('description') }}</textarea>
                </div>

                <div class="mb-3 form-check">
                    <input type="checkbox" name="is_done" value="1" id="is_done" class="form-check-input">
                    <label for="is_done" class="form-check-label">Sudah selesai</label>
                </div>

                <div class="d-flex gap-2">
                    <button type="submit" class="btn btn-primary">Simpan</button>
                    <a href="{{ route('tasks.index') }}" class="btn btn-outline-secondary">Batal</a>
                </div>
            </form>
        </div>
    </div>
@endsection
```

---

## Langkah 5: Form Edit & Show

**`resources/views/tasks/edit.blade.php`** — sama dengan create, tapi:
- `action="{{ route('tasks.update', $task) }}"`
- `@method('PUT')`
- `value="{{ old('title', $task->title) }}"`
- Checkbox: `{{ old('is_done', $task->is_done) ? 'checked' : '' }}`

**`resources/views/tasks/show.blade.php`:**

```blade
@extends('layouts.app')

@section('title', $task->title)

@section('content')
    <div class="card shadow-sm">
        <div class="card-body">
            <div class="d-flex justify-content-between align-items-start mb-3">
                <h1 class="h2 fw-bold mb-0">{{ $task->title }}</h1>
                <span class="badge {{ $task->is_done ? 'bg-success' : 'bg-warning text-dark' }}">
                    {{ $task->is_done ? 'Selesai' : 'Belum' }}
                </span>
            </div>
            @if($task->description)
                <p class="text-muted">{{ $task->description }}</p>
            @endif
            <p class="small text-muted mb-4">Dibuat: {{ $task->created_at->format('d M Y H:i') }}</p>
            <div class="d-flex gap-2">
                <a href="{{ route('tasks.edit', $task) }}" class="btn btn-warning">Edit</a>
                <a href="{{ route('tasks.index') }}" class="btn btn-outline-secondary">Kembali</a>
            </div>
        </div>
    </div>
@endsection
```

---

## Uji CRUD

| URL | Uji |
|---|---|
| `/tasks` | List + tombol tambah |
| `/tasks/create` | Form tambah |
| Submit form kosong | Validasi error muncul |
| Submit form valid | Redirect + flash success |
| `/tasks/1/edit` | Form terisi data lama |
| `/tasks/1` | Detail task |
| Hapus | Konfirmasi → data hilang |

---

## Latihan Modul 06

1. Validasi: judul wajib minimal 3 karakter
2. Tampilkan `@error` di semua field
3. Tambah search box — filter task by judul
4. Buat `StoreTaskRequest` terpisah (`php artisan make:request StoreTaskRequest`)
5. Tombol "Tandai selesai" langsung dari halaman index (tanpa buka edit)

---

## Git — Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/06-crud-task

git add .
git commit -m "feat: crud task lengkap dengan validasi dan flash message"
git push -u origin modul/06-crud-task
```

---

## Checklist Selesai

- [ ] 7 route resource terdaftar
- [ ] Tambah, edit, hapus, detail task berfungsi
- [ ] Validasi form jalan
- [ ] Flash message muncul
- [ ] Pagination di index
- [ ] Commit & push ke GitHub

---

**Modul sebelumnya:** [05 — Database](../05-database/README.md)  
**Modul berikutnya:** [07 — Authentication](../07-authentication/README.md)
