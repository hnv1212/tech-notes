# How to add Redis cache to a NestJS app

## What is caching?
In computing, a cache is frequently queried, temporary store of duplicate data. The data is kept in an easily accessed location to reduce latency.

## What is Redis?
Redis is an open source, in-memory data structure store used as a database, cache, message broker, and streaming engine.

Redis doesn't handle caching alone. It provides data structures like hashes, sets, strings, lists, bitmaps, and sorted sets with range queries, streams, HyperLogLogs, and geospatial indexes.

## Implementing Redis cache in a NestJS app
Installing packages:
- `node-cache-manager`: allows for easy wrapping of functions in cache, tiered caches, and a consistent interface.
- `@types/cache-manager`: the typescript implementation of node-cache-manager.
- `cache-manager-redis-store`: provides a very easy wrapper for passing configuration to the `node_redis` package. 
- `@types/cache-manager-redis-store`: the TypeScript implementation of the cache-manager-redis-store package

```typescript
import { Module, CacheModule } from '@nestjs/common';
import * as redisStore from 'cache-manager-redis-store';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
    imports: [CacheModule.register({ 
        store: redisStore,
        host: 'localhost',
        port: 6379
    })],
    controllers: [AppController],
    providers: [AppService],
})
export class AppModule {}
```

```typescript
import { Controller, Get, Inject, CACHE_MANAGER } from '@nestjs/common';
import Cache from 'cache-manager';
import { AppService } from './app.service';

@Controller()
export class AppController {
    randomNum = Math.floor(Math.random() * 10)

    constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}

    @Get('get-number-cache')
    async getNumber(): Promise<any> {
        const val = await this.cacheManager.get('number')
        if(val) {
            return {
                data: val,
            }
        } else {
            await this.cacheManager.set('number', this.randomNum, { ttl: 1000 })
            return {
                data: this.randomNum
            }
        }
    }
}
```

## Setting up automatic caching using Interceptor
Auto cache enables cache for every Get action method inside of the controller using the CacheInterceptor.

```typescript
import { Module, CacheModule, CacheInterceptor } from '@nestjs/common';
// ...
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
    // ...
    providers: [
        {
            provide: APP_INTERCEPTOR,
            useClass: CacheInterceptor
        }
    ]
})
export class AppModule {}
```

```typescript
import {Controller, Get, Inject, CACHE_MANAGER, UseInterceptors, CacheInterceptor} from '@nestjs/common';
// ...

@UseInterceptors(CacheInterceptor)
@Controller()
export class AppController {
    // ...

    @Get('auto-caching')
    @CacheKey('auto-caching-fake-model')
    @CacheTTL(10)
    getAutoCaching() {
        // ...
    }
}
```