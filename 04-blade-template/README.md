# Modul 04 — Blade Template + Bootstrap

Merapikan tampilan **Task Manager** dengan layout Blade dan **Bootstrap 5**.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 03 — Routing & Controller](../03-routing-controller/README.md)

---

## Tujuan Modul

- [ ] Layout master dengan `@extends` & `@yield`
- [ ] Navbar partial dengan `@include`
- [ ] Styling pakai **Bootstrap 5** (via CDN)
- [ ] Semua halaman pakai layout yang sama
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
resources/views/
├── layouts/app.blade.php      ← layout + Bootstrap CDN
├── partials/navbar.blade.php
├── partials/alert.blade.php
├── home.blade.php             ← refactor pakai layout
└── about.blade.php            ← refactor pakai layout
```

---

## Langkah 1: Layout Master (Bootstrap 5)

**`resources/views/layouts/app.blade.php`:**

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Task Manager')</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light d-flex flex-column min-vh-100">
    @include('partials.navbar')

    <main class="container py-4 flex-grow-1">
        @include('partials.alert')
        @yield('content')
    </main>

    <footer class="text-center text-muted py-3 border-top">
        &copy; {{ date('Y') }} Task Manager — Built with Laravel
    </footer>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

> Bootstrap di-load via **CDN** — tidak perlu npm/Vite untuk modul ini.

---

## Langkah 2: Partial Navbar

**`resources/views/partials/navbar.blade.php`:**

```blade
<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm border-bottom">
    <div class="container">
        <a class="navbar-brand fw-bold text-primary" href="{{ route('home') }}">Task Manager</a>
        <div class="navbar-nav ms-auto gap-2">
            <a class="nav-link {{ request()->routeIs('home') ? 'active fw-semibold' : '' }}"
               href="{{ route('home') }}">Beranda</a>
            <a class="nav-link {{ request()->routeIs('about') ? 'active fw-semibold' : '' }}"
               href="{{ route('about') }}">Tentang</a>
        </div>
    </div>
</nav>
```

---

## Langkah 3: Partial Alert (Flash Message)

**`resources/views/partials/alert.blade.php`:**

```blade
@if(session('success'))
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        {{ session('success') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif

@if(session('error'))
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        {{ session('error') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif
```

---

## Langkah 4: Refactor Halaman

**`resources/views/home.blade.php`:**

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <div class="text-center py-5">
        <h1 class="display-5 fw-bold mb-3">{{ $title }}</h1>
        <p class="lead text-muted mb-4">{{ $tagline }}</p>
        <a href="{{ route('about') }}" class="btn btn-primary">Pelajari Lebih Lanjut</a>
    </div>
@endsection
```

**`resources/views/about.blade.php`:**

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <h1 class="h2 fw-bold mb-4">{{ $title }}</h1>
    <div class="card shadow-sm">
        <div class="card-body">
            <p><strong>Aplikasi:</strong> {{ $appName }}</p>
            <p><strong>Versi:</strong> {{ $version }}</p>
            <p class="text-muted mb-0">{{ $description }}</p>
        </div>
    </div>
@endsection
```

---

## Class Bootstrap yang Sering Dipakai

| Class | Fungsi |
|---|---|
| `container` | Wrapper lebar tetap |
| `btn btn-primary` | Tombol biru |
| `btn btn-danger` | Tombol merah (hapus) |
| `card`, `card-body` | Kotak konten |
| `table table-striped` | Tabel bergaris |
| `form-control` | Input form |
| `alert alert-success` | Notifikasi sukses |
| `badge bg-success` | Label hijau |
| `d-flex`, `justify-content-between` | Flexbox layout |
| `mb-3`, `py-4` | Margin & padding |

---

## Sintaks Blade Penting

| Sintaks | Fungsi |
|---|---|
| `{{ $var }}` | Output variabel (auto-escape) |
| `@extends('layouts.app')` | Pakai layout induk |
| `@section('content')` | Definisi section |
| `@yield('content')` | Tempat section ditampilkan |
| `@include('partials.navbar')` | Sisipkan partial |
| `@if` / `@foreach` | Kondisi & loop |
| `@csrf` | Token keamanan form |

---

## Latihan Modul 04

1. Tambah halaman `/kontak` dengan layout Bootstrap yang sama
2. Tambah link Kontak di navbar
3. Buat partial `partials/card.blade.php`:

```blade
<div class="card shadow-sm mb-3">
    <div class="card-body">
        <h5 class="card-title">{{ $title }}</h5>
        <p class="card-text text-muted">{{ $body }}</p>
    </div>
</div>
```

4. Tampilkan 3 fitur app di beranda pakai `@foreach` + `list-group`

---

## Git — Commit & Push

```bash
git checkout main && git pull origin main
git checkout -b modul/04-blade-template

git add .
git commit -m "feat: tambah layout blade dengan bootstrap dan navbar"
git push -u origin modul/04-blade-template
```

---

## Checklist Selesai

- [ ] Layout dengan Bootstrap CDN dibuat
- [ ] Navbar & alert partial berfungsi
- [ ] Home & about pakai `@extends`
- [ ] Tampilan rapi dengan class Bootstrap
- [ ] Commit & push ke GitHub

---

**Modul sebelumnya:** [03 — Routing & Controller](../03-routing-controller/README.md)  
**Modul berikutnya:** [05 — Database](../05-database/README.md)
