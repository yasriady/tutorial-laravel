# Form dengan Validasi JavaScript

Validasi form tidak hanya dilakukan di sisi server (back-end) tetapi juga perlu diterapkan di sisi klien (front-end) untuk memberikan respons cepat kepada pengguna dan menghemat request ke server. Mari kita pelajari cara menambahkan validasi JavaScript pada form aplikasi mahasiswa.

## Keuntungan Validasi JavaScript

1. **Respons cepat** - Pengguna langsung mendapat feedback tanpa perlu menunggu respon server
2. **Mengurangi beban server** - Menolak data invalid sebelum dikirim ke server
3. **Pengalaman pengguna lebih baik** - Validasi real-time saat pengguna mengetik
4. **Melengkapi validasi server** - Lapisan keamanan tambahan (tetap harus validasi di server)

## Implementasi Validasi JavaScript

Kita akan menerapkan validasi JavaScript dengan tiga metode:

1. Validasi HTML5 dasar
2. Validasi dengan Bootstrap Validation
3. Validasi dengan library jQuery Validation

### 1. Validasi HTML5 Dasar

HTML5 menyediakan atribut validasi sederhana seperti `required`, `min`, `max`, `pattern`, dll.

```php
{{-- resources/views/mahasiswa/create.blade.php --}}
<form action="{{ route('mahasiswa.store') }}" method="POST" novalidate>
    @csrf
    
    <div class="mb-3">
        <label for="nim" class="form-label">NIM</label>
        <input type="text" class="form-control @error('nim') is-invalid @enderror" 
               id="nim" name="nim" value="{{ old('nim') }}" 
               required pattern="[0-9]{8}" title="NIM harus 8 digit angka">
        <div class="invalid-feedback">
            @error('nim') {{ $message }} @else NIM harus 8 digit angka @enderror
        </div>
    </div>
    
    <div class="mb-3">
        <label for="nama" class="form-label">Nama Lengkap</label>
        <input type="text" class="form-control @error('nama') is-invalid @enderror" 
               id="nama" name="nama" value="{{ old('nama') }}" 
               required minlength="3" maxlength="100">
        <div class="invalid-feedback">
            @error('nama') {{ $message }} @else Nama harus diisi (3-100 karakter) @enderror
        </div>
    </div>
    
    <div class="mb-3">
        <label for="email" class="form-label">Email</label>
        <input type="email" class="form-control @error('email') is-invalid @enderror" 
               id="email" name="email" value="{{ old('email') }}" required>
        <div class="invalid-feedback">
            @error('email') {{ $message }} @else Email harus valid @enderror
        </div>
    </div>
    
    <!-- Form lainnya -->
    
    <button type="submit" class="btn btn-primary">Simpan</button>
</form>
```

Tambahkan JavaScript untuk mengaktifkan validasi HTML5:

```javascript
// public/js/form-validation.js
(function() {
  'use strict';
  
  // Ambil semua form yang perlu validasi
  var forms = document.querySelectorAll('.needs-validation');
  
  // Loop untuk mencegah submission
  Array.prototype.slice.call(forms).forEach(function(form) {
    form.addEventListener('submit', function(event) {
      if (!form.checkValidity()) {
        event.preventDefault();
        event.stopPropagation();
      }
      
      form.classList.add('was-validated');
    }, false);
  });
})();
```

### 2. Validasi dengan Bootstrap 5

