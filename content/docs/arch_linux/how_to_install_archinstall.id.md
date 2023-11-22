+++
title = "Installing Arch Linux with (archinstall)"
description = "Panduan langkah demi langkah tentang cara menginstal Arch Linux menggunakan skrip archinstall."
date = 2023-11-22T18:00:00+00:00
updated = 2023-11-22T18:00:00+00:00
draft = false
weight = 10
sort_by = "berat"
template = "docs/page.html"

[extra]
lead = 'Pelajari cara menginstal Arch Linux dengan mudah menggunakan skrip 1archinstall, termasuk persiapan sistem, partisi, dan pengaturan.'
toc = true
top = false
+++

## Mempersiapkan Instalasi
Sebelum memulai, pastikan Anda memiliki drive USB yang dapat di-booting dengan ISO Arch Linux terbaru. Buatlah drive ini dengan menggunakan alat bantu seperti Rufus atau dd.

## Mem-boot dari USB
Masukkan drive USB Anda dan boot ulang komputer Anda. Masuk ke pengaturan BIOS atau UEFI untuk mengaktifkan booting dari USB.

### Memeriksa informasi disk
Periksa informasi disk Anda:
```bash
lsblk
```

### Buat partisi baru (jika tidak memiliki ruang kosong di partisi)
Periksa informasi disk Anda:
```bash
cfdisk /dev/<partisi disk Anda>
```

### Buatlah partisi baru (jika tidak ada ruang kosong dalam partisi)
Buatlah 2 partisi untuk `/` dan `/boot`. partisi boot hanya membutuhkan 1GB atau lebih rendah, dan partisi home
yang disarankan adalah `50GB ++`.

### Memformat partisi
memformat partisi home dan partisi boot Anda:
```bash
mkfs.ext4 /dev/<partisi disk Anda>/<partisi home Anda>
mkfs.vfat /dev/<partisi disk Anda>/<partisi boot Anda>
```

#### Menyambung ke Internet
Pastikan Anda telah tersambung ke internet. Untuk sambungan kabel, sambungan ini seharusnya otomatis. Untuk Wi-Fi:
```bash
iwctl
get-networks
station wlan0 sambungkan <jaringan Anda SSID>
```

### Memulai Archinstall
Setelah boot ke lingkungan live Arch Linux, Anda akan memulai instalasi dengan skrip `archinstall`:
``` bash
archinstall
```

### Memperbarui Archinstall (opsional)
Jika Anda ingin memperbarui
skrip `archinstall` Anda:
```bash
sudo pacman -Syy archinstall
```

### Memilih Bahasa dan Tata Letak Papan Ketik
- Pilih bahasa yang Anda inginkan untuk proses instalasi.
- Atur tata letak papan ketik agar sesuai dengan preferensi Anda.

### Konfigurasi Jaringan
- Sambungkan ke internet, baik melalui koneksi kabel atau Wi-Fi.

### Pemartisian dan Pemformatan Disk
- `archinstall` akan memandu Anda dalam mempartisi disk. Anda dapat memilih partisi otomatis atau partisi manual.
- Setelah mempartisi, pilih jenis sistem berkas untuk partisi Anda.

### Memilih Lingkungan Desktop (Opsional)
- Anda dapat memilih untuk menginstall lingkungan desktop atau window manager, atau melanjutkan dengan installasi minimal.

### Perangkat Lunak Tambahan (Opsional)
- Instal paket perangkat lunak tambahan sesuai kebutuhan.

### Mengatur Nama Host dan Kata Sandi Root
- Tentukan nama host untuk sistem Anda.
- Tetapkan kata sandi root yang aman.

### Membuat Akun Pengguna
- Buat akun pengguna non-root yang baru dan atur kata sandinya.

### Instalasi Bootloader
- Pilih dan instal bootloader, biasanya GRUB.

### Meninjau dan Menginstal
- Tinjau pilihan konfigurasi Anda. Jika semuanya memuaskan, lanjutkan dengan instalasi.

### Reboot ke Sistem Baru Anda
Setelah instalasi selesai:
```bash
sudo reboot
```
- Lepaskan drive USB selama proses reboot.

Selamat! Anda telah berhasil menginstall Arch Linux menggunakan skrip `archinstall`.
