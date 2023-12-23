# Serialization in NestJS: A different approach
Serialization is a process of preparing an object to be sent over the network to the end client. To prepare an object could be to exclude some of its sensitive or unnecessary properties or to add some additional ones.

NestJS provides a way to serialize objects returned from our API endpoints with the use of a little decorator magic and a library called class-transformer. The solution is good for very basic cases, but falls short in more complicated ones.

>Note: we must return an instance of the class. If you return a plain JavaScript object - for example, { user: new UserEntity() } - the object won't be properly serialized.

### Creating our own serialization mechanism
In order to create our own serialization, there are 2 things we have to implement:
1. We have to create a "parent" class that every serializer will extend. There, we'll put all the reusable methods, such as serialize or `serializeCollection`.
2. We have to create our own interceptor that will take care of actually running our serializers.

Serialization often includes checking user's roles to see what kind of properties of given object are they allowed to retrieve.

Aside from excluding the unwanted values from objects, our serialization will also provide other features, such as asynchronous serialization, nested serialization, and adding additional properties that weren't in the original object.

Here's the flow of our serialization.

Each controller marks which properties should be serialized like:
```ts
return {
    user: this.userSerializerService.markSerializableValue(user),
    otherProperty: true,
}
```

Then, the interceptor goes over the keys of the returned object and serizalizes the values that were marked. To "mark" an object, we simply wrap it into a class called `Serializable`. This way, from inside the interceptor, we can easily check whether a value of the property is an instance of the `Serializable` class.
```ts
export class Serializable<T> {
    public constructor(
        public readonly serialize: () => Promise<T | T[]>
    ) {}
}
```

The class doesn't do anything itself aside from keeping a reference to a function that will be use to serialize a value. The function will be provided by the serializer.

So the final shape of the object above would be:
```ts
return {
    user: Serializable<User>,
    otherProperty: true
}
```

Create the base serializer: `BaseSerializerService`

We are going to create an abstract class called BaseSerializerService that will provide all the reusable methods for all the serializers.
```ts
export abstract class BaseSerializerService<E, T> {
    // ...
}
```

The class takes two generic types, E and T, which stand for an entity and a serialized value, respectively.

### Serialization methods
```ts
public abstract async serialize(entity: E, role: UserRole): Promise<T>;

private serializeCollection(values: E[], role: UserRole): Promise<T[]> {
    return Promise.all<T>(values.map((v) => this.serialize(v, role)));
}
```

Each serializer will implement its own `serialize` method; thus, the method is `abstract` and has no implementation. The `serialize` method takes an entity and a user role. Then, taking into account the user role, it serializes the entity. Afterwards, the serialize object is ready to be sent to the end client.

The second method is called `serializeCollection`, which takes an array of entities and returns an array of serialized objects.

Admittedly, we could have used a single method called serialize and checked ourselves whether the provided value is an array, but it is better to keep the API as unambiguous as possible.

### Wrapping values
In order to mark the returned value as serializable so the interceptor could serialize it later, we provide two methods:
```ts
public markSerializableValue(value: E): Serializable<T> {
    return new Serializable<T>(this.serialize.bind(this, value));
}

public markSerializableCollection(values: E[]): Serializable<T[]> {
    return new Serializable<T[]>(this.serializeCollection.bind(this, values));
}
```

Both functions accept one parameter: in the first case, it is an entity, and in the second, a collection of entities.

With the serializers methods in place, we simply pass them to the `Serializable` class so they can be called later by the interceptor. Keep in mind that the serialization doesn't happen until the interceptor calls the provided functions.

Once again, the Serializable class doesn't do anything except for keeping a reference to the provided function so that it can be used later inside the interceptor.

### `SerializerInterceptor`
Interceptors in Nest are called before and after a request is handled, providing us with the opportunity to transform the object returned from a controller method.
```ts
export interface AuthenticatedRequest extends Request {
    readonly user: User;
}

@Injectable()
export class SerializerInterceptor implements NestInterceptor {
    private async serializeResponse(
        response: Response,
        role: UserRole
    ): Promise<Record<string, any>> {
        const serializedProperties = await Promise.all(
            Object.keys(response).map(async (key) => {
                const value = response[key];

                if(!(value instanceof Serializable)) {
                    return {
                        key,
                        value
                    }
                }

                const serializedValue = await value.serialize(role);

                return {
                    key,
                    value: serializedValue,
                };
            }),
        ),

        return serializedProperties.reduce((result, { key, value }) => {
            result[key] = value;
            return result;
        }, {})
    }

    public intercept(
        context: ExecutionContext,
        next: CallHandler
    ): Observable<any> {
        const request = context.switchToHttp().getRequest<AuthenticatedRequest>();

        return next.handle().pipe(
            switchMap((response) => {
                if(typeof response !== 'object' || response === null) {
                    return of(response);
                }

                return from(this.serializeResponse(response, request.user?.role));
            }),
        );
    }
}
```

