# Modul 04 — Blade Template

Merapikan tampilan **Task Manager** dengan layout Blade, navbar, dan partial.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 03 — Routing & Controller](../03-routing-controller/README.md)

---

## Tujuan Modul

- [ ] Layout master dengan `@extends` & `@yield`
- [ ] Navbar partial dengan `@include`
- [ ] Semua halaman pakai layout yang sama
- [ ] Paham sintaks Blade dasar
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
resources/views/
├── layouts/app.blade.php
├── partials/navbar.blade.php
├── partials/alert.blade.php
├── home.blade.php      ← refactor pakai layout
└── about.blade.php     ← refactor pakai layout
```

---

## Langkah 1: Layout Master

**`resources/views/layouts/app.blade.php`:**

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Task Manager')</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="bg-gray-50 text-gray-800 min-h-screen flex flex-col">
    @include('partials.navbar')

    <main class="flex-1 max-w-4xl mx-auto w-full p-6">
        @include('partials.alert')
        @yield('content')
    </main>

    <footer class="text-center text-sm text-gray-500 py-6 border-t">
        &copy; {{ date('Y') }} Task Manager — Built with Laravel
    </footer>
</body>
</html>
```

---

## Langkah 2: Partial Navbar

**`resources/views/partials/navbar.blade.php`:**

```blade
<nav class="bg-white shadow-sm border-b">
    <div class="max-w-4xl mx-auto px-6 py-4 flex items-center justify-between">
        <a href="{{ route('home') }}" class="text-lg font-bold text-blue-600">
            Task Manager
        </a>
        <div class="flex gap-6 text-sm">
            <a href="{{ route('home') }}"
               class="hover:text-blue-600 {{ request()->routeIs('home') ? 'text-blue-600 font-semibold' : '' }}">
                Beranda
            </a>
            <a href="{{ route('about') }}"
               class="hover:text-blue-600 {{ request()->routeIs('about') ? 'text-blue-600 font-semibold' : '' }}">
                Tentang
            </a>
        </div>
    </div>
</nav>
```

---

## Langkah 3: Partial Alert (Flash Message)

**`resources/views/partials/alert.blade.php`:**

```blade
@if(session('success'))
    <div class="bg-green-100 border border-green-300 text-green-800 px-4 py-3 rounded mb-4">
        {{ session('success') }}
    </div>
@endif

@if(session('error'))
    <div class="bg-red-100 border border-red-300 text-red-800 px-4 py-3 rounded mb-4">
        {{ session('error') }}
    </div>
@endif
```

> Partial ini akan dipakai di Modul 06 (CRUD) untuk notifikasi sukses/error.

---

## Langkah 4: Refactor Halaman

**`resources/views/home.blade.php`:**

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <div class="text-center py-12">
        <h1 class="text-4xl font-bold mb-4">{{ $title }}</h1>
        <p class="text-gray-600 text-lg mb-8">{{ $tagline }}</p>
        <a href="{{ route('about') }}"
           class="inline-block bg-blue-600 text-white px-6 py-2 rounded hover:bg-blue-700">
            Pelajari Lebih Lanjut
        </a>
    </div>
@endsection
```

**`resources/views/about.blade.php`:**

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <h1 class="text-2xl font-bold mb-6">{{ $title }}</h1>
    <div class="bg-white rounded-lg shadow-sm p-6 space-y-3">
        <p><strong>Aplikasi:</strong> {{ $appName }}</p>
        <p><strong>Versi:</strong> {{ $version }}</p>
        <p class="text-gray-600">{{ $description }}</p>
    </div>
@endsection
```

---

## Sintaks Blade Penting

| Sintaks | Fungsi |
|---|---|
| `{{ $var }}` | Output variabel (auto-escape) |
| `@extends('layouts.app')` | Pakai layout induk |
| `@section('content')` / `@endsection` | Definisi section |
| `@yield('content')` | Tempat section ditampilkan |
| `@include('partials.navbar')` | Sisipkan partial |
| `@if` / `@else` / `@endif` | Kondisi |
| `@foreach` / `@endforeach` | Loop |
| `@csrf` | Token keamanan form (Modul 06) |
| `{{ route('home') }}` | Generate URL dari named route |
| `{{ old('field') }}` | Nilai input sebelumnya setelah validasi gagal |

---

## Latihan Modul 04

1. Tambah halaman `/kontak` dengan layout yang sama
2. Tambah link Kontak di navbar
3. Buat partial `partials/card.blade.php`:

```blade
<div class="bg-white rounded-lg shadow-sm p-6">
    <h2 class="font-semibold text-lg mb-2">{{ $title }}</h2>
    <p class="text-gray-600">{{ $body }}</p>
</div>
```

4. Pakai `@include('partials.card', ['title' => '...', 'body' => '...'])` di halaman about
5. Tampilkan 3 fitur app di beranda pakai loop `@foreach`:

```php
// Di HomeController
$features = ['Kelola Task', 'Tandai Selesai', 'Filter Kategori'];
return view('home', compact('title', 'tagline', 'features'));
```

---

## Git — Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/04-blade-template

git add .
git commit -m "feat: tambah layout blade, navbar, dan refactor halaman"
git push -u origin modul/04-blade-template
```

---

## Checklist Selesai

- [ ] Layout `layouts/app.blade.php` dibuat
- [ ] Navbar & alert partial berfungsi
- [ ] Home & about pakai `@extends`
- [ ] Link navbar highlight halaman aktif
- [ ] Commit & push ke GitHub

---

**Modul sebelumnya:** [03 — Routing & Controller](../03-routing-controller/README.md)  
**Modul berikutnya:** [05 — Database](../05-database/README.md)
