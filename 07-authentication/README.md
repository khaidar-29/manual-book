# Modul 07 — Authentication (Manual)

Modul ketujuh: menambahkan **login, register, dan logout** ke **Task Manager** secara **manual** — tanpa Laravel UI, tanpa Breeze, tanpa npm. Hanya **Auth facade** bawaan Laravel + Bootstrap CDN.

**Estimasi waktu:** 2 hari  
**Prasyarat:** [Modul 06 — CRUD Task](../06-crud-task/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Membuat authentication manual dengan Auth facade
- [ ] Membuat `LoginController` dan `RegisterController` sendiri
- [ ] Membuat view login & register dengan Bootstrap CDN
- [ ] Register: validasi, hash password, auto login
- [ ] Login: `Auth::attempt`, redirect ke tasks
- [ ] Logout: `Auth::logout`, invalidate session
- [ ] Proteksi route dengan middleware `auth`
- [ ] Menghubungkan task ke user (`user_id`)
- [ ] Filter task hanya milik user yang login
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
app/Http/Controllers/Auth/LoginController.php
app/Http/Controllers/Auth/RegisterController.php
resources/views/auth/login.blade.php
resources/views/auth/register.blade.php
Kolom user_id di tabel tasks
Relasi User hasMany Task, Task belongsTo User
Middleware auth di route tasks
Update navbar (@auth / @guest)
```

> **TIDAK pakai:** `composer require laravel/ui`, Breeze, Jetstream, Fortify, npm, Vite, Tailwind.

---

## Mengapa Auth Manual?

Package seperti Laravel UI/Breeze mempercepat setup, tapi:

- Menambah dependency dan kompleksitas
- Sering butuh npm/Vite untuk asset
- Kamu tidak belajar **cara kerja auth di balik layar**

Dengan auth manual, kamu paham alur lengkap: validasi → hash password → session → middleware → proteksi data.

---

## Konsep: Authentication vs Authorization

| Istilah | Arti | Contoh |
|---|---|---|
| **Authentication** | "Siapa kamu?" | Login dengan email + password |
| **Authorization** | "Boleh akses apa?" | Hanya owner task yang bisa edit |

Modul ini fokus authentication + authorization dasar (task per user).

---

## Konsep: Session-Based Auth

Laravel auth bawaan pakai **session**:

```
1. User submit login form
2. Laravel cek email + password di database
3. Jika valid → buat session (cookie di browser)
4. Request berikutnya → Laravel baca session → tahu user siapa
5. Logout → hapus session
```

Tidak perlu JWT atau token API untuk aplikasi web tradisional.

---

## Konsep: Auth Facade

**Facade** = pintu akses ke service Laravel dengan sintaks statis.

```php
use Illuminate\Support\Facades\Auth;

Auth::attempt(['email' => $email, 'password' => $password]);
Auth::check();        // true jika sudah login
Auth::user();         // object User yang login
Auth::id();           // id user yang login
Auth::logout();       // keluar
```

Juga tersedia helper global:

```php
auth()->user();
auth()->id();
auth()->check();
```

Dan directive Blade:

```blade
@auth
    {{ auth()->user()->name }}
@endauth

@guest
    <a href="{{ route('login') }}">Login</a>
@endguest
```

---

## Konsep: Hash Password

Password **tidak pernah** disimpan plain text di database.

```php
use Illuminate\Support\Facades\Hash;

// Saat register
$user->password = Hash::make($request->password);

// Saat login — Auth::attempt otomatis membandingkan hash
Auth::attempt(['email' => $email, 'password' => $password]);
```

`Hash::make()` menggunakan bcrypt — one-way encryption.

---

## Konsep: Middleware auth

**Middleware** = filter yang dijalankan sebelum request masuk controller.

```php
Route::middleware('auth')->group(function () {
    Route::resource('tasks', TaskController::class);
});
```

Jika user belum login → redirect otomatis ke `/login`.

---

## Langkah 1: Buat Controller Auth

Buat folder dan controller:

```bash
mkdir -p app/Http/Controllers/Auth
php artisan make:controller Auth/RegisterController
php artisan make:controller Auth/LoginController
```

---

## Langkah 2: RegisterController

Edit `app/Http/Controllers/Auth/RegisterController.php`:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;

class RegisterController extends Controller
{
    /**
     * GET /register — Tampilkan form register.
     */
    public function create()
    {
        return view('auth.register');
    }

    /**
     * POST /register — Proses pendaftaran user baru.
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name'                  => 'required|string|max:255',
            'email'                 => 'required|email|max:255|unique:users,email',
            'password'              => 'required|string|min:8|confirmed',
        ], [
            'name.required'     => 'Nama wajib diisi.',
            'email.required'    => 'Email wajib diisi.',
            'email.email'       => 'Format email tidak valid.',
            'email.unique'      => 'Email sudah terdaftar.',
            'password.required' => 'Password wajib diisi.',
            'password.min'      => 'Password minimal 8 karakter.',
            'password.confirmed'=> 'Konfirmasi password tidak cocok.',
        ]);

        $user = User::create([
            'name'     => $validated['name'],
            'email'    => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        // Auto login setelah register
        Auth::login($user);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Selamat datang, ' . $user->name . '! Akun berhasil dibuat.');
    }
}
```

**Penjelasan baris penting:**

| Baris | Penjelasan |
|---|---|
| `'email' => 'unique:users,email'` | Email harus unik di tabel users |
| `'password' => 'confirmed'` | Wajib ada field `password_confirmation` |
| `Hash::make($validated['password'])` | Enkripsi password sebelum simpan |
| `Auth::login($user)` | Langsung login setelah register |
| `redirect()->route('tasks.index')` | Redirect ke halaman tasks |

---

## Langkah 3: LoginController

Edit `app/Http/Controllers/Auth/LoginController.php`:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * GET /login — Tampilkan form login.
     */
    public function create()
    {
        return view('auth.login');
    }

    /**
     * POST /login — Proses login user.
     */
    public function store(Request $request)
    {
        $credentials = $request->validate([
            'email'    => 'required|email',
            'password' => 'required|string',
        ], [
            'email.required'    => 'Email wajib diisi.',
            'password.required' => 'Password wajib diisi.',
        ]);

        $remember = $request->boolean('remember');

        if (Auth::attempt($credentials, $remember)) {
            $request->session()->regenerate();

            return redirect()
                ->intended(route('tasks.index'))
                ->with('success', 'Selamat datang kembali, ' . Auth::user()->name . '!');
        }

        return back()
            ->withInput($request->only('email'))
            ->withErrors([
                'email' => 'Email atau password salah.',
            ]);
    }

    /**
     * POST /logout — Keluar dari aplikasi.
     */
    public function destroy(Request $request)
    {
        Auth::logout();

        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect()
            ->route('login')
            ->with('success', 'Anda berhasil logout.');
    }
}
```

**Penjelasan baris penting:**

| Baris | Penjelasan |
|---|---|
| `Auth::attempt($credentials, $remember)` | Cek email+password, buat session jika valid |
| `$request->session()->regenerate()` | Cegah session fixation attack |
| `redirect()->intended(...)` | Redirect ke URL yang dimaksud sebelum login |
| `withInput($request->only('email'))` | Isi ulang email (bukan password!) |
| `Auth::logout()` | Hapus auth dari session |
| `session()->invalidate()` | Hapus seluruh data session |
| `regenerateToken()` | Token CSRF baru setelah logout |

---

## Langkah 4: Routes Auth

Edit `routes/web.php`:

```php
<?php

