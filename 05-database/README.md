# Modul 05 — Database

Modul kelima: menghubungkan **Task Manager** ke **MySQL** — migration, model, seeder, Eloquent, dan halaman list task dari database.

**Estimasi waktu:** 2 hari  
**Prasyarat:** [Modul 04 — Blade Template](../04-blade-template/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Memahami konsep migration dan cara menjalankannya
- [ ] Membuat model Eloquent dan mengonfigurasi `$fillable`
- [ ] Mengisi database dengan seeder
- [ ] Menulis query Eloquent di controller
- [ ] Mengeksplorasi data lewat `php artisan tinker`
- [ ] Menampilkan list task dari database di `/tasks`
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
database/migrations/..._create_tasks_table.php
app/Models/Task.php
database/seeders/TaskSeeder.php
app/Http/Controllers/TaskController.php   ← method index saja
resources/views/tasks/index.blade.php
GET /tasks → tasks.index
```

> Modul ini fokus **Read** (baca data) saja. CRUD lengkap (Create, Update, Delete) ada di [Modul 06](../06-crud-task/README.md).

---

## Apa yang Akan Dibuak?

Sebelum modul ini, halaman Task Manager hanya menampilkan HTML statis. Setelah modul ini:

1. Tabel `tasks` ada di database MySQL `task_manager`
2. Data dummy terisi lewat seeder
3. Halaman `/tasks` menampilkan task dari database — bukan hardcode di view
4. Kamu bisa query data lewat tinker

---

## Konsep: Mengapa Database?

Aplikasi web nyata menyimpan data secara permanen. Tanpa database:

- Data hilang saat server restart
- Tidak bisa share data antar user
- Tidak ada riwayat perubahan

Laravel menggunakan **Eloquent ORM** — cara PHP berinteraksi dengan MySQL tanpa menulis SQL mentah di setiap tempat.

```
Browser  →  Route  →  Controller  →  Model (Eloquent)  →  MySQL
                ↑                              ↓
              View  ←  data dikirim ke Blade  ←
```

---

## Konsep: Migration

**Migration** = file PHP yang mendeskripsikan struktur tabel database. Seperti "blueprint" atau versi kontrol untuk schema DB.

**Keuntungan migration:**

| Tanpa migration | Dengan migration |
|---|---|
| Buat tabel manual di phpMyAdmin | Schema tercatat di kode |
| Tim sulit sinkron struktur DB | `php artisan migrate` di semua mesin |
| Tidak ada riwayat perubahan | Setiap perubahan punya file migration |

**Perintah migration penting:**

```bash
php artisan migrate          # Jalankan migration yang belum dijalankan
php artisan migrate:status   # Lihat status semua migration
php artisan migrate:rollback # Batalkan migration terakhir (1 batch)
php artisan migrate:fresh    # Hapus semua tabel & migrate ulang (HATI-HATI: data hilang!)
```

Migration disimpan di folder `database/migrations/`. Nama file mengandung timestamp agar urutan eksekusi konsisten.

---

## Konsep: Model Eloquent

**Model** = class PHP yang mewakili satu tabel database. Satu model `Task` = satu tabel `tasks`.

Eloquent otomatis:

- Mengetahui nama tabel (`tasks` dari model `Task`)
- Mengenali primary key (`id`)
- Mengelola kolom `created_at` dan `updated_at`

Contoh query Eloquent vs SQL:

```php
// Eloquent
Task::where('is_done', false)->get();

// SQL setara
// SELECT * FROM tasks WHERE is_done = 0
```

---

## Konsep: Seeder

**Seeder** = script untuk mengisi database dengan data awal (dummy data untuk development).

Kapan dipakai:

- Data contoh saat belajar
- Data master (role, kategori default)
- Testing tanpa input manual

---

## Langkah 1: Buat Migration & Model

Jalankan perintah artisan:

```bash
php artisan make:model Task -m
```

**Penjelasan flag `-m`:**
- Membuat model **dan** migration sekaligus
- `-m` singkatan dari `--migration`

File yang dibuat:

```
app/Models/Task.php
database/migrations/2026_07_07_000000_create_tasks_table.php
```

(Timestamp di nama file akan berbeda di mesin kamu — itu normal.)

---

## Langkah 2: Edit Migration

Buka file migration `database/migrations/xxxx_create_tasks_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Jalankan saat php artisan migrate
     */
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();                              // BIGINT UNSIGNED, auto increment, PK
            $table->string('title');                   // VARCHAR(255), NOT NULL
            $table->text('description')->nullable();   // TEXT, boleh kosong
            $table->boolean('is_done')->default(false); // TINYINT(1), default 0
            $table->timestamps();                      // created_at & updated_at
        });
    }

    /**
     * Jalankan saat php artisan migrate:rollback
     */
    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

