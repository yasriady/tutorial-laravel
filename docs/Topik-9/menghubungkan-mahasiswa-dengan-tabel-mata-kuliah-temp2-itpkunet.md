# Menghubungkan Mahasiswa dengan Tabel Mata Kuliah (Lanjutan)

Dalam melanjutkan materi untuk menghubungkan model Mahasiswa dengan model Mata Kuliah, berikut adalah tambahan konsep dan implementasi praktis untuk relasi many-to-many yang lebih kompleks.

## Mengoptimalkan Query dengan Eager Loading

Eager loading adalah teknik penting untuk menghindari masalah N+1 query saat bekerja dengan relasi. Mari kita implementasikan dalam konteks mahasiswa dan mata kuliah:

```php
// Tanpa eager loading (akan menghasilkan banyak query)
$mahasiswas = Mahasiswa::all();
foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->nama . ' mengambil ' . $mahasiswa->mataKuliahs->count() . ' mata kuliah';
}

// Dengan eager loading (lebih efisien)
$mahasiswas = Mahasiswa::with('mataKuliahs')->get();
foreach ($mahasiswas as $mahasiswa) {
    echo $mahasiswa->nama . ' mengambil ' . $mahasiswa->mataKuliahs->count() . ' mata kuliah';
}
```

## Implementasi Perhitungan IPK

Untuk melengkapi fungsionalitas akademik, kita dapat menambahkan fitur perhitungan IPK berdasarkan nilai mata kuliah:

### Tambahkan Method di Model Mahasiswa

```php
// app/Models/Mahasiswa.php

public function hitungIPK($semester = null, $tahunAjaran = null)
{
    $query = $this->mataKuliahs();
    
    if ($semester) {
        $query->wherePivot('semester', $semester);
    }
    
    if ($tahunAjaran) {
        $query->wherePivot('tahun_ajaran', $tahunAjaran);
    }
    
    $mataKuliahs = $query->get();
    
    if ($mataKuliahs->isEmpty()) {
        return 0;
    }
    
    $totalSKS = 0;
    $totalNilai = 0;
    
    foreach ($mataKuliahs as $mk) {
        // Hanya hitung yang sudah ada nilainya
        if ($mk->pivot->nilai) {
            $bobot = $this->konversiNilai($mk->pivot->nilai);
            $totalSKS += $mk->sks;
            $totalNilai += ($bobot * $mk->sks);
        }
    }
    
    return $totalSKS > 0 ? round($totalNilai / $totalSKS, 2) : 0;
}

private function konversiNilai($nilai)
{
    switch ($nilai) {
        case 'A': return 4.0;
        case 'B': return 3.0;
        case 'C': return 2.0;
        case 'D': return 1.0;
        case 'E': return 0.0;
        default: return 0.0;
    }
}
```

### Tampilkan IPK di View

Modifikasi file `resources/views/mahasiswas/mata_kuliah.blade.php` untuk menampilkan IPK:

```html
<div class="card mb-4">
    <div class="card-header">
        <h5 class="mb-0">Indeks Prestasi</h5>
    </div>
    <div class="card-body">
        <div class="row">
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body text-center">
                        <h3>IPK</h3>
                        <h2 class="text-primary">{{ $mahasiswa->hitungIPK() }}</h2>
                    </div>
                </div>
            </div>
            
            @if(request('semester'))
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body text-center">
                        <h3>IP Semester {{ request('semester') }}</h3>
                        <h2 class="text-success">{{ $mahasiswa->hitungIPK(request('semester'), request('tahun_ajaran')) }}</h2>
                    </div>
                </div>
            </div>
            @endif
        </div>
    </div>
</div>
```

## Menerapkan Scope Query untuk Mata Kuliah

Untuk memudahkan filtering mata kuliah, kita dapat menambahkan scope query di model MataKuliah:

```php
// app/Models/MataKuliah.php

// Scope untuk mencari mata kuliah berdasarkan SKS
public function scopeSKSRange($query, $min, $max)
{
    return $query->whereBetween('sks', [$min, $max]);
}

// Scope untuk mata kuliah wajib
public function scopeWajib($query)
{
    return $query->where('status', 'wajib');
}

// Scope untuk mata kuliah pilihan
public function scopePilihan($query)
{
    return $query->where('status', 'pilihan');
}
```

