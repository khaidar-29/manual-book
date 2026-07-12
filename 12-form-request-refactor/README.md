# Modul 12 — Form Request & Refactor

Modul kedua belas: merapikan **Task Manager** dengan **Form Request** dan **Service class**. Controller menjadi tipis (thin), validasi terpisah, dan logika bisnis (termasuk upload lampiran dari Modul 09) pindah ke service.

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 11 — Middleware & Authorization](../11-middleware-authorization/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Menjelaskan mengapa fat controller bermasalah
- [ ] Membuat `StoreTaskRequest` dan `UpdateTaskRequest`
- [ ] Memakai `rules()`, `authorize()`, dan `messages()` di Form Request
- [ ] Membuat `App\Services\TaskService` (create / update / delete)
- [ ] Memindahkan handle attachment ke service (jika ada dari Modul 09)
- [ ] Merapikan `TaskController` menjadi thin controller
- [ ] (Opsional) Membuat `CategoryService` singkat
- [ ] Commit & push ke GitHub

**Yang ditambahkan / diubah di project:**

```
app/Http/Requests/StoreTaskRequest.php
app/Http/Requests/UpdateTaskRequest.php
app/Services/TaskService.php
app/Services/CategoryService.php          ← opsional
app/Http/Controllers/TaskController.php   ← di-refactor (tipis)
```

> **Stack tetap sama:** MySQL only, Bootstrap 5 CDN only. Tidak ada npm/Vite/Tailwind/Laravel UI.

---

## Mengapa Fat Controller Buruk?

**Fat controller** = controller yang berisi terlalu banyak tanggung jawab: validasi, query, upload file, redirect, flash message, authorization, dan logika bisnis sekaligus.

### Masalah fat controller

| Masalah | Dampak |
|---|---|
| Sulit dibaca | Method `store()` / `update()` jadi puluhan baris |
| Sulit diuji | Logika bisnis tercampur dengan HTTP (request/response) |
| Duplikasi | Create & update sering copy-paste aturan yang sama |
| Sulit diubah | Ubah aturan validasi → cari di banyak method |
| Melanggar SRP | Satu class punya terlalu banyak alasan untuk berubah |

### Prinsip yang kita pakai

```
HTTP layer (Controller)  → terima request, panggil service, return response
Validasi (Form Request)  → rules, authorize, messages
Business logic (Service) → create/update/delete, upload, sync tag, hapus file
Model (Eloquent)         → relasi, scope, cast
```

Controller **tipis** hanya “mengorkestrasi”. Detail kerja ada di Form Request + Service.

---

## Konsep: Form Request

**Form Request** = class khusus untuk validasi + otorisasi satu form/aksi.

```bash
php artisan make:request StoreTaskRequest
php artisan make:request UpdateTaskRequest
```

Laravel otomatis menjalankan Form Request **sebelum** method controller. Jika validasi gagal:

- Request web → redirect back + error bag + `old()` input
- Request API (Modul 13) → JSON 422 (otomatis jika `Accept: application/json`)

### Method penting

| Method | Fungsi |
|---|---|
| `authorize()` | `true` jika user boleh submit form ini |
| `rules()` | Array aturan validasi |
| `messages()` | Pesan error custom (opsional) |
| `attributes()` | Nama field ramah user (opsional) |
| `prepareForValidation()` | Normalisasi input sebelum validasi |
| `validated()` | Ambil hanya data yang lolos validasi |

Type-hint di controller:

```php
public function store(StoreTaskRequest $request)
{
    $validated = $request->validated();
    // ...
}
```

Tidak perlu `$request->validate([...])` lagi di dalam method.

---

## Konsep: Service Class

**Service** = class biasa di `app/Services/` yang menampung logika bisnis.

Bukan bawaan Artisan (tidak ada `make:service` di Laravel core). Kita buat manual:

```
app/Services/TaskService.php
```

Keuntungan:

- Controller tipis dan fokus HTTP
- Logika bisa dipanggil dari web controller **dan** API controller (Modul 13)
- Lebih mudah di-unit-test tanpa HTTP
- Upload/hapus lampiran terpusat di satu tempat

---

## Langkah 1: Buat Form Request

```bash
cd task-manager
php artisan make:request StoreTaskRequest
php artisan make:request UpdateTaskRequest
```

### `app/Http/Requests/StoreTaskRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class StoreTaskRequest extends FormRequest
{
    public function authorize(): bool
    {
        // Hanya user login yang boleh membuat task
        return auth()->check();
    }

    protected function prepareForValidation(): void
    {
        if ($this->category_id === '' || $this->category_id === null) {
            $this->merge(['category_id' => null]);
        }
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'min:3', 'max:255'],
            'description' => ['nullable', 'string', 'max:2000'],
            'is_done' => ['sometimes', 'boolean'],
            'category_id' => [
                'nullable',
                Rule::exists('categories', 'id')
                    ->where(fn ($q) => $q->where('user_id', auth()->id())),
            ],
            // Dari Modul 09 — lampiran (opsional)
            'attachment' => ['nullable', 'file', 'mimes:jpg,jpeg,png,pdf', 'max:2048'],
            // Dari Modul 10 — sync tag (opsional)
            'tag_ids' => ['nullable', 'array'],
            'tag_ids.*' => [
                'integer',
                Rule::exists('tags', 'id'),
            ],
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Judul task wajib diisi.',
            'title.min' => 'Judul minimal 3 karakter.',
            'title.max' => 'Judul maksimal 255 karakter.',
            'attachment.mimes' => 'Lampiran harus berupa JPG, PNG, atau PDF.',
            'attachment.max' => 'Ukuran lampiran maksimal 2 MB.',
            'category_id.exists' => 'Kategori tidak valid atau bukan milik Anda.',
        ];
    }
}
```

### `app/Http/Requests/UpdateTaskRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class UpdateTaskRequest extends FormRequest
{
    public function authorize(): bool
    {
        $task = $this->route('task');

        // Pastikan task milik user yang login
        // (Policy dari Modul 11 juga bisa dipakai di sini)
        return auth()->check()
            && $task
            && $task->user_id === auth()->id();
    }

    protected function prepareForValidation(): void
    {
        if ($this->category_id === '' || $this->category_id === null) {
            $this->merge(['category_id' => null]);
        }
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'min:3', 'max:255'],
            'description' => ['nullable', 'string', 'max:2000'],
            'is_done' => ['sometimes', 'boolean'],
            'category_id' => [
                'nullable',
                Rule::exists('categories', 'id')
                    ->where(fn ($q) => $q->where('user_id', auth()->id())),
            ],
            'attachment' => ['nullable', 'file', 'mimes:jpg,jpeg,png,pdf', 'max:2048'],
            'tag_ids' => ['nullable', 'array'],
            'tag_ids.*' => [
                'integer',
                Rule::exists('tags', 'id'),
            ],
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Judul task wajib diisi.',
            'title.min' => 'Judul minimal 3 karakter.',
            'attachment.mimes' => 'Lampiran harus berupa JPG, PNG, atau PDF.',
            'attachment.max' => 'Ukuran lampiran maksimal 2 MB.',
        ];
    }
}
```

**Catatan:**

- Jika Modul 09 belum menambah kolom `attachment`, hapus rule `attachment` dulu — atau biarkan (file tidak dikirim = lolos `nullable`).
- Jika Modul 10 belum ada tag, hapus `tag_ids`.
- Jika Modul 11 sudah pakai Policy, `authorize()` bisa diganti:

```php
public function authorize(): bool
{
    return $this->user()->can('update', $this->route('task'));
}
```

---

## Langkah 2: Buat TaskService

Buat folder dan file:

```bash
mkdir -p app/Services
```

### `app/Services/TaskService.php`

```php
<?php

