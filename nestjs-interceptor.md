NestJS interceptors are class-annotated with injectable decorators and implement the `NestInterceptor` interface. This interface has 2 methods: `intercept` and `handleRequest`. The `interceptor` method is called before sending the request to a controller, while the `handleRequest` method is called after the request has been processed by the controller and a response is returned.

## What are NestJS interceptors?
Interceptors are the most powerful form of the request-response pipeline. They have direct access to the request before hitting the route handler. We can mutate the response after it has passed through the route handler. 

In the interceptor, we can do any processes and modify the request before it's sent to the server. We can also set up the interceptor to intercept the response before being sent back to the client.

## Exploring the `intercept` method, `ExecutionContext`, and `CallHandler`
The intercept method is a method that implements a custom interceptor. It takes in 2 arguments, namely: `ExecutionContext` and `CallHandler`. The `ExecutionContext` is an object that provides methods to access the route handler and class that can be called or invoked. The `CallHandler` is an interface that provides access to an `Observable`, which represents the response stream from the route handler.

## Creating a custom interceptor
Create a file in the src folder called `custom.interceptors.ts`

The interceptor will be created using a class that will implement a `NestInterceptor`. Then, the intercept method will be implemented. This method will take the 2 params mentioned earlier (`ExecutionContext` and `Handler`)

```typescript
import { CallHandler, ExecutionContext, NestInterceptor } from '@nestjs/common';
import { map, Observable } from 'rxjs';
import { User } from 'src/app.service';

export class CustomInterceptors implements NestInterceptor {
    intercept(context: ExecutionContext, hander: CallHandler): Observable<any> {
        console.log("Before...");
        return handler.handle().pipe(
            map((data) => data.map((item: User) => {
                console.log("After...")
                const res = {
                    ...item,
                    firstName: item.first_name,
                    lastName: item.last_name,
                }
                delete res.first_name, delete res.last_name;
                return res;
            }))
        )
    }
}
```

Here, we are implementing a custom interceptor that intercepts the response sent back to a client when it makes a request to the endpoint at http://localhost:3000/. We want to modify the respone to have a CamelCase name instead of the first_name and last_name. 

## Logging texts
In the code above, we logged a `"Before..."` text when the interception occurs when the client makes the request before reaching API endpoint/server. Then, we called the `handler.handle` method to trigger the execution of the controller. We also used the `Interceptors` on the `pipe` method. This processes any additional modifications before the response is returned to the client. Inside the `handler.handle` method, we used the `map` operator to transform the returned data.

We also mapped over the array of data and logged an `"After..."` text. This is where the interceptor occurs when sending back the data to the client. From there, we took the item (each `User` object) from the `map` method and stored it in a variable `res`. This is where we spread the values in the object and add the CamelCase transformation we want to achieve for first_name and last_name.

Then, we used the `delete` operator to remove the initial first_name and last_name values from the transformed object. Lastly, we needed to bind this custom interceptor to the controller that returns this data for our response to be properly transformed. We do the binding in the `app.controller.ts` as follows:

```typescript
@Get()
@UseInterceptors(CustomInterceptors)
getUsers(): User[] {
    return this.appService.getUsers();
}
```

## Binding interceptors in NestJS
As we saw in the previous section, to set up our custom interceptor, we used the `@UseInterceptors()` decorator that applies the interceptor to that specific route (`getUsers`). So, if we have another route with getReports, the interceptor will not be applied because we only specified it to apply to a particular route. Like NestJS guards and pipes, interceptors can also be global-scoped, controller-scoped, or method-scoped.

To implement the interceptor to apply to each route handler defined in a controller (controller-scoped), we will define it above the controller itself, as follows:

```typescript
@Controller()
@UseInterceptors(CustomInterceptors)
export class AppController {
 constructor(private readonly appService: AppService) {}

 @Get()
 getUsers(): User[] {
   return this.appService.getUsers();
 }

@Get('/reports')
 getReports(): Reports[] {
   return this.appService.getReports();
 }
}
```

Now, when we make a request to either the `"/"` route or the `"/reports"`, the interceptor will always transform the response to the specified logic in the interceptor.

To set up a global interceptor, we will use the `useGlobalInterceptors` method in the main.ts file, as shown below:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
 const app = await NestFactory.create(AppModule);
 // global scoped interceptor
 app.useGlobalInterceptors(new CustomInterceptors());
 await app.listen(3000);
}
bootstrap();
```

With this, the `CustomInterceptor` will be applied across the entire application for all the controllers and router handlers. However, if we register our `CustomInterceptor` globally, we will not be able to inject any dependencies which are defined within a modular scope. To solve this, we can register our interceptor within a scoped module, as shown below:

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { CustomInterceptors } from './interceptors/custom.interceptor';

@Module({
 imports: [],
 controllers: [AppController],
 providers: [
   AppService,
   {
     provide: APP_INTERCEPTOR,
     useClass: CustomInterceptors,
   },
 ],
})
export class AppModule {}
```

## NestJS interceptors use cases
### Logging
NestJS interceptors can be used for logging. Developers often need to track their Nest application's request and responses. This is useful for debugging purposes and monitoring the performance of the application. 

### Data validation

### Authentication and authorization

### Exception mapping
Another use case of NestJS interceptors is exception mapping. This is basically overriding the predefined exceptions using the RxJS operator called `catchError()`. 

```typescript
import {
 BadRequestException,
 CallHandler,
 ExecutionContext,
 NestInterceptor,
} from '@nestjs/common';
import { catchError, throwError } from 'rxjs';

export class ExceptionInterceptor implements NestInterceptor {
 intercept(context: ExecutionContext, handler: CallHandler): any {
   return handler
     .handle()
     .pipe(catchError((err) => throwError(() => new BadRequestException())));
 }
}
```

```typescript
@Get('/exception')
@UseInterceptors(ExceptionInterceptor)
getUsersException() {
    throw new UnprocessableEntityException();
}
```