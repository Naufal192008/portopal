# 📋 LAPORAN PRAKTIK DEMONSTRASI USK
## Aplikasi Restoran POS — Laravel 11

**Nama:** [Isi Nama Kamu]  
**Kelas:** [Isi Kelas]  
**Tanggal:** [Isi Tanggal]  
**Penguji:** Pak Rian / Bu Eva  

---

## 🎯 A. SKENARIO PRAKTIK

Skenario praktik demonstrasi ini berisi alur kerja saat mengerjakan aplikasi Restoran POS dengan Laravel. Alur ini **bukan tugas terpisah**, melainkan metodologi yang harus ditunjukkan saat demo:

1. Mengidentifikasi Bug
2. Menemukan Lokasi Error
3. Menganalisis Error
4. Membuktikan Analisis Error
5. Melakukan pada semua error
6. Memperbaiki dan Validasi Script

**Tambahan:** Aplikasi harus ditambah **role Owner** (role baru dengan akses read-only).

---

## 🎯 B. TUJUAN

1. Membangun aplikasi Restoran POS berbasis Laravel 11 dari nol
2. Menerapkan metodologi debugging 6 langkah secara sistematis
3. Menambahkan role **owner** dengan hak akses terbatas (hanya dashboard & laporan)
4. Memvalidasi bahwa aplikasi berjalan normal setelah semua perbaikan

---

## 🎯 C. STRUKTUR APLIKASI

### Struktur Folder Utama
```
restoran-pos/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── AuthController.php
│   │   │   ├── DashboardController.php
│   │   │   ├── MenuController.php
│   │   │   ├── TableController.php
│   │   │   ├── OrderController.php
│   │   │   ├── TransactionController.php
│   │   │   └── ReportController.php
│   │   └── Middleware/
│   │       └── RoleMiddleware.php
│   └── Models/
│       ├── User.php
│       ├── MenuItem.php
│       ├── Table.php
│       ├── Order.php
│       ├── OrderItem.php
│       └── Transaction.php
├── database/
│   ├── migrations/
│   └── seeders/
├── resources/views/
│   ├── layouts/app.blade.php
│   ├── auth/login.blade.php
│   ├── dashboard.blade.php
│   ├── menu/
│   ├── tables/
│   ├── orders/
│   ├── transactions/
│   └── reports/
└── routes/web.php
```

### Matriks Hak Akses Role

| Fitur | Owner | Admin | Waiter | Kasir |
|-------|:-----:|:-----:|:------:|:-----:|
| Login | ✅ | ✅ | ✅ | ✅ |
| Dashboard | ✅ | ✅ | ✅ | ✅ |
| Kelola Meja | ❌ | ✅ | ❌ | ❌ |
| Kelola Menu | ❌ | ✅ | ✅ | ❌ |
| Buat Order | ❌ | ✅ | ✅ | ❌ |
| Transaksi/Bayar | ❌ | ✅ | ❌ | ✅ |
| Laporan | ✅ | ✅ | ❌ | ✅ |

### Akun Login (Seeder)

| Role | Username | Password |
|------|----------|----------|
| Owner | `owner` | `owner123` |
| Admin | `admin` | `admin123` |
| Waiter | `waiter1` | `waiter123` |
| Kasir | `kasir1` | `kasir123` |

---

## 🐛 D. ANALISIS ERROR (Metodologi 6 Langkah)

Setiap error di bawah dianalisis dengan 6 langkah:
1. Identifikasi Bug
2. Lokasi Error
3. Analisis
4. Pembuktian
5. Perbaikan
6. Validasi

---

### 🔴 ERROR #1: Koneksi Database Gagal

#### 1. Identifikasi Bug
Saat menjalankan `php artisan migrate`, muncul pesan:
```
SQLSTATE[HY000] [1049] Unknown database 'restaurant_pos'
```

#### 2. Lokasi Error
- File: `.env` pada baris `DB_DATABASE=restaurant_pos`
- File: `config/database.php`
- Error muncul saat perintah: `php artisan migrate`

