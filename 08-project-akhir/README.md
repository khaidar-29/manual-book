# Modul 08 — Project Akhir

Modul terakhir: menyelesaikan **Task Manager** — dashboard statistik, CRUD kategori, search & filter task. App siap demo dan deploy.

**Estimasi waktu:** 2–3 hari  
**Prasyarat:** [Modul 07 — Authentication](../07-authentication/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Membuat dashboard dengan statistik task (Bootstrap cards)
- [ ] Membuat CRUD kategori per user
- [ ] Menghubungkan task ke kategori (relasi)
- [ ] Implementasi search & filter di halaman tasks
- [ ] Menulis README project lengkap
- [ ] Demo aplikasi end-to-end
- [ ] Final commit & push ke GitHub

**Yang ditambahkan ke project:**

```
Tabel categories + relasi task belongsTo category
DashboardController + view dashboard
CategoryController + views categories/*
Search & filter di TaskController@index
README.md project lengkap di root task-manager/
```

---

## Gambaran App Akhir

Setelah modul ini, Task Manager punya fitur lengkap:

```
┌─────────────────────────────────────────────────────┐
│  📋 Task Manager          Halo, Budi  [Logout]      │
├─────────────────────────────────────────────────────┤
│  Beranda │ Tentang │ Tasks │ Dashboard │ Kategori   │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │ Total: 12│  │ Selesai:8│  │ Pending:4│          │
│  └──────────┘  └──────────┘  └──────────┘          │
│                                                     │
│  [Cari task...] [Kategori ▼] [Status ▼] [Filter]   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │ Task 1          [Work]  [Selesai]  [Edit]   │   │
│  │ Task 2          [Personal] [Belum] [Edit]   │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## Langkah 1: Migration Categories

```bash
php artisan make:model Category -m
php artisan make:controller CategoryController --resource
```

Edit migration `database/migrations/xxxx_create_categories_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('color')->default('primary');
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
};
```

**Penjelasan kolom `color`:**

Disimpan sebagai string nama warna Bootstrap: `primary`, `success`, `danger`, `warning`, `info`, `secondary`. Dipakai untuk class badge: `badge bg-{{ $category->color }}`.

---

## Langkah 2: Migration category_id di Tasks

```bash
php artisan make:migration add_category_id_to_tasks_table
```

Edit migration:

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
            $table->foreignId('category_id')
                  ->nullable()
                  ->after('is_done')
                  ->constrained()
                  ->nullOnDelete();
        });
    }

    public function down(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->dropForeign(['category_id']);
            $table->dropColumn('category_id');
        });
    }
};
```

Jalankan migration:

```bash
php artisan migrate
```

---

## Langkah 3: Model & Relasi

### Category Model

Edit `app/Models/Category.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Category extends Model
{
    protected $fillable = ['name', 'color', 'user_id'];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function tasks(): HasMany
    {
        return $this->hasMany(Task::class);
    }
}
```

### Update Task Model

Tambah di `app/Models/Task.php`:

```php
use Illuminate\Database\Eloquent\Relations\BelongsTo;

protected $fillable = [
    'user_id',
    'category_id',  // tambahkan
    'title',
    'description',
    'is_done',
];

public function category(): BelongsTo
{
    return $this->belongsTo(Category::class);
}
```

### Update User Model

Tambah di `app/Models/User.php`:

```php
public function categories(): HasMany
{
    return $this->hasMany(Category::class);
}
```

**Diagram relasi lengkap:**

```
User (1) ──hasMany──> Task (N)
User (1) ──hasMany──> Category (N)
Category (1) ──hasMany──> Task (N)
Task (N) ──belongsTo──> User (1)
Task (N) ──belongsTo──> Category (1, nullable)
```

---

## Langkah 4: CategorySeeder

```bash
php artisan make:seeder CategorySeeder
```

Edit `database/seeders/CategorySeeder.php`:

```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class CategorySeeder extends Seeder
{
    public function run(): void
    {
        $user = User::where('email', 'demo@taskmanager.test')->first();

        if (!$user) {
            return;
        }

        $categories = [
            ['name' => 'Work',     'color' => 'primary'],
            ['name' => 'Personal', 'color' => 'success'],
            ['name' => 'Study',    'color' => 'warning'],
        ];

        foreach ($categories as $category) {
            $user->categories()->create($category);
        }
    }
}
```

