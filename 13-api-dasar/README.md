# Modul 13 — API Dasar

Modul ketiga belas: menambah **lapisan API JSON** ke **Task Manager** dengan **Laravel Sanctum** (token auth). Aplikasi web Blade + Bootstrap CDN **tetap jalan**; API adalah lapisan terpisah untuk client seperti Postman, mobile, atau SPA.

**Estimasi waktu:** 2 hari  
**Prasyarat:** [Modul 12 — Form Request & Refactor](../12-form-request-refactor/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Membedakan web routes vs API routes
- [ ] Install Sanctum via Composer (tanpa npm)
- [ ] Menjalankan migrasi Sanctum dan memakai `HasApiTokens` di User
- [ ] Membuat endpoint register / login / logout berbasis token
- [ ] Membuat CRUD tasks API untuk user terautentikasi
- [ ] Mengembalikan JSON (sukses & error validasi)
- [ ] Menguji dengan **curl** dan **Postman**
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
composer require laravel/sanctum
Migrasi personal_access_tokens
User model → use HasApiTokens
routes/api.php
app/Http/Controllers/Api/AuthController.php
app/Http/Controllers/Api/TaskController.php
(opsional) Form Request / TaskService dari Modul 12 dipakai ulang
```

> **Penting:** `composer require` untuk Sanctum **diizinkan**. Tetap **tidak** pakai npm/Vite/Tailwind untuk frontend web. Web app tetap Bootstrap 5 CDN.

---

## Konsep: API vs Web Routes

| Aspek | Web (`routes/web.php`) | API (`routes/api.php`) |
|---|---|---|
| Response | HTML (Blade) | JSON |
| Auth | Session + cookie | Token (Sanctum) |
| CSRF | Wajib (`@csrf`) | Tidak (stateless token) |
| Prefix | `/` | `/api` (otomatis) |
| Client | Browser | Postman, mobile, SPA, curl |
| Redirect | Sering redirect + flash | Status code + body JSON |

### Kapan pakai API?

- Aplikasi mobile (Android/iOS)
- Frontend terpisah (React/Vue) — *di luar scope manual ini*
- Integrasi pihak ketiga
- Testing otomatis / automation

Di project ini: **web tetap utama**, API adalah kemampuan tambahan di project yang sama.

---

## Konsep: Laravel Sanctum

**Sanctum** = paket auth ringan untuk:

1. **API token** (personal access token) — dipakai di modul ini
2. SPA cookie auth (opsional, butuh setup CORS/domain — tidak kita pakai sekarang)

Alur token:

```
1. Client POST /api/login (email + password)
2. Server cek kredensial → buat token → return plain-text token sekali
3. Client simpan token
4. Request berikutnya: Header Authorization: Bearer {token}
5. Logout: hapus token di server
```

Tanpa session browser. Cocok untuk curl/Postman.

---

## Langkah 1: Install Sanctum

Di folder `task-manager`:

```bash
composer require laravel/sanctum
```

Publish config & migration (jika belum otomatis):

```bash
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

Jalankan migrasi (MySQL):

```bash
php artisan migrate
```

Tabel baru: `personal_access_tokens`.

> Database tetap **MySQL**. Jangan ganti ke SQLite hanya untuk API.

---

## Langkah 2: HasApiTokens di User

Edit `app/Models/User.php`:

```php
<?php

namespace App\Models;

use Database\Factories\UserFactory;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    /** @use HasFactory<UserFactory> */
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }

    public function tasks(): HasMany
    {
        return $this->hasMany(Task::class);
    }

    public function categories(): HasMany
    {
        return $this->hasMany(Category::class);
    }
}
```

Trait `HasApiTokens` menambah method: `createToken()`, `tokens()`, `currentAccessToken()`, dll.

---

## Langkah 3: Pastikan routes/api.php Terload

Di Laravel 11/12, API routes biasanya didaftarkan di `bootstrap/app.php`:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    api: __DIR__.'/../routes/api.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
)
```

Jika file `routes/api.php` belum ada, buat:

```bash
# Jika perlu
touch routes/api.php
```

Prefix `/api` otomatis. Middleware default group `api` (throttle, dll.).

---

## Langkah 4: Auth API Controller

```bash
php artisan make:controller Api/AuthController
```

### `app/Http/Controllers/Api/AuthController.php`

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function register(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'max:255', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ]);

        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => $validated['password'], // cast 'hashed' di model
        ]);

        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json([
            'message' => 'Registrasi berhasil.',
            'user' => $user,
            'token' => $token,
        ], 201);
    }

    public function login(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required', 'string'],
        ]);

        $user = User::where('email', $validated['email'])->first();

        if (! $user || ! Hash::check($validated['password'], $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['Email atau password salah.'],
            ]);
        }

        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json([
            'message' => 'Login berhasil.',
            'user' => [
                'id' => $user->id,
                'name' => $user->name,
                'email' => $user->email,
            ],
            'token' => $token,
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        // Hapus token yang sedang dipakai
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logout berhasil. Token dihapus.',
        ]);
    }

    public function me(Request $request): JsonResponse
    {
        return response()->json([
            'user' => $request->user(),
        ]);
    }
}
```