**Penjelasan kolom:**

| Method | Tipe MySQL | Keterangan |
|---|---|---|
| `$table->id()` | BIGINT UNSIGNED | Primary key auto-increment |
| `$table->string('title')` | VARCHAR(255) | Judul task, wajib diisi |
| `$table->text('description')->nullable()` | TEXT NULL | Deskripsi, boleh kosong |
| `$table->boolean('is_done')->default(false)` | TINYINT(1) | 0 = belum, 1 = selesai |
| `$table->timestamps()` | TIMESTAMP | `created_at` & `updated_at` otomatis |

**Method `up()` vs `down()`:**
- `up()` — membuat tabel (saat migrate)
- `down()` — menghapus tabel (saat rollback)

---

## Langkah 3: Jalankan Migration

Pastikan `.env` sudah mengarah ke MySQL (dari Modul 01):

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager
DB_USERNAME=root
DB_PASSWORD=
```

Jalankan migration:

```bash
php artisan migrate
```

Output yang diharapkan:

```
  INFO  Running migrations.

  2026_07_07_000000_create_tasks_table ........................ DONE
```

Verifikasi di MySQL:

```bash
mysql -u root -p task_manager
```

```sql
DESCRIBE tasks;
SELECT * FROM tasks;
EXIT;
```

Tabel `tasks` harus muncul dengan kolom yang benar. Saat ini masih kosong — seeder mengisi di langkah berikutnya.

---

## Langkah 4: Konfigurasi Model

Edit `app/Models/Task.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    /**
     * Kolom yang boleh diisi mass assignment (create/update sekaligus).
     * Tanpa ini, Task::create([...]) akan error.
     */
    protected $fillable = [
        'title',
        'description',
        'is_done',
    ];

    /**
     * Cast: ubah tipe data saat dibaca/ditulis.
     * is_done di DB = 0/1, di PHP = true/false
     */
    protected $casts = [
        'is_done' => 'boolean',
    ];
}
```

**Penjelasan `$fillable`:**

Laravel melindungi dari **mass assignment vulnerability** — mencegah user mengisi kolom yang tidak seharusnya (misalnya `is_admin`).

Hanya kolom di `$fillable` yang boleh diisi lewat:

```php
Task::create(['title' => '...', 'description' => '...']);
$task->update(['title' => '...']);
```

**Penjelasan `$casts`:**

Di MySQL, boolean disimpan sebagai `0` atau `1`. Dengan cast `'is_done' => 'boolean'`, kamu bisa pakai:

```php
if ($task->is_done) { ... }  // true/false, bukan 0/1
```

---

## Langkah 5: Buat Seeder

```bash
php artisan make:seeder TaskSeeder
```

Edit `database/seeders/TaskSeeder.php`:

```php
<?php

namespace Database\Seeders;

use App\Models\Task;
use Illuminate\Database\Seeder;

class TaskSeeder extends Seeder
{
    public function run(): void
    {
        $tasks = [
            [
                'title'       => 'Setup project Laravel',
                'description' => 'Install Laravel, konfigurasi .env, hubungkan MySQL',
                'is_done'     => true,
            ],
            [
                'title'       => 'Pelajari Routing & Controller',
                'description' => 'Memahami alur request → route → controller → view',
                'is_done'     => true,
            ],
            [
                'title'       => 'Buat layout Blade + Bootstrap CDN',
                'description' => 'Layout master, navbar, partial alert',
                'is_done'     => true,
            ],
            [
                'title'       => 'Hubungkan ke database MySQL',
                'description' => 'Migration, model Task, seeder, halaman list',
                'is_done'     => false,
            ],
            [
                'title'       => 'Buat CRUD task lengkap',
                'description' => 'Form tambah, edit, hapus dengan validasi',
                'is_done'     => false,
            ],
            [
                'title'       => 'Implementasi login & register',
                'description' => 'Auth manual tanpa Laravel UI',
                'is_done'     => false,
            ],
            [
                'title'       => 'Dashboard & kategori',
                'description' => 'Statistik task, filter, search',
                'is_done'     => false,
            ],
        ];

        foreach ($tasks as $task) {
            Task::create($task);
        }
    }
}
```

Daftarkan seeder di `database/seeders/DatabaseSeeder.php`:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            TaskSeeder::class,
        ]);
    }
}
```