namespace App\Services;

use App\Models\Task;
use App\Models\User;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

class TaskService
{
    /**
     * Buat task baru untuk user.
     *
     * @param  array<string, mixed>  $data  Data tervalidasi
     */
    public function create(User $user, array $data, ?UploadedFile $attachment = null): Task
    {
        $payload = $this->preparePayload($data);

        if ($attachment) {
            $payload['attachment'] = $this->storeAttachment($attachment);
        }

        /** @var Task $task */
        $task = $user->tasks()->create($payload);

        if (! empty($data['tag_ids'])) {
            $task->tags()->sync($data['tag_ids']);
        }

        return $task->load(['category', 'tags']);
    }

    /**
     * Update task yang sudah ada.
     *
     * @param  array<string, mixed>  $data
     */
    public function update(Task $task, array $data, ?UploadedFile $attachment = null): Task
    {
        $payload = $this->preparePayload($data);

        if ($attachment) {
            $this->deleteAttachmentIfExists($task);
            $payload['attachment'] = $this->storeAttachment($attachment);
        }

        $task->update($payload);

        if (array_key_exists('tag_ids', $data)) {
            $task->tags()->sync($data['tag_ids'] ?? []);
        }

        return $task->fresh(['category', 'tags']);
    }

    /**
     * Hapus task + file lampiran (jika ada).
     */
    public function delete(Task $task): void
    {
        $this->deleteAttachmentIfExists($task);

        // Soft delete (Modul 10) atau hard delete — sesuaikan model
        $task->delete();
    }