### Penggunaan Scope dalam Controller

```php
// Dalam MahasiswaMataKuliahController.php - method create

public function create($mahasiswaId)
{
    $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
    
    // Filter mata kuliah berdasarkan parameter request
    $query = MataKuliah::query();
    
    if (request()->has('sks_min') && request()->has('sks_max')) {
        $query->SKSRange(request('sks_min'), request('sks_max'));
    }
    
    if (request()->has('status') && in_array(request('status'), ['wajib', 'pilihan'])) {
        $query->{request('status')}();
    }
    
    $mataKuliahs = $query->get();
    
    return view('mahasiswas.pilih_mata_kuliah', compact('mahasiswa', 'mataKuliahs'));
}
```

## Implementasi Transkrip Nilai

Tambahkan fitur untuk mencetak transkrip nilai mahasiswa:

### Buat Controller Method untuk Transkrip

```php
// MahasiswaMataKuliahController.php

public function transkrip($mahasiswaId)
{
    $mahasiswa = Mahasiswa::with(['jurusan', 'mataKuliahs' => function($query) {
        $query->orderBy('semester')->orderBy('kode_mk');
    }])->findOrFail($mahasiswaId);
    
    // Kelompokkan mata kuliah berdasarkan semester
    $mataKuliahPerSemester = $mahasiswa->mataKuliahs->groupBy('pivot.semester');
    
    // Hitung IPK
    $ipk = $mahasiswa->hitungIPK();
    
    return view('mahasiswas.transkrip', compact('mahasiswa', 'mataKuliahPerSemester', 'ipk'));
}
```

### Buat View untuk Transkrip

Buat file `resources/views/mahasiswas/transkrip.blade.php`:

```html
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="card mb-4">
        <div class="card-header">
            <div class="d-flex justify-content-between align-items-center">
                <h3>Transkrip Nilai</h3>
                <button class="btn btn-primary" onclick="window.print()">Cetak</button>
            </div>
        </div>
        <div class="card-body">
            <div class="row mb-4">
                <div class="col-md-6">
                    <table class="table table-borderless">
                        <tr>
                            <th width="30%">NIM</th>
                            <td>: {{ $mahasiswa->nim }}</td>
                        </tr>
                        <tr>
                            <th>Nama</th>
                            <td>: {{ $mahasiswa->nama }}</td>
                        </tr>
                        <tr>
                            <th>Jurusan</th>
                            <td>: {{ $mahasiswa->jurusan->nama_jurusan }}</td>
                        </tr>
                    </table>
                </div>
                <div class="col-md-6">
                    <div class="card">
                        <div class="card-body text-center">
                            <h4>Indeks Prestasi Kumulatif (IPK)</h4>
                            <h2 class="text-primary">{{ $ipk }}</h2>
                        </div>
                    </div>
                </div>
            </div>
            
            @foreach($mataKuliahPerSemester as $semester => $mataKuliahs)
                <div class="card mb-4">
                    <div class="card-header bg-light">
                        <h5>Semester {{ $semester }}</h5>
                    </div>
                    <div class="card-body">
                        <table class="table table-bordered">
                            <thead>
                                <tr>
                                    <th>Kode MK</th>
                                    <th>Mata Kuliah</th>
                                    <th>SKS</th>
                                    <th>Nilai</th>
                                    <th>Bobot</th>
                                    <th>Tahun Ajaran</th>
                                </tr>
                            </thead>
                            <tbody>
                                @php 
                                    $totalSKS = 0; 
                                    $totalBobot = 0;
                                @endphp
                                
                                @foreach($mataKuliahs as $mk)
                                    @php
                                        $bobot = $mahasiswa->konversiNilai($mk->pivot->nilai ?: 'E');
                                        $totalSKS += $mk->sks;
                                        $totalBobot += ($bobot * $mk->sks);
                                    @endphp
                                    <tr>
                                        <td>{{ $mk->kode_mk }}</td>
                                        <td>{{ $mk->nama_mk }}</td>
                                        <td>{{ $mk->sks }}</td>
                                        <td>{{ $mk->pivot->nilai ?: '-' }}</td>
                                        <td>{{ $bobot }}</td>
                                        <td>{{ $mk->pivot->tahun_ajaran }}</td>
                                    </tr>
                                @endforeach
                                
                                <tr class="table-secondary">
                                    <th colspan="2">Total</th>
                                    <th>{{ $totalSKS }}</th>
                                    <th colspan="2">IP Semester: {{ $totalSKS > 0 ? round($totalBobot / $totalSKS, 2) : 0 }}</th>
                                    <th></th>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            @endforeach
        </div>
    </div>
</div>

<style type="text/css" media="print">
    @page {
        size: portrait;
        margin: 1cm;
    }
    
    .btn {
        display: none;
    }
    
    .card {
        border: none !important;
    }
    
    .card-header {
        background-color: white !important;
        border-bottom: 2px solid #000;
    }
</style>
@endsection
```

