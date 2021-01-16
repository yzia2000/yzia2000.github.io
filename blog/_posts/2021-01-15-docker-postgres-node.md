---
layout: post
title:  "Docker with Typescript Node and Postgres"
date:   2021-01-15 9:43:48 +0530
categories: Blog
---

I recently decided to bring docker to my development and deployment environment
primarily because I was tired of my projects failing due to dependency hell,
especially when working with other students.

I will not be going into too many technical details behind docker but simply be
discussing my web application configurations for development and deployment.

The tech-stack that I am working with contains Postgres and Node with compiled
typescript and it satisfies the following rules.

1. Live reload for development.
2. Building for low resource deployments.
3. Loosely coupled configurations.
4. Single run script for running each server.

I think this template is a perfect combination of convenience and security.

## Template repository

I have a template repository for web development on [GitHub](https://github.com/yzia2000/webdev-template)
that I will reuse to instantiate projects. I am working on a separate repo
for a microservices architecture. The file structure of the template is as follows.

```
webdev-template
├── backend
│   ├── dist
│   ├── docker-compose.override.yml % For development
│   ├── docker-compose.yml % For deployment
│   ├── Dockerfile % Node typescript
│   ├── package.json % node modules
│   ├── .env.local % Postgres database environment variables for local development
│   ├── .env.prod % Postgres database environment variables for deployment
│   ├── README.md
│   ├── src
│   │   ├── config.ts
│   │   └── index.ts
│   └── tsconfig.json
├── CONTRIBUTING.md
├── frontend
│   ├──...
└── README.md
```

There are separate root folders for the `backend` and `frontend` both containing
their individual `README.md` files. `package.json` contains the node packages
I use for development and production.

But more importantly, we have the npm scripts which will be used inside the
docker files.

```json
"scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon --watch 'src/**' --ext 'ts,json' --ignore 'src/**/*.spec.ts' --exec 'ts-node src/index.ts'",
    "build": "tsc -p ."
}
```

For development live-reloads we have the dev script that uses `nodemon` and `ts-node`
while for production we have the build script and the start script.

The node software configurations are isolated to the `package.json` while Postgres
configurations are handled by docker-compose. One point to note is that I am
using a `.env.local` file to store Postgres environment variables that can be
accessed by the development server and a `.env.prod` file ignored by git for
deployment to prevent leaking my deployment configurations and causing a security
hazard.

The .env files look like this:
```
# Example:
POSTGRES_PASSWORD=postgres
POSTGRES_USER=postgres
POSTGRES_DB=database
```

Next lets discuss the docker configs.

## Dockerfile: Node Multi-stage builds

I use docker [multi-stage
builds](https://docs.docker.com/develop/develop-images/multistage-build/) to
assist with not only development, deployment but also keep my docker containers
lean. Moreover I use alpine images of node and postgres. I know there are
leaner images out there but I do not want to compromise on safety.

I have 3 build stages: `base`, `builder` and `prod`.
1. `base` - npm packages get installed and cached. Can use this stage for development by attaching volumes and running the npm dev script in docker-compose.
2. `builder` - Picks up after base, compiles ts files to js files and outputs in `dist/` dir.
3. `prod` - Copies `node_modules/` and `dist` into a different node-alpine image and runs the deployment server. Lowers deployment size.

The Dockerfile is as follows:
```docker
FROM node:10-alpine AS base
WORKDIR /app
COPY package*.json ./
COPY tsconfig.json .
RUN npm install

FROM base AS builder 
COPY ./src src
RUN npm run build

FROM node:10-alpine AS prod
WORKDIR /app
COPY --from=builder /app/dist dist
COPY --from=builder /app/node_modules node_modules
COPY --from=builder /app/package.json package.json
CMD ["npm", "run", "start"]
```
Notice that we provide a version here. This not only ensures that all developers will use
the same version for development but also that things will not unnecessary break due to
updates on deployment.

## Docker Compose: Dev and Prod

This utility handles multiple containers and is extremely crucial for any
docker setup. Docker Compose fortunately allows overriding `docker-compose.yml`
with `docker-compose.override.yml` as well as the differences between the two.
Since, we will be running the local dev server more often, the override file
contains our dev server configs while the `docker-compose.yml` file contains are
deployment configs. With this we can use a simple command for running the dev
server:
```sh
docker-compose up --build
```
and a simple command for running the deployment server:
```sh
docker-compose up -f docker-compose.yml --build
```

Deployment configurations are quite straightforward. As you can see we
are using `.env.prod` here for the database environment variables and docker
volumes for db data persistence. Since we do not provide a target for the node
container, it assumes it will be the last stage, prod.

Lastly, we use `sql` files inside `dbscript/` to instantiate our database schema and relations. 

```yml
version: "3.7"
services:
    db:
        image: postgres:13-alpine
        container_name: db
        env_file:
            - .env.prod
        ports: 
            - '5432:5432'
        volumes:
            - db:/var/lib/postgresql/data
    api:
        image: yzia2000/project-api
        build:
            context: .
        container_name: project-api
        env_file:
            - .env.prod
        depends_on:
            - db
        ports:
            - '5000:5000'
volumes:
    db:
```
However, for `docker-compose.override.yml` we are making some changes.

```yml
version: "3.7"
services:
    db:
        env_file:
            - .env.local
    api:
        image: yzia2000/project-api:dev
        env_file:
            - .env.local
        build:
            context: .
            target: base
        container_name: project-api-dev
        command: npm run dev
        volumes:
            - ./src:/app/src
            - ./package.json:/app/package.json
volumes:
    db:
```

Here we stop the docker node build after the base stage, use docker volumes to
mount our source files and running the dev script from `pacakage.json`. This
allows live reload of source files and is extremely convenient for development.

There are some other perks of using this [template repo](https://github.com/yzia2000/webdev-template) and I request you guys
to try it out.

## Conclusion
By using docker we can not only successfully eliminate dependency hell, have
useful environments for software projects but also integrate our projects
with CI/CD pipelines. As a matter of fact we can deploy these containers very
easily on demand using AWS EC2, GCP or Heroku. Moreover using tools like
[Dokku](http://dokku.viewdocs.io/dokku/), you can setup a cool CD pipeline
on a VPS infrastructure. I definitely plan on integrating docker with GitHub Actions
for CI/CD. This will definitely help in achieving the dream of having loose coupled
yet highly integrated environments for Software Engineering.

If you have any queries or feedback on my configurations, do post it on the [forum](https://github.com/yzia2000/blog/discussions).
I look forward to hearing you feedback!