use App\Http\Controllers\Auth\LoginController;
use App\Http\Controllers\Auth\RegisterController;
use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Route;

// ── Halaman publik (tanpa login) ──
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');

// ── Auth: guest only (sudah login → redirect) ──
Route::middleware('guest')->group(function () {
    Route::get('/register', [RegisterController::class, 'create'])->name('register');
    Route::post('/register', [RegisterController::class, 'store']);

    Route::get('/login', [LoginController::class, 'create'])->name('login');
    Route::post('/login', [LoginController::class, 'store']);
});

// ── Logout (butuh login) ──
Route::post('/logout', [LoginController::class, 'destroy'])
    ->middleware('auth')
    ->name('logout');

// ── Halaman butuh login ──
Route::middleware('auth')->group(function () {
    Route::resource('tasks', TaskController::class);
});
```

Verifikasi route:

```bash
php artisan route:list --name=login
php artisan route:list --name=register
php artisan route:list --name=logout
php artisan route:list --name=tasks
```

---

## Langkah 5: View Register

Buat folder dan file:

```bash
mkdir -p resources/views/auth
```

Buat `resources/views/auth/register.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Register')

@section('content')
    <div class="row justify-content-center">
        <div class="col-md-6 col-lg-5">
            <div class="card shadow-sm">
                <div class="card-body p-4">
                    <h1 class="h3 fw-bold text-center mb-1">Daftar Akun</h1>
                    <p class="text-muted text-center small mb-4">Buat akun Task Manager baru</p>

                    <form action="{{ url('/register') }}" method="POST" novalidate>
                        @csrf

                        {{-- Nama --}}
                        <div class="mb-3">
                            <label for="name" class="form-label fw-semibold">Nama Lengkap</label>
                            <input type="text"
                                   name="name"
                                   id="name"
                                   value="{{ old('name') }}"
                                   class="form-control @error('name') is-invalid @enderror"
                                   placeholder="Nama kamu"
                                   autofocus>
                            @error('name')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>

                        {{-- Email --}}
                        <div class="mb-3">
                            <label for="email" class="form-label fw-semibold">Email</label>
                            <input type="email"
                                   name="email"
                                   id="email"
                                   value="{{ old('email') }}"
                                   class="form-control @error('email') is-invalid @enderror"
                                   placeholder="email@example.com">
                            @error('email')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>

                        {{-- Password --}}
                        <div class="mb-3">
                            <label for="password" class="form-label fw-semibold">Password</label>
                            <input type="password"
                                   name="password"
                                   id="password"
                                   class="form-control @error('password') is-invalid @enderror"
                                   placeholder="Minimal 8 karakter">
                            @error('password')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>

                        {{-- Konfirmasi Password --}}
                        <div class="mb-4">
                            <label for="password_confirmation" class="form-label fw-semibold">
                                Konfirmasi Password
                            </label>
                            <input type="password"
                                   name="password_confirmation"
                                   id="password_confirmation"
                                   class="form-control"
                                   placeholder="Ulangi password">
                        </div>

                        <button type="submit" class="btn btn-primary w-100 mb-3">
                            Daftar Sekarang
                        </button>

                        <p class="text-center small text-muted mb-0">
                            Sudah punya akun?
                            <a href="{{ route('login') }}" class="text-decoration-none">Login di sini</a>
                        </p>
                    </form>
                </div>
            </div>
        </div>
    </div>
