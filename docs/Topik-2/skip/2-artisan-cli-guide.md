# Pengenalan Artisan CLI - Tutorial Singkat

## Apa itu Artisan CLI?

Artisan adalah command-line interface (CLI) yang disertakan dengan Laravel. Artisan menyediakan banyak perintah yang berguna untuk membantu Anda membangun aplikasi Laravel dengan lebih cepat dan efisien. Perintah-perintah ini dapat mengotomatisasi banyak tugas pemrograman yang berulang dan membosankan.

## Prasyarat
- Laravel telah terinstal
- Akses terminal/command prompt
- Dasar-dasar PHP dan Laravel

## Langkah-langkah Tutorial

### 1. Mengakses Artisan CLI

- Buka terminal atau command prompt
- Navigasi ke direktori proyek Laravel Anda:
  ```bash
  cd path/ke/proyek/laravel
  # contoh: cd C:\laragon\www\mahasiswa-app
  ```

### 2. Melihat Semua Perintah Artisan yang Tersedia

- Jalankan perintah:
  ```bash
  php artisan list
  ```
  atau cukup:
  ```bash
  php artisan
  ```

- Anda akan melihat daftar semua perintah yang tersedia dikelompokkan berdasarkan kategori seperti:
  - `make` - Untuk membuat berbagai komponen Laravel
  - `migrate` - Untuk manajemen migrasi database
  - `db` - Untuk manajemen database
  - `cache` - Untuk manajemen cache
  - Dan lainnya

### 3. Mendapatkan Bantuan untuk Perintah Spesifik

- Untuk mendapatkan informasi detail tentang perintah tertentu:
  ```bash
  php artisan help nama:perintah
  ```
  
  Contoh:
  ```bash
  php artisan help make:controller
  ```

### 4. Perintah-perintah Artisan Dasar

#### Informasi Aplikasi
```bash
# Menampilkan versi Laravel
php artisan --version

# Menampilkan lingkungan aplikasi
php artisan env

# Menampilkan rute yang terdaftar
php artisan route:list
```

#### Pembersihan Cache
```bash
# Membersihkan cache konfigurasi
php artisan config:clear

# Membersihkan cache view
php artisan view:clear

# Membersihkan cache rute
php artisan route:clear

# Membersihkan cache aplikasi
php artisan cache:clear

# Membersihkan semua cache sekaligus
php artisan optimize:clear
```

### 5. Membuat Komponen Aplikasi dengan Artisan

#### Membuat Controller
```bash
# Controller dasar
php artisan make:controller MahasiswaController

# Controller dengan method resource (CRUD)
php artisan make:controller MahasiswaController --resource

# Controller dengan model
php artisan make:controller MahasiswaController --resource --model=Mahasiswa
```

#### Membuat Model
```bash
# Model dasar
php artisan make:model Mahasiswa

# Model dengan migrasi
php artisan make:model Mahasiswa -m

# Model dengan migrasi, factory, dan seeder
php artisan make:model Mahasiswa -mfs

# Model dengan controller resource
php artisan make:model Mahasiswa -mc --resource
```

#### Membuat Migrasi
```bash
# Migrasi dasar
php artisan make:migration create_mahasiswas_table

# Migrasi dengan tabel khusus
php artisan make:migration add_nim_to_mahasiswas_table --table=mahasiswas
```

#### Membuat File Lainnya
```bash
# Membuat middleware
php artisan make:middleware VerifikasiMahasiswa

# Membuat request untuk validasi
php artisan make:request StoreMahasiswaRequest

# Membuat seeder (pengisi data)
php artisan make:seeder MahasiswaSeeder

# Membuat factory (pembuat data)
php artisan make:factory MahasiswaFactory

# Membuat event
php artisan make:event MahasiswaTerdaftar

# Membuat listener
php artisan make:listener KirimEmailSelamatDatang
```

### 6. Mengelola Database dengan Artisan

#### Migrasi Database
```bash
# Menjalankan migrasi
php artisan migrate

# Menjalankan migrasi dengan seed
php artisan migrate --seed

# Rollback migrasi terakhir
php artisan migrate:rollback

# Rollback semua migrasi dan jalankan ulang
php artisan migrate:fresh

# Rollback semua migrasi, jalankan ulang, dan seed
php artisan migrate:fresh --seed
```

#### Seeding Database
```bash
# Menjalankan semua seeder
php artisan db:seed

# Menjalankan seeder tertentu
php artisan db:seed --class=MahasiswaSeeder
```

### 7. Menjalankan Artisan Tinker

