# Praktik: Membuat relasi Mahasiswa-Mata Kuliah

Pada bagian ini, kita akan mempraktikkan cara membuat dan mengelola relasi Many-to-Many antara Mahasiswa dan Mata Kuliah. Kita akan melalui langkah demi langkah mulai dari pembuatan migrasi, model, hingga implementasi CRUD untuk relasi tersebut.

## 1. Persiapan Database

### Membuat Migrasi untuk Tabel Mata Kuliah

Pertama, kita perlu membuat tabel Mata Kuliah:

```bash
php artisan make:migration create_mata_kuliahs_table --create=mata_kuliahs
```

Kemudian, isi file migrasi tersebut:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateMataKuliahsTable extends Migration
{
    public function up()
    {
        Schema::create('mata_kuliahs', function (Blueprint $table) {
            $table->id();
            $table->string('kode')->unique();
            $table->string('nama');
            $table->integer('sks');
            $table->string('semester');
            $table->text('deskripsi')->nullable();
            $table->foreignId('jurusan_id')->constrained('jurusans')->onDelete('cascade');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('mata_kuliahs');
    }
}
```

### Membuat Migrasi untuk Tabel Pivot Mahasiswa-Mata Kuliah

Selanjutnya, kita perlu membuat tabel pivot untuk relasi Many-to-Many:

```bash
php artisan make:migration create_mahasiswa_mata_kuliah_table
```

Isi file migrasi tabel pivot:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateMahasiswaMataKuliahTable extends Migration
{
    public function up()
    {
        Schema::create('mahasiswa_mata_kuliah', function (Blueprint $table) {
            $table->id();
            $table->foreignId('mahasiswa_id')->constrained()->onDelete('cascade');
            $table->foreignId('mata_kuliah_id')->constrained('mata_kuliahs')->onDelete('cascade');
            $table->string('semester_ambil');
            $table->integer('nilai')->nullable();
            $table->string('status')->default('aktif'); // aktif, selesai, mengulang
            $table->timestamps();

            // Unique untuk mencegah mahasiswa mengambil mata kuliah yang sama pada semester yang sama
            $table->unique(['mahasiswa_id', 'mata_kuliah_id', 'semester_ambil']);
        });
    }

    public function down()
    {
        Schema::dropIfExists('mahasiswa_mata_kuliah');
    }
}
```

### Menjalankan Migrasi

```bash
php artisan migrate
```

## 2. Membuat Model MataKuliah

Kita perlu membuat model untuk MataKuliah:

```bash
php artisan make:model MataKuliah
```

Kemudian, isi model tersebut:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class MataKuliah extends Model
{
    use HasFactory;

    protected $fillable = [
        'kode',
        'nama',
        'sks',
        'semester',
        'deskripsi',
        'jurusan_id'
    ];

    /**
     * Relasi ke jurusan
     */
    public function jurusan()
    {
        return $this->belongsTo(Jurusan::class);
    }

    /**
     * Relasi ke mahasiswa
     */
    public function mahasiswas()
    {
        return $this->belongsToMany(Mahasiswa::class, 'mahasiswa_mata_kuliah')
                    ->withPivot('semester_ambil', 'nilai', 'status')
                    ->withTimestamps();
    }
}
```

## 3. Memperbarui Model Mahasiswa

Selanjutnya, kita perlu memperbarui model Mahasiswa untuk menambahkan relasi dengan MataKuliah:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Mahasiswa extends Model
{
    use HasFactory;

    protected $fillable = [
        'nim',
        'nama',
        'email',
        'tanggal_lahir',
        'alamat',
        'jurusan_id'
    ];

    // Relasi ke jurusan
    public function jurusan()
    {
        return $this->belongsTo(Jurusan::class);
    }

    // Tambahkan relasi ke mata kuliah
    public function mataKuliahs()
    {
        return $this->belongsToMany(MataKuliah::class, 'mahasiswa_mata_kuliah')
                    ->withPivot('semester_ambil', 'nilai', 'status')
                    ->withTimestamps();
    }
    
    // Menghitung IPK
    public function hitungIPK()
    {
        $mataKuliahs = $this->mataKuliahs()
                            ->wherePivot('status', 'selesai')
                            ->wherePivot('nilai', '>=', 0)
                            ->get();
        
        if ($mataKuliahs->isEmpty()) {
            return 0;
        }
        
        $totalNilai = 0;
        $totalSKS = 0;
        
        foreach ($mataKuliahs as $mk) {
            // Konversi nilai angka ke bobot
            $bobot = $this->konversiNilaiKeBobot($mk->pivot->nilai);
            $totalNilai += ($bobot * $mk->sks);
            $totalSKS += $mk->sks;
        }
        
        return $totalSKS > 0 ? round($totalNilai / $totalSKS, 2) : 0;
    }
    
    // Helper untuk konversi nilai ke bobot
    private function konversiNilaiKeBobot($nilai)
    {
        if ($nilai >= 85) return 4.0;
        if ($nilai >= 80) return 3.7;
        if ($nilai >= 75) return 3.3;
        if ($nilai >= 70) return 3.0;
        if ($nilai >= 65) return 2.7;
        if ($nilai >= 60) return 2.3;
        if ($nilai >= 55) return 2.0;
        if ($nilai >= 50) return 1.7;
        if ($nilai >= 40) return 1.0;
        return 0;
    }
}
```

## 4. Membuat Seed untuk Data Dummy

Untuk mempermudah pengujian, kita buat seed untuk data dummy:

```bash
php artisan make:seeder MataKuliahSeeder
```

Isi file seeder:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\MataKuliah;

class MataKuliahSeeder extends Seeder
{
    public function run()
    {
        // Asumsi jurusan_id 1 adalah Teknik Informatika
        $mataKuliahs = [
            [
                'kode' => 'IF101',
                'nama' => 'Algoritma dan Pemrograman',
                'sks' => 4,
                'semester' => 'Ganjil',
                'deskripsi' => 'Mata kuliah dasar pemrograman',
                'jurusan_id' => 1
            ],
            [
                'kode' => 'IF102',
                'nama' => 'Struktur Data',
                'sks' => 3,
                'semester' => 'Genap',
                'deskripsi' => 'Pengenalan struktur data dasar',
                'jurusan_id' => 1
            ],
            [
                'kode' => 'IF201',
                'nama' => 'Basis Data',
                'sks' => 3,
                'semester' => 'Ganjil',
                'deskripsi' => 'Pengenalan database relasional',
                'jurusan_id' => 1
            ],
            [
                'kode' => 'IF202',
                'nama' => 'Pemrograman Web',
                'sks' => 4,
                'semester' => 'Genap',
                'deskripsi' => 'Pembuatan aplikasi berbasis web',
                'jurusan_id' => 1
            ],
            [
                'kode' => 'IF301',
                'nama' => 'Framework Laravel',
                'sks' => 3,
                'semester' => 'Ganjil',
                'deskripsi' => 'Pengembangan web menggunakan Laravel',
                'jurusan_id' => 1
            ],
        ];

        foreach ($mataKuliahs as $mk) {
            MataKuliah::create($mk);
        }
    }
}
```

Kemudian, tambahkan seeder ini ke database seeder:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call([
            // Seeder lainnya...
            MataKuliahSeeder::class,
        ]);
    }
}
```

Jalankan seeder:

```bash
php artisan db:seed --class=MataKuliahSeeder
```

## 5. Membuat Controller untuk Manajemen Relasi

### Membuat MataKuliahController

```bash
php artisan make:controller MataKuliahController --resource
```

### Membuat MahasiswaMataKuliahController

```bash
php artisan make:controller MahasiswaMataKuliahController
```

Isi MahasiswaMataKuliahController:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Mahasiswa;
use App\Models\MataKuliah;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class MahasiswaMataKuliahController extends Controller
{
    /**
     * Menampilkan mata kuliah yang diambil oleh mahasiswa
     */
    public function index($mahasiswaId)
    {
        $mahasiswa = Mahasiswa::with(['mataKuliahs' => function($query) {
            $query->orderBy('semester');
        }])->findOrFail($mahasiswaId);
        
        $ipk = $mahasiswa->hitungIPK();
        
        return view('mahasiswa_matakuliah.index', compact('mahasiswa', 'ipk'));
    }
    
    /**
     * Menampilkan form untuk menambah mata kuliah pada mahasiswa
     */
    public function create($mahasiswaId)
    {
        $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
        $mataKuliahs = MataKuliah::where('jurusan_id', $mahasiswa->jurusan_id)
                                 ->orderBy('semester')
                                 ->orderBy('nama')
                                 ->get();
        
        // Daftar semester untuk dropdown
        $semesterOptions = [
            'Ganjil 2022/2023',
            'Genap 2022/2023',
            'Ganjil 2023/2024',
            'Genap 2023/2024',
            'Ganjil 2024/2025',
            'Genap 2024/2025',
        ];
        
        return view('mahasiswa_matakuliah.create', compact('mahasiswa', 'mataKuliahs', 'semesterOptions'));
    }
    
    /**
     * Menyimpan relasi mata kuliah yang diambil mahasiswa
     */
    public function store(Request $request, $mahasiswaId)
    {
        $validator = Validator::make($request->all(), [
            'mata_kuliah_id' => 'required|exists:mata_kuliahs,id',
            'semester_ambil' => 'required|string',
        ]);
        
        if ($validator->fails()) {
            return redirect()->back()
                             ->withErrors($validator)
                             ->withInput();
        }
        
        $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
        
        // Cek apakah mahasiswa sudah mengambil mata kuliah ini di semester yang sama
        $exists = $mahasiswa->mataKuliahs()
                            ->wherePivot('mata_kuliah_id', $request->mata_kuliah_id)
                            ->wherePivot('semester_ambil', $request->semester_ambil)
                            ->exists();
        
        if ($exists) {
            return redirect()->back()
                            ->with('error', 'Mahasiswa sudah mengambil mata kuliah ini pada semester tersebut.');
        }
        
        // Tambahkan relasi
        $mahasiswa->mataKuliahs()->attach($request->mata_kuliah_id, [
            'semester_ambil' => $request->semester_ambil,
            'status' => 'aktif'
        ]);
        
        return redirect()->route('mahasiswa.matakuliah.index', $mahasiswaId)
                        ->with('success', 'Mata kuliah berhasil ditambahkan.');
    }
    
    /**
     * Menampilkan form untuk input nilai
     */
    public function editNilai($mahasiswaId, $mataKuliahId, $semesterAmbil)
    {
        $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
        $mataKuliah = MataKuliah::findOrFail($mataKuliahId);
        
        $enrollment = $mahasiswa->mataKuliahs()
                                ->wherePivot('mata_kuliah_id', $mataKuliahId)
                                ->wherePivot('semester_ambil', $semesterAmbil)
                                ->first();
        
        if (!$enrollment) {
            return redirect()->back()->with('error', 'Data tidak ditemukan.');
        }
        
        return view('mahasiswa_matakuliah.edit_nilai', compact('mahasiswa', 'mataKuliah', 'enrollment', 'semesterAmbil'));
    }
    
    /**
     * Menyimpan nilai mata kuliah
     */
    public function updateNilai(Request $request, $mahasiswaId, $mataKuliahId, $semesterAmbil)
    {
        $validator = Validator::make($request->all(), [
            'nilai' => 'required|integer|min:0|max:100',
            'status' => 'required|in:aktif,selesai,mengulang',
        ]);
        
        if ($validator->fails()) {
            return redirect()->back()
                             ->withErrors($validator)
                             ->withInput();
        }
        
        $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
        
        // Update nilai dan status
        $mahasiswa->mataKuliahs()->updateExistingPivot($mataKuliahId, [
            'nilai' => $request->nilai,
            'status' => $request->status,
        ], false, [
            'semester_ambil' => $semesterAmbil
        ]);
        
        return redirect()->route('mahasiswa.matakuliah.index', $mahasiswaId)
                        ->with('success', 'Nilai berhasil diperbarui.');
    }
    
    /**
     * Menghapus mata kuliah dari daftar yang diambil mahasiswa
     */
    public function destroy($mahasiswaId, $mataKuliahId, $semesterAmbil)
    {
        $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
        
        // Cari dan hapus record spesifik berdasarkan semester_ambil
        $mahasiswa->mataKuliahs()->wherePivot('mata_kuliah_id', $mataKuliahId)
                                 ->wherePivot('semester_ambil', $semesterAmbil)
                                 ->detach();
        
        return redirect()->route('mahasiswa.matakuliah.index', $mahasiswaId)
                        ->with('success', 'Mata kuliah berhasil dihapus dari daftar.');
    }
    
    /**
     * Melihat transkrip nilai
     */
    public function transkrip($mahasiswaId)
    {
        $mahasiswa = Mahasiswa::with(['mataKuliahs' => function($query) {
            $query->wherePivot('status', 'selesai')
                  ->orderBy('semester')
                  ->orderBy('nama');
        }, 'jurusan'])->findOrFail($mahasiswaId);
        
        $ipk = $mahasiswa->hitungIPK();
        
        // Nilai per semester
        $nilaiPerSemester = [];
        
        foreach ($mahasiswa->mataKuliahs as $mk) {
            $semesterAmbil = $mk->pivot->semester_ambil;
            
            if (!isset($nilaiPerSemester[$semesterAmbil])) {
                $nilaiPerSemester[$semesterAmbil] = [
                    'mata_kuliahs' => [],
                    'total_sks' => 0,
                    'total_nilai' => 0,
                ];
            }
            
            $bobot = $mahasiswa->konversiNilaiKeBobot($mk->pivot->nilai);
            $nilaiPoint = $bobot * $mk->sks;
            
            $nilaiPerSemester[$semesterAmbil]['mata_kuliahs'][] = $mk;
            $nilaiPerSemester[$semesterAmbil]['total_sks'] += $mk->sks;
            $nilaiPerSemester[$semesterAmbil]['total_nilai'] += $nilaiPoint;
        }
        
        // Hitung IPS (Indeks Prestasi Semester)
        foreach ($nilaiPerSemester as $semester => &$data) {
            $data['ips'] = $data['total_sks'] > 0 ? 
                round($data['total_nilai'] / $data['total_sks'], 2) : 0;
        }
        
        return view('mahasiswa_matakuliah.transkrip', compact('mahasiswa', 'ipk', 'nilaiPerSemester'));
    }
}
```

## 6. Membuat Routes

Tambahkan route berikut di `routes/web.php`:

```php
// Route untuk manajemen mata kuliah
Route::resource('mata-kuliah', MataKuliahController::class);

// Route untuk relasi mahasiswa-mata kuliah
Route::get('mahasiswa/{mahasiswa}/mata-kuliah', [MahasiswaMataKuliahController::class, 'index'])
     ->name('mahasiswa.matakuliah.index');
     
Route::get('mahasiswa/{mahasiswa}/mata-kuliah/create', [MahasiswaMataKuliahController::class, 'create'])
     ->name('mahasiswa.matakuliah.create');
     
Route::post('mahasiswa/{mahasiswa}/mata-kuliah', [MahasiswaMataKuliahController::class, 'store'])
     ->name('mahasiswa.matakuliah.store');
     
Route::get('mahasiswa/{mahasiswa}/mata-kuliah/{mataKuliah}/{semester}/edit-nilai', [MahasiswaMataKuliahController::class, 'editNilai'])
     ->name('mahasiswa.matakuliah.edit-nilai');
     
Route::put('mahasiswa/{mahasiswa}/mata-kuliah/{mataKuliah}/{semester}', [MahasiswaMataKuliahController::class, 'updateNilai'])
     ->name('mahasiswa.matakuliah.update-nilai');
     
Route::delete('mahasiswa/{mahasiswa}/mata-kuliah/{mataKuliah}/{semester}', [MahasiswaMataKuliahController::class, 'destroy'])
     ->name('mahasiswa.matakuliah.destroy');
     
Route::get('mahasiswa/{mahasiswa}/transkrip', [MahasiswaMataKuliahController::class, 'transkrip'])
     ->name('mahasiswa.transkrip');
```

## 7. Membuat View untuk Manajemen Relasi

### View Index - resources/views/mahasiswa_matakuliah/index.blade.php

```php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-12 mb-4">
            <div class="d-flex justify-content-between align-items-center">
                <h1>Daftar Mata Kuliah - {{ $mahasiswa->nama }} ({{ $mahasiswa->nim }})</h1>
                <div>
                    <a href="{{ route('mahasiswa.matakuliah.create', $mahasiswa->id) }}" class="btn btn-primary">Tambah Mata Kuliah</a>
                    <a href="{{ route('mahasiswa.transkrip', $mahasiswa->id) }}" class="btn btn-info">Lihat Transkrip</a>
                    <a href="{{ route('mahasiswa.index') }}" class="btn btn-secondary">Kembali</a>
                </div>
            </div>
            <p>Program Studi: {{ $mahasiswa->jurusan->nama }}</p>
            <p>IPK: {{ $ipk }}</p>
        </div>
    </div>

    @if(session('success'))
    <div class="alert alert-success">
        {{ session('success') }}
    </div>
    @endif

    @if(session('error'))
    <div class="alert alert-danger">
        {{ session('error') }}
    </div>
    @endif

    <div class="card">
        <div class="card-header">
            <h5>Mata Kuliah yang Diambil</h5>
        </div>
        <div class="card-body">
            <table class="table table-striped">
                <thead>
                    <tr>
                        <th>Kode</th>
                        <th>Nama Mata Kuliah</th>
                        <th>SKS</th>
                        <th>Semester</th>
                        <th>Semester Ambil</th>
                        <th>Nilai</th>
                        <th>Status</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    @forelse($mahasiswa->mataKuliahs as $mk)
                    <tr>
                        <td>{{ $mk->kode }}</td>
                        <td>{{ $mk->nama }}</td>
                        <td>{{ $mk->sks }}</td>
                        <td>{{ $mk->semester }}</td>
                        <td>{{ $mk->pivot->semester_ambil }}</td>
                        <td>{{ $mk->pivot->nilai ?? '-' }}</td>
                        <td>
                            @if($mk->pivot->status == 'aktif')
                                <span class="badge bg-primary">Aktif</span>
                            @elseif($mk->pivot->status == 'selesai')
                                <span class="badge bg-success">Selesai</span>
                            @elseif($mk->pivot->status == 'mengulang')
                                <span class="badge bg-warning">Mengulang</span>
                            @endif
                        </td>
                        <td>
                            <a href="{{ route('mahasiswa.matakuliah.edit-nilai', [$mahasiswa->id, $mk->id, $mk->pivot->semester_ambil]) }}" class="btn btn-sm btn-info">Edit Nilai</a>
                            
                            <form action="{{ route('mahasiswa.matakuliah.destroy', [$mahasiswa->id, $mk->id, $mk->pivot->semester_ambil]) }}" method="POST" class="d-inline">
                                @csrf
                                @method('DELETE')
                                <button type="submit" class="btn btn-sm btn-danger" onclick="return confirm('Apakah Anda yakin ingin menghapus mata kuliah ini?')">Hapus</button>
                            </form>
                        </td>
                    </tr>
                    @empty
                    <tr>
                        <td colspan="8" class="text-center">Belum ada mata kuliah yang diambil</td>
                    </tr>
                    @endforelse
                </tbody>
            </table>
        </div>
    </div>
</div>
@endsection
```

### Form Tambah Mata Kuliah - resources/views/mahasiswa_matakuliah/create.blade.php

```php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-12 mb-4">
            <h1>Tambah Mata Kuliah - {{ $mahasiswa->nama }} ({{ $mahasiswa->nim }})</h1>
            <p>Program Studi: {{ $mahasiswa->jurusan->nama }}</p>
        </div>
    </div>

    @if(session('error'))
    <div class="alert alert-danger">
        {{ session('error') }}
    </div>
    @endif

    <div class="card">
        <div class="card-header">
            <h5>Form Tambah Mata Kuliah</h5>
        </div>
        <div class="card-body">
            <form action="{{ route('mahasiswa.matakuliah.store', $mahasiswa->id) }}" method="POST">
                @csrf

                <div class="mb-3">
                    <label for="mata_kuliah_id" class="form-label">Mata Kuliah</label>
                    <select name="mata_kuliah_id" id="mata_kuliah_id" class="form-control @error('mata_kuliah_id') is-invalid @enderror" required>
                        <option value="">-- Pilih Mata Kuliah --</option>
                        @foreach($mataKuliahs as $mk)
                        <option value="{{ $mk->id }}">{{ $mk->kode }} - {{ $mk->nama }} ({{ $mk->sks }} SKS, Semester {{ $mk->semester }})</option>
                        @endforeach
                    </select>
                    @error('mata_kuliah_id')
                    <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-3">
                    <label for="semester_ambil" class="form-label">Semester Pengambilan</label>
                    <select name="semester_ambil" id="semester_ambil" class="form-control @error('semester_ambil') is-invalid @enderror" required>
                        <option value="">-- Pilih Semester --</option>
                        @foreach($semesterOptions as $semester)
                        <option value="{{ $semester }}">{{ $semester }}</option>
                        @endforeach
                    </select>
                    @error('semester_ambil')
                    <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-3">
                    <button type="submit" class="btn btn-primary">Simpan</button>
                    <a href="{{ route('mahasiswa.matakuliah.index', $mahasiswa->id) }}" class="btn btn-secondary">Batal</a>
                </div>
            </form>
        </div>
    </div>
</div>
@endsection
```

### Form Edit Nilai - resources/views/mahasiswa_matakuliah/edit_nilai.blade.php

```php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-12 mb-4">
            <h1>Input Nilai Mata Kuliah</h1>
            <p>Mahasiswa: {{ $mahasiswa->nama }} ({{ $mahasiswa->nim }})</p>
            <p>Mata Kuliah: {{ $mataKuliah->kode }} - {{ $mataKuliah->nama }} ({{ $mataKuliah->sks }} SKS)</p>
            <p>Semester Pengambilan: {{ $semesterAmbil }}</p>
        </div>
    </div>

    <div class="card">
        <div class="card-header">
            <h5>Form Input Nilai</h5>
        </div>
        <div class="card-body">
            <form action="{{ route('mahasiswa.matakuliah.update-nilai', [$mahasiswa->id, $mataKuliah->id, $semesterAmbil]) }}" method="POST">
                @csrf
                @method('PUT')

                <div class="mb-3">
                    <label for="nilai" class="form-label">Nilai (0-100)</label>
                    <input type="number" name="nilai" id="nilai" class="form-control @error('nilai') is-invalid @enderror" value="{{ old('nilai', $enrollment->pivot->nilai) }}" min="0" max="100" required>
                    @error('nilai')
                    <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-3">
                    <label for="status" class="form-label">Status</label>
                    <select name="status" id="status" class="form-control @error('status') is-invalid @enderror" required>
                        <option value="aktif" {{ $enrollment->pivot->status == 'aktif' ? 'selected' : '' }}>Aktif</option>
                        <option value="selesai" {{ $enrollment->pivot->status == 'selesai' ? 'selected' : '' }}>Selesai</option>
                        <option value="mengulang" {{ $enrollment->pivot->status == 'mengulang' ? 'selected' : '' }}>Mengulang</option>
                    </select>
                    @error('status')
                    <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-3">
                    <button type="submit" class="btn btn-primary">Simpan</button>
                    <a href="{{ route('mahasiswa.matakuliah.index', $mahasiswa->id) }}" class="btn btn-secondary">Batal</a>
                </div>
            </form>
        </div>
    </div>
</div>
@endsection
```

### View Transkrip - resources/views/mahasiswa_matakuliah/transkrip.blade.php

```php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-12 mb-4">
            <div class="d-flex justify-content-between align-items-center">
                <h1>Transkrip Nilai</h1>
                <a href="{{ route('mahasiswa.matakuliah.index', $mahasiswa->id) }}" class="btn btn-secondary">Kembali</a>
            </div>
        </div>
    </div>

    <div class="card mb-4">
        <div class="card-header">
            <h5>Data Mahasiswa</h5>
        </div>
        <div class="card-body">
            <table class="table table-bordered">
                <tr>
                    <th width="200">Nama</th>
                    <td>{{ $mahasiswa->nama }}</td>
                </tr>
                <tr>
                    <th>NIM</th>
                    <td>{{ $mahasiswa->nim }}</td>
                </tr>
                <tr>
                    <th>Program Studi</th>
                    <td>{{ $mahasiswa->jurusan->nama }}</td>
                </tr>
                <tr>
                    <th>IPK</th>
                    <td><strong>{{ $ipk }}</strong></td>
                </tr>
            </table>
        </div>
    </div>

    @if(count($nilaiPerSemester) > 0)
        @foreach($nilaiPerSemester as $semester => $data)
        <div class="card mb-4">
            <div class="card-header bg-light">
                <h5>{{ $semester }}</h5>
            </div>
            <div class="card-body">
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>Kode</th>
                            <th>Mata Kuliah</th>
                            <th>SKS</th>
                            <th>Nilai</th>
                            <th>Bobot</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach($data['mata_kuliahs'] as $mk)
                        <tr>
                            <td>{{ $mk->kode }}</td>
                            <td>{{ $mk->nama }}</td>
                            <td>{{ $mk->sks }}</td>
                            <td>{{ $mk->pivot->nilai }}</td>
                            <td>
                                @php
                                    $bobot = $mahasiswa->konversiNilaiKeBobot($mk->pivot->nilai);
                                    echo $bobot;
                                @endphp
                            </td>
                        </tr>
                        @endforeach
                    </tbody>
                    <tfoot>
                        <tr>
                            <th colspan="2">Total</th>
                            <th>{{ $data['total_sks'] }} SKS</th>
                            <th colspan="2">IPS: {{ $data['ips'] }}</th>
                        </tr>
                    </tfoot>
                </table>
            </div>
        </div>
        @endforeach
    @else
        <div class="alert alert-info">
            Belum ada data nilai yang tersedia untuk ditampilkan dalam transkrip.
        </div>
    @endif
</div>
@endsection
```

## 8. Testing Integrasi

Setelah semua kode di atas diimplementasikan, kita perlu melakukan testing integrasi untuk memastikan bahwa relasi Mahasiswa-Mata Kuliah berfungsi dengan baik. Berikut langkah-langkah testing yang bisa dilakukan:

### 8.1 Menambahkan Mata Kuliah pada Mahasiswa

1. Akses halaman detail mahasiswa
2. Klik tombol "Daftar Mata Kuliah"
3. Klik tombol "Tambah Mata Kuliah"
4. Pilih mata kuliah dari dropdown
5. Pilih semester pengambilan
6. Klik tombol "Simpan"
7. Verifikasi mata kuliah berhasil ditambahkan

### 8.2 Memasukkan Nilai Mata Kuliah

1. Akses halaman daftar mata kuliah mahasiswa
2. Klik tombol "Edit Nilai" pada mata kuliah yang sudah diambil
3. Input nilai (0-100)
4. Pilih status (aktif, selesai, mengulang)
5. Klik tombol "Simpan"
6. Verifikasi nilai berhasil disimpan

### 8.3 Melihat Transkrip Nilai

1. Akses halaman daftar mata kuliah mahasiswa
2. Klik tombol "Lihat Transkrip"
3. Verifikasi semua mata kuliah dengan status "selesai" ditampilkan dengan benar
4. Verifikasi perhitungan IPS per semester dan IPK total sudah benar

### 8.4 Menghapus Mata Kuliah

1. Akses halaman daftar mata kuliah mahasiswa
2. Klik tombol "Hapus" pada mata kuliah yang ingin dihapus
3. Konfirmasi penghapusan
4. Verifikasi mata kuliah berhasil dihapus dari daftar

## 9. Perluasan Fitur

Setelah implementasi dasar, kita bisa memperluas fitur dengan beberapa tambahan berikut:

### 9.1 Batasan Pengambilan SKS

Tambahkan fitur untuk membatasi jumlah SKS yang bisa diambil berdasarkan IP semester sebelumnya:

```php
// Tambahkan di MahasiswaMataKuliahController
public function getMaksimumSKS($mahasiswaId, $semesterAmbil)
{
    $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
    
    // Cari semester sebelumnya dari format semester saat ini (misalnya: "Ganjil 2023/2024")
    $semesterTerakhir = $this->getSemesterSebelumnya($semesterAmbil);
    
    // Hitung IPS semester terakhir
    $ipsSemesterTerakhir = $this->hitungIPSSemester($mahasiswaId, $semesterTerakhir);
    
    // Tentukan maksimum SKS berdasarkan IPS
    if ($ipsSemesterTerakhir >= 3.5) {
        return 24;
    } elseif ($ipsSemesterTerakhir >= 3.0) {
        return 22;
    } elseif ($ipsSemesterTerakhir >= 2.5) {
        return 20;
    } elseif ($ipsSemesterTerakhir >= 2.0) {
        return 18;
    } else {
        return 16;
    }
}
```

### 9.2 Prasyarat Mata Kuliah

Tambahkan fitur untuk memeriksa prasyarat mata kuliah:

```php
// Tambahkan relasi di model MataKuliah
public function prasyarats()
{
    return $this->belongsToMany(MataKuliah::class, 'mata_kuliah_prasyarat', 'mata_kuliah_id', 'prasyarat_id');
}

// Cek apakah mahasiswa memenuhi prasyarat
public function checkPrasyarat($mahasiswaId, $mataKuliahId)
{
    $mataKuliah = MataKuliah::with('prasyarats')->findOrFail($mataKuliahId);
    $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
    
    // Jika tidak ada prasyarat
    if ($mataKuliah->prasyarats->isEmpty()) {
        return true;
    }
    
    $prasyaratTerpenuhi = true;
    
    foreach ($mataKuliah->prasyarats as $prasyarat) {
        // Cek apakah mahasiswa sudah lulus mata kuliah prasyarat
        $sudahLulus = $mahasiswa->mataKuliahs()
                                ->wherePivot('mata_kuliah_id', $prasyarat->id)
                                ->wherePivot('status', 'selesai')
                                ->wherePivot('nilai', '>=', 60) // Nilai minimal untuk lulus
                                ->exists();
        
        if (!$sudahLulus) {
            $prasyaratTerpenuhi = false;
            break;
        }
    }
    
    return $prasyaratTerpenuhi;
}
```

### 9.3 Riwayat Akademik

Tambahkan fitur untuk melihat riwayat akademik secara lengkap:

```php
// Tambahkan di MahasiswaMataKuliahController
public function riwayatAkademik($mahasiswaId)
{
    $mahasiswa = Mahasiswa::with(['mataKuliahs', 'jurusan'])->findOrFail($mahasiswaId);
    
    // Kelompokkan berdasarkan semester
    $riwayatSemester = [];
    
    foreach ($mahasiswa->mataKuliahs as $mk) {
        $semesterAmbil = $mk->pivot->semester_ambil;
        
        if (!isset($riwayatSemester[$semesterAmbil])) {
            $riwayatSemester[$semesterAmbil] = [
                'mata_kuliahs' => [],
                'total_sks' => 0,
                'ips' => 0,
            ];
        }
        
        $riwayatSemester[$semesterAmbil]['mata_kuliahs'][] = $mk;
        $riwayatSemester[$semesterAmbil]['total_sks'] += $mk->sks;
    }
    
    // Hitung IPS untuk setiap semester
    foreach ($riwayatSemester as $semester => &$data) {
        $data['ips'] = $this->hitungIPSSemester($mahasiswaId, $semester);
    }
    
    // Hitung IPK
    $ipk = $mahasiswa->hitungIPK();
    
    return view('mahasiswa_matakuliah.riwayat_akademik', compact('mahasiswa', 'riwayatSemester', 'ipk'));
}
```

## 10. Best Practices dan Tips

### 10.1 Validasi Input

Selalu validasi input dari pengguna untuk memastikan integritas data:

```php
// Contoh validasi untuk penambahan mata kuliah
$validator = Validator::make($request->all(), [
    'mata_kuliah_id' => 'required|exists:mata_kuliahs,id',
    'semester_ambil' => 'required|string',
]);

if ($validator->fails()) {
    return redirect()->back()
                     ->withErrors($validator)
                     ->withInput();
}
```

### 10.2 Gunakan Transaction untuk Update Multiple Records

Gunakan transaction database untuk memastikan konsistensi data ketika melakukan update multiple records:

```php
DB::transaction(function() use ($mahasiswaId, $request) {
    // Hapus mata kuliah lama
    DB::table('mahasiswa_mata_kuliah')
        ->where('mahasiswa_id', $mahasiswaId)
        ->where('semester_ambil', $request->semester)
        ->delete();
    
    // Tambahkan mata kuliah baru
    foreach ($request->mata_kuliah_ids as $mkId) {
        DB::table('mahasiswa_mata_kuliah')->insert([
            'mahasiswa_id' => $mahasiswaId,
            'mata_kuliah_id' => $mkId,
            'semester_ambil' => $request->semester,
            'created_at' => now(),
            'updated_at' => now(),
        ]);
    }
});
```

### 10.3 Optimasi Query dengan Eager Loading

Selalu gunakan eager loading untuk menghindari N+1 query problem:

```php
// Tidak optimal
$mahasiswas = Mahasiswa::all();
foreach ($mahasiswas as $m) {
    echo $m->jurusan->nama;
    foreach ($m->mataKuliahs as $mk) {
        echo $mk->nama;
    }
}

// Optimal dengan eager loading
$mahasiswas = Mahasiswa::with('jurusan', 'mataKuliahs')->get();
foreach ($mahasiswas as $m) {
    echo $m->jurusan->nama;
    foreach ($m->mataKuliahs as $mk) {
        echo $mk->nama;
    }
}
```

### 10.4 Gunakan Events untuk Melacak Perubahan

Gunakan events Laravel untuk melacak perubahan pada relasi:

```php
// Di model Mahasiswa
protected static function booted()
{
    static::updated(function ($mahasiswa) {
        // Log perubahan data mahasiswa
    });
}

// Di Observer
public function updatingPivot(Mahasiswa $mahasiswa, MataKuliah $mataKuliah, array $attributes)
{
    // Log perubahan nilai
    if (isset($attributes['nilai'])) {
        // Simpan log perubahan nilai
    }
}
```

### 10.5 Caching untuk Performa

Gunakan caching untuk meningkatkan performa, terutama untuk transkrip dan perhitungan IPK:

```php
public function hitungIPK()
{
    $mahasiswaId = $this->id;
    
    return Cache::remember('ipk_mahasiswa_' . $mahasiswaId, now()->addMinutes(60), function() use ($mahasiswaId) {
        $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
        
        // Hitung IPK seperti biasa
        $mataKuliahs = $mahasiswa->mataKuliahs()
                                ->wherePivot('status', 'selesai')
                                ->get();
        
        // Perhitungan IPK
        // ...
        
        return $ipk;
    });
}
```

## Kesimpulan

Pada praktik ini, kita telah mengimplementasikan relasi Many-to-Many antara Mahasiswa dan Mata Kuliah dengan fitur-fitur yang komprehensif. Kita telah menerapkan pembuatan migrasi, model, controller, view, dan fitur-fitur tambahan seperti perhitungan IPK, transkrip nilai, dan manajemen mata kuliah.

Dengan implementasi ini, aplikasi manajemen data Mahasiswa kita dapat menangani berbagai kebutuhan terkait akademik seperti pengambilan mata kuliah, input nilai, perhitungan IPK, dan pembuatan transkrip. Semuanya diimplementasikan dengan memanfaatkan relasi Many-to-Many yang kuat di Laravel.

Pengembangan lebih lanjut bisa dilakukan dengan menambahkan fitur-fitur seperti KRS online, persetujuan pengambilan mata kuliah oleh dosen wali, atau integrasi dengan sistem lain seperti pembayaran biaya kuliah.