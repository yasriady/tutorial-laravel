# Tutorial: Membuat View Pertama di Laravel

View adalah komponen yang menangani tampilan antarmuka pengguna dalam aplikasi Laravel. Pada tutorial ini, kita akan belajar cara membuat view pertama untuk aplikasi manajemen data Mahasiswa.

## Prasyarat
- Laravel sudah terinstal melalui Laragon
- Project `mahasiswa-app` sudah dibuat
- Sudah memahami dasar routing dan controller di Laravel

## Langkah 1: Memahami Struktur View Laravel

View dalam Laravel disimpan di direktori `resources/views`. Laravel menggunakan Blade sebagai template engine, yang memungkinkan kita mencampur HTML dengan kode PHP dalam bentuk yang lebih sederhana dan mudah dibaca.

File view Laravel memiliki ekstensi `.blade.php`, yang menandakan bahwa file tersebut menggunakan template engine Blade.

## Langkah 2: Membuat File View Sederhana

Mari kita buat halaman beranda sederhana untuk aplikasi kita:

1. Buat file `welcome.blade.php` di direktori `resources/views` (atau gunakan file yang sudah ada):

```
touch resources/views/welcome.blade.php
```

2. Buka file tersebut di Visual Studio Code dan tambahkan kode berikut:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Manajemen Mahasiswa</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f4f4f4;
        }
        .container {
            text-align: center;
            padding: 20px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            max-width: 800px;
        }
        h1 {
            color: #3490dc;
        }
        .btn {
            display: inline-block;
            background-color: #3490dc;
            color: white;
            padding: 10px 20px;
            text-decoration: none;
            border-radius: 5px;
            margin-top: 20px;
        }
        .btn:hover {
            background-color: #2779bd;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Selamat Datang di Aplikasi Manajemen Mahasiswa</h1>
        <p>Aplikasi sederhana untuk mengelola data mahasiswa menggunakan Laravel.</p>
        <a href="/mahasiswa" class="btn">Lihat Daftar Mahasiswa</a>
    </div>
</body>
</html>
```

## Langkah 3: Menghubungkan Route dengan View

Buka file `routes/web.php` dan tambahkan route berikut:

```php
Route::get('/', function () {
    return view('welcome');
});
```

Route ini akan menampilkan view `welcome.blade.php` ketika pengguna mengakses halaman utama aplikasi.

## Langkah 4: Menguji View

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

Anda akan melihat halaman selamat datang yang baru saja dibuat.

## Langkah 5: Menggunakan Blade Template Engine

Blade memiliki berbagai fitur yang memudahkan pembuatan template. Mari kita buat view baru yang memanfaatkan beberapa fitur Blade:

1. Buat file `about.blade.php` di direktori `resources/views`:

```
touch resources/views/about.blade.php
```

2. Isi file dengan kode berikut:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tentang Aplikasi</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 20px;
            background-color: #f4f4f4;
        }
        .container {
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            padding: 20px;
            max-width: 800px;
            margin: 0 auto;
        }
        h1 {
            color: #3490dc;
        }
        .features {
            margin-top: 20px;
        }
        .feature {
            margin-bottom: 10px;
            padding-left: 20px;
            position: relative;
        }
        .feature:before {
            content: "✓";
            color: #38c172;
            position: absolute;
            left: 0;
        }
        .nav {
            margin-top: 20px;
        }
        .nav a {
            color: #3490dc;
            margin-right: 15px;
            text-decoration: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Tentang Aplikasi Manajemen Mahasiswa</h1>
        
        <p>Versi Aplikasi: {{ $version }}</p>
        
        <div class="features">
            <h2>Fitur Aplikasi:</h2>
            
            @foreach($features as $feature)
                <div class="feature">{{ $feature }}</div>
            @endforeach
        </div>
        
        @if($isAdmin)
            <div class="admin-info">
                <h3>Informasi Admin</h3>
                <p>Anda memiliki akses admin ke aplikasi ini.</p>
            </div>
        @else
            <p>Login sebagai admin untuk melihat informasi tambahan.</p>
        @endif
        
        <div class="nav">
            <a href="/">Beranda</a>
            <a href="/mahasiswa">Daftar Mahasiswa</a>
        </div>
    </div>
</body>
</html>
```

3. Tambahkan route baru di `routes/web.php`:

```php
Route::get('/about', function () {
    return view('about', [
        'version' => '1.0.0',
        'features' => [
            'Manajemen data mahasiswa',
            'Pencarian mahasiswa',
            'Laporan akademik',
            'Manajemen jurusan'
        ],
        'isAdmin' => true
    ]);
});
```

## Langkah 6: Memahami Passing Data ke View

Pada contoh di atas, kita mengirim data ke view menggunakan parameter kedua pada fungsi `view()`. Kita dapat mengirimkan data berupa:

- String (`'version' => '1.0.0'`)
- Array (`'features' => [...]`)
- Boolean (`'isAdmin' => true`)

Di dalam view, data ini dapat diakses menggunakan sintaks Blade `{{ $namaVariabel }}`.

## Langkah 7: Memahami Direktif Blade

Blade menyediakan berbagai direktif yang memudahkan kita dalam menulis kode:

- `@foreach` dan `@endforeach` - Untuk melakukan perulangan
- `@if`, `@else`, `@elseif`, dan `@endif` - Untuk kondisional
- `@include` - Untuk menyertakan view lain
- `@yield` dan `@section` - Untuk template inheritance

## Langkah 8: Membuat Layout dengan Blade

Mari buat layout master yang dapat digunakan kembali:

1. Buat direktori `layouts` dalam `resources/views`:

```
mkdir -p resources/views/layouts
```

2. Buat file `app.blade.php` di dalam direktori tersebut:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Aplikasi Mahasiswa')</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        .container {
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px;
        }
        header {
            background-color: #3490dc;
            color: white;
            padding: 10px 0;
            text-align: center;
        }
        .content {
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            padding: 20px;
            margin-top: 20px;
        }
        footer {
            text-align: center;
            margin-top: 20px;
            padding: 10px;
            background-color: #f8f9fa;
            border-radius: 8px;
        }
        nav {
            background-color: #2779bd;
            padding: 10px;
        }
        nav a {
            color: white;
            text-decoration: none;
            margin: 0 10px;
        }
    </style>
    @yield('styles')
