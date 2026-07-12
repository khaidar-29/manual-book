# Modul 11 — Middleware & Authorization

Modul kesebelas: menambahkan **role admin/user**, **middleware** proteksi route admin, dan **Policy** agar task hanya bisa diubah owner **atau** admin — semua di project **Task Manager** yang sama.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 10 — Relasi Lanjutan](../10-relasi-lanjutan/README.md)

> Stack tetap: **MySQL only**, **Bootstrap 5 CDN only**, auth **manual** (Modul 07).  
> Jangan install Laravel UI / Breeze / Spatie Permission untuk modul ini — kita bangun sendiri agar konsepnya jelas.

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Membedakan authentication vs authorization (ulang singkat + praktik)
- [ ] Menambah kolom `role` di users (`user` / `admin`) via migration
- [ ] Membuat seeder admin
- [ ] Membuat custom middleware `EnsureUserIsAdmin`
- [ ] Mendaftarkan middleware alias di `bootstrap/app.php` (Laravel 11/12)
- [ ] Melindungi route admin-only
- [ ] Membuat `TaskPolicy` (view/update/delete: owner ATAU admin)
- [ ] Memanggil authorize di `TaskController`
- [ ] Memahami Gate secara singkat
- [ ] Menampilkan badge Admin & menu berbeda di Bootstrap UI
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
Migration: add_role_to_users_table
UserSeeder / update DatabaseSeeder → akun admin
app/Http/Middleware/EnsureUserIsAdmin.php
Alias middleware 'admin' di bootstrap/app.php
Admin dashboard (stats global) ATAU daftar users
app/Policies/TaskPolicy.php
Authorize di TaskController
Update navbar: badge Admin, link menu admin
Helper isAdmin() di model User
```

---

## Konsep: Authentication vs Authorization (lagi, tapi lebih dalam)

| | Authentication | Authorization |
|---|---|---|
| Pertanyaan | Siapa kamu? | Kamu boleh melakukan apa? |
| Modul terkait | 07 (login/register) | **11 (middleware + policy)** |
| Contoh | Email + password benar | Hanya admin bisa lihat semua user |
| Tool Laravel | `Auth`, session, middleware `auth` | Middleware custom, Policy, Gate |

Alur request tipikal:

```
Request
  → middleware auth        (harus login)
  → middleware admin       (harus role admin)   ← opsional, khusus route admin
  → controller
      → $this->authorize() (Policy: boleh update task ini?)
  → bisnis logic
```

Middleware cocok untuk **batasan kasar per route/group**.  
Policy cocok untuk **aturan per model/instance** (task milik siapa).

---

## Bagian A — Kolom `role` di Users

### Langkah A1: Migration

```bash
php artisan make:migration add_role_to_users_table --table=users
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('role', 20)->default('user')->after('email');
            // nilai yang dipakai: 'user' | 'admin'
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('role');
        });
    }
};
```

```bash
php artisan migrate
```

> Kenapa `string` bukan `enum`? Lebih fleksibel di MySQL lintas versi; kita validasi di aplikasi (`in:user,admin`).

### Langkah A2: Update Model User

Edit `app/Models/User.php`:

```php
protected $fillable = [
    'name',
    'email',
    'password',
    'role',
];

/**
 * Cek apakah user adalah admin.
 */
public function isAdmin(): bool
{
    return $this->role === 'admin';
}
```

Di Blade nanti:

```blade
@if (auth()->user()->isAdmin())
    ...