### Tambahkan Route untuk Transkrip

```php
// routes/web.php
Route::get('mahasiswas/{mahasiswa}/transkrip', [MahasiswaMataKuliahController::class, 'transkrip'])
     ->name('mahasiswas.transkrip');
```

## Implementasi Statistik Akademik

Untuk membantu mahasiswa memvisualisasikan performa akademik mereka, tambahkan fitur statistik:

### Buat Method Controller untuk Statistik

```php
// MahasiswaMataKuliahController.php

public function statistik($mahasiswaId)
{
    $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
    
    // Data untuk statistik nilai
    $nilaiDistribusi = [
        'A' => $mahasiswa->mataKuliahs()->wherePivot('nilai', 'A')->count(),
        'B' => $mahasiswa->mataKuliahs()->wherePivot('nilai', 'B')->count(),
        'C' => $mahasiswa->mataKuliahs()->wherePivot('nilai', 'C')->count(),
        'D' => $mahasiswa->mataKuliahs()->wherePivot('nilai', 'D')->count(),
        'E' => $mahasiswa->mataKuliahs()->wherePivot('nilai', 'E')->count(),
        'Belum Nilai' => $mahasiswa->mataKuliahs()->whereNull('nilai')->count(),
    ];
    
    // Data IP Semester
    $ipSemester = [];
    for ($i = 1; $i <= 8; $i++) {
        $ipSemester[$i] = $mahasiswa->hitungIPK($i);
    }
    
    // Total SKS per semester
    $sksSemester = [];
    for ($i = 1; $i <= 8; $i++) {
        $sksSemester[$i] = $mahasiswa->mataKuliahs()
            ->wherePivot('semester', $i)
            ->sum('sks');
    }
    
    return view('mahasiswas.statistik', compact(
        'mahasiswa', 
        'nilaiDistribusi', 
        'ipSemester',
        'sksSemester'
    ));
}
```

### Buat View untuk Statistik

Buat file `resources/views/mahasiswas/statistik.blade.php`:

```html
@extends('layouts.app')

@section('content')
<div class="container">
    <h2>Statistik Akademik - {{ $mahasiswa->nama }}</h2>
    
    <div class="row">
        <div class="col-md-6 mb-4">
            <div class="card h-100">
                <div class="card-header">
                    <h5 class="mb-0">Distribusi Nilai</h5>
                </div>
                <div class="card-body">
                    <canvas id="chartNilai" width="400" height="300"></canvas>
                </div>
            </div>
        </div>
        
        <div class="col-md-6 mb-4">
            <div class="card h-100">
                <div class="card-header">
                    <h5 class="mb-0">IP Per Semester</h5>
                </div>
                <div class="card-body">
                    <canvas id="chartIpSemester" width="400" height="300"></canvas>
                </div>
            </div>
        </div>
    </div>
    
    <div class="row">
        <div class="col-md-12 mb-4">
            <div class="card">
                <div class="card-header">
                    <h5 class="mb-0">Total SKS Per Semester</h5>
                </div>
                <div class="card-body">
                    <canvas id="chartSksSemester" width="400" height="200"></canvas>
                </div>
            </div>
        </div>
    </div>
    
    <div class="card mb-4">
        <div class="card-header">
            <h5 class="mb-0">Ringkasan Akademik</h5>
        </div>
        <div class="card-body">
            <div class="row">
                <div class="col-md-3 mb-3">
                    <div class="card bg-primary text-white">
                        <div class="card-body text-center">
                            <h5>IPK</h5>
                            <h3>{{ $mahasiswa->hitungIPK() }}</h3>
                        </div>
                    </div>
                </div>
                <div class="col-md-3 mb-3">
                    <div class="card bg-success text-white">
                        <div class="card-body text-center">
                            <h5>Total SKS</h5>
                            <h3>{{ array_sum($sksSemester) }}</h3>
                        </div>
                    </div>
                </div>
                <div class="col-md-3 mb-3">
                    <div class="card bg-info text-white">
                        <div class="card-body text-center">
                            <h5>Mata Kuliah</h5>
                            <h3>{{ $mahasiswa->mataKuliahs->count() }}</h3>
                        </div>
                    </div>
                </div>
                <div class="col-md-3 mb-3">
                    <div class="card bg-warning text-white">
                        <div class="card-body text-center">
                            <h5>Semester Aktif</h5>
                            <h3>{{ count(array_filter($sksSemester)) }}</h3>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="card-footer">
            <a href="{{ route('mahasiswas.mata_kuliah.index', $mahasiswa->id) }}" class="btn btn-secondary">Kembali ke Daftar Mata Kuliah</a>
            <a href="{{ route('mahasiswas.transkrip', $mahasiswa->id) }}" class="btn btn-primary">Lihat Transkrip</a>
        </div>
    </div>
</div>

@endsection

@section('scripts')
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.7.0/chart.min.js"></script>
<script>
document.addEventListener('DOMContentLoaded', function() {
    // Data untuk grafik
    const nilaiLabels = ['A', 'B', 'C', 'D', 'E', 'Belum Nilai'];
    const nilaiData = [
        {{ $nilaiDistribusi['A'] }}, 
        {{ $nilaiDistribusi['B'] }}, 
        {{ $nilaiDistribusi['C'] }}, 
        {{ $nilaiDistribusi['D'] }}, 
        {{ $nilaiDistribusi['E'] }}, 
        {{ $nilaiDistribusi['Belum Nilai'] }}
    ];
    
    const semesterLabels = Array.from({length: 8}, (_, i) => `Semester ${i+1}`);
    const ipData = [
        {{ $ipSemester[1] }}, 
        {{ $ipSemester[2] }}, 
        {{ $ipSemester[3] }}, 
        {{ $ipSemester[4] }}, 
        {{ $ipSemester[5] }}, 
        {{ $ipSemester[6] }}, 
        {{ $ipSemester[7] }}, 
        {{ $ipSemester[8] }}
    ];
    
    const sksData = [
        {{ $sksSemester[1] }}, 
        {{ $sksSemester[2] }}, 
        {{ $sksSemester[3] }}, 
        {{ $sksSemester[4] }}, 
        {{ $sksSemester[5] }}, 
        {{ $sksSemester[6] }}, 
        {{ $sksSemester[7] }}, 
        {{ $sksSemester[8] }}
    ];
    
    // Chart Distribusi Nilai
    const ctxNilai = document.getElementById('chartNilai').getContext('2d');
    new Chart(ctxNilai, {
        type: 'pie',
        data: {
            labels: nilaiLabels,
            datasets: [{
                data: nilaiData,
                backgroundColor: [
                    'rgba(52, 152, 219, 0.7)',
                    'rgba(46, 204, 113, 0.7)',
                    'rgba(241, 196, 15, 0.7)',
                    'rgba(230, 126, 34, 0.7)',
                    'rgba(231, 76, 60, 0.7)',
                    'rgba(189, 195, 199, 0.7)'
                ],
                borderColor: [
                    'rgba(52, 152, 219, 1)',
                    'rgba(46, 204, 113, 1)',
                    'rgba(241, 196, 15, 1)',
                    'rgba(230, 126, 34, 1)',
                    'rgba(231, 76, 60, 1)',
                    'rgba(189, 195, 199, 1)'
                ],
                borderWidth: 1
            }]
        },
        options: {
            responsive: true,
            plugins: {
                legend: {
                    position: 'right',
                },
                title: {
                    display: true,
                    text: 'Distribusi Nilai'
                }
            }
        }
    });
    
    // Chart IP Semester
    const ctxIp = document.getElementById('chartIpSemester').getContext('2d');
    new Chart(ctxIp, {
        type: 'line',
        data: {
            labels: semesterLabels,
            datasets: [{
                label: 'IP Semester',
                data: ipData,
                fill: false,
                borderColor: 'rgb(75, 192, 192)',
                tension: 0.1
            }]
        },
        options: {
            responsive: true,
            scales: {
                y: {
                    beginAtZero: true,
                    max: 4
                }
            }
        }
    });
    
    // Chart SKS per Semester
    const ctxSks = document.getElementById('chartSksSemester').getContext('2d');
    new Chart(ctxSks, {
        type: 'bar',
        data: {
            labels: semesterLabels,
            datasets: [{
                label: 'Total SKS',
                data: sksData,
                backgroundColor: 'rgba(54, 162, 235, 0.2)',
                borderColor: 'rgba(54, 162, 235, 1)',
                borderWidth: 1
            }]
        },
        options: {
            responsive: true,
            scales: {
                y: {
                    beginAtZero: true
                }
            }
        }
    });
});
</script>
@endsection

```

