# Menghubungkan Mahasiswa dengan Tabel Mata Kuliah (Lanjutan)

Pada bagian sebelumnya, kita telah mempelajari dasar-dasar menghubungkan model Mahasiswa dengan model Mata Kuliah. Sekarang, mari kita lanjutkan dengan aspek-aspek yang lebih mendalam dari relasi Many-to-Many dan teknik query lanjutan yang akan mengoptimalkan aplikasi kita.

## Mengoptimalkan Query dengan Eager Loading

Saat bekerja dengan relasi Many-to-Many, salah satu masalah umum yang muncul adalah "N+1 Query Problem". Mari kita bahas bagaimana cara mengatasinya.

### Masalah N+1 Query dan Solusinya

```mermaid
flowchart TD
    A[Tanpa Eager Loading] --> B[Mengambil 10 Mahasiswa]
    B --> C[Query 1: SELECT * FROM mahasiswas LIMIT 10]
    C --> D[Query 2-11: 10 Query Terpisah<br>untuk Mengambil Mata Kuliah]
    
    E[Dengan Eager Loading] --> F[Mengambil 10 Mahasiswa dengan Mata Kuliah]
    F --> G[Query 1: SELECT * FROM mahasiswas LIMIT 10]
    G --> H[Query 2: SELECT * FROM mata_kuliahs<br>JOIN mahasiswa_mata_kuliah<br>WHERE mahasiswa_id IN (1,2,3,...)]
```

### Implementasi Eager Loading di Controller

Edit file `app/Http/Controllers/MahasiswaController.php` untuk mengoptimalkan query:

```php
public function index()
{
    // Tanpa eager loading (akan menyebabkan N+1 query)
    // $mahasiswas = Mahasiswa::all();
    
    // Dengan eager loading
    $mahasiswas = Mahasiswa::with(['jurusan', 'mataKuliahs'])->get();
    
    return view('mahasiswas.index', compact('mahasiswas'));
}

public function show($id)
{
    $mahasiswa = Mahasiswa::with(['jurusan', 'mataKuliahs'])->findOrFail($id);
    return view('mahasiswas.show', compact('mahasiswa'));
}
```

## Analisis Kinerja Mahasiswa dengan Query Builder Lanjutan

Selanjutnya, kita akan membuat fitur analisis kinerja mahasiswa berdasarkan nilai mata kuliah. Tambahkan kode berikut ke `MahasiswaMataKuliahController.php`:

```php
public function analisis($mahasiswaId)
{
    $mahasiswa = Mahasiswa::findOrFail($mahasiswaId);
    
    // Statistik nilai
    $nilaiStats = [
        'A' => 0,
        'B' => 0,
        'C' => 0,
        'D' => 0,
        'E' => 0,
        'Belum Nilai' => 0
    ];
    
    // Total SKS per semester
    $sksBySemester = [];
    
    // IP dan IPK
    $ipByTahunSemester = [];
    $totalBobot = 0;
    $totalSKS = 0;
    
    // Konversi nilai ke bobot
    $bobotNilai = [
        'A' => 4,
        'B' => 3,
        'C' => 2,
        'D' => 1,
        'E' => 0
    ];
    
    // Ambil semua mata kuliah yang diambil mahasiswa
    $mataKuliahs = $mahasiswa->mataKuliahs;
    
    foreach ($mataKuliahs as $mk) {
        // Hitung statistik nilai
        if ($mk->pivot->nilai) {
            $nilaiStats[$mk->pivot->nilai]++;
        } else {
            $nilaiStats['Belum Nilai']++;
        }
        
        // Hitung SKS per semester
        $semesterKey = $mk->pivot->semester . ' (' . $mk->pivot->tahun_ajaran . ')';
        if (!isset($sksBySemester[$semesterKey])) {
            $sksBySemester[$semesterKey] = 0;
        }
        $sksBySemester[$semesterKey] += $mk->sks;
        
        // Hitung IP per semester
        if ($mk->pivot->nilai) {
            $ipKey = $semesterKey;
            if (!isset($ipByTahunSemester[$ipKey])) {
                $ipByTahunSemester[$ipKey] = [
                    'total_bobot' => 0,
                    'total_sks' => 0
                ];
            }
            
            $bobot = $bobotNilai[$mk->pivot->nilai] * $mk->sks;
            $ipByTahunSemester[$ipKey]['total_bobot'] += $bobot;
            $ipByTahunSemester[$ipKey]['total_sks'] += $mk->sks;
            
            // Akumulasi untuk IPK
            $totalBobot += $bobot;
            $totalSKS += $mk->sks;
        }
    }
    
    // Hitung IP per semester
    foreach ($ipByTahunSemester as $key => $data) {
        if ($data['total_sks'] > 0) {
            $ipByTahunSemester[$key]['ip'] = round($data['total_bobot'] / $data['total_sks'], 2);
        } else {
            $ipByTahunSemester[$key]['ip'] = 0;
        }
    }
    
    // Hitung IPK
    $ipk = $totalSKS > 0 ? round($totalBobot / $totalSKS, 2) : 0;
    
    return view('mahasiswas.analisis', compact(
        'mahasiswa', 
        'nilaiStats', 
        'sksBySemester', 
        'ipByTahunSemester', 
        'ipk'
    ));
}
```