@endsection
```

---

## Langkah 6: View Login

Buat `resources/views/auth/login.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Login')

@section('content')
    <div class="row justify-content-center">
        <div class="col-md-6 col-lg-5">
            <div class="card shadow-sm">
                <div class="card-body p-4">
                    <h1 class="h3 fw-bold text-center mb-1">Login</h1>
                    <p class="text-muted text-center small mb-4">Masuk ke Task Manager</p>

                    <form action="{{ url('/login') }}" method="POST" novalidate>
                        @csrf

                        {{-- Email --}}
                        <div class="mb-3">
                            <label for="email" class="form-label fw-semibold">Email</label>
                            <input type="email"
                                   name="email"
                                   id="email"
                                   value="{{ old('email') }}"
                                   class="form-control @error('email') is-invalid @enderror"
                                   placeholder="email@example.com"
                                   autofocus>
                            @error('email')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>

                        {{-- Password --}}
                        <div class="mb-3">
                            <label for="password" class="form-label fw-semibold">Password</label>
                            <input type="password"
                                   name="password"
                                   id="password"
                                   class="form-control @error('password') is-invalid @enderror"
                                   placeholder="Password kamu">
                            @error('password')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>

                        {{-- Remember me --}}
                        <div class="mb-4 form-check">
                            <input type="checkbox"
                                   name="remember"
                                   id="remember"
                                   class="form-check-input"
                                   {{ old('remember') ? 'checked' : '' }}>
                            <label for="remember" class="form-check-label">
                                Ingat saya
                            </label>
                        </div>

                        <button type="submit" class="btn btn-primary w-100 mb-3">
                            Login
                        </button>

                        <p class="text-center small text-muted mb-0">
                            Belum punya akun?
                            <a href="{{ route('register') }}" class="text-decoration-none">Daftar di sini</a>
                        </p>
                    </form>
                </div>
            </div>
        </div>
    </div>