    /**
     * Siapkan field aman untuk mass assignment.
     *
     * @param  array<string, mixed>  $data
     * @return array<string, mixed>
     */
    protected function preparePayload(array $data): array
    {
        return [
            'title' => $data['title'],
            'description' => $data['description'] ?? null,
            'is_done' => (bool) ($data['is_done'] ?? false),
            'category_id' => $data['category_id'] ?? null,
        ];
    }

    protected function storeAttachment(UploadedFile $file): string
    {
        // Simpan di disk public → storage/app/public/attachments
        return $file->store('attachments', 'public');
    }

    protected function deleteAttachmentIfExists(Task $task): void
    {
        if (! empty($task->attachment)) {
            Storage::disk('public')->delete($task->attachment);
        }
    }
}
```

**Pastikan:**

1. Kolom `attachment` ada di `$fillable` model `Task` (Modul 09).
2. `php artisan storage:link` sudah dijalankan.
3. Relasi `tags()` ada jika sync tag dipakai (Modul 10).

Jika belum ada attachment/tag, sederhanakan service:

```php
public function create(User $user, array $data): Task
{
    return $user->tasks()->create([
        'title' => $data['title'],
        'description' => $data['description'] ?? null,
        'is_done' => (bool) ($data['is_done'] ?? false),
        'category_id' => $data['category_id'] ?? null,
    ]);
}
```

---

## Langkah 3: Refactor TaskController (Thin)

### Sebelum (fat — contoh)

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|string|min:3|max:255',
        'description' => 'nullable|string|max:2000',
        'is_done' => 'sometimes|boolean',
        'category_id' => 'nullable|exists:categories,id',
        'attachment' => 'nullable|file|mimes:jpg,jpeg,png,pdf|max:2048',
    ]);

    $validated['is_done'] = $request->boolean('is_done');

    if ($request->hasFile('attachment')) {
        $validated['attachment'] = $request->file('attachment')
            ->store('attachments', 'public');
    }

    $task = auth()->user()->tasks()->create($validated);

    if ($request->filled('tag_ids')) {
        $task->tags()->sync($request->tag_ids);
    }

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task berhasil ditambahkan!');
}
```

Validasi, upload, sync tag, create — semua di controller.

### Sesudah (thin)

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreTaskRequest;
use App\Http\Requests\UpdateTaskRequest;
use App\Models\Task;
use App\Services\TaskService;
use Illuminate\Http\Request;
use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class TaskController extends Controller implements HasMiddleware
{
    public function __construct(
        protected TaskService $tasks
    ) {}

    public static function middleware(): array
    {
        return [
            new Middleware('auth'),
        ];
    }

    public function index(Request $request)
    {
        $user = auth()->user();

        $query = $user->tasks()->with(['category', 'tags'])->latest();

        if ($request->filled('search')) {
            $query->where('title', 'like', '%'.$request->search.'%');
        }

        if ($request->filled('category')) {
            $query->where('category_id', $request->category);
        }

        if ($request->filled('status')) {
            if ($request->status === 'done') {
                $query->done();
            } elseif ($request->status === 'pending') {
                $query->pending();
            }
        }

        $tasks = $query->paginate(10)->withQueryString();
        $categories = $user->categories()->orderBy('name')->get();

        return view('tasks.index', compact('tasks', 'categories'));
    }

    public function create()
    {
        $categories = auth()->user()->categories()->orderBy('name')->get();
        $tags = \App\Models\Tag::orderBy('name')->get(); // jika Modul 10

        return view('tasks.create', compact('categories', 'tags'));
    }

    public function store(StoreTaskRequest $request)
    {
        $this->tasks->create(
            auth()->user(),
            $request->validated(),
            $request->file('attachment')
        );

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil ditambahkan!');
    }

    public function show(Task $task)
    {
        $this->authorize('view', $task); // Policy Modul 11
        $task->load(['category', 'tags']);

        return view('tasks.show', compact('task'));
    }

    public function edit(Task $task)
    {
        $this->authorize('update', $task);

        $categories = auth()->user()->categories()->orderBy('name')->get();
        $tags = \App\Models\Tag::orderBy('name')->get();

        return view('tasks.edit', compact('task', 'categories', 'tags'));
    }

    public function update(UpdateTaskRequest $request, Task $task)
    {
        // authorize sudah di Form Request; Policy tetap boleh dipanggil
        $this->authorize('update', $task);

        $this->tasks->update(
            $task,
            $request->validated(),
            $request->file('attachment')
        );

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil diupdate!');
    }

    public function destroy(Task $task)
    {
        $this->authorize('delete', $task);

        $this->tasks->delete($task);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task berhasil dihapus!');
    }
}
```

**Constructor injection:** Laravel otomatis resolve `TaskService` tanpa binding manual di container (class konkret).

---

## Langkah 4 (Opsional): CategoryService

Pola sama untuk kategori — ringkas:

```php
<?php

