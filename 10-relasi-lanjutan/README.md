# Modul 10 — Relasi Lanjutan

Modul kesepuluh: memperdalam Eloquent di **Task Manager** dengan **SoftDeletes**, relasi **many-to-many Tags**, CRUD tag sederhana, attach/sync di form task, serta **eager loading** & `withCount` untuk menghindari masalah N+1.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 09 — Upload & Storage](../09-upload-storage/README.md)

> Project yang sama: `task-manager`.  
> **MySQL only.** Bootstrap 5 CDN only. Auth manual Modul 07 tetap dipakai.

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Menjelaskan SoftDeletes vs hard delete
- [ ] Menambah `deleted_at` ke tasks dan memakai `withTrashed`, `restore`, `forceDelete`
- [ ] Membuat tabel `tags` dan pivot `task_tag`
- [ ] Mendefinisikan relasi `belongsToMany` di Task & Tag
- [ ] Membuat CRUD tag sederhana
- [ ] Attach/sync tag ke task lewat checkbox Bootstrap
- [ ] Menjelaskan masalah N+1 dengan contoh konkret
- [ ] Memakai `with(['category', 'tags'])` dan `withCount`
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
SoftDeletes pada model Task (+ migration deleted_at)
Tabel tags + model Tag + TagController (CRUD sederhana)
Tabel pivot task_tag
Relasi Task ↔ Tag (belongsToMany)
Checkbox tags di form create/edit task
Eager loading & withCount di index/dashboard
(Opsional UI) halaman trash + restore
```

---

## Bagian A — SoftDeletes

### Mengapa Soft Delete?

Hard delete (`$task->delete()` tanpa SoftDeletes) menghapus baris dari database **selamanya**. Soft delete hanya mengisi kolom `deleted_at`:

| Jenis | Efek di DB | Query default |
|---|---|---|
| Hard delete | Row hilang | Tidak bisa ditemukan |
| Soft delete | `deleted_at` terisi timestamp | Otomatis disembunyikan |

Manfaat:

- User bisa "Undo" / restore
- Audit: data tidak langsung hilang
- Aman dari salah klik hapus

### Langkah A1: Migration `deleted_at`

```bash
php artisan make:migration add_soft_deletes_to_tasks_table --table=tasks
```

Edit migration:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->softDeletes(); // menambahkan deleted_at nullable timestamp
        });
    }

    public function down(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->dropSoftDeletes();
        });
    }
};
```

Jalankan:

```bash
php artisan migrate
```

### Langkah A2: Trait SoftDeletes di Model

Edit `app/Models/Task.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Task extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'title',
        'description',
        'is_done',
        'user_id',
        'category_id',
        'attachment',
    ];

    protected $casts = [
        'is_done' => 'boolean',
        'deleted_at' => 'datetime',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    // tags() akan ditambahkan di Bagian B
}
```

Setelah trait aktif:

```php
$task->delete();        // soft delete → isi deleted_at
$task->restore();       // kembalikan (deleted_at = null)
$task->forceDelete();   // hard delete permanen
```

Query default **otomatis** menambah `WHERE deleted_at IS NULL`.

### Langkah A3: Query Soft Delete

```php
// Hanya yang belum dihapus (default)
Task::where('user_id', auth()->id())->get();

// Termasuk yang soft-deleted
Task::withTrashed()->where('user_id', auth()->id())->get();

// Hanya yang soft-deleted
Task::onlyTrashed()->where('user_id', auth()->id())->get();

// Cari satu termasuk trash
$task = Task::withTrashed()->findOrFail($id);
```

### Langkah A4: Update `destroy` (tetap soft delete)

Method `destroy` yang sudah ada akan otomatis soft-delete setelah trait dipasang:

```php
public function destroy(Task $task)
{
    $this->authorizeTask($task);

    // File lampiran: putuskan kebijakanmu
    // Opsi 1 (disarankan untuk soft delete): JANGAN hapus file dulu,
    //         supaya restore masih punya lampiran.
    // Opsi 2: hapus file hanya di forceDelete.

    $task->delete(); // soft delete

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task dipindahkan ke trash.');
}
```

**Rekomendasi modul ini:** saat soft delete, **jangan** hapus file attachment. Hapus file hanya saat `forceDelete`.

### Langkah A5: Route & Method Restore / Force Delete / Trash

