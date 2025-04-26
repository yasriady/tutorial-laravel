# Pembuatan Proyek Laravel Pertama - Tutorial Singkat

## Pengenalan
Laravel adalah framework PHP yang elegan untuk pengembangan aplikasi web modern. Pada tutorial ini, kita akan membuat proyek Laravel pertama dan mempelajari struktur dasar framework ini.

## Prasyarat
Sebelum memulai, pastikan Anda telah menginstal:
- PHP 8.1 atau lebih baru
- Composer
- Laragon (atau lingkungan pengembangan lainnya)
- Visual Studio Code

## Langkah-langkah Pembuatan Proyek Laravel

### 1. Membuat Proyek Laravel Baru

#### Menggunakan Laragon (Cara Mudah)
- Buka aplikasi Laragon
- Klik kanan di area kosong panel Laragon
- Pilih **Quick create** → **Laravel**
- Masukkan nama proyek, misalnya: `mahasiswa-app`
- Tunggu hingga proses instalasi selesai

#### Menggunakan Composer (Command Line)
- Buka Command Prompt atau Terminal
- Arahkan ke direktori web server Anda:
  ```bash
  cd C:\laragon\www
  ```
- Jalankan perintah Composer untuk membuat proyek Laravel baru:
  ```bash
  composer create-project laravel/laravel mahasiswa-app
  ```
- Tunggu hingga semua dependensi terinstal

### 2. Mengakses Proyek Laravel

#### Melalui Browser
- Jalankan Laragon dan klik tombol "Start All"
- Buka browser dan akses URL:
  - Jika menggunakan Laragon dengan virtual host: `http://mahasiswa-app.test`
  - Jika menggunakan server lokal: `http://localhost/mahasiswa-app/public`

#### Mengakses dan Mengedit Kode
- Buka VS Code
- Pilih **File** → **Open Folder**
- Navigasi ke folder proyek Laravel Anda (misalnya: `C:\laragon\www\mahasiswa-app`)
- Atau klik kanan pada proyek di Laragon dan pilih "Open in VS Code"

### 3. Mengenal Struktur Folder Laravel

Berikut adalah struktur folder utama yang perlu Anda ketahui:

- **app/**: Berisi kode utama aplikasi
  - **Http/Controllers/**: Tempat menyimpan controller
  - **Models/**: Tempat menyimpan model
  - **Providers/**: Service providers aplikasi
- **config/**: File konfigurasi aplikasi
- **database/**:
  - **migrations/**: File migrasi database
  - **seeds/**: File seeder database
- **public/**: Dokumen root web server, berisi file index.php dan aset
- **resources/**:
  - **views/**: File tampilan Blade
  - **css/ & js/**: File sumber daya frontend
- **routes/**: File definisi rute
  - **web.php**: Rute untuk aplikasi web
  - **api.php**: Rute untuk API
- **.env**: File konfigurasi lingkungan

### 4. Konfigurasi Database

- Buka file `.env` di root proyek
- Sesuaikan konfigurasi database:
  ```
  DB_CONNECTION=mysql
  DB_HOST=127.0.0.1
  DB_PORT=3306
  DB_DATABASE=mahasiswa_app
  DB_USERNAME=root
  DB_PASSWORD=
  ```
- Buat database baru di Laragon:
  - Klik menu **Database** di Laragon
  - Pilih HeidiSQL atau PHPMyAdmin
  - Buat database baru dengan nama `mahasiswa_app`

### 5. Menjalankan Migrasi Dasar

- Buka Terminal di VS Code (Menu **Terminal** → **New Terminal**)
- Jalankan perintah untuk migrasi:
  ```bash
  php artisan migrate
  ```
- Jika muncul pesan konfirmasi, ketik "yes" untuk membuat tabel migrasi

### 6. Membuat Halaman Beranda Sederhana

#### Modifikasi Route
- Buka file `routes/web.php`
- Ubah route default menjadi:
  ```php
  Route::get('/', function () {
      return view('welcome', [
          'title' => 'Aplikasi Manajemen Mahasiswa',
          'description' => 'Selamat datang di aplikasi manajemen data mahasiswa'
      ]);
  });
  ```

#### Modifikasi View
- Buka `resources/views/welcome.blade.php`
- Sesuaikan tampilannya, misalnya:
  ```html
  <!DOCTYPE html>
  <html lang="id">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>{{ $title }}</title>
      <style>
          body {
              font-family: 'Nunito', sans-serif;
              margin: 0;
              padding: 0;
              background-color: #f7f8fa;
          }
          .container {
              max-width: 800px;
              margin: 0 auto;
              padding: 2rem;
              text-align: center;
          }
          h1 {
              color: #3490dc;
          }
          p {
              color: #606f7b;
              font-size: 1.1rem;
          }
      </style>
  </head>
  <body>
      <div class="container">
          <h1>{{ $title }}</h1>
          <p>{{ $description }}</p>
          <p>Dibuat dengan Laravel v{{ Illuminate\Foundation\Application::VERSION }}</p>
      </div>
  </body>
  </html>
  ```

### 7. Membuat Controller Pertama

- Jalankan perintah Artisan untuk membuat controller:
  ```bash
  php artisan make:controller HomeController
  ```
- Buka file `app/Http/Controllers/HomeController.php` yang baru dibuat
- Edit controller:
  ```php
  <?php

  namespace App\Http\Controllers;

  use Illuminate\Http\Request;

  class HomeController extends Controller
  {
      public function index()
      {
          return view('welcome', [
              'title' => 'Aplikasi Manajemen Mahasiswa',
              'description' => 'Selamat datang di aplikasi manajemen data mahasiswa'
          ]);
      }
  }
  ```

- Ubah `routes/web.php` untuk menggunakan controller:
  ```php
  use App\Http\Controllers\HomeController;

  Route::get('/', [HomeController::class, 'index']);
  ```

### 8. Menjalankan Server Pengembangan

- Di Terminal VS Code, jalankan:
  ```bash
  php artisan serve
  ```
- Aplikasi akan berjalan di `http://127.0.0.1:8000`
- Catatan: Langkah ini opsional jika sudah menggunakan Laragon

### 9. Eksplorasi Perintah Artisan

- Lihat semua perintah Artisan yang tersedia:
  ```bash
  php artisan list
  ```

- Beberapa perintah penting:
  ```bash
  # Membuat controller
  php artisan make:controller NamaController
  
  # Membuat model
  php artisan make:model Nama
  
  # Membuat model sekaligus dengan migrasi
  php artisan make:model Nama -m
  
  # Membuat migrasi
  php artisan make:migration create_nama_table
  
  # Melihat rute yang terdaftar
  php artisan route:list
  
  # Membersihkan cache
  php artisan cache:clear
  ```

### 10. Mengaktifkan Fitur Debugging

- Buka file `.env`
- Ubah `APP_DEBUG=true` (untuk lingkungan pengembangan)
- Jika ingin menggunakan package tambahan untuk debugging, instal Laravel Debugbar:
  ```bash
  composer require barryvdh/laravel-debugbar --dev
  ```

## Poin Penting untuk Diingat

1. File `.env` berisi konfigurasi sensitif, jangan pernah commit file ini ke repository Git
2. Pada lingkungan produksi, pastikan APP_DEBUG=false untuk keamanan
3. Selalu gunakan fitur migrasi untuk mengelola struktur database
4. Gunakan Artisan untuk membuat file-file boilerplate untuk memastikan struktur yang benar
5. Laravel memiliki dokumentasi lengkap di [https://laravel.com/docs](https://laravel.com/docs)

## Kesimpulan

Selamat! Anda telah berhasil membuat proyek Laravel pertama. Anda telah mempelajari struktur dasar, cara membuat rute, controller, dan view sederhana. Langkah selanjutnya adalah mempelajari lebih dalam tentang sistem database Laravel, Eloquent ORM, dan fitur-fitur lainnya.
