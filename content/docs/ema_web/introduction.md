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
lead = 'This is the technical documentation of Ema Web App. The doc aim to help onboards new developers'
toc = true
top = false
+++

## Quick Start
This section will help you run the app locally on your machine to start developing.

### Requirements

Here are the required software to run the app locally

- Node JS
- Node Package Manager
- Docker
- Supabase CLI

#### Node JS  
Install it using your package manager. As of the making of this document we're using v21.2.0. But, using latest should work in most cases.
```bash
sudo pacman -S nodejs
```
If you prefer another installation method you have the freedom.

#### Node Package Manager
Install it using your package manager. Use the latest version.
```bash
sudo pacman -S npm
```
If you prefer another installation method you have the freedom.

#### Docker Engine
What we need is the Docker Engine, install it using your package manager. Use the latest version.
```bash
sudo pacman -S docker
```

For *windows* just download the Docker Desktop it comes with docker engine. Use the latest version.

Also you might want to run docker as non root, you can do so by adding the current user to the docker group
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

You can reboot your pc to make sure that the group and daemon change is applied.  
To verify if docker is working and can be run as a non root user
```bash
docker run hello-world
```


#### Supabase CLI
If you're using arch you can use AUR to install it. Use the latest version.
```bash
yay -S supabase-bin
```

For mac you can use brew
```bash
brew install supabase/tap/supabase
```

If you use neither please read the official [Documentation](https://supabase.com/docs/guides/cli/getting-started)

### Cloning the repository
Make sure you have git installed. If you don't have access to the repo ask for invite first.  
After you have your access clone the repo.
```bash
git clone git@github.com:lontarlab/cluster-ema-web.git
```
Then install the required npm package
```bash
cd cluster-ema-web
npm install
```

### Setting up the local database

#### Run database and migration
We will set the local database using the supabase-cli so make sure you installed it.  
In your directory of choice run the command
```bash
supabase init
```
This will create a new directory called supabase, enter the directory and start the database
```bash
cd supabase
supabase start
```
If the operation is successful it will output something like this
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
Or you can verify it by looking at the running containers 
```bash
docker ps
```
Next you'll have to run the migration and the seeder.  

Download the schema and seeder [here](https://drive.google.com/drive/folders/10_8ibm83SqRNCWrDtalSUZQi9LMJ9I7g?usp=sharing)

We're going to manually do it using psql
```bash
psql 'postgresql://postgres:postgres@localhost:54322/postgres'
\i path/to/schema.sql
\i path/to/seeder.sql
```
This will populate the database with real data from October 2023

#### Setup auth
Because the seeder doesn't come with auth user we will have to make it ourselves.  

Go to the supabase studio [http://localhost:54323](http://localhost:54323/project/default/auth/users) and check for the users table to see the user available.  
  
<img src="../users.png" alt="Auth" style="max-width: 100%; margin-bottom: 12px"/>    

Then go auth page to create the user, enter the same email and password  

<img src="../auth.png" alt="Auth" style="max-width: 100%; margin-bottom: 12px"/>

### Run the project
Make sure that the value prod db is set to false in .env.development
```
NEXT_PUBLIC_USE_PRODUCTION_DB=false
```


All the setup is done, now you can run the web app and you are done.
```bash
npm run dev
```

## Convention
This section will list the agreed upon convention among the team.  

### Database
#### Naming 

- use camelCase for table name
- use plural words for table name
- use English for the key / field

#### Table 

- All table must have at least createdAt
- Never delete any record use isDeleted and deletedAt instead
- Create view if it's required

#### Performance 

- Create an index for tables which will get queried often
- Run a vacuum operation once in a while
- Avoid using JSONB if possible

#### Safety
- Never run a destructive query on production database before counting and making sure everything is correct
- If the production database goes unresponsive, restart the database as soon as possible
- update the schema file when adding or updating table / view

### Code

#### Naming

- use snake_case for files associated with next
- use PascalCase for files associated with libs
- use camelCase for variables

#### NextJS

- use normal function instead of arrow function for non lambda function
- use NPM for the package manager
- use client side data fetching
- use the page router instead of app router
- use sweet alert to show information to users
- use proper type instead of any
- validate data coming from database and going to database using joi schema
- use let instead of var
- use const whenever possible
- call database operation through nextjs api
- provide key for child component generated from a loop
- use form validation before sending any data
- use view for datatable data source
- make sure ui element have enough contrast and feedback 