Bootstrap 5 memiliki fitur validasi form yang terintegrasi:

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
                    <form action="{{ route('mahasiswa.store') }}" method="POST" class="needs-validation" novalidate>
                        @csrf
                        
                        <div class="mb-3">
                            <label for="nim" class="form-label">NIM</label>
                            <input type="text" class="form-control @error('nim') is-invalid @enderror" 
                                   id="nim" name="nim" value="{{ old('nim') }}" 
                                   required pattern="[0-9]{8}">
                            <div class="invalid-feedback">
                                @error('nim') {{ $message }} @else NIM harus 8 digit angka @enderror
                            </div>
                        </div>
                        
                        <div class="mb-3">
                            <label for="nama" class="form-label">Nama Lengkap</label>
                            <input type="text" class="form-control @error('nama') is-invalid @enderror" 
                                   id="nama" name="nama" value="{{ old('nama') }}" 
                                   required minlength="3" maxlength="100">
                            <div class="invalid-feedback">
                                @error('nama') {{ $message }} @else Nama harus diisi (3-100 karakter) @enderror
                            </div>
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
                            <div class="invalid-feedback">
                                @error('jurusan_id') {{ $message }} @else Pilih jurusan terlebih dahulu @enderror
                            </div>
                        </div>
                        
                        <div class="mb-3">
                            <label for="email" class="form-label">Email</label>
                            <input type="email" class="form-control @error('email') is-invalid @enderror" 
                                   id="email" name="email" value="{{ old('email') }}" required>
                            <div class="invalid-feedback">
                                @error('email') {{ $message }} @else Format email tidak valid @enderror
                            </div>
                        </div>
                        
                        <div class="mb-3">
                            <label for="tanggal_lahir" class="form-label">Tanggal Lahir</label>
                            <input type="date" class="form-control @error('tanggal_lahir') is-invalid @enderror" 
                                   id="tanggal_lahir" name="tanggal_lahir" value="{{ old('tanggal_lahir') }}" 
                                   required max="{{ date('Y-m-d', strtotime('-17 years')) }}">
                            <div class="invalid-feedback">
                                @error('tanggal_lahir') {{ $message }} @else Mahasiswa minimal berusia 17 tahun @enderror
                            </div>
                        </div>
                        
                        <div class="mb-3">
                            <label for="alamat" class="form-label">Alamat</label>
                            <textarea class="form-control @error('alamat') is-invalid @enderror" 
                                      id="alamat" name="alamat" rows="3" required minlength="10">{{ old('alamat') }}</textarea>
                            <div class="invalid-feedback">
                                @error('alamat') {{ $message }} @else Alamat harus diisi (min. 10 karakter) @enderror
                            </div>
                        </div>
                        
                        <div class="mb-3">
                            <label class="form-label d-block">Jenis Kelamin</label>
                            <div class="form-check form-check-inline">
                                <input class="form-check-input @error('jenis_kelamin') is-invalid @enderror" 
                                       type="radio" name="jenis_kelamin" id="jk_l" value="L" 
                                       {{ old('jenis_kelamin') == 'L' ? 'checked' : '' }} required>
                                <label class="form-check-label" for="jk_l">Laki-laki</label>
                            </div>
                            <div class="form-check form-check-inline">
                                <input class="form-check-input @error('jenis_kelamin') is-invalid @enderror" 
                                       type="radio" name="jenis_kelamin" id="jk_p" value="P" 
                                       {{ old('jenis_kelamin') == 'P' ? 'checked' : '' }}>
                                <label class="form-check-label" for="jk_p">Perempuan</label>
                            </div>
                            @error('jenis_kelamin')
                                <div class="text-danger mt-1">{{ $message }}</div>
                            @else
                                <div class="invalid-feedback d-block" id="jk_feedback" style="display: none;">
                                    Pilih jenis kelamin
                                </div>
                            @enderror
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

@section('scripts')
<script>
// Aktivasi validasi Bootstrap
(function () {
  'use strict'

  // Ambil semua forms yang membutuhkan validasi
  var forms = document.querySelectorAll('.needs-validation')

  // Loop dan cegah submission
  Array.prototype.slice.call(forms)
    .forEach(function (form) {
      form.addEventListener('submit', function (event) {
        // Validasi radio button jenis kelamin
        var radioButtons = form.querySelectorAll('input[name="jenis_kelamin"]');
        var radioSelected = false;
        
        for (var i = 0; i < radioButtons.length; i++) {
            if (radioButtons[i].checked) {
                radioSelected = true;
                break;
            }
        }
        
        if (!radioSelected) {
            document.getElementById('jk_feedback').style.display = 'block';
        } else {
            document.getElementById('jk_feedback').style.display = 'none';
        }
        
        if (!form.checkValidity() || !radioSelected) {
          event.preventDefault()
          event.stopPropagation()
        }

        form.classList.add('was-validated')
      }, false)
    })
})()

