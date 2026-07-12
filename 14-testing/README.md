# Modul 14 — Testing

Modul keempat belas: menulis **feature test** untuk autentikasi dan CRUD task di **Task Manager**. Kamu memastikan perilaku app tetap benar setiap kali kode berubah — tanpa harus klik manual semua halaman.

**Estimasi waktu:** 2 hari  
**Prasyarat:** [Modul 13 — API Dasar](../13-api-dasar/README.md)

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Menjelaskan mengapa automated testing penting
- [ ] Membuat feature test dengan `php artisan make:test`
- [ ] Memakai `RefreshDatabase` dan `actingAs`
- [ ] Menulis test: register, login, guest diblokir, CRUD milik sendiri, tidak bisa update task orang lain
- [ ] (Opsional) Membuat factory `Task`
- [ ] Menjalankan `php artisan test` dan membaca kegagalan
- [ ] Commit & push ke GitHub

**Yang ditambahkan ke project:**

```
tests/Feature/AuthTest.php
tests/Feature/TaskCrudTest.php
database/factories/TaskFactory.php   ← opsional
(phpunit.xml / .env.testing — MySQL testing atau sqlite in-memory sesuai setup)
```

> **Stack:** Tes tetap menguji app MySQL + Blade CDN + Sanctum. Untuk kecepatan lokal, banyak proyek memakai SQLite in-memory **hanya di environment testing** — itu boleh. Database aplikasi utama tetap MySQL.

---

## Mengapa Testing?

| Tanpa test | Dengan test |
|---|---|
| Cek manual setiap ubah kode | Satu perintah: `php artisan test` |
| Mudah lupa edge case | Edge case tertulis & diulang |
| Refactor menakutkan | Refactor aman jika test hijau |
| Bug ketahuan di production | Bug ketahuan di laptop |

**Feature test** = simulasi HTTP request (GET/POST) ke aplikasi, lalu assert response/status/database.

Bukan mengganti uji manual UI sepenuhnya, tapi menutup regresi di alur penting (auth & CRUD).

---

## Konsep: Jenis Test di Laravel

| Jenis | Fokus | Folder |
|---|---|---|
| **Unit** | Class/method kecil tanpa HTTP | `tests/Unit` |
| **Feature** | Request → response / DB | `tests/Feature` |

Modul ini fokus **Feature test**.

---

## Konsep: RefreshDatabase & actingAs

### `RefreshDatabase`

Trait yang **migrate ulang** database testing setiap test (atau pakai transaction + migrate sekali — tergantung versi). Data antar test tidak saling kotor.

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class TaskCrudTest extends TestCase
{
    use RefreshDatabase;
}
```

### `actingAs`

Mensimulasikan user yang sudah login (session web):

```php
$user = User::factory()->create();
$this->actingAs($user);

$response = $this->get('/tasks');
$response->assertOk();
```

Untuk API Sanctum:

```php
$this->actingAs($user, 'sanctum');
// atau
Sanctum::actingAs($user);
```

---

## Setup Database Testing

Buka `phpunit.xml` di root project. Pastikan environment testing terdefinisi:

```xml
<php>
    <env name="APP_ENV" value="testing"/>
    <env name="DB_CONNECTION" value="sqlite"/>
    <env name="DB_DATABASE" value=":memory:"/>
    <env name="CACHE_STORE" value="array"/>
    <env name="SESSION_DRIVER" value="array"/>
    <env name="QUEUE_CONNECTION" value="sync"/>
    <env name="MAIL_MAILER" value="array"/>
</php>
```

**Penjelasan:** SQLite `:memory:` hanya untuk **proses test** agar cepat. App development/production kamu tetap MySQL di `.env`.

Jika ingin test juga ke MySQL terpisah:

```xml
<env name="DB_CONNECTION" value="mysql"/>
<env name="DB_DATABASE" value="task_manager_test"/>
```

Buat database kosong:

```sql
CREATE DATABASE task_manager_test;
```

---

## Langkah 1: Pastikan User Factory Ada

Laravel biasanya sudah punya `database/factories/UserFactory.php`. Contoh minimal:

```php
<?php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;