Jalankan seeder:

```bash
php artisan db:seed --class=TaskSeeder
```

Atau seed semua:

```bash
php artisan db:seed
```

Verifikasi:

```bash
php artisan tinker
```

```php
App\Models\Task::count();  // harus 7
exit
```

---

## Langkah 6: Controller (index saja)

```bash
php artisan make:controller TaskController
```

Edit `app/Http/Controllers/TaskController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;

class TaskController extends Controller
{
    /**
     * Tampilkan daftar semua task dari database.
     */
    public function index()
    {
        // latest() = ORDER BY created_at DESC (task terbaru di atas)
        // get()    = ambil semua record sebagai Collection
        $tasks = Task::latest()->get();

        // Kirim $tasks ke view tasks/index.blade.php
        return view('tasks.index', compact('tasks'));
    }
}
```

**Penjelasan query:**

```php
Task::latest()->get();
```

Setara dengan:

```sql
SELECT * FROM tasks ORDER BY created_at DESC
```

**Alternatif query yang sering dipakai:**

```php
Task::all();                              // Semua task, urutan id
Task::where('is_done', true)->get();      // Hanya yang selesai
Task::where('is_done', false)->get();     // Hanya yang belum
Task::orderBy('title')->get();            // Urut judul A-Z
Task::find(1);                            // Satu task by id
Task::count();                            // Jumlah total
Task::where('title', 'like', '%Laravel%')->get();  // Cari judul
```

---

## Langkah 7: View — Halaman List Task

Buat folder dan file:

```bash
mkdir -p resources/views/tasks
```

Buat `resources/views/tasks/index.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Daftar Task')

@section('content')
    <div class="d-flex justify-content-between align-items-center mb-4">
        <div>
            <h1 class="h2 fw-bold mb-1">Daftar Task</h1>
            <p class="text-muted mb-0 small">Data diambil dari database MySQL</p>
        </div>
        <span class="badge bg-primary fs-6">{{ $tasks->count() }} task</span>
    </div>

    @if($tasks->isEmpty())
        <div class="alert alert-info">
            Belum ada task di database.
            Jalankan <code>php artisan db:seed --class=TaskSeeder</code>
        </div>
    @else
        <div class="list-group shadow-sm">
            @foreach($tasks as $task)
                <div class="list-group-item list-group-item-action d-flex justify-content-between align-items-start py-3">
                    <div class="me-3">
                        <h5 class="mb-1 fw-semibold {{ $task->is_done ? 'text-decoration-line-through text-muted' : '' }}">
                            {{ $task->title }}
                        </h5>
                        @if($task->description)
                            <p class="mb-1 small text-muted">{{ $task->description }}</p>
                        @endif
                        <small class="text-muted">
                            Dibuat: {{ $task->created_at->format('d M Y, H:i') }}
                        </small>
                    </div>
                    <span class="badge {{ $task->is_done ? 'bg-success' : 'bg-warning text-dark' }} align-self-center">
                        {{ $task->is_done ? 'Selesai' : 'Belum' }}
                    </span>
                </div>
            @endforeach
        </div>
    @endif
@endsection
```

**Penjelasan class Bootstrap:**

| Class | Fungsi |
|---|---|
| `list-group` | Container daftar vertikal |
| `list-group-item` | Satu item dalam list |
| `badge bg-success` | Label hijau "Selesai" |
| `badge bg-warning text-dark` | Label kuning "Belum" |
| `text-decoration-line-through` | Coret teks task selesai |
| `d-flex justify-content-between` | Judul kiri, badge kanan |

---

## Langkah 8: Route & Navbar

Edit `routes/web.php` — tambahkan import dan route:

```php
<?php

use App\Http\Controllers\HomeController;
use App\Http\Controllers\PageController;
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Route;

Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/tentang', [PageController::class, 'about'])->name('about');

// Route baru Modul 05
Route::get('/tasks', [TaskController::class, 'index'])->name('tasks.index');
```

