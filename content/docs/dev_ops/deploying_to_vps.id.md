+++
title = "Deploying to VPS"
description = ""
date = 2023-11-20T18:00:00+00:00
updated = 2023-11-20T18:00:00+00:00
draft = false
weight = 10
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'This doc will provide guide on deploying to VPS lontar droplet'
toc = true
top = false
+++

Panduan ini dibuat secara spesifik untuk deploy app ke lontar deoplet. Namun, prinsip yang sama dapat diaplikasikan ke vps lain.

## SSH into the server
Ssh ke server menggunakan ssh client, kita akan menggunakan open-ssh sebagai contoh
```bash
ssh root@165.232.160.224
```
Masukan passwordnya jika diminta

## Clone the app
Selanjutnya kita akan close app kita dari github.
masuk ke directory app dan clone app nya dari repo github.
```bash
cd app
git clone git@github.com:lontarlab/certificate-lookup.git
```

## Check for available port
Check terlebih dahulu port yang dipakai menggunakan docker
```bash
docker ps
```
Jika tidak ada conflict port kita bisa deploy container kita

## Deploying
Masuk ke directory app kita lalu jalankan containernya
```bash
cd certificate-lookup
docker compose up -d
```
Jika tidak menggunakan compose
```bash
cd certificate-lookup
docker build --tag=app .
docker run --name=app --rm --detach app
```

## Set Reverse Proxy
Kita menggunakan nginx untuk web servernya.  
Buat config baru untuk appnya 
```bash
vim /etc/nginx/sites-enabled/sertifikat.lontarlab.com
```

```
server {
        server_name sertifikat.lontarlab.com www.sertifikat.lontarlab.com;
        location / {
               proxy_pass http://127.0.0.1:3000;
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
        }
}
```
Jangan lupa untuk membuat symlink ke sites-available
```bash
sudo ln -s  ./sertifikat.lontarlab.com ../sites-available/sertifikat.lontarlab.com
```

## Enabling ssl
Hunakan let's encrypt cerbot
```bash
certbot --nginx -d sertifikat.lontarlab.com
```
Selesai