/**
 * @extends Factory<User>
 */
class UserFactory extends Factory
{
    protected static ?string $password;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => str()->random(10),
        ];
    }
}
```

Model `User` harus `use HasFactory`.

---

## Langkah 2 (Opsional): Task Factory

```bash
php artisan make:factory TaskFactory --model=Task
```

### `database/factories/TaskFactory.php`

```php
<?php

namespace Database\Factories;

use App\Models\Task;
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * @extends Factory<Task>
 */
class TaskFactory extends Factory
{
    protected $model = Task::class;

    public function definition(): array
    {
        return [
            'user_id' => User::factory(),
            'category_id' => null,
            'title' => fake()->sentence(3),
            'description' => fake()->optional()->paragraph(),
            'is_done' => false,
            // 'attachment' => null, // jika kolom ada (Modul 09)
        ];
    }

    public function done(): static
    {
        return $this->state(fn () => ['is_done' => true]);
    }
}
```

Pastikan model `Task`:

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Task extends Model
{
    use HasFactory; // + SoftDeletes jika Modul 10
}
```

Pemakaian:

```php
$task = Task::factory()->create(['user_id' => $user->id]);
$done = Task::factory()->done()->create(['user_id' => $user->id]);
```

---

## Langkah 3: Feature Test Auth

```bash
php artisan make:test AuthTest
```