#### 3. Analisis
Laravel mencoba terhubung ke database `restaurant_pos`, tetapi database tersebut belum dibuat di MySQL. Laravel **tidak bisa** membuat database secara otomatis — database harus dibuat manual terlebih dahulu.

#### 4. Pembuktian
Jalankan perintah di CMD:
```cmd
mysql -u root -e "SHOW DATABASES;"
```
**Hasil:** `restaurant_pos` **tidak muncul** dalam daftar → analisis terbukti benar.

#### 5. Perbaikan
```cmd
mysql -u root -e "CREATE DATABASE restaurant_pos;"
```

#### 6. Validasi
```cmd
php artisan migrate
```
**Hasil:** ✅ Migration berhasil, tabel-tabel dibuat.

---

### 🔴 ERROR #2: Class Controller Not Found

#### 1. Identifikasi Bug
Akses halaman `/login`, muncul:
```
Class "App\Http\Controllers\AuthController" not found
```

#### 2. Lokasi Error
- File: `routes/web.php`
- File target: `app/Http/Controllers/AuthController.php`

#### 3. Analisis
Kemungkinan penyebab:
- File `AuthController.php` belum dibuat
- Namespace tidak sesuai (`App\Http\Controllers`)
- Nama file tidak match dengan nama class (case-sensitive di Linux)

#### 4. Pembuktian
```cmd
dir app\Http\Controllers\
```
**Hasil:** File `AuthController.php` **tidak ada** → analisis benar.

#### 5. Perbaikan
Buat file `app/Http/Controllers/AuthController.php` dengan namespace benar:
```php
<?php
namespace App\Http\Controllers;
class AuthController extends Controller { /* ... */ }
```

#### 6. Validasi
```cmd
php artisan route:list
```
**Hasil:** ✅ Route muncul tanpa error.

---

### 🔴 ERROR #3: Target Class Does Not Exist (Route)

#### 1. Identifikasi Bug
Akses halaman `/menu`, muncul:
```
Target class [App\Http\Controllers\MenuController] does not exist.
```

#### 2. Lokasi Error
- File: `routes/web.php` pada `use App\Http\Controllers\MenuController;`

#### 3. Analisis
Ada **typo** pada penulisan `use` statement, atau controller belum di-import. Laravel 11 memerlukan import eksplisit di `web.php`.

#### 4. Pembuktian
```cmd
php artisan route:list
```
**Hasil:** Error yang sama muncul → terbukti masalah di route definition.

#### 5. Perbaikan
Cek dan perbaiki baris atas `web.php`:
```php
use App\Http\Controllers\MenuController;  // pastikan benar
```
Lalu:
```cmd
php artisan route:clear
php artisan config:clear
```

#### 6. Validasi
```cmd
php artisan route:list
```
**Hasil:** ✅ Semua route terdaftar tanpa error.

---

### 🔴 ERROR #4: Method links() Does Not Exist

#### 1. Identifikasi Bug
Akses halaman `/menu`, muncul:
```
Method Illuminate\Database\Eloquent\Collection::links does not exist.
```

#### 2. Lokasi Error
- File: `resources/views/menu/index.blade.php` baris `{{ $items->links() }}`
- File: `app/Http/Controllers/MenuController.php`

#### 3. Analisis
Method `->links()` hanya bisa dipanggil pada hasil `paginate()`, **bukan** pada `get()` atau `all()`. Controller menggunakan `MenuItem::all()`, sehingga variabel `$items` adalah `Collection`, bukan `LengthAwarePaginator`.

#### 4. Pembuktian
Cek controller:
```php
$items = MenuItem::all();  // ❌ Collection tidak punya ->links()
```

#### 5. Perbaikan
Ubah di controller:
```php
$items = MenuItem::paginate(10);  // ✅
```

#### 6. Validasi
Refresh halaman → **Hasil:** ✅ Pagination muncul, tidak error.

---

### 🔴 ERROR #5: Table Not Found

#### 1. Identifikasi Bug
Akses halaman `/orders`, muncul:
```
SQLSTATE[42S02]: Base table or view not found: 1146 Table 'restaurant_pos.orders' doesn't exist
```

#### 2. Lokasi Error
- File: `database/migrations/2024_01_01_000004_create_orders_table.php`
- Database `restaurant_pos`