@endsection
```

---

## Langkah 7: Update Navbar

Edit `resources/views/partials/navbar.blade.php` — ganti bagian menu kanan:

```blade
<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
    <div class="container">
        <a class="navbar-brand fw-bold text-primary" href="{{ route('home') }}">
            📋 Task Manager
        </a>

        <button class="navbar-toggler" type="button"
                data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav me-auto">
                <li class="nav-item">
                    <a class="nav-link {{ request()->routeIs('home') ? 'active fw-semibold' : '' }}"
                       href="{{ route('home') }}">Beranda</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link {{ request()->routeIs('about') ? 'active fw-semibold' : '' }}"
                       href="{{ route('about') }}">Tentang</a>
                </li>
                @auth
                    <li class="nav-item">
                        <a class="nav-link {{ request()->routeIs('tasks.*') ? 'active fw-semibold' : '' }}"
                           href="{{ route('tasks.index') }}">Tasks</a>
                    </li>
                @endauth
            </ul>

            <ul class="navbar-nav align-items-lg-center gap-lg-2">
                @auth
                    <li class="nav-item">
                        <span class="nav-link text-muted small py-lg-0">
                            Halo, <strong>{{ auth()->user()->name }}</strong>
                        </span>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link text-danger" href="#"
                           onclick="event.preventDefault(); document.getElementById('logout-form').submit();">
                            Logout
                        </a>
                        <form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
                            @csrf
                        </form>
                    </li>
                @else
                    <li class="nav-item">
                        <a class="nav-link {{ request()->routeIs('login') ? 'active fw-semibold' : '' }}"
                           href="{{ route('login') }}">Login</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link btn btn-primary btn-sm text-white px-3 {{ request()->routeIs('register') ? 'active' : '' }}"
                           href="{{ route('register') }}">Register</a>
                    </li>
                @endauth
            </ul>
        </div>
    </div>
</nav>
```

**Penjelasan:**

| Sintaks | Fungsi |
|---|---|
| `@auth` / `@endauth` | Blok hanya tampil jika sudah login |
| `@else` / `@endauth` | Blok jika belum login (guest) |
| `auth()->user()->name` | Nama user yang login |
| Logout via form POST | Logout harus POST (bukan link GET) |

---

## Langkah 8: Migration user_id di Tasks

Setiap task harus milik satu user. Tambah kolom `user_id`:

```bash
php artisan make:migration add_user_id_to_tasks_table
```

Edit file migration:

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
            $table->foreignId('user_id')
                  ->after('id')
                  ->constrained()
                  ->cascadeOnDelete();
        });
    }

    public function down(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->dropForeign(['user_id']);
            $table->dropColumn('user_id');
        });
    }
};
```

**Penjelasan:**

| Method | Fungsi |
|---|---|
| `foreignId('user_id')` | Kolom BIGINT UNSIGNED |
| `->constrained()` | Foreign key ke tabel `users(id)` |
| `->cascadeOnDelete()` | Hapus user → hapus semua task-nya |

Jalankan migration:

```bash
php artisan migrate
```

> **Jika error** karena sudah ada data task tanpa `user_id`, reset database:

```bash
php artisan migrate:fresh --seed
```

---

## Langkah 9: Relasi Model

### Task Model

Edit `app/Models/Task.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Task extends Model
{
    protected $fillable = [
        'user_id',
        'title',
        'description',
        'is_done',
    ];

    protected $casts = [
        'is_done' => 'boolean',
    ];

    /**
     * Task belongs to one User.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

### User Model

Edit `app/Models/User.php` — tambah relasi:

```php
use Illuminate\Database\Eloquent\Relations\HasMany;

/**
 * User has many Tasks.
 */
public function tasks(): HasMany
{
    return $this->hasMany(Task::class);
}
```

**Penjelasan relasi:**

```
User (1) ────── hasMany ──────> Task (N)
Task (N) ────── belongsTo ────> User (1)
```

Contoh penggunaan:

```php
$user = auth()->user();
$user->tasks;                    // Collection semua task user
$user->tasks()->count();         // Jumlah task
auth()->user()->tasks()->create([...]);  // Buat task untuk user ini
$task->user->name;               // Nama pemilik task
```

---

## Langkah 10: Update TaskController

Edit `app/Http/Controllers/TaskController.php` — filter task per user dan proteksi ownership:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function index()
    {
        $tasks = auth()->user()
            ->tasks()
            ->latest()
            ->paginate(10);

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
            'description' => 'nullable|string|max:1000',
        ]);

        $validated['is_done'] = $request->boolean('is_done');

        auth()->user()->tasks()->create($validated);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil ditambahkan!');
    }

    public function show(Task $task)
    {
        $this->authorizeTask($task);

        return view('tasks.show', compact('task'));
    }

    public function edit(Task $task)
    {
        $this->authorizeTask($task);

        return view('tasks.edit', compact('task'));
    }

    public function update(Request $request, Task $task)
    {
        $this->authorizeTask($task);

        $validated = $request->validate([
            'title'       => 'required|string|min:3|max:255',
            'description' => 'nullable|string|max:1000',
        ]);

        $validated['is_done'] = $request->boolean('is_done');

        $task->update($validated);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil diupdate!');
    }

    public function destroy(Task $task)
    {
        $this->authorizeTask($task);

        $task->delete();

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil dihapus!');
    }

    /**
     * Pastikan task milik user yang sedang login.
     * Jika bukan → HTTP 403 Forbidden.
     */
    private function authorizeTask(Task $task): void
    {
        if ($task->user_id !== auth()->id()) {
            abort(403, 'Anda tidak berhak mengakses task ini.');
        }
    }
}
```

