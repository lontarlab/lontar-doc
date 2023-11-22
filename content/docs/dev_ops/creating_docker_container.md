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
lead = 'This doc will provide guide on creating docker container for the purpose of deployment'
toc = true
top = false
+++

## Installation
What we need is the Docker Engine, install it using our package manager. Use the latest version.
```bash
sudo pacman -S docker
```

For *windows* just download the Docker Desktop it comes with docker engine. Use the latest version.

Also we might want to run docker as non root, we can do so by adding the current user to the docker group
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

Then enable the docker daemon
```bash
sudo systemctl enable docker
sudo systemctl enable containerd
```

You can reboot our pc to make sure that the group and daemon change is applied.  
To verify if docker is working and can be run as a non root user
```bash
docker run hello-world
```

## Creating the container
There will be two example of creating container Node and Go. Because there will be a slight difference between a compiled app like Go and interpreted app like Node. With both example covered we can adapt the needs for our specific container.

### Node Container
First we need to create a Dockerfile in our root project directory. We will make our container image inside this file.

For every runtime usually there's an official docker image for that, for example Node has its own official image.  
So instead of starting from the base OS image
```Dockerfile
FROM ubuntu
```
We start from the base runtime image
```Dockerfile
FROM node
```
However, this is still not optimal, if possible choose the most minimal image, it's usually the alpine tag. This mean that the base OS is alpine instead of something heavier like ubuntu, also we need to specify the version.
```Dockerfile
FROM node:21-alpine3.17
```
Keep in mind that using a light image means that it will have less command, we can always change it to debian or ubuntu based image if it becomes a problem.

Then we copy our project file and run the build command.
```Dockerfile
FROM node:21-alpine3.17
COPY . .
RUN npm install
CMD ["npm", "run", "dev"]
```
This is will work but we can still improve it.  
Let's set the working directory so it's more organized
```Dockerfile
FROM node:21-alpine3.17
WORKDIR /usr/src/app
COPY . .
RUN npm install
CMD ["npm", "run", "dev"]

```
We can also specify the port we're going to expose.  
**This does not do anything, it's just make our intention clear, we still have to set the port in the code**
```Dockerfile
FROM node:21-alpine3.17
WORKDIR /usr/src/app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "run", "dev"]
```
The last thing we can do to improve it is to install the node_modules inside the container rather than copying it.  
The example would be node specific but the concept can be applied to other runtime.  
First we create a .dockerignore
```Dockerfile
node_modules
```
This will prevent the node_modules from being copied.   
There are still improvement that can be made, this is just the basic acceptable container.
### Go Container
Generally it will be the same process as creating the Node Container.  
But, we will use multi stage container in this Go example. Because go app will produce an executable we can first build the executable and run it in the most lightweight base image possible.
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

In this example we use bookworm (Debian) instead of alpine because it doesn't matter as this is only the first stage of the container which mean it will not make it into the final container

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
On the second stage we just copied the executable and assets from the first stage and run it. But, we use scratch as a base so it'll be super lightweight. Do keep in mind that scratch base has nothing in it.

## Docker Compose
If our container needs two or more app to be run at the same time to works, we'll need Docker Compose.  
Compose essentially string together Dockerfile to run at the same time with configuration.  

You need to install it first, use our package manager.
```bash
sudo pacman -S docker-compose
```

Then on the app directory create a compose file called `docker-compose.yml`  
we'll use our previous app as an example, because the app requires postgresql we will add postgresql using docker compose

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
First we pull the official postgresql image with alpine tag and specific version, give the container name, and set the port. environment is the environment variables and volumes is used to persist the data because docker containers are ephemeral by default.  

And for the go app because we already have Dockerfile we use build instead of pulling a new image, depends_on mean that the container will not start if postgresdb is not there and we are done.

The command to run docker compose is
```bash
docker compose up
```
