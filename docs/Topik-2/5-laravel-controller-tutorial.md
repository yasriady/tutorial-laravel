# Tutorial: Membuat Controller Pertama di Laravel

Controller adalah komponen penting dalam aplikasi Laravel yang menangani logika bisnis dan menjembatani interaksi antara Model dan View. Tutorial ini akan memandu Anda langkah demi langkah dalam membuat controller pertama Anda untuk aplikasi manajemen data Mahasiswa.

## Prasyarat
- Laravel sudah terinstal melalui Laragon
- Project `mahasiswa-app` sudah dibuat
- Sudah mengenal dasar routing di Laravel

## Langkah 1: Membuat Controller Menggunakan Artisan CLI

Buka terminal di root project Anda dan jalankan perintah berikut:

```
php artisan make:controller MahasiswaController
```

Perintah ini akan membuat file `MahasiswaController.php` di direktori `app/Http/Controllers/`.

## Langkah 2: Membuka dan Mengedit Controller

Buka file `MahasiswaController.php` menggunakan Visual Studio Code. File tersebut akan terlihat seperti ini:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class MahasiswaController extends Controller
{
    //
}
```

## Langkah 3: Menambahkan Method Index

Mari tambahkan method `index()` yang akan menampilkan daftar mahasiswa:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class MahasiswaController extends Controller
{
    /**
     * Menampilkan daftar mahasiswa
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        // Untuk sementara kita akan menampilkan data dummy
        $mahasiswa = [
            ['nim' => '20001', 'nama' => 'Budi Santoso', 'jurusan' => 'Teknik Informatika'],
            ['nim' => '20002', 'nama' => 'Ani Wijaya', 'jurusan' => 'Sistem Informasi'],
            ['nim' => '20003', 'nama' => 'Dewi Lestari', 'jurusan' => 'Teknik Komputer']
        ];
        
        return view('mahasiswa.index', compact('mahasiswa'));
    }
}
```

## Langkah 4: Membuat View untuk Controller

Buat folder baru bernama `mahasiswa` dalam direktori `resources/views/` dan buat file `index.blade.php` di dalamnya:

```
mkdir -p resources/views/mahasiswa
touch resources/views/mahasiswa/index.blade.php
```

Kemudian, isi file `index.blade.php` dengan kode berikut:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar Mahasiswa</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        h1 {
            color: #333;
        }
    </style>
</head>
<body>
    <h1>Daftar Mahasiswa</h1>
    
    <table>
        <thead>
            <tr>
                <th>NIM</th>
                <th>Nama</th>
                <th>Jurusan</th>
            </tr>
        </thead>
        <tbody>
            @foreach($mahasiswa as $mhs)
            <tr>
                <td>{{ $mhs['nim'] }}</td>
                <td>{{ $mhs['nama'] }}</td>
                <td>{{ $mhs['jurusan'] }}</td>
            </tr>
            @endforeach
        </tbody>
    </table>
</body>
</html>
```

## Langkah 5: Menambahkan Route untuk Controller

Buka file `routes/web.php` dan tambahkan route berikut:

```php
use App\Http\Controllers\MahasiswaController;

Route::get('/mahasiswa', [MahasiswaController::class, 'index']);
```

## Langkah 6: Menguji Controller

1. Pastikan server Laravel berjalan:
   ```
   php artisan serve
   ```
   Atau gunakan Laragon yang sudah berjalan

2. Buka browser dan akses:
   ```
   http://mahasiswa-app.test/mahasiswa
   ```
   Atau jika menggunakan server lokal:
   ```
   http://localhost/mahasiswa-app/public/mahasiswa
   ```

Anda akan melihat daftar mahasiswa yang ditampilkan dalam bentuk tabel.

## Langkah 7: Menambahkan Method Lain pada Controller

Sekarang tambahkan method `create()` untuk menampilkan form tambah mahasiswa:

```php
/**
 * Menampilkan form untuk tambah data
 *
 * @return \Illuminate\Http\Response
 */
public function create()
{
    return view('mahasiswa.create');
}
```

## Langkah 8: Penjelasan Resource Controller

Laravel menyediakan jenis controller yang disebut Resource Controller, yang secara otomatis membuat 7 method CRUD standar. Untuk membuat resource controller, jalankan:

```
php artisan make:controller MahasiswaController --resource
```

Perintah ini akan membuat controller dengan method-method berikut:
- `index()` - Menampilkan daftar resource
- `create()` - Menampilkan form untuk pembuatan resource baru
- `store()` - Menyimpan resource baru
- `show()` - Menampilkan resource tertentu
- `edit()` - Menampilkan form untuk edit resource
- `update()` - Memperbarui resource tertentu
- `destroy()` - Menghapus resource tertentu

## Kesimpulan

Selamat! Anda telah berhasil membuat controller pertama di Laravel untuk aplikasi manajemen mahasiswa. Controller ini memungkinkan Anda untuk mengatur logika bisnis aplikasi secara terpisah dari tampilan, yang membuat kode Anda lebih terorganisir dan mudah dikelola.

Pada tutorial berikutnya, kita akan belajar lebih lanjut tentang bagaimana membuat model dan menghubungkannya dengan database untuk menyimpan data mahasiswa secara permanen.