The pubic method `intercept` is required by Nest, and it is called before each request. It has two parameters: context and next.

Thanks to the `context` object, we can easily get access to the underlying `http` request.

We are going to assume that there are some guards or middleware that set the authenticated user object in the request.user property.

Having access to the `user` object, we can easily get the role of the authenticated user. Just to be safe, we are using the optional chaining operator ? introduced recently in TypeScript in case the user object has not been set.

The next object has a .handle() method that resumes the processing of the request. If we decided that a request wasn’t supposed to be handled, we could have ended the execution early and returned an empty observable instead of calling next.handle().

The next.handle() method returns an observable that at one point in time will emit the response. We are using RxJS' `switchMap` operator, which ensures that only one response is returned. There are cases in which this would not be the expected behavior - for example, if the interceptor was used with WebSockets.

Inside the function we provided to the switchMap operator, we check whether the response is even an object in the first place, because if it isn't, then there's nothing to serialize. Note that instead of returning the `response` itself, we have to wrap it in an observable using the `of` function since switchMap expects us to return an observable.

If the `response` is indeed an object, we are going to use the `serializeResponse` method. Since we support asynchronous serialization, we are wrapping the returned promise in a `from` function to create an observable from the promise.

Knowing that the response provided as an argument is an object, we can safely use the Object.keys method to iterate over the object's keys.

The method can be split into 2 parts: serializing the properties and forming the response object.

First, we map over the keys and check whether their respective value is an instance of `Serializable`. If it is, we execute the `serialize` and await its result, returning it as the new value. Otherwise, we just return the existing value.

We have to wrap the mapping in the Promise.all method to make sure every promise is resolved before continuing. As a result, after the process takes place, we are left with an array of objects with the following shape: `{ key, value }`. Thanks to the usage of Promise.all, we are able to run the serializations of many properties simultaneously.

Next, we reduce the array of objects and values into an object, returning the object with the exact shape like the original one, but with all the properties serialized.

## Real-world use case
```ts
@Entity(USER_TABLE_NAME)
export class User {
    @PrimaryGeneratedColumn('uuid')
    public id: string;

    @Column('text', { unique: true })
    public email: string;

    @Column('text')
    public password: string;

    @Column({ type: 'enum', enum: UserRole })
    public role: UserRole;

    @OneToMany(
        () => Article,
        (article) => article.author,
    )
    public articles: Article[];
}
```

Our goal in serialization would be to make sure that the `password` property is removed and nested articles are also serialized. In order to keep the code clean and reusable, it would be best to use the `articleSerializerService` to serialize an article instead of writing the same logic in the `userSerializerService`.
```ts
@Injectable()
export class UserSerializatorService extends BaseSerializerService<User, SerializedUserDTO> {
    public constructor(
        private readonly articleSerializatorService: ArticleSerializatorService
    ) {
        super();
    }

    public async serialize(
        entity: User,
        role: UserRole,
    ): Promise<SerializedUserDTO> {
        const strippedEntity = _.omit(entity, ['password']);
        const articles = await this.articleSerializatorService.serializeCollectionForRole(entity.charters, role);

        return {
            ...strippedEntity,
            articles
        }
    }
}
```
```ts
@UseInterceptors(SerializerInterceptor)
@Controller(USER_ENDPOINT)
export class UserController {
    public constructor(
        private readonly userSerializatorService: UserSerializatorService, 
    ) {}

    @Get(USER_ID_ROUTE)
    public async get(
        @Param(USER_ID_PARAM) userId: string
    ): Promise<GetUserResDTO> {
        const user = await this.userService.findOne({ userId })

        return {
            user: this.userSerializatorService.markSerializableValue(user)
        }
    }
}
```

We also have access to the requesting user's role (passed from the interceptor), so we can strip some properties based on that.

Due to how the serializers are structured, the articleSerializer could also have some nested properties. Each serializer makes sure that the entity from its domain is correctly serialized and delegates serialization of other entities to their respective serializers.

