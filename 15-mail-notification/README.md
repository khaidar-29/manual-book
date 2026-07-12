# Modul 15 — Mail & Notification

Modul kelima belas: mengirim **email notifikasi** saat task dibuat di **Task Manager**, menggunakan **Mailtrap** sebagai inbox pengujian SMTP. Kamu juga akan mengenal konsep **Mailable**, template email Blade, dan opsional **Notification** (mail + database).

**Estimasi waktu:** 1–2 hari  
**Prasyarat:** [Modul 14 — Testing](../14-testing/README.md)

> **Stack tetap sama:** MySQL only, Bootstrap 5 CDN untuk web UI, **tanpa** Vite / Tailwind / Laravel UI. Email template boleh HTML sederhana (table layout) — tidak wajib Bootstrap di email.

---

## Tujuan Modul

Setelah modul ini selesai, kamu sudah bisa:

- [ ] Mengonfigurasi `.env` untuk Mailtrap SMTP
- [ ] Membuat class `TaskCreatedMail` (Mailable)
- [ ] Membuat Blade template email HTML
- [ ] Mengirim email saat task dibuat (dari `TaskService` atau controller)
- [ ] Memverifikasi email di inbox Mailtrap
- [ ] (Opsional) Membuat Notification class channel `mail` + `database`
- [ ] Memahami perbedaan sync vs queue untuk pengiriman email
- [ ] Commit & push: `feat: email notifikasi task`

**Yang ditambahkan ke project:**

```
.env / .env.example          ← konfigurasi MAIL_* Mailtrap
app/Mail/TaskCreatedMail.php
resources/views/emails/task-created.blade.php
Integrasi kirim email di TaskService (atau TaskController@store)
(Opsional) app/Notifications/TaskCreatedNotification.php
(Opsional) migration notifications table
```

---

## Mengapa Email di Task Manager?

Di aplikasi nyata, user sering ingin tahu saat:

- Task baru dibuat (konfirmasi)
- Task di-assign ke mereka
- Deadline mendekat (reminder)
- Task selesai

Modul ini fokus pada **TaskCreatedMail** — fondasi yang sama dipakai untuk notifikasi lain.

---

## Konsep: Mail vs Notification

| Konsep | Kapan dipakai | Ciri |
|---|---|---|
| **Mailable** | Kirim email spesifik, kontrol penuh | Class di `app/Mail/`, template Blade |
| **Notification** | Satu event → banyak channel (mail, database, Slack, dll.) | Class di `app/Notifications/`, method `via()` |

Untuk pemula: mulai dengan **Mailable**. Notification opsional di akhir modul.

---

## Konsep: Mailtrap

**Mailtrap** = sandbox SMTP untuk development.

- Email **tidak** sampai ke inbox sungguhan
- Semua email tertangkap di dashboard Mailtrap
- Aman: tidak spam user nyata saat testing

Alur:

```
App Laravel → SMTP sandbox.smtp.mailtrap.io → Inbox Mailtrap (bukan Gmail user)
```

