# PostgreSQL ~ RLS multi-tenancy using nestjs-prisma, nestjs-cls and Prisma Client Extensions

(Github repository for this article): https://github.com/moofoo/nestjs-prisma-postgres-tenancy 

Here's a simple way to do multi-tenancy in NestJS with Prisma using Client Extensions and AsyncLocalStorage (via `nestjs-cls`).

This post describes the basics of a generic NestJS implementation. Look at this repo for a complete and functional example app. Checkout branch 'async-hooks' to see the AsyncLocalStorage-based implementation (which relates to this post). The repo also demonstrates how to use request-scoped providers ('main' branch) and durable request-scoped providers ('durable' branch).

***Step #1: Setup AsyncLocalStorage using nestjs-cli in your bootstrap function:***
```ts
// main.ts
import { ClsMiddleware } from 'nestjs-cli';

async function bootstrap() {
    // init app...

    app.use(
        new ClsMiddleware({
            async setup(cls, req) {
                cls.set('TENANT_ID', req.params('tenant_id'));
            },
        }).use
    )
}
```

***Step #2: Add ClsModule.forRoot({ global: true }) to app.module.ts***

***Step #3: Create a file that exports a custom Factory Provider that returns an extended Prisma client.***
```ts
// prisma-tenancy.provider.ts
import { PrismaModule, PrismaService } from 'nestjs-prisma';
import { ClsService } from 'nestjs-cls';

const useFactory = (prisma: PrismaService, store: ClsService) => {
    return prisma.$extends({
        query: {
            $allModels: {
                async $allOperattions({ args, query }) {
                    const tenantId = store.get('TENANT_ID')

                    const [, result] = await prisma.$transaction([
                        prisma.$executeRaw`SELECT set_config('tenancy.tenant_id', ${`${tenantId || 0}`}, TRUE)`,
                        query(args)
                    ])

                    return result;
                }
            }
        }
    });
};

export type ExtendedTenantClient = ReturnType<typeof useFactory>;

export const TENANCY_CLIENT_TOKEN = Symbol('TENANCY_CLIENT_TOKEN');

export const PrismaTenancyClientProvider = {
    provide: TENANCY_CLIENT_TOKEN,
    imports: [PrismaModule],
    inject: [PrismaService, ClsService],
    useFactory
}
```

***Step #4: Create a module that exports the above Factory Provider***
```ts
// prisma-tenancy.module.ts
import { Module, Global } from '@nestjs/common';
import { PrismaTenancyClientProvider, TENANCY_CLIENT_TOKEN } from './prisma-tenancy.provider';
import { PrismaModule } from 'nestjs-prisma';

@Global()
@Module({
    imports: [PrismaModule],
    providers: [PrismaTenancyClientProvider]m
    exports: [TENANCY_CLIENT_TOKEN]
})
export class PrismaTenancyModule {}
```

***Step #5: Add the module above to your root app.module.ts***
Now, you can inject the extended client using the token for your custom provider (`TENANCY_CLIENT_TOKEN`, in this case). 
```ts
import { Injectable, Inject } from "@nestjs/common";
import { 
    TENANCY_CLIENT_TOKEN,
    ExtendedTenantClient,
} from "./prisma-tenancy.provider";

@Injectable()
export class SomeService {
    constructor(
        @Inject(TENANCY_CLIENT_TOKEN) private readonly prisma: ExtendedTenantClient
    ) {}

    // etc
}
```