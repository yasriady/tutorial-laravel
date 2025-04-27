# Tutorial: Routing di Laravel

## Pengantar

Routing adalah salah satu konsep fundamental dalam Laravel yang menentukan bagaimana aplikasi Anda merespons permintaan pada URL tertentu. Dengan routing, Anda dapat mengatur endpoint aplikasi yang akan mengarahkan pengguna ke controller dan tindakan yang tepat.

## Dasar-Dasar Routing

### 1. Lokasi File Route

Di Laravel, semua konfigurasi route didefinisikan di dalam direktori `routes`. File-file utama yang perlu Anda perhatikan:

- `routes/web.php` - untuk route yang diakses melalui browser (biasanya mengembalikan view)
- `routes/api.php` - untuk API endpoint
- `routes/console.php` - untuk perintah console
- `routes/channels.php` - untuk broadcast channels

### 2. Membuat Route Dasar

Mari kita mulai dengan membuat route sederhana di file `routes/web.php`:

```php
// routes/web.php
use Illuminate\Support\Facades\Route;

Route::get('/mahasiswa', function () {
    return 'Daftar Mahasiswa';
});
```

Route di atas akan mengembalikan teks "Daftar Mahasiswa" ketika pengguna mengakses URL `http://aplikasi-anda.com/mahasiswa`.

### 3. HTTP Verbs

Laravel mendukung berbagai HTTP verbs untuk route:

```php
// GET Request
Route::get('/mahasiswa', function () {
    return 'Daftar Mahasiswa';
});

// POST Request (untuk form submit)
Route::post('/mahasiswa', function () {
    return 'Mahasiswa berhasil ditambahkan';
});

// PUT/PATCH Request (untuk update)
Route::put('/mahasiswa/{id}', function ($id) {
    return 'Mahasiswa dengan ID ' . $id . ' berhasil diperbarui';
});

// DELETE Request
Route::delete('/mahasiswa/{id}', function ($id) {
    return 'Mahasiswa dengan ID ' . $id . ' berhasil dihapus';
});
```

### 4. Route Parameter

Anda dapat menentukan parameter dinamis dalam URL route:

```php
Route::get('/mahasiswa/{id}', function ($id) {
    return 'Detail Mahasiswa dengan ID: ' . $id;
});
```

#### Parameter Opsional

Tambahkan `?` setelah nama parameter dan berikan nilai default:

```php
Route::get('/mahasiswa/{id?}', function ($id = null) {
    if ($id) {
        return 'Detail Mahasiswa dengan ID: ' . $id;
    }
    return 'Daftar Semua Mahasiswa';
});
```

### 5. Constraint Parameter dengan Regular Expression

Anda dapat membatasi format parameter menggunakan metode `where`:

```php
Route::get('/mahasiswa/{id}', function ($id) {
    return 'Detail Mahasiswa dengan ID: ' . $id;
})->where('id', '[0-9]+'); // Hanya menerima angka
```

Untuk constraint yang sering digunakan:

```php
Route::get('/mahasiswa/{nim}', function ($nim) {
    return 'Mahasiswa dengan NIM: ' . $nim;
})->whereAlphaNumeric('nim'); // Hanya menerima karakter alpha-numeric
```

## Routing ke Controller

### 1. Membuat Controller

Sebelum melanjutkan, buatlah controller untuk mahasiswa:

```bash
php artisan make:controller MahasiswaController
```

### 2. Routing ke Method Controller

```php
// routes/web.php
use App\Http\Controllers\MahasiswaController;

Route::get('/mahasiswa', [MahasiswaController::class, 'index']);
Route::get('/mahasiswa/{id}', [MahasiswaController::class, 'show']);
Route::post('/mahasiswa', [MahasiswaController::class, 'store']);
Route::put('/mahasiswa/{id}', [MahasiswaController::class, 'update']);
Route::delete('/mahasiswa/{id}', [MahasiswaController::class, 'destroy']);
```

### 3. Implementasi Controller

```php
// app/Http/Controllers/MahasiswaController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class MahasiswaController extends Controller
{
    public function index()
    {
        return 'Daftar semua mahasiswa';
    }

    public function show($id)
    {
        return 'Menampilkan detail mahasiswa dengan ID: ' . $id;
    }

    public function store(Request $request)
    {
        // Logic untuk menyimpan data mahasiswa baru
        return 'Mahasiswa baru berhasil ditambahkan';
    }

    public function update(Request $request, $id)
    {
        // Logic untuk memperbarui data mahasiswa
        return 'Data mahasiswa dengan ID ' . $id . ' berhasil diperbarui';
    }

    public function destroy($id)
    {
        // Logic untuk menghapus data mahasiswa
        return 'Mahasiswa dengan ID ' . $id . ' berhasil dihapus';
    }
}
```

## Route Groups

### 1. Group Berdasarkan Prefix

```php
Route::prefix('admin')->group(function () {
    Route::get('/mahasiswa', [MahasiswaController::class, 'index']); // URL: /admin/mahasiswa
    Route::get('/mahasiswa/{id}', [MahasiswaController::class, 'show']); // URL: /admin/mahasiswa/{id}
    // ...
});
```

