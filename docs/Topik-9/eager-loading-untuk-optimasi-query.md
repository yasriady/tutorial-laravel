# Eager Loading untuk Optimasi Query

Eager loading adalah teknik yang sangat penting dalam Laravel untuk mengatasi masalah N+1 queries yang kerap terjadi saat bekerja dengan relasi antar model. Pada bagian ini, kita akan mempelajari cara menggunakan eager loading untuk mengoptimalkan aplikasi manajemen data Mahasiswa.

## Memahami Masalah N+1 Query

Masalah N+1 query adalah salah satu masalah performa yang umum terjadi dalam pengembangan aplikasi web. Mari kita lihat contoh kasus pada aplikasi manajemen data Mahasiswa:

```php
// Tanpa eager loading - Menyebabkan masalah N+1 query
$mahasiswas = Mahasiswa::all(); // 1 query pertama

foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->jurusan->nama; // N query tambahan (1 untuk setiap mahasiswa)
}
```

Dalam contoh di atas, kita melakukan 1 query untuk mengambil semua data mahasiswa, kemudian untuk setiap mahasiswa, kita melakukan 1 query tambahan untuk mengambil data jurusan. Jika ada 100 mahasiswa, maka akan ada 101 query (1 + 100) yang dijalankan!

## Solusi: Eager Loading dengan `with()`

Laravel menyediakan metode `with()` untuk melakukan eager loading relasi:

```php
// Dengan eager loading - Hanya 2 query
$mahasiswas = Mahasiswa::with('jurusan')->get();

foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->jurusan->nama; // Tidak ada query tambahan!
}
```

Dengan eager loading, Laravel hanya menjalankan 2 query:
1. Query pertama untuk mengambil semua data mahasiswa
2. Query kedua untuk mengambil semua data jurusan yang terkait dengan mahasiswa tersebut

## Eager Loading Multiple Relationships

Kita dapat melakukan eager loading untuk beberapa relasi sekaligus:

```php
$mahasiswas = Mahasiswa::with(['jurusan', 'matakuliahs', 'pembimbing'])->get();

foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->jurusan->nama;
    echo $mahasiswa->pembimbing->nama;
    
    foreach ($mahasiswa->matakuliahs as $matakuliah) {
        echo $matakuliah->nama;
    }
}
```

## Nested Eager Loading

Untuk relasi bertingkat, kita dapat melakukan nested eager loading:

```php
// Mengambil mahasiswa dengan jurusan dan fakultas dari jurusan tersebut
$mahasiswas = Mahasiswa::with('jurusan.fakultas')->get();

foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->jurusan->fakultas->nama;
}
```

## Eager Loading dengan Kondisi

### Memfilter Relasi yang di-Load

```php
// Hanya eager load matakuliah dengan nilai di atas 80
$mahasiswas = Mahasiswa::with(['matakuliahs' => function($query) {
    $query->where('pivot.nilai', '>', 80);
}])->get();
```

### Memfilter Model Utama Berdasarkan Relasi

```php
// Mengambil mahasiswa yang memiliki nilai di atas 80 pada mata kuliah tertentu
$mahasiswas = Mahasiswa::whereHas('matakuliahs', function($query) {
    $query->where('matakuliahs.id', 1)
          ->where('pivot.nilai', '>', 80);
})->with('matakuliahs')->get();
```

## Lazy Eager Loading

Terkadang kita perlu melakukan eager loading setelah model sudah di-query:

```php
$mahasiswas = Mahasiswa::all();

// Jika ternyata kemudian kita membutuhkan data jurusan
if ($someCondition) {
    $mahasiswas->load('jurusan');
}
```

## Memuat Relasi Spesifik dengan Subquery Selects

Laravel 6.0+ mendukung eager loading relasi dengan subquery. Ini berguna untuk memuat kolom spesifik dari relasi:

```php
$mahasiswas = Mahasiswa::select('id', 'nama', 'nim')
    ->withCount('matakuliahs')
    ->with(['jurusan' => function($query) {
        $query->select('id', 'nama');
    }])
    ->get();
```

## Menampilkan Informasi Agregat dari Relasi

### Menghitung Jumlah Relasi

```php
// Menghitung jumlah matakuliah yang diambil mahasiswa
$mahasiswas = Mahasiswa::withCount('matakuliahs')->get();

foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->matakuliahs_count; // Jumlah matakuliah
}
```

### Menghitung Nilai Rata-rata

```php
// Menghitung rata-rata nilai mahasiswa
$mahasiswas = Mahasiswa::withAvg('matakuliahs as nilai_rata_rata', 'pivot.nilai')->get();

foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->nilai_rata_rata;
}
```

### Fungsi Agregat Lainnya