## Membuat View untuk Analisis Kinerja

Buat file `resources/views/mahasiswas/analisis.blade.php`:

```html
@extends('layouts.app')

@section('content')
<div class="container">
    <h2>Analisis Kinerja Akademik - {{ $mahasiswa->nama }}</h2>
    
    <div class="card mb-4">
        <div class="card-header">
            <h5 class="mb-0">Informasi Mahasiswa</h5>
        </div>
        <div class="card-body">
            <div class="row">
                <div class="col-md-6">
                    <p><strong>NIM:</strong> {{ $mahasiswa->nim }}</p>
                    <p><strong>Nama:</strong> {{ $mahasiswa->nama }}</p>
                </div>
                <div class="col-md-6">
                    <p><strong>Jurusan:</strong> {{ $mahasiswa->jurusan->nama_jurusan }}</p>
                    <p><strong>IPK:</strong> <span class="badge badge-{{ $ipk >= 3.5 ? 'success' : ($ipk >= 3.0 ? 'info' : ($ipk >= 2.0 ? 'warning' : 'danger')) }}">{{ $ipk }}</span></p>
                </div>
            </div>
        </div>
    </div>
    
    <div class="row">
        <div class="col-md-6">
            <div class="card mb-4">
                <div class="card-header">
                    <h5 class="mb-0">Statistik Nilai</h5>
                </div>
                <div class="card-body">
                    <table class="table table-bordered">
                        <thead>
                            <tr>
                                <th>Nilai</th>
                                <th>Jumlah</th>
                            </tr>
                        </thead>
                        <tbody>
                            @foreach($nilaiStats as $nilai => $jumlah)
                                <tr>
                                    <td>{{ $nilai }}</td>
                                    <td>{{ $jumlah }}</td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
        
        <div class="col-md-6">
            <div class="card mb-4">
                <div class="card-header">
                    <h5 class="mb-0">Total SKS per Semester</h5>
                </div>
                <div class="card-body">
                    <table class="table table-bordered">
                        <thead>
                            <tr>
                                <th>Semester</th>
                                <th>Total SKS</th>
                            </tr>
                        </thead>
                        <tbody>
                            @foreach($sksBySemester as $semester => $totalSKS)
                                <tr>
                                    <td>{{ $semester }}</td>
                                    <td>{{ $totalSKS }}</td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
    
    <div class="card mb-4">
        <div class="card-header">
            <h5 class="mb-0">Indeks Prestasi (IP) per Semester</h5>
        </div>
        <div class="card-body">
            @if(count($ipByTahunSemester) > 0)
                <table class="table table-bordered">
                    <thead>
                        <tr>
                            <th>Semester</th>
                            <th>Total SKS</th>
                            <th>IP</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach($ipByTahunSemester as $semester => $data)
                            <tr>
                                <td>{{ $semester }}</td>
                                <td>{{ $data['total_sks'] }}</td>
                                <td>
                                    <span class="badge badge-{{ $data['ip'] >= 3.5 ? 'success' : ($data['ip'] >= 3.0 ? 'info' : ($data['ip'] >= 2.0 ? 'warning' : 'danger')) }}">
                                        {{ $data['ip'] }}
                                    </span>
                                </td>
                            </tr>
                        @endforeach
                    </tbody>
                </table>
            @else
                <div class="alert alert-info">
                    Belum ada nilai yang dimasukkan.
                </div>
            @endif
        </div>
    </div>
    
    <div class="card mb-4">
        <div class="card-header">
            <h5 class="mb-0">Grafik Perkembangan IP</h5>
        </div>
        <div class="card-body">
            <canvas id="ipChart" width="400" height="200"></canvas>
        </div>
    </div>
    
    <div class="card-footer">
        <a href="{{ route('mahasiswas.mata_kuliah.index', $mahasiswa->id) }}" class="btn btn-primary">Kembali ke Daftar Mata Kuliah</a>
        <a href="{{ route('mahasiswas.show', $mahasiswa->id) }}" class="btn btn-secondary">Kembali ke Detail Mahasiswa</a>
    </div>
</div>

@push('scripts')
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
    document.addEventListener('DOMContentLoaded', function() {
        var ctx = document.getElementById('ipChart').getContext('2d');
        var ipData = @json(collect($ipByTahunSemester)->map(function($data) { return $data['ip'] ?? 0; }));
        var labels = @json(array_keys($ipByTahunSemester));
        
        var chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    label: 'Indeks Prestasi (IP)',
                    data: ipData,
                    backgroundColor: 'rgba(54, 162, 235, 0.2)',
                    borderColor: 'rgba(54, 162, 235, 1)',
                    borderWidth: 2,
                    pointBackgroundColor: 'rgba(54, 162, 235, 1)',
                    tension: 0.1
                }]
            },
            options: {
                scales: {
                    y: {
                        beginAtZero: true,
                        max: 4
                    }
                }
            }
        });
    });
</script>
@endpush
@endsection
```