Daftar gratis di: [https://mailtrap.io](https://mailtrap.io)

---

## Langkah 1: Buat Akun Mailtrap & Ambil Kredensial

1. Daftar / login di Mailtrap
2. Buat (atau pilih) **Email Testing** inbox
3. Buka tab **SMTP Settings** → pilih **Laravel** atau **SMTP**
4. Catat nilai berikut:

| Setting | Contoh nilai |
|---|---|
| Host | `sandbox.smtp.mailtrap.io` |
| Port | `2525` (atau `587`) |
| Username | string dari Mailtrap |
| Password | string dari Mailtrap |
| Encryption | `tls` (biasanya) |

> **Jangan** commit username/password ke GitHub. Hanya taruh di `.env` lokal.

---

## Langkah 2: Konfigurasi `.env`

Buka file `.env` di root project Task Manager. Ubah bagian mail:

```env
MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=isi_username_mailtrap_kamu
MAIL_PASSWORD=isi_password_mailtrap_kamu
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="noreply@taskmanager.test"
MAIL_FROM_NAME="${APP_NAME}"
```

Penjelasan singkat:

| Key | Fungsi |
|---|---|
| `MAIL_MAILER=smtp` | Pakai driver SMTP (bukan `log` / `array`) |
| `MAIL_HOST` | Server Mailtrap |
| `MAIL_PORT` | Port SMTP (2525 umum di Mailtrap) |
| `MAIL_USERNAME` / `MAIL_PASSWORD` | Auth ke sandbox |
| `MAIL_FROM_ADDRESS` | Alamat pengirim yang tampil di email |
| `MAIL_FROM_NAME` | Nama pengirim (biasanya nama app) |

Update juga `.env.example` (tanpa secret nyata):

```env
MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="noreply@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

Setelah ubah `.env`, clear config cache jika pernah di-cache:

```bash
php artisan config:clear
```

---

## Langkah 3: Pastikan Queue Sync (Awal)

Untuk modul ini, kirim email **langsung (sync)** dulu agar mudah di-debug.

Di `.env`:

```env
QUEUE_CONNECTION=sync
```

Dengan `sync`, `Mail::send()` / `Mail::to()->send()` berjalan di request yang sama — tidak perlu worker.

> Catatan singkat tentang queue ada di bagian **Opsional: Queue** di bawah. Untuk production nanti, queue sangat disarankan.

---

## Langkah 4: Buat Mailable `TaskCreatedMail`

```bash
php artisan make:mail TaskCreatedMail --markdown=emails.task-created
```

Atau tanpa markdown (HTML Blade biasa — **disarankan untuk modul ini**):

```bash
php artisan make:mail TaskCreatedMail
```

Lalu buat view sendiri di `resources/views/emails/task-created.blade.php`.

### Isi `app/Mail/TaskCreatedMail.php`

```php
<?php

namespace App\Mail;

use App\Models\Task;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class TaskCreatedMail extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct(
        public Task $task
    ) {}

    /**
     * Get the message envelope.
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Task Baru Dibuat: ' . $this->task->title,
        );
    }

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'emails.task-created',
            with: [
                'task' => $this->task,
                'user' => $this->task->user,
            ],
        );
    }

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [];
    }
}
```

Catatan:

- Property `public Task $task` otomatis tersedia di Blade sebagai `$task`
- `SerializesModels` penting jika nanti email di-queue (Eloquent model di-serialize aman)
- Subject bisa dinamis berdasarkan judul task

---

## Langkah 5: Buat Blade Template Email

Buat file `resources/views/emails/task-created.blade.php`:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Task Baru Dibuat</title>
</head>
<body style="margin:0; padding:0; background:#f4f4f4; font-family: Arial, Helvetica, sans-serif;">
    <table width="100%" cellpadding="0" cellspacing="0" role="presentation" style="background:#f4f4f4; padding:24px 0;">
        <tr>
            <td align="center">
                <table width="600" cellpadding="0" cellspacing="0" role="presentation" style="background:#ffffff; border:1px solid #e5e5e5;">
                    {{-- Header --}}
                    <tr>
                        <td style="background:#0d6efd; color:#ffffff; padding:20px 24px; font-size:20px; font-weight:bold;">
                            {{ config('app.name', 'Task Manager') }}
                        </td>
                    </tr>

                    {{-- Body --}}
                    <tr>
                        <td style="padding:24px; color:#333333; font-size:14px; line-height:1.6;">
                            <p style="margin:0 0 12px;">Halo <strong>{{ $user->name ?? 'User' }}</strong>,</p>

                            <p style="margin:0 0 16px;">
                                Task baru berhasil dibuat. Berikut ringkasannya:
                            </p>

                            <table width="100%" cellpadding="8" cellspacing="0" role="presentation" style="border:1px solid #eeeeee; margin-bottom:16px;">
                                <tr>
                                    <td style="width:140px; background:#f8f9fa; font-weight:bold;">Judul</td>
                                    <td>{{ $task->title }}</td>
                                </tr>
                                <tr>
                                    <td style="background:#f8f9fa; font-weight:bold;">Status</td>
                                    <td>{{ $task->is_done ? 'Selesai' : 'Belum selesai' }}</td>
                                </tr>
                                @if(!empty($task->due_date))
                                <tr>
                                    <td style="background:#f8f9fa; font-weight:bold;">Deadline</td>
                                    <td>{{ \Illuminate\Support\Carbon::parse($task->due_date)->format('d M Y') }}</td>
                                </tr>
                                @endif
                                @if($task->category)
                                <tr>
                                    <td style="background:#f8f9fa; font-weight:bold;">Kategori</td>
                                    <td>{{ $task->category->name }}</td>
                                </tr>
                                @endif
                                @if(!empty($task->description))
                                <tr>
                                    <td style="background:#f8f9fa; font-weight:bold; vertical-align:top;">Deskripsi</td>
                                    <td>{{ $task->description }}</td>
                                </tr>
                                @endif
                            </table>

                            <p style="margin:0 0 8px;">
                                <a href="{{ url('/tasks/' . $task->id) }}"
                                   style="display:inline-block; background:#0d6efd; color:#ffffff; text-decoration:none; padding:10px 16px; border-radius:4px;">
                                    Lihat Task
                                </a>
                            </p>

                            <p style="margin:16px 0 0; color:#777777; font-size:12px;">
                                Email ini dikirim otomatis oleh sistem Task Manager.
                            </p>
                        </td>
                    </tr>

                    {{-- Footer --}}
                    <tr>
                        <td style="padding:16px 24px; background:#f8f9fa; color:#888888; font-size:12px;">
                            &copy; {{ date('Y') }} {{ config('app.name', 'Task Manager') }}
                        </td>
                    </tr>
                </table>
            </td>
        </tr>
    </table>
</body>
</html>
```

Kenapa table layout?

- Banyak client email (Gmail, Outlook) lebih stabil dengan table
- CSS modern terbatas di email
- Bootstrap CDN **tidak** dipakai di email template (boleh tetap dipakai di UI web app)

Pastikan relasi `user` dan `category` tersedia. Jika belum di-load, eager load saat kirim email (lihat langkah berikutnya).

---

## Langkah 6: Kirim Email Saat Task Dibuat

Ada dua tempat umum. Pilih sesuai struktur project setelah Modul 12 (Form Request & Refactor).

### Opsi A — Dari `TaskService` (disarankan)

Jika kamu punya `app/Services/TaskService.php`:

```php
<?php

namespace App\Services;

use App\Mail\TaskCreatedMail;
use App\Models\Task;
use App\Models\User;
use Illuminate\Support\Facades\Mail;

class TaskService
{
    public function create(User $user, array $data): Task
    {
        $task = $user->tasks()->create($data);

        // Pastikan relasi ter-load untuk template email
        $task->load(['user', 'category']);

        Mail::to($user->email)->send(new TaskCreatedMail($task));

        return $task;
    }

    // ... method update, delete, dll.
}
```

Controller tetap tipis:

```php
public function store(StoreTaskRequest $request, TaskService $taskService)
{
    $task = $taskService->create(auth()->user(), $request->validated());

    return redirect()
        ->route('tasks.show', $task)
        ->with('success', 'Task berhasil dibuat. Notifikasi email telah dikirim.');
}
```

### Opsi B — Dari `TaskController@store`

Jika belum pakai service:

```php
use App\Mail\TaskCreatedMail;
use Illuminate\Support\Facades\Mail;

public function store(Request $request)
{
    $validated = $request->validate([
        'title'       => ['required', 'string', 'max:255'],
        'description' => ['nullable', 'string'],
        'category_id' => ['nullable', 'exists:categories,id'],
        'due_date'    => ['nullable', 'date'],
    ]);

    $task = auth()->user()->tasks()->create($validated);
    $task->load(['user', 'category']);

    Mail::to(auth()->user()->email)->send(new TaskCreatedMail($task));

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task berhasil dibuat.');
}
```

### Jangan kirim email di `update` / `destroy` dulu

Fokus modul: **hanya saat create**. Latihan di akhir akan meminta email lain.

---

## Langkah 7: Uji Kirim Email & Verifikasi di Mailtrap

1. Pastikan `php artisan serve` jalan
2. Login ke Task Manager
3. Buat task baru lewat form create
4. Buka dashboard Mailtrap → inbox Email Testing
5. Seharusnya muncul email dengan subject seperti:  
   `Task Baru Dibuat: Judul Task Kamu`

Checklist verifikasi:

| # | Cek | OK? |
|---|---|---|
| 1 | Email masuk ke Mailtrap (bukan error di browser) | |
| 2 | Subject mengandung judul task | |
| 3 | Nama user penerima benar | |
| 4 | Judul, status, deskripsi tampil | |
| 5 | Link "Lihat Task" mengarah ke URL yang masuk akal | |
| 6 | From address sesuai `MAIL_FROM_ADDRESS` | |

Jika request lambat beberapa detik — normal untuk SMTP sync. Itu salah satu alasan production memakai queue.

---

## Langkah 8 (Opsional): Notification Class (Mail + Database)

Notification berguna jika suatu saat kamu ingin:

- Email + notifikasi di dalam app (bell icon)
- Banyak channel tanpa duplikasi logika

### 8.1 Migration tabel `notifications`

```bash
php artisan notifications:table
php artisan migrate
```

### 8.2 Buat Notification

```bash
php artisan make:notification TaskCreatedNotification
```

Isi `app/Notifications/TaskCreatedNotification.php`:

```php
<?php

namespace App\Notifications;

use App\Models\Task;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;

class TaskCreatedNotification extends Notification
{
    use Queueable;

    public function __construct(
        public Task $task
    ) {}

    /**
     * Channel yang dipakai.
     *
     * @return array<int, string>
     */
    public function via(object $notifiable): array
    {
        return ['mail', 'database'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Task Baru Dibuat: ' . $this->task->title)
            ->greeting('Halo ' . $notifiable->name . ',')
            ->line('Task baru berhasil dibuat.')
            ->line('Judul: ' . $this->task->title)
            ->action('Lihat Task', url('/tasks/' . $this->task->id))
            ->line('Terima kasih telah menggunakan Task Manager!');
    }

    /**
     * @return array<string, mixed>
     */
    public function toDatabase(object $notifiable): array
    {
        return [
            'task_id' => $this->task->id,
            'title'   => $this->task->title,
            'message' => 'Task baru dibuat: ' . $this->task->title,
        ];
    }
}
```

### 8.3 Kirim via User model

User model Laravel biasanya sudah `use Notifiable`.

Di `TaskService`:

```php
use App\Notifications\TaskCreatedNotification;

// Ganti atau lengkapi Mail::send
$user->notify(new TaskCreatedNotification($task));
```

Atau tetap pakai Mailable untuk email custom HTML, dan Notification hanya untuk database — terserah, tapi jangan kirim **dua email** ganda tanpa sengaja.

Rekomendasi latihan:

- Production-like: Mailable untuk HTML custom
- Atau: Notification `toMail()` saja (template default Laravel)

### 8.4 Baca notifikasi database (opsional UI)

```php
auth()->user()->unreadNotifications;
auth()->user()->notifications;
```

Tandai sudah dibaca:

```php
$notification->markAsRead();
```

UI Bootstrap sederhana (opsional, di navbar):

```blade
@auth
    @php $unread = auth()->user()->unreadNotifications->count(); @endphp
    <span class="badge text-bg-secondary">{{ $unread }} notifikasi</span>
@endauth
```

---

## Opsional: Catatan Queue (Singkat)

### Sync (modul ini)

```env
QUEUE_CONNECTION=sync
```

Email dikirim di request HTTP yang sama.

### Database queue (pengenalan)

Jika ingin mencoba async:

```env
QUEUE_CONNECTION=database
```

```bash
php artisan queue:table
php artisan migrate
php artisan queue:work
```

Ubah pengiriman:

```php
Mail::to($user->email)->queue(new TaskCreatedMail($task));
// atau
Mail::to($user->email)->send(new TaskCreatedMail($task)); // sync
```

Agar Mailable bisa di-queue dengan baik, implementasikan `ShouldQueue`:

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class TaskCreatedMail extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;
    // ...
}
```

Untuk modul 15: **sync sudah cukup**. Queue diperdalam saat deploy / scaling.

---

## Integrasi dengan Fitur yang Sudah Ada

Ingat: forgot password / reset password di Modul 07 (atau fitur tambahan) **mungkin sudah mengirim email**. Jangan rusak konfigurasi itu.

Cek:

1. Setelah ubah `.env` ke Mailtrap, uji juga **Forgot Password** (jika ada)
2. Email reset harus muncul di Mailtrap juga
3. Link reset harus tetap valid (`APP_URL` benar)

Ini membuktikan satu konfigurasi SMTP melayani semua mail app.

---

## Troubleshooting

### Email tidak muncul di Mailtrap

| Kemungkinan | Solusi |
|---|---|
| `MAIL_MAILER=log` masih aktif | Set `smtp`, lalu `php artisan config:clear` |
| Username/password salah | Copy ulang dari Mailtrap SMTP settings |
| Port salah | Coba `2525` atau `587` |
| Config ter-cache | `php artisan config:clear` |
| Exception ditelan | Cek `storage/logs/laravel.log` |
| Email dikirim ke alamat lain | Pastikan `Mail::to($user->email)` benar |

### Error "Connection could not be established"

- Cek internet
- Firewall / antivirus memblokir outbound SMTP
- Host harus `sandbox.smtp.mailtrap.io` (bukan `127.0.0.1`)

### Error "Swift_TransportException" / "Failed to authenticate"

- Username/password Mailtrap salah atau expired
- Inbox Mailtrap dihapus — buat inbox baru

### Email masuk tapi tampilan berantakan

- Hindari CSS flex/grid modern di email
- Pakai inline `style=""`
- Pakai table layout seperti contoh

### Request timeout saat create task

- SMTP sync lambat — normal di local
- Untuk development, sementara bisa `MAIL_MAILER=log` (email ke log file)
- Atau pakai queue + `queue:work`

### `Undefined relationship [user]`

```php
$task->load('user');
// atau pastikan Task belongsTo User
```

### Link di email `http://localhost` tidak bisa diklik dari HP

Di local itu wajar. Saat deploy (Modul 16), set:

```env
APP_URL=https://domain-kamu.com
```

---

## Best Practices

1. **Jangan commit `.env`** — hanya `.env.example` tanpa secret
2. **Kirim email setelah data tersimpan** — jangan sebelum `create()` sukses
3. **Eager load relasi** sebelum render template
4. **Subject jelas** — user harus paham isi email tanpa buka body
5. **Satu tanggung jawab** — create task ≠ update task; bedakan Mailable
6. **Gagal kirim email jangan selalu gagalkan create** (opsional advanced):

```php
try {
    Mail::to($user->email)->send(new TaskCreatedMail($task));
} catch (\Throwable $e) {
    report($e);
    // task tetap tersimpan; user lihat flash warning
}
```

Untuk modul ini, biarkan exception muncul dulu agar mudah di-debug.

---

## Latihan Modul 15

### Latihan 1: `TaskCompletedMail`

Buat Mailable + template email yang dikirim saat task ditandai selesai (`is_done = true`).

Hint:

```php
if ($task->is_done) {
    Mail::to($task->user->email)->send(new TaskCompletedMail($task));
}
```

Kirim hanya saat **berubah** dari belum selesai → selesai (bukan setiap update).

### Latihan 2: `TaskAssignedMail` atau Reminder

Pilih salah satu:

**A. TaskAssigned** (jika ada kolom `assigned_to` / assign user lain):

- Email ke user yang di-assign
- Subject: `Kamu ditugaskan: {judul}`

**B. Reminder sederhana** (tanpa cron dulu):

- Buat command Artisan `tasks:remind-due-tomorrow`
- Query task dengan `due_date = tomorrow`
- Kirim email reminder ke owner
- Jalankan manual: `php artisan tasks:remind-due-tomorrow`

```bash
php artisan make:command RemindDueTasks
php artisan make:mail TaskReminderMail
```

### Latihan 3: Tes Manual Lengkap

| # | Skenario | Hasil |
|---|---|---|
| 1 | Create task | Email TaskCreated di Mailtrap |
| 2 | Update judul saja | Tidak kirim TaskCreated lagi |
| 3 | Tandai selesai | Email TaskCompleted (latihan 1) |
| 4 | Forgot password (jika ada) | Email reset di Mailtrap |
| 5 | Username Mailtrap salah | Error jelas di log / halaman |

### Latihan 4 (Opsional): Notification Database

- Simpan notifikasi ke DB saat create task
- Tampilkan jumlah unread di navbar Bootstrap
- Halaman `/notifications` list + mark as read

---

## Checklist Modul 15

### Konfigurasi

- [ ] Akun Mailtrap dibuat
- [ ] `.env` memakai `MAIL_MAILER=smtp` + host Mailtrap
- [ ] `.env.example` diupdate (tanpa password nyata)
- [ ] `php artisan config:clear` sudah dijalankan
- [ ] `QUEUE_CONNECTION=sync` untuk uji awal

### Kode

- [ ] `app/Mail/TaskCreatedMail.php` ada
- [ ] `resources/views/emails/task-created.blade.php` ada (HTML table)
- [ ] Email dikirim dari `TaskService` atau `TaskController@store`
- [ ] Relasi `user` (dan `category` jika dipakai) di-load
- [ ] (Opsional) Notification + migration `notifications`

### Verifikasi

- [ ] Create task → email muncul di Mailtrap
- [ ] Subject, body, link benar
- [ ] Forgot password (jika ada) masih jalan via Mailtrap
- [ ] Tidak ada secret Mailtrap di Git

### Git

- [ ] Commit pesan: `feat: email notifikasi task`
- [ ] Push ke `main`

---

## Git — Commit & Push

```bash
git add .
git commit -m "feat: email notifikasi task"
git push
```

Ingat: **hanya** `add` → `commit` → `push` ke `main`. Tidak perlu branch, PR, atau merge.

Sebelum commit, pastikan `.env` **tidak** ter-stage:

```bash
git status
# .env harus tetap untracked / ignored
```

---

## Ringkasan Alur

```
User submit form create task
        ↓
Controller / TaskService menyimpan Task ke MySQL
        ↓
Load relasi (user, category)
        ↓
Mail::to(user->email)->send(new TaskCreatedMail($task))
        ↓
SMTP ke sandbox.smtp.mailtrap.io
        ↓
Email tampil di inbox Mailtrap
```

---

## Apa yang Sudah Kamu Kuasai

- Konfigurasi SMTP Laravel via `.env`
- Testing email aman dengan Mailtrap
- Membuat Mailable + Blade email HTML
- Memicu email dari layer service/controller
- (Opsional) Notification multi-channel
- Dasar perbedaan sync vs queue

---

## Modul Selanjutnya

Di Modul 16 kamu akan **deploy** Task Manager ke production (shared hosting / VPS), termasuk checklist keamanan `.env` dan optimasi cache.

---

**Modul sebelumnya:** [14 — Testing](../14-testing/README.md)  
**Modul berikutnya:** [16 — Deploy](../16-deploy/README.md)  
**Kembali ke:** [Panduan Utama](../README.md)