### 2. Group Berdasarkan Middleware

```php
Route::middleware(['auth'])->group(function () {
    Route::get('/mahasiswa', [MahasiswaController::class, 'index']);
    Route::post('/mahasiswa', [MahasiswaController::class, 'store']);
    // ...
});
```

### 3. Group Berdasarkan Namespace (Laravel 8+)

```php
// Laravel 8+ menggunakan pendekatan yang berbeda
Route::group(['namespace' => 'App\Http\Controllers\Admin'], function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});
```

### 4. Group Berdasarkan Nama

```php
Route::name('mahasiswa.')->group(function () {
    Route::get('/mahasiswa', [MahasiswaController::class, 'index'])->name('index');
    Route::get('/mahasiswa/{id}', [MahasiswaController::class, 'show'])->name('show');
});
```

## Route dengan Nama (Named Routes)

### 1. Memberikan Nama pada Route

```php
Route::get('/mahasiswa', [MahasiswaController::class, 'index'])->name('mahasiswa.index');
```

### 2. Menggunakan Named Route dalam Aplikasi

Dalam controller:

```php
return redirect()->route('mahasiswa.index');
```

Dalam Blade view:

```php
<a href="{{ route('mahasiswa.index') }}">Daftar Mahasiswa</a>
```

Dengan parameter:

```php
<a href="{{ route('mahasiswa.show', ['id' => $mahasiswa->id]) }}">Detail</a>
```

## Resource Controller

### 1. Membuat Resource Controller

```bash
php artisan make:controller MahasiswaController --resource
```

### 2. Mendaftarkan Resource Route

```php
Route::resource('mahasiswa', MahasiswaController::class);
```

Perintah di atas akan secara otomatis menghasilkan 7 route untuk operasi CRUD:

| Verb      | URI                    | Action  | Route Name           |
|-----------|------------------------|---------|----------------------|
| GET       | /mahasiswa             | index   | mahasiswa.index      |
| GET       | /mahasiswa/create      | create  | mahasiswa.create     |
| POST      | /mahasiswa             | store   | mahasiswa.store      |
| GET       | /mahasiswa/{id}        | show    | mahasiswa.show       |
| GET       | /mahasiswa/{id}/edit   | edit    | mahasiswa.edit       |
| PUT/PATCH | /mahasiswa/{id}        | update  | mahasiswa.update     |
| DELETE    | /mahasiswa/{id}        | destroy | mahasiswa.destroy    |

### 3. Membatasi Resource Route

```php
// Hanya beberapa method
Route::resource('mahasiswa', MahasiswaController::class)->only(['index', 'show']);

// Kecuali beberapa method
Route::resource('mahasiswa', MahasiswaController::class)->except(['destroy']);
```

## Melihat Semua Route

Untuk melihat semua route yang terdaftar di aplikasi Laravel Anda:

```bash
php artisan route:list
```

## Contoh Praktis: Sistem Manajemen Mahasiswa

Mari kita buat contoh sistem routing lengkap untuk manajemen data mahasiswa:

```php
// routes/web.php
use App\Http\Controllers\MahasiswaController;
use App\Http\Controllers\JurusanController;
use Illuminate\Support\Facades\Route;

// Route untuk halaman beranda
Route::get('/', function () {
    return view('welcome');
});

// Route untuk autentikasi
Route::middleware(['auth'])->group(function () {
    // Dashboard
    Route::get('/dashboard', function () {
        return view('dashboard');
    })->name('dashboard');
    
    // Manajemen Mahasiswa
    Route::resource('mahasiswa', MahasiswaController::class);
    
    // Pencarian Mahasiswa
    Route::get('/mahasiswa/search', [MahasiswaController::class, 'search'])->name('mahasiswa.search');
    
    // Export Data Mahasiswa
    Route::get('/mahasiswa/export', [MahasiswaController::class, 'export'])->name('mahasiswa.export');
    
    // Jurusan Mahasiswa
    Route::resource('jurusan', JurusanController::class);
    
    // Mahasiswa berdasarkan Jurusan
    Route::get('/jurusan/{jurusan}/mahasiswa', [JurusanController::class, 'mahasiswa'])
        ->name('jurusan.mahasiswa');
});
```

## Kesimpulan

Routing di Laravel menyediakan cara yang kuat dan fleksibel untuk menangani permintaan ke aplikasi Anda. Dengan memahami konsep-konsep dasar routing, Anda dapat dengan mudah membangun aplikasi web yang terstruktur dan mudah dipelihara. 

Beberapa poin penting untuk diingat:
- Route didefinisikan di direktori `routes/`
- Gunakan HTTP verbs yang sesuai (GET, POST, PUT, DELETE)
- Manfaatkan parameter untuk data dinamis
- Gunakan route groups untuk mengorganisasi route
- Berikan nama pada route untuk referensi yang lebih mudah
- Gunakan resource controller untuk operasi CRUD standar

Dengan pemahaman yang baik tentang routing, Anda memiliki dasar yang kuat untuk mengembangkan aplikasi Laravel yang kompleks dan terstruktur dengan baik.