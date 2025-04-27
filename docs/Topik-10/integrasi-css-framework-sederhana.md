# Integrasi CSS Framework Sederhana

CSS Framework sangat membantu mempercepat proses pengembangan tampilan aplikasi web. Framework ini menyediakan komponen dan utilitas yang sudah jadi, sehingga Anda tidak perlu menulis CSS dari awal. Dalam materi ini, kita akan mengintegrasikan Bootstrap, salah satu CSS framework paling populer, ke dalam aplikasi Laravel Mahasiswa.

## Cara Mengintegrasikan Bootstrap ke Laravel

Ada beberapa cara untuk mengintegrasikan Bootstrap ke aplikasi Laravel. Mari bahas masing-masing metode secara sistematis:

### 1. Menggunakan CDN (Content Delivery Network)

Cara termudah adalah dengan menambahkan link CDN Bootstrap di layout master:

```php
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="en">
<head>
    <!-- Meta tags dan title -->
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <!-- CSS Kustom Anda -->
    <link href="{{ asset('css/app.css') }}" rel="stylesheet">
    
    @yield('styles')
</head>
<body>
    <!-- Konten website -->
    
    <!-- Bootstrap JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    
    @yield('scripts')
</body>
</html>
```

**Kelebihan metode CDN:**
- Sangat mudah diterapkan
- Tidak perlu instalasi tambahan
- Browser mungkin sudah menyimpan cache Bootstrap dari website lain

**Kekurangan metode CDN:**
- Memerlukan koneksi internet
- Tidak dapat dikustomisasi dengan Sass

### 2. Menggunakan Laravel Mix (Untuk Laravel 8 ke bawah)

Laravel Mix adalah wrapper untuk Webpack yang mempermudah kompilasi aset:

1. **Install Node.js dan NPM** jika belum terinstall.

2. **Install Bootstrap dan dependensinya**:
   ```bash
   npm install bootstrap @popperjs/core
   ```

3. **Edit file webpack.mix.js**:
   ```javascript
   const mix = require('laravel-mix');

   mix.js('resources/js/app.js', 'public/js')
      .sass('resources/sass/app.scss', 'public/css');
   ```

4. **Buat/Edit file resources/sass/app.scss**:
   ```scss
   // Variabel Bootstrap bisa dikustomisasi di sini
   $primary: #4e73df;
   $secondary: #858796;
   $success: #1cc88a;

   // Import Bootstrap
   @import '~bootstrap/scss/bootstrap';

   // CSS kustom Anda
   body {
       font-family: 'Nunito', sans-serif;
       background-color: #f8f9fc;
   }

   // ... CSS lainnya
   ```

5. **Buat/Edit file resources/js/app.js**:
   ```javascript
   // Import Bootstrap JavaScript
   import 'bootstrap';

   // JavaScript kustom Anda
   console.log('Aplikasi Mahasiswa dimuat');
   ```

6. **Kompilasi aset**:
   ```bash
   npm run dev
   ```
   Atau untuk production:
   ```bash
   npm run prod
   ```

7. **Tambahkan aset yang telah dikompilasi ke layout**:
   ```php
   <link href="{{ asset('css/app.css') }}" rel="stylesheet">
   <script src="{{ asset('js/app.js') }}" defer></script>
   ```

### 3. Menggunakan Vite (Laravel 9+)

Laravel 9+ menggunakan Vite sebagai pengganti Laravel Mix:

1. **Install Bootstrap**:
   ```bash
   npm install bootstrap @popperjs/core
   ```

2. **Edit vite.config.js**:
   ```javascript
   import { defineConfig } from 'vite';
   import laravel from 'laravel-vite-plugin';

   export default defineConfig({
       plugins: [
           laravel({
               input: ['resources/css/app.css', 'resources/js/app.js'],
               refresh: true,
           }),
       ],
       resolve: {
           alias: {
               '~bootstrap': 'node_modules/bootstrap',
           }
       },
   });
   ```

3. **Buat resources/css/app.css**:
   ```css
   @import '~bootstrap/dist/css/bootstrap.min.css';

   /* CSS kustom Anda */
   ```

4. **Edit resources/js/app.js**:
   ```javascript
   import '../css/app.css';
   import * as bootstrap from 'bootstrap';

   // JavaScript kustom Anda
   ```

5. **Jalankan Vite**:
   ```bash
   npm run dev
   ```

6. **Tambahkan ke layout**:
   ```php
   @vite(['resources/css/app.css', 'resources/js/app.js'])
   ```

## Implementasi Praktis untuk Aplikasi Mahasiswa

Setelah mengintegrasikan Bootstrap, kita bisa mulai menggunakannya untuk mempercantik aplikasi Mahasiswa:

### 1. Dashboard dengan Card dan Grid System

