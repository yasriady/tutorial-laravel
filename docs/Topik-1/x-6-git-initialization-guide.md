# Inisialisasi Repositori Git dan Commit Pertama - Tutorial Singkat

## Pengenalan
Git adalah sistem kontrol versi terdistribusi yang memungkinkan Anda melacak perubahan kode, berkolaborasi dengan developer lain, dan mengelola versi proyek. Dalam tutorial ini, kita akan belajar cara menginisialisasi repositori Git untuk proyek Laravel dan melakukan commit pertama.

## Prasyarat
- Git terinstal di komputer Anda
- Proyek Laravel sudah dibuat
- Visual Studio Code (atau editor lain)

## Langkah-langkah Tutorial

### 1. Instalasi Git (jika belum terinstal)

#### Untuk Windows
- Unduh Git dari [git-scm.com](https://git-scm.com/downloads)
- Jalankan installer dan ikuti petunjuk instalasi
- Opsi yang disarankan:
  - Gunakan Git dari Git Bash dan Command Prompt
  - Gunakan OpenSSL untuk HTTPS
  - Checkout Windows-style, commit Unix-style
  - Gunakan MinTTY sebagai terminal default

#### Untuk macOS
- Instal melalui Homebrew: `brew install git`
- Atau unduh dari [git-scm.com](https://git-scm.com/downloads)

#### Untuk Linux (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install git
```

### 2. Konfigurasi Git Global

- Buka Terminal atau Command Prompt
- Konfigurasi nama pengguna:
  ```bash
  git config --global user.name "Nama Anda"
  ```
- Konfigurasi email:
  ```bash
  git config --global user.email "email@anda.com"
  ```
- (Opsional) Konfigurasi editor default:
  ```bash
  git config --global core.editor "code --wait"
  ```

### 3. Inisialisasi Repositori Git

- Buka Terminal di VS Code atau Command Prompt
- Navigasi ke direktori proyek Laravel:
  ```bash
  cd path/ke/proyek/laravel
  # contoh: cd C:\laragon\www\mahasiswa-app
  ```
- Inisialisasi repositori Git:
  ```bash
  git init
  ```
- Anda akan melihat pesan bahwa repositori kosong telah diinisialisasi

### 4. Membuat File .gitignore

Laravel sudah menyertakan file `.gitignore` default yang baik, tapi kita bisa memastikan dan menyesuaikannya:

- Buka file `.gitignore` di root proyek dengan VS Code
- Pastikan file tersebut sudah berisi entri berikut:
  ```
  /node_modules
  /public/hot
  /public/storage
  /storage/*.key
  /vendor
  .env
  .env.backup
  .phpunit.result.cache
  Homestead.json
  Homestead.yaml
  npm-debug.log
  yarn-error.log
  ```
- Tambahkan entri lain sesuai kebutuhan, misalnya:
  ```
  /.idea
  /.vscode
  *.log
  .DS_Store
  ```

### 5. Memeriksa Status Repositori

- Periksa status repositori:
  ```bash
  git status
  ```
- Anda akan melihat daftar file yang belum di-track (untracked files) dan file yang telah dimodifikasi

### 6. Menambahkan File ke Staging Area

- Tambahkan semua file (kecuali yang di-exclude oleh .gitignore):
  ```bash
  git add .
  ```
- Atau tambahkan file tertentu saja:
  ```bash
  git add nama-file
  ```

### 7. Melakukan Commit Pertama

- Commit perubahan dengan pesan deskriptif:
  ```bash
  git commit -m "Initial commit: Proyek Laravel mahasiswa-app"
  ```
- Pesan commit sebaiknya menjelaskan apa yang Anda lakukan, bukan bagaimana Anda melakukannya

### 8. Melihat Log Commit

- Lihat log commit untuk memastikan commit berhasil:
  ```bash
  git log
  ```
- Ini akan menampilkan informasi commit, termasuk ID commit, author, tanggal, dan pesan

### 9. Membuat Branch Development (Opsional)

- Sebagai praktik terbaik, buat branch development terpisah:
  ```bash
  git branch development
  ```
- Pindah ke branch development:
  ```bash
  git checkout development
  ```
- Atau dalam satu perintah:
  ```bash
  git checkout -b development
  ```

### 10. Menghubungkan dengan Repositori Remote (GitHub/GitLab/Bitbucket)

#### Membuat Repositori Baru di GitHub
- Buka [GitHub](https://github.com/) dan login
- Klik tombol "New repository"
- Isi nama repositori: `mahasiswa-app`
- Jangan centang opsi "Initialize this repository with a README"
- Klik "Create repository"

#### Menghubungkan Repositori Lokal dengan Remote
- Tambahkan URL repositori remote:
  ```bash
  git remote add origin https://github.com/username/mahasiswa-app.git
  ```
- Push commit ke repositori remote:
  ```bash
  git push -u origin master
  ```
- Jika menggunakan branch development:
  ```bash
  git push -u origin development
  ```

### 11. Praktik Terbaik Git untuk Proyek Laravel

#### Strategi Branching
- `master` atau `main`: Branch produksi, hanya berisi kode stabil
- `development`: Branch pengembangan utama
- `feature/nama-fitur`: Branch untuk fitur tertentu
- `hotfix/nama-perbaikan`: Branch untuk perbaikan cepat

#### Pola Commit
Gunakan awalan pada pesan commit untuk memperjelas tujuan:
- `feat:` untuk fitur baru
- `fix:` untuk perbaikan bug
- `docs:` untuk perubahan dokumentasi
- `style:` untuk perubahan formatting
- `refactor:` untuk refactoring kode
- `test:` untuk menambah atau memperbaiki test
- `chore:` untuk perubahan build atau tools

Contoh:
```bash
git commit -m "feat: Tambah fitur login mahasiswa"
git commit -m "fix: Perbaiki validasi input NIM"
```

### 12. Workflow Git Sehari-hari

1. Sebelum mulai bekerja, pull perubahan terbaru:
   ```bash
   git pull origin development
   ```

2. Buat branch fitur baru:
   ```bash
   git checkout -b feature/nama-fitur
   ```

3. Lakukan perubahan dan commit secara berkala:
   ```bash
   git add .
   git commit -m "feat: Deskripsi fitur"
   ```

4. Push branch ke remote:
   ```bash
   git push -u origin feature/nama-fitur
   ```

5. Setelah selesai, merge ke development melalui Pull Request di GitHub/GitLab/Bitbucket

## Troubleshooting

### Konflik Merge
- Jika terjadi konflik saat merge atau pull:
  ```bash
  git status  # Identifikasi file yang berkonflik
  ```
- Buka file yang berkonflik, dan selesaikan konflik
- Setelah selesai:
  ```bash
  git add .
  git commit -m "fix: Resolve merge conflicts"
  ```

### Membatalkan Perubahan
- Membatalkan perubahan belum di-staged:
  ```bash
  git checkout -- nama-file
  ```
- Membatalkan perubahan yang sudah di-staged:
  ```bash
  git reset nama-file
  git checkout -- nama-file
  ```
- Membatalkan commit terakhir tapi menyimpan perubahan:
  ```bash
  git reset --soft HEAD~1
  ```

## Menggunakan Git dengan VS Code

VS Code memiliki integrasi Git yang bagus:

1. Panel Git di sidebar (ikon cabang)
2. Indikasi perubahan di editor
3. Diff visual untuk file yang dimodifikasi
4. Penyelesaian konflik dengan bantuan visual
5. Commit, push, dan pull melalui UI

## Kesimpulan

Anda telah berhasil menginisialisasi repositori Git untuk proyek Laravel dan melakukan commit pertama. Praktik ini sangat penting untuk mengelola versi proyek dan berkolaborasi dengan developer lain. Ingatlah untuk selalu melakukan commit dengan pesan yang jelas, dan gunakan branch untuk mengembangkan fitur terpisah.