#### 3. Analisis
Migration untuk tabel `orders` belum dijalankan, atau nama tabel di model tidak sesuai dengan migration (singular vs plural).

#### 4. Pembuktian
```cmd
php artisan migrate:status
```
**Hasil:** Migration `create_orders_table` statusnya **Pending** → terbukti.

#### 5. Perbaikan
```cmd
php artisan migrate
```
Jika nama tabel model tidak sesuai, cek di `Order.php`:
```php
protected $table = 'orders'; // pastikan benar
```

#### 6. Validasi
```cmd
php artisan migrate:status
```
**Hasil:** ✅ Semua migration status "Ran".

---

### 🔴 ERROR #6: Call to a Member Function on Null

#### 1. Identifikasi Bug
Akses halaman `/orders/{id}`, muncul:
```
Call to a member function on null
```

#### 2. Lokasi Error
- File: `resources/views/orders/show.blade.php` baris `{{ $order->user->name }}`

#### 3. Analisis
Relasi `$order->user` menghasilkan `null` karena `user_id` tidak valid atau user sudah dihapus. Access langsung ke `->name` pada `null` menyebabkan error.

#### 4. Pembuktian
Tambahkan debug:
```php
dd($order->user);
```
**Hasil:** `null` → terbukti relasi kosong.

#### 5. Perbaikan
Gunakan **null-safe operator**:
```blade
{{ $order->user?->name ?? '-' }}
```
Atau perbaiki data foreign key di database.

#### 6. Validasi
Refresh halaman → **Hasil:** ✅ Tampil "-" jika null, tidak error.

---

### 🔴 ERROR #7: 419 Page Expired (CSRF Token)

#### 1. Identifikasi Bug
Submit form login, muncul:
```
419 | Page Expired
```

#### 2. Lokasi Error
- File: `resources/views/auth/login.blade.php`
- Middleware: `VerifyCsrfToken`

#### 3. Analisis
Form POST tidak menyertakan CSRF token. Laravel menolak request POST tanpa token valid.

#### 4. Pembuktian
Cek kode form → tidak ada `@csrf` di dalam `<form>`.

#### 5. Perbaikan
Tambahkan di dalam `<form method="POST">`:
```blade
@csrf
```

#### 6. Validasi
Submit form → **Hasil:** ✅ Login berhasil.

---

### 🔴 ERROR #8: Route [login] Not Defined

#### 1. Identifikasi Bug
Akses halaman yang butuh login tanpa login dulu, muncul:
```
Route [login] not defined.
```

#### 2. Lokasi Error
- File: `routes/web.php`
- Middleware: `auth` di `bootstrap/app.php`

#### 3. Analisis
Middleware `auth` mencoba redirect ke route bernama `login`, tapi route tersebut belum didefinisikan dengan `->name('login')`.

#### 4. Pembuktian
```cmd
php artisan route:list | findstr login
```
**Hasil:** Kosong → terbukti route belum ada.

#### 5. Perbaikan
Di `routes/web.php`:
```php
Route::get('/login', [AuthController::class, 'index'])->name('login');
```

#### 6. Validasi
```cmd
php artisan route:list
```
**Hasil:** ✅ Route `login` muncul.

---

### 🔴 ERROR #9: Role Owner Tidak Bisa Login

#### 1. Identifikasi Bug
Login dengan `owner / owner123` → gagal, muncul "Username atau password salah".

#### 2. Lokasi Error
- File: `database/migrations/2024_01_01_000001_create_users_table.php`
- File: `database/seeders/UserSeeder.php`

#### 3. Analisis
Enum kolom `role` di tabel users hanya berisi `['administrator', 'waiter', 'kasir']`. Role `owner` tidak ada di enum, sehingga insert gagal atau user owner tidak tercipta.

#### 4. Pembuktian
```sql
SHOW COLUMNS FROM users LIKE 'role';
```
**Hasil:** Enum tidak ada 'owner' → terbukti.