namespace App\Services;

use App\Models\Category;
use App\Models\User;

class CategoryService
{
    public function create(User $user, array $data): Category
    {
        return $user->categories()->create([
            'name' => $data['name'],
            'color' => $data['color'] ?? 'primary',
        ]);
    }

    public function update(Category $category, array $data): Category
    {
        $category->update([
            'name' => $data['name'],
            'color' => $data['color'] ?? $category->color,
        ]);

        return $category->fresh();
    }

    public function delete(Category $category): void
    {
        // Task yang pakai kategori: category_id jadi null (nullOnDelete)
        $category->delete();
    }
}
```

Di `CategoryController`:

```php
public function __construct(
    protected CategoryService $categories
) {}

public function store(StoreCategoryRequest $request)
{
    $this->categories->create(auth()->user(), $request->validated());

    return redirect()
        ->route('categories.index')
        ->with('success', 'Kategori berhasil ditambahkan!');
}
```

Buat juga `StoreCategoryRequest` / `UpdateCategoryRequest` jika belum ada.

---

## Perbandingan Before / After

### Validasi

| Sebelum | Sesudah |
|---|---|
| `$request->validate([...])` di controller | `StoreTaskRequest` / `UpdateTaskRequest` |
| Rules tersebar di `store` & `update` | Satu sumber kebenaran per aksi |
| Sulit reuse di API | Form Request bisa dipakai API juga |

### Create / Update / Delete

| Sebelum | Sesudah |
|---|---|
| Upload file di controller | `TaskService::storeAttachment()` |
| Sync tag di controller | `TaskService` memanggil `tags()->sync()` |
| Hapus file mudah terlupa | `delete()` selalu bersihkan attachment |
| Method 30–50 baris | Method 5–10 baris |

### Struktur folder setelah modul

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── TaskController.php      ← tipis
│   │   └── CategoryController.php
│   └── Requests/
│       ├── StoreTaskRequest.php
│       ├── UpdateTaskRequest.php
│       ├── StoreCategoryRequest.php   ← opsional
│       └── UpdateCategoryRequest.php  ← opsional
├── Services/
│   ├── TaskService.php
│   └── CategoryService.php            ← opsional
└── Models/
    ├── Task.php
    ├── Category.php
    └── ...
```

---

## Alur Request Setelah Refactor

```
1. User submit form create
   → POST /tasks + @csrf + file attachment

2. Laravel resolve StoreTaskRequest
   → authorize() → rules() → jika gagal: redirect + errors

3. TaskController@store
   → $this->tasks->create(user, validated, file)

4. TaskService
   → preparePayload → storeAttachment → create → sync tags

5. Redirect /tasks + flash success
```

---

## Uji Manual

Jalankan server:

```bash
php artisan serve
```

| # | Aksi | Hasil yang diharapkan |
|---|---|---|
| 1 | Submit create kosong | Error validasi (judul wajib) |
| 2 | Submit judul 2 karakter | Error minimal 3 karakter |
| 3 | Create valid tanpa file | Task tersimpan, flash sukses |
| 4 | Create dengan PDF/JPG valid | File di `storage/app/public/attachments` |
| 5 | Create file > 2 MB | Error ukuran lampiran |
| 6 | Edit task milik sendiri | Form terisi, update sukses |
| 7 | Edit task user lain (ID) | 403 (authorize Form Request / Policy) |
| 8 | Hapus task ber-attachment | Record hilang + file terhapus dari disk |
| 9 | Cek `TaskController` | Method store/update/destroy pendek |