```php
$mahasiswas = Mahasiswa::withSum('matakuliahs as total_sks', 'sks')
    ->withMax('matakuliahs as nilai_tertinggi', 'pivot.nilai')
    ->withMin('matakuliahs as nilai_terendah', 'pivot.nilai')
    ->get();
```

## Optimasi Eager Loading untuk Relasi Many-to-Many

Dalam konteks aplikasi manajemen data Mahasiswa, relasi many-to-many antara Mahasiswa dan MataKuliah sangat umum:

```php
// Di model Mahasiswa
public function matakuliahs()
{
    return $this->belongsToMany(MataKuliah::class, 'mahasiswa_matakuliah')
                ->withPivot('nilai', 'semester')
                ->withTimestamps();
}

// Melakukan eager loading dengan data pivot
$mahasiswas = Mahasiswa::with('matakuliahs')->get();

foreach ($mahasiswas as $mahasiswa) {
    foreach ($mahasiswa->matakuliahs as $matakuliah) {
        echo "Nilai: " . $matakuliah->pivot->nilai;
        echo "Semester: " . $matakuliah->pivot->semester;
    }
}
```

## Optimasi Query dengan Eager Loading dan Query Builder

Kita dapat menggabungkan eager loading dengan query builder untuk query yang lebih kompleks:

```php
$mahasiswas = Mahasiswa::with(['jurusan', 'matakuliahs' => function($query) {
    $query->where('semester', 'Ganjil 2023/2024');
}])
->whereHas('jurusan', function($query) {
    $query->where('fakultas_id', 1);
})
->where('angkatan', 2022)
->orderBy('nim')
->paginate(20);
```

## Troubleshooting dan Debug Eager Loading

### Melihat Query yang Dieksekusi

Untuk debugging, kita dapat melihat query yang dihasilkan oleh eager loading:

```php
DB::enableQueryLog();

$mahasiswas = Mahasiswa::with('jurusan', 'matakuliahs')->get();

dd(DB::getQueryLog());
```

### Memperbarui Relasi dengan Eager Loading

```php
$mahasiswa = Mahasiswa::with('matakuliahs')->find(1);

// Memperbarui nilai untuk mata kuliah tertentu
$mahasiswa->matakuliahs()->updateExistingPivot($matakuliahId, [
    'nilai' => 90
]);
```

## Praktik Terbaik untuk Eager Loading

1. **Selalu gunakan eager loading** untuk relasi yang Anda akses dalam loop.
2. **Pertimbangkan untuk hanya memuat kolom yang diperlukan** untuk mengurangi ukuran memory:
   ```php
   $mahasiswas = Mahasiswa::with(['jurusan:id,nama', 'matakuliahs:id,kode,nama'])->get();
   ```
3. **Gunakan lazy eager loading** ketika Anda tidak yakin apakah relasi akan digunakan.
4. **Manfaatkan cache** untuk data yang jarang berubah:
   ```php
   $fakultas = cache()->remember('fakultas', now()->addHours(24), function() {
       return Fakultas::with('jurusans')->get();
   });
   ```
5. **Hindari over-eager loading** - hanya load relasi yang benar-benar Anda butuhkan.

## Mengukur Peningkatan Performa

Untuk membuktikan manfaat eager loading, lakukan pengukuran performa:

```php
$startTime = microtime(true);

// Query tanpa eager loading
$mahasiswas = Mahasiswa::all();
foreach ($mahasiswas as $mahasiswa) {
    $jurusan = $mahasiswa->jurusan;
}

$endTime = microtime(true);
echo "Tanpa eager loading: " . ($endTime - $startTime) . " detik";

// Reset
$startTime = microtime(true);

// Query dengan eager loading
$mahasiswas = Mahasiswa::with('jurusan')->get();
foreach ($mahasiswas as $mahasiswa) {
    $jurusan = $mahasiswa->jurusan;
}

$endTime = microtime(true);
echo "Dengan eager loading: " . ($endTime - $startTime) . " detik";
```

## Kesimpulan

Eager loading adalah teknik yang sangat penting untuk mengoptimalkan performa aplikasi Laravel, terutama untuk aplikasi dengan relasi data yang kompleks seperti sistem manajemen data Mahasiswa. Dengan menerapkan eager loading secara tepat, Anda dapat mengurangi waktu eksekusi query dan penggunaan sumber daya server secara signifikan.

Pada aplikasi manajemen data Mahasiswa, eager loading sangat bermanfaat ketika menampilkan data Mahasiswa beserta informasi terkait seperti Jurusan, Fakultas, dan Mata Kuliah. Implementasi yang tepat akan membuat aplikasi Anda lebih responsif dan mampu menangani jumlah data yang lebih besar tanpa menurunkan performa.