**Validasi error sebagai JSON:** Jika request mengirim header `Accept: application/json` (Postman/curl biasanya), Laravel otomatis return **422** dengan body:

```json
{
  "message": "The email field is required. (and 1 more error)",
  "errors": {
    "email": ["The email field is required."],
    "password": ["The password field is required."]
  }
}
```

Tanpa header itu, request API kadang diarahkan seperti web (redirect). Selalu kirim:

```
Accept: application/json
Content-Type: application/json
```

---

## Langkah 5: Task API Controller

```bash
php artisan make:controller Api/TaskController
```

Pakai ulang `TaskService` dari Modul 12 agar konsisten:

### `app/Http/Controllers/Api/TaskController.php`

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreTaskRequest;
use App\Http\Requests\UpdateTaskRequest;
use App\Models\Task;
use App\Services\TaskService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function __construct(
        protected TaskService $tasks
    ) {}

    public function index(Request $request): JsonResponse
    {
        $tasks = $request->user()
            ->tasks()
            ->with(['category', 'tags'])
            ->latest()
            ->paginate(10);

        return response()->json($tasks);
    }

    public function store(StoreTaskRequest $request): JsonResponse
    {
        $task = $this->tasks->create(
            $request->user(),
            $request->validated(),
            $request->file('attachment')
        );

        return response()->json([
            'message' => 'Task berhasil dibuat.',
            'data' => $task,
        ], 201);
    }

    public function show(Request $request, Task $task): JsonResponse
    {
        if ($task->user_id !== $request->user()->id) {
            return response()->json(['message' => 'Forbidden.'], 403);
        }

        $task->load(['category', 'tags']);

        return response()->json(['data' => $task]);
    }

    public function update(UpdateTaskRequest $request, Task $task): JsonResponse
    {
        if ($task->user_id !== $request->user()->id) {
            return response()->json(['message' => 'Forbidden.'], 403);
        }

        $task = $this->tasks->update(
            $task,
            $request->validated(),
            $request->file('attachment')
        );

        return response()->json([
            'message' => 'Task berhasil diupdate.',
            'data' => $task,
        ]);
    }

    public function destroy(Request $request, Task $task): JsonResponse
    {
        if ($task->user_id !== $request->user()->id) {
            return response()->json(['message' => 'Forbidden.'], 403);
        }

        $this->tasks->delete($task);

        return response()->json([
            'message' => 'Task berhasil dihapus.',
        ]);
    }
}
```

> Upload file via API: pakai `multipart/form-data`, bukan pure JSON. Field file = `attachment`.

Jika Form Request `authorize()` terlalu ketat untuk API, pastikan tetap `auth()->check()` — middleware `auth:sanctum` sudah menjamin user login via token.

---

## Langkah 6: Daftarkan routes/api.php

### `routes/api.php`

```php
<?php