Tambahkan di `routes/web.php` **di dalam** group `auth` (sebaiknya **sebelum** `Route::resource` agar tidak bentrok dengan `{task}`):

```php
use App\Http\Controllers\TaskController;

Route::middleware('auth')->group(function () {
    // ... dashboard, categories, dll.

    Route::get('/tasks/trash', [TaskController::class, 'trash'])->name('tasks.trash');
    Route::post('/tasks/{id}/restore', [TaskController::class, 'restore'])->name('tasks.restore');
    Route::delete('/tasks/{id}/force', [TaskController::class, 'forceDelete'])->name('tasks.force-delete');

    Route::resource('tasks', TaskController::class);
});
```

> Pakai `{id}` (bukan route model binding default) karena task soft-deleted tidak ketemu dengan binding biasa.

Tambahkan method di `TaskController`:

```php
public function trash()
{
    $tasks = Task::onlyTrashed()
        ->where('user_id', auth()->id())
        ->with(['category', 'tags'])
        ->latest('deleted_at')
        ->paginate(10);

    return view('tasks.trash', compact('tasks'));
}

public function restore(string $id)
{
    $task = Task::onlyTrashed()
        ->where('user_id', auth()->id())
        ->findOrFail($id);

    $task->restore();

    return redirect()
        ->route('tasks.trash')
        ->with('success', 'Task berhasil direstore!');
}

public function forceDelete(string $id)
{
    $task = Task::onlyTrashed()
        ->where('user_id', auth()->id())
        ->findOrFail($id);

    if ($task->attachment) {
        \Illuminate\Support\Facades\Storage::disk('public')->delete($task->attachment);
    }

    // Detach tags sebelum hapus permanen (opsional, cascade pivot juga bisa)
    $task->tags()->detach();

    $task->forceDelete();

    return redirect()
        ->route('tasks.trash')
        ->with('success', 'Task dihapus permanen.');
}
```

### Langkah A6: View Trash (Bootstrap CDN)

Buat `resources/views/tasks/trash.blade.php`:

```blade
@extends('layouts.app')

@section('title', 'Trash Task')

@section('content')
<div class="d-flex justify-content-between align-items-center mb-4">
    <h1 class="h3 mb-0">Trash</h1>
    <a href="{{ route('tasks.index') }}" class="btn btn-outline-secondary">Kembali ke Tasks</a>
</div>

@if ($tasks->isEmpty())
    <div class="alert alert-info">Trash kosong.</div>
@else
    <div class="table-responsive">
        <table class="table table-hover align-middle">
            <thead>
                <tr>
                    <th>Judul</th>
                    <th>Dihapus</th>
                    <th class="text-end">Aksi</th>
                </tr>
            </thead>
            <tbody>
                @foreach ($tasks as $task)
                    <tr>
                        <td>{{ $task->title }}</td>
                        <td>{{ $task->deleted_at?->diffForHumans() }}</td>
                        <td class="text-end">
                            <form action="{{ route('tasks.restore', $task->id) }}" method="POST" class="d-inline">
                                @csrf
                                <button class="btn btn-sm btn-success">Restore</button>
                            </form>
                            <form action="{{ route('tasks.force-delete', $task->id) }}" method="POST" class="d-inline"
                                  onsubmit="return confirm('Hapus permanen? Tidak bisa dibatalkan.')">
                                @csrf
                                @method('DELETE')
                                <button class="btn btn-sm btn-danger">Hapus Permanen</button>
                            </form>
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    </div>

    {{ $tasks->links() }}
@endif
@endsection
```

Tambahkan link di navbar (hanya `@auth`):

```blade
<li class="nav-item">
    <a class="nav-link" href="{{ route('tasks.trash') }}">Trash</a>
</li>
```

---

## Bagian B — Tags Many-to-Many

### Konsep Many-to-Many

Satu task bisa punya banyak tag (`urgent`, `backend`, `bug`).  
Satu tag bisa dipakai banyak task.

```
tasks          task_tag (pivot)         tags
------         ----------------         ----
id    <------> task_id                  id
title          tag_id  <--------------> name
...            timestamps?              ...
```

Ini berbeda dari:

| Relasi | Contoh di project |
|---|---|
| `belongsTo` / `hasMany` | Task belongsTo Category; User hasMany Task |
| `belongsToMany` | Task belongsToMany Tag |

