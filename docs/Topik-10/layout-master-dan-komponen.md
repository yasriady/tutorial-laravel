# Layout Master dan Komponen

Salah satu kekuatan utama Blade adalah kemampuannya untuk membuat layout master dan komponen yang dapat digunakan kembali. Ini memungkinkan Anda mempertahankan konsistensi UI dan menghindari duplikasi kode.

## Layout Master (Template Inheritance)

Layout master adalah template dasar yang berisi struktur umum dari aplikasi Anda, seperti header, footer, navbar, dan sidebar. Halaman-halaman lain kemudian dapat "mewarisi" dari layout ini.

### Membuat Layout Master

```php
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Aplikasi Mahasiswa')</title>
    
    <!-- CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    @yield('styles')
</head>
<body>
    <!-- Navbar -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="{{ route('home') }}">Sistem Mahasiswa</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link" href="{{ route('home') }}">Home</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{{ route('mahasiswa.index') }}">Mahasiswa</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{{ route('jurusan.index') }}">Jurusan</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{{ route('matakuliah.index') }}">Mata Kuliah</a>
                    </li>
                </ul>
                <ul class="navbar-nav ms-auto">
                    @auth
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown">
                                {{ Auth::user()->name }}
                            </a>
                            <ul class="dropdown-menu">
                                <li><a class="dropdown-item" href="{{ route('profile') }}">Profile</a></li>
                                <li><hr class="dropdown-divider"></li>
                                <li>
                                    <form action="{{ route('logout') }}" method="POST">
                                        @csrf
                                        <button type="submit" class="dropdown-item">Logout</button>
                                    </form>
                                </li>
                            </ul>
                        </li>
                    @else
                        <li class="nav-item">
                            <a class="nav-link" href="{{ route('login') }}">Login</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="{{ route('register') }}">Register</a>
                        </li>
                    @endauth
                </ul>
            </div>
        </div>
    </nav>

    <!-- Content -->
    <main class="container py-4">
        @yield('content')
    </main>

    <!-- Footer -->
    <footer class="bg-light py-4 mt-auto">
        <div class="container text-center">
            <p>&copy; {{ date('Y') }} Aplikasi Mahasiswa. All rights reserved.</p>
        </div>
    </footer>

    <!-- JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    @yield('scripts')
</body>
</html>
```

### Menggunakan Layout Master

```php
{{-- resources/views/mahasiswa/index.blade.php --}}
@extends('layouts.app')

@section('title', 'Daftar Mahasiswa')

@section('content')
    <div class="card">
        <div class="card-header d-flex justify-content-between align-items-center">
            <h5>Daftar Mahasiswa</h5>
            <a href="{{ route('mahasiswa.create') }}" class="btn btn-primary">Tambah Mahasiswa</a>
        </div>
        <div class="card-body">
            <!-- Konten halaman daftar mahasiswa disini -->
            @if(session('success'))
                <div class="alert alert-success">
                    {{ session('success') }}
                </div>
            @endif
            
            <!-- Table of mahasiswa -->
        </div>
    </div>
@endsection

@section('scripts')
    <script>
        // JavaScript khusus untuk halaman ini
        console.log('Halaman daftar mahasiswa loaded');
    </script>
@endsection
```

## Partials/Includes

Untuk bagian-bagian UI yang digunakan di beberapa halaman, Anda dapat membuat partials:

```php
{{-- resources/views/partials/alert.blade.php --}}
<div class="alert alert-{{ $type ?? 'info' }} alert-dismissible fade show" role="alert">
    {{ $message }}
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
</div>
```

Kemudian menggunakan partials tersebut:

```php
@if(session('success'))
    @include('partials.alert', ['type' => 'success', 'message' => session('success')])
@endif

@if(session('error'))
    @include('partials.alert', ['type' => 'danger', 'message' => session('error')])
@endif
```

## Komponen (Laravel 7+)

Laravel 7+ memperkenalkan komponen class-based dan anonymous yang membuat sistem templating lebih powerful:

### Class-based Components

1. Membuat komponen dengan artisan:

```bash
php artisan make:component Alert
```

Ini akan membuat:
- `app/View/Components/Alert.php`
- `resources/views/components/alert.blade.php`

2. Edit class komponen:

```php
// app/View/Components/Alert.php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public $type;
    public $message;

    public function __construct($type = 'info', $message)
    {
        $this->type = $type;
        $this->message = $message;
    }

    public function render()
    {
        return view('components.alert');
    }
}
```

3. Edit template komponen:

```php
{{-- resources/views/components/alert.blade.php --}}
<div class="alert alert-{{ $type }} alert-dismissible fade show" role="alert">
    {{ $message }}
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
</div>
```

4. Menggunakan komponen:

```php
<x-alert type="success" message="Data berhasil disimpan!" />
```

### Anonymous Components

Untuk komponen sederhana tanpa logika:

```php
{{-- resources/views/components/input.blade.php --}}
<div class="mb-3">
    <label for="{{ $id }}" class="form-label">{{ $label }}</label>
    <input type="{{ $type ?? 'text' }}" class="form-control @error($name) is-invalid @enderror" 
           id="{{ $id }}" name="{{ $name }}" value="{{ old($name, $value ?? '') }}">
    
    @error($name)
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror
</div>
```

Kemudian menggunakannya di form:

```php
<x-input id="nim" name="nim" label="NIM Mahasiswa" value="{{ $mahasiswa->nim ?? '' }}" />
<x-input id="nama" name="nama" label="Nama Mahasiswa" value="{{ $mahasiswa->nama ?? '' }}" />
<x-input id="email" name="email" label="Email" type="email" value="{{ $mahasiswa->email ?? '' }}" />
```

## Slots

Slots memungkinkan Anda memasukkan konten ke dalam komponen:

```php
{{-- resources/views/components/card.blade.php --}}
<div class="card">
    <div class="card-header">{{ $header }}</div>
    <div class="card-body">
        {{ $slot }}
    </div>
    @if(isset($footer))
        <div class="card-footer">
            {{ $footer }}
        </div>
    @endif
</div>
```

Penggunaan:

```php
<x-card>
    <x-slot name="header">Detail Mahasiswa</x-slot>
    
    <p>Nama: {{ $mahasiswa->nama }}</p>
    <p>NIM: {{ $mahasiswa->nim }}</p>
    <p>Jurusan: {{ $mahasiswa->jurusan->nama }}</p>
    
    <x-slot name="footer">
        <a href="{{ route('mahasiswa.edit', $mahasiswa->id) }}" class="btn btn-primary">Edit</a>
    </x-slot>
</x-card>
```

Dengan menggunakan layout master dan komponen, Anda dapat membuat aplikasi yang lebih terstruktur dan mudah di-maintain. UI akan konsisten di seluruh aplikasi dan perubahan pada elemen umum cukup dilakukan di satu tempat.

Selanjutnya, kita akan membahas: **Integrasi CSS Framework Sederhana**