```php
{{-- resources/views/dashboard.blade.php --}}
@extends('layouts.app')

@section('content')
<div class="container">
    <h1 class="mb-4">Dashboard</h1>
    
    <div class="row">
        <!-- Card: Total Mahasiswa -->
        <div class="col-md-4 mb-4">
            <div class="card border-left-primary shadow h-100">
                <div class="card-body">
                    <div class="row no-gutters align-items-center">
                        <div class="col mr-2">
                            <div class="text-xs font-weight-bold text-primary text-uppercase mb-1">
                                Total Mahasiswa</div>
                            <div class="h5 mb-0 font-weight-bold text-gray-800">{{ $totalMahasiswa }}</div>
                        </div>
                        <div class="col-auto">
                            <i class="fas fa-users fa-2x text-gray-300"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Card: Total Jurusan -->
        <div class="col-md-4 mb-4">
            <div class="card border-left-success shadow h-100">
                <div class="card-body">
                    <div class="row no-gutters align-items-center">
                        <div class="col mr-2">
                            <div class="text-xs font-weight-bold text-success text-uppercase mb-1">
                                Total Jurusan</div>
                            <div class="h5 mb-0 font-weight-bold text-gray-800">{{ $totalJurusan }}</div>
                        </div>
                        <div class="col-auto">
                            <i class="fas fa-university fa-2x text-gray-300"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Card: Total Mata Kuliah -->
        <div class="col-md-4 mb-4">
            <div class="card border-left-info shadow h-100">
                <div class="card-body">
                    <div class="row no-gutters align-items-center">
                        <div class="col mr-2">
                            <div class="text-xs font-weight-bold text-info text-uppercase mb-1">
                                Total Mata Kuliah</div>
                            <div class="h5 mb-0 font-weight-bold text-gray-800">{{ $totalMataKuliah }}</div>
                        </div>
                        <div class="col-auto">
                            <i class="fas fa-book fa-2x text-gray-300"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Grafik Mahasiswa per Jurusan -->
    <div class="row">
        <div class="col-12">
            <div class="card shadow mb-4">
                <div class="card-header py-3">
                    <h6 class="m-0 font-weight-bold text-primary">Mahasiswa per Jurusan</h6>
                </div>
                <div class="card-body">
                    <div class="chart-bar">
                        <canvas id="mahasiswaJurusanChart"></canvas>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

### 2. Tabel Responsif untuk Daftar Mahasiswa

```php
{{-- resources/views/mahasiswa/index.blade.php --}}
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="d-sm-flex align-items-center justify-content-between mb-4">
        <h1 class="h3 mb-0 text-gray-800">Daftar Mahasiswa</h1>
        <a href="{{ route('mahasiswa.create') }}" class="btn btn-primary">
            <i class="fas fa-plus fa-sm text-white-50"></i> Tambah Mahasiswa
        </a>
    </div>
    
    @if(session('success'))
        <div class="alert alert-success alert-dismissible fade show" role="alert">
            {{ session('success') }}
            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
        </div>
    @endif
    
    <div class="card shadow mb-4">
        <div class="card-header py-3">
            <div class="row align-items-center">
                <div class="col">
                    <h6 class="m-0 font-weight-bold text-primary">Data Mahasiswa</h6>
                </div>
                <div class="col-auto">
                    <form action="{{ route('mahasiswa.index') }}" method="GET" class="d-flex">
                        <input type="text" name="search" class="form-control form-control-sm mr-2" 
                               placeholder="Cari mahasiswa..." value="{{ request('search') }}">
                        <button type="submit" class="btn btn-sm btn-primary">Cari</button>
                    </form>
                </div>
            </div>
        </div>
        <div class="card-body">
            <div class="table-responsive">
                <table class="table table-bordered table-striped" width="100%" cellspacing="0">
                    <thead class="bg-primary text-white">
                        <tr>
                            <th>No</th>
                            <th>NIM</th>
                            <th>Nama</th>
                            <th>Jurusan</th>
                            <th>Email</th>
                            <th>Aksi</th>
                        </tr>
                    </thead>
                    <tbody>
                        @forelse($mahasiswas as $index => $mahasiswa)
                            <tr>
                                <td>{{ $index + $mahasiswas->firstItem() }}</td>
                                <td>{{ $mahasiswa->nim }}</td>
                                <td>{{ $mahasiswa->nama }}</td>
                                <td>{{ $mahasiswa->jurusan->nama }}</td>
                                <td>{{ $mahasiswa->email }}</td>
                                <td>
                                    <a href="{{ route('mahasiswa.show', $mahasiswa->id) }}" class="btn btn-info btn-sm">
                                        <i class="fas fa-eye"></i>
                                    </a>
                                    <a href="{{ route('mahasiswa.edit', $mahasiswa->id) }}" class="btn btn-warning btn-sm">
                                        <i class="fas fa-edit"></i>
                                    </a>
                                    <form action="{{ route('mahasiswa.destroy', $mahasiswa->id) }}" method="POST" class="d-inline">
                                        @csrf
                                        @method('DELETE')
                                        <button type="submit" class="btn btn-danger btn-sm" onclick="return confirm('Yakin ingin menghapus?')">
                                            <i class="fas fa-trash"></i>
                                        </button>
                                    </form>
                                </td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="6" class="text-center">Tidak ada data mahasiswa</td>
                            </tr>
                        @endforelse
                    </tbody>
                </table>
            </div>
            
            <div class="d-flex justify-content-end">
                {{ $mahasiswas->links() }}
            </div>
        </div>
    </div>