### Langkah B1: Model & Migration Tags

```bash
php artisan make:model Tag -m
php artisan make:controller TagController --resource
php artisan make:migration create_task_tag_table
```

Edit migration `create_tags_table`:

```php
public function up(): void
{
    Schema::create('tags', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('slug')->unique();
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->timestamps();

        $table->unique(['user_id', 'name']);
    });
}

public function down(): void
{
    Schema::dropIfExists('tags');
}
```

**Kenapa `user_id`?**  
Tag bersifat per-user agar User A tidak mengotak-atik tag User B (konsisten dengan kategori di Modul 08).

### Langkah B2: Migration Pivot `task_tag`

Konvensi nama pivot: alphabetical singular — `tag` + `task` → `task_tag`.

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('task_tag', function (Blueprint $table) {
            $table->id();
            $table->foreignId('task_id')->constrained()->cascadeOnDelete();
            $table->foreignId('tag_id')->constrained()->cascadeOnDelete();
            $table->timestamps();

            $table->unique(['task_id', 'tag_id']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('task_tag');
    }
};
```

Jalankan:

```bash
php artisan migrate
```

### Langkah B3: Model Tag

`app/Models/Tag.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Support\Str;

class Tag extends Model
{
    protected $fillable = [
        'name',
        'slug',
        'user_id',
    ];

    protected static function booted(): void
    {
        static::creating(function (Tag $tag) {
            if (empty($tag->slug)) {
                $tag->slug = Str::slug($tag->name);
            }
        });
    }

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function tasks(): BelongsToMany
    {
        return $this->belongsToMany(Task::class)->withTimestamps();
    }
}
```

### Langkah B4: Relasi di Task & User

Di `Task`:

```php
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class)->withTimestamps();
}
```

Di `User` (opsional tapi berguna):

```php
public function tags()
{
    return $this->hasMany(Tag::class);
}
```

### Attach, Detach, Sync — Apa Bedanya?

```php
$task->tags()->attach([1, 2]);      // tambah relasi (bisa duplikat jika dipanggil ulang tanpa unique)
$task->tags()->detach([2]);         // hapus relasi tag 2
$task->tags()->detach();            // hapus semua tag task ini
$task->tags()->sync([1, 3, 5]);     // jadikan tepat {1,3,5} — tambah yang kurang, hapus yang lebih
$task->tags()->syncWithoutDetaching([4]); // tambah tanpa menghapus yang lama
```

Untuk form edit dengan checkbox, **`sync()`** adalah pilihan terbaik.

---

## Bagian C — CRUD Tags Sederhana

### Routes

```php
use App\Http\Controllers\TagController;

Route::middleware('auth')->group(function () {
    Route::resource('tags', TagController::class)->except(['show']);
});
```

### TagController (inti)

```php
<?php

namespace App\Http\Controllers;

use App\Models\Tag;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

class TagController extends Controller
{
    public function index()
    {
        $tags = auth()->user()
            ->tags()
            ->withCount('tasks')
            ->orderBy('name')
            ->paginate(15);

        return view('tags.index', compact('tags'));
    }

    public function create()
    {
        return view('tags.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => ['required', 'string', 'max:50'],
        ]);

        $validated['slug'] = Str::slug($validated['name']);
        $validated['user_id'] = auth()->id();

        // unik per user
        $exists = auth()->user()->tags()
            ->where('name', $validated['name'])
            ->exists();

        if ($exists) {
            return back()
                ->withInput()
                ->withErrors(['name' => 'Tag dengan nama ini sudah ada.']);
        }

        Tag::create($validated);

        return redirect()
            ->route('tags.index')
            ->with('success', 'Tag berhasil dibuat!');
    }

    public function edit(Tag $tag)
    {
        abort_unless($tag->user_id === auth()->id(), 403);

        return view('tags.edit', compact('tag'));
    }

    public function update(Request $request, Tag $tag)
    {
        abort_unless($tag->user_id === auth()->id(), 403);

        $validated = $request->validate([
            'name' => ['required', 'string', 'max:50'],
        ]);

        $validated['slug'] = Str::slug($validated['name']);

        $tag->update($validated);

        return redirect()
            ->route('tags.index')
            ->with('success', 'Tag berhasil diupdate!');
    }

    public function destroy(Tag $tag)
    {
        abort_unless($tag->user_id === auth()->id(), 403);

        $tag->tasks()->detach();
        $tag->delete();

        return redirect()
            ->route('tags.index')
            ->with('success', 'Tag dihapus.');
    }
}
```

### View `tags/index.blade.php` (ringkas)

```blade
@extends('layouts.app')

