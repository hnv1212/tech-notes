Backend developers often apply some common tasks to the requests that our service receives.

Some of these tasks are applied before fulfilling the request, like authentication and authorization. Other are applied after the request is processes, but just before the response is sent, such as log of the resource accessed.

Instead of adding this logic to our controllers, we can add these tasks to middleware and apply them to all routes or multiple specific routes.

### What is middleware in NestJS?
Middleware is a function that is called before a route handler, also called a controller. It can also be configured to be called after the route handler executes the logic, but before the response is sent. These functions get access to the request and response objects and perform actions on them.

In NestJS, middleware can: 
- Perform any operations
- Access and make changes to the request and response objects
- Call the next middleware function in the stack
- End the request-response cycle

Note that you must call the `next()` middleware function to pass control to the next middleware function if it does not end the request-response cycle. Otherwise, the request will be left hanging.

### Creating NestJS middleware
NestJS middleware can be written as either a function or a class that uses the `@Injectable()` decorator and implements the `NestMiddleware` interface. 

@Injectable() tells Nest to instantiate that class and inject the instance everywhere it is a dependency. This means if you define a class that needs an instance of your `@Injectable()` class, you will not have to create and assign the instance yourself.

<u>Creating a class middleware</u>

we will create our class file `simple-logger.middleware.ts` and fill it in like so:

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class SimpleLoggerMiddleware implements NestMiddleware {
    use (req: Request, res: Response, next: NextFunction) {
        // do some tasks
        console.log('executing request...')
        next();
    }
}
```

The most important feature of our class is the `use` method, which is define and required by the `NestMiddleware`. Any logic we would like our middleware to execute - before or after - in our request/response cycle must be written here.

<u>Create a function middleware</u>

let's create a new file `simple-func.middleware.ts` in which we will create and export a function that can be applied as a middleware:

```typescript
import { Request, Response, NextFunction } from 'express';

export function simpleFunc(req: Request, res: Response, next: NextFunction) {
  // In here do some stuff :p
  console.log('Executing request after the function middleware...');
  next();
}
```

You'll notice that our function above is pretty similar to the `use` method of our `SimpleLoggerMiddleware` class; it takes Request, Response, and NextFunction instances as arguments.

### Applying our NestJS middleware in a module class
Our middleware can be applied in our module class - not within the @Module() decorator where we typically register dependencies of a module, like imports, controllers, services, and others.

When it comes to middleware, we instead apply the consumer in the module class - in this example, the `TreeModule` class - using the `configure()` method made available by implementing the NestModule interface.

```typescript
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { TreeController } from './tree.controller';
import { Tree } from './tree.entity';
import { TreeService } from './tree.service';
import { SimpleLoggerMiddleware } from '../middleware';

@Module({
    imports: [TypeOrmModule.forFeature([Tree])],
    providers: [TreeService],
    controllers: [TreeController],
})
export class TreeModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        consumer.apply(SimpleLoggerMiddleware).forRoutes('trees');
    }
}
```

In our snippet above, in our module class TreeModule, the `configure()` method takes a middleware consumer. This consumer applies our middleware using the `apply()` method.

Then, we go a step further to tell the consumer to only apply the middleware to specific routes using the `forRoutes()` method. With this code, we have applied our SimpleLoggerMiddleware on all /trees route requests.

We can also make our applied middleware relavant to more specific routes. Using `forRoutes`, we can pass the resource and resource method that we want to be intercepted by the middleware.

Here is an example of using forRoutes to apply the middleware only to POST method requests on the `/trees` resource:

```typescript
.forRoutes({ path: 'trees', method: RequestMethod.POST })
```

### Applying middleware for all routes in a module class
Let's apply our function middlware `simpleFunc` to all routes and register it in the AppModule class:

```typescript
// app.module.ts

import { TypeOrmModule } from '@nestjs/typeorm';
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { TreeModule } from './tree/tree.module';
import { Tree } from './tree/tree.entity';
import { simpleFunc } from './middleware';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'xxxx',
      database: 'fruit-tree',
      entities: [Tree],
      synchronize: true,
    }),
    TreeModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(simpleFunc).forRoutes('*');
  }
}
```

If we decide to exclude some specific routes - for example, a health check route like `/healthcheck` - we can chain our `consumer.apply(...)` with the method `exclude()` like so:

```typescript
// app.module.ts

export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(simpleFunc).exclude('healthcheck').forRoutes('*');
  }
}
```

The consumer methods apply(), exclude(), and forRoutes also accept a list of comma-separated values to allow us to register multiple middlewares and routes respectively.

### Applying middleware globally in our application
We've looked at applying our middleware in module classes, but we can also apply a middleware - or several middleware - to all routes using the `use()` method of our Nest application's INestApplication instance.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SimpleLoggerMiddleware } from './middleware';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(SimpleLoggerMiddleware);
  await app.listen(3000);
}

bootstrap();
```

In the above snippet, we have registered the middleware on the Nest application instance, making it applicable to every route.

### Common middleware used NestJS projects
<u>Authentication and authorization</u>

<u>CORS</u>

wherever we would like to apply the CORS middleware — in this case, let’s use it our AppModule class — we will import and apply it to the middleware consumer:

```typescript
// app.module.ts

import cors from 'cors';

...

export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(simpleFunc, cors()).forRoutes('*');
  }
}
```

CORS does not necessarily have to be applied as a middleware, but since it can be, we can think of it as one here.

<u>Helmet</u>
Helmet is a Node.js library that helps set some important HTTP headers by default. This headers can help minimize some well-known security vulnerabilities. 

```typescript
// app.module.ts

import helmet from 'helmet';

...

export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(simpleFunc, cors(), helmet()).forRoutes('*');
  }
}
```

### Benefits of middleware in NestJS
Using middleware in NestJS application is a good way to separate the core functionality of a resource from chores that need to be performed before or after the request is processed.

Middleware also provides a good way to reapply this chore logic where needed by beingn able to configure it for different resources - possibly in different modules - without repeating the logic.

### Drawbacks of using middleware

<u>Middleware can slow down your application server</u>

This can happen when middleware function executions are not terminated correctly using the next() function. By not calling the next() function, our requests can remain stuckin the middleware, causing the server to be overloaded and ultimately slower.

<u>Middleware can be a source of bugs and unintended behaviors

When written correctly, middleware functions should be written with single jobs so it is clear what applying them to a resource or route does.

But if written poorly - for example, by combining multiple actions in one middleware function - we might alter the expected request or response for the application resulting in unintended behaviour.

<u>Middleware can be a security risk

Since middleware have the ability to alter request and response objects, there are opportunities to alter these objects to expose or delete relevant data. This can pose an additional risk to users making requests to the application.