</div>
@endsection
```

### 3. Form Responsif untuk Tambah/Edit Mahasiswa

```php
{{-- resources/views/mahasiswa/create.blade.php --}}
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card shadow">
                <div class="card-header bg-primary text-white">
                    <h5 class="mb-0">Tambah Mahasiswa Baru</h5>
                </div>
                <div class="card-body">
                    <form action="{{ route('mahasiswa.store') }}" method="POST">
                        @csrf
                        
                        <div class="mb-3">
                            <label for="nim" class="form-label">NIM</label>
                            <input type="text" class="form-control @error('nim') is-invalid @enderror" 
                                   id="nim" name="nim" value="{{ old('nim') }}" required>
                            @error('nim')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>
                        
                        <div class="mb-3">
                            <label for="nama" class="form-label">Nama Lengkap</label>
                            <input type="text" class="form-control @error('nama') is-invalid @enderror" 
                                   id="nama" name="nama" value="{{ old('nama') }}" required>
                            @error('nama')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>
                        
                        <div class="mb-3">
                            <label for="jurusan_id" class="form-label">Jurusan</label>
                            <select class="form-select @error('jurusan_id') is-invalid @enderror" 
                                    id="jurusan_id" name="jurusan_id" required>
                                <option value="">-- Pilih Jurusan --</option>
                                @foreach($jurusans as $jurusan)
                                    <option value="{{ $jurusan->id }}" {{ old('jurusan_id') == $jurusan->id ? 'selected' : '' }}>
                                        {{ $jurusan->nama }}
                                    </option>
                                @endforeach
                            </select>
                            @error('jurusan_id')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>
                        
                        <div class="mb-3">
                            <label for="email" class="form-label">Email</label>
                            <input type="email" class="form-control @error('email') is-invalid @enderror" 
                                   id="email" name="email" value="{{ old('email') }}" required>
                            @error('email')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>
                        
                        <div class="mb-3">
                            <label for="alamat" class="form-label">Alamat</label>
                            <textarea class="form-control @error('alamat') is-invalid @enderror" 
                                      id="alamat" name="alamat" rows="3">{{ old('alamat') }}</textarea>
                            @error('alamat')
                                <div class="invalid-feedback">{{ $message }}</div>
                            @enderror
                        </div>
                        
                        <div class="mb-3">
                            <label class="form-label">Jenis Kelamin</label>
                            <div>
                                <div class="form-check form-check-inline">
                                    <input class="form-check-input" type="radio" name="jenis_kelamin" 
                                           id="jk_l" value="L" {{ old('jenis_kelamin') == 'L' ? 'checked' : '' }} required>
                                    <label class="form-check-label" for="jk_l">Laki-laki</label>
                                </div>
                                <div class="form-check form-check-inline">
                                    <input class="form-check-input" type="radio" name="jenis_kelamin" 
                                           id="jk_p" value="P" {{ old('jenis_kelamin') == 'P' ? 'checked' : '' }}>
                                    <label class="form-check-label" for="jk_p">Perempuan</label>
                                </div>
                                @error('jenis_kelamin')
                                    <div class="text-danger mt-1">{{ $message }}</div>
                                @enderror
                            </div>
                        </div>
                        
                        <div class="d-grid gap-2 d-md-flex justify-content-md-end">
                            <a href="{{ route('mahasiswa.index') }}" class="btn btn-secondary me-md-2">Batal</a>
                            <button type="submit" class="btn btn-primary">Simpan</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

### Menambahkan Font Awesome untuk Icon

Font Awesome sangat berguna untuk menambahkan icon di aplikasi:

1. **Tambahkan link CDN Font Awesome ke layout**:
```php
<!-- Font Awesome -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
```

2. **Atau install via npm**:
```bash
npm install @fortawesome/fontawesome-free
```

3. **Tambahkan ke file Sass** (jika menggunakan Laravel Mix):
```scss
// resources/sass/app.scss
@import '~@fortawesome/fontawesome-free/css/all.min.css';
```

## Tips Tambahan

1. **Responsive Mobile First**: Selalu gunakan pendekatan mobile-first saat membuat tampilan.

2. **Custom CSS**: Buat file CSS kustom untuk override style Bootstrap jika diperlukan:
```css
/* public/css/custom.css */
.sidebar {
    background: linear-gradient(180deg, #4e73df 10%, #224abe 100%);
}

.btn-primary {
    background-color: #4e73df;
    border-color: #4e73df;
}
```

3. **Tema**: Jika ingin tampilan lebih menarik, pertimbangkan untuk menggunakan tema Bootstrap seperti SB Admin 2, AdminLTE, atau Tabler.

Dengan mengintegrasikan CSS framework seperti Bootstrap, aplikasi Laravel Mahasiswa Anda akan memiliki tampilan yang profesional dan responsif tanpa perlu menulis banyak CSS dari awal.

Selanjutnya, kita akan membahas: **Form dengan Validasi JavaScript**