@section('title', 'Tags')

@section('content')
<div class="d-flex justify-content-between align-items-center mb-4">
    <h1 class="h3 mb-0">Tags</h1>
    <a href="{{ route('tags.create') }}" class="btn btn-primary">Tambah Tag</a>
</div>

<table class="table table-hover">
    <thead>
        <tr>
            <th>Nama</th>
            <th>Slug</th>
            <th>Jumlah Task</th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        @forelse ($tags as $tag)
            <tr>
                <td><span class="badge text-bg-secondary">{{ $tag->name }}</span></td>
                <td><code>{{ $tag->slug }}</code></td>
                <td>{{ $tag->tasks_count }}</td>
                <td class="text-end">
                    <a href="{{ route('tags.edit', $tag) }}" class="btn btn-sm btn-outline-primary">Edit</a>
                    <form action="{{ route('tags.destroy', $tag) }}" method="POST" class="d-inline"
                          onsubmit="return confirm('Hapus tag ini?')">
                        @csrf
                        @method('DELETE')
                        <button class="btn btn-sm btn-outline-danger">Hapus</button>
                    </form>
                </td>
            </tr>
        @empty
            <tr>
                <td colspan="4" class="text-muted">Belum ada tag.</td>
            </tr>
        @endforelse
    </tbody>
</table>

{{ $tags->links() }}
@endsection
```

Form create/edit mirip kategori (Modul 08): satu field `name`, Bootstrap form control, `@csrf`, `@error`.

Tambahkan menu navbar:

```blade
<li class="nav-item">
    <a class="nav-link" href="{{ route('tags.index') }}">Tags</a>
</li>
```

---

## Bagian D — Checkbox Tags di Form Task

### Kirim daftar tag ke form

Di `create()` dan `edit()` TaskController:

```php
public function create()
{
    $categories = auth()->user()->categories()->orderBy('name')->get();
    $tags = auth()->user()->tags()->orderBy('name')->get();

    return view('tasks.create', compact('categories', 'tags'));
}

public function edit(Task $task)
{
    $this->authorizeTask($task);

    $categories = auth()->user()->categories()->orderBy('name')->get();
    $tags = auth()->user()->tags()->orderBy('name')->get();

    return view('tasks.edit', compact('task', 'categories', 'tags'));
}
```

### Blade — checkbox Bootstrap

Di `create.blade.php` dan `edit.blade.php`:

```blade
<div class="mb-3">
    <label class="form-label d-block">Tags</label>

    @forelse ($tags as $tag)
        <div class="form-check form-check-inline">
            <input
                class="form-check-input"
                type="checkbox"
                name="tags[]"
                id="tag-{{ $tag->id }}"
                value="{{ $tag->id }}"
                @checked(collect(old('tags', isset($task) ? $task->tags->pluck('id')->all() : []))->contains($tag->id))
            >
            <label class="form-check-label" for="tag-{{ $tag->id }}">
                {{ $tag->name }}
            </label>
        </div>
    @empty
        <p class="text-muted mb-0">
            Belum ada tag.
            <a href="{{ route('tags.create') }}">Buat tag dulu</a>.
        </p>
    @endforelse

    @error('tags')
        <div class="text-danger small">{{ $message }}</div>
    @enderror
    @error('tags.*')
        <div class="text-danger small">{{ $message }}</div>
    @enderror
</div>
```

### Validasi + sync di `store` / `update`

```php
$validated = $request->validate([
    'title'       => ['required', 'string', 'max:255'],
    'description' => ['nullable', 'string'],
    'is_done'     => ['sometimes', 'boolean'],
    'category_id' => ['nullable', 'exists:categories,id'],
    'attachment'  => ['nullable', 'file', 'mimes:jpg,jpeg,png,gif,webp,pdf', 'max:2048'],
    'tags'        => ['nullable', 'array'],
    'tags.*'      => ['integer', 'exists:tags,id'],
]);

// ... simpan task seperti biasa (termasuk attachment Modul 09) ...