// Validasi NIM hanya angka
document.getElementById('nim').addEventListener('input', function(e) {
    this.value = this.value.replace(/[^0-9]/g, '');
});

// Validasi tanggal lahir
document.getElementById('tanggal_lahir').addEventListener('change', function() {
    const today = new Date();
    const birthDate = new Date(this.value);
    const age = today.getFullYear() - birthDate.getFullYear();
    
    if (age < 17) {
        this.setCustomValidity('Mahasiswa harus berusia minimal 17 tahun');
    } else {
        this.setCustomValidity('');
    }
});

// Validasi email format
document.getElementById('email').addEventListener('input', function() {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    
    if (!emailRegex.test(this.value)) {
        this.setCustomValidity('Format email tidak valid');
    } else {
        this.setCustomValidity('');
    }
});
</script>
@endsection
```

### 3. Validasi dengan jQuery Validation Plugin

Untuk validasi yang lebih kuat, kita bisa menggunakan jQuery Validation Plugin:

1. **Tambahkan jQuery dan plugin-nya**:

```html
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jquery-validation@1.19.5/dist/jquery.validate.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jquery-validation@1.19.5/dist/additional-methods.min.js"></script>
```

2. **Implementasi validasi di view**:

```php
@section('scripts')
<script>
$(document).ready(function() {
    // Terjemahkan pesan error ke Bahasa Indonesia
    $.extend($.validator.messages, {
        required: "Kolom ini wajib diisi",
        email: "Masukkan alamat email yang valid",
        minlength: $.validator.format("Minimal {0} karakter"),
        maxlength: $.validator.format("Maksimal {0} karakter"),
        digits: "Hanya angka yang diperbolehkan",
        date: "Masukkan format tanggal yang valid"
    });
    
    // Tambah metode validasi kustom untuk NIM
    $.validator.addMethod("nimFormat", function(value, element) {
        return this.optional(element) || /^[0-9]{8}$/.test(value);
    }, "NIM harus 8 digit angka");
    
    // Tambah metode validasi untuk usia minimum
    $.validator.addMethod("minAge", function(value, element, minAge) {
        var today = new Date();
        var birthDate = new Date(value);
        var age = today.getFullYear() - birthDate.getFullYear();
        
        if (today.getMonth() < birthDate.getMonth() || 
            (today.getMonth() == birthDate.getMonth() && today.getDate() < birthDate.getDate())) {
            age--;
        }
        
        return age >= minAge;
    }, "Mahasiswa harus berusia minimal 17 tahun");
    
    // Inisialisasi validasi
    $("#formMahasiswa").validate({
        rules: {
            nim: {
                required: true,
                nimFormat: true
            },
            nama: {
                required: true,
                minlength: 3,
                maxlength: 100
            },
            jurusan_id: {
                required: true
            },
            email: {
                required: true,
                email: true
            },
            tanggal_lahir: {
                required: true,
                date: true,
                minAge: 17
            },
            alamat: {
                required: true,
                minlength: 10
            },
            jenis_kelamin: {
                required: true
            }
        },
        messages: {
            nim: {
                required: "NIM wajib diisi",
                nimFormat: "NIM harus 8 digit angka"
            },
            nama: {
                required: "Nama wajib diisi",
                minlength: "Nama minimal 3 karakter",
                maxlength: "Nama maksimal 100 karakter"
            },
            jurusan_id: {
                required: "Jurusan wajib dipilih"
            },
            email: {
                required: "Email wajib diisi",
                email: "Format email tidak valid"
            },
            tanggal_lahir: {
                required: "Tanggal lahir wajib diisi",
                date: "Format tanggal tidak valid",
                minAge: "Mahasiswa harus berusia minimal 17 tahun"
            },
            alamat: {
                required: "Alamat wajib diisi",
                minlength: "Alamat minimal 10 karakter"
            },
            jenis_kelamin: {
                required: "Jenis kelamin wajib dipilih"
            }
        },
        errorElement: 'div',
        errorPlacement: function(error, element) {
            error.addClass('invalid-feedback');
            
            if (element.prop('type') === 'radio') {
                error.insertAfter(element.parent().parent());
            } else {
                error.insertAfter(element);
            }
        },
        highlight: function(element, errorClass, validClass) {
            $(element).addClass('is-invalid').removeClass('is-valid');
        },
        unhighlight: function(element, errorClass, validClass) {
            $(element).addClass('is-valid').removeClass('is-invalid');
        },
        submitHandler: function(form) {
            // Kirim form jika validasi sukses
            form.submit();
        }
    });
    
    // Real-time validation untuk NIM
    $("#nim").on('keyup', function() {
        this.value = this.value.replace(/[^0-9]/g, '');
    });
});
</script>
@endsection
```

3. **Update form dengan ID**:

```php
<form action="{{ route('mahasiswa.store') }}" method="POST" id="formMahasiswa">
```

## Validasi Real-time dengan AJAX

Untuk validasi tertentu seperti memeriksa NIM yang sudah terdaftar, kita bisa menggunakan AJAX:

```javascript
// Validasi NIM dengan AJAX
$("#nim").on('blur', function() {
    var nim = $(this).val();
    if (nim.length === 8) {
        $.ajax({
            url: "{{ route('check.nim') }}",
            type: "POST",
            data: {
                "_token": "{{ csrf_token() }}",
                "nim": nim,
                "mahasiswa_id": "{{ isset($mahasiswa) ? $mahasiswa->id : '' }}"
            },
            success: function(response) {
                if (response.exists) {
                    $("#nim").addClass('is-invalid').removeClass('is-valid');
                    $("#nim-error").text("NIM sudah terdaftar").show();
                } else {
                    $("#nim").addClass('is-valid').removeClass('is-invalid');
                    $("#nim-error").hide();
                }
            }
        });
    }
});
```

Buat route dan controller untuk melakukan pengecekan:

```php
// routes/web.php
Route::post('/check-nim', [MahasiswaController::class, 'checkNim'])->name('check.nim');

