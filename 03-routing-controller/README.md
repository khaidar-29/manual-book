# Modul 03 — Routing & Controller

Menambahkan route dan controller ke **Task Manager** — belum pakai database.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 02 — Git & GitHub](../02-git-github/README.md)

---

## Tujuan Modul

- [ ] Paham alur Route → Controller → View
- [ ] Halaman beranda & tentang jalan via controller
- [ ] Route punya nama (`->name()`)
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
GET /           → HomeController@index
GET /tentang    → PageController@about
```

---

## Konsep MVC

```
Browser → Route → Controller → View → HTML
```

Database **belum dipakai** di modul ini.

---

## Langkah 1: Buat Controller

```bash
php artisan make:controller HomeController
php artisan make:controller PageController
```

**`app/Http/Controllers/HomeController.php`:**

```php
<?php

namespace App\Http\Controllers;

class HomeController extends Controller
{
    public function index()
    {
        return view('home', [
            'title' => 'Task Manager',
            'tagline' => 'Kelola task harianmu dengan mudah.',
        ]);
    }
}
```

**`app/Http/Controllers/PageController.php`:**

```php
<?php

namespace App\Http\Controllers;

class PageController extends Controller
{
    public function about()
    {
        return view('about', [
            'title' => 'Tentang',
            'appName' => 'Task Manager',
            'version' => '1.0.0',
            'description' => 'Aplikasi manajemen task sederhana built with Laravel.',
        ]);
    }
}
```

---

## Langkah 2: Buat View Sederhana

View masih plain HTML — layout rapi nanti di Modul 04.

**`resources/views/home.blade.php`:**

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>{{ $title }}</title>
</head>
<body>
    <h1>{{ $title }}</h1>
    <p>{{ $tagline }}</p>
    <a href="{{ route('about') }}">Tentang Aplikasi</a>
</body>
</html>
```

**`resources/views/about.blade.php`:**

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>{{ $title }} — {{ $appName }}</title>
</head>
<body>
    <h1>{{ $title }}</h1>
    <p><strong>Aplikasi:</strong> {{ $appName }}</p>
    <p><strong>Versi:</strong> {{ $version }}</p>
    <p>{{ $description }}</p>
    <a href="{{ route('home') }}">← Kembali</a>
</body>
</html>
```

---

## Langkah 3: Daftarkan Route

**`routes/web.php`:**

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use Illuminate\Support\Facades\Route;

Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');
```

Cek route:

```bash
php artisan route:list
```

---

## Langkah 4: Latihan Route Parameter

Tambah route sapaan (hapus setelah latihan jika mau):

```php
Route::get('/halo/{nama}', function (string $nama) {
    return "Halo, $nama! Selamat belajar Laravel.";
});
```

Buka `http://localhost:8000/halo/Budi`

---

## Latihan Modul 03

1. Buat halaman `/kontak` via `PageController@contact` dengan data email & telepon
2. Tambah named route `contact`
3. Link antar halaman pakai `route('...')`, bukan URL hardcode
4. Buat route `/info` yang return JSON:

```php
Route::get('/info', function () {
    return response()->json([
        'app' => 'Task Manager',
        'laravel' => app()->version(),
    ]);
});
```

---

## Git — Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/03-routing-controller

git add .
git commit -m "feat: tambah route, controller, halaman home dan tentang"
git push -u origin modul/03-routing-controller
```

Buat Pull Request → merge → `git checkout main && git pull`

---

## Checklist Selesai

- [ ] `HomeController` & `PageController` dibuat
- [ ] `/` dan `/tentang` tampil di browser
- [ ] Data dari controller tampil di view
- [ ] Route punya nama
- [ ] `php artisan route:list` menampilkan route kamu
- [ ] Commit & push ke GitHub

---

**Modul sebelumnya:** [02 — Git & GitHub](../02-git-github/README.md)  
**Modul berikutnya:** [04 — Blade Template](../04-blade-template/README.md)
