# Instalasi dan Konfigurasi Visual Studio Code - Tutorial Singkat

## Pengenalan
Visual Studio Code (VS Code) adalah editor kode sumber yang ringan namun kuat, tersedia untuk Windows, macOS, dan Linux. VS Code menyediakan fitur-fitur untuk pengembangan web modern, termasuk dukungan untuk PHP dan Laravel.

## Langkah-langkah Instalasi

### 1. Unduh Visual Studio Code
- Kunjungi situs resmi: [https://code.visualstudio.com/](https://code.visualstudio.com/)
- Klik tombol "Download" sesuai sistem operasi Anda (Windows, macOS, atau Linux)
- Unduh installer (.exe untuk Windows, .dmg untuk macOS, atau paket untuk Linux)

### 2. Instalasi Dasar
#### Untuk Windows:
- Jalankan file installer (.exe) yang sudah diunduh
- Setujui perjanjian lisensi
- Pilih lokasi instalasi (disarankan biarkan default)
- Pilih opsi tambahan:
  - Tambahkan "Open with Code" di menu konteks Explorer
  - Tambahkan VS Code ke PATH
  - Daftarkan VS Code sebagai editor untuk tipe file yang didukung
- Klik "Next" dan "Install", tunggu proses selesai

#### Untuk macOS:
- Buka file .dmg yang sudah diunduh
- Drag aplikasi Visual Studio Code ke folder Applications
- Launch aplikasi dari folder Applications atau Launchpad

#### Untuk Linux (Ubuntu/Debian):
```bash
sudo apt update
sudo apt install software-properties-common apt-transport-https wget
wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
sudo apt update
sudo apt install code
```

### 3. Konfigurasi Awal
- Buka VS Code setelah instalasi selesai
- Pilih tema tampilan (Light/Dark)
- Atur font dan ukuran sesuai preferensi:
  - File > Preferences > Settings (Windows/Linux)
  - Code > Preferences > Settings (macOS)
  - Cari "font" dan sesuaikan pengaturan

## Plugin Penting untuk Pengembangan Laravel

### 1. Plugin Dasar PHP dan Laravel
- **PHP Intelephense**: Linter dan IntelliSense untuk PHP
  - Klik tab Extensions (Ctrl+Shift+X)
  - Cari "PHP Intelephense"
  - Klik Install

- **Laravel Extension Pack**: Paket ekstensi untuk Laravel
  - Cari "Laravel Extension Pack"
  - Klik Install (Berisi beberapa ekstensi populer untuk Laravel)

- **Laravel Blade Snippets**: Snippet dan syntax highlighting untuk Blade
  - Cari "Laravel Blade Snippets"
  - Klik Install

- **Laravel Artisan**: Menjalankan perintah Artisan dari VS Code
  - Cari "Laravel Artisan"
  - Klik Install

### 2. Plugin untuk Development Workflow
- **PHP Debug**: Debugger untuk PHP (menggunakan XDebug)
  - Cari "PHP Debug"
  - Klik Install

- **GitLens**: Integrasi Git yang lebih kuat
  - Cari "GitLens"
  - Klik Install

- **DotENV**: Syntax highlighting untuk file .env
  - Cari "DotENV"
  - Klik Install

- **EditorConfig for VS Code**: Konsistensi kode antar editor
  - Cari "EditorConfig for VS Code"
  - Klik Install

### 3. Plugin untuk Front-end
- **Prettier - Code formatter**: Formatter untuk HTML, CSS, JS
  - Cari "Prettier - Code formatter"
  - Klik Install

- **ESLint**: Linter untuk JavaScript
  - Cari "ESLint"
  - Klik Install

- **Tailwind CSS IntelliSense**: Jika menggunakan Tailwind CSS
  - Cari "Tailwind CSS IntelliSense"
  - Klik Install

## Konfigurasi Plugin untuk Laravel

### 1. Konfigurasi PHP Intelephense
- Buka Settings (Ctrl+,)
- Cari "intelephense"
- Sesuaikan pengaturan berikut:
  - `intelephense.files.maxSize`: Atur ke nilai yang lebih besar jika bekerja dengan file besar
  - `intelephense.environment.phpVersion`: Sesuaikan dengan versi PHP yang digunakan (mis. "8.1.0")

### 2. Konfigurasi Laravel Blade
- Buka Settings (Ctrl+,)
- Cari "blade"
- Pastikan "Files: Associations" memiliki `*.blade.php` yang terkait dengan `blade`

### 3. Konfigurasi Git
- Buka Settings (Ctrl+,)
- Cari "git enabled"
- Pastikan "Git: Enabled" dicentang
- Konfigurasikan identitas Git jika belum:
  ```bash
  git config --global user.name "Nama Anda"
  git config --global user.email "email@anda.com"
  ```

## Kustomisasi Tambahan

### 1. Membuat Kunci Pintas (Keyboard Shortcuts)
- File > Preferences > Keyboard Shortcuts (Windows/Linux)
- Code > Preferences > Keyboard Shortcuts (macOS)
- Tambahkan shortcuts yang sering digunakan, misalnya:
  - Format dokumen: Shift+Alt+F
  - Terminal baru: Ctrl+Shift+`

### 2. Mengatur Autosave
- File > Preferences > Settings
- Cari "Auto Save"
- Pilih opsi yang diinginkan (misalnya "afterDelay")

### 3. Mengatur Spasi vs Tab
- File > Preferences > Settings
- Cari "Tab Size" dan "Insert Spaces"
- Atur sesuai standar proyek (umumnya 4 spasi untuk PHP/Laravel)

## Integrasi dengan Laragon

### 1. Membuka Project dari Laragon
- Klik kanan pada proyek di panel Laragon
- Pilih "Open in VS Code"
- Alternatinya, buka VS Code dan pilih File > Open Folder, kemudian navigasi ke folder proyek (C:\laragon\www\nama-proyek)

### 2. Mengatur Terminal Terintegrasi
- Terminal > New Terminal (atau Ctrl+Shift+`)
- Pastikan terminal mengarah ke folder proyek
- Untuk mengubah terminal default:
  - File > Preferences > Settings
  - Cari "terminal.integrated.defaultProfile.windows" (untuk Windows)
  - Pilih "Command Prompt", "PowerShell", atau "Git Bash"

## Tips Menggunakan VS Code dengan Laravel

1. **Gunakan Command Palette**: Tekan Ctrl+Shift+P untuk akses cepat ke semua perintah VS Code
2. **Buka Terminal Terintegrasi**: Gunakan Ctrl+` untuk toggle terminal
3. **Multi-kursor**: Tahan Alt sambil klik untuk menambahkan kursor di beberapa lokasi
4. **Navigasi Cepat**: Ctrl+P untuk mencari file, Ctrl+Shift+F untuk pencarian global
5. **Snippets**: Ketik "blade" atau "php" diikuti dengan Tab untuk menggunakan snippet

Dengan konfigurasi di atas, VS Code Anda siap digunakan untuk pengembangan Laravel secara produktif.