### `tests/Feature/AuthTest.php`

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class AuthTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_register(): void
    {
        $response = $this->post('/register', [
            'name' => 'Budi Santoso',
            'email' => 'budi@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ]);

        $response->assertRedirect(); // biasanya ke /tasks atau /dashboard
        $this->assertAuthenticated();
        $this->assertDatabaseHas('users', [
            'email' => 'budi@example.com',
            'name' => 'Budi Santoso',
        ]);
    }

    public function test_user_can_login(): void
    {
        $user = User::factory()->create([
            'email' => 'budi@example.com',
            'password' => 'password123',
        ]);

        $response = $this->post('/login', [
            'email' => 'budi@example.com',
            'password' => 'password123',
        ]);

        $response->assertRedirect();
        $this->assertAuthenticatedAs($user);
    }

    public function test_login_fails_with_wrong_password(): void
    {
        User::factory()->create([
            'email' => 'budi@example.com',
            'password' => 'password123',
        ]);

        $response = $this->from('/login')->post('/login', [
            'email' => 'budi@example.com',
            'password' => 'salah',
        ]);

        $response->assertRedirect('/login');
        $this->assertGuest();
    }

    public function test_register_validation_requires_fields(): void
    {
        $response = $this->from('/register')->post('/register', []);

        $response->assertRedirect('/register');
        $response->assertSessionHasErrors(['name', 'email', 'password']);
        $this->assertGuest();
    }
}
```

**Sesuaikan URL** dengan route auth kamu di Modul 07 (`/login`, `/register`, nama form field). Jika pakai named route:

```php
$this->post(route('register'), [...]);
```

---

## Langkah 4: Feature Test Task CRUD & Authorization

```bash
php artisan make:test TaskCrudTest
```

### `tests/Feature/TaskCrudTest.php`

```php
<?php

namespace Tests\Feature;

use App\Models\Task;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class TaskCrudTest extends TestCase
{
    use RefreshDatabase;

    public function test_guest_cannot_access_tasks_index(): void
    {
        $response = $this->get('/tasks');

        $response->assertRedirect('/login');
    }

    public function test_guest_cannot_store_task(): void
    {
        $response = $this->post('/tasks', [
            'title' => 'Task tanpa login',
            'description' => 'Seharusnya ditolak',
        ]);

        $response->assertRedirect('/login');
        $this->assertDatabaseCount('tasks', 0);
    }

    public function test_authenticated_user_can_create_task(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/tasks', [
            'title' => 'Belajar Testing',
            'description' => 'Menulis feature test Laravel',
            'is_done' => false,
        ]);

        $response->assertRedirect(route('tasks.index'));
        $this->assertDatabaseHas('tasks', [
            'user_id' => $user->id,
            'title' => 'Belajar Testing',
        ]);
    }

    public function test_authenticated_user_can_view_own_tasks(): void
    {
        $user = User::factory()->create();
        Task::factory()->create([
            'user_id' => $user->id,
            'title' => 'Task Saya',
        ]);

        $response = $this->actingAs($user)->get('/tasks');

        $response->assertOk();
        $response->assertSee('Task Saya');
    }

    public function test_authenticated_user_can_update_own_task(): void
    {
        $user = User::factory()->create();
        $task = Task::factory()->create([
            'user_id' => $user->id,
            'title' => 'Judul Lama',
        ]);

        $response = $this->actingAs($user)->put("/tasks/{$task->id}", [
            'title' => 'Judul Baru',
            'description' => 'Deskripsi diupdate',
            'is_done' => true,
        ]);

        $response->assertRedirect(route('tasks.index'));
        $this->assertDatabaseHas('tasks', [
            'id' => $task->id,
            'title' => 'Judul Baru',
            'is_done' => true,
        ]);
    }

    public function test_authenticated_user_can_delete_own_task(): void
    {
        $user = User::factory()->create();
        $task = Task::factory()->create(['user_id' => $user->id]);

        $response = $this->actingAs($user)->delete("/tasks/{$task->id}");

        $response->assertRedirect(route('tasks.index'));

        // Soft delete (Modul 10):
        $this->assertSoftDeleted('tasks', ['id' => $task->id]);
        // Jika hard delete:
        // $this->assertDatabaseMissing('tasks', ['id' => $task->id]);
    }

    public function test_user_cannot_update_others_task(): void
    {
        $owner = User::factory()->create();
        $intruder = User::factory()->create();

        $task = Task::factory()->create([
            'user_id' => $owner->id,
            'title' => 'Milik Owner',
        ]);

        $response = $this->actingAs($intruder)->put("/tasks/{$task->id}", [
            'title' => 'Diganti Intruder',
            'description' => 'Tidak boleh',
            'is_done' => true,
        ]);

        $response->assertForbidden(); // 403
        $this->assertDatabaseHas('tasks', [
            'id' => $task->id,
            'title' => 'Milik Owner',
            'user_id' => $owner->id,
        ]);
    }

    public function test_user_cannot_delete_others_task(): void
    {
        $owner = User::factory()->create();
        $intruder = User::factory()->create();
        $task = Task::factory()->create(['user_id' => $owner->id]);

        $response = $this->actingAs($intruder)->delete("/tasks/{$task->id}");

        $response->assertForbidden();
        $this->assertDatabaseHas('tasks', ['id' => $task->id]);
    }

    public function test_user_cannot_see_others_task_on_show(): void
    {
        $owner = User::factory()->create();
        $intruder = User::factory()->create();
        $task = Task::factory()->create(['user_id' => $owner->id]);

        $response = $this->actingAs($intruder)->get("/tasks/{$task->id}");

        $response->assertForbidden();
    }

    public function test_store_task_validation_requires_title(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->from(route('tasks.create'))
            ->post('/tasks', [
                'title' => '',
                'description' => 'Deskripsi saja',
            ]);

        $response->assertRedirect(route('tasks.create'));
        $response->assertSessionHasErrors(['title']);
        $this->assertDatabaseCount('tasks', 0);
    }
}
```

**Sesuaikan** field wajib dengan `StoreTaskRequest` / `UpdateTaskRequest` Modul 12 (misalnya jika `description` required min 10, sesuaikan payload test).

---

## Langkah 5 (Disarankan): Test API Sanctum

```bash
php artisan make:test ApiTaskTest
```

### `tests/Feature/ApiTaskTest.php`

```php
<?php

namespace Tests\Feature;

use App\Models\Task;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Laravel\Sanctum\Sanctum;
use Tests\TestCase;

class ApiTaskTest extends TestCase
{
    use RefreshDatabase;

