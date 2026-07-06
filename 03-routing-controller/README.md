# Modul 03 — Routing & Controller

Modul ketiga: memahami alur **Route → Controller → View** di project Task Manager.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 02 — Git & GitHub](../02-git-github/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Memahami alur request Laravel (MVC)
- [ ] Membuat route GET dengan named route
- [ ] Membuat controller dan method
- [ ] Mengirim data dari controller ke view
- [ ] Menampilkan data di halaman Blade
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

| URL | Controller | Method | View |
|---|---|---|---|
| `/` | HomeController | index | home |
| `/tentang` | PageController | about | about |

> Modul ini **belum pakai database** dan **belum pakai Bootstrap**. Fokus ke alur dasar dulu.

---

## Konsep: Alur Request Laravel

Ketika user buka URL di browser, ini yang terjadi:

```
1. Browser request  →  GET http://localhost:8000/tentang
2. Route             →  web.php cocokkan URL ke Controller
3. Controller        →  Jalankan method, siapkan data
4. View (Blade)      →  Render HTML dengan data
5. Response          →  HTML dikirim ke browser
```

Inilah pola **MVC (Model-View-Controller)**:
- **Model** — data (database) → Modul 05
- **View** — tampilan (Blade)
- **Controller** — logika, penghubung route & view

---

## Langkah 1: Buat Controller

Jalankan perintah artisan:

```bash
php artisan make:controller HomeController
php artisan make:controller PageController
```

Artisan membuat file di `app/Http/Controllers/`.

### HomeController

Buka `app/Http/Controllers/HomeController.php`:

```php
<?php

namespace App\Http\Controllers;

class HomeController extends Controller
{
    /**
     * Tampilkan halaman beranda.
     */
    public function index()
    {
        $title = 'Task Manager';
        $tagline = 'Kelola task harianmu dengan mudah.';

        return view('home', compact('title', 'tagline'));
    }
}
```

**Penjelasan:**
- `public function index()` — method yang dipanggil saat route diakses
- `$title`, `$tagline` — data yang dikirim ke view
- `compact('title', 'tagline')` — shortcut kirim variabel ke view
- `view('home', ...)` — render file `resources/views/home.blade.php`

### PageController

Buka `app/Http/Controllers/PageController.php`:

```php
<?php

namespace App\Http\Controllers;

class PageController extends Controller
{
    /**
     * Tampilkan halaman tentang aplikasi.
     */
    public function about()
    {
        return view('about', [
            'title' => 'Tentang Aplikasi',
            'appName' => 'Task Manager',
            'version' => '1.0.0',
            'author' => 'Nama Kamu',
            'description' => 'Aplikasi manajemen task sederhana yang dibangun dengan Laravel sebagai media belajar.',
        ]);
    }
}
```

**Penjelasan:**
- Bisa pakai array `['key' => 'value']` sebagai parameter kedua `view()`
- Di Blade, akses dengan `{{ $appName }}`, `{{ $author }}`, dll.

---

## Langkah 2: Buat View (Blade)

View masih HTML sederhana — layout Bootstrap nanti di Modul 04.

### home.blade.php

Buat file `resources/views/home.blade.php`:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $title }}</title>
</head>
<body>
    <h1>{{ $title }}</h1>
    <p>{{ $tagline }}</p>
    <p><a href="{{ route('about') }}">Tentang Aplikasi →</a></p>
</body>
</html>
```

### about.blade.php

Buat file `resources/views/about.blade.php`:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $title }}</title>
</head>
<body>
    <h1>{{ $title }}</h1>
    <ul>
        <li><strong>Aplikasi:</strong> {{ $appName }}</li>
        <li><strong>Versi:</strong> {{ $version }}</li>
        <li><strong>Pembuat:</strong> {{ $author }}</li>
    </ul>
    <p>{{ $description }}</p>
    <p><a href="{{ route('home') }}">← Kembali ke Beranda</a></p>
</body>
</html>
```

**Penjelasan sintaks Blade:**
- `{{ $title }}` — output variabel (auto-escape, aman dari XSS)
- `{{ route('about') }}` — generate URL dari named route (lebih aman dari hardcode URL)

---

## Langkah 3: Daftarkan Route

