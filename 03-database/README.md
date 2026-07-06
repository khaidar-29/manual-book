# Modul 03 — Database

Modul ketiga: migration, model, seeder, dan menampilkan data dari database.

**Estimasi waktu:** 2–3 hari  
**Prasyarat:** [Modul 02 — MVC Dasar](../02-mvc-dasar/README.md)

---

## Tujuan Pembelajaran

Setelah menyelesaikan modul ini, kamu sudah bisa:

- [ ] Membuat migration untuk tabel baru
- [ ] Membuat model Eloquent
- [ ] Mengisi data dummy dengan seeder
- [ ] Menampilkan data dari database di halaman web
- [ ] Menjalankan query dasar Eloquent

---

## Konsep

```
Migration   → Mendefinisikan struktur tabel (schema)
Model       → Representasi tabel di PHP
Seeder      → Mengisi data awal / dummy
Controller  → Mengambil data via Model
View        → Menampilkan data ke user
```

Di modul ini fokus ke **menampilkan data** (Read). CRUD lengkap ada di modul 04.

---

## Langkah 1: Buat Migration & Model

```bash
php artisan make:model Task -m
```

Flag `-m` sekaligus membuat file migration.

Edit file migration di `database/migrations/xxxx_create_tasks_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description')->nullable();
            $table->boolean('is_done')->default(false);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

Jalankan migration:

```bash
php artisan migrate
```

---

## Langkah 2: Konfigurasi Model

Edit `app/Models/Task.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    protected $fillable = [
        'title',
        'description',
        'is_done',
    ];

    protected $casts = [
        'is_done' => 'boolean',
    ];
}
```

> `$fillable` menentukan kolom mana yang boleh diisi lewat `create()` atau `update()`.

---

## Langkah 3: Buat Seeder

```bash
php artisan make:seeder TaskSeeder
```

Isi `database/seeders/TaskSeeder.php`:

```php
<?php

namespace Database\Seeders;

use App\Models\Task;
use Illuminate\Database\Seeder;

class TaskSeeder extends Seeder
{
    public function run(): void
    {
        $tasks = [
            [
                'title' => 'Belajar Routing',
                'description' => 'Memahami cara kerja route di Laravel',
                'is_done' => true,
            ],
            [
                'title' => 'Belajar Controller',
                'description' => 'Membuat controller dan mengirim data ke view',
                'is_done' => true,
            ],
            [
                'title' => 'Belajar Migration',
                'description' => 'Membuat tabel tasks di database',
                'is_done' => false,
            ],
            [
                'title' => 'Belajar Eloquent',
                'description' => 'Query data menggunakan model',
                'is_done' => false,
            ],
            [
                'title' => 'Buat CRUD Task',
                'description' => 'Tambah, edit, dan hapus task',
                'is_done' => false,
            ],
        ];

        foreach ($tasks as $task) {
            Task::create($task);
        }
    }
}
```

Daftarkan di `database/seeders/DatabaseSeeder.php`:

```php
public function run(): void
{
    $this->call(TaskSeeder::class);
}
```

Jalankan seeder:

```bash
php artisan db:seed --class=TaskSeeder
```

---

## Langkah 4: Buat Controller

```bash
php artisan make:controller TaskController
```

Isi `app/Http/Controllers/TaskController.php` (hanya method `index` dulu):

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;

class TaskController extends Controller
{
    public function index()
    {
        $tasks = Task::latest()->get();

        return view('tasks.index', compact('tasks'));
    }
}
```

---

## Langkah 5: Buat View

Buat `resources/views/tasks/index.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Daftar Task')

@section('content')
    <h1 class="text-2xl font-bold mb-6">Daftar Task</h1>

    @if($tasks->isEmpty())
        <p class="text-gray-500">Belum ada task.</p>
    @else
        <div class="space-y-3">
            @foreach($tasks as $task)
                <div class="bg-white p-4 rounded shadow-sm border">
                    <div class="flex items-center justify-between">
                        <h2 class="font-semibold {{ $task->is_done ? 'line-through text-gray-400' : '' }}">
                            {{ $task->title }}
                        </h2>
                        <span class="text-xs px-2 py-1 rounded {{ $task->is_done ? 'bg-green-100 text-green-700' : 'bg-yellow-100 text-yellow-700' }}">
                            {{ $task->is_done ? 'Selesai' : 'Belum' }}
                        </span>
                    </div>
                    @if($task->description)
                        <p class="text-sm text-gray-600 mt-1">{{ $task->description }}</p>
                    @endif
                </div>
            @endforeach
        </div>
    @endif
@endsection
```

---

## Langkah 6: Daftarkan Route

Edit `routes/web.php`, tambahkan:

```php
use App\Http\Controllers\TaskController;

Route::get('/tasks', [TaskController::class, 'index'])->name('tasks.index');
```

Tambahkan link di navbar (`partials/navbar.blade.php`):

```blade
<a href="{{ route('tasks.index') }}">Tasks</a>
```

Buka `http://localhost:8000/tasks` — data dari seeder harus tampil.

---

## Query Eloquent Dasar

Coba di `php artisan tinker`:

```php
// Ambil semua
App\Models\Task::all();

// Ambil yang belum selesai
App\Models\Task::where('is_done', false)->get();

// Hitung jumlah
App\Models\Task::count();

// Ambil satu
App\Models\Task::find(1);

// Urutkan terbaru
App\Models\Task::latest()->get();

// Pagination (untuk data banyak)
App\Models\Task::paginate(5);
```

---

## Perintah Database Penting

```bash
php artisan migrate                  # Jalankan migration baru
php artisan migrate:rollback         # Batalkan migration terakhir
php artisan migrate:fresh            # Drop semua tabel & migrate ulang
php artisan migrate:fresh --seed     # Fresh + jalankan seeder
php artisan db:seed                  # Jalankan semua seeder
php artisan db:seed --class=TaskSeeder  # Jalankan seeder tertentu
```

---

## Latihan

1. Tambah kolom `priority` (string: low, medium, high) ke tabel tasks via migration baru
2. Tampilkan badge warna berbeda untuk setiap priority di halaman index
3. Tampilkan hanya task yang belum selesai — buat route `/tasks/pending`
4. Tampilkan jumlah task selesai vs belum di bagian atas halaman
5. Ganti `get()` menjadi `paginate(3)` dan tambahkan `{{ $tasks->links() }}` di view

### Bonus

Buat migration untuk menambah kolom:

```bash
php artisan make:migration add_priority_to_tasks_table
```

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->string('priority')->default('medium')->after('is_done');
    });
}
```

---

## Checklist Selesai

- [ ] Tabel `tasks` dibuat via migration
- [ ] Model `Task` dengan `$fillable` dikonfigurasi
- [ ] Seeder mengisi minimal 5 data dummy
- [ ] Halaman `/tasks` menampilkan data dari database
- [ ] Bisa menjalankan query dasar di tinker
- [ ] Paham beda migration, model, dan seeder

---

**Modul sebelumnya:** [02 — MVC Dasar](../02-mvc-dasar/README.md)  
**Modul berikutnya:** [04 — CRUD Task](../04-crud-task/README.md)
