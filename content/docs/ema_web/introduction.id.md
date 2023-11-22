+++
title = "On-boarding"
description = ""
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 10
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'Doc ini bertujuan untuk membantu developer baru masuk ke project Ema web'
toc = true
top = false
+++

## Quick Start
Bagian ini akan menjelaskan cara untuk menjalankan Ema Web di lokal.

### Requirements

Berikut merupakan software yang dibutuhkan

- Node JS
- Node Package Manager
- Docker
- Supabase CLI

#### Node JS  
Install menggunakan package manager kita. Saat ini kita menggunakan versi v21.2.0. Tapi, menggunakan latest atau LTS juga bisa.
```bash
sudo pacman -S nodejs
```
Jika ingin menginstall dengan cara lain silahkan saja.

#### Node Package Manager

Install menggunakan package manager kita. Gunakan versi latest.
```bash
sudo pacman -S npm
```
Jika ingin menginstall dengan cara lain silahkan saja.

#### Docker Engine
Kita akan membutuhkan docker engine.
```bash
sudo pacman -S docker
```

Untuk *windows* cukup download docker desktop karena sudah termasuk docker engine.

Jika ingin menjalankan docker sebagai non root, tambahkan user ke usergroup docker
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

Nyalakan docker daemon
```bash
sudo systemctl enable docker
sudo systemctl enable containerd
```

Restart pc untuk memastikan perubahan usergroup dan daemon tersimpan.
Untuk melakukan verifikasi bisa menggunakan command docker sebagai non root
```bash
docker run hello-world
```

#### Supabase CLI
Jika menggunakan arch kita bisa menggunakan AUR. Gunakan versi latest
```bash
yay -S supabase-bin
```
Untuk Mac bisa menggunakan brew
```bash
brew install supabase/tap/supabase
```

Kalah bukan keduanya ada opsi lain di [Dokumentasi](https://supabase.com/docs/guides/cli/getting-started)

### Cloning the repository
Passtikan git sudah terinstall. Jika belum memiliki akses reponya, request invite terlebih dahulu.
Setelah mempunyai akses repo.
```bash
git clone git@github.com:lontarlab/cluster-ema-web.git
```
Lalu install package npm
```bash
cd cluster-ema-web
npm install
```

### Setting up the local database

#### Run database and migration
Kita akan menjalankan local database menggunakan supabase-cli, pastikan sudah terinstall.
Di directory yang dipilih jalankan command
```bash
supabase init
```
Ini akan membuat directory baru yang bernama supabase, masuk ke directorynya dan jalankan databasenya
```bash
cd supabase
supabase start
```
Jika berhasil maka akan mengeluarkan output seperti ini
```bash
API URL: http://localhost:54321
GraphQL URL: http://localhost:54321/graphql/v1
DB URL: postgresql://postgres:postgres@localhost:54322/postgres
Studio URL: http://localhost:54323
Inbucket URL: http://localhost:54324
JWT secret: super-secret-jwt-token-with-at-least-32-characters-long
anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0
service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImV4cCI6MTk4MzgxMjk5Nn0.EGIM96RAZx35lJzdJsyH-qQwv8Hdp7fsn3W0YpN81IU
```
Atau bisa di verifikasi dengan melihat container yang berjalan
```bash
docker ps
```
Selanjutnya kita akan menjalankan migration dan seedernya

Download [disini](https://drive.google.com/drive/folders/10_8ibm83SqRNCWrDtalSUZQi9LMJ9I7g?usp=sharing)

Kita akan menjalankannya manual menggunakan psql
```bash
psql 'postgresql://postgres:postgres@localhost:54322/postgres'
\i path/to/schema.sql
\i path/to/seeder.sql
```
Ini akan mengisi database dengan data real dari October 2023

#### Setup auth
Karena seedernya tidak membuat user auth kita harus membuatnya sendiri

Buka supabase studio [http://localhost:54323](http://localhost:54323/project/default/auth/users) dan check users table dan lihat user yang tersedia
  
<img src="../users.png" alt="Auth" style="max-width: 100%; margin-bottom: 12px"/>    

Lalu ke halaman auth dan bikin usernya, samakan email dan password

<img src="../auth.png" alt="Auth" style="max-width: 100%; margin-bottom: 12px"/>

### Run the project
Pastikan kita menggunakan db local dengan set use production db ke false di .env.development
```
NEXT_PUBLIC_USE_PRODUCTION_DB=false
```

Setup seelsai sekarang kita bisa jalankan appnya.
```bash
npm run dev
```

## Convention
Bagian ini akan menampilkan list convensi yang sudah disetujui oleh tim.

### Database
#### Naming 

- camelCase untuk nama table
- Kata jamak untuk nama table
- Bahasa inggris untuk key / field

#### Table 

- Semua table harus setidaknya memiliki createdAt
- Jangan pernah menghapus record, gunakan isDeleted atau deletedAt
- Bikin view jika diperlukan

#### Performance 

- Buat index untuk table yang sering diquery
- Jalan vacuum sesekali
- Hindari penggunaan JSONB

#### Safety
- Jangan pernah jalankan sql destructive sebelum menjalankan count dan memastikan semuanya sudah benar
- Jika db prod unresponsive segera restart database
- Update file schema jika ada perubahan atau tambahan table / view
- update the schema file when adding or updating table / view

### Code

#### Naming

- Gunakan snake_case untuk file nextjs
- Gunakan PascalCase untuk file library
- Gunakan camcelCase untuk variable

#### NextJS

- Gunakan function daripada arrow function untuk function non lambda
- Gunakan npm untuk package manager
- Gunakan client side data fetching
- Gunakan page router
- Gunakan sweet alert untuk menampilkan informasi pada user
- Gunakan type yang sesuai daripada any
- Validasi data masuk dan keluar database menggunkan schema joi
- Gunakan let daripada var
- Gunakan const jika bisa
- Panggil operasi database lewat nextjs api
- Beri child component yang dihasilkan loop key
- Gunakan validasi form sebelum mengirim data
- Gunakan view untuk sumber data datatable
- Pastikan ui element sudah memiliki feedback dan contrast yang cukup