@endif
```

### Langkah A3: Register tetap role `user`

Pastikan `RegisterController` **tidak** mengizinkan client mengirim `role=admin`.

```php
User::create([
    'name'     => $validated['name'],
    'email'    => $validated['email'],
    'password' => Hash::make($validated['password']),
    'role'     => 'user', // HARDCODE — jangan dari $request
]);
```

Jika `role` ada di `$fillable`, mass assignment dari request berbahaya. Lebih aman:

- Tetap hardcode saat register, **atau**
- Keluarkan `role` dari `$fillable` dan set eksplisit hanya di seeder/admin action.

### Langkah A4: Seeder Admin

```bash
php artisan make:seeder AdminUserSeeder
```

`database/seeders/AdminUserSeeder.php`:

```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class AdminUserSeeder extends Seeder
{
    public function run(): void
    {
        User::updateOrCreate(
            ['email' => 'admin@taskmanager.test'],
            [
                'name'     => 'Administrator',
                'password' => Hash::make('password'),
                'role'     => 'admin',
            ]
        );

        // Pastikan ada minimal 1 user biasa untuk testing
        User::updateOrCreate(
            ['email' => 'user@taskmanager.test'],
            [
                'name'     => 'User Biasa',
                'password' => Hash::make('password'),
                'role'     => 'user',
            ]
        );
    }
}
```

Panggil di `DatabaseSeeder`:

```php
public function run(): void
{
    $this->call([
        AdminUserSeeder::class,
        // seeder lain jika ada...
    ]);
}
```

Jalankan:

```bash
php artisan db:seed --class=AdminUserSeeder
```

Atau (hati-hati: hapus data):

```bash
php artisan migrate:fresh --seed
```

**Akun uji:**

| Email | Password | Role |
|---|---|---|
| `admin@taskmanager.test` | `password` | admin |
| `user@taskmanager.test` | `password` | user |

> Ganti password di production. Ini hanya untuk belajar lokal.

---

## Bagian B — Custom Middleware `EnsureUserIsAdmin`

### Langkah B1: Buat Middleware

Laravel 11/12:

```bash
php artisan make:middleware EnsureUserIsAdmin
```

Edit `app/Http/Middleware/EnsureUserIsAdmin.php`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsAdmin
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();

        if (!$user || !$user->isAdmin()) {
            abort(403, 'Akses khusus admin.');
        }

        return $next($request);
    }
}
```

**Penjelasan:**

1. `$request->user()` = user yang login (null jika guest)  
2. Jika bukan admin → **403 Forbidden**  
3. Jika admin → lanjut ke controller  

Middleware ini **mengasumsikan** user sudah login. Karena itu, di route kita gabungkan dengan `auth`:

```php
Route::middleware(['auth', 'admin'])->group(...)
```

### Langkah B2: Daftarkan Alias di `bootstrap/app.php`

Di Laravel 11/12, **tidak** ada `app/Http/Kernel.php` seperti Laravel 10. Alias middleware didaftarkan di `bootstrap/app.php`:

```php
<?php

use App\Http\Middleware\EnsureUserIsAdmin;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->alias([
            'admin' => EnsureUserIsAdmin::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions): void {
        //
    })->create();
```

Sekarang kamu bisa menulis `middleware('admin')` di route.

### Langkah B3: Route Admin

Buat controller:

```bash
php artisan make:controller Admin/DashboardController
```

`app/Http/Controllers/Admin/DashboardController.php`:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\Task;
use App\Models\User;
use App\Models\Category;
use App\Models\Tag;

class DashboardController extends Controller
{
    public function index()
    {
        $stats = [
            'users'      => User::count(),
            'admins'     => User::where('role', 'admin')->count(),
            'tasks'      => Task::withTrashed()->count(),
            'tasks_live' => Task::count(),
            'trashed'    => Task::onlyTrashed()->count(),
            'categories' => Category::count(),
            'tags'       => Tag::count(),
        ];

        $latestUsers = User::latest()->take(5)->get();
        $latestTasks = Task::with(['user', 'category'])->latest()->take(10)->get();

        return view('admin.dashboard', compact('stats', 'latestUsers', 'latestTasks'));
    }
}
```

Opsional — list users:

```bash
php artisan make:controller Admin/UserController
```

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\User;

class UserController extends Controller
{
    public function index()
    {
        $users = User::withCount('tasks')
            ->latest()
            ->paginate(15);

        return view('admin.users.index', compact('users'));
    }
}
```

Di `routes/web.php`:

```php
use App\Http\Controllers\Admin\DashboardController as AdminDashboardController;
use App\Http\Controllers\Admin\UserController as AdminUserController;

Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/', [AdminDashboardController::class, 'index'])->name('dashboard');
        Route::get('/users', [AdminUserController::class, 'index'])->name('users.index');
    });
```

Hasil:

| URL | Name | Siapa yang boleh |
|---|---|---|
| `/admin` | `admin.dashboard` | auth + admin |
| `/admin/users` | `admin.users.index` | auth + admin |

User biasa yang buka `/admin` → **403**.

### Langkah B4: View Admin Dashboard (Bootstrap)

