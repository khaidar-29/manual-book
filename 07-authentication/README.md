# Modul 07 — Authentication

Menambahkan login & register ke **Task Manager** dengan **Laravel UI + Bootstrap**. Task milik masing-masing user.

**Estimasi waktu:** 2 hari  
**Prasyarat:** [Modul 06 — CRUD Task](../06-crud-task/README.md)

---

## Tujuan Modul

- [ ] Laravel UI (Bootstrap) terinstall
- [ ] Register, login, logout berfungsi
- [ ] Route task terproteksi middleware `auth`
- [ ] Task terhubung ke user (`user_id`)
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
Login / Register / Logout (Bootstrap)
Middleware auth di route tasks
Kolom user_id di tabel tasks
Relasi User hasMany Task
```

---

## Langkah 1: Install Laravel UI (Bootstrap)

```bash
composer require laravel/ui
php artisan ui bootstrap --auth
npm install && npm run build
php artisan migrate
```

Laravel UI otomatis membuat:
- Halaman `/login` dan `/register` dengan Bootstrap
- Controller auth
- View auth di `resources/views/auth/`

> **Penting:** Setelah install, sesuaikan layout auth agar konsisten dengan `layouts/app.blade.php` kamu, atau pakai layout bawaan Laravel UI.

---

## Langkah 2: Uji Auth

1. Buka `/register` — daftar akun baru
2. Setelah register → redirect ke `/home`
3. Logout → login kembali di `/login`
4. Coba buka `/tasks` tanpa login → harus redirect ke login

---

## Langkah 3: Proteksi Route

**`routes/web.php`:**

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Route;

Auth::routes();

// Publik
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');

// Butuh login
Route::middleware('auth')->group(function () {
    Route::get('/home', function () {
        return redirect()->route('tasks.index');
    })->name('home.auth');

    Route::get('/dashboard', function () {
        return view('dashboard');
    })->name('dashboard');

    Route::resource('tasks', TaskController::class);
});
```

---

## Langkah 4: Hubungkan Task ke User

### Migration

```bash
php artisan make:migration add_user_id_to_tasks_table
```

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->foreignId('user_id')->after('id')->constrained()->cascadeOnDelete();
    });
}
```

```bash
php artisan migrate
```

> Jika ada data lama di `tasks`, reset dulu: `php artisan migrate:fresh --seed`

### Model

**`app/Models/Task.php`:**

```php
public function user()
{
    return $this->belongsTo(User::class);
}
```

**`app/Models/User.php`:**

```php
public function tasks()
{
    return $this->hasMany(Task::class);
}
```

### Controller — filter & assign user

```php
public function index()
{
    $tasks = auth()->user()->tasks()->latest()->paginate(10);
    return view('tasks.index', compact('tasks'));
}

public function store(Request $request)
{
    // ... validasi ...
    auth()->user()->tasks()->create($validated);
    return redirect()->route('tasks.index')->with('success', 'Task berhasil ditambahkan!');
}

private function authorizeTask(Task $task): void
{
    if ($task->user_id !== auth()->id()) {
        abort(403);
    }
}
```

Panggil `$this->authorizeTask($task)` di method `show`, `edit`, `update`, `destroy`.

---

## Langkah 5: Update Navbar

**`resources/views/partials/navbar.blade.php`:**

```blade
<div class="navbar-nav ms-auto align-items-center gap-2">
    @auth
        <span class="nav-link text-muted small">Halo, {{ auth()->user()->name }}</span>
        <a class="nav-link" href="{{ route('tasks.index') }}">Tasks</a>
        <a class="nav-link" href="{{ route('dashboard') }}">Dashboard</a>
        <a class="nav-link text-danger" href="{{ route('logout') }}"
           onclick="event.preventDefault(); document.getElementById('logout-form').submit();">
            Logout
        </a>
        <form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
            @csrf
        </form>
    @else
        <a class="nav-link" href="{{ route('login') }}">Login</a>
        <a class="nav-link" href="{{ route('register') }}">Register</a>
    @endauth
</div>
```

---

## Langkah 6: Redirect Setelah Login

Edit `app/Providers/AppServiceProvider.php` atau `app/Http/Controllers/Auth/LoginController.php`:

**`LoginController.php`:**

```php
protected $redirectTo = '/tasks';
```

---

## Latihan Modul 07

1. Register 2 akun berbeda — pastikan task user A tidak tampil di user B
2. Update seeder: assign task ke user pertama
3. Buat halaman dashboard sederhana: total task, selesai, belum
4. Coba akses `/tasks/1/edit` milik user lain → harus 403
5. Sesuaikan halaman login/register agar konsisten dengan layout app

---

## Git — Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/07-authentication

git add .
git commit -m "feat: authentication laravel ui bootstrap dan task per user"
git push -u origin modul/07-authentication
```

---

## Checklist Selesai

- [ ] Laravel UI Bootstrap terinstall, register & login jalan
- [ ] `/tasks` redirect ke login jika belum auth
- [ ] Task punya `user_id`, hanya owner yang bisa akses
- [ ] Navbar tampilkan nama user + logout
- [ ] Commit & push ke GitHub

---

**Modul sebelumnya:** [06 — CRUD Task](../06-crud-task/README.md)  
**Modul berikutnya:** [08 — Project Akhir](../08-project-akhir/README.md)