use App\Http\Controllers\Api\AuthController;
use App\Http\Controllers\Api\TaskController;
use Illuminate\Support\Facades\Route;

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/me', [AuthController::class, 'me']);
    Route::post('/logout', [AuthController::class, 'logout']);

    Route::apiResource('tasks', TaskController::class);
});
```

Cek route:

```bash
php artisan route:list --path=api
```

Output yang diharapkan (ringkas):

| Method | URI | Name |
|---|---|---|
| POST | `api/register` | — |
| POST | `api/login` | — |
| POST | `api/logout` | — |
| GET | `api/me` | — |
| GET | `api/tasks` | `tasks.index` (api) |
| POST | `api/tasks` | `tasks.store` |
| GET | `api/tasks/{task}` | `tasks.show` |
| PUT/PATCH | `api/tasks/{task}` | `tasks.update` |
| DELETE | `api/tasks/{task}` | `tasks.destroy` |

`apiResource` **tidak** menyertakan `create`/`edit` (tidak perlu form HTML).

---

## Contoh Response JSON

### Login sukses — `200`

```json
{
  "message": "Login berhasil.",
  "user": {
    "id": 1,
    "name": "Budi",
    "email": "budi@example.com"
  },
  "token": "1|aBcDeFgHiJkLmNoPqRsTuVwXyZ123456"
}
```

### Validasi gagal — `422`

```json
{
  "message": "The title field is required.",
  "errors": {
    "title": ["Judul task wajib diisi."]
  }
}
```

### Unauthorized (tanpa token) — `401`

```json
{
  "message": "Unauthenticated."
}
```

### Forbidden (task milik orang lain) — `403`

```json
{
  "message": "Forbidden."
}
```

### List tasks — `200` (pagination)

```json
{
  "current_page": 1,
  "data": [
    {
      "id": 5,
      "user_id": 1,
      "title": "Belajar Sanctum",
      "description": "Buat API token",
      "is_done": false,
      "category": null,
      "tags": []
    }
  ],
  "first_page_url": "http://127.0.0.1:8000/api/tasks?page=1",
  "from": 1,
  "last_page": 1,
  "per_page": 10,
  "total": 1
}
```

### Created — `201`

```json
{
  "message": "Task berhasil dibuat.",
  "data": {
    "id": 6,
    "title": "Task API baru",
    "is_done": false
  }
}
```

---

## Uji dengan curl

Jalankan server:

```bash
php artisan serve
```

### 1) Register

```bash
curl -s -X POST http://127.0.0.1:8000/api/register \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Budi API",
    "email": "budi.api@example.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'
```

Simpan nilai `token` dari response.

### 2) Login

```bash
curl -s -X POST http://127.0.0.1:8000/api/login \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "budi.api@example.com",
    "password": "password123"
  }'
```

Export token ke variabel shell:

```bash
TOKEN="1|tempel_token_di_sini"
```

### 3) Profil saya

```bash
curl -s http://127.0.0.1:8000/api/me \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN"
```

### 4) Buat task

```bash
curl -s -X POST http://127.0.0.1:8000/api/tasks \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Task dari API",
    "description": "Dibuat via curl",
    "is_done": false
  }'
```

### 5) List tasks

```bash
curl -s http://127.0.0.1:8000/api/tasks \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN"
```

### 6) Update task (ganti `{id}`)

```bash
curl -s -X PUT http://127.0.0.1:8000/api/tasks/1 \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Task diupdate",
    "description": "Update via curl",
    "is_done": true
  }'
```

### 7) Hapus task

```bash
curl -s -X DELETE http://127.0.0.1:8000/api/tasks/1 \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN"
```

### 8) Logout

```bash
curl -s -X POST http://127.0.0.1:8000/api/logout \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN"
```

Setelah logout, request dengan token yang sama harus `401`.

### 9) Uji validasi (sengaja kosong)

```bash
curl -s -X POST http://127.0.0.1:8000/api/tasks \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{}'
```

Harus dapat **422** + object `errors`.

---

## Uji dengan Postman

### Setup collection

1. Buka Postman → **New Collection** → nama: `Task Manager API`
2. Di tab **Variables** collection, buat:
   - `base_url` = `http://127.0.0.1:8000`
   - `token` = (kosong dulu)

### Request Login

1. Method **POST** → `{{base_url}}/api/login`
2. Headers:
   - `Accept: application/json`
   - `Content-Type: application/json`
3. Body → **raw** → **JSON**:

```json
{
  "email": "budi.api@example.com",
  "password": "password123"
}
```

4. Kirim → copy `token` dari response
5. Tempel ke variable collection `token`

**Tips:** Di tab **Scripts** → **Post-response**:

```javascript
const json = pm.response.json();
if (json.token) {
  pm.collectionVariables.set('token', json.token);
}
```

Token tersimpan otomatis setelah login.

### Request terproteksi

Untuk `GET {{base_url}}/api/tasks`:

1. Tab **Authorization** → Type: **Bearer Token**
2. Token: `{{token}}`
3. Header `Accept: application/json`

Atau manual di Headers:

```
Authorization: Bearer {{token}}
```

### Checklist Postman

| Request | Method | URL | Auth |
|---|---|---|---|
| Register | POST | `/api/register` | Tidak |
| Login | POST | `/api/login` | Tidak |
| Me | GET | `/api/me` | Bearer |
| List tasks | GET | `/api/tasks` | Bearer |
| Create task | POST | `/api/tasks` | Bearer |
| Show task | GET | `/api/tasks/:id` | Bearer |
| Update task | PUT | `/api/tasks/:id` | Bearer |
| Delete task | DELETE | `/api/tasks/:id` | Bearer |
| Logout | POST | `/api/logout` | Bearer |