`resources/views/admin/dashboard.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Admin Dashboard')

@section('content')
<h1 class="h3 mb-4">Admin Dashboard</h1>

<div class="row g-3 mb-4">
    <div class="col-md-3">
        <div class="border rounded p-3">
            <div class="text-muted small">Users</div>
            <div class="fs-3">{{ $stats['users'] }}</div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="border rounded p-3">
            <div class="text-muted small">Tasks (aktif)</div>
            <div class="fs-3">{{ $stats['tasks_live'] }}</div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="border rounded p-3">
            <div class="text-muted small">Trash</div>
            <div class="fs-3">{{ $stats['trashed'] }}</div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="border rounded p-3">
            <div class="text-muted small">Tags</div>
            <div class="fs-3">{{ $stats['tags'] }}</div>
        </div>
    </div>
</div>

<div class="row">
    <div class="col-lg-6 mb-4">
        <h2 class="h5">User terbaru</h2>
        <ul class="list-group">
            @foreach ($latestUsers as $user)
                <li class="list-group-item d-flex justify-content-between">
                    <span>{{ $user->name }} <small class="text-muted">{{ $user->email }}</small></span>
                    @if ($user->isAdmin())
                        <span class="badge text-bg-dark">Admin</span>
                    @else
                        <span class="badge text-bg-secondary">User</span>
                    @endif
                </li>
            @endforeach
        </ul>
        <a href="{{ route('admin.users.index') }}" class="btn btn-sm btn-outline-primary mt-2">Lihat semua users</a>
    </div>

    <div class="col-lg-6 mb-4">
        <h2 class="h5">Task terbaru (semua user)</h2>
        <ul class="list-group">
            @foreach ($latestTasks as $task)
                <li class="list-group-item">
                    <strong>{{ $task->title }}</strong>
                    <div class="small text-muted">
                        oleh {{ $task->user?->name ?? '-' }}
                        @if ($task->category)
                            · {{ $task->category->name }}
                        @endif
                    </div>
                </li>
            @endforeach
        </ul>
    </div>
</div>
@endsection
```

`resources/views/admin/users/index.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Kelola Users')

@section('content')
<div class="d-flex justify-content-between align-items-center mb-4">
    <h1 class="h3 mb-0">Users</h1>
    <a href="{{ route('admin.dashboard') }}" class="btn btn-outline-secondary">Admin Dashboard</a>
</div>

<table class="table table-hover align-middle">
    <thead>
        <tr>
            <th>Nama</th>
            <th>Email</th>
            <th>Role</th>
            <th>Tasks</th>
            <th>Bergabung</th>
        </tr>
    </thead>
    <tbody>
        @foreach ($users as $user)
            <tr>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
                <td>
                    @if ($user->isAdmin())
                        <span class="badge text-bg-dark">Admin</span>
                    @else
                        <span class="badge text-bg-secondary">User</span>
                    @endif
                </td>
                <td>{{ $user->tasks_count }}</td>
                <td>{{ $user->created_at?->format('d M Y') }}</td>
            </tr>
        @endforeach
    </tbody>
</table>

{{ $users->links() }}
@endsection
```

---

## Bagian C — TaskPolicy

Di Modul 07 kamu mungkin punya helper `authorizeTask()` manual. Policy adalah cara Laravel yang lebih rapi dan terpusat.

### Langkah C1: Generate Policy

```bash
php artisan make:policy TaskPolicy --model=Task
```

File: `app/Policies/TaskPolicy.php`

```php
<?php

namespace App\Policies;

use App\Models\Task;
use App\Models\User;

class TaskPolicy
{
    /**
     * Admin boleh semua aksi task.
     * Return true → izinkan; false → tolak; null → lanjut cek method lain.
     */
    public function before(User $user, string $ability): ?bool
    {
        if ($user->isAdmin()) {
            return true;
        }

        return null;
    }

    public function viewAny(User $user): bool
    {
        return true; // semua user login boleh lihat list (nanti di-filter query)
    }

    public function view(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }

    public function create(User $user): bool
    {
        return true;
    }

    public function update(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }

    public function delete(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }

    public function restore(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }

    public function forceDelete(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }
}
```

**Catatan `before()`:**  
Jika admin, langsung `return true` untuk **semua** ability. Method `view`/`update`/`delete` tidak perlu mengulang cek admin.

Tanpa `before()`, kamu bisa menulis:

```php
return $user->isAdmin() || $user->id === $task->user_id;
```

di setiap method — sama valid.

### Langkah C2: Registrasi Policy (Laravel 11/12)

Biasanya **auto-discovery** bekerja jika nama mengikuti konvensi `Task` → `TaskPolicy`.  
Jika tidak terdeteksi, daftar manual di `AppServiceProvider`:

```php
use App\Models\Task;
use App\Policies\TaskPolicy;
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::policy(Task::class, TaskPolicy::class);
}
```

### Langkah C3: Authorize di TaskController

Ganti / lengkapi pengecekan ownership:

```php
public function show(Task $task)
{
    $this->authorize('view', $task);

    $task->load(['category', 'tags', 'user']);

    return view('tasks.show', compact('task'));
}

public function edit(Task $task)
{
    $this->authorize('update', $task);

    $categories = auth()->user()->categories()->orderBy('name')->get();
    $tags = auth()->user()->tags()->orderBy('name')->get();

    // Admin mengedit task user lain: kategori/tag milik owner task
    if (auth()->user()->isAdmin() && $task->user_id !== auth()->id()) {
        $categories = $task->user->categories()->orderBy('name')->get();
        $tags = $task->user->tags()->orderBy('name')->get();
    }

    return view('tasks.edit', compact('task', 'categories', 'tags'));
}

public function update(Request $request, Task $task)
{
    $this->authorize('update', $task);

    // validasi + sync seperti Modul 09/10 ...
}

public function destroy(Task $task)
{
    $this->authorize('delete', $task);

    $task->delete();

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task dipindahkan ke trash.');
}
```

Alternatif sintaks:

```php
Gate::authorize('update', $task);
// atau
abort_unless(auth()->user()->can('update', $task), 403);
```

Di Blade:

```blade
@can('update', $task)
    <a href="{{ route('tasks.edit', $task) }}" class="btn btn-sm btn-primary">Edit</a>
@endcan

@can('delete', $task)
    <form action="{{ route('tasks.destroy', $task) }}" method="POST" class="d-inline"
          onsubmit="return confirm('Hapus task ini?')">
        @csrf
        @method('DELETE')
        <button class="btn btn-sm btn-danger">Hapus</button>
    </form>
@endcan
```

### Langkah C4: Index untuk Admin vs User

```php
public function index(Request $request)
{
    $this->authorize('viewAny', Task::class);

    if (auth()->user()->isAdmin()) {
        $query = Task::query()->with(['category', 'tags', 'user']);
    } else {
        $query = auth()->user()->tasks()->with(['category', 'tags']);
    }

    // search/filter Modul 08 tetap bisa diterapkan ke $query

    $tasks = $query->latest()->paginate(10)->withQueryString();

    return view('tasks.index', compact('tasks'));
}
```

Di index Blade, untuk admin tampilkan kolom pemilik:

```blade
@if (auth()->user()->isAdmin())
    <td>{{ $task->user?->name }}</td>
@endif
```

### Langkah C5: Soft delete restore/forceDelete

```php
public function restore(string $id)
{
    $task = Task::onlyTrashed()->findOrFail($id);
    $this->authorize('restore', $task);
    $task->restore();

    return back()->with('success', 'Task direstore.');
}

public function forceDelete(string $id)
{
    $task = Task::onlyTrashed()->findOrFail($id);
    $this->authorize('forceDelete', $task);

    if ($task->attachment) {
        \Illuminate\Support\Facades\Storage::disk('public')->delete($task->attachment);
    }

    $task->tags()->detach();
    $task->forceDelete();

    return back()->with('success', 'Task dihapus permanen.');
}
```

Untuk trash list admin:

```php
public function trash()
{
    $query = auth()->user()->isAdmin()
        ? Task::onlyTrashed()->with(['user', 'category', 'tags'])
        : Task::onlyTrashed()->where('user_id', auth()->id())->with(['category', 'tags']);

    $tasks = $query->latest('deleted_at')->paginate(10);

    return view('tasks.trash', compact('tasks'));
}
```

---

## Bagian D — Gate (Opsional, Singkat)

**Gate** = authorization untuk aksi yang **tidak** terikat satu model, atau aturan global.

Di `AppServiceProvider::boot()`:

```php
use Illuminate\Support\Facades\Gate;

Gate::define('access-admin', function (User $user) {
    return $user->isAdmin();
});
```

Pemakaian:

```php
if (Gate::allows('access-admin')) { ... }

Gate::authorize('access-admin');

@can('access-admin')
    <a href="{{ route('admin.dashboard') }}">Admin</a>
@endcan
```

Untuk Task Manager modul ini, **Policy + middleware sudah cukup**. Gate ditampilkan agar kamu tahu opsinya.

Perbandingan cepat:

| Tool | Cocok untuk |
|---|---|
| Middleware | Proteksi group route (`/admin/*`) |
| Policy | Aturan per model (`Task`, `Post`) |
| Gate | Aturan generik / non-model |

