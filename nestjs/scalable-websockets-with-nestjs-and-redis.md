# Scalable WebSockets with NestJS and Redis

### Adding Redis
Redis is an in-memory data structure store that can be used as a database, cache, or publish/subscribe client.

To communicate with Redis from Node runtime, there are a few libraries available. We are going to use ioredis due to the great number of features it provides while maintaining robust performance.

We have to create a Nest module to encapsulate the code related to Redis. Inside the RedisModule, we have the providers array, in which we create the ioredis clients to communicate with Redis. We also implement RedisService, which abstracts away both listening on and sending Redis messages.

We create two Redis clients with different purposes: one for subscribing and one for publishing messages.
```ts
// redis.provider.ts

import { Provider } from '@nestjs/common';
import Redis from 'ioredis';

import { REDIS_PUBLISHER_CLIENT, REDIS_SUBSCRIBER_CLIENT } from './redis.constants';

export type RedisClient = Redis.Redis;

export const redisProviders: Provider[] = [
    {
        useFactory: (): RedisClient => {
            return new Redis({
                host: 'socket-redis',
                port: 6379
            })
        },
        provide: REDIS_SUBSCRIBER_CLIENT,
    },
    {
        useFactory: (): RedisClient => {
            return new Redis({
                host: 'socket-redis',
                port: 6379
            })
        },
        provide: REDIS_PUBLISHER_CLIENT
    }
]
```

With those providers registered in the RedisModule, we are able to inject them as dependencies in our service.
```ts
// redis.service.ts
import { REDIS_PUBLISHER_CLIENT, REDIS_SUBSCRIBER_CLIENT } from './redis.constants';
import { RedisClient } from './redis.providers';

export interface RedisSubscribeMessage {
    readonly message: string;
    readonly channel: string;
}

@Injectable()
export class RedisService {
    public constructor(
        @Inject(REDIS_SUBSCRIBER_CLIENT)
        private readonly redisSubscriberClient: RedisClient,
        @Inject(REDIS_PUBLISHER_CLIENT)
        private readonly redisPublisherClient: RedisClient;
    ) {}

    // ...
}
```

Then we define 2 methods: `fromEvent` and `publish`.
```ts
public fromEvent<T>(eventName: string): Observable<T> {
    this.redisSubscriberClient.subscribe(eventName);

    return Observable.create((observer: Observer<RedisSubscribeMessage>) => {
        this.redisSubscriberClient.on('message', (channel, message) => observer.next({ channel, message })),
    }).pipe(
        filter(({ channel }) => channel === eventName ),
        map(({message}) => JSON.parse(message)),
    )
}
```

It tells Redis to keep an eye out for the provided event by using the subscribe method of the redisSubscriberClient. Then we return an observable in which we are listening for any new messages by attaching a listener on the `message` event.

When we receive a new message, we first check whether the `channel` (Redis name for event) is equal to the provided `eventName`. If it is, we use JSON.parse to turn the Redis-sent string into an object.

```ts
public async publish(channel: string, value: unknown): Promise<number> {
    return new Promise<number>((resolve, reject) => {
        return this.redisPublisherClient.publish(channel, JSON.stringify(value), (error, reply) => {
            if(error) {
                return reject(error);
            }
            return resolve(reply);
        })
    })
}
```
The `publish` method takes a channel and an `unknown` value and uses the `redisPublisherClient` to publish it. We assume that the provided value can be stringified with JSON.stringify since Redis has no way of transporting Javascript objects.

With these 2 methods, we have successfully abstracted away all the troublesome code of connecting to the underlying Redis clients and can now use a reliable API to send events between instances by using the RedisService.

continuing: https://blog.logrocket.com/scalable-websockets-with-nestjs-and-redis/