#### 5. Perbaikan
Buat migration baru:
```cmd
php artisan make:migration add_owner_to_users_role
```
Isi file:
```php
public function up(): void
{
    DB::statement("ALTER TABLE users MODIFY COLUMN role ENUM('owner', 'administrator', 'waiter', 'kasir') DEFAULT 'waiter'");
}
public function down(): void
{
    DB::statement("ALTER TABLE users MODIFY COLUMN role ENUM('administrator', 'waiter', 'kasir') DEFAULT 'waiter'");
}
```
Update seeder:
```php
User::create([
    'name' => 'Owner Resto',
    'username' => 'owner',
    'password' => Hash::make('owner123'),
    'role' => 'owner',
]);
```
Jalankan:
```cmd
php artisan migrate && php artisan db:seed --class=UserSeeder
```

#### 6. Validasi
Login dengan `owner / owner123` → **Hasil:** ✅ Berhasil login sebagai Owner.

---

### 🔴 ERROR #10: Owner Bisa Akses Halaman Admin

#### 1. Identifikasi Bug
Setelah login sebagai owner, owner bisa buka `/menu`, `/tables`, `/orders`, `/transactions` — seharusnya tidak boleh.

#### 2. Lokasi Error
- File: `routes/web.php`
- Belum ada middleware role

#### 3. Analisis
Route hanya dilindungi middleware `auth` (cek login), bukan middleware `role` (cek role). Siapapun yang login bisa akses semua halaman.

#### 4. Pembuktian
Login sebagai owner, buka `/menu` → bisa diakses → **terbukti** hak akses tidak dibatasi.

#### 5. Perbaikan
Buat middleware:
```cmd
php artisan make:middleware RoleMiddleware
```
Isi `app/Http/Middleware/RoleMiddleware.php`:
```php
<?php
namespace App\Http\Middleware;
use Closure;
use Illuminate\Http\Request;

class RoleMiddleware
{
    public function handle(Request $request, Closure $next, ...$roles)
    {
        if (!auth()->check()) {
            return redirect()->route('login');
        }
        if (!in_array(auth()->user()->role, $roles)) {
            abort(403, 'Akses ditolak. Role Anda tidak diizinkan.');
        }
        return $next($request);
    }
}
```
Daftarkan di `bootstrap/app.php`:
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'role' => \App\Http\Middleware\RoleMiddleware::class,
    ]);
})
```
Update `routes/web.php`:
```php
Route::middleware('auth')->group(function () {
    Route::get('/', [DashboardController::class, 'index'])->name('dashboard');

    Route::middleware('role:administrator')->group(function () {
        Route::resource('tables', TableController::class);
    });

    Route::middleware('role:administrator,waiter')->group(function () {
        Route::resource('menu', MenuController::class);
        Route::resource('orders', OrderController::class);
        Route::post('orders/{order}/add-item', [OrderController::class, 'addItem'])->name('orders.addItem');
    });

    Route::middleware('role:administrator,kasir')->group(function () {
        Route::get('transactions', [TransactionController::class, 'index'])->name('transactions.index');
        Route::post('transactions', [TransactionController::class, 'store'])->name('transactions.store');
    });

    Route::middleware('role:owner,administrator,kasir')->group(function () {
        Route::get('reports/daily', [ReportController::class, 'daily'])->name('reports.daily');
        Route::get('reports/best-selling', [ReportController::class, 'bestSelling'])->name('reports.best_selling');
    });
});
```

#### 6. Validasi
Login sebagai owner, buka `/menu` → **Hasil:** ✅ Muncul "403 Akses ditolak".

---

### 🔴 ERROR #11: Sidebar Owner Menampilkan Menu Admin

#### 1. Identifikasi Bug
Sidebar owner masih menampilkan "Kelola Menu", "Kelola Meja", "Buat Order", "Transaksi" — seharusnya owner hanya lihat Dashboard & Laporan.

#### 2. Lokasi Error
- File: `resources/views/layouts/app.blade.php`

#### 3. Analisis
Kondisi `@if` pada sidebar belum mem-filter role owner, hanya membedakan administrator, waiter, kasir.

#### 4. Pembuktian
Login owner → lihat sidebar → masih ada menu yang tidak seharusnya.

#### 5. Perbaikan
Update `app.blade.php`:
```blade
{{-- Dashboard: semua role --}}
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('dashboard') ? 'active' : '' }}" href="{{ route('dashboard') }}">
        <i class="fas fa-home"></i> Dashboard
    </a>