### Tambahkan Route untuk Statistik

```php
// routes/web.php
Route::get('mahasiswas/{mahasiswa}/statistik', [MahasiswaMataKuliahController::class, 'statistik'])
     ->name('mahasiswas.statistik');
```

### Tambahkan Link ke Statistik di View Mata Kuliah

Modifikasi file `resources/views/mahasiswas/mata_kuliah.blade.php`:

```html
<div class="card-footer">
    <a href="{{ route('mahasiswas.mata_kuliah.create', $mahasiswa->id) }}" class="btn btn-primary">Tambah Mata Kuliah</a>
    <a href="{{ route('mahasiswas.statistik', $mahasiswa->id) }}" class="btn btn-info">Lihat Statistik</a>
    <a href="{{ route('mahasiswas.transkrip', $mahasiswa->id) }}" class="btn btn-success">Cetak Transkrip</a>
    <a href="{{ route('mahasiswas.show', $mahasiswa->id) }}" class="btn btn-secondary">Kembali ke Detail Mahasiswa</a>
</div>
```

## Implementasi Global Scope untuk Filtering Data

Untuk memudahkan filtering data secara global, buat sebuah Scope khusus untuk model MataKuliah:

```php
// app/Scopes/AktifScope.php
<?php

namespace App\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class AktifScope implements Scope
{
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('status', 'aktif');
    }
}
```

Terapkan scope tersebut ke model MataKuliah:

```php
// app/Models/MataKuliah.php

use App\Scopes\AktifScope;

class MataKuliah extends Model
{
    // ...
    
    protected static function boot()
    {
        parent::boot();
        
        static::addGlobalScope(new AktifScope);
    }
    
    // Method untuk mengakses semua data termasuk yang tidak aktif
    public static function withNonAktif()
    {
        return static::withoutGlobalScope(AktifScope::class);
    }
}
```

## Query Builder Advanced untuk Analisis Data

Tambahkan method di repository atau controller untuk analisis data lanjutan:

