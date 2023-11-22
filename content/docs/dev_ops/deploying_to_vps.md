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

This is guide is specifically made for deploying app to lontar droplet. However, the same principle can be used in any other vps.

## SSH into the server
ssh into the server as root using our ssh client, we'll be using open-ssh as an example
```bash
ssh root@165.232.160.224
```
Enter the password when prompted

## Clone the app
Next we will clone the app from github.  
First we enter the app directory and clone the github repository
```bash
cd app
git clone git@github.com:lontarlab/certificate-lookup.git
```

## Check for available port
You can check the available port using docker
```bash
docker ps
```
If there's no port conflict we can deploy our container

## Deploying
Enter the app directory and run the container
```bash
cd certificate-lookup
docker compose up -d
```
If not using compose
```bash
cd certificate-lookup
docker build --tag=app .
docker run --name=app --rm --detach app
```

## Set Reverse Proxy
We're using nginx for web server.
Create new config file for the app.
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
Don't forget to link it to sites-available
```bash
sudo ln -s  ./sertifikat.lontarlab.com ../sites-available/sertifikat.lontarlab.com
```

## Enabling ssl
use let's encrypt certbot
```bash
certbot --nginx -d sertifikat.lontarlab.com
```

Done