    public function test_api_login_returns_token(): void
    {
        $user = User::factory()->create([
            'email' => 'api@example.com',
            'password' => 'password123',
        ]);

        $response = $this->postJson('/api/login', [
            'email' => 'api@example.com',
            'password' => 'password123',
        ]);

        $response->assertOk()
            ->assertJsonStructure(['message', 'user', 'token']);
    }

    public function test_api_guest_cannot_list_tasks(): void
    {
        $this->getJson('/api/tasks')->assertUnauthorized();
    }

    public function test_api_user_can_create_task(): void
    {
        $user = User::factory()->create();
        Sanctum::actingAs($user);

        $response = $this->postJson('/api/tasks', [
            'title' => 'Task API',
            'description' => 'Dari feature test API',
        ]);

        $response->assertCreated()
            ->assertJsonPath('data.title', 'Task API');

        $this->assertDatabaseHas('tasks', [
            'user_id' => $user->id,
            'title' => 'Task API',
        ]);
    }

    public function test_api_user_cannot_update_others_task(): void
    {
        $owner = User::factory()->create();
        $intruder = User::factory()->create();
        $task = Task::factory()->create(['user_id' => $owner->id]);

        Sanctum::actingAs($intruder);

        $this->putJson("/api/tasks/{$task->id}", [
            'title' => 'Hacked',
            'description' => 'Tidak boleh',
        ])->assertForbidden();
    }
}
```

`postJson` / `getJson` otomatis set `Accept: application/json`.

---

## Langkah 6: Jalankan Test

```bash
php artisan test
```

Atau filter:

```bash
php artisan test --filter=AuthTest
php artisan test --filter=test_user_cannot_update_others_task
php artisan test tests/Feature/TaskCrudTest.php
```

Output sukses (contoh):

```
PASS  Tests\Feature\AuthTest
✓ user can register
✓ user can login

PASS  Tests\Feature\TaskCrudTest
✓ guest cannot access tasks index
✓ authenticated user can create task
✓ user cannot update others task

