# Query Builder Lanjutan dalam Laravel

Query Builder Laravel menyediakan antarmuka yang intuitif untuk membuat dan menjalankan query database. Setelah mempelajari dasar-dasar Query Builder, kita akan memperluas pengetahuan dengan teknik-teknik yang lebih canggih untuk memaksimalkan kemampuan Query Builder dalam menangani kasus yang lebih kompleks.

## 1. Query Join Lanjutan

### Join Multiple Tables

```php
DB::table('mahasiswa')
    ->join('jurusan', 'mahasiswa.jurusan_id', '=', 'jurusan.id')
    ->join('fakultas', 'jurusan.fakultas_id', '=', 'fakultas.id')
    ->select('mahasiswa.*', 'jurusan.nama as jurusan', 'fakultas.nama as fakultas')
    ->get();
```

### Left Join dan Right Join

```php
// Left Join - menampilkan semua mahasiswa meskipun tidak memiliki jurusan
DB::table('mahasiswa')
    ->leftJoin('jurusan', 'mahasiswa.jurusan_id', '=', 'jurusan.id')
    ->select('mahasiswa.*', 'jurusan.nama as jurusan')
    ->get();

// Right Join - menampilkan semua jurusan meskipun tidak memiliki mahasiswa
DB::table('mahasiswa')
    ->rightJoin('jurusan', 'mahasiswa.jurusan_id', '=', 'jurusan.id')
    ->select('mahasiswa.*', 'jurusan.nama as jurusan')
    ->get();
```

### Cross Join

```php
// Menghasilkan cartesian product dari kedua tabel
DB::table('mahasiswa')
    ->crossJoin('jurusan')
    ->select('mahasiswa.nama', 'jurusan.nama as jurusan')
    ->get();
```

## 2. Subquery dan Derived Tables

### Subquery dalam Select

```php
DB::table('mahasiswa')
    ->select('nama', 'nim', DB::raw('(SELECT AVG(nilai) FROM nilai WHERE nilai.mahasiswa_id = mahasiswa.id) as rata_nilai'))
    ->get();
```

### Subquery dalam Where

```php
DB::table('mahasiswa')
    ->whereIn('jurusan_id', function($query) {
        $query->select('id')
              ->from('jurusan')
              ->where('fakultas_id', 1);
    })
    ->get();
```

### Subquery sebagai Derived Table

```php
DB::table(function($query) {
    $query->select('mahasiswa_id', DB::raw('AVG(nilai) as nilai_rata'))
          ->from('nilai')
          ->groupBy('mahasiswa_id')
          ->havingRaw('AVG(nilai) > 80');
}, 'nilai_tinggi')
    ->join('mahasiswa', 'nilai_tinggi.mahasiswa_id', '=', 'mahasiswa.id')
    ->select('mahasiswa.*', 'nilai_tinggi.nilai_rata')
    ->get();
```

## 3. Advanced Where Clauses

### WhereExists dan WhereNotExists

```php
DB::table('mahasiswa')
    ->whereExists(function($query) {
        $query->select(DB::raw(1))
              ->from('kehadiran')
              ->whereColumn('kehadiran.mahasiswa_id', 'mahasiswa.id')
              ->where('tanggal', today())
              ->limit(1);
    })
    ->get(); // Mahasiswa yang hadir hari ini
```

### Where dengan JSON Column (Laravel 5.6+)

```php
// Asumsikan ada kolom 'metadata' bertipe JSON
DB::table('mahasiswa')
    ->where('metadata->alamat->kota', 'Jakarta')
    ->get();
```

### Where Between Date Range

```php
DB::table('mahasiswa')
    ->whereBetween('tanggal_lahir', [
        now()->subYears(20),
        now()->subYears(18)
    ])
    ->get(); // Mahasiswa berusia 18-20 tahun
```

## 4. Advanced Group By dan Having

### Group By Multiple Columns

```php
DB::table('nilai')
    ->select('mahasiswa_id', 'mata_kuliah_id', DB::raw('AVG(nilai) as rata_nilai'))
    ->groupBy('mahasiswa_id', 'mata_kuliah_id')
    ->get();
```

### Having dengan Raw SQL

```php
DB::table('nilai')
    ->select('mahasiswa_id', DB::raw('COUNT(*) as jumlah_nilai'), DB::raw('AVG(nilai) as rata_nilai'))
    ->groupBy('mahasiswa_id')
    ->havingRaw('AVG(nilai) > ? AND COUNT(*) > ?', [75, 3])
    ->get();
```

## 5. Union Queries

```php
$mahasiswaAktif = DB::table('mahasiswa')
    ->select('id', 'nama', 'nim')
    ->where('status', 'aktif');

$mahasiswaCuti = DB::table('mahasiswa')
    ->select('id', 'nama', 'nim')
    ->where('status', 'cuti');

$hasil = $mahasiswaAktif->union($mahasiswaCuti)->get();
```

