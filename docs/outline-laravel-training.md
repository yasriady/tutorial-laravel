# Outline Pelatihan Pemrograman Laravel untuk Pemula

## Ringkasan Pelatihan
Pelatihan ini dirancang untuk memperkenalkan peserta ke dasar-dasar pengembangan aplikasi web menggunakan Laravel dengan fokus pada manajemen data Mahasiswa. Peserta akan belajar dari instalasi lingkungan pengembangan hingga pengembangan fitur CRUD lengkap.


## Alat yang Digunakan
- Laragon (sebagai lingkungan pengembangan lokal)
- Visual Studio Code (editor kode)
- Laravel Framework
- Git (untuk version control)
- Postman (untuk testing API)

## Project Detail
- Nama direktori project: mahasiswa-app
- Platform: Windows 10/11, Linux
- Url project: http://mahasiswa-app.test
- Jika menggunakan server lokal, Url: http://localhost/mahasiswa-app/public

## Struktur Pelatihan

### Pertemuan 1: Pengenalan dan Persiapan (2 jam)
- [Pengenalan tentang Laravel dan ekosistemnya](Topik-1/1-laravel-introduction.html)
- [Instalasi dan konfigurasi Laragon](Topik-1/2b-laragon-installation-guide.html)
- [Instalasi dan konfigurasi Visual Studio Code](Topik-1/3b-vscode-installation-tutorial.html)
- [Pengenalan struktur folder Laravel](Topik-1/4-laravel-folder-structure.html)
- [Pembuatan proyek Laravel pertama](Topik-1/5-first-laravel-project.html)
- [Pengenalan Git untuk version control](Topik-1/6-introduction-to-git.html)
- [Praktik: Inisialisasi repositori Git dan commit pertama](Topik-1/7-git-repository-initialization.html)

### Pertemuan 2: Dasar-Dasar Laravel (2 jam)
- Konfigurasi database di Laravel
- Pengenalan Artisan CLI
- Pengenalan Request Lifecycle di Laravel
- Routing di Laravel
- Membuat Controller pertama
- Membuat View pertama
- [Praktik: Membuat halaman home sederhana](Topik-2/7b-laravel-home-page-tutorial.html)

### Pertemuan 3: Model dan Database (2 jam)
- Pengenalan Model pada Laravel
- Membuat migrasi database untuk tabel Mahasiswa
- Implementasi model Mahasiswa
- Eloquent ORM dasar
- Seeder dan Factory untuk data dummy
- Praktik: Membuat model dan migrasi untuk Mahasiswa

### Pertemuan 4: Error Handling dan Debugging (2 jam)
- Pengenalan error handling di Laravel
- Menggunakan logging untuk debugging
- Laravel Debugbar untuk monitoring queries
- Error reporting dan pemecahan masalah umum
- Try-catch dan exception handling
- Praktik: Debugging aplikasi dengan skenario error

### Pertemuan 5: CRUD - Create dan Read (2 jam)
- Membuat form tambah data Mahasiswa
- Validasi input data
- Menampilkan daftar Mahasiswa
- Implementasi pagination
- Implementasi pencarian data Mahasiswa
- Praktik: Membuat halaman daftar dan form tambah Mahasiswa

### Pertemuan 6: CRUD - Update dan Delete (2 jam)
- Membuat form edit data Mahasiswa
- Implementasi update data
- Implementasi hapus data
- Konfirmasi hapus data
- Flash messages untuk notifikasi
- Praktik: Melengkapi CRUD dengan fitur edit dan hapus

### Pertemuan 7: Authentication dan Authorization (2 jam)
- Implementasi sistem login dan register
- Middleware di Laravel
- Pembatasan akses halaman
- Role dan permission sederhana
- Proteksi form dan data
- Praktik: Mengimplementasikan login dan proteksi halaman

### Pertemuan 8: Relationship dan Queries Dasar (2 jam)
- Relationship antar tabel (One-to-Many)
- Menghubungkan Mahasiswa dengan tabel Jurusan
- Query builder dasar
- Menampilkan data relasi
- Praktik: Membuat relasi Mahasiswa-Jurusan

### Pertemuan 9: Advanced Relationship dan Queries (2 jam)
- Relationship lanjutan (Many-to-Many)
- Menghubungkan Mahasiswa dengan tabel Mata Kuliah
- Query builder lanjutan
- Eager loading untuk optimasi query
- Praktik: Membuat relasi Mahasiswa-Mata Kuliah

### Pertemuan 10: Laravel UI dan UX (2 jam)
- Penggunaan Blade template
- Layout master dan komponen
- Integrasi CSS framework sederhana
- Form dengan validasi JavaScript
- Datatable untuk manajemen data Mahasiswa
- Praktik: Mempercantik tampilan aplikasi

### Pertemuan 11: API dan JSON Response (2 jam)
- Pengenalan RESTful API
- Membuat API endpoint untuk data Mahasiswa
- Resource dan Collection di Laravel
- Testing API dengan Postman
- Authentication API dengan token
- Praktik: Membuat API sederhana untuk data Mahasiswa

### Pertemuan 12: Project Akhir dan Deployment (2 jam)
- Penyempurnaan project aplikasi Mahasiswa
- Best practices pengembangan Laravel
- Persiapan deployment
- Review code dan optimasi performa
- Diskusi dan tanya jawab
- Praktik: Deployment aplikasi ke server sederhana

## Total Jam Pelatihan
- 12 pertemuan x 2 jam = 24 jam

Pelatihan ini menyediakan dasar yang kuat untuk pengembangan aplikasi web dengan Laravel, dengan fokus khusus pada pengelolaan data Mahasiswa sebagai contoh kasus nyata sepanjang kursus. Setiap pertemuan diakhiri dengan praktik untuk memastikan pemahaman peserta terhadap materi yang diajarkan.
