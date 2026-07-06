# Modul 07 — Authentication

Menambahkan login & register ke **Task Manager**. Task milik masing-masing user.

**Estimasi waktu:** 2 hari  
**Prasyarat:** [Modul 06 — CRUD Task](../06-crud-task/README.md)

---

## Tujuan Modul

- [ ] Laravel Breeze terinstall
- [ ] Register, login, logout berfungsi
- [ ] Route task terproteksi middleware `auth`
- [ ] Task terhubung ke user (`user_id`)
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
Login / Register / Logout
Middleware auth di route tasks
Kolom user_id di tabel tasks
Relasi User hasMany Task
```

---

## Langkah 1: Install Laravel Breeze

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install && npm run dev
php artisan migrate
```

Breeze otomatis membuat halaman auth, dashboard, dan layout baru.

---

## Langkah 2: Uji Auth

1. Buka `/register` — daftar akun baru
2. Setelah register → redirect ke `/dashboard`
3. Logout → login kembali di `/login`
4. Coba buka `/tasks` tanpa login → harus redirect ke login

---

## Langkah 3: Proteksi Route

**`routes/web.php`:**

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use App\Http\Controllers\ProfileController;
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Route;

// Publik
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');

// Butuh login
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', function () {
        return view('dashboard');
    })->name('dashboard');

    Route::resource('tasks', TaskController::class);
});

// Profile (dari Breeze)
Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});

require __DIR__.'/auth.php';
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

> Jika ada data lama di `tasks`, hapus dulu: `php artisan migrate:fresh --seed`

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

public function show(Task $task)
{
    $this->authorizeTask($task);
    return view('tasks.show', compact('task'));
}

// Tambahkan di class:
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

```blade
@auth
    <span class="text-sm text-gray-600">Halo, {{ auth()->user()->name }}</span>
    <a href="{{ route('tasks.index') }}">Tasks</a>
    <a href="{{ route('dashboard') }}">Dashboard</a>
    <form method="POST" action="{{ route('logout') }}" class="inline">
        @csrf
        <button type="submit" class="text-sm text-red-600">Logout</button>
    </form>
@else
    <a href="{{ route('login') }}">Login</a>
    <a href="{{ route('register') }}">Register</a>
@endauth
```

---

## Langkah 6: Redirect Setelah Login

Edit `app/Http/Controllers/Auth/AuthenticatedSessionController.php`:

```php
return redirect()->intended(route('tasks.index', absolute: false));
```

---

## Latihan Modul 07

1. Register 2 akun berbeda — pastikan task user A tidak tampil di user B
2. Update seeder: assign task ke user pertama
3. Buat halaman dashboard sederhana: total task, selesai, belum
4. Coba akses `/tasks/1/edit` milik user lain → harus 403
5. Tampilkan nama user di halaman detail task

---

## Git — Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/07-authentication

git add .
git commit -m "feat: authentication dengan breeze dan task per user"
git push -u origin modul/07-authentication
```

---

## Checklist Selesai

- [ ] Breeze terinstall, register & login jalan
- [ ] `/tasks` redirect ke login jika belum auth
- [ ] Task punya `user_id`, hanya owner yang bisa akses
- [ ] Navbar tampilkan nama user + logout
- [ ] Commit & push ke GitHub

---

**Modul sebelumnya:** [06 — CRUD Task](../06-crud-task/README.md)  
**Modul berikutnya:** [08 — Project Akhir](../08-project-akhir/README.md)