**Perubahan penting dari Modul 06:**

| Sebelum | Sesudah |
|---|---|
| `Task::latest()->get()` | `auth()->user()->tasks()->latest()->get()` |
| `Task::create($validated)` | `auth()->user()->tasks()->create($validated)` |
| Tidak ada proteksi | `authorizeTask()` di show/edit/update/destroy |

---

## Langkah 11: Update TaskSeeder

Edit `database/seeders/TaskSeeder.php` — assign task ke user:

```php
<?php

namespace Database\Seeders;

use App\Models\Task;
use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class TaskSeeder extends Seeder
{
    public function run(): void
    {
        // Buat user demo jika belum ada
        $user = User::firstOrCreate(
            ['email' => 'demo@taskmanager.test'],
            [
                'name'     => 'Demo User',
                'password' => Hash::make('password123'),
            ]
        );

        $tasks = [
            ['title' => 'Setup project Laravel', 'description' => 'Install & konfigurasi environment', 'is_done' => true],
            ['title' => 'Pelajari Routing & Controller', 'description' => 'Memahami alur MVC', 'is_done' => true],
            ['title' => 'Buat layout Blade', 'description' => 'Layout, navbar, partial', 'is_done' => true],
            ['title' => 'Hubungkan ke database', 'description' => 'Migration, model, seeder', 'is_done' => false],
            ['title' => 'Buat CRUD task', 'description' => 'Tambah, edit, hapus task', 'is_done' => false],
        ];

        foreach ($tasks as $task) {
            $user->tasks()->create($task);
        }
    }
}
```

Reset dan seed ulang:

```bash
php artisan migrate:fresh --seed
```

Akun demo: `demo@taskmanager.test` / `password123`

---

## Langkah 12: Uji Authentication

```bash
php artisan serve
```

### Checklist uji manual

| # | Aksi | Hasil yang diharapkan |
|---|---|---|
| 1 | Buka `/tasks` tanpa login | Redirect ke `/login` |
| 2 | Buka `/register` | Form register tampil |
| 3 | Register dengan email duplikat | Error "Email sudah terdaftar" |
| 4 | Register password tidak cocok | Error "Konfirmasi password tidak cocok" |
| 5 | Register valid | Auto login → redirect `/tasks` + alert |
|  6 | Logout | Redirect ke `/login` |
| 7 | Login dengan password salah | Error "Email atau password salah" |
| 8 | Login valid | Redirect `/tasks` + alert selamat datang |
| 9 | Register user B, buat task | Task muncul di user B saja |
| 10 | Login user A | Task user B tidak tampil |
| 11 | Akses `/tasks/{id}` milik user lain | Halaman 403 Forbidden |
| 12 | Navbar tampil "Halo, {nama}" | Saat login |
| 13 | Navbar tampil Login/Register | Saat logout |

---

## Alur Lengkap: Register → Login → Task

```
REGISTER:
Browser → POST /register
       → RegisterController@store
       → Validasi (name, email unique, password confirmed)
       → User::create + Hash::make
       → Auth::login($user)
       → Redirect /tasks

LOGIN:
Browser → POST /login
       → LoginController@store
       → Auth::attempt(email, password)
       → Jika valid: session regenerate → redirect /tasks
       → Jika gagal: back + error

PROTEKSI TASK:
Browser → GET /tasks
       → Middleware auth (cek session)
       → Jika belum login → redirect /login
       → Jika login → TaskController@index
       → auth()->user()->tasks() → hanya task milik user
```

---

## Latihan Modul 07

### Latihan 1: Dua User Terpisah

1. Register akun "Alice" dan "Bob"
2. Login sebagai Alice, buat 3 task
3. Logout, login sebagai Bob, buat 2 task
4. Pastikan Alice hanya lihat 3 task-nya, Bob hanya 2 task-nya

