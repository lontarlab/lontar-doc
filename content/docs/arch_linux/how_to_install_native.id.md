+++
title = "Installing Linux (native)"
description = "Panduan langkah demi langkah tentang cara menginstal Linux menggunakan metode Native."
date = 2023-11-22T18:00:00+00:00
updated = 2023-11-22T18:00:00+00:00
draft = false
weight = 10
sort_by = "berat"
template = "docs/page.html"

[extra]
lead = 'Pelajari cara menginstal Linux menggunakan metode asli, termasuk mempersiapkan sistem Anda, mempartisi, dan pengaturan sistem.'
toc = true
top = false
+++

## Mempersiapkan Instalasi
Sebelum memulai, pastikan Anda memiliki drive USB yang dapat di-boot dengan ISO Arch Linux. Anda dapat membuat drive ini menggunakan alat bantu seperti Rufus atau dd.

## Mem-boot dari USB
Masukkan drive USB Anda dan boot ulang komputer Anda. Pastikan Anda masuk ke pengaturan BIOS atau UEFI untuk memprioritaskan booting dari USB.

## Lingkungan Langsung
Setelah boot dari USB, Anda akan berada di lingkungan langsung Arch Linux. Anda dapat memulai proses instalasi di sini.

### Mengatur Tata Letak Keyboard
Tata letak keyboard default adalah US. Jika Anda membutuhkan tata letak yang berbeda:
```bash
loadkeys [tata letak]
```

### Verifikasi Mode Booting
Periksa apakah sistem Anda mendukung UEFI:
```bash
ls /sys/firmware/efi/efivars
```

### Hubungkan ke Internet
Pastikan Anda tersambung ke internet. Untuk koneksi kabel, koneksi ini seharusnya otomatis. Untuk Wi-Fi:
```bash
iwctl
get-networks
station wlan0 sambungkan <jaringan Anda SSID>
```

### Perbarui Jam Sistem
Pastikan jam sistem sudah akurat:
```bash
timedatectl set-ntp true
```

### Mempartisi Disk
Partisi disk Anda sesuai dengan preferensi Anda menggunakan alat bantu seperti fdisk atau parted.

### Memformat Partisi
Memformat partisi. Misalnya, untuk partisi root:
```bash
mkfs.ext4 /dev/sdaX
```

### Memasang Partisi
Pasang partisi Anda ke direktori /mnt.

### Instalasi paket dasar
Sekarang, instal paket-paket dasar:
``` bash
pacstrap /mnt base linux linux-firmware
```

### Menghasilkan Fstab
Buatlah sebuah berkas fstab:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot
Mengubah root ke dalam sistem yang baru:
```bash
arch-chroot /mnt
```

### Zona Waktu
Mengatur zona waktu:
```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

### Pelokalan
Hapus komentar en_US.UTF-8 UTF-8 dan lokalisasi lain yang dibutuhkan di `/etc/locale.gen`, dan buatlah:
``` bash
locale-gen
```

### Konfigurasi Jaringan
Buat berkas nama host dan konfigurasikan berkas host.

### Initramfs
Membuat initramfs baru:
``` bash
mkinitcpio -P
```

### Kata Sandi Root
Mengatur kata sandi root:
```bash
passwd
```

### Boot Loader
Instal dan konfigurasikan bootloader seperti GRUB.

### Reboot
Keluar dari chroot, lepaskan partisi, dan boot ulang. Lepaskan drive USB sebelum melakukan boot ulang.
```bash
keluar
sudo reboot
```

Selamat! Anda seharusnya sudah menginstal sistem Arch Linux dasar.

Translated with www.DeepL.com/Translator (free version)