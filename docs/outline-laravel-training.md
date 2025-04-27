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

### Topik 1: Pengenalan dan Persiapan (2 jam)
- [Pengenalan tentang Laravel dan ekosistemnya](Topik-1/1-laravel-introduction.html)
- [Instalasi dan konfigurasi Laragon](Topik-1/2b-laragon-installation-guide.html)
- [Instalasi dan konfigurasi Visual Studio Code](Topik-1/3b-vscode-installation-tutorial.html)
- [Pengenalan struktur folder Laravel](Topik-1/4-laravel-folder-structure.html)
- [Pembuatan proyek Laravel pertama](Topik-1/5-first-laravel-project.html)
- [Pengenalan Git untuk version control](Topik-1/6-introduction-to-git.html)
- [Praktik: Inisialisasi repositori Git dan commit pertama](Topik-1/7-git-repository-initialization.html)

### Topik 2: Dasar-Dasar Laravel (2 jam)
- [Konfigurasi database di Laravel](Topik-2/1-konfigurasi-database-laravel.html)
- [Pengenalan Artisan CLI](Topik-2/2-artisan-cli-guide.html)
- [Pengenalan Request Lifecycle di Laravel](Topik-2/3-laravel-request-lifecycle-new.html)
- [Routing di Laravel](Topik-2/4-laravel-routing-new.html)
- [Membuat Controller pertama](Topik-2/5-membuat-controller-pertama-new.html)
- [Membuat View pertama](Topik-2/6-membuat-view-pertama-new.html)
- [Praktik: Membuat halaman home sederhana](Topik-2/7b-laravel-home-page-tutorial.html)

### Topik 3: Model dan Database (2 jam)
- [Pengenalan Model pada Laravel](Topik-3/1-pengenalan-model-laravel.html)
- [Membuat migrasi database untuk tabel Mahasiswa](Topik-3/2-membuat-migrasi-database.html)
- [Implementasi model Mahasiswa](Topik-3/3-implementasi-model-mahasiswa.html)
- [Eloquent ORM dasar](Topik-3/4-eloquent-orm-dasar.html)
- [Seeder dan Factory untuk data dummy](Topik-3/5-seeder-dan-factory-data-dummy.html)
- [Praktik: Membuat model dan migrasi untuk Mahasiswa](Topik-3/6-praktik-membuat-model-dan-migrasi-untuk-mahasiswa.html)

### Topik 4: Error Handling dan Debugging (2 jam)
- [Pengenalan error handling di Laravel](Topik-4/1-pengenalan-error-handling.html)
- [Menggunakan logging untuk debugging](Topik-4/2-menggunakan-logging-untuk-debugging.html)
- [Laravel Debugbar untuk monitoring queries](Topik-4/3-laravel-debugbar-untuk-monitoring-queries.html)
- [Error reporting dan pemecahan masalah umum](Topik-4/4-error-reporting-dan-pemecahan-umum.html)
- [Try-catch dan exception handling](Topik-4/5-try-catch-dan-exception-handling.html)
- [Praktik: Debugging aplikasi dengan skenario error](Topik-4/6-praktik-debugging-aplikasi-dengan-skenario%20error.html)

### Topik 5: CRUD - Create dan Read (2 jam)
- [Membuat form tambah data Mahasiswa](Topik-5/1-membuat-form-tambah-data-mahasiswa.html)
- [Validasi input data](Topik-5/2-validasi-input-data.html)
- [Menampilkan daftar Mahasiswa](Topik-5/3-menampilkan-daftar-mahasiswa.html)
- [Implementasi pagination](Topik-5/4-implementasi-pagination.html)
- [Implementasi pencarian data Mahasiswa](Topik-5/5-pencarian-data-mahasiswa.html)
- [Praktik: Membuat halaman daftar dan form tambah Mahasiswa](Topik-5/6-tutorial-daftar-tambah-mahasiswa.html)

### Topik 6: CRUD - Update dan Delete (2 jam)
- [Membuat form edit data Mahasiswa](Topik-6/1-tutorial-form-edit-mahasiswa.html)
- [Implementasi update data](Topik-6/2-mplementasi-update-data.html)
- [Implementasi hapus data](Topik-6/3-implementasi-hapus-data.html)
- [Konfirmasi hapus data](Topik-6/4-konfirmasi-hapus-data.html)
- [Flash messages untuk notifikasi](Topik-6/5-flash-messages-untuk-notifikasi.html)
- [Praktik: Melengkapi CRUD dengan fitur edit dan hapus](Topik-6/6-praktik-melengkapi-CRUD-dengan-fitur-edit-dan-hapus.html)