</li>

{{-- Kelola Meja: hanya admin --}}
@if(auth()->user()->role == 'administrator')
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('tables.*') ? 'active' : '' }}" href="{{ route('tables.index') }}">
        <i class="fas fa-chair"></i> Kelola Meja
    </a>
</li>
@endif

{{-- Kelola Menu: admin & waiter --}}
@if(in_array(auth()->user()->role, ['administrator', 'waiter']))
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('menu.*') ? 'active' : '' }}" href="{{ route('menu.index') }}">
        <i class="fas fa-pizza-slice"></i> Kelola Menu
    </a>
</li>
@endif

{{-- Order: waiter & admin --}}
@if(in_array(auth()->user()->role, ['waiter', 'administrator']))
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('orders.*') ? 'active' : '' }}" href="{{ route('orders.index') }}">
        <i class="fas fa-clipboard-list"></i> Buat Order
    </a>
</li>
@endif

{{-- Transaksi: kasir & admin --}}
@if(in_array(auth()->user()->role, ['kasir', 'administrator']))
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('transactions.*') ? 'active' : '' }}" href="{{ route('transactions.index') }}">
        <i class="fas fa-cash-register"></i> Transaksi
    </a>
</li>
@endif

{{-- Laporan: owner, kasir, admin (BUKAN waiter) --}}
@if(in_array(auth()->user()->role, ['owner', 'kasir', 'administrator']))
<li class="nav-item">
    <a class="nav-link {{ request()->routeIs('reports.*') ? 'active' : '' }}" href="{{ route('reports.daily') }}">
        <i class="fas fa-chart-bar"></i> Laporan
    </a>
</li>
@endif
```

Update juga badge role:
```blade
<span class="badge bg-{{ 
    auth()->user()->role == 'owner' ? 'dark' : 
    (auth()->user()->role == 'administrator' ? 'danger' : 
    (auth()->user()->role == 'waiter' ? 'success' : 'warning')) 
}}">
    {{ ucfirst(auth()->user()->role ?? '') }}
</span>
```

#### 6. Validasi
Login owner → sidebar hanya tampil Dashboard & Laporan → **Hasil:** ✅ sesuai matriks akses.

---

### 🔴 ERROR #12: Login View Belum Ada Akun Owner

#### 1. Identifikasi Bug
Halaman login tidak menampilkan demo akun owner.

#### 2. Lokasi Error
- File: `resources/views/auth/login.blade.php`

#### 3. Analisis
Bagian `.demo-accounts` hanya menampilkan admin, waiter, kasir. Owner belum ditambahkan.

#### 4. Pembuktian
Buka halaman login → hanya 3 akun terlihat.

#### 5. Perbaikan
Update `login.blade.php`:
```blade
<div class="demo-accounts">
    <small><strong>📋 Demo Account</strong></small>
    <small>👑 Owner: <code>owner / owner123</code></small>
    <small>🛠️ Admin: <code>admin / admin123</code></small>
    <small>👨‍🍳 Waiter: <code>waiter1 / waiter123</code></small>
    <small>💰 Kasir: <code>kasir1 / kasir123</code></small>