## 6. Pagination dan Chunk Processing

### Manual Pagination

```php
$perPage = 15;
$currentPage = request()->input('page', 1);
$offset = ($currentPage - 1) * $perPage;

$mahasiswa = DB::table('mahasiswa')
    ->offset($offset)
    ->limit($perPage)
    ->get();

$totalRecords = DB::table('mahasiswa')->count();
$totalPages = ceil($totalRecords / $perPage);
```

### Chunk Processing untuk Data Besar

```php
DB::table('mahasiswa')->orderBy('id')->chunk(100, function($mahasiswas) {
    foreach ($mahasiswas as $mahasiswa) {
        // Proses data mahasiswa
    }
    
    // Return false untuk menghentikan proses
    // return false;
});
```

## 7. Locking untuk Operasi Transactional

### Shared Lock (Lock for Read)

```php
DB::transaction(function() {
    $mahasiswa = DB::table('mahasiswa')
                   ->where('id', 1)
                   ->sharedLock()
                   ->first();
    
    // Operasi lainnya
});
```

### Exclusive Lock (Lock for Update)

```php
DB::transaction(function() {
    $mahasiswa = DB::table('mahasiswa')
                   ->where('id', 1)
                   ->lockForUpdate()
                   ->first();
    
    // Update data yang dikunci
    DB::table('mahasiswa')
        ->where('id', 1)
        ->update(['status' => 'lulus']);
});
```

## 8. Advanced Insert, Update, dan Delete

### Insert dengan Data yang Kompleks

```php
DB::table('mahasiswa')->insert([
    ['nama' => 'Ahmad', 'nim' => '10001', 'jurusan_id' => 1, 'created_at' => now()],
    ['nama' => 'Budi', 'nim' => '10002', 'jurusan_id' => 1, 'created_at' => now()],
    ['nama' => 'Citra', 'nim' => '10003', 'jurusan_id' => 2, 'created_at' => now()],
]);
```

### Update atau Insert (Upsert)

```php
DB::table('mahasiswa')->updateOrInsert(
    ['nim' => '10001'], // Kondisi pencarian
    ['nama' => 'Ahmad Revisi', 'jurusan_id' => 1, 'updated_at' => now()] // Data yang diupdate/diinsert
);
```

### Update dari Subquery

```php
DB::table('mahasiswa')
    ->where('jurusan_id', 1)
    ->update([
        'status' => 'alumni',
        'tahun_lulus' => DB::raw('(SELECT YEAR(CURDATE()))')
    ]);
```

### Delete dengan Join

```php
DB::table('mahasiswa')
    ->join('status_registrasi', 'mahasiswa.id', '=', 'status_registrasi.mahasiswa_id')
    ->where('status_registrasi.status', 'DO')
    ->where('status_registrasi.tahun', '<', 2020)
    ->delete();
```

## 9. Optimizing Queries

### Penggunaan Indexes dalam Query

```php
// Pastikan kolom 'nim' telah diindex pada migrasi
// $table->string('nim')->index();

// Query yang memanfaatkan index akan lebih cepat
$mahasiswa = DB::table('mahasiswa')
    ->where('nim', '10001')
    ->first();
```

### Selecting Only Required Columns

```php
// Lebih efisien daripada select *
DB::table('mahasiswa')
    ->select('id', 'nama', 'nim')
    ->where('jurusan_id', 1)
    ->get();
```

### Menggunakan Explain untuk Analisis Query

```php
DB::table('mahasiswa')
    ->join('jurusan', 'mahasiswa.jurusan_id', '=', 'jurusan.id')
    ->where('jurusan.nama', 'Informatika')
    ->select('mahasiswa.*')
    ->explain();
```

## 10. Kesimpulan dan Best Practices

1. **Gunakan Query Builder dengan Bijak** - Untuk query kompleks, kadang Raw SQL lebih mudah dibaca.
2. **Pertimbangkan Performa** - Selalu pertimbangkan dampak performa pada database saat menulis query.
3. **Hindari N+1 Problem** - Gunakan eager loading atau join untuk menghindari multiple query.
4. **Gunakan Caching untuk Query Berat** - Cache hasil query yang komputasinya berat dan jarang berubah.
5. **Validasi Input** - Selalu validasi input untuk menghindari SQL injection.

Query Builder Laravel adalah alat yang sangat kuat untuk mengelola data dalam aplikasi Anda. Dengan memahami teknik lanjutan, Anda dapat mengoptimalkan operasi database dan membuat fitur yang lebih kompleks dengan kode yang lebih bersih dan terstruktur.