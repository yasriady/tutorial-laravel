# Tutorial: Pengenalan Request Lifecycle di Laravel

## Pengantar

Request Lifecycle (Siklus Permintaan) dalam Laravel adalah rangkaian proses yang terjadi ketika aplikasi menerima permintaan HTTP dari pengguna hingga mengirimkan respons kembali. Memahami siklus ini akan membantu Anda mengetahui bagaimana Laravel menangani permintaan dan di mana Anda dapat melakukan intervensi dalam siklus tersebut.

## Langkah-langkah Siklus Permintaan Laravel

### 1. Permintaan HTTP Masuk

Setiap interaksi dengan aplikasi Laravel dimulai ketika pengguna mengirimkan permintaan HTTP ke server Anda (misalnya ketika membuka halaman atau mengirim formulir).

```
https://aplikasi-anda.com/mahasiswa
```

### 2. Entry Point: index.php

Semua permintaan masuk ke file `public/index.php`, yang berfungsi sebagai entry point aplikasi Laravel Anda.

```php
// public/index.php
<?php

// Memuat autoloader Composer
require __DIR__.'/../vendor/autoload.php';

// Bootstrap aplikasi Laravel
$app = require_once __DIR__.'/../bootstrap/app.php';

// Mendapatkan kernel HTTP
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

// Menjalankan aplikasi dan mendapatkan respons
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

// Mengirim respons ke browser
$response->send();

// Menjalankan tugas-tugas akhir
$kernel->terminate($request, $response);
```

### 3. Bootstrap Aplikasi

Laravel memuat file `bootstrap/app.php` yang menciptakan instance aplikasi Laravel dan mendaftarkan komponen-komponen utama seperti:
- Service Container
- Service Providers
- Facades

### 4. HTTP Kernel

Permintaan HTTP masuk diproses oleh HTTP Kernel (`app/Http/Kernel.php`), yang menjalankan middleware global sebelum permintaan diproses lebih lanjut.

Middleware global biasanya mencakup:
- Pengecekan CSRF token
- Session handling
- Enkripsi cookies
- dll.

### 5. Routing

Setelah melewati middleware global, Laravel mencocokkan URL permintaan dengan rute yang didefinisikan dalam file rute Anda (`routes/web.php` atau `routes/api.php`).

```php
// routes/web.php
Route::get('/mahasiswa', [MahasiswaController::class, 'index']);
```

### 6. Middleware Rute

Jika rute cocok ditemukan, Laravel menjalankan middleware khusus yang terkait dengan rute tersebut.

```php
Route::get('/mahasiswa', [MahasiswaController::class, 'index'])
    ->middleware('auth');  // Middleware yang memastikan pengguna sudah login
```

### 7. Controller/Closure

Setelah melewati middleware rute, Laravel menjalankan controller atau closure yang ditentukan pada rute tersebut.

```php
// app/Http/Controllers/MahasiswaController.php
public function index()
{
    $mahasiswa = Mahasiswa::all();
    return view('mahasiswa.index', compact('mahasiswa'));
}
```

### 8. Model dan Database (Opsional)

Controller biasanya berinteraksi dengan model untuk mengakses database.

```php
$mahasiswa = Mahasiswa::all();  // Mengambil semua data mahasiswa dari database
```

### 9. View (Opsional)

Controller biasanya mengembalikan view yang berisi respons HTML untuk browser.

```php
return view('mahasiswa.index', compact('mahasiswa'));
```

### 10. Response

Laravel mengubah hasil dari controller menjadi objek Response, yang kemudian diproses melalui middleware output.

### 11. Middleware Output

Response melalui middleware dalam urutan terbalik, memungkinkan transformasi response sebelum dikirim ke browser.

### 12. Pengiriman Response

Akhirnya, response dikirim kembali ke browser pengguna.

## Visualisasi Siklus Permintaan

```
Request -> index.php -> Bootstrap -> HTTP Kernel -> Middleware Global -> 
Routing -> Middleware Rute -> Controller -> Model -> View -> Response -> 
Middleware Output -> Browser
```

## Contoh Praktis

Misalkan pengguna mengakses halaman daftar mahasiswa:

1. Pengguna mengakses URL `http://aplikasi-anda.com/mahasiswa`
2. Permintaan diarahkan ke `public/index.php`
3. Laravel bootstrap aplikasi
4. HTTP Kernel memproses middleware global
5. Router mencocokkan URL dengan rute `/mahasiswa`
6. Middleware `auth` memastikan pengguna sudah login
7. `MahasiswaController@index` dijalankan
8. Model `Mahasiswa` mengambil data dari database
9. View `mahasiswa.index` dirender dengan data
10. Response HTML dihasilkan
11. Response diproses melalui middleware output
12. HTML dikirim ke browser pengguna

## Bagaimana Anda Bisa Memanfaatkan Siklus Ini?

1. **Middleware**: Tambahkan logika yang perlu dijalankan sebelum atau sesudah permintaan diproses
2. **Service Providers**: Daftarkan layanan yang diperlukan aplikasi
3. **Event Listeners**: Tangani peristiwa tertentu selama siklus permintaan
4. **Middleware Terminable**: Jalankan kode setelah respons dikirim ke browser

## Latihan Praktik

Buatlah middleware sederhana yang mencatat semua permintaan ke aplikasi Anda:

1. Buat middleware baru menggunakan perintah Artisan:
```bash
php artisan make:middleware LogRequests
```

2. Edit file `app/Http/Middleware/LogRequests.php`:
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Log;

class LogRequests
{
    public function handle($request, Closure $next)
    {
        // Mencatat informasi permintaan
        Log::info('Request URL: ' . $request->fullUrl());
        
        return $next($request);
    }
}
```

3. Daftarkan middleware di `app/Http/Kernel.php` dalam array `$middleware`:
```php
protected $middleware = [
    // Middleware lainnya...
    \App\Http\Middleware\LogRequests::class,
];
```

Sekarang setiap permintaan ke aplikasi Anda akan dicatat dalam log, memberi Anda informasi berharga tentang traffic aplikasi.

## Kesimpulan

Memahami Request Lifecycle Laravel sangat penting bagi pengembang untuk:
- Melakukan intervensi tepat pada titik yang tepat dalam siklus
- Mengoptimalkan performa aplikasi
- Memecahkan masalah dengan lebih efektif
- Mengembangkan fitur yang lebih canggih

Dengan pengetahuan ini, Anda dapat mengembangkan aplikasi Laravel yang lebih baik dan efisien.