Tests:  12 passed
```

---

## Cara Membaca Kegagalan

### Contoh 1: Expected 403, got 200

```
Failed asserting that the response status code is 403.
Expected: 403
Actual  : 200
```

**Arti:** Authorization belum jalan — user lain masih bisa update. Perbaiki Policy / Form Request / cek `user_id`.

### Contoh 2: Redirect salah

```
Failed asserting that two strings are equal.
Expected: http://localhost/login
Actual  : http://localhost/tasks
```

**Arti:** Guest tidak diarahkan ke login, atau middleware `auth` belum dipasang.

### Contoh 3: Database missing row

```
Failed asserting that a row in the table [tasks] matches the attributes {...}
```

**Arti:** Create gagal (validasi, fillable, service error). Cek payload test vs rules Form Request.

### Contoh 4: Session errors tidak ada

```
Session is missing expected key [errors]
```

**Arti:** Validasi tidak memicu error — mungkin field optional, atau request tidak lewat Form Request.

### Tips debug

```php
$response->dump();        // body
$response->dumpSession(); // session / errors
dd($response->status());
```

Jalankan **satu** test yang gagal dulu (`--filter`), perbaiki, baru jalankan semua.

---

## Assertion yang Sering Dipakai

| Assertion | Kegunaan |
|---|---|
| `assertOk()` | Status 200 |
| `assertCreated()` | Status 201 |
| `assertForbidden()` | Status 403 |
| `assertUnauthorized()` | Status 401 |
| `assertRedirect(...)` | Redirect URL |
| `assertSee('teks')` | HTML mengandung teks |
| `assertSessionHasErrors([...])` | Validasi gagal |
| `assertAuthenticated()` | User login (session) |
| `assertGuest()` | Belum login |
| `assertDatabaseHas(...)` | Row ada di DB |
| `assertSoftDeleted(...)` | Soft delete |
| `assertJsonPath(...)` | Nilai JSON path |
| `assertJsonStructure([...])` | Struktur JSON |

---

## Latihan Modul 14

### Latihan 1: Test CRUD Kategori

Buat `CategoryCrudTest`: guest diblokir, user bisa create/update/delete kategori sendiri, user tidak bisa menghapus kategori milik orang lain.

### Latihan 2: Test Search & Filter

Buat beberapa task, request `GET /tasks?search=laporan`, assert hanya task matching yang muncul (`assertSee` / `assertDontSee`).

### Latihan 3: Test API Register + Logout

1. `POST /api/register` → dapat token  
2. `GET /api/me` dengan Bearer → 200  
3. `POST /api/logout` → 200  
4. Request berikutnya dengan token lama → 401  

### Latihan 4: Test Upload Attachment

Pakai `Illuminate\Http\UploadedFile::fake()->create('doc.pdf', 100, 'application/pdf')` di `post` multipart. Assert file tersimpan / path di DB.

### Latihan 5: Test Validasi API 422

`postJson('/api/tasks', [])` sebagai user Sanctum → `assertStatus(422)` + `assertJsonValidationErrors(['title'])`.

### Latihan 6 (Bonus): Policy Unit-ish Test

```bash
php artisan make:test TaskPolicyTest
```

Pakai `$user->can('update', $task)` dengan `assertTrue` / `assertFalse` tanpa full HTTP (tetap Feature atau Unit — yang penting Policy teruji).

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| `could not find driver (sqlite)` | Extension sqlite belum aktif | Aktifkan `pdo_sqlite` **atau** pakai MySQL test DB |
| Semua test gagal migrate | Migration error | Jalankan migrate di env testing; cek softDeletes/kolom |
| `CSRF token mismatch` | Post tanpa bypass | Feature test Laravel otomatis bypass CSRF — pastikan extends `Tests\TestCase` |
| `419` di test | Salah base TestCase | Jangan extends PHPUnit langsung |
| Factory `Task` error | Belum HasFactory / fillable | Tambah trait + kolom fillable |
| `assertForbidden` dapat 404 | Route model binding / scoping | Cek apakah query global scope menyembunyikan record |
| Password cast double-hash | Hash::make + cast hashed | Di factory cukup plain `'password'` jika model cast `hashed` |
| Test API 302 ke login | Bukan JSON | Pakai `getJson`/`postJson` atau header Accept |
| Soft delete assert gagal | Pakai assertDatabaseMissing | Ganti `assertSoftDeleted` |
| Data bocor antar test | Lupa RefreshDatabase | `use RefreshDatabase;` |

---

## Checklist Selesai

Centang sebelum lanjut ke Modul 15:

- [ ] `AuthTest`: register & login lulus
- [ ] Guest tidak bisa akses `/tasks` (redirect login)
- [ ] User bisa create / update / delete task sendiri
- [ ] User **tidak** bisa update (dan idealnya delete/show) task orang lain → 403
- [ ] `php artisan test` hijau (semua test yang kamu tulis)
- [ ] (Opsional) `TaskFactory` dipakai di test
- [ ] (Opsional/disarankan) `ApiTaskTest` untuk Sanctum
- [ ] Kamu paham cara baca failure message

---

## Git — Commit & Push

```bash
git add .
git commit -m "test: feature test auth dan crud task"
git push
```

Hanya tiga perintah itu. Tidak perlu branch, PR, atau merge.

---

## Ringkasan Modul 14

| Topik | Yang dipelajari |
|---|---|
| Feature test | Simulasi HTTP + assert response/DB |
| `RefreshDatabase` | DB testing bersih tiap run |
| `actingAs` / Sanctum | Login palsu di test |
| Factory | Data dummy cepat & konsisten |
| Authorization tests | Pastikan 403 untuk resource orang lain |
| `php artisan test` | Menjalankan & memfilter test |
| Membaca failure | Status, redirect, DB, validation |

---

**Modul sebelumnya:** [13 — API Dasar](../13-api-dasar/README.md)  
**Modul berikutnya:** [15 — Mail & Notification](../15-mail-notification/README.md)
