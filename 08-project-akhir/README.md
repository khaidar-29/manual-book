# Modul 08 — Project Akhir

Menyelesaikan **Task Manager** — dashboard, kategori, search. App siap demo.

**Estimasi waktu:** 2–3 hari  
**Prasyarat:** [Modul 07 — Authentication](../07-authentication/README.md)

---

## Tujuan Modul

- [ ] Dashboard ringkasan task
- [ ] CRUD kategori & relasi ke task
- [ ] Filter task by kategori
- [ ] Search task by judul
- [ ] README project & final push ke GitHub

**Yang ditambahkan ke project:**

```
Tabel categories + relasi task belongsTo category
DashboardController + view dashboard
Filter & search di TaskController
README.md project lengkap
```

---

## Langkah 1: Tabel Categories

```bash
php artisan make:model Category -m
php artisan make:controller CategoryController --resource
```

**Migration `create_categories_table`:**

```php
public function up(): void
{
    Schema::create('categories', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('color')->default('blue'); // blue, green, red, yellow
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->timestamps();
    });
}
```

**Migration tambah category_id ke tasks:**

```bash
php artisan make:migration add_category_id_to_tasks_table
```

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->foreignId('category_id')->nullable()->after('is_done')
              ->constrained()->nullOnDelete();
    });
}
```

```bash
php artisan migrate
```

---

## Langkah 2: Model & Relasi

**`app/Models/Category.php`:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    protected $fillable = ['name', 'color', 'user_id'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function tasks()
    {
        return $this->hasMany(Task::class);
    }
}
```

**Update `app/Models/Task.php`:**

```php
public function category()
{
    return $this->belongsTo(Category::class);
}
```

**Update `app/Models/User.php`:**

```php
public function categories()
{
    return $this->hasMany(Category::class);
}
```

---

## Langkah 3: Seeder Kategori

**`database/seeders/CategorySeeder.php`:**

```php
public function run(): void
{
    $user = \App\Models\User::first();
    if (!$user) return;

    $categories = [
        ['name' => 'Work', 'color' => 'blue'],
        ['name' => 'Personal', 'color' => 'green'],
        ['name' => 'Study', 'color' => 'yellow'],
    ];

    foreach ($categories as $cat) {
        $user->categories()->create($cat);
    }
}
```

---

## Langkah 4: Dashboard

```bash
php artisan make:controller DashboardController
```

**`app/Http/Controllers/DashboardController.php`:**

```php
<?php

namespace App\Http\Controllers;

class DashboardController extends Controller
{
    public function index()
    {
        $user = auth()->user();
        $total = $user->tasks()->count();
        $done = $user->tasks()->where('is_done', true)->count();
        $pending = $total - $done;
        $recentTasks = $user->tasks()->latest()->take(5)->get();
        $categories = $user->categories()->withCount('tasks')->get();

        return view('dashboard', compact('total', 'done', 'pending', 'recentTasks', 'categories'));
    }
}
```

**Route:**

```php
Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
```

**Update `resources/views/dashboard.blade.php`:**

```blade
@extends('layouts.app')

@section('title', 'Dashboard')

@section('content')
    <h1 class="text-2xl font-bold mb-6">Dashboard</h1>

    <div class="grid grid-cols-3 gap-4 mb-8">
        <div class="bg-white p-4 rounded-lg shadow-sm text-center">
            <p class="text-3xl font-bold text-blue-600">{{ $total }}</p>
            <p class="text-sm text-gray-500">Total Task</p>
        </div>
        <div class="bg-white p-4 rounded-lg shadow-sm text-center">
            <p class="text-3xl font-bold text-green-600">{{ $done }}</p>
            <p class="text-sm text-gray-500">Selesai</p>
        </div>
        <div class="bg-white p-4 rounded-lg shadow-sm text-center">
            <p class="text-3xl font-bold text-yellow-600">{{ $pending }}</p>
            <p class="text-sm text-gray-500">Belum Selesai</p>
        </div>
    </div>

    <div class="grid grid-cols-2 gap-6">
        <div>
            <h2 class="font-semibold mb-3">Task Terbaru</h2>
            @forelse($recentTasks as $task)
                <div class="bg-white p-3 rounded border mb-2 text-sm">{{ $task->title }}</div>
            @empty
                <p class="text-gray-500 text-sm">Belum ada task.</p>
            @endforelse
        </div>
        <div>
            <h2 class="font-semibold mb-3">Kategori</h2>
            @foreach($categories as $cat)
                <div class="flex justify-between bg-white p-3 rounded border mb-2 text-sm">
                    <span>{{ $cat->name }}</span>
                    <span class="text-gray-500">{{ $cat->tasks_count }} task</span>
                </div>
            @endforeach
        </div>
    </div>
@endsection
```

