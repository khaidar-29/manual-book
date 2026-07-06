# Modul 05 — Database

Menambahkan tabel `tasks` ke **Task Manager** dan menampilkan data dari database.

**Estimasi waktu:** 2 hari  
**Prasyarat:** [Modul 04 — Blade Template](../04-blade-template/README.md)

---

## Tujuan Modul

- [ ] Migration tabel `tasks` dibuat & dijalankan
- [ ] Model `Task` dikonfigurasi
- [ ] Seeder mengisi data dummy
- [ ] Halaman `/tasks` menampilkan list dari DB
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
database/migrations/..._create_tasks_table.php
app/Models/Task.php
database/seeders/TaskSeeder.php
app/Http/Controllers/TaskController.php  (method index saja)
resources/views/tasks/index.blade.php
GET /tasks → tasks.index
```

> Modul ini fokus **Read** saja. CRUD lengkap di Modul 06.

---

## Langkah 1: Migration & Model

```bash
php artisan make:model Task -m
```

Edit migration `database/migrations/xxxx_create_tasks_table.php`:

```php
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
```

```bash
php artisan migrate
```

---

## Langkah 2: Model

**`app/Models/Task.php`:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    protected $fillable = ['title', 'description', 'is_done'];

    protected $casts = [
        'is_done' => 'boolean',
    ];
}
```

---

## Langkah 3: Seeder

```bash
php artisan make:seeder TaskSeeder
```

**`database/seeders/TaskSeeder.php`:**

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
            ['title' => 'Setup project Laravel', 'description' => 'Install & konfigurasi environment', 'is_done' => true],
            ['title' => 'Pelajari Routing & Controller', 'description' => 'Memahami alur MVC', 'is_done' => true],
            ['title' => 'Buat layout Blade', 'description' => 'Layout, navbar, partial', 'is_done' => true],
            ['title' => 'Hubungkan ke database', 'description' => 'Migration, model, seeder', 'is_done' => false],
            ['title' => 'Buat CRUD task', 'description' => 'Tambah, edit, hapus task', 'is_done' => false],
        ];

        foreach ($tasks as $task) {
            Task::create($task);
        }
    }
}
```

Daftarkan di `DatabaseSeeder.php`:

```php
$this->call(TaskSeeder::class);
```

```bash
php artisan db:seed --class=TaskSeeder
```

---

## Langkah 4: Controller (index saja)

```bash
php artisan make:controller TaskController
```

**`app/Http/Controllers/TaskController.php`:**

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

## Langkah 5: View & Route

**`resources/views/tasks/index.blade.php`:**

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
                <div class="bg-white p-4 rounded-lg shadow-sm border flex justify-between items-start">
                    <div>
                        <h2 class="font-semibold {{ $task->is_done ? 'line-through text-gray-400' : '' }}">
                            {{ $task->title }}
                        </h2>
                        @if($task->description)
                            <p class="text-sm text-gray-500 mt-1">{{ $task->description }}</p>
                        @endif
                    </div>
                    <span class="text-xs px-2 py-1 rounded shrink-0 ml-4
                        {{ $task->is_done ? 'bg-green-100 text-green-700' : 'bg-yellow-100 text-yellow-700' }}">
                        {{ $task->is_done ? 'Selesai' : 'Belum' }}
                    </span>
                </div>
            @endforeach
        </div>
    @endif
@endsection
```

**`routes/web.php`** — tambahkan:

```php
use App\Http\Controllers\TaskController;

Route::get('/tasks', [TaskController::class, 'index'])->name('tasks.index');
```

**Navbar** — tambah link:

```blade
<a href="{{ route('tasks.index') }}">Tasks</a>
```

---

## Eksplorasi di Tinker

```bash
php artisan tinker
```

```php
App\Models\Task::count();
App\Models\Task::where('is_done', false)->get();
App\Models\Task::find(1);
```

---

## Latihan Modul 05

1. Tambah kolom `priority` (low/medium/high) via migration baru
2. Tampilkan badge warna berbeda per priority
3. Buat route `/tasks/pending` — hanya task belum selesai
4. Tampilkan statistik di atas list: "X selesai, Y belum"
5. Ganti `get()` jadi `paginate(5)` + `{{ $tasks->links() }}`

---

## Git — Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/05-database

git add .
git commit -m "feat: tambah migration tasks, seeder, dan halaman list task"
git push -u origin modul/05-database
```

---

## Checklist Selesai

- [ ] Tabel `tasks` ada di database
- [ ] Seeder mengisi ≥5 task
- [ ] `/tasks` menampilkan data dari DB
- [ ] Link Tasks ada di navbar
- [ ] Bisa query dasar di tinker
- [ ] Commit & push ke GitHub

---

**Modul sebelumnya:** [04 — Blade Template](../04-blade-template/README.md)  
**Modul berikutnya:** [06 — CRUD Task](../06-crud-task/README.md)