Daftarkan di `DatabaseSeeder.php`:

```php
$this->call([
    TaskSeeder::class,
    CategorySeeder::class,
]);
```

---

## Langkah 5: DashboardController

```bash
php artisan make:controller DashboardController
```

Edit `app/Http/Controllers/DashboardController.php`:

```php
<?php

namespace App\Http\Controllers;

class DashboardController extends Controller
{
    public function index()
    {
        $user = auth()->user();

        $total   = $user->tasks()->count();
        $done    = $user->tasks()->where('is_done', true)->count();
        $pending = $total - $done;

        $recentTasks = $user->tasks()
            ->with('category')
            ->latest()
            ->take(5)
            ->get();

        $categories = $user->categories()
            ->withCount('tasks')
            ->orderByDesc('tasks_count')
            ->get();

        return view('dashboard', compact(
            'total',
            'done',
            'pending',
            'recentTasks',
            'categories'
        ));
    }
}
```

**Penjelasan query:**

| Query | Fungsi |
|---|---|
| `$user->tasks()->count()` | Total task user |
| `->where('is_done', true)->count()` | Task selesai |
| `->with('category')` | Eager load — hindari N+1 query |
| `->take(5)->get()` | 5 task terbaru |
| `->withCount('tasks')` | Hitung task per kategori |

Tambah route di `routes/web.php` (di dalam group `auth`):

```php
Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
```

---

## Langkah 6: View Dashboard

Buat `resources/views/dashboard.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Dashboard')

@section('content')
    <div class="d-flex justify-content-between align-items-center mb-4">
        <div>
            <h1 class="h2 fw-bold mb-1">Dashboard</h1>
            <p class="text-muted mb-0 small">Ringkasan task kamu, {{ auth()->user()->name }}</p>
        </div>
        <a href="{{ route('tasks.create') }}" class="btn btn-primary btn-sm">+ Task Baru</a>
    </div>

    {{-- Statistik Cards --}}
    <div class="row g-3 mb-4">
        <div class="col-md-4">
            <div class="card text-center shadow-sm border-0">
                <div class="card-body py-4">
                    <p class="display-5 fw-bold text-primary mb-1">{{ $total }}</p>
                    <p class="text-muted small mb-0">Total Task</p>
                </div>
            </div>
        </div>
        <div class="col-md-4">
            <div class="card text-center shadow-sm border-0">
                <div class="card-body py-4">
                    <p class="display-5 fw-bold text-success mb-1">{{ $done }}</p>
                    <p class="text-muted small mb-0">Selesai</p>
                </div>
            </div>
        </div>
        <div class="col-md-4">
            <div class="card text-center shadow-sm border-0">
                <div class="card-body py-4">
                    <p class="display-5 fw-bold text-warning mb-1">{{ $pending }}</p>
                    <p class="text-muted small mb-0">Belum Selesai</p>
                </div>
            </div>
        </div>
    </div>

    <div class="row g-4">
        {{-- Task Terbaru --}}
        <div class="col-md-6">
            <div class="card shadow-sm">
                <div class="card-header bg-white fw-semibold">
                    Task Terbaru
                </div>
                <div class="card-body p-0">
                    @forelse($recentTasks as $task)
                        <div class="d-flex justify-content-between align-items-center px-3 py-2 border-bottom">
                            <div>
                                <a href="{{ route('tasks.show', $task) }}"
                                   class="text-decoration-none fw-semibold small {{ $task->is_done ? 'text-muted text-decoration-line-through' : '' }}">
                                    {{ $task->title }}
                                </a>
                                @if($task->category)
                                    <span class="badge bg-{{ $task->category->color }} ms-1" style="font-size: 0.65rem;">
                                        {{ $task->category->name }}
                                    </span>
                                @endif
                            </div>
                            <span class="badge {{ $task->is_done ? 'bg-success' : 'bg-warning text-dark' }}" style="font-size: 0.65rem;">
                                {{ $task->is_done ? 'Selesai' : 'Belum' }}
                            </span>
                        </div>
                    @empty
                        <p class="text-muted small p-3 mb-0">Belum ada task.</p>
                    @endforelse
                </div>
            </div>
        </div>

        {{-- Kategori --}}
        <div class="col-md-6">
            <div class="card shadow-sm">
                <div class="card-header bg-white fw-semibold d-flex justify-content-between align-items-center">
                    <span>Kategori</span>
                    <a href="{{ route('categories.index') }}" class="btn btn-outline-primary btn-sm">Kelola</a>
                </div>
                <div class="card-body p-0">
                    @forelse($categories as $cat)
                        <div class="d-flex justify-content-between align-items-center px-3 py-2 border-bottom">
                            <span>
                                <span class="badge bg-{{ $cat->color }}">{{ $cat->name }}</span>
                            </span>
                            <span class="text-muted small">{{ $cat->tasks_count }} task</span>
                        </div>
                    @empty
                        <p class="text-muted small p-3 mb-0">
                            Belum ada kategori.
                            <a href="{{ route('categories.create') }}">Buat sekarang</a>
                        </p>
                    @endforelse
                </div>
            </div>
        </div>
    </div>
@endsection
```