Buka `routes/web.php`. **Ganti seluruh isi** dengan:

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use Illuminate\Support\Facades\Route;

Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');
```

**Penjelasan per baris:**
- `Route::get(...)` — tangani request HTTP GET
- `'/'` — URL path
- `[HomeController::class, 'index']` — panggil method `index` di HomeController
- `->name('home')` — beri nama route, dipanggil via `route('home')`

### Cek route terdaftar

```bash
php artisan route:list
```

Output yang diharapkan (minimal):

```
GET|HEAD  /           home    › HomeController@index
GET|HEAD  tentang     about   › PageController@about
```

---

## Langkah 4: Uji di Browser

Pastikan server jalan:

```bash
php artisan serve
```

| URL | Hasil yang diharapkan |
|---|---|
| http://localhost:8000/ | Judul "Task Manager" + tagline |
| http://localhost:8000/tentang | Info aplikasi + link kembali |
| Klik link antar halaman | Navigasi jalan |

---

## Langkah 5: Route dengan Parameter (Opsional)

Tambah route sapaan untuk latihan. Di `routes/web.php`:

```php
Route::get('/halo/{nama}', function (string $nama) {
    return "Halo, {$nama}! Selamat belajar Laravel.";
});
```

Buka: `http://localhost:8000/halo/Budi` → tampil "Halo, Budi!"

**Penjelasan:**
- `{nama}` — parameter dinamis dari URL
- `function (string $nama)` — Laravel otomatis inject nilai URL ke variabel

---

## Konsep Route Lengkap

### HTTP Methods

| Method | Fungsi | Contoh |
|---|---|---|
| GET | Ambil/tampilkan data | Halaman web |
| POST | Kirim/simpan data | Submit form |
| PUT/PATCH | Update data | Edit form |
| DELETE | Hapus data | Tombol hapus |

### Named Route — Mengapa Penting?

```php
// ❌ Buruk — hardcode URL
<a href="/tentang">Tentang</a>

// ✅ Baik — named route
<a href="{{ route('about') }}">Tentang</a>
```

Keuntungan named route:
- Jika URL berubah, cukup ubah di `web.php`
- Semua link otomatis ikut berubah

---

## Latihan Modul 03

### Latihan 1: Halaman Kontak

1. Tambah method `contact()` di `PageController` dengan data: nama, email, telepon
2. Buat view `contact.blade.php`
3. Tambah route `GET /kontak` dengan name `contact`
4. Tambah link ke halaman kontak dari beranda

### Latihan 2: Route Info JSON

Tambah route yang return JSON:

```php
Route::get('/info', function () {
    return response()->json([
        'app' => 'Task Manager',
        'version' => '1.0.0',
        'laravel' => app()->version(),
        'php' => PHP_VERSION,
    ]);
});
```

Buka `/info` — harus tampil JSON, bukan HTML.

### Latihan 3: Kirim Array ke View

Di `HomeController`, kirim array fitur:

```php
$features = [
    'Kelola task harian',
    'Tandai task selesai',
    'Filter berdasarkan kategori',
];
return view('home', compact('title', 'tagline', 'features'));
```

Di view, tampilkan dengan loop:

```blade
<ul>
    @foreach($features as $feature)
        <li>{{ $feature }}</li>
    @endforeach
</ul>
```

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| 404 Not Found | Cek route di `web.php`, jalankan `php artisan route:list` |
| View not found | Pastikan file ada di `resources/views/` dengan nama benar |
| Variable undefined | Pastikan variabel dikirim dari controller via `compact()` atau array |
| Perubahan tidak tampil | Refresh browser, pastikan `php artisan serve` masih jalan |

---

## Checklist Selesai

- [ ] `HomeController` dan `PageController` dibuat
- [ ] View `home.blade.php` dan `about.blade.php` dibuat
- [ ] Route `/` dan `/tentang` terdaftar dengan named route
- [ ] Data dari controller tampil di view
- [ ] Link antar halaman pakai `route()` bukan hardcode
- [ ] `php artisan route:list` menampilkan route kamu
- [ ] Latihan kontak selesai (opsional)

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: tambah route controller halaman home dan tentang"
git push
```

---

**Modul sebelumnya:** [02 — Git & GitHub](../02-git-github/README.md)  
**Modul berikutnya:** [04 — Blade + Bootstrap](../04-blade-template/README.md)