Cek route terdaftar:

```bash
php artisan route:list --name=tasks
```

Output:

```
GET|HEAD  tasks .......... tasks.index › TaskController@index
```

Update `resources/views/partials/navbar.blade.php` — tambah link Tasks:

```blade
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('tasks.*') ? 'active fw-semibold' : '' }}"
       href="{{ route('tasks.index') }}">Tasks</a>
</li>
```

---

## Langkah 9: Uji di Browser

```bash
php artisan serve
```

Buka `http://localhost:8000/tasks`

**Yang harus terlihat:**
- List 7 task dari seeder
- Badge "Selesai" (hijau) dan "Belum" (kuning)
- Task selesai judulnya dicoret
- Tanggal dibuat di bawah deskripsi
- Navbar link "Tasks" aktif (highlight)

---

## Eksplorasi: Artisan Tinker

**Tinker** = REPL (Read-Eval-Print Loop) untuk Laravel. Tempat eksperimen query tanpa buat route/controller.

```bash
php artisan tinker
```

### Query dasar

```php
// Hitung jumlah task
App\Models\Task::count();

// Ambil semua
App\Models\Task::all();

// Ambil satu by id
App\Models\Task::find(1);

// Task belum selesai
App\Models\Task::where('is_done', false)->get();

// Task selesai
App\Models\Task::where('is_done', true)->get();

// Buat task baru (langsung ke DB!)
App\Models\Task::create([
    'title' => 'Task dari Tinker',
    'description' => 'Dibuat lewat tinker',
    'is_done' => false,
]);

// Update task
$task = App\Models\Task::find(1);
$task->title = 'Judul diupdate';
$task->save();

// Hapus task
App\Models\Task::find(8)?->delete();
```

### Chain query

```php
App\Models\Task::where('is_done', false)
    ->orderBy('title')
    ->take(3)
    ->get(['id', 'title']);  // hanya kolom id & title
```

Keluar dari tinker:

```php
exit
```

---

## Referensi Eloquent Query

| Method | SQL setara | Keterangan |
|---|---|---|
| `Task::all()` | `SELECT * FROM tasks` | Semua record |
| `Task::find($id)` | `SELECT * WHERE id = ?` | Satu record atau null |
| `Task::where('col', $val)->get()` | `SELECT * WHERE col = ?` | Filter |
| `Task::where('col', 'like', '%x%')->get()` | `LIKE '%x%'` | Pencarian partial |
| `Task::orderBy('title')->get()` | `ORDER BY title` | Urutkan |
| `Task::latest()->get()` | `ORDER BY created_at DESC` | Terbaru dulu |
| `Task::count()` | `SELECT COUNT(*)` | Jumlah |
| `Task::create([...])` | `INSERT INTO ...` | Buat record |
| `$task->update([...])` | `UPDATE ... WHERE id = ?` | Update record |
| `$task->delete()` | `DELETE WHERE id = ?` | Hapus record |

