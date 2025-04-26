# Instalasi dan Konfigurasi Laragon - Tutorial Singkat

## Pengenalan
Laragon adalah lingkungan pengembangan lokal berbasis Windows yang terintegrasi untuk PHP, Apache, Nginx, MySQL dan banyak tools lainnya. Laragon sangat populer untuk pengembangan Laravel karena kemudahan instalasi dan konfigurasinya.

## Langkah-langkah Instalasi

### 1. Unduh Laragon
- Kunjungi situs resmi Laragon: [https://laragon.org/download/](https://laragon.org/download/)
- Pilih versi yang sesuai (disarankan Laragon Full)
- Unduh installer (.exe)
- https://github.com/leokhoa/laragon/releases
- https://github.com/leokhoa/laragon/releases/download/6.0.0/laragon-wamp.exe

### 2. Instalasi Dasar
- Jalankan file installer yang sudah diunduh
- Pilih lokasi instalasi (disarankan: `C:\laragon`)
- Pilih opsi instalasi sesuai kebutuhan:
  - Buat menu Start
  - Tambahkan ke PATH
  - Auto start saat Windows startup
- Klik "Next" dan tunggu proses instalasi selesai

### 3. Menjalankan Laragon
- Buka aplikasi Laragon dari menu Start atau shortcut desktop
- Akan muncul panel utama Laragon dengan tombol "Start All"
- Klik "Start All" untuk menjalankan server web dan database

### 4. Verifikasi Instalasi
- Buka browser dan akses `http://localhost` 
- Jika muncul halaman Laragon, instalasi berhasil
- Untuk memeriksa MySQL, buka Laragon → Menu → Database → HeidiSQL

## Konfigurasi Laragon untuk Laravel

### 1. Pengaturan PHP
- Buka Laragon → Menu → PHP → Version → Pilih PHP 8.1 atau versi terbaru
- Buka Laragon → Menu → PHP → Extensions → Aktifkan ekstensi berikut:
  - fileinfo
  - mbstring
  - openssl
  - pdo_mysql
  - curl
  - gd
  - zip

### 2. Pengaturan Web Server
- Buka Laragon → Menu → Apache → Version → Pilih Apache 2.4
- Untuk mengubah port (jika diperlukan): Menu → Preferences → Services & Ports

### 3. Pengaturan Database
- Buka Laragon → Menu → MySQL → Version → Pilih MySQL 8.0 (atau MariaDB)
- Password default untuk root kosong, bisa diubah melalui HeidiSQL

### 4. Mengaktifkan Pretty URL (Hostname Otomatis)
- Buka Laragon → Menu → Preferences → General
- Aktifkan "Auto create virtual hosts"
- Atur format hostname (misalnya: `{name}.test`)
- Restart Laragon

### 5. Menginstall Composer
- Buka Laragon → Menu → Tools → Quick add → pilih "Composer"
- Tunggu proses instalasi selesai

### 6. Membuat Project Laravel Pertama
- Klik kanan pada bagian kosong panel Laragon
- Pilih Quick Create → Laravel
- Masukkan nama project (misalnya: `mahasiswa-app`)
- Tunggu hingga proses pembuatan project selesai
- Akses project di browser: `http://mahasiswa-app.test`

## Pengaturan Lanjutan (Opsional)

### 1. Mengubah Versi PHP per Project
- Buat file `.env.php` di root folder project
- Isi dengan: `<?php return '8.1'; ?>` (sesuaikan dengan versi PHP yang diinginkan)

### 2. Menambahkan Tools Tambahan
- Klik kanan pada panel Laragon
- Pilih Tools → Quick add
- Pilih tool yang dibutuhkan (Node.js, Git, Visual Studio Code, dll)

### 3. Mengatur Lingkungan Development
- Buka Laragon → Menu → Preferences → Services
- Aktifkan service yang diperlukan untuk startup otomatis

## Troubleshooting Umum

### Port yang Sudah Digunakan
- Buka Laragon → Menu → Preferences → Services & Ports
- Ubah port Apache/Nginx atau MySQL jika terjadi konflik

### Permission Denied
- Jalankan Laragon sebagai Administrator (klik kanan pada icon → Run as administrator)

### Database Connection Failed
- Periksa service MySQL apakah sudah berjalan
- Periksa konfigurasi `.env` file di project Laravel

## Kesimpulan
Laragon kini telah terinstal dan siap digunakan untuk pengembangan aplikasi Laravel. Anda dapat dengan mudah membuat, mengelola, dan mengakses project-project Laravel Anda dalam lingkungan pengembangan lokal yang terintegrasi.