$task = Task::create($validated); // store
// atau: $task->update($validated); // update

$tagIds = collect($request->input('tags', []))
    ->map(fn ($id) => (int) $id)
    ->all();

// Pastikan semua tag milik user login
$ownedTagIds = auth()->user()->tags()
    ->whereIn('id', $tagIds)
    ->pluck('id')
    ->all();

$task->tags()->sync($ownedTagIds);
```

Di **update**, panggil `sync` setelah `$task->update(...)`.

Di **show** dan **index**, tampilkan badge tag:

```blade
@foreach ($task->tags as $tag)
    <span class="badge text-bg-light border">{{ $tag->name }}</span>
@endforeach
```

---

## Bagian E — Eager Loading & N+1

### Apa itu N+1?

Misalnya index menampilkan 10 task, tiap task menampilkan nama kategori + tags:

```php
// ❌ LAZY LOADING — berbahaya
$tasks = Task::where('user_id', auth()->id())->get();

foreach ($tasks as $task) {
    echo $task->category->name;   // query ekstra per task
    foreach ($task->tags as $tag) {
        echo $tag->name;          // query ekstra lagi
    }
}
```

Query yang terjadi kira-kira:

```
1 query  → ambil 10 tasks
10 query → category per task
10 query → tags per task
= 21 query  (atau lebih)
```

Ini disebut **N+1**: 1 query utama + N query per relasi.

### Solusi: Eager Loading `with()`

```php
$tasks = Task::where('user_id', auth()->id())
    ->with(['category', 'tags'])
    ->latest()
    ->paginate(10);
```

Query jadi kira-kira:

```
1 query → tasks
1 query → categories WHERE id IN (...)
1 query → tags via pivot WHERE task_id IN (...)
= 3 query
```

### `withCount`

Sering butuh jumlah tanpa load semua relasi:

```php
$tags = auth()->user()
    ->tags()
    ->withCount('tasks')
    ->get();

// akses: $tag->tasks_count
```

Di dashboard:

```php
$stats = [
    'total'      => auth()->user()->tasks()->count(),
    'done'       => auth()->user()->tasks()->where('is_done', true)->count(),
    'pending'    => auth()->user()->tasks()->where('is_done', false)->count(),
    'trashed'    => auth()->user()->tasks()->onlyTrashed()->count(),
];

$latestTasks = auth()->user()
    ->tasks()
    ->with(['category', 'tags'])
    ->withCount('tags')
    ->latest()
    ->take(5)
    ->get();
```

### Update `index` TaskController

```php
public function index(Request $request)
{
    $query = auth()->user()
        ->tasks()
        ->with(['category', 'tags'])
        ->withCount('tags');

    // search & filter dari Modul 08 tetap bisa di sini...

    $tasks = $query->latest()->paginate(10)->withQueryString();

    $categories = auth()->user()->categories()->orderBy('name')->get();
    $tags = auth()->user()->tags()->orderBy('name')->get();

    return view('tasks.index', compact('tasks', 'categories', 'tags'));
}
```

### Debug N+1 (opsional)

Install di local (bukan wajib modul): Laravel Debugbar, atau log query:

```php
\DB::listen(function ($query) {
    logger($query->sql);
});
```

Atau di Tinker bandingkan jumlah query sebelum/sesudah `with()`.

---

## Filter Task by Tag (opsional tapi bagus)

```php
if ($request->filled('tag')) {
    $tagId = (int) $request->tag;
    $query->whereHas('tags', function ($q) use ($tagId) {
        $q->where('tags.id', $tagId);
    });
}
```

Di view filter:

```blade
<select name="tag" class="form-select">
    <option value="">Semua Tag</option>
    @foreach ($tags as $tag)
        <option value="{{ $tag->id }}" @selected(request('tag') == $tag->id)>
            {{ $tag->name }}
        </option>
    @endforeach