Dokumentasi lengkap: [Laravel Eloquent](https://laravel.com/docs/eloquent)

---

## Latihan Modul 05

Kerjakan latihan berikut untuk memperdalam pemahaman. Setiap latihan bisa dikerjakan terpisah.

### Latihan 1: Kolom Priority

1. Buat migration baru: `php artisan make:migration add_priority_to_tasks_table`
2. Tambah kolom `priority` enum/string: `low`, `medium`, `high` (default `medium`)
3. Jalankan `php artisan migrate`
4. Update model `$fillable` dan seeder
5. Tampilkan badge warna berbeda di view:
   - `low` → `badge bg-secondary`
   - `medium` → `badge bg-primary`
   - `high` → `badge bg-danger`

### Latihan 2: Halaman Task Pending

1. Tambah method `pending()` di `TaskController`
2. Query: `Task::where('is_done', false)->latest()->get()`
3. Buat route `GET /tasks/pending` → name `tasks.pending`
4. Buat view `tasks/pending.blade.php` (salin dari index, sesuaikan)
5. Tambah link "Belum Selesai" di navbar

### Latihan 3: Statistik di Atas List

Di controller, kirim data statistik ke view:

```php
$total   = Task::count();
$done    = Task::where('is_done', true)->count();
$pending = $total - $done;

return view('tasks.index', compact('tasks', 'total', 'done', 'pending'));
```

Tampilkan di view dengan 3 card Bootstrap kecil:

```blade
<div class="row g-3 mb-4">
    <div class="col-md-4">
        <div class="card text-center">
            <div class="card-body py-2">
                <strong>{{ $total }}</strong> Total
            </div>
        </div>
    </div>
    {{-- done & pending --}}
</div>
```

### Latihan 4: Pagination

Ganti `get()` dengan `paginate(5)` di controller:

```php
$tasks = Task::latest()->paginate(5);
```

Tambah di bawah list di view:

```blade
<div class="mt-3">
    {{ $tasks->links() }}
</div>
```

Refresh halaman — harus ada navigasi halaman di bawah list.

### Latihan 5: Scope di Model

Tambah scope di `Task.php`:

```php
public function scopeDone($query)
{
    return $query->where('is_done', true);
}

public function scopePending($query)
{
    return $query->where('is_done', false);
}
```

Pakai di controller:

```php
Task::pending()->latest()->get();
Task::done()->count();
```

Uji di tinker: `App\Models\Task::pending()->get()`

### Latihan 6 (Bonus): Due Date

Tambah kolom `due_date` (date, nullable) via migration. Tampilkan di list dengan badge merah jika sudah lewat hari ini dan belum selesai.

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| `SQLSTATE[HY000] [1049] Unknown database 'task_manager'` | Database belum dibuat | Buat DB: `CREATE DATABASE task_manager;` |
| `SQLSTATE[HY000] [1045] Access denied` | Kredensial `.env` salah | Cek `DB_USERNAME`, `DB_PASSWORD` |
| `could not find driver` | Extension `pdo_mysql` tidak aktif | Install/aktifkan `php8.x-mysql` |
| `Base table or view not found: tasks` | Migration belum dijalankan | `php artisan migrate` |
| `Add [title] to fillable property` | Kolom tidak ada di `$fillable` | Tambahkan ke array `$fillable` di model |
| Halaman `/tasks` kosong | Seeder belum dijalankan | `php artisan db:seed --class=TaskSeeder` |
| `Class "App\Models\Task" not found` | Autoload belum refresh | `composer dump-autoload` |
| Migration error "table already exists" | Tabel sudahout ada | `php artisan migrate:status` lalu rollback atau fresh |
| Tampilan plain tanpa style | Bootstrap CDN tidak load | Pastikan `@extends('layouts.app')` benar |

### Reset database (development only)

Jika ingin mulai dari nol:

```bash
php artisan migrate:fresh --seed
```

**Peringatan:** Perintah ini **menghapus semua data** di semua tabel, lalu migrate + seed ulang.

---

## Checklist Selesai

Centang sebelum lanjut ke Modul 06:

- [ ] Migration `create_tasks_table` ada dan sudah dijalankan
- [ ] Tabel `tasks` terlihat di MySQL (`DESCRIBE tasks`)
- [ ] Model `Task` punya `$fillable` dan `$casts`
- [ ] `TaskSeeder` mengisi minimal 5 task
- [ ] `/tasks` menampilkan data dari database (bukan hardcode)
- [ ] Badge status Selesai/Belum tampil benar
- [ ] Link "Tasks" ada di navbar
- [ ] Bisa query dasar di `php artisan tinker`
- [ ] `php artisan route:list` menampilkan route `tasks.index`

---

## Git — Commit & Push

Setelah semua berfungsi, simpan progress ke GitHub:

```bash
git add .
git commit -m "feat: tambah migration tasks, seeder, dan halaman list task"
git push
```

Tidak perlu buat branch atau Pull Request. Push langsung ke `main`.

---

## Ringkasan Modul 05

| Topik | Yang dipelajari |
|---|---|
| Migration | Blueprint tabel, `up()` / `down()`, artisan migrate |
| Model | Eloquent, `$fillable`, `$casts` |
| Seeder | Data dummy, `DatabaseSeeder`, `db:seed` |
| Controller | Query Eloquent, kirim data ke view |
| View | Loop `@foreach`, kondisi `@if`, Bootstrap list-group & badge |
| Tinker | Eksperimen query interaktif |

---

**Modul sebelumnya:** [04 — Blade Template](../04-blade-template/README.md)  
**Modul berikutnya:** [06 — CRUD Task](../06-crud-task/README.md)