</head>
<body>
    <header>
        <h1>Aplikasi Manajemen Mahasiswa</h1>
    </header>
    
    <nav>
        <div class="container">
            <a href="/">Beranda</a>
            <a href="/about">Tentang</a>
            <a href="/mahasiswa">Daftar Mahasiswa</a>
        </div>
    </nav>
    
    <div class="container">
        <div class="content">
            @yield('content')
        </div>
        
        <footer>
            &copy; {{ date('Y') }} Aplikasi Manajemen Mahasiswa
        </footer>
    </div>
    
    @yield('scripts')
</body>
</html>
```

3. Buat file view baru yang menggunakan layout di atas:

```
touch resources/views/dashboard.blade.php
```

4. Isi file dengan kode berikut:

```php
@extends('layouts.app')

@section('title', 'Dashboard')

@section('styles')
<style>
    .dashboard-stats {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 20px;
        margin-bottom: 20px;
    }
    .stat-card {
        background-color: #e6f2ff;
        border-radius: 8px;
        padding: 15px;
        text-align: center;
    }
    .stat-value {
        font-size: 24px;
        font-weight: bold;
        color: #3490dc;
    }
</style>
@endsection

@section('content')
    <h2>Dashboard</h2>
    
    <div class="dashboard-stats">
        <div class="stat-card">
            <p>Total Mahasiswa</p>
            <div class="stat-value">{{ $totalMahasiswa }}</div>
        </div>
        
        <div class="stat-card">
            <p>Total Jurusan</p>
            <div class="stat-value">{{ $totalJurusan }}</div>
        </div>
        
        <div class="stat-card">
            <p>Mahasiswa Aktif</p>
            <div class="stat-value">{{ $mahasiswaAktif }}</div>
        </div>
    </div>
    
    <p>Selamat datang di dashboard Aplikasi Manajemen Mahasiswa. Disini Anda dapat melihat statistik dan informasi penting.</p>
@endsection

@section('scripts')
<script>
    console.log('Dashboard loaded!');
</script>
@endsection
```

5. Tambahkan route untuk dashboard:

```php
Route::get('/dashboard', function () {
    return view('dashboard', [
        'totalMahasiswa' => 120,
        'totalJurusan' => 5,
        'mahasiswaAktif' => 98
    ]);
});
```

## Kesimpulan

Selamat! Anda telah berhasil membuat view pertama di Laravel dan mempelajari dasar-dasar Blade template engine. Dalam tutorial ini, Anda telah belajar:

1. Membuat file view dasar
2. Menghubungkan route dengan view
3. Mengirim data ke view
4. Menggunakan direktif Blade seperti `@foreach` dan `@if`
5. Membuat layout master dengan `@yield` dan `@extends`

Blade template engine sangat kuat dan fleksibel, memungkinkan Anda membuat tampilan yang dinamis dan mudah dikelola untuk aplikasi Laravel Anda. Pada tutorial selanjutnya, kita akan belajar bagaimana mengintegrasikan view dengan model dan controller untuk membuat aplikasi yang lengkap.
