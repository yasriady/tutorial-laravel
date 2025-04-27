# Penggunaan Blade Template

Blade adalah sistem templating yang kuat di Laravel, yang memungkinkan Anda menulis kode PHP dalam tampilan Anda dengan sintaks yang lebih mudah dan elegan. Mari kita pelajari dasar-dasar Blade dan bagaimana menggunakannya secara efektif.

## Apa itu Blade Template?

Blade adalah engine templating yang disediakan oleh Laravel. File template Blade disimpan dengan ekstensi `.blade.php` dan biasanya disimpan di direktori `resources/views`. Beberapa keunggulan Blade:

- Sintaks yang sederhana dan mudah dipahami
- Inheritance layout yang kuat
- Kompilasi ke PHP native (performa lebih baik)
- Tidak membatasi penggunaan PHP reguler dalam template

## Sintaks Dasar Blade

### 1. Menampilkan Data

```php
{{-- Menampilkan variabel --}}
{{ $nama }}

{{-- Menampilkan variabel dengan escape HTML (default) --}}
{{ $html_content }}

{{-- Menampilkan variabel tanpa escape HTML --}}
{!! $html_content !!}
```

> **Catatan Keamanan**: Selalu gunakan `{{ }}` (dengan escape) untuk mencegah XSS attack, kecuali Anda benar-benar yakin dengan konten yang ditampilkan.

### 2. Direktif Kondisional

```php
{{-- If statement --}}
@if($user->isAdmin)
    <p>Selamat datang, Admin!</p>
@elseif($user->isTeacher)
    <p>Selamat datang, Pengajar!</p>
@else
    <p>Selamat datang, Mahasiswa!</p>
@endif

{{-- Unless (kebalikan if) --}}
@unless($mahasiswa->isActive)
    <p>Mahasiswa tidak aktif</p>
@endunless

{{-- Isset dan empty check --}}
@isset($mahasiswa)
    <p>NIM: {{ $mahasiswa->nim }}</p>
@endisset

@empty($daftarMahasiswa)
    <p>Tidak ada mahasiswa terdaftar</p>
@endempty
```

### 3. Perulangan

```php
{{-- For loop --}}
@for($i = 0; $i < 10; $i++)
    <p>Iterasi ke-{{ $i }}</p>
@endfor

{{-- Foreach loop --}}
@foreach($mahasiswas as $mahasiswa)
    <p>{{ $mahasiswa->nama }} ({{ $mahasiswa->nim }})</p>
@endforeach

{{-- Menggunakan $loop variable --}}
@foreach($mahasiswas as $mahasiswa)
    @if($loop->first)
        <p>Ini adalah mahasiswa pertama</p>
    @endif
    
    <p>{{ $loop->iteration }}: {{ $mahasiswa->nama }}</p>
    
    @if($loop->last)
        <p>Ini adalah mahasiswa terakhir</p>
    @endif
@endforeach

{{-- While loop --}}
@while($condition)
    <p>Loop while berjalan</p>
@endwhile
```

### 4. Komentar

```php
{{-- Ini adalah komentar Blade yang tidak akan muncul di HTML yang dihasilkan --}}

<!-- Ini adalah komentar HTML yang akan terlihat di source code -->
```

### 5. Direktif PHP

```php
@php
    $total = 0;
    foreach($mahasiswas as $mahasiswa) {
        $total += $mahasiswa->nilai;
    }
    $average = $total / count($mahasiswas);
@endphp

<p>Nilai rata-rata: {{ $average }}</p>
```

## Penggunaan dalam Aplikasi Mahasiswa

Mari terapkan pengetahuan ini dalam konteks aplikasi manajemen data mahasiswa:

```php
{{-- resources/views/mahasiswa/index.blade.php --}}

<h1>Daftar Mahasiswa</h1>

@if(session('success'))
    <div class="alert alert-success">
        {{ session('success') }}
    </div>
@endif

@if($mahasiswas->isEmpty())
    <p>Belum ada data mahasiswa</p>
@else
    <table class="table">
        <thead>
            <tr>
                <th>No</th>
                <th>NIM</th>
                <th>Nama</th>
                <th>Jurusan</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach($mahasiswas as $index => $mahasiswa)
                <tr>
                    <td>{{ $index + 1 }}</td>
                    <td>{{ $mahasiswa->nim }}</td>
                    <td>{{ $mahasiswa->nama }}</td>
                    <td>{{ $mahasiswa->jurusan->nama }}</td>
                    <td>
                        <a href="{{ route('mahasiswa.edit', $mahasiswa->id) }}" class="btn btn-sm btn-primary">Edit</a>
                        <a href="{{ route('mahasiswa.show', $mahasiswa->id) }}" class="btn btn-sm btn-info">Detail</a>
                        
                        <form action="{{ route('mahasiswa.destroy', $mahasiswa->id) }}" method="POST" style="display: inline;">
                            @csrf
                            @method('DELETE')
                            <button type="submit" class="btn btn-sm btn-danger" onclick="return confirm('Yakin ingin menghapus?')">Hapus</button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
    
    {{ $mahasiswas->links() }}
@endif
```

## Memanggil Blade Template dari Controller

Untuk merender view Blade dari controller:

```php
// MahasiswaController.php
public function index()
{
    $mahasiswas = Mahasiswa::with('jurusan')->paginate(10);
    return view('mahasiswa.index', compact('mahasiswas'));
}
```

Selanjutnya, kita akan membahas sub-topik berikutnya: **Layout Master dan Komponen**