---

## Langkah 5: Search & Filter

Update `TaskController@index`:

```php
public function index(Request $request)
{
    $query = auth()->user()->tasks()->with('category')->latest();

    if ($request->filled('search')) {
        $query->where('title', 'like', '%' . $request->search . '%');
    }

    if ($request->filled('category')) {
        $query->where('category_id', $request->category);
    }

    if ($request->filled('status')) {
        $query->where('is_done', $request->status === 'done');
    }

    $tasks = $query->paginate(10)->withQueryString();
    $categories = auth()->user()->categories()->get();

    return view('tasks.index', compact('tasks', 'categories'));
}
```

Tambah form filter di `tasks/index.blade.php`:

```blade
<form method="GET" class="flex gap-3 mb-6">
    <input type="text" name="search" value="{{ request('search') }}"
           placeholder="Cari task..." class="border rounded px-3 py-2 text-sm">
    <select name="category" class="border rounded px-3 py-2 text-sm">
        <option value="">Semua Kategori</option>
        @foreach($categories as $cat)
            <option value="{{ $cat->id }}" {{ request('category') == $cat->id ? 'selected' : '' }}>
                {{ $cat->name }}
            </option>
        @endforeach
    </select>
    <select name="status" class="border rounded px-3 py-2 text-sm">
        <option value="">Semua Status</option>
        <option value="done" {{ request('status') === 'done' ? 'selected' : '' }}>Selesai</option>
        <option value="pending" {{ request('status') === 'pending' ? 'selected' : '' }}>Belum</option>
    </select>
    <button type="submit" class="bg-gray-800 text-white px-4 py-2 rounded text-sm">Filter</button>
</form>
```

Tambah dropdown kategori di form create/edit task.

---

## Langkah 6: CRUD Kategori (Ringkas)

Implement `CategoryController` resource — mirip CRUD task:
- Route: `Route::resource('categories', CategoryController::class);`
- View: `categories/index`, `create`, `edit`
- Validasi: `name` required, unique per user

---

## Langkah 7: README Project

Buat **`README.md`** di root project `task-manager/`:

```markdown
# Task Manager

Aplikasi manajemen task — project belajar Laravel.

## Fitur
- Register & Login
- CRUD Task per user
- Kategori task
- Dashboard ringkasan
- Search & filter

## Tech Stack
- Laravel 12, PHP 8.2+
- Blade + Tailwind CSS
- SQLite / MySQL
- Laravel Breeze

## Cara Menjalankan
\`\`\`bash
composer install
cp .env.example .env
php artisan key:generate
touch database/database.sqlite
php artisan migrate --seed
npm install && npm run dev
php artisan serve
\`\`\`

## Demo
- Register akun → buat task → assign kategori → lihat dashboard
```

---

## Latihan Modul 08

1. Dashboard tampil statistik benar per user
2. Filter kategori & status berfungsi
3. Search by judul case-insensitive
4. CRUD kategori lengkap
5. Badge kategori tampil di list task
6. Final README project

---

## Git — Final Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/08-project-akhir

git add .
git commit -m "feat: dashboard, kategori, search filter, dan readme project"
git push -u origin modul/08-project-akhir
```

Setelah merge PR, pastikan repo GitHub punya:
- README project yang lengkap
- Riwayat commit per modul (lihat `git log --oneline`)
- App yang bisa di-clone & dijalankan orang lain

---

## Checklist Project Selesai

### Fitur
- [ ] Register, login, logout
- [ ] CRUD task (per user)
- [ ] CRUD kategori
- [ ] Dashboard statistik
- [ ] Search & filter task
- [ ] Layout & navbar konsisten

### Git
- [ ] Commit history rapi (1 commit/PR per modul)
- [ ] README project di GitHub
- [ ] Repo bisa di-clone & jalan oleh orang lain

### Demo
- [ ] Bisa demo alur: register → buat task → kategori → dashboard
- [ ] Filter & search berfungsi live

---

## Selamat!

Kamu sudah menyelesaikan **Task Manager App** — project Laravel lengkap dari nol.

### Riwayat perjalanan

| Modul | Milestone |
|---|---|
| 01 | Laravel setup |
| 02 | Git & GitHub |
| 03 | Routing & controller |
| 04 | Blade template |
| 05 | Database |
| 06 | CRUD task |
| 07 | Authentication |
| 08 | Dashboard & kategori |

### Lanjutan (opsional)

- Upload file/gambar
- API JSON dengan Laravel Sanctum
- Email notification task deadline
- PHPUnit feature test
- Deploy ke server (Laravel Forge / VPS)

---

**Modul sebelumnya:** [07 — Authentication](../07-authentication/README.md)  
**Kembali ke:** [Panduan Utama](../README.md)
