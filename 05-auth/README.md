# Modul 05 — Authentication

Modul kelima: menambahkan login, register, dan proteksi halaman.

**Estimasi waktu:** 2–3 hari  
**Prasyarat:** [Modul 04 — CRUD Task](../04-crud-task/README.md)

---

## Tujuan Pembelajaran

Setelah menyelesaikan modul ini, kamu sudah bisa:

- [ ] Menginstall Laravel Breeze untuk autentikasi
- [ ] Memahami halaman login dan register
- [ ] Memproteksi route agar hanya user login yang bisa akses
- [ ] Menampilkan info user di navbar
- [ ] Memahami middleware `auth` dan `guest`

---

## Apa itu Authentication?

Authentication (auth) memastikan hanya user yang sudah login bisa mengakses fitur tertentu.

```
User belum login → redirect ke /login
User sudah login → bisa akses /tasks, /dashboard, dll.
```

---

## Langkah 1: Install Laravel Breeze

Laravel Breeze menyediakan sistem login/register siap pakai dengan tampilan Tailwind CSS.

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install
npm run dev
php artisan migrate
```

Setelah install, Breeze otomatis membuat:
- Halaman `/login` dan `/register`
- Halaman `/dashboard`
- Route dan controller auth
- View auth di `resources/views/auth/`
- Layout `resources/views/layouts/app.blade.php` (bisa timpa layout lama)

---

## Langkah 2: Uji Register & Login

Jalankan server:

```bash
php artisan serve
```

1. Buka `http://localhost:8000/register`
2. Daftar akun baru (nama, email, password)
3. Setelah register, kamu otomatis login dan diarahkan ke `/dashboard`
4. Coba logout, lalu login kembali di `/login`

---

## Langkah 3: Proteksi Route Task

Edit `routes/web.php`:

```php
<?php

use App\Http\Controllers\ProfileController;
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Route;

// Route publik
Route::get('/', function () {
    return view('welcome');
});

// Route yang butuh login
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', function () {
        return view('dashboard');
    })->name('dashboard');

    Route::resource('tasks', TaskController::class);
});

// Route profile (dari Breeze)
Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});

require __DIR__.'/auth.php';
```

Sekarang coba buka `/tasks` tanpa login — harus redirect ke halaman login.

---

## Langkah 4: Update Navbar

Edit navbar agar menampilkan status login. Di `resources/views/layouts/navigation.blade.php` (dari Breeze) atau navbar kamu:

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

## Langkah 5: Hubungkan Task ke User (Opsional)

Agar setiap user hanya melihat task miliknya sendiri.

### Migration — tambah kolom user_id

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

### Model — tambah relasi

Di `app/Models/Task.php`:

```php
public function user()
{
    return $this->belongsTo(User::class);
}
```

Di `app/Models/User.php`:

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
    $validated = $request->validate([
        'title'       => 'required|string|max:255',
        'description' => 'nullable|string',
        'is_done'     => 'boolean',
    ]);

    $validated['is_done'] = $request->boolean('is_done');

    auth()->user()->tasks()->create($validated);

    return redirect()->route('tasks.index')
        ->with('success', 'Task berhasil ditambahkan!');
}
```

---

## Middleware yang Perlu Diketahui

| Middleware | Fungsi |
|---|---|
| `auth` | Hanya user yang sudah login |
| `guest` | Hanya user yang belum login |
| `verified` | Hanya user yang sudah verifikasi email |

Contoh penggunaan:

```php
// Satu route
Route::get('/tasks', [TaskController::class, 'index'])->middleware('auth');

// Sekelompok route
Route::middleware(['auth', 'verified'])->group(function () {
    Route::resource('tasks', TaskController::class);
});
```

---

## Cek User di Controller & View

```php
// Di Controller
$user = auth()->user();
$userId = auth()->id();
$isLoggedIn = auth()->check();
```

```blade
{{-- Di Blade --}}
@auth
    <p>Halo, {{ auth()->user()->name }}</p>
@endauth

@guest
    <a href="{{ route('login') }}">Silakan login</a>
@endguest
```

---

## Struktur File Auth dari Breeze

```
resources/views/
├── auth/
│   ├── login.blade.php
│   ├── register.blade.php
│   ├── forgot-password.blade.php
│   └── reset-password.blade.php
├── layouts/
│   ├── app.blade.php
│   ├── guest.blade.php
│   └── navigation.blade.php
└── profile/
    └── edit.blade.php

routes/
└── auth.php          ← Route login, register, logout
```

---

## Latihan

1. Proteksi semua route task — hanya user login yang bisa akses
2. Tampilkan nama user di navbar setelah login
3. Redirect ke `/tasks` (bukan `/dashboard`) setelah login berhasil
4. Buat halaman dashboard yang menampilkan ringkasan: total task, selesai, belum
5. (Opsional) Hubungkan task ke user — setiap user hanya lihat task sendiri

### Redirect setelah login

Edit `app/Providers/AppServiceProvider.php`:

```php
use Illuminate\Support\Facades\Route;

public function boot(): void
{
    Route::redirect('/dashboard', '/tasks');
}
```

Atau edit `app/Http/Controllers/Auth/AuthenticatedSessionController.php`:

```php
return redirect()->intended(route('tasks.index', absolute: false));
```

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| Halaman login tidak ada style | Jalankan `npm run dev` |
| Setelah login redirect ke `/dashboard` kosong | Normal, Breeze default. Customize redirect |
| Route `login` not defined | Pastikan `require __DIR__.'/auth.php'` ada di `web.php` |
| Migration error setelah Breeze | Jalankan `php artisan migrate` |

---

## Checklist Selesai

- [ ] Laravel Breeze terinstall
- [ ] Bisa register akun baru
- [ ] Bisa login dan logout
- [ ] Route `/tasks` terproteksi (redirect ke login jika belum auth)
- [ ] Nama user tampil di navbar
- [ ] Paham middleware `auth` dan `guest`
- [ ] (Opsional) Task terhubung ke user

---

## Selesai!

Selamat, kamu sudah menyelesaikan semua modul pembelajaran Laravel dasar:

| Modul | Topik |
|---|---|
| [01 — Setup](../01-setup/README.md) | Install & jalankan Laravel |
| [02 — MVC Dasar](../02-mvc-dasar/README.md) | Route, Controller, Blade |
| [03 — Database](../03-database/README.md) | Migration, Model, Seeder |
| [04 — CRUD Task](../04-crud-task/README.md) | Create, Read, Update, Delete |
| [05 — Authentication](../05-auth/README.md) | Login & Register |

### Lanjutan (opsional)

- Upload gambar
- Relasi database (kategori, tag)
- API dengan Laravel Sanctum
- Testing dengan PHPUnit
- Deploy ke server

Lihat juga [Panduan Lengkap](../README.md) untuk referensi detail setiap topik.
