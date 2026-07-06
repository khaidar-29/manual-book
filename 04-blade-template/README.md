# Modul 04 — Blade Template + Bootstrap CDN

Modul keempat: merapikan tampilan Task Manager dengan **layout Blade** dan **Bootstrap 5 via CDN**.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 03 — Routing & Controller](../03-routing-controller/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Membuat layout master dengan `@extends`, `@section`, `@yield`
- [ ] Membuat partial view dengan `@include`
- [ ] Memahami sintaks Blade (`@if`, `@foreach`, `@csrf`)
- [ ] Menggunakan Bootstrap 5 via CDN (tanpa npm/install)
- [ ] Refactor halaman Modul 03 pakai layout baru
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
resources/views/
├── layouts/
│   └── app.blade.php         ← Layout master + Bootstrap CDN
├── partials/
│   ├── navbar.blade.php      ← Navbar
│   └── alert.blade.php       ← Flash message
├── home.blade.php            ← Refactor
└── about.blade.php           ← Refactor
```

---

## Mengapa Blade Template?

Tanpa layout, setiap halaman harus ulang tulis `<html>`, `<head>`, navbar, footer. Blade menyelesaikan ini:

- **Layout** — tulis struktur HTML sekali
- **Section** — setiap halaman isi bagian kontennya saja
- **Partial** — potongan HTML reusable (navbar, alert)

---

## Mengapa Bootstrap CDN?

Bootstrap via CDN artinya:
- Cukup tambah `<link>` di HTML
- **Tidak perlu** `npm install`
- **Tidak perlu** `npm run dev`
- **Tidak perlu** Vite/Webpack

Cocok untuk belajar karena setup minimal.

---

## Langkah 1: Layout Master

Buat folder dan file:

```bash
mkdir -p resources/views/layouts
mkdir -p resources/views/partials
```

Buat `resources/views/layouts/app.blade.php`:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Task Manager')</title>

    {{-- Bootstrap 5 CSS via CDN --}}
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light d-flex flex-column min-vh-100">

    {{-- Navbar --}}
    @include('partials.navbar')

    {{-- Konten utama --}}
    <main class="container py-4 flex-grow-1">
        @include('partials.alert')
        @yield('content')
    </main>

    {{-- Footer --}}
    <footer class="text-center text-muted py-3 border-top mt-auto">
        <small>&copy; {{ date('Y') }} Task Manager — Built with Laravel & Bootstrap</small>
    </footer>

    {{-- Bootstrap 5 JS via CDN --}}
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

**Penjelasan:**
- `@yield('title', 'Task Manager')` — tempat judul halaman, default "Task Manager"
- `@yield('content')` — tempat isi halaman
- `@include('partials.navbar')` — sisipkan file partial
- Bootstrap CSS & JS di-load dari CDN jsDelivr

---

## Langkah 2: Partial Navbar

Buat `resources/views/partials/navbar.blade.php`:

```blade
<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
    <div class="container">
        <a class="navbar-brand fw-bold text-primary" href="{{ route('home') }}">
            📋 Task Manager
        </a>

        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav ms-auto">
                <li class="nav-item">
                    <a class="nav-link {{ request()->routeIs('home') ? 'active fw-semibold' : '' }}"
                       href="{{ route('home') }}">Beranda</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link {{ request()->routeIs('about') ? 'active fw-semibold' : '' }}"
                       href="{{ route('about') }}">Tentang</a>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

**Penjelasan:**
- `request()->routeIs('home')` — cek apakah halaman aktif, untuk highlight menu
- `navbar-toggler` — tombol hamburger untuk mobile (fitur Bootstrap)
- `data-bs-toggle="collapse"` — butuh Bootstrap JS (sudah di-load di layout)

---

## Langkah 3: Partial Alert (Flash Message)

Buat `resources/views/partials/alert.blade.php`:

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

**Kapan dipakai?**
- Modul 06 (CRUD): setelah tambah/edit/hapus task
- Controller: `return redirect()->route('tasks.index')->with('success', 'Task berhasil ditambahkan!');`

---

## Langkah 4: Refactor Halaman Home

Ganti isi `resources/views/home.blade.php`:

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <div class="text-center py-5">
        <h1 class="display-5 fw-bold mb-3">{{ $title }}</h1>
        <p class="lead text-muted mb-4">{{ $tagline }}</p>
        <a href="{{ route('about') }}" class="btn btn-primary btn-lg">
            Pelajari Lebih Lanjut
        </a>
    </div>
@endsection
```

**Penjelasan:**
- `@extends('layouts.app')` — pakai layout master
- `@section('title', $title)` — isi section title
- `@section('content')` ... `@endsection` — isi konten halaman
- Class Bootstrap: `display-5`, `btn btn-primary`, `lead`, dll.

---

## Langkah 5: Refactor Halaman About

Ganti isi `resources/views/about.blade.php`:

```blade
@extends('layouts.app')

@section('title', $title)

@section('content')
    <h1 class="h2 fw-bold mb-4">{{ $title }}</h1>

    <div class="card shadow-sm">
        <div class="card-body">
            <table class="table table-borderless mb-3">
                <tr>
                    <td width="120"><strong>Aplikasi</strong></td>
                    <td>{{ $appName }}</td>
                </tr>
                <tr>
                    <td><strong>Versi</strong></td>
                    <td>{{ $version }}</td>
                </tr>
                <tr>
                    <td><strong>Pembuat</strong></td>
                    <td>{{ $author }}</td>
                </tr>
            </table>
            <p class="text-muted mb-0">{{ $description }}</p>
        </div>
    </div>
@endsection
```

---

## Langkah 6: Uji di Browser

```bash
php artisan serve
```

| Halaman | Cek |
|---|---|
| `/` | Layout + navbar + footer tampil |
| `/tentang` | Card Bootstrap tampil, menu "Tentang" aktif |
| Mobile (resize browser) | Navbar collapse jadi hamburger |

---

## Referensi Sintaks Blade

| Sintaks | Fungsi | Contoh |
|---|---|---|
| `{{ $var }}` | Output variabel | `{{ $title }}` |
| `@extends('layout')` | Pakai layout induk | `@extends('layouts.app')` |
| `@section('name')` | Definisi section | `@section('content')` |
| `@endsection` | Tutup section | |
| `@yield('name')` | Tempat section (di layout) | `@yield('content')` |
| `@include('partial')` | Sisipkan partial | `@include('partials.navbar')` |
| `@if` / `@endif` | Kondisi | `@if($tasks->isEmpty())` |
| `@foreach` / `@endforeach` | Loop | `@foreach($tasks as $task)` |
| `@csrf` | Token keamanan form | Wajib di setiap form POST |
| `@error('field')` | Tampilkan error validasi | Modul 06 |
| `{{ old('field') }}` | Nilai input sebelumnya | Modul 06 |
| `{{ route('name') }}` | Generate URL | `{{ route('home') }}` |

---

## Referensi Class Bootstrap Penting

| Class | Fungsi |
|---|---|
| `container` | Wrapper lebar responsive |
| `row` / `col-md-6` | Grid system |
| `btn btn-primary` | Tombol biru |
| `btn btn-danger` | Tombol merah |
| `btn btn-warning` | Tombol kuning |
| `card` / `card-body` | Kotak konten |
| `table table-striped` | Tabel bergaris |
| `form-control` | Style input form |
| `form-select` | Style dropdown |
| `alert alert-success` | Notifikasi hijau |
| `badge bg-success` | Label kecil hijau |
| `d-flex justify-content-between` | Flexbox |
| `mb-3`, `py-4`, `mt-2` | Margin & padding |

Dokumentasi lengkap: [getbootstrap.com/docs/5.3](https://getbootstrap.com/docs/5.3)

---

## Latihan Modul 04

### Latihan 1: Halaman Kontak

1. Tambah method `contact()` di PageController
2. Buat view `contact.blade.php` dengan `@extends('layouts.app')`
3. Tampilkan email & telepon dalam card Bootstrap
4. Tambah route dan link di navbar

### Latihan 2: Partial Card

Buat `resources/views/partials/card.blade.php`:

```blade
<div class="card shadow-sm mb-3">
    <div class="card-body">
        <h5 class="card-title">{{ $title }}</h5>
        <p class="card-text text-muted">{{ $body }}</p>
    </div>
</div>
```

Pakai di halaman about:

```blade
@include('partials.card', ['title' => 'Laravel 12', 'body' => 'Framework PHP modern'])
@include('partials.card', ['title' => 'Bootstrap 5', 'body' => 'CSS framework via CDN'])
@include('partials.card', ['title' => 'MySQL 8', 'body' => 'Database relational'])
```

### Latihan 3: List Fitur di Beranda

Update HomeController:

```php
$features = ['Kelola Task Harian', 'Tandai Selesai', 'Filter Kategori', 'Dashboard Statistik'];
return view('home', compact('title', 'tagline', 'features'));
```

Tampilkan di view:

```blade
<div class="row mt-5">
    @foreach($features as $feature)
        <div class="col-md-3 mb-3">
            <div class="card text-center shadow-sm h-100">
                <div class="card-body">
                    <p class="card-text">{{ $feature }}</p>
                </div>
            </div>
        </div>
    @endforeach
</div>
```

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| Tampilan plain tanpa style | Pastikan link Bootstrap CDN ada di `<head>` layout |
| Navbar hamburger tidak jalan | Pastikan Bootstrap JS di-load sebelum `</body>` |
| `@extends` error | Pastikan `@section` dan `@endsection` berpasangan |
| Layout tidak muncul | File layout harus di `resources/views/layouts/app.blade.php` |

---

## Checklist Selesai

- [ ] Layout `layouts/app.blade.php` dengan Bootstrap CDN
- [ ] Navbar responsive dengan highlight menu aktif
- [ ] Partial alert untuk flash message
- [ ] Home & about di-refactor pakai `@extends`
- [ ] Halaman kontak ditambahkan (latihan)
- [ ] Tampilan rapi di desktop dan mobile

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: tambah layout blade dan bootstrap cdn"
git push
```

---

**Modul sebelumnya:** [03 — Routing & Controller](../03-routing-controller/README.md)  
**Modul berikutnya:** [05 — Database](../05-database/README.md)
