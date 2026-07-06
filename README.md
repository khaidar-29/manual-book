# Panduan Belajar Laravel

Panduan lengkap mempelajari framework PHP modern. Cocok untuk pemula maupun siapa pun yang ingin mulai membangun aplikasi web dengan Laravel.

---

## Modul Pembelajaran

Ikuti modul berikut secara berurutan. Setiap modul berisi langkah-langkah praktik yang bisa dikerjakan langsung.

| Modul | Topik | Estimasi |
|---|---|---|
| [01 — Setup](./01-setup/README.md) | Install & jalankan Laravel | 1–2 hari |
| [02 — MVC Dasar](./02-mvc-dasar/README.md) | Route, Controller, Blade | 2–3 hari |
| [03 — Database](./03-database/README.md) | Migration, Model, Seeder | 2–3 hari |
| [04 — CRUD Task](./04-crud-task/README.md) | Create, Read, Update, Delete | 3–4 hari |
| [05 — Authentication](./05-auth/README.md) | Login & Register | 2–3 hari |

**Total estimasi:** ±2 minggu

---

## Daftar Isi

1. [Apa itu Laravel?](#1-apa-itu-laravel)
2. [Persiapan Environment](#2-persiapan-environment)
3. [Instalasi Laravel](#3-instalasi-laravel)
4. [Menjalankan Project](#4-menjalankan-project)
5. [Struktur Folder Laravel](#5-struktur-folder-laravel)
6. [Alur Kerja Request (MVC)](#6-alur-kerja-request-mvc)
7. [Routing](#7-routing)
8. [Controller](#8-controller)
9. [View & Blade Template](#9-view--blade-template)
10. [Model & Eloquent ORM](#10-model--eloquent-orm)
11. [Migration & Database](#11-migration--database)
12. [Tutorial CRUD Lengkap](#12-tutorial-crud-lengkap)
13. [Validasi Form](#13-validasi-form)
14. [Relasi Database](#14-relasi-database)
15. [Authentication (Login & Register)](#15-authentication-login--register)
16. [Artisan Command yang Sering Dipakai](#16-artisan-command-yang-sering-dipakai)
17. [Best Practice](#17-best-practice)
18. [Git Workflow](#18-git-workflow)
19. [Troubleshooting](#19-troubleshooting)
20. [Referensi & Latihan](#20-referensi--latihan)

---

## 1. Apa itu Laravel?

**Laravel** adalah framework PHP open-source yang memudahkan pembuatan aplikasi web. Framework ini menyediakan:

- **Routing** — mengatur URL ke fungsi tertentu
- **ORM (Eloquent)** — berinteraksi dengan database tanpa menulis SQL mentah
- **Blade** — template engine untuk tampilan HTML
- **Migration** — version control untuk struktur database
- **Authentication** — sistem login/register siap pakai
- **Artisan** — command-line tool bawaan Laravel

### Kenapa Laravel?

| Keuntungan | Penjelasan |
|---|---|
| Dokumentasi lengkap | [laravel.com/docs](https://laravel.com/docs) sangat detail |
| Komunitas besar | Banyak tutorial, package, dan jawaban di Stack Overflow |
| Konvensi jelas | Struktur folder konsisten antar project |
| Produktivitas tinggi | Banyak fitur siap pakai (auth, queue, mail, dll.) |

---

## 2. Persiapan Environment

Sebelum mulai, pastikan software berikut sudah terinstall di komputer kamu.

### Software yang Dibutuhkan

| Software | Versi Minimum | Fungsi |
|---|---|---|
| PHP | 8.2+ | Bahasa pemrograman utama |
| Composer | 2.x | Package manager PHP |
| Node.js & NPM | 18+ | Build asset frontend (CSS/JS) |
| MySQL / MariaDB | 8.x | Database |
| Git | Terbaru | Version control |

### Cara Cek Versi

```bash
php -v
composer -V
node -v
npm -v
mysql --version
git --version
```

### Install di macOS (menggunakan Homebrew)

```bash
# Install Homebrew jika belum ada
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install PHP & Composer
brew install php composer

# Install Node.js
brew install node

# Install MySQL
brew install mysql
brew services start mysql
```

### Install di Windows

1. Download **Laragon** dari [laragon.org](https://laragon.org) — sudah include PHP, MySQL, Composer, dan Node.js
2. Atau install manual: PHP, Composer, Node.js, dan XAMPP/WAMP

### Install di Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install php8.2 php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip
sudo apt install mysql-server nodejs npm
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

### Extension PHP yang Wajib Aktif

Pastikan extension berikut aktif (cek dengan `php -m`):

- `pdo_mysql`
- `mbstring`
- `openssl`
- `tokenizer`
- `xml`
- `ctype`
- `json`
- `bcmath`
- `fileinfo`

---

## 3. Instalasi Laravel

### Cara 1: Menggunakan Composer (Recommended)

```bash
# Buat project baru
composer create-project laravel/laravel nama-project

# Masuk ke folder project
cd nama-project
```

### Cara 2: Menggunakan Laravel Installer

```bash
# Install Laravel Installer global
composer global require laravel/installer

# Buat project baru
laravel new nama-project
```

### Clone Project yang Sudah Ada

Jika bergabung ke project yang sudah berjalan:

```bash
git clone <url-repository>
cd nama-project
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
npm install
npm run dev
```

---

## 4. Menjalankan Project

### Setup Environment (.env)

File `.env` menyimpan konfigurasi project (database, mail, dll.). Jangan pernah di-commit ke Git!

```bash
# Copy file environment
cp .env.example .env

# Generate application key
php artisan key:generate
```

Edit file `.env` dan sesuaikan konfigurasi database:

```env
APP_NAME="Nama Aplikasi"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_database
DB_USERNAME=root
DB_PASSWORD=
```

### Buat Database

```bash
# Login ke MySQL
mysql -u root -p

# Buat database
CREATE DATABASE nama_database;
EXIT;
```

### Jalankan Migration

```bash
php artisan migrate
```

### Jalankan Development Server

```bash
# Terminal 1: Laravel server
php artisan serve

# Terminal 2: Vite (untuk CSS/JS)
npm run dev
```

Buka browser: **http://localhost:8000**

---

## 5. Struktur Folder Laravel

```
nama-project/
├── app/                    # Inti aplikasi
│   ├── Http/
│   │   ├── Controllers/    # Controller (logika bisnis)
│   │   └── Middleware/     # Filter request (auth, cors, dll.)
│   ├── Models/             # Model Eloquent (representasi tabel DB)
│   └── Providers/          # Service provider
├── bootstrap/              # Bootstrap framework
├── config/                 # File konfigurasi
├── database/
│   ├── migrations/         # File migration (struktur tabel)
│   ├── seeders/            # Data dummy untuk testing
│   └── factories/          # Generator data fake
├── public/                 # Entry point web (index.php)
├── resources/
│   ├── views/              # File Blade template (HTML)
│   ├── css/                # File CSS
│   └── js/                 # File JavaScript
├── routes/
│   ├── web.php             # Route untuk web (browser)
│   └── api.php             # Route untuk API
├── storage/                # File upload, cache, log
├── tests/                  # Unit & feature test
├── .env                    # Konfigurasi environment (JANGAN di-commit!)
├── artisan                 # Command-line tool
└── composer.json           # Dependency PHP
```

### Folder yang Paling Sering Kamu Sentuh

| Folder/File | Kapan Dipakai |
|---|---|
| `routes/web.php` | Menambah URL baru |
| `app/Http/Controllers/` | Menulis logika bisnis |
| `app/Models/` | Model database |
| `resources/views/` | Tampilan HTML |
| `database/migrations/` | Membuat/mengubah tabel |
| `database/seeders/` | Mengisi data awal |

---

## 6. Alur Kerja Request (MVC)

Laravel menggunakan pola **MVC (Model-View-Controller)**:

```
Browser Request (URL)
       ↓
   Route (web.php)        → Menentukan URL ke Controller mana
       ↓
   Controller             → Memproses logika, ambil data dari Model
       ↓
   Model (Eloquent)       → Berinteraksi dengan Database
       ↓
   View (Blade)           → Render HTML ke browser
       ↓
   Response (HTML/JSON)
```

### Contoh Alur Sederhana

1. User buka `http://localhost:8000/users`
2. Route di `web.php` mengarahkan ke `UserController@index`
3. Controller mengambil data user dari database via Model
4. Controller mengirim data ke View `users.index`
5. Blade merender HTML dan dikirim ke browser

---

## 7. Routing

Route mendefinisikan URL apa yang memanggil fungsi/controller mana.

### Route Dasar

File: `routes/web.php`

```php
use Illuminate\Support\Facades\Route;

// Route sederhana (closure)
Route::get('/', function () {
    return 'Halo, selamat datang!';
});

// Route ke Controller
Route::get('/users', [UserController::class, 'index']);

// Route dengan parameter
Route::get('/users/{id}', [UserController::class, 'show']);

// Route dengan nama (untuk dipanggil di view)
Route::get('/users', [UserController::class, 'index'])->name('users.index');
```

### HTTP Methods

| Method | Fungsi | Contoh URL |
|---|---|---|
| `GET` | Menampilkan data | `/users` |
| `POST` | Menyimpan data baru | `/users` (form submit) |
| `PUT/PATCH` | Update data | `/users/1` |
| `DELETE` | Hapus data | `/users/1` |

### Route Resource (CRUD Otomatis)

```php
use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);
```

Ini otomatis membuat 7 route:

| Method | URI | Action | Nama Route |
|---|---|---|---|
| GET | `/products` | index | products.index |
| GET | `/products/create` | create | products.create |
| POST | `/products` | store | products.store |
| GET | `/products/{id}` | show | products.show |
| GET | `/products/{id}/edit` | edit | products.edit |
| PUT/PATCH | `/products/{id}` | update | products.update |
| DELETE | `/products/{id}` | destroy | products.destroy |

### Cek Semua Route

```bash
php artisan route:list
```

---

## 8. Controller

Controller menampung logika bisnis aplikasi.

### Membuat Controller

```bash
php artisan make:controller ProductController
```

### Contoh Controller

File: `app/Http/Controllers/ProductController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // Tampilkan semua produk
    public function index()
    {
        $products = Product::latest()->paginate(10);
        return view('products.index', compact('products'));
    }

    // Form tambah produk
    public function create()
    {
        return view('products.create');
    }

    // Simpan produk baru
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name'  => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
        ]);

        Product::create($validated);

        return redirect()->route('products.index')
            ->with('success', 'Produk berhasil ditambahkan!');
    }

    // Tampilkan detail produk
    public function show(Product $product)
    {
        return view('products.show', compact('product'));
    }

    // Form edit produk
    public function edit(Product $product)
    {
        return view('products.edit', compact('product'));
    }

    // Update produk
    public function update(Request $request, Product $product)
    {
        $validated = $request->validate([
            'name'  => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
        ]);

        $product->update($validated);

        return redirect()->route('products.index')
            ->with('success', 'Produk berhasil diupdate!');
    }

    // Hapus produk
    public function destroy(Product $product)
    {
        $product->delete();

        return redirect()->route('products.index')
            ->with('success', 'Produk berhasil dihapus!');
    }
}
```

---

## 9. View & Blade Template

View adalah file HTML yang ditampilkan ke user. Laravel menggunakan **Blade** sebagai template engine.

### Sintaks Blade Penting

```blade
{{-- Komentar Blade --}}

{{-- Output variabel (auto escape XSS) --}}
<h1>{{ $title }}</h1>

{{-- Output tanpa escape (hati-hati!) --}}
{!! $htmlContent !!}

{{-- If/Else --}}
@if($products->count() > 0)
    <p>Ada {{ $products->count() }} produk</p>
@else
    <p>Belum ada produk</p>
@endif

{{-- Loop --}}
@foreach($products as $product)
    <div>{{ $product->name }} - Rp {{ number_format($product->price) }}</div>
@endforeach

{{-- Include partial view --}}
@include('partials.navbar')

{{-- Extend layout --}}
@extends('layouts.app')

@section('content')
    <h1>Halaman Produk</h1>
@endsection

{{-- CSRF Token (WAJIB di setiap form POST) --}}
<form method="POST" action="{{ route('products.store') }}">
    @csrf
    ...
</form>

{{-- Method spoofing untuk PUT/DELETE --}}
<form method="POST" action="{{ route('products.update', $product) }}">
    @csrf
    @method('PUT')
    ...
</form>
```

### Layout Template

File: `resources/views/layouts/app.blade.php`

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Aplikasi')</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
  @include('partials.navbar')

  <main class="container">
    {{-- Flash message --}}
    @if(session('success'))
      <div class="alert alert-success">{{ session('success') }}</div>
    @endif

    @yield('content')
  </main>
</body>
</html>
```

File: `resources/views/products/index.blade.php`

```blade
@extends('layouts.app')

@section('title', 'Daftar Produk')

@section('content')
  <h1>Daftar Produk</h1>

  <a href="{{ route('products.create') }}" class="btn btn-primary">Tambah Produk</a>

  <table>
    <thead>
      <tr>
        <th>No</th>
        <th>Nama</th>
        <th>Harga</th>
        <th>Aksi</th>
      </tr>
    </thead>
    <tbody>
      @foreach($products as $index => $product)
        <tr>
          <td>{{ $products->firstItem() + $index }}</td>
          <td>{{ $product->name }}</td>
          <td>Rp {{ number_format($product->price) }}</td>
          <td>
            <a href="{{ route('products.edit', $product) }}">Edit</a>
            <form action="{{ route('products.destroy', $product) }}" method="POST" style="display:inline">
              @csrf
              @method('DELETE')
              <button type="submit" onclick="return confirm('Yakin hapus?')">Hapus</button>
            </form>
          </td>
        </tr>
      @endforeach
    </tbody>
  </table>

  {{ $products->links() }}
@endsection
```

---

## 10. Model & Eloquent ORM

Model merepresentasikan tabel di database. Eloquent ORM memungkinkan kamu berinteraksi dengan database menggunakan PHP, bukan SQL mentah.

### Membuat Model

```bash
# Model saja
php artisan make:model Product

# Model + Migration sekaligus
php artisan make:model Product -m
```

### Contoh Model

File: `app/Models/Product.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
  protected $fillable = [
    'name',
    'description',
    'price',
    'stock',
  ];

  protected $casts = [
    'price' => 'decimal:2',
  ];
}
```

### Query Eloquent yang Sering Dipakai

```php
// Ambil semua
$products = Product::all();

// Ambil dengan kondisi
$products = Product::where('price', '>', 10000)->get();

// Ambil satu
$product = Product::find(1);
$product = Product::where('name', 'Laptop')->first();

// Buat data baru
Product::create(['name' => 'Laptop', 'price' => 5000000]);

// Update
$product = Product::find(1);
$product->update(['price' => 4500000]);

// Hapus
$product->delete();

// Pagination
$products = Product::paginate(10);

// Urutkan
$products = Product::orderBy('created_at', 'desc')->get();

// Hitung
$count = Product::count();
```

---

## 11. Migration & Database

Migration adalah version control untuk struktur database. Setiap perubahan tabel dicatat dalam file migration.

### Membuat Migration

```bash
php artisan make:migration create_products_table
```

### Contoh Migration

File: `database/migrations/xxxx_create_products_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
  public function up(): void
  {
    Schema::create('products', function (Blueprint $table) {
      $table->id();
      $table->string('name');
      $table->text('description')->nullable();
      $table->decimal('price', 12, 2);
      $table->integer('stock')->default(0);
      $table->timestamps(); // created_at & updated_at otomatis
    });
  }

  public function down(): void
  {
    Schema::dropIfExists('products');
  }
};
```

### Tipe Kolom yang Sering Dipakai

| Method | Tipe Database |
|---|---|
| `$table->id()` | BIGINT auto increment (primary key) |
| `$table->string('name')` | VARCHAR(255) |
| `$table->text('description')` | TEXT |
| `$table->integer('stock')` | INTEGER |
| `$table->decimal('price', 12, 2)` | DECIMAL |
| `$table->boolean('is_active')` | BOOLEAN |
| `$table->date('published_at')` | DATE |
| `$table->timestamp('deleted_at')` | TIMESTAMP |
| `$table->foreignId('user_id')` | Foreign key ke tabel users |
| `$table->timestamps()` | created_at & updated_at |

### Menjalankan Migration

```bash
# Jalankan semua migration
php artisan migrate

# Rollback migration terakhir
php artisan migrate:rollback

# Reset semua & jalankan ulang
php artisan migrate:fresh

# Reset + jalankan seeder
php artisan migrate:fresh --seed
```

### Seeder (Data Awal)

```bash
php artisan make:seeder ProductSeeder
```

File: `database/seeders/ProductSeeder.php`

```php
<?php

namespace Database\Seeders;

use App\Models\Product;
use Illuminate\Database\Seeder;

class ProductSeeder extends Seeder
{
  public function run(): void
  {
    Product::create([
      'name' => 'Laptop ASUS',
      'description' => 'Laptop gaming performa tinggi',
      'price' => 12000000,
      'stock' => 5,
    ]);

    Product::create([
      'name' => 'Mouse Logitech',
      'description' => 'Mouse wireless ergonomis',
      'price' => 350000,
      'stock' => 20,
    ]);
  }
}
```

Daftarkan di `database/seeders/DatabaseSeeder.php`:

```php
public function run(): void
{
  $this->call(ProductSeeder::class);
}
```

Jalankan:

```bash
php artisan db:seed
```

---

## 12. Tutorial CRUD Lengkap

Ikuti langkah-langkah ini untuk membuat fitur CRUD Produk dari nol.

### Langkah 1: Buat Model & Migration

```bash
php artisan make:model Product -m
```

Edit file migration (lihat contoh di [Bagian 11](#11-migration--database)), lalu jalankan:

```bash
php artisan migrate
```

### Langkah 2: Buat Controller

```bash
php artisan make:controller ProductController --resource
```

Flag `--resource` otomatis membuat 7 method CRUD.

### Langkah 3: Daftarkan Route

File: `routes/web.php`

```php
use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);
```

### Langkah 4: Buat View

```bash
mkdir -p resources/views/products
```

Buat file-file berikut:
- `resources/views/layouts/app.blade.php` (layout utama)
- `resources/views/products/index.blade.php` (daftar produk)
- `resources/views/products/create.blade.php` (form tambah)
- `resources/views/products/edit.blade.php` (form edit)
- `resources/views/products/show.blade.php` (detail produk)

### Langkah 5: Isi Controller

Lihat contoh lengkap di [Bagian 8](#8-controller).

### Langkah 6: Test

```bash
php artisan serve
```

Buka `http://localhost:8000/products` dan coba:
- [ ] Lihat daftar produk
- [ ] Tambah produk baru
- [ ] Edit produk
- [ ] Hapus produk

---

## 13. Validasi Form

Laravel menyediakan validasi built-in yang powerful.

### Validasi di Controller

```php
public function store(Request $request)
{
  $validated = $request->validate([
    'name'        => 'required|string|max:255',
    'email'       => 'required|email|unique:users,email',
    'price'       => 'required|numeric|min:0',
    'stock'       => 'required|integer|min:0',
    'category_id' => 'required|exists:categories,id',
    'image'       => 'nullable|image|mimes:jpg,png|max:2048',
  ]);

  Product::create($validated);
}
```

### Rule Validasi yang Sering Dipakai

| Rule | Keterangan |
|---|---|
| `required` | Wajib diisi |
| `nullable` | Boleh kosong |
| `string` | Harus string |
| `integer` | Harus angka bulat |
| `numeric` | Harus angka |
| `email` | Format email valid |
| `min:3` | Minimal 3 karakter/angka |
| `max:255` | Maksimal 255 karakter |
| `unique:table,column` | Nilai harus unik di tabel |
| `exists:table,column` | Nilai harus ada di tabel |
| `confirmed` | Harus cocok dengan field `_confirmation` |
| `image` | Harus file gambar |
| `mimes:jpg,png` | Format file yang diizinkan |

### Tampilkan Error di View

```blade
<input type="text" name="name" value="{{ old('name') }}">
@error('name')
  <span class="text-danger">{{ $message }}</span>
@enderror
```

Fungsi `old('name')` mengembalikan nilai input sebelumnya jika validasi gagal.

---

## 14. Relasi Database

Eloquent mendukung relasi antar tabel.

### One to Many (Satu ke Banyak)

Contoh: Satu **Category** punya banyak **Product**.

```php
// app/Models/Category.php
class Category extends Model
{
  public function products()
  {
    return $this->hasMany(Product::class);
  }
}

// app/Models/Product.php
class Product extends Model
{
  public function category()
  {
    return $this->belongsTo(Category::class);
  }
}
```

Penggunaan:

```php
// Ambil semua produk dari kategori
$category = Category::find(1);
$products = $category->products;

// Ambil kategori dari produk
$product = Product::find(1);
$categoryName = $product->category->name;

// Eager loading (hindari N+1 problem)
$products = Product::with('category')->get();
```

### One to One

```php
// User punya satu Profile
class User extends Model
{
  public function profile()
  {
    return $this->hasOne(Profile::class);
  }
}
```

### Many to Many

```php
// Student punya banyak Course, Course punya banyak Student
class Student extends Model
{
  public function courses()
  {
    return $this->belongsToMany(Course::class);
  }
}
```

---

## 15. Authentication (Login & Register)

Laravel menyediakan sistem autentikasi siap pakai.

### Install Laravel Breeze (Recommended untuk pemula)

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install
npm run dev
php artisan migrate
```

Ini otomatis membuat:
- Halaman login & register
- Halaman dashboard
- Middleware auth
- Route terproteksi

### Proteksi Route

```php
// Hanya user yang sudah login
Route::middleware('auth')->group(function () {
  Route::resource('products', ProductController::class);
});

// Hanya guest (belum login)
Route::middleware('guest')->group(function () {
  // route login/register
});
```

### Cek User di Controller/View

```php
// Di Controller
$user = auth()->user();
$userId = auth()->id();

// Di Blade
@auth
  <p>Halo, {{ auth()->user()->name }}</p>
@endauth

@guest
  <a href="{{ route('login') }}">Login</a>
@endguest
```

---

## 16. Artisan Command yang Sering Dipakai

Artisan adalah CLI bawaan Laravel. Buka terminal di root project.

### Membuat File

```bash
php artisan make:controller NamaController       # Controller
php artisan make:controller NamaController -r  # Resource Controller
php artisan make:model NamaModel               # Model
php artisan make:model NamaModel -m              # Model + Migration
php artisan make:migration create_table_name     # Migration
php artisan make:seeder NamaSeeder               # Seeder
php artisan make:request NamaRequest             # Form Request (validasi)
php artisan make:middleware NamaMiddleware       # Middleware
```

### Database

```bash
php artisan migrate                  # Jalankan migration
php artisan migrate:rollback         # Batalkan migration terakhir
php artisan migrate:fresh            # Drop semua tabel & migrate ulang
php artisan migrate:fresh --seed     # Fresh + isi data seeder
php artisan db:seed                  # Jalankan seeder
```

### Development

```bash
php artisan serve                    # Jalankan dev server (port 8000)
php artisan serve --port=8080        # Custom port
php artisan route:list               # Lihat semua route
php artisan tinker                   # REPL interaktif (test query)
```

### Cache

```bash
php artisan cache:clear              # Hapus cache
php artisan config:clear             # Hapus config cache
php artisan view:clear               # Hapus compiled view
php artisan optimize:clear           # Hapus semua cache sekaligus
```

### Tinker (Testing Cepat)

```bash
php artisan tinker

>>> App\Models\Product::count()
>>> App\Models\Product::create(['name' => 'Test', 'price' => 1000])
>>> App\Models\Product::find(1)
```

---

## 17. Best Practice

### Coding

1. **Ikuti konvensi Laravel** — nama Controller pakai PascalCase, method pakai camelCase, route pakai kebab-case
2. **Jangan tulis logika di View** — semua logika di Controller atau Service class
3. **Gunakan `$fillable` di Model** — jangan pakai `$guarded = []` di production
4. **Selalu validasi input** — jangan percaya data dari user
5. **Pakai Route Model Binding** — `public function show(Product $product)` lebih aman dari `find($id)`
6. **Eager loading** — pakai `with()` untuk hindari N+1 query
7. **Pagination** — jangan `::all()` untuk data besar, pakai `::paginate()`

### Keamanan

1. **Selalu pakai `@csrf`** di setiap form POST
2. **Jangan commit file `.env`** — sudah ada di `.gitignore`
3. **Pakai `{{ }}` bukan `{!! !!}`** kecuali memang perlu render HTML
4. **Set `APP_DEBUG=false`** di production

### Kerja Tim

1. **Buat branch baru** untuk setiap fitur (`git checkout -b fitur/nama-fitur`)
2. **Commit message yang jelas** — contoh: `feat: tambah CRUD produk`, `fix: perbaiki validasi harga`
3. **Pull sebelum push** — `git pull origin main` dulu sebelum push
4. **Jangan push langsung ke main** — selalu lewat Pull Request
5. **Tanya jika bingung** — lebih baik tanya daripada asal coding

### Struktur Commit Message

```
feat: tambah fitur baru
fix: perbaiki bug
docs: update dokumentasi
refactor: perbaiki struktur kode tanpa ubah fungsi
style: perbaiki format/indentasi
test: tambah atau perbaiki test
```

---

## 18. Git Workflow

### Flow Harian

```bash
# 1. Pastikan di branch main dan up-to-date
git checkout main
git pull origin main

# 2. Buat branch baru untuk task kamu
git checkout -b fitur/daftar-produk

# 3. Coding... lalu cek perubahan
git status
git diff

# 4. Stage & commit
git add .
git commit -m "feat: tambah halaman daftar produk"

# 5. Push branch
git push -u origin fitur/daftar-produk

# 6. Buat Pull Request di GitHub/GitLab
# 7. Tunggu review dari tim, lalu merge
```

### Command Git Penting

```bash
git status              # Lihat status file
git log --oneline -10   # Lihat 10 commit terakhir
git diff                # Lihat perubahan
git stash               # Simpan perubahan sementara
git stash pop           # Kembalikan perubahan yang di-stash
git branch              # Lihat semua branch
git checkout nama-branch # Pindah branch
```

---

## 19. Troubleshooting

### Error Umum & Solusinya

#### `SQLSTATE[HY000] [1045] Access denied`

Database credentials salah. Cek file `.env`:

```env
DB_USERNAME=root
DB_PASSWORD=password_kamu
```

#### `SQLSTATE[42S02] Base table or view not found`

Tabel belum dibuat. Jalankan migration:

```bash
php artisan migrate
```

#### `No application encryption key has been specified`

```bash
php artisan key:generate
```

#### `Class "App\Models\XXX" not found`

```bash
composer dump-autoload
```

#### `Vite manifest not found`

```bash
npm install
npm run dev
```

#### `Permission denied` pada folder storage

```bash
chmod -R 775 storage bootstrap/cache
```

#### Halaman blank / error 500

1. Cek `storage/logs/laravel.log` untuk detail error
2. Pastikan `APP_DEBUG=true` di `.env` saat development
3. Clear cache:

```bash
php artisan optimize:clear
```

#### Port 8000 sudah dipakai

```bash
php artisan serve --port=8080
```

---

## 20. Referensi & Latihan

### Dokumentasi Resmi

- [Laravel Documentation](https://laravel.com/docs) — baca ini kalau bingung
- [Laracasts](https://laracasts.com) — video tutorial Laravel (ada yang gratis)
- [Laravel News](https://laravel-news.com) — berita & tips Laravel

### Latihan Praktik

Kerjakan latihan berikut secara berurutan:

#### Level 1: Dasar
- [ ] Install Laravel dan jalankan `php artisan serve`
- [ ] Buat route `/halo` yang menampilkan nama kamu
- [ ] Buat halaman "Tentang Saya" dengan Blade template
- [ ] Buat layout master dan extend di 3 halaman

#### Level 2: Database
- [ ] Buat tabel `tasks` (id, title, description, is_done, timestamps)
- [ ] Buat Model Task dan migration
- [ ] Isi data dummy dengan Seeder
- [ ] Tampilkan semua task di halaman index

#### Level 3: CRUD
- [ ] Buat CRUD lengkap untuk Task (Create, Read, Update, Delete)
- [ ] Tambahkan validasi form
- [ ] Tambahkan pagination
- [ ] Tambahkan flash message sukses/error

#### Level 4: Relasi
- [ ] Buat tabel `categories` dan relasikan dengan `products`
- [ ] Tampilkan nama kategori di daftar produk
- [ ] Filter produk berdasarkan kategori

#### Level 5: Authentication
- [ ] Install Laravel Breeze
- [ ] Proteksi halaman CRUD agar hanya user login yang bisa akses
- [ ] Tampilkan nama user di navbar

#### Level 6: Advanced
- [ ] Upload gambar produk
- [ ] Buat pencarian produk
- [ ] Buat API endpoint JSON untuk daftar produk
- [ ] Tulis Feature Test untuk CRUD produk

---

## Cheat Sheet Cepat

```bash
# Setup project baru
composer create-project laravel/laravel my-app
cd my-app
cp .env.example .env
php artisan key:generate
# edit .env (database)
php artisan migrate
npm install && npm run dev
php artisan serve

# Buat CRUD lengkap
php artisan make:model Product -m
php artisan make:controller ProductController --resource
# edit migration, model, controller, views
# tambah route: Route::resource('products', ProductController::class);
php artisan migrate

# Debug
php artisan route:list
php artisan tinker
tail -f storage/logs/laravel.log
```

---

> **Tips:** Jangan takut error — setiap developer pasti mengalaminya. Baca pesan errornya dengan teliti, cek `storage/logs/laravel.log`, dan cari solusinya di dokumentasi atau komunitas. Jika sudah cukup lama terjebak, diskusikan dengan rekan tim. Selamat belajar!