// MahasiswaController.php
public function checkNim(Request $request)
{
    $nim = $request->nim;
    $mahasiswaId = $request->mahasiswa_id;
    
    $query = Mahasiswa::where('nim', $nim);
    
    // Jika sedang edit, exclude mahasiswa yang sedang diedit
    if (!empty($mahasiswaId)) {
        $query->where('id', '!=', $mahasiswaId);
    }
    
    $exists = $query->exists();
    
    return response()->json(['exists' => $exists]);
}
```

## Tips Tambahan untuk Validasi Form

1. **Balancing UX**: Jangan terlalu agresif dengan validasi—biarkan pengguna menyelesaikan input sebelum memvalidasi

2. **Visual Cues**: Gunakan warna dan ikon untuk menunjukkan status validasi (hijau untuk valid, merah untuk error)

3. **Pesan Error yang Jelas**: Berikan petunjuk spesifik tentang apa yang salah dan bagaimana memperbaikinya

4. **Performa**: Validasi yang kompleks bisa memperlambat browser, jadi pastikan kode JavaScript efisien

5. **Aksesibilitas**: Pastikan pesan error dapat dibaca oleh screen reader untuk pengguna disabilitas

6. **Fallback**: Selalu lakukan validasi server-side sebagai backup jika JavaScript dinonaktifkan

Dengan menerapkan validasi JavaScript, form aplikasi mahasiswa Anda akan lebih user-friendly dan interaktif, sambil tetap mempertahankan validasi server-side untuk keamanan.

Selanjutnya, kita akan membahas: **Datatable untuk Manajemen Data Mahasiswa**