### Topik 7: Authentication dan Authorization (2 jam)
- [Implementasi sistem login dan register](Topik-7/1-implementasi-sistem-login.html)
- [Middleware di Laravel](Topik-7/2-middleware-di-laravel.html)
- [Pembatasan akses halaman](Topik-7/3-pembatasan-akses-halaman.html)
- [Role dan permission sederhana](Topik-7/4-role-dan-permission-sederhana.html)
- [Proteksi form dan data](Topik-7/5-proteksi-form-dan-data.html)
- [Praktik: Mengimplementasikan login dan proteksi halaman](Topik-7/6-praktik-mengimplementasikan-login-dan-proteksi.html)

### Topik 8: Relationship dan Queries Dasar (2 jam)
- [Relationship antar tabel (One-to-Many)](Topik-8/relation-One-to-Many.html)
- [Menghubungkan Mahasiswa dengan tabel Jurusan](Topik-8/menghubungkan-mhs-jurusan.html)
- [Query builder dasar](Topik-8/query-builder-dasar.html)
- [Menampilkan data relasi](Topik-8/menampilkan-data-relasi.html)
- [Praktik: Membuat relasi Mahasiswa-Jurusan](Topik-8/praktik-membuat-relasi-mahasiswa-jurusan.html)

### Topik 9: Advanced Relationship dan Queries (2 jam)
- [Relationship lanjutan (Many-to-Many)](Topik-9/relationship-lanjutan-many-to-many.html)
- Menghubungkan Mahasiswa dengan tabel Mata Kuliah
- [Query builder lanjutan](Topik-9/query-builder-lanjutan-dalam-laravel-b.html)
- [Eager loading untuk optimasi query](Topik-9/eager-loading-untuk-optimasi-query.html)
- [Praktik: Membuat relasi Mahasiswa-Mata Kuliah](Topik-9/praktik-membuat-relasi-mahasiswa-mata-kuliah.html)

### Topik 10: Laravel UI dan UX (2 jam)
- [Penggunaan Blade template](Topik-10/penggunaan-blade-template.html)
- [Layout master dan komponen](Topik-10/layout-master-dan-komponen.html)
- [Integrasi CSS framework sederhana](Topik-10/integrasi-css-framework-sederhana.html)
- [Form dengan validasi JavaScript](Topik-10/form-dengan-validasi-javascript.html)
- [Datatable untuk manajemen data Mahasiswa](Topik-10/datatable-untuk-manajemen-data-mahasiswa-Tanpa-lib.html)
- [Praktik: Mempercantik tampilan aplikasi](Topik-10/praktik-mempercantik-tampilan-aplikasi.html)

### Topik 11: API dan JSON Response (2 jam)
- [Pengenalan RESTful API](Topik-11/laravel-api-tutorial.html#1-pengenalan-restful-api)
- [Membuat API endpoint untuk data Mahasiswa](Topik-11/laravel-api-tutorial.html#2-membuat-api-endpoint-untuk-data-mahasiswa)
- [Resource dan Collection di Laravel](Topik-11/laravel-api-tutorial.html#3-resource-dan-collection-di-laravel)
- [Testing API dengan Postman](Topik-11/laravel-api-tutorial.html#4-testing-api-dengan-postman)
- [Authentication API dengan token](Topik-11/laravel-api-tutorial.html#5-authentication-api-dengan-token)
- [Praktik: Membuat API sederhana untuk data Mahasiswa](Topik-11/praktik.html) 

### Topik 12: Project Akhir dan Deployment (2 jam)
- Penyempurnaan project aplikasi Mahasiswa
- Best practices pengembangan Laravel
- Persiapan deployment
- Review code dan optimasi performa
- Diskusi dan tanya jawab
- Praktik: Deployment aplikasi ke server sederhana

## Total Jam Pelatihan
- 12 Topik x 2 jam = 24 jam

Pelatihan ini menyediakan dasar yang kuat untuk pengembangan aplikasi web dengan Laravel, dengan fokus khusus pada pengelolaan data Mahasiswa sebagai contoh kasus nyata sepanjang kursus. Setiap pertemuan diakhiri dengan praktik untuk memastikan pemahaman peserta terhadap materi yang diajarkan.
