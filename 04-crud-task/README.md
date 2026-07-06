# Modul 04 — CRUD Task

Modul keempat: membuat fitur Create, Read, Update, Delete lengkap untuk Task.

**Estimasi waktu:** 3–4 hari  
**Prasyarat:** [Modul 03 — Database](../03-database/README.md)

---

## Tujuan Pembelajaran

Setelah menyelesaikan modul ini, kamu sudah bisa:

- [ ] Membuat resource controller dengan 7 method CRUD
- [ ] Membuat form tambah dan edit data
- [ ] Memvalidasi input form
- [ ] Menghapus data dengan konfirmasi
- [ ] Menampilkan flash message setelah aksi berhasil

---

## Apa itu CRUD?

| Aksi | Method HTTP | Route | Fungsi |
|---|---|---|---|
| **C**reate | POST | `/tasks` | Tambah data baru |
| **R**ead | GET | `/tasks`, `/tasks/{id}` | Lihat daftar & detail |
| **U**pdate | PUT/PATCH | `/tasks/{id}` | Edit data |
| **D**elete | DELETE | `/tasks/{id}` | Hapus data |

---

## Langkah 1: Resource Controller

Hapus `TaskController` lama, buat ulang sebagai resource:

```bash
php artisan make:controller TaskController --resource
```

Isi `app/Http/Controllers/TaskController.php`:

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
            'title'       => 'required|string|max:255',
            'description' => 'nullable|string',
            'is_done'     => 'boolean',
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
            'title'       => 'required|string|max:255',
            'description' => 'nullable|string',
            'is_done'     => 'boolean',
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

> `Task $task` di parameter method adalah **Route Model Binding** — Laravel otomatis cari task berdasarkan ID di URL.

---

## Langkah 2: Daftarkan Resource Route

Edit `routes/web.php`:

```php
use App\Http\Controllers\TaskController;

Route::resource('tasks', TaskController::class);
```

Cek 7 route yang otomatis dibuat:

```bash
php artisan route:list --name=tasks
```

---

## Langkah 3: View — Daftar Task

Update `resources/views/tasks/index.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Daftar Task')

@section('content')
    <div class="flex items-center justify-between mb-6">
        <h1 class="text-2xl font-bold">Daftar Task</h1>
        <a href="{{ route('tasks.create') }}"
           class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">
            + Tambah Task
        </a>
    </div>

    @if($tasks->isEmpty())
        <p class="text-gray-500">Belum ada task. <a href="{{ route('tasks.create') }}" class="text-blue-600 underline">Tambah sekarang</a></p>
    @else
        <table class="w-full bg-white rounded shadow-sm">
            <thead class="bg-gray-100">
                <tr>
                    <th class="text-left p-3">No</th>
                    <th class="text-left p-3">Judul</th>
                    <th class="text-left p-3">Status</th>
                    <th class="text-left p-3">Aksi</th>
                </tr>
            </thead>
            <tbody>
                @foreach($tasks as $index => $task)
                    <tr class="border-t">
                        <td class="p-3">{{ $tasks->firstItem() + $index }}</td>
                        <td class="p-3">
                            <a href="{{ route('tasks.show', $task) }}" class="text-blue-600 hover:underline">
                                {{ $task->title }}
                            </a>
                        </td>
                        <td class="p-3">
                            <span class="text-xs px-2 py-1 rounded {{ $task->is_done ? 'bg-green-100 text-green-700' : 'bg-yellow-100 text-yellow-700' }}">
                                {{ $task->is_done ? 'Selesai' : 'Belum' }}
                            </span>
                        </td>
                        <td class="p-3 flex gap-2">
                            <a href="{{ route('tasks.edit', $task) }}"
                               class="text-sm text-yellow-600 hover:underline">Edit</a>
                            <form action="{{ route('tasks.destroy', $task) }}" method="POST"
                                  onsubmit="return confirm('Yakin hapus task ini?')">
                                @csrf
                                @method('DELETE')
                                <button type="submit" class="text-sm text-red-600 hover:underline">Hapus</button>
                            </form>
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>

        <div class="mt-4">
            {{ $tasks->links() }}
        </div>
    @endif
@endsection
```

---

## Langkah 4: View — Form Tambah

Buat `resources/views/tasks/create.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Tambah Task')

@section('content')
    <h1 class="text-2xl font-bold mb-6">Tambah Task</h1>

    <form action="{{ route('tasks.store') }}" method="POST" class="bg-white p-6 rounded shadow-sm space-y-4">
        @csrf

        <div>
            <label class="block text-sm font-medium mb-1">Judul <span class="text-red-500">*</span></label>
            <input type="text" name="title" value="{{ old('title') }}"
                   class="w-full border rounded px-3 py-2 @error('title') border-red-500 @enderror">
            @error('title')
                <p class="text-red-500 text-sm mt-1">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label class="block text-sm font-medium mb-1">Deskripsi</label>
            <textarea name="description" rows="3"
                      class="w-full border rounded px-3 py-2">{{ old('description') }}</textarea>
        </div>

        <div class="flex items-center gap-2">
            <input type="checkbox" name="is_done" value="1" id="is_done" {{ old('is_done') ? 'checked' : '' }}>
            <label for="is_done">Sudah selesai</label>
        </div>

        <div class="flex gap-3">
            <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">
                Simpan
            </button>
            <a href="{{ route('tasks.index') }}" class="px-4 py-2 border rounded hover:bg-gray-50">Batal</a>
        </div>
    </form>
@endsection
```