Update navbar — tambah link Dashboard dan Kategori (di dalam `@auth`):

```blade
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('dashboard') ? 'active fw-semibold' : '' }}"
       href="{{ route('dashboard') }}">Dashboard</a>
</li>
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('categories.*') ? 'active fw-semibold' : '' }}"
       href="{{ route('categories.index') }}">Kategori</a>
</li>
```

---

## Langkah 7: Search & Filter Task

Update method `index()` di `TaskController`:

```php
public function index(Request $request)
{
    $query = auth()->user()
        ->tasks()
        ->with('category')
        ->latest();

    // Search by judul
    if ($request->filled('search')) {
        $query->where('title', 'like', '%' . $request->search . '%');
    }

    // Filter by kategori
    if ($request->filled('category')) {
        $query->where('category_id', $request->category);
    }

    // Filter by status
    if ($request->filled('status')) {
        if ($request->status === 'done') {
            $query->where('is_done', true);
        } elseif ($request->status === 'pending') {
            $query->where('is_done', false);
        }
    }

    $tasks = $query->paginate(10)->withQueryString();
    $categories = auth()->user()->categories()->orderBy('name')->get();

    return view('tasks.index', compact('tasks', 'categories'));
}
```

**Penjelasan:**

| Bagian | Fungsi |
|---|---|
| `$request->filled('search')` | Cek parameter ada dan tidak kosong |
| `where('title', 'like', '%...%')` | Pencarian partial match |
| `->withQueryString()` | Pagination pertahankan parameter filter |
| `->with('category')` | Load relasi kategori sekaligus |

Tambah form filter di `resources/views/tasks/index.blade.php` (di atas tabel):

```blade
{{-- Search & Filter --}}
<form method="GET" action="{{ route('tasks.index') }}" class="card shadow-sm mb-4">
    <div class="card-body py-3">
        <div class="row g-2 align-items-end">
            <div class="col-md-4">
                <label class="form-label small fw-semibold mb-1">Cari Task</label>
                <input type="text"
                       name="search"
                       value="{{ request('search') }}"
                       placeholder="Ketik judul task..."
                       class="form-control form-control-sm">
            </div>
            <div class="col-md-3">
                <label class="form-label small fw-semibold mb-1">Kategori</label>
                <select name="category" class="form-select form-select-sm">
                    <option value="">Semua Kategori</option>
                    @foreach($categories as $cat)
                        <option value="{{ $cat->id }}"
                            {{ request('category') == $cat->id ? 'selected' : '' }}>
                            {{ $cat->name }}
                        </option>
                    @endforeach
                </select>
            </div>
            <div class="col-md-3">
                <label class="form-label small fw-semibold mb-1">Status</label>
                <select name="status" class="form-select form-select-sm">
                    <option value="">Semua Status</option>
                    <option value="done" {{ request('status') === 'done' ? 'selected' : '' }}>
                        Selesai
                    </option>
                    <option value="pending" {{ request('status') === 'pending' ? 'selected' : '' }}>
                        Belum Selesai
                    </option>
                </select>
            </div>
            <div class="col-md-2 d-flex gap-1">
                <button type="submit" class="btn btn-dark btn-sm flex-grow-1">Filter</button>
                <a href="{{ route('tasks.index') }}" class="btn btn-outline-secondary btn-sm">Reset</a>
            </div>
        </div>
    </div>
</form>
```

Tampilkan badge kategori di kolom judul tabel:

```blade
@if($task->category)
    <span class="badge bg-{{ $task->category->color }} ms-1" style="font-size: 0.7rem;">
        {{ $task->category->name }}
    </span>
@endif
```

