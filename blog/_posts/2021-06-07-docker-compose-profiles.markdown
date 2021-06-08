---
layout: post
title:  "Docker compose profiles"
date:   2021-01-07 18:43:48 +0530
categories: Blog
---

It's been a while since my previous blog post, long enough for dockercon 2021
to suprise us with some exciting features. My review of some of the features is
on the way but for this post I would like to discuss one of my most anticipated compose
features, profiles.

If you remember my previous docker [blog post](https://yzia2000.github.io/blog/2021/01/15/docker-postgres-node.html), 
in order to modularize container services, I decided to use `docker-compose.yml` overriding to start different
services for different environments. You can finally add services to one or multiple profiles and then
start one or multiple profiles.

## Demo
Let's say we have three environments: 
1. `dev`: Hotreloading with typescript without building.
2. `test`: Running integration tests. 
3. `prod`: Running in the production environment in a lightweight container. 

The setup used here is the same as in the previous [post](https://yzia2000.github.io/blog/2021/01/15/docker-postgres-node.html),
where we have a node app, postgres as database with docker volumes and a multistage `Dockerfile`.
We can now setup our new single `docker-compose.yml` with 3 profiles: `api-dev`, `api-test`, `api-prod`
as follows:
```yml
version: "3.9"
services:
    db:
        image: postgres:13-alpine
        environment:
            - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
            - POSTGRES_USER=${POSTGRES_USER}
            - POSTGRES_DB=${POSTGRES_DB}
        container_name: db-dev
        ports: 
            - '5432:5432'
        volumes:
            - dbdata:/var/lib/postgresql/data/
            - ./dbscripts:/docker-entrypoint-initdb.d

    api-dev:
        image: yzia2000/temp-api:dev
        environment:
            - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
        build:
            context: .
            target: base
        container_name: api-dev
        command: npm run dev
        volumes:
            - ./src:/app/src
            - ./package.json:/app/package.json
        depends_on:
            - db
        ports:
            - '5000:5000'
        profiles: ['api-dev']

    api-test:
        image: yzia2000/temp-api:test
        environment:
            - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
            - NODE_ENV=test
        build:
            context: .
            target: builder
        container_name: api-test
        command: ./wait-for-it.sh db:5432 -- /bin/sh -c "npm run test"
        depends_on:
            - db
        ports:
            - '5000:5000'
        profiles: ['api-test']

    api-prod:
        image: yzia2000/temp-api:latest
        environment:
            - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
            - NODE_ENV=production
        build:
            context: .
        container_name: api
        depends_on:
            - db
        ports:
            - '5000:5000'
        profiles: ['api-prod']

volumes:
    dbdata:
```

To run a profile, simply:
```sh
docker-compose --profile api-dev up
```

If you have some knowledge of `docker-compose.yml`, this will look self-explanatory. Profiles
are easy to understand too. You might find the lack of a profile attached to the db container
interesting. Containers that do not have a profile attached to them start automatically. Since, all
our profiles depend on the db container, we do not need to attach multiple profiles to it. 

Profiles can be especially useful when onboarding new members into your team as they no longer need
to worry about starting or stopping different containers and can get to work immediately. This is
one of the ways you can reduce developer cold start (new post coming soon).

To learn more about docker compose profiles you can read the docker [documentation](https://docs.docker.com/compose/profiles/).

## Conclusion
Now you longer need to maintain multiple `docker-compose.yml`, though I would recommend
doing that if you different teams on a monorepo setup (separated by feature and not environment).
Having a single `docker-compose.yml` means that you spend less time moving through different files
when changing things as simple as environment variables.

More docker features from the dockercon 2021 will be discussed soon in my upcoming blog posts
and I hope you look forward to them.