</div>
```

#### 6. Validasi
Refresh halaman login → **Hasil:** ✅ Akun owner muncul.

---

### 🔴 ERROR #13: Order Total Tidak Ter-Update Setelah Tambah Item

#### 1. Identifikasi Bug
Setelah menambahkan item ke order, kolom `total` di tabel `orders` tetap 0.

#### 2. Lokasi Error
- File: `app/Http/Controllers/OrderController.php` method `addItem`

#### 3. Analisis
Setelah insert `OrderItem`, tidak ada proses update agregat `subtotal`, `tax`, dan `total` di tabel `orders`.

#### 4. Pembuktian
Tambah item → cek tabel `orders`:
```sql
SELECT id, subtotal, tax, total FROM orders;
```
**Hasil:** Nilai masih 0 → terbukti.

#### 5. Perbaikan
Update di `addItem`:
```php
$subtotal = $order->items()->sum('subtotal');
$tax = $subtotal * 0.1;
$order->update(['subtotal' => $subtotal, 'tax' => $tax, 'total' => $subtotal + $tax]);
```

#### 6. Validasi
Tambah item lagi → cek total → **Hasil:** ✅ total ter-update sesuai.

---

### 🔴 ERROR #14: Pembayaran Melebihi Total Order Tidak Ditolak

#### 1. Identifikasi Bug
Kasir bisa input jumlah bayar kurang dari total dan transaksi tetap tersimpan dengan kembalian negatif.

#### 2. Lokasi Error
- File: `app/Http/Controllers/TransactionController.php`

#### 3. Analisis
Tidak ada validasi `amount_paid >= order->total` sebelum insert transaksi.

#### 4. Pembuktian
Input bayar Rp 1000 untuk order Rp 50000 → transaksi tersimpan dengan kembalian Rp -49000 → terbukti.

#### 5. Perbaikan
Tambahkan validasi:
```php
if ($request->amount_paid < $order->total) {
    return back()->with('error', 'Jumlah bayar kurang dari total!');
}
```

#### 6. Validasi
Coba input bayar kurang → **Hasil:** ✅ Muncul pesan error, transaksi tidak tersimpan.

---

### 🔴 ERROR #15: Meja Tidak Kembali ke Status "Available" Setelah Bayar

#### 1. Identifikasi Bug
Setelah transaksi selesai, meja masih berstatus "occupied".

#### 2. Lokasi Error
- File: `app/Http/Controllers/TransactionController.php`

#### 3. Analisis
Setelah insert transaksi dan update status order ke `paid`, tidak ada update status meja kembali ke `available`.

#### 4. Pembuktian
Selesaikan transaksi → cek tabel `tables`:
```sql
SELECT table_number, status FROM tables;
```
**Hasil:** Meja masih `occupied` → terbukti.

#### 5. Perbaikan
Tambahkan:
```php
Table::where('id', $order->table_id)->update(['status' => 'available']);
```

#### 6. Validasi
Coba transaksi lain → **Hasil:** ✅ Meja kembali available.

---

## 🛠️ E. PERBAIKAN & VALIDASI SCRIPT

### E.1 Perbaikan File yang Terkena Dampak Role Owner

#### File 1: `database/migrations/2024_01_01_000001_create_users_table.php`
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('username')->unique();
            $table->string('password');
            $table->enum('role', ['owner', 'administrator', 'waiter', 'kasir'])->default('waiter');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

#### File 2: `database/seeders/UserSeeder.php`
```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::create([
            'name' => 'Owner Resto',
            'username' => 'owner',
            'password' => Hash::make('owner123'),
            'role' => 'owner',
        ]);

        User::create([
            'name' => 'Administrator',
            'username' => 'admin',
            'password' => Hash::make('admin123'),
            'role' => 'administrator',
        ]);

        User::create([
            'name' => 'Budi Waiter',
            'username' => 'waiter1',
            'password' => Hash::make('waiter123'),
            'role' => 'waiter',
        ]);

        User::create([
            'name' => 'Ani Waiter',
            'username' => 'waiter2',
            'password' => Hash::make('waiter123'),
            'role' => 'waiter',
        ]);

        User::create([
            'name' => 'Citra Kasir',
            'username' => 'kasir1',
            'password' => Hash::make('kasir123'),
            'role' => 'kasir',
        ]);
    }
}
```

#### File 3: `app/Http/Middleware/RoleMiddleware.php`
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class RoleMiddleware
{
    public function handle(Request $request, Closure $next, ...$roles)
    {
        if (!auth()->check()) {
            return redirect()->route('login');
        }

        if (!in_array(auth()->user()->role, $roles)) {
            abort(403, 'Akses ditolak. Role Anda tidak diizinkan.');
        }

        return $next($request);
    }
}
```

#### File 4: `bootstrap/app.php`
```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => \App\Http\Middleware\RoleMiddleware::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

#### File 5: `routes/web.php`
```php
<?php