## Tambahkan Rute untuk Analisis

Edit file `routes/web.php`:

```php
Route::get('mahasiswas/{mahasiswa}/analisis', [MahasiswaMataKuliahController::class, 'analisis'])
     ->name('mahasiswas.analisis');
```

## Tambahkan Link ke Analisis pada View Mahasiswa

Edit file `resources/views/mahasiswas/show.blade.php` untuk menambahkan link ke analisis kinerja:

```html
<div class="card">
    <div class="card-header">
        <h5 class="mb-0">Mata Kuliah</h5>
    </div>
    <div class="card-body">
        <a href="{{ route('mahasiswas.mata_kuliah.index', $mahasiswa->id) }}" class="btn btn-info">Lihat Mata Kuliah</a>
        <a href="{{ route('mahasiswas.mata_kuliah.create', $mahasiswa->id) }}" class="btn btn-primary">Tambah Mata Kuliah</a>
        <a href="{{ route('mahasiswas.analisis', $mahasiswa->id) }}" class="btn btn-success">Analisis Kinerja</a>
    </div>
</div>
```

## Implementasi Transkript Nilai

Selanjutnya, kita akan membuat fitur untuk menghasilkan transkrip nilai mahasiswa. Tambahkan kode berikut ke `MahasiswaMataKuliahController.php`:

```php
public function transkrip($mahasiswaId)
{
    $mahasiswa = Mahasiswa::with(['jurusan', 'mataKuliahs'])->findOrFail($mahasiswaId);
    
    // Konversi nilai ke bobot
    $bobotNilai = [
        'A' => 4,
        'B' => 3,
        'C' => 2,
        'D' => 1,
        'E' => 0
    ];
    
    // Group mata kuliah berdasarkan semester dan tahun ajaran
    $mataKuliahsBySemester = [];
    $totalSKS = 0;
    $totalBobot = 0;
    
    foreach ($mahasiswa->mataKuliahs as $mk) {
        $semesterKey = 'Semester ' . $mk->pivot->semester . ' (' . $mk->pivot->tahun_ajaran . ')';
        
        if (!isset($mataKuliahsBySemester[$semesterKey])) {
            $mataKuliahsBySemester[$semesterKey] = [
                'mata_kuliahs' => [],
                'total_sks' => 0,
                'total_bobot' => 0
            ];
        }
        
        $mataKuliahsBySemester[$semesterKey]['mata_kuliahs'][] = $mk;
        $mataKuliahsBySemester[$semesterKey]['total_sks'] += $mk->sks;
        
        if ($mk->pivot->nilai) {
            $bobot = isset($bobotNilai[$mk->pivot->nilai]) ? $bobotNilai[$mk->pivot->nilai] * $mk->sks : 0;
            $mataKuliahsBySemester[$semesterKey]['total_bobot'] += $bobot;
            
            $totalBobot += $bobot;
            $totalSKS += $mk->sks;
        }
    }
    
    // Hitung IP semester
    foreach ($mataKuliahsBySemester as $semester => &$data) {
        if ($data['total_sks'] > 0) {
            $data['ip'] = round($data['total_bobot'] / $data['total_sks'], 2);
        } else {
            $data['ip'] = 0;
        }
    }
    
    // Hitung IPK
    $ipk = $totalSKS > 0 ? round($totalBobot / $totalSKS, 2) : 0;
    
    return view('mahasiswas.transkrip', compact('mahasiswa', 'mataKuliahsBySemester', 'ipk', 'totalSKS'));
}
```

## Membuat View untuk Transkrip Nilai

Buat file `resources/views/mahasiswas/transkrip.blade.php`:

```html
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="text-center mb-4">
        <h2>TRANSKRIP NILAI AKADEMIK</h2>
        <h3>{{ config('app.name') }}</h3>
    </div>
    
    <div class="card mb-4">
        <div class="card-body">
            <div class="row">
                <div class="col-md-6">
                    <table class="table table-borderless">
                        <tr>
                            <th>Nama</th>
                            <td>: {{ $mahasiswa->nama }}</td>
                        </tr>
                        <tr>
                            <th>NIM</th>
                            <td>: {{ $mahasiswa->nim }}</td>
                        </tr>
                        <tr>
                            <th>Program Studi</th>
                            <td>: {{ $mahasiswa->jurusan->nama_jurusan }}</td>
                        </tr>
                    </table>
                </div>
                <div class="col-md-6">
                    <table class="table table-borderless">
                        <tr>
                            <th>Total SKS</th>
                            <td>: {{ $totalSKS }}</td>
                        </tr>
                        <tr>
                            <th>IPK</th>
                            <td>: {{ $ipk }}</td>
                        </tr>
                        <tr>
                            <th>Status</th>
                            <td>: <span class="badge badge-{{ $ipk >= 3.0 ? 'success' : 'warning' }}">{{ $ipk >= 3.0 ? 'Baik' : 'Perlu Perhatian' }}</span></td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
    </div>
    
    @foreach($mataKuliahsBySemester as $semester => $data)
    <div class="card mb-4">
        <div class="card-header d-flex justify-content-between align-items-center">
            <h5 class="mb-0">{{ $semester }}</h5>
            <div>
                IP: <span class="badge badge-{{ $data['ip'] >= 3.5 ? 'success' : ($data['ip'] >= 3.0 ? 'info' : ($data['ip'] >= 2.0 ? 'warning' : 'danger')) }}">{{ $data['ip'] }}</span>
            </div>
        </div>
        <div class="card-body">
            <table class="table table-bordered">
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
                    @php $totalSemesterSKS = 0; @endphp
                    @foreach($data['mata_kuliahs'] as $mk)
                        @php 
                            $totalSemesterSKS += $mk->sks;
                            $bobotNilai = [
                                'A' => 4,
                                'B' => 3,
                                'C' => 2,
                                'D' => 1,
                                'E' => 0
                            ];
                            $bobot = isset($bobotNilai[$mk->pivot->nilai]) ? $bobotNilai[$mk->pivot->nilai] : '-';
                        @endphp
                        <tr>
                            <td>{{ $mk->kode_mk }}</td>
                            <td>{{ $mk->nama_mk }}</td>
                            <td>{{ $mk->sks }}</td>
                            <td>
                                @if($mk->pivot->nilai)
                                    <span class="badge badge-{{ $mk->pivot->nilai == 'A' || $mk->pivot->nilai == 'B' ? 'success' : ($mk->pivot->nilai == 'C' ? 'warning' : 'danger') }}">
                                        {{ $mk->pivot->nilai }}
                                    </span>
                                @else
                                    <span class="badge badge-secondary">Belum Ada</span>
                                @endif
                            </td>
                            <td>{{ $mk->pivot->nilai ? $bobot : '-' }}</td>
                        </tr>
                    @endforeach
                </tbody>
                <tfoot>
                    <tr>
                        <th colspan="2">Total</th>
                        <th>{{ $totalSemesterSKS }}</th>
                        <th colspan="2">IP: {{ $data['ip'] }}</th>
                    </tr>
                </tfoot>
            </table>
        </div>
    </div>
    @endforeach
    
    <div class="card mb-4">
        <div class="card-header">
            <h5 class="mb-0">Keterangan Nilai</h5>
        </div>
        <div class="card-body">
            <table class="table table-bordered">
                <thead>
                    <tr>
                        <th>Nilai</th>
                        <th>Bobot</th>
                        <th>Keterangan</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>A</td>
                        <td>4.00</td>
                        <td>Sangat Baik</td>
                    </tr>
                    <tr>
                        <td>B</td>
                        <td>3.00</td>
                        <td>Baik</td>
                    </tr>
                    <tr>
                        <td>C</td>
                        <td>2.00</td>
                        <td>Cukup</td>
                    </tr>
                    <tr>
                        <td>D</td>
                        <td>1.00</td>
                        <td>Kurang</td>
                    </tr>
                    <tr>
                        <td>E</td>
                        <td>0.00</td>
                        <td>Gagal</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
    
    <div class="text-center mb-4">
        <a href="{{ route('mahasiswas.transkrip.pdf', $mahasiswa->id) }}" class="btn btn-danger"><i class="fa fa-file-pdf"></i> Export PDF</a>
        <a href="{{ route('mahasiswas.analisis', $mahasiswa->id) }}" class="btn btn-info">Lihat Analisis</a>
        <a href="{{ route('mahasiswas.mata_kuliah.index', $mahasiswa->id) }}" class="btn btn-primary">Kembali ke Daftar Mata Kuliah</a>
    </div>
</div>
@endsection
```