</select>
```

---

## Seeder Tag (opsional)

```bash
php artisan make:seeder TagSeeder
```

```php
public function run(): void
{
    $user = \App\Models\User::first();
    if (!$user) {
        return;
    }

    foreach (['Urgent', 'Backend', 'Bug', 'Ide'] as $name) {
        $user->tags()->firstOrCreate(
            ['name' => $name],
            ['slug' => \Illuminate\Support\Str::slug($name)]
        );
    }
}
```

Panggil dari `DatabaseSeeder` jika diinginkan.

---

## Latihan

### Latihan 1: Badge Warna Tag

Tambah kolom `color` di tags (mirip categories) dan tampilkan `badge bg-{{ $tag->color }}`.

### Latihan 2: Restore Massal

Di halaman trash, tambah tombol "Restore Semua" yang me-restore semua soft-deleted task milik user.

### Latihan 3: Cegah Force Delete jika masih ada lampiran (kebijakan lain)

Atau sebaliknya: tampilkan peringatan "lampiran akan ikut terhapus".

### Latihan 4: `whereHas` + Search Gabungan

Filter: search judul **dan** tag sekaligus. Pastikan pagination `withQueryString()`.

### Latihan 5: Hitung Tag Terpopuler

Di dashboard, tampilkan 5 tag dengan `tasks_count` tertinggi milik user (`withCount` + `orderByDesc`).

### Latihan 6: Pivot Extra Column

Tambah kolom `tagged_at` atau `note` di `task_tag`, lalu akses:

```php
$task->tags()->withPivot('note')->get();
foreach ($task->tags as $tag) {
    echo $tag->pivot->note;
}
```

### Latihan 7 (Bonus): Global Scope vs SoftDeletes

Pelajari bagaimana SoftDeletes mendaftarkan global scope, dan coba `withoutGlobalScopes()` (hati-hati).

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| Task hilang setelah delete tapi masih di DB | SoftDeletes aktif | Cek `deleted_at`; pakai `withTrashed` / halaman trash |
| Restore 404 | Route model binding default | Pakai `onlyTrashed()->findOrFail($id)` |
| `Table task_tag doesn't exist` | Migration pivot belum jalan | `php artisan migrate` |
| `SQLSTATE duplicate entry` pivot | Attach dua kali tanpa unique | Pakai `sync` + unique index |
| Tag user lain ikut ke-sync | Tidak filter ownership | Filter `auth()->user()->tags()->whereIn(...)` |
| N+1 tetap terjadi | Lupa `with()` | Tambah `with(['category','tags'])` |
| `tags_count` null/error | Belum `withCount('tags')` | Tambahkan di query |
| Checkbox tidak tercentang saat edit | Salah `old()` / pluck | Pakai `old('tags', $task->tags->pluck('id'))` |
| Detach tidak jalan saat hapus tag | Lupa detach | `$tag->tasks()->detach()` sebelum delete |
| Cascade menghapus task saat hapus tag | FK salah arah | Pastikan pivot `tag_id` cascade hanya hapus **baris pivot**, bukan task |

---

## Checklist Selesai

- [ ] Kolom `deleted_at` ada di `tasks` (MySQL)
- [ ] Model Task memakai trait `SoftDeletes`
- [ ] Halaman trash + restore + forceDelete berfungsi
- [ ] Force delete menghapus attachment & detach tags
- [ ] Tabel `tags` dan pivot `task_tag` ada
- [ ] Relasi `belongsToMany` di Task & Tag
- [ ] CRUD tags sederhana jalan (index/create/edit/delete)
- [ ] Checkbox tags di form task + `sync()` aman (hanya tag milik user)
- [ ] Index/show menampilkan badge tags
- [ ] Eager loading `with(['category','tags'])` dipakai
- [ ] `withCount` dipakai di tags index dan/atau tasks
- [ ] Memahami N+1 dan bisa menjelaskannya
- [ ] Tidak pakai SQLite / npm / Vite / Tailwind / Laravel UI
- [ ] Commit & push

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: tag many-to-many dan soft delete"
git push
```

Tanpa branch, tanpa PR, tanpa merge.

---

## Ringkasan Modul 10

| Topik | Yang dipelajari |
|---|---|
| SoftDeletes | `delete`, `restore`, `forceDelete`, `withTrashed`, `onlyTrashed` |
| Many-to-many | Tabel pivot, `belongsToMany`, `attach` / `detach` / `sync` |
| CRUD Tag | Resource sederhana per user |
| Form | Checkbox array `tags[]` + Bootstrap |
| Performa | N+1 problem, `with()`, `withCount`, `whereHas` |

---

**Modul sebelumnya:** [09 — Upload & Storage](../09-upload-storage/README.md)  
**Modul berikutnya:** [11 — Middleware & Authorization](../11-middleware-authorization/README.md)