use App\Http\Controllers\AuthController;
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\MenuController;
use App\Http\Controllers\OrderController;
use App\Http\Controllers\ReportController;
use App\Http\Controllers\TableController;
use App\Http\Controllers\TransactionController;
use Illuminate\Support\Facades\Route;

// Auth
Route::get('/login', [AuthController::class, 'index'])->name('login');
Route::post('/login', [AuthController::class, 'login']);
Route::post('/logout', [AuthController::class, 'logout'])->name('logout');

// Protected
Route::middleware('auth')->group(function () {
    Route::get('/', [DashboardController::class, 'index'])->name('dashboard');

    Route::middleware('role:administrator')->group(function () {
        Route::resource('tables', TableController::class);
    });

    Route::middleware('role:administrator,waiter')->group(function () {
        Route::resource('menu', MenuController::class);
        Route::resource('orders', OrderController::class);
        Route::post('orders/{order}/add-item', [OrderController::class, 'addItem'])->name('orders.addItem');
    });

    Route::middleware('role:administrator,kasir')->group(function () {
        Route::get('transactions', [TransactionController::class, 'index'])->name('transactions.index');
        Route::post('transactions', [TransactionController::class, 'store'])->name('transactions.store');
    });

    Route::middleware('role:owner,administrator,kasir')->group(function () {
        Route::get('reports/daily', [ReportController::class, 'daily'])->name('reports.daily');
        Route::get('reports/best-selling', [ReportController::class, 'bestSelling'])->name('reports.best_selling');
    });
});
```

#### File 6: `resources/views/layouts/app.blade.php` (bagian sidebar)
```blade
<ul class="nav flex-column p-2">
    <li class="nav-item">
        <a class="nav-link {{ request()->routeIs('dashboard') ? 'active' : '' }}" href="{{ route('dashboard') }}">
            <i class="fas fa-home"></i> Dashboard
        </a>
    </li>

    @if(auth()->user()->role == 'administrator')
    <li class="nav-item">
        <a class="nav-link {{ request()->routeIs('tables.*') ? 'active' : '' }}" href="{{ route('tables.index') }}">
            <i class="fas fa-chair"></i> Kelola Meja
        </a>
    </li>
    @endif

    @if(in_array(auth()->user()->role, ['administrator', 'waiter']))
    <li class="nav-item">
        <a class="nav-link {{ request()->routeIs('menu.*') ? 'active' : '' }}" href="{{ route('menu.index') }}">
            <i class="fas fa-pizza-slice"></i> Kelola Menu
        </a>
    </li>
    @endif

    @if(in_array(auth()->user()->role, ['waiter', 'administrator']))
    <li class="nav-item">
        <a class="nav-link {{ request()->routeIs('orders.*') ? 'active' : '' }}" href="{{ route('orders.index') }}">
            <i class="fas fa-clipboard-list"></i> Buat Order
        </a>
    </li>
    @endif

    @if(in_array(auth()->user()->role, ['kasir', 'administrator']))
    <li class="nav-item">
        <a class="nav-link {{ request()->routeIs('transactions.*') ? 'active' : '' }}" href="{{ route('transactions.index') }}">
            <i class="fas fa-cash-register"></i> Transaksi
        </a>
    </li>
    @endif

    @if(in_array(auth()->user()->role, ['owner', 'kasir', 'administrator']))
    <li class="nav-item">
        <a class="nav-link {{ request()->routeIs('reports.*') ? 'active' : '' }}" href="{{ route('reports.daily') }}">
            <i class="fas fa-chart-bar"></i> Laporan
        </a>
    </li>
    @endif

    <hr class="border-light">
    <li class="nav-item">
        <form method="POST" action="{{ route('logout') }}">
            @csrf
            <button type="submit" class="nav-link text-danger bg-transparent border-0 w-100 text-start">
                <i class="fas fa-sign-out-alt"></i> Logout
            </button>
        </form>
    </li>
</ul>
```

#### File 7: `resources/views/auth/login.blade.php` (bagian demo)
```blade
<div class="demo-accounts">
    <small><strong>📋 Demo Account</strong></small>
    <small>👑 Owner: <code>owner / owner123</code></small>
    <small>🛠️ Admin: <code>admin / admin123</code></small>
    <small>👨‍🍳 Waiter: <code>waiter1 / waiter123</code></small>
    <small>💰 Kasir: <code>kasir1 / kasir123</code></small>