---

## Langkah 8: Dropdown Kategori di Form Task

Update method `create()` dan `edit()` di TaskController:

```php
public function create()
{
    $categories = auth()->user()->categories()->orderBy('name')->get();

    return view('tasks.create', compact('categories'));
}

public function edit(Task $task)
{
    $this->authorizeTask($task);

    $categories = auth()->user()->categories()->orderBy('name')->get();

    return view('tasks.edit', compact('task', 'categories'));
}
```

Update validasi di `store()` dan `update()`:

```php
$validated = $request->validate([
    'title'       => 'required|string|min:3|max:255',
    'description' => 'nullable|string|max:1000',
    'category_id' => 'nullable|exists:categories,id',
]);
```

Tambah dropdown di form create/edit (setelah field deskripsi):

```blade
<div class="mb-3">
    <label for="category_id" class="form-label fw-semibold">Kategori</label>
    <select name="category_id" id="category_id" class="form-select">
        <option value="">— Tanpa Kategori —</option>
        @foreach($categories as $cat)
            <option value="{{ $cat->id }}"
                {{ old('category_id', $task->category_id ?? '') == $cat->id ? 'selected' : '' }}>
                {{ $cat->name }}
            </option>
        @endforeach
    </select>
</div>
```

---

## Langkah 9: CRUD Kategori (Panduan Ringkas)

Implementasi CRUD kategori mengikuti pola yang sama dengan CRUD task di Modul 06.

### Route

Tambah di group `auth` di `routes/web.php`:

```php
Route::resource('categories', CategoryController::class);
```

### CategoryController

```php
<?php

namespace App\Http\Controllers;

use App\Models\Category;
use Illuminate\Http\Request;

class CategoryController extends Controller
{
    public function index()
    {
        $categories = auth()->user()
            ->categories()
            ->withCount('tasks')
            ->orderBy('name')
            ->get();

        return view('categories.index', compact('categories'));
    }

    public function create()
    {
        return view('categories.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name'  => 'required|string|max:100',
            'color' => 'required|in:primary,success,danger,warning,info,secondary',
        ]);

        auth()->user()->categories()->create($validated);

        return redirect()
            ->route('categories.index')
            ->with('success', 'Kategori berhasil ditambahkan!');
    }

    public function edit(Category $category)
    {
        $this->authorizeCategory($category);

        return view('categories.edit', compact('category'));
    }

    public function update(Request $request, Category $category)
    {
        $this->authorizeCategory($category);

        $validated = $request->validate([
            'name'  => 'required|string|max:100',
            'color' => 'required|in:primary,success,danger,warning,info,secondary',
        ]);

        $category->update($validated);

        return redirect()
            ->route('categories.index')
            ->with('success', 'Kategori berhasil diupdate!');
    }

    public function destroy(Category $category)
    {
        $this->authorizeCategory($category);

        $category->delete();

        return redirect()
            ->route('categories.index')
            ->with('success', 'Kategori berhasil dihapus!');
    }

    private function authorizeCategory(Category $category): void
    {
        if ($category->user_id !== auth()->id()) {
            abort(403, 'Anda tidak berhak mengakses kategori ini.');
        }
    }
}
```

### View categories/index.blade.php

```blade
@extends('layouts.app')

@section('title', 'Kategori')

@section('content')
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h1 class="h2 fw-bold mb-0">Kategori</h1>
        <a href="{{ route('categories.create') }}" class="btn btn-primary btn-sm">+ Tambah Kategori</a>
    </div>

    @if($categories->isEmpty())
        <div class="alert alert-info">Belum ada kategori.</div>
    @else
        <div class="row g-3">
            @foreach($categories as $cat)
                <div class="col-md-4">
                    <div class="card shadow-sm">
                        <div class="card-body d-flex justify-content-between align-items-center">
                            <div>
                                <span class="badge bg-{{ $cat->color }} fs-6">{{ $cat->name }}</span>
                                <p class="text-muted small mb-0 mt-1">{{ $cat->tasks_count }} task</p>
                            </div>
                            <div class="d-flex gap-1">
                                <a href="{{ route('categories.edit', $cat) }}"
                                   class="btn btn-warning btn-sm">Edit</a>
                                <form action="{{ route('categories.destroy', $cat) }}"
                                      method="POST" class="d-inline"
                                      onsubmit="return confirm('Hapus kategori? Task terkait akan kehilangan kategori.')">
                                    @csrf
                                    @method('DELETE')
                                    <button class="btn btn-danger btn-sm">Hapus</button>
                                </form>
                            </div>
                        </div>
                    </div>
                </div>
            @endforeach
        </div>
    @endif
@endsection
```

