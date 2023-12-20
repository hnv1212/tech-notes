# How to use NestJS with Prisma

### What is Prisma?
Prisma is a next-generation Node and TypeScript object-relational mapper (ORM). It provides an open source database toolkit for PostgresSQL, MySQL, SQL Server, SQLite, and MongDB, enabling developers to build apps faster and with fewer errors.

Prisma provides you with a declarative method for defining your app's data models and relations in a more legible format. Plus, if you already have a database, you don’t have to go through the pain of creating database models from scratch because Prisma’s introspection features handle that for you — it’s that flexible.

### What is Prisma used for?
Prisma improves type safety by simplifying database access, saving and reducing repetitive CRUD boilderplate. Prisma is easy to integrate into your preferred framework and is an ideal database toolkit for creating dependable and scalable web APIs. Prisma integrates quickly with various frameworks, such as GraphQL, nextjs, nest, apolo, and express.js.

Prisma addresses many shortcomings of tranditional ORMs, such as a lack of type safety, mixed business and storage logic, and unpredictable queries caused by lazy loading.

### Project setup
Create you intial Prisma setup using the Prisma `init` command:
```
npx prisma init
```
The above command creates a new Prisma directory with the following files:
- schema.prisma: specifies your database connection and contains the database schema
- .env: a dotenv file typically used to store your database credentials in a group of environment variables.

Open the datasource/schema.prisma: 
```prisma
generator client {
    provider = "prisma-client-js"
}

datasource db {
    provider = "sqlite"
    url = env("DATABASE_URL")
}

model Todo {
    id          Int      @id @default(autoincrement())
    title       String
    description String?
    completed   Boolean? @default(false)
    user        String
}
```

```env
<!-- .env -->
DATABASE_URL="file:./todos.sqlite"
```
Generate your SQL migration files and run them against the database with the command:
```
npx prisma migrate dev --name init
```
The above command will generate the folder structure below:
```
prisma
 ┣ migrations
 ┃ ┣ 20220315212227_init
 ┃ ┃ ┗ migration.sql
 ┃ ┗ migration_lock.toml
 ┣ schema.prisma
 ┣ todos.sqlite
 ┗ todos.sqlite-journal
```

### Setting up Prisma Client and Prisma Service
Prisma Client is a type-safe database client generated from your Prisma model definition. It exposes the CRUD operations tailored specifically to your models.
```
npm install @prisma/client
```

With Prisma Client setup, create a prisma.service file in the src folder to abstract away the Prisma Client API for database queries within a service:
```ts
import { INestApplication, Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable() 
export class PrismaService extends PrismaClient implements OnModuleInit {
    async onModuleInit() {
        await this.$connect();
    }

    async enableShutdownHooks(app: INestApplication) {
        this.$on('beforeExit', async () => {
            await app.close();
        })
    }
}
```

### Generating a todo module
```bash
nest generate module todo
```
```bash
nest generate service todo/service/todo --flat
```
```ts
// <!-- todo.service --> 
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../../prisma.service';
import { Todo, Prisma } from '@prisma/client';

@Injectable()
export class TodoService {
    constructor(private prisma: PrismaService) {}

    async getAllTodo(): Promise<Todo[]> {
        return this.prisma.todo.findMany();
    }

    async getTodo(id: number): Promise<Todo | null> {
        return this.prisma.todo.findUnique({ where: { id: Number(id)}});
    }

    async createTodo(data: Todo): Promise<Todo> {
        return this.prisma.todo.create({
            data,
        })
    } 

    async updateTodo(id: number): Promise<Todo> {
        return this.prisma.todo.update({
            where: {id: Number(id)},
            data: { completed: true }
        })
    }

    async deleteTodo(id: number): Promise<Todo> {
        return this.prisma.todo.delete({
            where: {id: Number(id)}
        })
    }
}
```
```
nest generate controller todo/controller/todo --flat
```
```ts
// <!-- todo.controller.ts -->
import {
  Controller,
  Get,
  Param,
  Post,
  Body,
  Put,
  Delete,
} from '@nestjs/common';
import { TodoService } from '../service/todo.service';
import { Todo } from '@prisma/client';

@Controller('api/v1/todo')
export class TodoController {
    constructor(private readonly todoService: TodoService) {}

    @Get()
    async getAllTodo(): Promise<Todo[]> {
        return this.todoService.getAllTodo();
    }

    @Post()
    async createTodo(@Body() postData: Todo): Promise<Todo> {
        return this.todoService.createTodo(postData)
    }

    @Get(':id')
    async getTodo(@Param('id') id: number): Promise<Todo | null> {
        return this.todoService.getTodo(id);
    }

    @Put(':id')
    async Update(@Param('id') id: number): Promise<Todo> {
        return this.todoService.updateTodo(id);
    }

    @Delete(':id')
    async Delete(@Param('id') id: number): Promise<Todo> {
        return this.todoService.deleteTodo(id);
    }
}
```
```ts
// <!-- todo.module.ts -->
import { PrismaService } from 'src/prisma.service';

@Module({
    controllers: [ ... ],
    providers: [..., PrismaService],
})
// ...
```