### Upload attachment di Postman

1. POST `/api/tasks`
2. Body → **form-data** (bukan raw JSON)
3. Key `title` (Text), `description` (Text), `attachment` (File)
4. Authorization Bearer tetap diisi

---

## Catatan Arsitektur

```
┌─────────────────────────────────────────────┐
│              Task Manager App               │
├──────────────────┬──────────────────────────┤
│  Web (Blade)     │  API (JSON)              │
│  Bootstrap CDN   │  Sanctum token           │
│  Session auth    │  Bearer token            │
│  routes/web.php  │  routes/api.php          │
│  TaskController  │  Api\TaskController      │
└────────┬─────────┴───────────┬──────────────┘
         │                     │
         └──────────┬──────────┘
                    ▼
            TaskService (Modul 12)
            Models + MySQL
```

Satu database, satu model, dua “pintu masuk”. Web UI tidak diganti.

---

## Latihan Modul 13

### Latihan 1: Endpoint Categories API

Buat `Api\CategoryController` + `Route::apiResource('categories', ...)` di dalam group `auth:sanctum`. Response JSON CRUD kategori milik user.

### Latihan 2: Filter Query di API

Di `GET /api/tasks`, dukung query `?status=done|pending` dan `?search=kata`. Return pagination tetap.

### Latihan 3: Logout Semua Device

Tambah endpoint `POST /api/logout-all` yang memanggil:

```php
$request->user()->tokens()->delete();
```

### Latihan 4: Token Abilities

Saat create token, beri ability:

```php
$user->createToken('api-token', ['tasks:read', 'tasks:write']);
```

Batasi destroy hanya jika token punya `tasks:write` (`$tokenCan`).

### Latihan 5: API Resources (Transformer)

Buat `TaskResource` dengan `php artisan make:resource TaskResource`. Rapikan field JSON (sembunyikan `user_id` internal jika perlu, format tanggal).

### Latihan 6 (Bonus): Rate Limiting

Di `bootstrap/app.php` atau route group, terapkan throttle ketat untuk `/api/login` (mis. 5/minute) agar brute-force lebih sulit.

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| `404` di `/api/login` | `api.php` belum di-load | Cek `bootstrap/app.php` punya `api: ...` |
| `401 Unauthenticated` | Token salah / hilang | Cek header `Authorization: Bearer ...` |
| `Route [login] not defined` | Auth exception redirect ke web login | Kirim `Accept: application/json` |
| Validasi return HTML | Bukan JSON request | Header `Accept` + `Content-Type` |
| `Class HasApiTokens not found` | Sanctum belum terpasang | `composer require laravel/sanctum` |
| Token null setelah create | Salah ambil property | Pakai `->plainTextToken` (bukan object token saja) |
| Tidak bisa akses task user lain | Benar jika 403 | Pastikan cek `user_id` |
| Upload API gagal | Body JSON | Pakai multipart form-data |
| CSRF 419 di API | Salah hit web route / session | Pastikan URL `/api/...` + Bearer |
| Migration error | Tabel sudah ada | Cek migrasi Sanctum sudah jalan |

---

## Checklist Selesai

Centang sebelum lanjut ke Modul 14:

- [ ] `laravel/sanctum` terpasang via Composer
- [ ] Migrasi `personal_access_tokens` sukses di MySQL
- [ ] `User` memakai `HasApiTokens`
- [ ] `POST /api/register` & `/api/login` return token
- [ ] `auth:sanctum` melindungi tasks & logout
- [ ] CRUD `/api/tasks` hanya untuk task milik user
- [ ] Error validasi return JSON 422
- [ ] Diuji dengan curl **dan** Postman
- [ ] Web Blade + Bootstrap CDN masih berfungsi normal

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: api tasks dengan sanctum"
git push
```

Hanya tiga perintah itu. Tidak perlu branch, PR, atau merge.

---

## Ringkasan Modul 13

| Topik | Yang dipelajari |
|---|---|
| Web vs API | HTML/session vs JSON/token |
| Sanctum | Personal access token |
| `routes/api.php` | Prefix `/api`, `apiResource` |
| `auth:sanctum` | Proteksi endpoint |
| JSON errors | 422 validation, 401, 403 |
| curl & Postman | Cara uji API secara praktis |
| Reuse service | Api controller memakai `TaskService` |

---

**Modul sebelumnya:** [12 — Form Request & Refactor](../12-form-request-refactor/README.md)  
**Modul berikutnya:** [14 — Testing](../14-testing/README.md)
