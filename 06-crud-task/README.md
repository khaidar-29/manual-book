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

Tambah tombol tambah, link edit/hapus, pagination di `tasks/index.blade.php`:

```blade
@section('content')
    <div class="flex items-center justify-between mb-6">
        <h1 class="text-2xl font-bold">Daftar Task</h1>
        <a href="{{ route('tasks.create') }}"
           class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 text-sm">
            + Tambah Task
        </a>
    </div>

    {{-- loop task seperti modul 05, tambah kolom aksi: --}}
    <a href="{{ route('tasks.edit', $task) }}" class="text-yellow-600 text-sm">Edit</a>
    <form action="{{ route('tasks.destroy', $task) }}" method="POST" class="inline"
          onsubmit="return confirm('Yakin hapus?')">
        @csrf
        @method('DELETE')
        <button type="submit" class="text-red-600 text-sm">Hapus</button>
    </form>

    <div class="mt-4">{{ $tasks->links() }}</div>
@endsection
```

---

## Langkah 4: Form Create

**`resources/views/tasks/create.blade.php`:**

```blade
@extends('layouts.app')

@section('title', 'Tambah Task')

@section('content')
    <h1 class="text-2xl font-bold mb-6">Tambah Task</h1>

    <form action="{{ route('tasks.store') }}" method="POST"
          class="bg-white rounded-lg shadow-sm p-6 space-y-4">
        @csrf

        <div>
            <label class="block text-sm font-medium mb-1">Judul <span class="text-red-500">*</span></label>
            <input type="text" name="title" value="{{ old('title') }}"
                   class="w-full border rounded px-3 py-2 @error('title') border-red-500 @enderror">
            @error('title')<p class="text-red-500 text-sm mt-1">{{ $message }}</p>@enderror
        </div>

        <div>
            <label class="block text-sm font-medium mb-1">Deskripsi</label>
            <textarea name="description" rows="3"
                      class="w-full border rounded px-3 py-2">{{ old('description') }}</textarea>
        </div>

        <div class="flex items-center gap-2">
            <input type="checkbox" name="is_done" value="1" id="is_done">
            <label for="is_done">Sudah selesai</label>
        </div>

        <div class="flex gap-3">
            <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded">Simpan</button>
            <a href="{{ route('tasks.index') }}" class="px-4 py-2 border rounded">Batal</a>
        </div>
    </form>
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
    <div class="bg-white rounded-lg shadow-sm p-6">
        <div class="flex justify-between items-start mb-4">
            <h1 class="text-2xl font-bold">{{ $task->title }}</h1>
            <span class="text-xs px-2 py-1 rounded
                {{ $task->is_done ? 'bg-green-100 text-green-700' : 'bg-yellow-100 text-yellow-700' }}">
                {{ $task->is_done ? 'Selesai' : 'Belum' }}
            </span>
        </div>
        @if($task->description)
            <p class="text-gray-600 mb-4">{{ $task->description }}</p>
        @endif
        <p class="text-sm text-gray-400 mb-6">Dibuat: {{ $task->created_at->format('d M Y H:i') }}</p>
        <div class="flex gap-3">
            <a href="{{ route('tasks.edit', $task) }}" class="bg-yellow-500 text-white px-4 py-2 rounded">Edit</a>
            <a href="{{ route('tasks.index') }}" class="px-4 py-2 border rounded">Kembali</a>
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
