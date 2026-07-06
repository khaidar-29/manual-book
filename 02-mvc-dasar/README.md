# Modul 02 — MVC Dasar

Modul kedua: memahami alur Route → Controller → View tanpa database.

**Estimasi waktu:** 2–3 hari  
**Prasyarat:** [Modul 01 — Setup](../01-setup/README.md)

---

## Tujuan Pembelajaran

Setelah menyelesaikan modul ini, kamu sudah bisa:

- [ ] Membuat route GET dan POST
- [ ] Membuat controller dan memanggilnya dari route
- [ ] Membuat layout Blade dan halaman turunannya
- [ ] Mengirim data dari controller ke view
- [ ] Memahami alur MVC di Laravel

---

## Konsep MVC

```
URL (Browser)
    ↓
Route (web.php)       → Menentukan URL ke controller mana
    ↓
Controller            → Memproses logika, menyiapkan data
    ↓
View (Blade)          → Merender HTML
    ↓
Response ke Browser
```

Di modul ini **belum pakai database**. Fokusnya memahami alur di atas.

---

## Langkah 1: Buat Controller

```bash
php artisan make:controller HomeController
php artisan make:controller PageController
```

Isi `app/Http/Controllers/HomeController.php`:

```php
<?php

namespace App\Http\Controllers;

class HomeController extends Controller
{
    public function index()
    {
        return view('home', [
            'title' => 'Beranda',
            'message' => 'Selamat datang di aplikasi belajar Laravel!',
        ]);
    }
}
```

Isi `app/Http/Controllers/PageController.php`:

```php
<?php

namespace App\Http\Controllers;

class PageController extends Controller
{
    public function about()
    {
        return view('about', [
            'title' => 'Tentang',
            'name' => 'Nama Kamu',
            'bio' => 'Sedang belajar Laravel.',
        ]);
    }

    public function contact()
    {
        return view('contact', [
            'title' => 'Kontak',
            'email' => 'email@example.com',
        ]);
    }
}
```

---

## Langkah 2: Buat Layout Blade

Buat file `resources/views/layouts/app.blade.php`:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Belajar Laravel')</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="bg-gray-50 text-gray-800">
    @include('partials.navbar')

    <main class="max-w-4xl mx-auto p-6">
        @if(session('success'))
            <div class="bg-green-100 text-green-800 p-3 rounded mb-4">
                {{ session('success') }}
            </div>
        @endif

        @yield('content')
    </main>

    <footer class="text-center text-sm text-gray-500 py-6">
        &copy; {{ date('Y') }} Belajar Laravel
    </footer>
</body>
</html>
```

Buat file `resources/views/partials/navbar.blade.php`:

```blade
<nav class="bg-white shadow-sm">
    <div class="max-w-4xl mx-auto px-6 py-4 flex gap-6">
        <a href="{{ route('home') }}" class="font-semibold">Beranda</a>
        <a href="{{ route('about') }}">Tentang</a>
        <a href="{{ route('contact') }}">Kontak</a>
    </div>
</nav>
```

---

## Langkah 3: Buat Halaman View

Buat `resources/views/home.blade.php`:

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <h1 class="text-2xl font-bold mb-4">{{ $title }}</h1>
    <p>{{ $message }}</p>
@endsection
```

Buat `resources/views/about.blade.php`:

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <h1 class="text-2xl font-bold mb-4">{{ $title }}</h1>
    <p><strong>Nama:</strong> {{ $name }}</p>
    <p><strong>Bio:</strong> {{ $bio }}</p>
@endsection
```

Buat `resources/views/contact.blade.php`:

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <h1 class="text-2xl font-bold mb-4">{{ $title }}</h1>
    <p>Hubungi kami di: <a href="mailto:{{ $email }}">{{ $email }}</a></p>
@endsection
```

---

## Langkah 4: Daftarkan Route

Edit `routes/web.php`:

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use Illuminate\Support\Facades\Route;

Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');
Route::get('/kontak', [PageController::class, 'contact'])->name('contact');
```

Cek route yang terdaftar:

```bash
php artisan route:list
```

---

## Langkah 5: Latihan Route dengan Closure

Sebelum lanjut ke database, coba route sederhana tanpa controller:

```php
Route::get('/halo/{nama}', function (string $nama) {
    return "Halo, $nama!";
});
```

Buka `http://localhost:8000/halo/Budi` — harus menampilkan "Halo, Budi!".

Hapus route ini setelah latihan selesai.

---

## Sintaks Blade yang Perlu Dikuasai

| Sintaks | Fungsi |
|---|---|
| `{{ $variabel }}` | Output variabel (aman dari XSS) |
| `@extends('layouts.app')` | Pakai layout induk |
| `@section('content')` | Isi bagian layout |
| `@yield('content')` | Tempat konten ditampilkan di layout |
| `@include('partials.navbar')` | Sisipkan partial view |
| `@if` / `@else` / `@endif` | Kondisi |
| `@foreach` / `@endforeach` | Loop |

---

## Latihan

1. Tambah halaman `/layanan` yang menampilkan 3 layanan dalam bentuk list
2. Buat route `/halo/{nama}` yang menampilkan sapaan personal
3. Tambah link "Layanan" di navbar
4. Ubah warna navbar dan footer sesuai selera
5. Tampilkan tahun di footer menggunakan `{{ date('Y') }}`

### Bonus

Buat halaman `/layanan` dengan data array dari controller (belum pakai database):

```php
public function services()
{
    $services = ['Web Development', 'Mobile App', 'UI/UX Design'];

    return view('services', compact('services'));
}
```

---

## Checklist Selesai

- [ ] Controller `HomeController` dan `PageController` dibuat
- [ ] Layout `layouts/app.blade.php` dan navbar berfungsi
- [ ] 3 halaman (beranda, tentang, kontak) bisa diakses
- [ ] Route punya nama (`->name()`)
- [ ] Paham alur Route → Controller → View
- [ ] Bisa kirim data dari controller ke view

---

**Modul sebelumnya:** [01 — Setup](../01-setup/README.md)  
**Modul berikutnya:** [03 — Database](../03-database/README.md)