### View categories/create.blade.php & edit.blade.php

Form sederhana dengan field:
- **Nama kategori** — input text (`form-control`)
- **Warna** — select dropdown (`form-select`) dengan opsi: Primary, Success, Danger, Warning, Info, Secondary

```blade
<select name="color" class="form-select">
    <option value="primary"   {{ old('color', $category->color ?? '') === 'primary'   ? 'selected' : '' }}>Biru (Primary)</option>
    <option value="success"   {{ old('color', $category->color ?? '') === 'success'   ? 'selected' : '' }}>Hijau (Success)</option>
    <option value="danger"    {{ old('color', $category->color ?? '') === 'danger'    ? 'selected' : '' }}>Merah (Danger)</option>
    <option value="warning"   {{ old('color', $category->color ?? '') === 'warning'   ? 'selected' : '' }}>Kuning (Warning)</option>
    <option value="info"      {{ old('color', $category->color ?? '') === 'info'      ? 'selected' : '' }}>Cyan (Info)</option>
    <option value="secondary" {{ old('color', $category->color ?? '') === 'secondary' ? 'selected' : '' }}>Abu (Secondary)</option>
</select>
```

---

## Langkah 10: README Project

Buat atau overwrite **`README.md`** di root project Laravel (`task-manager/README.md`):

```markdown
# Task Manager

Aplikasi web manajemen task (to-do list) — project pembelajaran Laravel dari nol.

Setiap user bisa mendaftar, login, lalu mengelola task dan kategori miliknya sendiri.

## Fitur

- **Authentication** — Register, login, logout (implementasi manual)
- **CRUD Task** — Tambah, lihat, edit, hapus task dengan validasi
- **Kategori** — Organisasi task per kategori dengan warna badge
- **Dashboard** — Statistik total, selesai, belum selesai
- **Search & Filter** — Cari task by judul, filter kategori & status
- **Responsif** — Bootstrap 5 via CDN, mobile-friendly

## Tech Stack

| Layer | Teknologi |
|---|---|
| Backend | Laravel 12, PHP 8.2+ |
| Database | MySQL 8 |
| Frontend | Blade Template + Bootstrap 5 (CDN) |
| Auth | Manual (Auth facade bawaan Laravel) |
| Version Control | Git + GitHub |

> **Catatan:** Project ini **tidak memerlukan npm, Node.js, Vite, atau Tailwind**.
> Bootstrap di-load via CDN. Tidak ada build step frontend.

## Prasyarat

| Software | Versi Minimum |
|---|---|
| PHP | 8.2+ dengan extension: pdo_mysql, mbstring, xml, curl, zip, bcmath |
| Composer | 2.x |
| MySQL | 8.x |
| Git | Terbaru |

Cek versi:

\`\`\`bash
php -v
composer -V
mysql --version
\`\`\`

## Instalasi & Menjalankan

### 1. Clone repository

\`\`\`bash
git clone https://github.com/USERNAME/task-manager.git
cd task-manager
\`\`\`

### 2. Install dependency PHP

\`\`\`bash
composer install
\`\`\`

### 3. Setup environment

\`\`\`bash
cp .env.example .env
php artisan key:generate
\`\`\`

### 4. Konfigurasi database MySQL

Edit file `.env`:

\`\`\`env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager
DB_USERNAME=root
DB_PASSWORD=
\`\`\`

Buat database di MySQL:

\`\`\`bash
mysql -u root -p
\`\`\`

\`\`\`sql
CREATE DATABASE task_manager CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EXIT;
\`\`\`

### 5. Migration & seed data

\`\`\`bash
php artisan migrate --seed
\`\`\`

### 6. Jalankan server

\`\`\`bash
php artisan serve
\`\`\`

Buka browser: **http://localhost:8000**

### 7. Akun demo (dari seeder)

| Email | Password |
|---|---|
| demo@taskmanager.test | password123 |

Atau register akun baru di `/register`.

## Struktur Database

\`\`\`
users
├── id, name, email, password, timestamps

categories
├── id, name, color, user_id (FK), timestamps

tasks
├── id, user_id (FK), category_id (FK, nullable)
├── title, description, is_done, timestamps
\`\`\`

## Alur Demo

1. Buka `/register` → buat akun baru
2. Setelah login → buat beberapa task di `/tasks`
3. Buat kategori di `/categories` (Work, Personal, Study)
4. Assign kategori ke task via form edit
5. Lihat dashboard di `/dashboard` — statistik & task terbaru
6. Coba search & filter di halaman tasks

## Troubleshooting

| Masalah | Solusi |
|---|---|
| `could not find driver` | Install/aktifkan PHP extension `pdo_mysql` |
| `Access denied for user` | Cek `DB_USERNAME` dan `DB_PASSWORD` di `.env` |
| `Unknown database task_manager` | Buat database: `CREATE DATABASE task_manager;` |
| `No application encryption key` | Jalankan: `php artisan key:generate` |
| Halaman tanpa style | Pastikan koneksi internet aktif (Bootstrap CDN) |
| Port 8000 sudah dipakai | `php artisan serve --port=8080` |

## Lisensi

Project pembelajaran — bebas digunakan untuk belajar.

## Author

[Nama Kamu] — [GitHub Profile URL]
```