</div>
```

---

### E.2 Perintah Setup & Validasi Final

```cmd
cd /d C:\xampp\htdocs\restoran-pos
php artisan migrate:fresh --seed
php artisan route:clear
php artisan config:clear
php artisan cache:clear
php artisan serve
```

---

## ✅ F. VALIDASI AKHIR

### F.1 Validasi Role & Hak Akses

| Test | Akun | Aksi | Expected | Hasil |
|------|------|------|----------|:-----:|
| 1 | owner | Login | Masuk dashboard | ✅ |
| 2 | owner | Buka `/menu` | 403 Akses ditolak | ✅ |
| 3 | owner | Buka `/reports/daily` | Bisa akses | ✅ |
| 4 | admin | Buka `/tables` | Bisa akses | ✅ |
| 5 | waiter | Buka `/orders` | Bisa akses | ✅ |
| 6 | kasir | Buka `/transactions` | Bisa akses | ✅ |
| 7 | kasir | Buka `/menu` | 403 Akses ditolak | ✅ |
| 8 | waiter | Buka `/reports/daily` | 403 Akses ditolak | ✅ |

### F.2 Validasi Fungsional

| Test | Aksi | Expected | Hasil |
|------|------|----------|:-----:|
| 1 | Tambah menu | Data tersimpan | ✅ |
| 2 | Edit menu | Data terupdate | ✅ |
| 3 | Hapus menu | Data terhapus | ✅ |
| 4 | Buat order baru | Order tercipta | ✅ |
| 5 | Tambah item ke order | Item + total terupdate | ✅ |
| 6 | Proses pembayaran | Transaksi tersimpan | ✅ |
| 7 | Bayar kurang dari total | Ditolak, muncul error | ✅ |
| 8 | Setelah bayar | Meja kembali available | ✅ |
| 9 | Laporan harian | Data sesuai tanggal | ✅ |
| 10 | Menu terlaris | Top 10 tampil | ✅ |

### F.3 Hasil Akhir

✅ **Semua error berhasil diidentifikasi, dianalisis, dibuktikan, diperbaiki, dan divalidasi.**  
✅ **Role Owner berhasil ditambahkan dengan hak akses read-only (Dashboard & Laporan).**  
✅ **Aplikasi berjalan normal tanpa error.**

---

## 📝 G. KESIMPULAN

1. Metodologi 6 langkah debugging (identifikasi → lokasi → analisis → bukti → perbaiki → validasi) terbukti efektif untuk menyelesaikan error secara sistematis.
2. Total **15 error** berhasil dianalisis dan diperbaiki, mulai dari error koneksi database, route, view, hingga error logic pada controller.
3. Penambahan **role Owner** memerlukan modifikasi **7 file** utama (migration, seeder, middleware, routes, layout, view login, bootstrap/app.php).
4. Validasi akhir menunjukkan semua fitur berjalan normal sesuai matriks hak akses yang ditentukan.

---

## 📎 H. LAMPIRAN

### Screenshot Bukti
- [ ] Screenshot error sebelum perbaikan
- [ ] Screenshot error message (stack trace)
- [ ] Screenshot setelah perbaikan (berhasil)
- [ ] Screenshot dashboard per role

### Tools Debugging yang Digunakan
- `php artisan route:list` — cek daftar route
- `php artisan migrate:status` — cek status migration
- `php artisan tinker` — debug interaktif
- `dd()` dan `dump()` — debug variabel
- `laravel.log` di `storage/logs/` — log error detail
- Chrome DevTools → Network & Console

### Referensi
- Laravel 11 Documentation: https://laravel.com/docs/11.x
- Bootstrap 5: https://getbootstrap.com/docs/5.3/
- MySQL Documentation: https://dev.mysql.com/doc/

---

**Laporan ini disusun sebagai bukti pelaksanaan Praktik Demonstrasi USK.**  
**Tanggal:** [isi tanggal]  
**Tanda Tangan:** _______________