---

## Langkah 5: View — Form Edit

Buat `resources/views/tasks/edit.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Edit Task')

@section('content')
    <h1 class="text-2xl font-bold mb-6">Edit Task</h1>

    <form action="{{ route('tasks.update', $task) }}" method="POST" class="bg-white p-6 rounded shadow-sm space-y-4">
        @csrf
        @method('PUT')

        <div>
            <label class="block text-sm font-medium mb-1">Judul <span class="text-red-500">*</span></label>
            <input type="text" name="title" value="{{ old('title', $task->title) }}"
                   class="w-full border rounded px-3 py-2 @error('title') border-red-500 @enderror">
            @error('title')
                <p class="text-red-500 text-sm mt-1">{{ $message }}</p>
            @enderror
        </div>

        <div>
            <label class="block text-sm font-medium mb-1">Deskripsi</label>
            <textarea name="description" rows="3"
                      class="w-full border rounded px-3 py-2">{{ old('description', $task->description) }}</textarea>
        </div>

        <div class="flex items-center gap-2">
            <input type="checkbox" name="is_done" value="1" id="is_done"
                   {{ old('is_done', $task->is_done) ? 'checked' : '' }}>
            <label for="is_done">Sudah selesai</label>
        </div>

        <div class="flex gap-3">
            <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">
                Update
            </button>
            <a href="{{ route('tasks.index') }}" class="px-4 py-2 border rounded hover:bg-gray-50">Batal</a>
        </div>
    </form>
@endsection
```

---

## Langkah 6: View — Detail Task

Buat `resources/views/tasks/show.blade.php`:

```blade
@extends('layouts.app')

@section('title', $task->title)

@section('content')
    <div class="bg-white p-6 rounded shadow-sm">
        <div class="flex items-center justify-between mb-4">
            <h1 class="text-2xl font-bold">{{ $task->title }}</h1>
            <span class="text-xs px-2 py-1 rounded {{ $task->is_done ? 'bg-green-100 text-green-700' : 'bg-yellow-100 text-yellow-700' }}">
                {{ $task->is_done ? 'Selesai' : 'Belum' }}
            </span>
        </div>

        @if($task->description)
            <p class="text-gray-600 mb-4">{{ $task->description }}</p>
        @endif

        <p class="text-sm text-gray-400 mb-6">
            Dibuat: {{ $task->created_at->format('d M Y H:i') }}
        </p>

        <div class="flex gap-3">
            <a href="{{ route('tasks.edit', $task) }}"
               class="bg-yellow-500 text-white px-4 py-2 rounded hover:bg-yellow-600">Edit</a>
            <a href="{{ route('tasks.index') }}"
               class="px-4 py-2 border rounded hover:bg-gray-50">Kembali</a>
        </div>
    </div>
@endsection
```

---

## Langkah 7: Uji Semua Fitur

Jalankan `php artisan serve` lalu uji:

| URL | Yang diuji |
|---|---|
| `/tasks` | Daftar task tampil |
| `/tasks/create` | Form tambah tampil |
| Submit form tambah | Data masuk DB, redirect + flash message |
| `/tasks/1/edit` | Form edit terisi data lama |
| Submit form edit | Data terupdate |
| `/tasks/1` | Detail task tampil |
| Klik hapus | Data terhapus setelah konfirmasi |
| Submit form kosong | Validasi error tampil |

---

## Validasi yang Dipakai

```php
$validated = $request->validate([
    'title'       => 'required|string|max:255',
    'description' => 'nullable|string',
    'is_done'     => 'boolean',
]);
```

| Rule | Arti |
|---|---|
| `required` | Wajib diisi |
| `nullable` | Boleh kosong |
| `string` | Harus teks |
| `max:255` | Maksimal 255 karakter |
| `boolean` | true/false |

---

## Latihan

1. Tambah validasi: judul minimal 3 karakter
2. Tambah kolom pencarian — filter task berdasarkan judul
3. Tambah tombol "Tandai selesai" tanpa harus buka halaman edit
4. Tampilkan total task selesai vs belum di halaman index
5. Buat halaman `/tasks/done` yang hanya menampilkan task selesai

### Bonus

Buat **Form Request** untuk validasi terpisah:

```bash
php artisan make:request StoreTaskRequest
```

```php
public function rules(): array
{
    return [
        'title'       => 'required|string|min:3|max:255',
        'description' => 'nullable|string',
        'is_done'     => 'boolean',
    ];
}
```

Lalu di controller: `public function store(StoreTaskRequest $request)`

---

## Checklist Selesai

- [ ] Resource route `tasks` terdaftar (7 route)
- [ ] Bisa tambah task baru
- [ ] Bisa edit task
- [ ] Bisa hapus task dengan konfirmasi
- [ ] Bisa lihat detail task
- [ ] Validasi form berfungsi
- [ ] Flash message muncul setelah create/update/delete
- [ ] Pagination berfungsi di halaman index

---

**Modul sebelumnya:** [03 — Database](../03-database/README.md)  
**Modul berikutnya:** [05 — Authentication](../05-auth/README.md)
