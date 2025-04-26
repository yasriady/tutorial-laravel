# Tutorial: Membuat Halaman Home Sederhana di Laravel

Halaman home adalah halaman utama yang akan dilihat pengguna saat pertama kali mengakses aplikasi web Anda. Di tutorial ini, kita akan membuat halaman home sederhana namun menarik untuk aplikasi manajemen data Mahasiswa menggunakan Laravel.

## Prasyarat
- Laravel sudah terinstal melalui Laragon
- Project `mahasiswa-app` sudah dibuat
- Sudah memahami dasar routing dan view di Laravel

## Langkah 1: Membuat Controller untuk Halaman Home

Pertama, mari buat controller khusus untuk menangani halaman home:

```
php artisan make:controller HomeController
```

Perintah di atas akan membuat file `HomeController.php` di direktori `app/Http/Controllers/`.

## Langkah 2: Menambahkan Method Index pada HomeController

Buka file `HomeController.php` menggunakan VS Code dan tambahkan method `index()`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class HomeController extends Controller
{
    /**
     * Menampilkan halaman home
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $appName = "Sistem Manajemen Data Mahasiswa";
        $features = [
            'Pengelolaan Data Mahasiswa',
            'Pengelolaan Data Jurusan',
            'Pencarian Data Mahasiswa',
            'Laporan Akademik'
        ];
        
        return view('home', compact('appName', 'features'));
    }
}
```

Controller ini akan mengirimkan data nama aplikasi dan daftar fitur ke view.

## Langkah 3: Membuat View untuk Halaman Home

Selanjutnya, buat file `home.blade.php` di direktori `resources/views/`:

```
touch resources/views/home.blade.php
```

## Langkah 4: Mengisi Konten View Home

Buka file `home.blade.php` dan isi dengan kode berikut:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $appName }}</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            color: #333;
            background-color: #f8f9fa;
        }
        .hero {
            background-color: #3490dc;
            color: white;
            padding: 60px 20px;
            text-align: center;
        }
        .hero h1 {
            font-size: 2.5rem;
            margin-bottom: 20px;
        }
        .hero p {
            font-size: 1.2rem;
            max-width: 600px;
            margin: 0 auto;
        }
        .container {
            max-width: 1100px;
            margin: 0 auto;
            padding: 40px 20px;
        }
        .features {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 30px;
            margin-top: 40px;
        }
        .feature-card {
            background-color: white;
            border-radius: 8px;
            padding: 25px;
            box-shadow: 0 3px 10px rgba(0,0,0,0.1);
            transition: transform 0.3s ease;
        }
        .feature-card:hover {
            transform: translateY(-5px);
        }
        .feature-card h3 {
            color: #3490dc;
            margin-top: 0;
        }
        .cta {
            text-align: center;
            margin-top: 60px;
        }
        .btn {
            display: inline-block;
            background-color: #3490dc;
            color: white;
            padding: 12px 25px;
            text-decoration: none;
            border-radius: 5px;
            font-weight: bold;
            transition: background-color 0.3s ease;
        }
        .btn:hover {
            background-color: #2779bd;
        }
        .footer {
            background-color: #343a40;
            color: white;
            text-align: center;
            padding: 20px;
            margin-top: 60px;
        }
        .nav {
            background-color: #fff;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            padding: 15px 0;
        }
        .nav-container {
            display: flex;
            justify-content: space-between;
            align-items: center;
            max-width: 1100px;
            margin: 0 auto;
            padding: 0 20px;
        }
        .logo {
            font-size: 1.5rem;
            font-weight: bold;
            color: #3490dc;
            text-decoration: none;
        }
        .nav-links a {
            color: #333;
            text-decoration: none;
            margin-left: 20px;
            transition: color 0.3s ease;
        }
        .nav-links a:hover {
            color: #3490dc;
        }
    </style>
</head>
<body>
    <nav class="nav">
        <div class="nav-container">
            <a href="/" class="logo">{{ $appName }}</a>
            <div class="nav-links">
                <a href="/">Home</a>
                <a href="/mahasiswa">Mahasiswa</a>
                <a href="/jurusan">Jurusan</a>
                <a href="/about">Tentang</a>
            </div>
        </div>
    </nav>

    <section class="hero">
        <h1>Selamat Datang di {{ $appName }}</h1>
        <p>Sistem informasi terpadu untuk pengelolaan data mahasiswa dengan mudah, cepat, dan efisien.</p>
    </section>

    <div class="container">
        <h2>Fitur Utama</h2>
        <div class="features">
            @foreach($features as $feature)
                <div class="feature-card">
                    <h3>{{ $feature }}</h3>
                    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam in dui mauris. Vivamus hendrerit arcu sed erat molestie vehicula.</p>
                </div>
            @endforeach
        </div>

        <div class="cta">
            <h2>Mulai Kelola Data Mahasiswa Sekarang</h2>
            <p>Klik tombol di bawah untuk melihat daftar mahasiswa</p>
            <a href="/mahasiswa" class="btn">Lihat Data Mahasiswa</a>
        </div>
    </div>

    <footer class="footer">
        <p>&copy; {{ date('Y') }} {{ $appName }}. All rights reserved.</p>
    </footer>
</body>
</html>
```

## Langkah 5: Menambahkan Route untuk Halaman Home

Buka file `routes/web.php` dan tambahkan route berikut:

```php
use App\Http\Controllers\HomeController;

// Route untuk halaman home
Route::get('/', [HomeController::class, 'index']);
```

## Langkah 6: Menguji Halaman Home

1. Pastikan server Laravel berjalan:
   ```
   php artisan serve
   ```
   Atau gunakan Laragon yang sudah berjalan

2. Buka browser dan akses:
   ```
   http://mahasiswa-app.test
   ```
   Atau jika menggunakan server lokal:
   ```
   http://localhost/mahasiswa-app/public
   ```

Anda akan melihat halaman home yang menarik dengan navigasi, banner hero, daftar fitur, dan call-to-action.

## Langkah 7: Menyempurnakan dengan Layout Master

Agar lebih mudah dikelola, mari pindahkan tampilan ke dalam layout master:

1. Buat direktori `layouts` di dalam `resources/views`:
   ```
   mkdir -p resources/views/layouts
   ```

2. Buat file `app.blade.php` di direktori tersebut:
   ```
   touch resources/views/layouts/app.blade.php
   ```

3. Isi `app.blade.php` dengan kode berikut:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Aplikasi Mahasiswa')</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            color: #333;
            background-color: #f8f9fa;
        }
        .hero {
            background-color: #3490dc;
            color: white;
            padding: 60px 20px;
            text-align: center;
        }
        .hero h1 {
            font-size: 2.5rem;
            margin-bottom: 20px;
        }
        .hero p {
            font-size: 1.2rem;
            max-width: 600px;
            margin: 0 auto;
        }
        .container {
            max-width: 1100px;
            margin: 0 auto;
            padding: 40px 20px;
        }
        .features {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 30px;
            margin-top: 40px;
        }
        .feature-card {
            background-color: white;
            border-radius: 8px;
            padding: 25px;
            box-shadow: 0 3px 10px rgba(0,0,0,0.1);
            transition: transform 0.3s ease;
        }
        .feature-card:hover {
            transform: translateY(-5px);
        }
        .feature-card h3 {
            color: #3490dc;
            margin-top: 0;
        }
        .cta {
            text-align: center;
            margin-top: 60px;
        }
        .btn {
            display: inline-block;
            background-color: #3490dc;
            color: white;
            padding: 12px 25px;
            text-decoration: none;
            border-radius: 5px;
            font-weight: bold;
            transition: background-color 0.3s ease;
        }
        .btn:hover {
            background-color: #2779bd;
        }
        .footer {
            background-color: #343a40;
            color: white;
            text-align: center;
            padding: 20px;
            margin-top: 60px;
        }
        .nav {
            background-color: #fff;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            padding: 15px 0;
        }
        .nav-container {
            display: flex;
            justify-content: space-between;
            align-items: center;
            max-width: 1100px;
            margin: 0 auto;
            padding: 0 20px;
        }
        .logo {
            font-size: 1.5rem;
            font-weight: bold;
            color: #3490dc;
            text-decoration: none;
        }
        .nav-links a {
            color: #333;
            text-decoration: none;
            margin-left: 20px;
            transition: color 0.3s ease;
        }
        .nav-links a:hover {
            color: #3490dc;
        }
        
        /* Tambahkan style tambahan disini */
        @yield('styles')
    </style>
</head>
<body>
    <nav class="nav">
        <div class="nav-container">
            <a href="/" class="logo">@yield('appName', 'Aplikasi Mahasiswa')</a>
            <div class="nav-links">
                <a href="/">Home</a>
                <a href="/mahasiswa">Mahasiswa</a>
                <a href="/jurusan">Jurusan</a>
                <a href="/about">Tentang</a>
            </div>
        </div>
    </nav>

    @yield('content')

    <footer class="footer">
        <p>&copy; {{ date('Y') }} @yield('appName', 'Aplikasi Mahasiswa'). All rights reserved.</p>
    </footer>
    
    @yield('scripts')
</body>
</html>
```

4. Update file `home.blade.php` untuk menggunakan layout:

```php
@extends('layouts.app')

@section('title', $appName)

@section('appName', $appName)

@section('content')
    <section class="hero">
        <h1>Selamat Datang di {{ $appName }}</h1>
        <p>Sistem informasi terpadu untuk pengelolaan data mahasiswa dengan mudah, cepat, dan efisien.</p>
    </section>

    <div class="container">
        <h2>Fitur Utama</h2>
        <div class="features">
            @foreach($features as $feature)
                <div class="feature-card">
                    <h3>{{ $feature }}</h3>
                    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam in dui mauris. Vivamus hendrerit arcu sed erat molestie vehicula.</p>
                </div>
            @endforeach
        </div>

        <div class="cta">
            <h2>Mulai Kelola Data Mahasiswa Sekarang</h2>
            <p>Klik tombol di bawah untuk melihat daftar mahasiswa</p>
            <a href="/mahasiswa" class="btn">Lihat Data Mahasiswa</a>
        </div>
    </div>
@endsection

@section('scripts')
<script>
    console.log('Halaman home berhasil dimuat');
</script>
@endsection
```

## Langkah 8: Coba Halaman dengan Layout Baru

Refresh browser Anda untuk melihat halaman home yang menggunakan layout master. Hasilnya akan terlihat sama, tetapi struktur kode sekarang lebih terorganisir dan dapat digunakan kembali untuk halaman-halaman lain.

## Kesimpulan

Selamat! Anda telah berhasil membuat halaman home yang sederhana namun menarik untuk aplikasi manajemen mahasiswa menggunakan Laravel. Anda telah belajar:

1. Membuat controller khusus untuk halaman home
2. Membuat view menarik dengan HTML dan CSS
3. Mengirim data dari controller ke view
4. Menggunakan Blade template untuk menampilkan data dinamis
5. Membuat dan menggunakan layout master untuk konsistensi tampilan

Halaman home yang baik akan membuat pengguna lebih tertarik untuk mengeksplorasi aplikasi Anda lebih lanjut. Pada tutorial selanjutnya, kita akan melanjutkan dengan membuat fitur CRUD untuk data mahasiswa.