Ganti `USERNAME` dan informasi author dengan data kamu.

---

## Langkah 11: Uji End-to-End

### Skenario demo lengkap

```
1. Register akun baru "Budi"
2. Login → redirect ke /tasks (kosong)
3. Buat kategori: Work (biru), Personal (hijau)
4. Buat 5 task, assign kategori ke masing-masing
5. Tandai 2 task sebagai selesai
6. Buka /dashboard → cek statistik: Total 5, Selesai 2, Pending 3
7. Search "laporan" → hanya task matching tampil
8. Filter kategori "Work" → hanya task Work tampil
9. Filter status "Selesai" → hanya 2 task selesai
10. Klik Reset → semua task tampil kembali
11. Logout → redirect /login
12. Login akun lain → task Budi tidak tampil
```

---

## Latihan Capstone Modul 08

Latihan capstone lebih menantang — gabungan semua modul.

### Capstone 1: Progress Bar Dashboard

Tambah progress bar Bootstrap di dashboard:

```blade
@php $percent = $total > 0 ? round(($done / $total) * 100) : 0; @endphp
<div class="progress mb-4" style="height: 20px;">
    <div class="progress-bar bg-success" style="width: {{ $percent }}%">
        {{ $percent }}% selesai
    </div>
</div>
```

### Capstone 2: Export Task ke CSV

Buat route `/tasks/export` yang download file CSV berisi semua task user:

```php
public function export()
{
    $tasks = auth()->user()->tasks()->with('category')->get();

    $csv = "Judul,Deskripsi,Status,Kategori,Dibuat\n";
    foreach ($tasks as $task) {
        $csv .= implode(',', [
            '"' . $task->title . '"',
            '"' . ($task->description ?? '') . '"',
            $task->is_done ? 'Selesai' : 'Belum',
            $task->category?->name ?? '-',
            $task->created_at->format('Y-m-d'),
        ]) . "\n";
    }

    return response($csv)
        ->header('Content-Type', 'text/csv')
        ->header('Content-Disposition', 'attachment; filename="tasks.csv"');
}
```

### Capstone 3: Due Date & Overdue Badge

1. Tambah kolom `due_date` (date, nullable) via migration
2. Tampilkan di list task
3. Badge merah "Terlambat" jika `due_date < today()` dan belum selesai

### Capstone 4: Halaman Welcome Post-Login

Buat middleware atau logic: setelah login pertama kali, redirect ke `/dashboard` (bukan `/tasks`). User returning tetap ke `/tasks`.

### Capstone 5: API JSON Endpoint

Buat route `/api/my-tasks` (middleware auth) yang return JSON:

```php
Route::get('/api/my-tasks', function () {
    return auth()->user()->tasks()->with('category')->latest()->get();
})->middleware('auth');
```

Uji di browser atau Postman — harus return JSON array task.

### Capstone 6: Deploy ke Hosting

Deploy app ke hosting PHP + MySQL (misalnya shared hosting, Railway, atau VPS):