Tinker adalah REPL (Read-Eval-Print-Loop) interaktif untuk Laravel yang memungkinkan Anda berinteraksi dengan aplikasi Anda dari command line:

```bash
php artisan tinker
```

Contoh penggunaan Tinker:
```php
// Membuat data Mahasiswa baru
$m = new App\Models\Mahasiswa();
$m->nama = 'Budi Santoso';
$m->nim = '12345678';
$m->save();

// Query data Mahasiswa
App\Models\Mahasiswa::all();
App\Models\Mahasiswa::where('nama', 'like', '%Budi%')->get();

// Menghitung jumlah Mahasiswa
App\Models\Mahasiswa::count();
```

### 8. Maintenance Mode

```bash
# Mengaktifkan mode maintenance
php artisan down

# Mengaktifkan dengan pesan kustom
php artisan down --message="Sedang maintenance, coba lagi nanti"

# Mengaktifkan dengan retry-after (dalam detik)
php artisan down --retry=60

# Mengaktifkan dengan halaman kustom
php artisan down --render="errors.maintenance"

# Mematikan mode maintenance
php artisan up
```

### 9. Membuat Perintah Artisan Kustom

Anda dapat membuat perintah Artisan kustom untuk mengotomatisasi tugas spesifik aplikasi:

```bash
# Membuat perintah Artisan kustom
php artisan make:command GenerateMahasiswaReport
```

Edit file yang dihasilkan di `app/Console/Commands/GenerateMahasiswaReport.php`:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\Mahasiswa;

class GenerateMahasiswaReport extends Command
{
    protected $signature = 'mahasiswa:report {--format=text : Format output (text/json)}';
    protected $description = 'Generate laporan data mahasiswa';

    public function handle()
    {
        $format = $this->option('format');
        $jumlahMahasiswa = Mahasiswa::count();
        
        $this->info("Laporan Data Mahasiswa");
        $this->info("====================");
        $this->info("Jumlah mahasiswa: $jumlahMahasiswa");
        
        if ($format === 'json') {
            $this->output->writeln(json_encode([
                'total_mahasiswa' => $jumlahMahasiswa,
                'generated_at' => now()->toDateTimeString()
            ]));
        }
        
        return Command::SUCCESS;
    }
}
```

Untuk menjalankan perintah kustom:
```bash
php artisan mahasiswa:report
php artisan mahasiswa:report --format=json
```

### 10. Scheduling dengan Artisan

Laravel memungkinkan Anda menjadwalkan perintah Artisan untuk dijalankan secara otomatis.

Edit file `app/Console/Kernel.php`:

```php
protected function schedule(Schedule $schedule)
{
    // Menjalankan perintah artisan setiap hari jam 8 pagi
    $schedule->command('mahasiswa:report')->dailyAt('8:00');
    
    // Menjalankan perintah artisan setiap jam
    $schedule->command('cache:clear')->hourly();
    
    // Menjalankan closure setiap 5 menit
    $schedule->call(function () {
        DB::table('recent_users')->delete();
    })->everyFiveMinutes();
}
```

Untuk menjalankan scheduler, tambahkan entry berikut di server cron:
```cron
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

### 11. Tips Menggunakan Artisan CLI

#### Alias untuk Artisan
Untuk menghemat waktu, tambahkan alias untuk perintah artisan di file `~/.bashrc` atau `~/.zshrc`:

```bash
alias pa="php artisan"
```

Dengan alias ini, Anda bisa menggunakan:
```bash
pa make:controller MahasiswaController
```

#### Autocompletion
Beberapa shell mendukung autocompletion untuk Artisan. Untuk membuat file autocompletion:

```bash
php artisan completion bash > artisan-completion.bash
source artisan-completion.bash
```

#### Mematikan Output
Untuk mematikan output perintah artisan (mode silent):

```bash
php artisan migrate --quiet
```

#### Menggunakan --help
Setiap perintah memiliki opsi --help yang menampilkan informasi lengkap:

```bash
php artisan make:model --help
```

## Kesimpulan

Artisan CLI adalah alat yang sangat kuat dalam ekosistem Laravel. Dengan menguasai perintah-perintah Artisan, Anda dapat meningkatkan produktivitas pengembangan secara signifikan. Dari pembuatan komponen aplikasi, manajemen database, hingga automasi tugas, Artisan menyederhanakan banyak aspek pengembangan Laravel. Sebagai langkah selanjutnya, Anda dapat mempelajari cara membuat perintah Artisan kustom untuk mengotomatisasi tugas spesifik dalam aplikasi Mahasiswa Anda.