## Tambahkan Rute untuk Transkrip

Edit file `routes/web.php`:

```php
Route::get('mahasiswas/{mahasiswa}/transkrip', [MahasiswaMataKuliahController::class, 'transkrip'])
     ->name('mahasiswas.transkrip');
Route::get('mahasiswas/{mahasiswa}/transkrip/pdf', [MahasiswaMataKuliahController::class, 'transkripPdf'])
     ->name('mahasiswas.transkrip.pdf');
```

## Implementasi Export Transkrip ke PDF

Pertama, install package DomPDF:

```bash
composer require barryvdh/laravel-dompdf
```

Lalu tambahkan method berikut di `MahasiswaMataKuliahController.php`:

```php
public function transkripPdf($mahasiswaId)
{
    $mahasiswa = Mahasiswa::with(['jurusan', 'mataKuliahs'])->findOrFail($mahasiswaId);
    
    // Kode sama seperti method transkrip()
    // ...
    
    $pdf = PDF::loadView('mahasiswas.transkrip_pdf', compact('mahasiswa', 'mataKuliahsBySemester', 'ipk', 'totalSKS'));
    return $pdf->download('transkrip-'.$mahasiswa->nim.'.pdf');
}
```

## Melihat Perkembangan Akademik dengan Chart

Kita akan menambahkan visualisasi untuk perkembangan IP mahasiswa. Tambahkan kode berikut ke view analisis:

```javascript
<script>
    document.addEventListener('DOMContentLoaded', function() {
        // IP Chart
        var ipCtx = document.getElementById('ipChart').getContext('2d');
        var ipData = @json(collect($ipByTahunSemester)->map(function($data) { return $data['ip'] ?? 0; }));
        var labels = @json(array_keys($ipByTahunSemester));
        
        var ipChart = new Chart(ipCtx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    label: 'Indeks Prestasi (IP)',
                    data: ipData,
                    backgroundColor: 'rgba(54, 162, 235, 0.2)',
                    borderColor: 'rgba(54, 162, 235, 1)',
                    borderWidth: 2,
                    pointBackgroundColor: 'rgba(54, 162, 235, 1)',
                    tension: 0.1
                }]
            },
            options: {
                scales: {
                    y: {
                        beginAtZero: true,
                        max: 4
                    }
                }
            }
        });
        
        // Nilai Distribution Chart
        var nilaiCtx = document.getElementById('nilaiChart').getContext('2d');
        var nilaiData = @json($nilaiStats);
        
        var nilaiChart = new Chart(nilaiCtx, {
            type: 'doughnut',
            data: {
                labels: Object.keys(nilaiData),
                datasets: [{
                    data: Object.values(nilaiData),
                    backgroundColor: [
                        'rgba(75, 192, 192, 0.6)',
                        'rgba(54, 162, 235, 0.6)',
                        'rgba(255, 206, 86, 0.6)',
                        'rgba(255, 99, 132, 0.6)',
                        'rgba(153, 102, 255, 0.6)',
                        'rgba(201, 203, 207, 0.6)'
                    ],
                    borderColor: [
                        'rgba(75, 192, 192, 1)',
                        'rgba(54, 162, 235, 1)',
                        'rgba(255, 206, 86, 1)',
                        'rgba(255, 99, 132, 1)',
                        'rgba(153, 102, 255, 1)',
                        'rgba(201, 203, 207, 1)'
                    ],
                    borderWidth: 1
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: {
                        position: 'top',
                    },
                    title: {
                        display: true,
                        text: 'Distribusi Nilai'
                    }
                }
            }
        });
    });
</script>
```

## Query Lanjutan untuk Reporting

### 1. Query Rata-rata IP Seluruh Mahasiswa

```php
public function ipRataRata()
{
    // Ambil semua mahasiswa dengan mata kuliah mereka
    $mahasiswas = Mahasiswa::with('mataKuliahs')->get();
    
    $ipkData = [];
    $bobotNilai = [
        'A' => 4,
        'B' => 3,
        'C' => 2,
        'D' => 1,
        'E' => 0
    ];
    
    foreach ($mahasiswas as $mahasiswa) {
        $totalBobot = 0;
        $totalSKS = 0;
        
        foreach ($mahasiswa->mataKuliahs as $mk) {
            if ($mk->pivot->nilai && isset($bobotNilai[$mk->pivot->nilai])) {
                $bobot = $bobotNilai[$mk->pivot->nilai] * $mk->sks;
                $totalBobot += $bobot;
                $totalSKS += $mk->sks;
            }
        }
        
        if ($totalSKS > 0) {
            $ipkData[] = [
                'mahasiswa_id' => $mahasiswa->id,
                'nama' => $mahasiswa->nama,
                'jurusan' => $mahasiswa->jurusan->nama_jurusan,
                'ipk' => round($totalBobot / $totalSKS, 2)
            ];
        }
    }
    
    return $ipkData;
}
```

### 2. Query Mata Kuliah Paling Populer

```php
public function mataKuliahPopuler()
{
    return MataKuliah::withCount('mahasiswas')
        ->orderByDesc('mahasiswas_count')
        ->take(5)
        ->get();
}
```

### 3. Query Mahasiswa dengan IP Tertinggi per Jurusan

```php
public function ipTertinggiPerJurusan()
{
    $ipkData = $this->ipRataRata();
    $jurusanData = [];
    
    foreach ($ipkData as $data) {
        $jurusan = $data['jurusan'];
        
        if (!isset($jurusanData[$jurusan]) || $data['ipk'] > $jurusanData[$jurusan]['ipk']) {
            $jurusanData[$jurusan] = [
                'nama' => $data['nama'],
                'ipk' => $data['ipk']
            ];
        }
    }
    
    return $jurusanData;
}
```

## Flowchart Proses Pengembangan Relasi Mahasiswa dan Mata Kuliah

```mermaid
flowchart TD
    A[Start]