---

## Bagian E — Bootstrap UI: Badge & Menu by Role

Edit navbar di `resources/views/layouts/app.blade.php` (atau partial nav):

```blade
@auth
    <ul class="navbar-nav me-auto">
        <li class="nav-item">
            <a class="nav-link" href="{{ route('dashboard') }}">Dashboard</a>
        </li>
        <li class="nav-item">
            <a class="nav-link" href="{{ route('tasks.index') }}">Tasks</a>
        </li>
        <li class="nav-item">
            <a class="nav-link" href="{{ route('categories.index') }}">Kategori</a>
        </li>
        <li class="nav-item">
            <a class="nav-link" href="{{ route('tags.index') }}">Tags</a>
        </li>
        <li class="nav-item">
            <a class="nav-link" href="{{ route('tasks.trash') }}">Trash</a>
        </li>

        @can('access-admin')
            {{-- jika Gate belum dibuat, ganti dengan auth()->user()->isAdmin() --}}
        @endcan

        @if (auth()->user()->isAdmin())
            <li class="nav-item">
                <a class="nav-link" href="{{ route('admin.dashboard') }}">Admin</a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="{{ route('admin.users.index') }}">Users</a>
            </li>
        @endif
    </ul>

    <ul class="navbar-nav">
        <li class="nav-item d-flex align-items-center gap-2">
            <span class="navbar-text">
                Halo, {{ auth()->user()->name }}
                @if (auth()->user()->isAdmin())
                    <span class="badge text-bg-dark">Admin</span>
                @endif
            </span>
            <form action="{{ route('logout') }}" method="POST" class="d-inline">
                @csrf
                <button class="btn btn-sm btn-outline-secondary">Logout</button>
            </form>
        </li>
    </ul>
@endauth
```

**Prinsip UI:**  
Sembunyikan menu admin dari user biasa (UX). Tapi **jangan andalkan UI sebagai keamanan** — middleware & policy tetap wajib. User bisa ketik URL `/admin` manual.

---

## View Error 403 (opsional, bagus untuk UX)

`resources/views/errors/403.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Akses Ditolak')

@section('content')
<div class="text-center py-5">
    <h1 class="display-6">403</h1>
    <p class="text-muted">Kamu tidak punya izin untuk mengakses halaman ini.</p>
    <a href="{{ url()->previous() }}" class="btn btn-outline-secondary">Kembali</a>
    <a href="{{ route('home') }}" class="btn btn-primary">Beranda</a>
</div>
@endsection
```

---

## Uji Manual Skenario Authorization

| # | Aksi | User biasa | Admin |
|---|---|---|---|
| 1 | Login | OK | OK |
| 2 | Buka `/admin` | 403 | 200 + stats |
| 3 | Lihat task sendiri | OK | OK |
| 4 | Edit task sendiri | OK | OK |
| 5 | Edit task milik orang lain (URL langsung) | 403 | OK |
| 6 | Hapus task orang lain | 403 | OK (soft delete) |
| 7 | Menu Admin di navbar | Tidak tampil | Tampil + badge |
| 8 | Register user baru | Role selalu `user` | — |

Cara uji cepat task orang lain:

1. Login sebagai user A, catat ID task (mis. `/tasks/3/edit`)  
2. Logout, login sebagai user B  
3. Buka `/tasks/3/edit` → harus 403  
4. Login admin → harus bisa  

---

## Latihan

### Latihan 1: Middleware Log Admin Access

Buat middleware kedua yang hanya `Log::info('Admin access', [...])` lalu pasang di group admin bersama `admin`.

### Latihan 2: Promote User jadi Admin

Di `Admin\UserController`, tambah tombol "Jadikan Admin" / "Jadikan User" (PATCH) — hanya admin, dan **larang** admin mendemotion dirinya sendiri jika dia admin terakhir.

### Latihan 3: Policy Category & Tag

Buat `CategoryPolicy` dan `TagPolicy` dengan pola sama (owner atau admin).

### Latihan 4: Blade `@cannot`

Sembunyikan tombol edit jika `@cannot('update', $task)` dan tampilkan teks "Hanya pemilik / admin".

### Latihan 5: Guest vs Auth vs Admin Matrix

Dokumentasikan di README project-mu sebuah tabel: halaman mana yang `guest`, `auth`, `auth+admin`.

### Latihan 6: Ganti `before()` dengan cek eksplisit