### Latihan 2: Halaman Profil Sederhana

Buat route `/profile` (middleware auth) yang menampilkan:
- Nama user
- Email
- Jumlah total task
- Jumlah task selesai

```php
Route::get('/profile', function () {
    $user = auth()->user();
    return view('profile', [
        'user'  => $user,
        'total' => $user->tasks()->count(),
        'done'  => $user->tasks()->where('is_done', true)->count(),
    ]);
})->middleware('auth')->name('profile');
```

### Latihan 3: Redirect Intended

1. Logout
2. Buka langsung `/tasks/1/edit` di browser
3. Harus redirect ke `/login`
4. Login → harus redirect kembali ke `/tasks/1/edit` (bukan `/tasks`)

Ini sudah ditangani `redirect()->intended()` di LoginController.

### Latihan 4: Validasi Password Kuat

Tambah rule validasi password:

```php
'password' => ['required', 'string', 'min:8', 'confirmed', 'regex:/[A-Z]/', 'regex:/[0-9]/'],
```

Pesan custom: "Password harus mengandung minimal 1 huruf besar dan 1 angka."

### Latihan 5: Middleware di Controller

Alternatif middleware di route, terapkan di controller:

```php
public function __construct()
{
    $this->middleware('auth');
}
```

Hapus `Route::middleware('auth')->group()` dari routes, pindahkan ke constructor TaskController.

### Latihan 6 (Bonus): Forgot Password

Pelajari `Password::sendResetLink()` bawaan Laravel. Buat halaman `/forgot-password` sederhana (opsional, tidak wajib modul ini).

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| Redirect loop login | Middleware auth + guest bentrok | Pastikan route login pakai `middleware('guest')` |
| `/tasks` bisa diakses tanpa login | Route tidak diproteksi | Wrap dengan `Route::middleware('auth')->group()` |
| Task user A muncul di user B | Query masih `Task::all()` | Ganti ke `auth()->user()->tasks()` |
| Error FK constraint saat migrate | Data task lama tanpa user_id | `php artisan migrate:fresh --seed` |
| Login selalu gagal | Password tidak di-hash saat register | Pastikan `Hash::make()` di RegisterController |
| 419 saat logout | Form logout tanpa `@csrf` | Tambahkan `@csrf` di form logout navbar |
| `Auth::attempt` return false | Email/password salah atau user tidak ada | Cek tabel users di MySQL |
| Register error "email unique" padahal baru | Email sudah ada dari seeder/testing | Hapus user lama atau pakai email lain |
| Session hilang setiap refresh | `SESSION_DRIVER=array` di .env | Set `SESSION_DRIVER=file` atau `database` |
| Halaman login plain tanpa style | View tidak `@extends('layouts.app')` | Pastikan extends layout dengan Bootstrap CDN |

---

## Checklist Selesai

Centang sebelum lanjut ke Modul 08:

- [ ] `RegisterController` — validasi, Hash::make, Auth::login
- [ ] `LoginController` — Auth::attempt, logout, session regenerate
- [ ] View `auth/login.blade.php` dan `auth/register.blade.php` dengan Bootstrap CDN
- [ ] Route auth terdaftar (login, register, logout)
- [ ] Middleware `auth` proteksi route tasks
- [ ] Middleware `guest` proteksi route login/register
- [ ] Kolom `user_id` ada di tabel tasks dengan foreign key
- [ ] Relasi User-Task berfungsi
- [ ] TaskController filter per user + authorizeTask()
- [ ] Navbar tampil @auth / @guest dengan benar
- [ ] Logout via POST form berfungsi
- [ ] User A tidak bisa akses task user B (403)
- [ ] **TIDAK** install laravel/ui, breeze, atau npm

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: authentication login register manual dan task per user"
git push
```

---

## Ringkasan Modul 07

| Topik | Yang dipelajari |
|---|---|
| Auth Facade | attempt, login, logout, check, user |
| Hash | Password encryption dengan bcrypt |
| Middleware | auth (wajib login), guest (wajib belum login) |
| Session | regenerate, invalidate, remember me |
| Relasi | User hasMany Task, Task belongsTo User |
| Authorization | authorizeTask() — cek ownership |
| Blade Auth | @auth, @guest, auth()->user() |
| Foreign Key | user_id constrained cascadeOnDelete |

---

**Modul sebelumnya:** [06 — CRUD Task](../06-crud-task/README.md)  
**Modul berikutnya:** [08 — Project Akhir](../08-project-akhir/README.md)