```php
// app/Repositories/MahasiswaRepository.php (buat file ini)
<?php

namespace App\Repositories;

use App\Models\Mahasiswa;
use App\Models\MataKuliah;
use Illuminate\Support\Facades\DB;

class MahasiswaRepository
{
    // Mencari mahasiswa dengan IPK tertinggi
    public function getMahasiswaIPKTertinggi($limit = 10)
    {
        $mahasiswas = Mahasiswa::withCount('mataKuliahs')->get();
        
        // Filter hanya yang memiliki minimal 10 mata kuliah dengan nilai
        $filtered = $mahasiswas->filter(function($mahasiswa) {
            return $mahasiswa->mataKuliahs()
                ->whereNotNull('nilai')
                ->count() >= 10;
        });
        
        // Hitung IPK untuk setiap mahasiswa
        $withIPK = $filtered->map(function($mahasiswa) {
            $mahasiswa->ipk = $mahasiswa->hitungIPK();
            return $mahasiswa;
        });
        
        // Urutkan berdasarkan IPK tertinggi
        return $withIPK->sortByDesc('ipk')->take($limit);
    }
    
    // Analisis mata kuliah dengan persentase kelulusan tertinggi
    public function getTopPassingRateMataKuliah($limit = 10)
    {
        return DB::table('mata_kuliahs')
            ->select(
                'mata_kuliahs.id',
                'mata_kuliahs.kode_mk',
                'mata_kuliahs.nama_mk',
                DB::raw('COUNT(mahasiswa_mata_kuliah.mahasiswa_id) as total_mahasiswa'),
                DB::raw('SUM(CASE WHEN mahasiswa_mata_kuliah.nilai IN ("A", "B", "C") THEN 1 ELSE 0 END) as passed'),
                DB::raw('(SUM(CASE WHEN mahasiswa_mata_kuliah.nilai IN ("A", "B", "C") THEN 1 ELSE 0 END) / COUNT(mahasiswa_mata_kuliah.mahasiswa_id)) * 100 as passing_rate')
            )
            ->join('mahasiswa_mata_kuliah', 'mata_kuliahs.id', '=', 'mahasiswa_mata_kuliah.mata_kuliah_id')
            ->whereNotNull('mahasiswa_mata_kuliah.nilai')
            ->groupBy('mata_kuliahs.id', 'mata_kuliahs.kode_mk', 'mata_kuliahs.nama_mk')
            ->having('total_mahasiswa', '>=', 5)
            ->orderByDesc('passing_rate')
            ->limit($limit)
            ->get();
    }
    
    // Mencari mahasiswa dengan jumlah SKS terbanyak
    public function getMahasiswaSKSTerbanyak($limit = 10)
    {
        return DB::table('mahasiswas')
            ->select(
                'mahasiswas.id',
                'mahasiswas.nim',
                'mahasiswas.nama',
                'jurusans.nama_jurusan',
                DB::raw('SUM(mata_kuliahs.sks) as total_sks')
            )
            ->join('mahasiswa_mata_kuliah', 'mahasiswas.id', '=', 'mahasiswa_mata_kuliah.mahasiswa_id')
            ->join('mata_kuliahs', 'mahasiswa_mata_kuliah.mata_kuliah_id', '=', 'mata_kuliahs.id')
            ->join('jurusans', 'mahasiswas.jurusan_id', '=', 'jurusans.id')
            ->groupBy('mahasiswas.id', 'mahasiswas.nim', 'mahasiswas.nama', 'jurusans.nama_jurusan')
            ->orderByDesc('total_sks')
            ->limit($limit)
            ->get();
    }
}
```

## Dashboard Akademik dengan Data Agregat

Buat dashboard untuk melihat statistik agregat mata kuliah:

### Buat MataKuliahController Method

```php
// app/Http/Controllers/MataKuliahController.php

public function dashboard()
{
    $repository = new MahasiswaRepository();
    
    // Total data
    $totalMahasiswa = Mahasiswa::count();
    $totalMataKuliah = MataKuliah::count();
    $totalJurusan = \App\Models\Jurusan::count();
    
    // Mahasiswa dengan IPK tertinggi
    $mahasiswaTopIPK = $repository->getMahasiswaIPKTertinggi(5);
    
    // Mata kuliah dengan passing rate tertinggi
    $topPassingRate = $repository->getTopPassingRateMataKuliah(5);
    
    // Mahasiswa dengan total SKS terbanyak
    $topSKS = $repository->getMahasiswaSKSTerbanyak(5);
    
    return view('mata_kuliahs.dashboard', compact(
        'totalMahasiswa', 
        'totalMataKuliah', 
        'totalJurusan',
        'mahasiswaTopIPK',
        'topPassingRate',
        'topSKS'
    ));
}
```

### Buat View untuk Dashboard

Buat file `resources/views/mata_kuliahs/dashboard.blade.php`:

```html
@extends('layouts.app')

@section('content')
<div class="container">
    <h2 class="mb-4">Dashboard Akademik</h2>
    
    <div class="row mb-4">
        <div class="col-md-4">
            <div class="card bg-primary text-white">
                <div class="card-body text-center">
                    <h3>Total Mahasiswa</h3>
                    <h2>{{ $totalMahasiswa }}</h2>
                </div>
            </div>
        </div>
        <div class="col-md-4">
            <div class="card bg-success text-white">
                <div class="card-body text-center">
                    <h3>Total