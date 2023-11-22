+++
title = "Creating Docker Container"
description = ""
date = 2023-11-20T18:00:00+00:00
updated = 2023-11-20T18:00:00+00:00
draft = false
weight = 10
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'Doc ini adalah panduan untuk membuat container docker untuk keperluan deployment'
toc = true
top = false
+++

## Installation
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

## Creating the container
Akan ada dua contoh untuk membuat container yaitu Node dan Go. Karena akan ada sedikit perbedaan dintara compiled app dengan interpreted app. Contoh ini bisa disesuaikan dengab kebutuhan container yang akan dibuat.

### Node Container
Pertama kita akan membuat Dockerfile di root directory project kita. Container image akan dibuat di dalam file ini.

Biasanya untuk setiap runtime tersedia official image yang disediakan, contohnya node punya official imagenya sendiri.  
Jadi daripada pake base OS
```Dockerfile
FROM ubuntu
```
Langsung aja pake base runtime yang akan kita gunakan
```Dockerfile
FROM node
```
Tapi, ini masih belum optimal, kalau memungkikan kita harus memilih base image yang minimal, contohnya alpine. Artinya base OS yang akan kita gunakan adalah alpine dan cenderung lebih ringan dibanding debangan ubntun, kita juga harus mencantumkan versi spesifik.
```Dockerfile
FROM node:21-alpine3.17
```
Harus diingat bahwa kalau menggunakan base OS yang ringan itu akan memiliki lebih sedikit command yang tersedia, kalau misalkan ada masalah dengan image alpine, coba dengan image yang lebih besar seperti ubuntu atau debian.

Lalu copy isi project nya dan jalankan command untuk memulai appnya.
```Dockerfile
FROM node:21-alpine3.17
COPY . .
RUN npm install
CMD ["npm", "run", "dev"]
```
Ini bakal jalan cuman masih bisa di improve.
This is will work but we can still improve it.    
Tambahkan working directory agar lebih teroganisir.
```Dockerfile
FROM node:21-alpine3.17
WORKDIR /usr/src/app
COPY . .
RUN npm install
CMD ["npm", "run", "dev"]

```
Kita juga bisa mencantumkan port yang akan kita expose
**Ini tidak melakukan apa-apa hanya memperjelas niatan kita untuk menggunakan port tersebut**
```Dockerfile
FROM node:21-alpine3.17
WORKDIR /usr/src/app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "run", "dev"]
```
Hal terakhir yang kita bisa lakukan untuk mengimprove ini adalah untuk menginstall node_modules di dalam container nya daripada mengcopy node_modules yang ada di local.  
Contoh ini spesifik untuk app Node, tapi prinsipnya bisa di aplikasi untuk runtime lain.  
Pertama kita buat file .dockerignore
```Dockerfile
node_modules
```
Ini akan mencegah node_modules tercopy kedalam container.  
Masih ada bebearapa improvement yang bisa dilakukan tapi untuk basic ini sudah cukup.
### Go Container
Secara garis besar prosesnya sama dengan pembuatan container Node.  
Perbedaanya ada disini kita akan menggunakan multi stage container. Karena go akana menghasilkan executable kita bisa membuat executable tersebut dan memindahkannya ke base container yang sangat ringan.
```Dockerfile
FROM golang:1.21.3-bookworm as build

WORKDIR /app

COPY main.go .
COPY go.mod .
COPY go.sum .

RUN go mod tidy

COPY public ./public
COPY views ./views

# Build script for production
RUN go build -ldflags="-linkmode external -extldflags -static" -tags netgo
```
Dalam contoh ini kita menggunakan tag bookworm (Debian) dan bukan alpine karena ini tidak akan ngaruh, soalnya ini hanya digunakan dalam first stage container yang artinya tidak akan terbawa ke container akhir.
```Dockerfile
FROM golang:1.21.3-bookworm as build

WORKDIR /app

COPY main.go .
COPY go.mod .
COPY go.sum .

RUN go mod tidy

COPY public ./public
COPY views ./views

RUN go build -ldflags="-linkmode external -extldflags -static" -tags netgo

# Second container stage
FROM scratch
COPY --from=build /app/certlookup ./certlookup
COPY --from=build /app/views ./views/
COPY --from=build /app/public ./public/
EXPOSE 3000
ENTRYPOINT ["./certlookup"]
```
Di stage kedua kita hanya memindahkan executable dan assets yang diperlukan dari first stage dan menjalakannya. Namun, karena kita menggunakan scratch sebagai base kita tidak mempunyai command apapun tapi container nya jadi sangat ringan.

## Docker Compose
Jika container kita membutuhkan dua atau lebih app untuk berjalan bersamaan untuk berfungsi kita membutuhkan docker ocmpose.  
Compose pada dasarnya akan menggabungkan dua docker image yang dijalankan bersamaan menggunakan config.

Pertama kita harus menginstall docker compose terlebih dahulu menggunakan package manager
```bash
sudo pacman -S docker-compose
```
Lalu di directory app kita buat compose file namanya `docker-compose.yml`
Kita akan menggunakan aplikasi go yang sudah kita buat sebagai contoh, karena appnya membutuhkan postgresql untuk dapat berfungsi kita akan menggunakan docker compose.
```yml
version: '3'
services:
  postgresdb:
    image: postgres:16.0-alpine3.18
    container_name: postgresdb
    restart: always
    ports:
      - 5432:5432
    expose:
      - 5432
    environment:
      - POSTGRES_PASSWORD=ucingdialungken
      - POSTGRES_DB=certificate
    volumes:
      - ./pgdata:/var/lib/postgresql/data
      - ./db:/docker-entrypoint-initdb.d

  certlookup:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: certlookup
    depends_on:
      - postgresdb
    ports:
      - 3000:3000
    expose:
      - 3000
```
Peratama kita pull official image postgresql dengan tag alpine dan versinya, memberi containernya nama lalu menentukan portnya. environment ada environment variable appnya dan volumes digunakan untuk menyimpan data karena bawaanya docker bersifat ephemeral atau sementara.

Dan untuk app gonya karena kita sudah memiliki Docker file kita hanya perlu memanggil build, depends_on artinya container go ini tidak akan berjalan jika postgresdb tidak ada.

Command untuk menjalankan script compose adalah
```bash
docker compose up
```