Hapus method `before()`, tulis `$user->isAdmin() || $user->id === $task->user_id` di setiap method. Bandingkan keterbacaan.

### Latihan 7 (Bonus): Permission lebih dari 2 role

Tambah role `moderator` yang boleh `view` semua task tapi tidak boleh `forceDelete`. Perluas Policy.

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| Middleware `admin` tidak ketemu | Alias belum di `bootstrap/app.php` | Daftarkan `$middleware->alias([...])` |
| Admin tetap 403 | `isAdmin()` salah / role beda string | Cek DB: `role` harus tepat `admin` |
| User biasa masih bisa `/admin` | Route tanpa middleware | Pastikan `middleware(['auth','admin'])` |
| Policy tidak jalan | Nama salah / belum discover | Daftarkan `Gate::policy` di provider |
| `$this->authorize` error | Trait / controller base | Pastikan extends `Controller` bawaan yang punya `AuthorizesRequests` (Laravel 11: cek base controller) |
| 403 di semua task milik sendiri | `user_id` null / salah | Cek data task & auth id |
| Register jadi admin | `role` dari request | Hardcode `'user'`; jangan mass-assign role dari input |
| Navbar admin tampil tapi route 403 | UI vs server mismatch | Cek role di session/user terbaru; logout-login ulang |
| `Call to undefined method isAdmin()` | Method belum ditambah | Tambah `isAdmin()` di model User |
| Seed admin tidak jalan | Seeder belum dipanggil | `db:seed --class=AdminUserSeeder` |

### Catatan Laravel 11 Controller Trait

Jika `$this->authorize()` error, pastikan `app/Http/Controllers/Controller.php` mengandung:

```php
<?php

namespace App\Http\Controllers;

abstract class Controller
{
    //
}
```

Di Laravel 11+, trait authorization sering dipanggil lewat helper/`Illuminate\Foundation\Auth\Access\AuthorizesRequests`. Jika perlu:

```php
use Illuminate\Foundation\Auth\Access\AuthorizesRequests;

abstract class Controller
{
    use AuthorizesRequests;
}
```

Atau pakai `Gate::authorize('update', $task)` di method controller.

---

## Checklist Selesai

- [ ] Kolom `role` di users (default `user`) — MySQL
- [ ] `User::isAdmin()` tersedia
- [ ] Seeder admin + user biasa
- [ ] Register **tidak** bisa set role admin dari form
- [ ] Middleware `EnsureUserIsAdmin` dibuat
- [ ] Alias `admin` terdaftar di `bootstrap/app.php`
- [ ] Route `/admin` & `/admin/users` terproteksi
- [ ] View admin dashboard menampilkan statistik global
- [ ] `TaskPolicy` dengan view/update/delete (owner OR admin)
- [ ] Controller memakai `$this->authorize(...)` / Gate
- [ ] Blade `@can` / `isAdmin()` untuk tombol & menu
- [ ] Badge **Admin** di navbar
- [ ] User biasa kena 403 untuk task orang lain & `/admin`
- [ ] Admin bisa kelola / lihat lintas user sesuai desain
- [ ] Tidak pakai SQLite, npm, Vite, Tailwind, Laravel UI, Spatie
- [ ] Commit & push

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: role admin dan policy authorization"
git push
```

Tanpa branch baru, tanpa Pull Request, tanpa merge.

---

## Ringkasan Modul 11

| Topik | Yang dipelajari |
|---|---|
| Role | Kolom `role`, `isAdmin()`, seeder admin |
| Middleware | Custom `EnsureUserIsAdmin`, alias di `bootstrap/app.php` |
| Route group | `middleware(['auth','admin'])`, prefix `/admin` |
| Policy | `TaskPolicy`, `before()`, `authorize`, `@can` |
| Gate | Definisi singkat aturan non-model |
| UI | Badge Admin, menu kondisional Bootstrap |
| Keamanan | UI ≠ security; validasi server selalu wajib |

---

## Peta Fitur Task Manager sampai Modul 11

```
Auth manual (07)
  → Dashboard, kategori, search (08)
  → Upload lampiran (09)
  → Soft delete + tags M2M (10)
  → Role admin + Policy (11)  ← kamu di sini
  → Form Request & refactor (12)
```

---

**Modul sebelumnya:** [10 — Relasi Lanjutan](../10-relasi-lanjutan/README.md)  
**Modul berikutnya:** [12 — Form Request & Refactor](../12-form-request-refactor/README.md)