---

## Latihan Modul 12

### Latihan 1: Custom Attribute Names

Tambah di Form Request:

```php
public function attributes(): array
{
    return [
        'title' => 'judul',
        'description' => 'deskripsi',
        'attachment' => 'lampiran',
    ];
}
```

Pastikan pesan error memakai nama Indonesia.

### Latihan 2: Form Request Kategori

Buat `StoreCategoryRequest` & `UpdateCategoryRequest`. Pindahkan validasi dari `CategoryController`. Rules minimal: `name` required unique per user, `color` in daftar warna Bootstrap.

### Latihan 3: Hapus Attachment Saja

Tambah method di `TaskService`:

```php
public function removeAttachment(Task $task): Task
{
    $this->deleteAttachmentIfExists($task);
    $task->update(['attachment' => null]);

    return $task->fresh();
}
```

Buat route `DELETE /tasks/{task}/attachment` + tombol di form edit.

### Latihan 4: Shared Rules Trait

Buat trait `app/Http/Requests/Concerns/TaskRules.php` berisi method `taskRules(): array`. Pakai di Store & Update agar tidak duplikasi.

### Latihan 5: Toggle Status via Service

Pindahkan logic toggle `is_done` ke `TaskService::toggle(Task $task): Task`. Controller hanya memanggil service + `back()`.

### Latihan 6 (Bonus): Interface + Binding

Buat interface `TaskServiceInterface`, implementasikan di `TaskService`, bind di `AppServiceProvider`. Latihan untuk Dependency Inversion (berguna saat testing dengan mock).

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| `Target class [TaskService] does not exist` | Namespace / autoload | Cek `namespace App\Services;` lalu `composer dump-autoload` |
| Validasi tidak jalan | Type-hint salah | Pastikan parameter `StoreTaskRequest`, bukan `Request` |
| 403 saat create | `authorize()` return false | Pastikan `auth()->check()` dan user login |
| 403 saat update | Cek ownership salah | Bandingkan `$task->user_id` dengan `auth()->id()` |
| Attachment tidak tersimpan | Kolom / fillable | Pastikan migration + `$fillable` punya `attachment` |
| File tidak bisa diakses URL | Belum storage link | `php artisan storage:link` |
| `Call to undefined method tags()` | Modul 10 belum | Hapus sync tag dari service sementara |
| Policy & Form Request bentrok | Double authorize | Pilih satu sumber otorisasi, atau keduanya konsisten |
| `prepareForValidation` tidak jalan | Method typo | Nama harus tepat: `prepareForValidation` |

---

## Checklist Selesai

Centang sebelum lanjut ke Modul 13:

- [ ] `StoreTaskRequest` & `UpdateTaskRequest` ada dan dipakai controller
- [ ] `authorize()` + `rules()` + `messages()` terisi
- [ ] `TaskService` punya `create`, `update`, `delete`
- [ ] Handle attachment (jika Modul 09) ada di service, bukan controller
- [ ] `TaskController` method store/update/destroy tipis (panggil service)
- [ ] CRUD web masih berjalan (Bootstrap CDN, MySQL)
- [ ] Uji create/update/delete + validasi error
- [ ] (Opsional) `CategoryService` dipakai di CategoryController

---

## Git — Commit & Push

```bash
git add .
git commit -m "refactor: form request dan service class"
git push
```

Hanya tiga perintah itu. Tidak perlu branch, PR, atau merge.

---

## Ringkasan Modul 12

| Topik | Yang dipelajari |
|---|---|
| Fat controller | Kenapa bermasalah & cara menghindarinya |
| Form Request | `authorize`, `rules`, `messages`, `validated()` |
| Service class | Logika bisnis terpusat di `app/Services` |
| Thin controller | HTTP orchestration saja |
| Attachment | Upload/hapus file di service (Modul 09) |
| DI | Constructor injection `TaskService` |

---

**Modul sebelumnya:** [11 — Middleware & Authorization](../11-middleware-authorization/README.md)  
**Modul berikutnya:** [13 — API Dasar](../13-api-dasar/README.md)
