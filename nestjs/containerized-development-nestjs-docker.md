# Containerized development with NestJS and Docker

### Adding Docker with multi-stage build
Containerizing our applications with Docker has many advantages. For us, the two most important are that the application will behave as expected regardless of the environment, and that it is possible to install all the external dependencies (in our case, Redis and PostgresSQL) automatically when starting the application.

Also, Docker images are easily deployable on platforms such as Heroku and work well with CI solutions like CircleCI.

We are going to use a recently added feature called multi-stage build. It helps us keep the built production image as small as possible by keeping all the development dependencies in the intermediate layer, which may, in turn, result in faster deployments.

With that said, at the root of our application, let’s create a Dockerfile that makes use of the multi-stage build feature:

```yml
FROM node:12.13-alpine As development

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --only=development

COPY . .

RUN npm run build

FROM node:12.13-alpine as production

ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --only=production

COPY . .

COPY --from=development /usr/src/app/dist ./dist

CMD ["node", "dist/main"]
```

### Adding docker-compose
```yml
version: '3.7'

services:
    main: 
        container_name: main
        build: 
            context: .
            target: development
        volumns:
            - .:/usr/src/app
            - /usr/src/app/node_modules
        ports: 
            - ${SERVER_PORT}:${SERVER_PORT}
            - 9229:9229
        command: npm run start:dev
        env_file:
            - .env
        networks:
            - webnet
        depends_on:
            - redis
            - postgres
    redis:
        container_name: redis
        image: redis:5
        networks: 
            - webnet
    postgres:
        container_name: postgres
        image: postgres:12
        networks: 
            - webnet
        environment:
            POSTGRES_PASSWORD: ${DB_PASSWORD}
            POSTGRES_USER: ${DB_USERNAME}
            POSTGRES_DB: ${DB_DATABASE_NAME}
            PG_DATA: /var/lib/postgresql/data
        ports: 
            - 5432:5432
        volumes:
            - pgdata:/var/lib/postgresql/data
networks:
    webnet:
volumes:
    pgdata:
```