1. Upload file project (tanpa folder `vendor/` — install via `composer install` di server)
2. Set `.env` production (APP_ENV=production, APP_DEBUG=false)
3. `php artisan migrate --seed --force`
4. Point domain ke folder `public/`

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| Dashboard statistik 0 semua | Query tidak filter user | Pastikan `auth()->user()->tasks()` |
| Filter tidak bekerja | Form method bukan GET | Pastikan `method="GET"` di form filter |
| Pagination hilang filter | Tidak pakai withQueryString | Tambahkan `->withQueryString()` |
| Badge kategori tidak muncul | Relasi belum di-load | Tambahkan `->with('category')` di query |
| Kategori user A muncul di user B | authorizeCategory tidak dipanggil | Panggil di edit/update/destroy |
| N+1 query lambat | Tidak eager load | Pakai `with('category')` dan `withCount('tasks')` |
| Dropdown kategori kosong | User belum buat kategori | Buat kategori dulu di `/categories` |
| Search case-sensitive | Collation MySQL | Normal di utf8mb4_unicode_ci (case-insensitive by default) |
| README npm instruction | Copy dari template lain | Hapus semua referensi npm — project ini CDN only |

---

## Checklist Project Selesai

### Fitur Wajib

- [ ] Register, login, logout (auth manual, tanpa Laravel UI)
- [ ] CRUD task lengkap dengan validasi (per user)
- [ ] CRUD kategori dengan warna badge
- [ ] Task bisa di-assign ke kategori
- [ ] Dashboard: statistik total, selesai, pending
- [ ] Dashboard: task terbaru & ringkasan kategori
- [ ] Search task by judul
- [ ] Filter task by kategori & status
- [ ] Navbar konsisten (@auth / @guest)
- [ ] Bootstrap 5 via CDN (tanpa npm/Vite)
- [ ] MySQL sebagai database (bukan SQLite)

### Dokumentasi & Git

- [ ] README.md project lengkap di root `task-manager/`
- [ ] `.env.example` tidak berisi secret nyata
- [ ] Commit history rapi (1 commit per modul)
- [ ] Repo GitHub bisa di-clone & dijalankan orang lain

### Demo

- [ ] Alur demo: register → task → kategori → dashboard → search
- [ ] Dua user tidak saling lihat data
- [ ] App tampil rapi di desktop dan mobile

---

## Git — Final Commit & Push

```bash
git add .
git commit -m "feat: dashboard kategori search filter dan readme project"
git push
```

Ini commit terakhir. Project Task Manager kamu sudah selesai!

---

## Selamat!

Kamu sudah menyelesaikan **Task Manager App** — project Laravel lengkap dari nol sampai siap demo.

### Riwayat Perjalanan

| Modul | Milestone | Commit message contoh |
|---|---|---|
| 01 | Laravel + MySQL setup | `chore: initial laravel setup` |
| 02 | Push ke GitHub | `docs: tambah readme project` |
| 03 | Route & controller | `feat: tambah route dan controller` |
| 04 | Blade + Bootstrap CDN | `feat: tambah layout blade bootstrap cdn` |
| 05 | Database & list task | `feat: tambah migration tasks dan halaman list` |
| 06 | CRUD task | `feat: crud task lengkap dengan validasi` |
| 07 | Auth manual | `feat: authentication login register manual` |
| 08 | Dashboard & kategori | `feat: dashboard kategori search filter readme` |

### Apa yang Sudah Kamu Kuasai

- Setup Laravel + MySQL dari nol
- MVC pattern: Route → Controller → Model → View
- Blade templating + Bootstrap CDN
- Database migration, model, seeder, Eloquent
- CRUD lengkap dengan validasi & flash message
- Authentication manual dengan Auth facade
- Relasi database (hasMany, belongsTo)
- Middleware auth & authorization
- Search, filter, pagination
- Git workflow dasar

### Lanjutan (Opsional)

| Topik | Deskripsi |
|---|---|
| File upload | Lampiran gambar/file di task |
| Email notification | Reminder task deadline via Mail |
| PHPUnit testing | Feature test CRUD & auth |
| Laravel Sanctum | API JSON + SPA/mobile client |
| Queue & Jobs | Background job untuk email |
| Deploy production | VPS, Laravel Forge, atau Railway |
| Policy & Gate | Authorization lebih granular |
| Livewire / Alpine.js | Interaktivitas tanpa full SPA |

---

**Modul sebelumnya:** [07 — Authentication](../07-authentication/README.md)  
**Kembali ke:** [Panduan Utama](../README.md)
