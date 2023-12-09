# End-to-end testing in NestJS with TypeORM
Quality assurance strategy is a key aspect of software development. High-end tests such as unit and end-to-end tests are important aspects of a QA strategy.

In this post, our focus will be on creating end-to-end tests in NestJS. NestJS comes with Jest and Supertest out of the box for creating unit tests. We'll explore the thought process behind test-driven development and unit tests for development.

We'll use unit tests to guide development of the product creation feature in our API, while end-to-end tests will simulate the end user of the application. TypeORM and PostgresSQL provide database connectivity in our application.

[What is test-driven development](#what-is-test-driven-development)
[Test-driven development steps](#test-driven-development-steps)
[Developing our product API with TDD](#developing-our-product-api-with-tdd)

## What is test-driven development?
Test-driven development (TDD) enables developers to build software guided by tests. This enables rapid testing, coding, and refactoring.

There are 3 simple rules to abide by test-driven development:
1. Write a test for next bit of functionality
2. The code must be functional before the test passes
3. Refactoring is critical to ensure that both old and new code is well-structured

Test-driven development provides great advantages when practiced. Developers working with TDD have to model the interface of the code, which enables the separation of the interface from the implementation. The tests are created from the perpective of a class's public interface, which means the key focus is on the class's behavior and not the implementation.

In a production pipeline, these tests run on almost every build, ensuring the behavior of the code. If a team member makes a change to the code that changes the behavior, the tests fail, signaling a mistake. This saves time on debugging.

The image below underlies the philosophy behind test-driven development:
https://blog.logrocket.com/wp-content/uploads/2022/10/tdd-cycle.png

To use TDD, the "red", "green", and "refactor" cycle is mandatory. The cycle is to run until the new features is ready and all tests pass.

## Test-driven development steps
### 1. Think
The point of TDD is to create small tests and write code that forces the tests to pass. The iterative process continues until the feature is complete. In the "think" step, it's important to imagine and note what behavior the code should exhibit. The smallest increment that requires the fewest lines of code is key - you can create a small unit of tests that fail until the behavior is present this way.

### 2. Red bar
In the Red bar, the unit tests that exist should cover the current increment of behavior. The test can contain method and class names that don't exist yet. The design of the class's interface is created from the perspective of the user of the class. 

Once the tests are in place and you run the entire suite, the new test should fail. In most TDD testing tools, the failed test produces a red bar where we gain insights into the cause of the failure.

### 3. Green bar
In this step, we focus on writing production-ready code to get the test to pass. Once the tests run, our result should be a green progress bar, depending on the TDD testing tool that is in use. The green bar step provides another opportunity to check our intent against reality.

## Developing our product API with TDD
github repo: https://github.com/dueka/nestjsPipeline

To assist with mocking HTTP objects, we'll use the `node-mocks-http` library.
Add the following code to product.controller.spec.ts file:
```typescript
import { Test, TestingModule } from "@nestjs/testing";
import { DeleteResult, UpdateResult } from "typeorm";
import { User } from "../../auth/models/user.class";
import { JwtGuard } from "../../auth/guards/jwt.guard";
import { UserService } from "../../auth/services/user.service";
import { ProductController } from "./product.controller";
import { ProductService } from "../services/product.service";
import { Product } from "../models/product.interface";

const httpMocks = require("node-mocks-http");

describe("ProductController", () => {
    let productController: ProductController;

    const mockRequest = httpMocks.createRequest();
    mockRequest.user = new User();
    mockRequest.user = "DUE";

    const mockProduct: Product = {
        body: "nivea",
        createdAt: new Date(),
        creator: mockRequest.user,
    }

    const mockProductService = {
        createProduct: jest
            .fn()
            .mockImplementation((user: User, product: Product) => {
                return {
                    id: 1,
                    ...product
                }
            })
    };
    const mockUserService = {}

    // create fake module 
    beforeEach(async () => {
        const moduleRef: TestingModule = await Test.createTestingModule({
            controllers: [ProductController],
            providers: [
                ProductService,
                { provide: UserService, useValue: mockUserService },
                {
                    provide: JwtGuard,
                    useValue: jest.fn().mockImplementation(() => true)
                },
            ],
        })
            .overrideProvider(ProductService)
            .useValue(mockProductService)
            .compile();

        productController = moduleRef.get<ProductController>(ProductController);
    });
});
```

To close out the unit test, the action and its response are set. Paste the below code at the end of the `describe` function.
```typescript
it('should create a product', () => {
    expect(productController.create(mockProduct, mockRequest)).toEqual({
        id: expect.any(Number),
        ...mockProduct
    });
});
```

In the product.controller.ts file, set up the `create` method. Import the dependencies, the product interface, and the product entity.
```typescript
import { Body, Controller, Post, Request, UseGuards } from "@nestjs/common";
import { Observable } from "rxjs";

import { JwtGuard } from "../auth/guards/jwt.guard";
import { Product } from "../product/models/product.interface";
import { ProductService } from "../product/services/product.service";

@Controller("product")
export class ProductController {
  constructor(private productService: ProductService) {}
  @UseGuards(JwtGuard)
  @Post()
  create(@Body() product: Product, @Request() req): Observable<Product> {
    return this.productService.createProduct(req.user, product);
  }
}
```

**Run the first unit test**
In a new terminal window, run the created test with the following command:
```bash
$ yarn test product.controller
```
The test should fail because the implementation isn't complete yet. This provides insight into how we can make of use of TDD to improve our development and catch bugs. Our test is failing because the method isn't in place in the product.service.ts file.
https://blog.logrocket.com/wp-content/uploads/2022/10/failing-test.png

open the product.service.ts file and paste the code block below:
```typescript
import { Inject, Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { from, Observable } from "rxjs";
import { DeleteResult, Repository, UpdateResult } from "typeorm";

import { User } from "src/auth/models/user.class";
import { ProductEntity } from "../models/product.entity";
import { Product } from "../models/product.interface";

@Injectable()
export class ProductService {
  constructor(
    @InjectRepository(ProductEntity)
    private readonly productRepository: Repository<ProductEntity>
  ) {}

  createProduct(user: User, product: Product): Observable<Product> {
    product.creator = user;
    return from(this.productRepository.save(product));
  }
}
```
In the terminal window, re-run the product.controller test

Fix the issue by adding the createProduct method in our service class.

https://blog.logrocket.com/wp-content/uploads/2022/10/fixing-issue.png

This cycle is greater for developing new features in applications. The cycle provides assurance that the created methods are well thought out.

**Creating the Get Product feature**
The mockProductService object holds a new method that can receive values during retrieval.
```typescript
const mockProductService = {
    createProduct: jest
      .fn()
      .mockImplementation((user: User, product: Product) => {
        return {
          id: 1,
          ...product,
        };
      }),
    findProducts: jest
      .fn()
      .mockImplementation((numberToTake: number, numberToSkip: number) => {
        const productsAfterSkipping = mockProducts.slice(numberToSkip);
        const filteredProducts = productsAfterSkipping.slice(0, numberToTake);
        return filteredProducts;
      }),
  };
```

The next step is define the test that we want to run. Our `take` and `skip` values are 2 and 1, respectively. Those values serve as parameters.
```typescript
 it("should get 2 products, skipping the first", () => 
    expect(productController.findSelected(2, 1)).toEqual(mockProducts.slice(1));
  });
```

Run the test. It should fail, as the necessary methods aren’t available in the controller.

In the product.controller.ts file, create the GET route:

```typescript
...
@UseGuards(JwtGuard)
@Get()
findSelected(
    @Query("take") take: number = 1,
    @Query("skip") skip: number = 1
): Observable<Product[]> {
    take = take > 20 ? 20 : take;
    return this.productService.findProducts(take, skip);
}
...
```

In the product.service.ts file, define the findProduct method; the findSelected route calls this method.
```typescript
...
findProducts(take: number = 10, skip: number = 0): Observable<Product[]> {
    return from(
        this.productRepository
            .createQueryBuilder("product")
            .innerJoinAndSelect("product.creator", "creator")
            .orderBy("product.createdAt", "DESC")
            .take(take)
            .skip(skip)
            .getMany()
    );
};
```
## What are end-to-end tests?
And end-to-end testing suite enables developers to test the software product. The end goal of this test is to ensure that the application behaves as expected.

Simulating how users interact with the application in end-to-end test suites is key. The simulation enables developers to confirm how the system operates under test. We would set up an end-to-end testing suite for the authentication controller.

## End-to-end tests in NestJS
NestJS provides E2E testing by default. To create an instance of our application, we rely on Supertest. Supertest enables the detection of endpoints in our application and creates requests.

There are 2 approaches to explore when creating E2E tests:
1. Mock all testing modules and then call the controllers and services
2. Test the endpoints with a database

Our approach enables direct requests to the endpoints of the API.

## Creating end-to-end tests using auth controller
In the auth controller, the implementation of E2E tests for registering a user and login are key. An auth.e2e-spec.ts file holds the E2E tests we need.

In the root directory, locate the test folder, and create an auth.e2e-spec.ts file. The describe method and auth URL for the authController will live in this file with our E2E tests for it.
```typescript
import { HttpStatus } from '@nestjs/common';
import * as jwt from 'jsonwebtoken';
import * as request from 'supertest';

import { User } from '../src/auth/models/user.class';

describe('AuthController (e2e)', () => {
      const authUrl = `http://localhost:3000/api/auth`;
});
```

Test the registration service and controller to check the `user` object after registration.

The code block below is the `registerAccount` method to test.
```typescript
doesUserExist(email: string): Observable<boolean> {
    return from(this.userRepository.findOne({ email })).pipe(
        switchMap((user: User) => {
            return of(!!user);
        })
    )
}

registerAccount(user: User): Observable<User> {
    const { givenName, familyName, email, password } = user;

    return this.doesUserExist(email).pipe(
        tap((doesUserExist: boolean) => {
            if(doesUserExist) {
                throw new HttpException(
                    "A user has already been created with this email address",
                    HttpStatus.BAD_REQUEST
                )
            }
        }),
        switchMap(() => {
            return this.hashPassword(password).pipe(
                switchMap((hashedPassword: string) => {
                    return from(
                        this.userRepository.save({
                            givenName,
                            familyName,
                            email,
                            password: hashedPassword,
                        })
                    ).pipe(
                        map((user: User) => {
                            delete user.password;
                            return user;
                        })
                    )
                })
            )
        })
    )
}
```

The code snippet above provides information on the variables needed to register users. The returned user object represents the response received in the client application.

Inside the `describe` block, there is a nested `describe` block for targeting the '/auth/register/' endpoint. In the `it` block, create a request, and return the mockUser object as part of the response. The `expect` variable is key for checking the validity of the responses.
```typescript
import { HttpStatus } from "@nestjs/common";
import * as jwt from "jsonwebtoken";
import * as request from "supertest";

import { User } from "../src/auth/models/user.class";

describe("AuthController (e2e)", () => {
    const authUrl = `http://localhost:6000/auth`;

    const mockUser: User = {
        givenName: "givenName",
        familyName: "familyName",
        email: "email@homtail.com",
        password: "password",
    };

    describe("/auth/register (POST)", () => {
        it("should register a user and return the new user object", () => {
            return request(authUrl)
                .post("/register")
                .set("Accept", "application/json")
                .send(mockUser)
                .expect((response: request.Response) => {
                    const {
                        id,
                        givenName,
                        familyName,
                        password,
                        email,
                        imagePath,
                        role,
                    } = response.body;

                    expect(typeof id).toBe("number"),
                    expect(givenName)toEqual(mockUser.givenName),
                    expect(familyName).toEqual(mockUser.familyName),
                    expect(email).toEqual(mockUser.email),
                    expect(password).toBeUndefined();
                    expect(imagePath).toBeNull();
                    expect(role).toEqual("user");
                })
                .expect(HttpStatus.CREATED)
        })
    })
})
```

Test registration service to ensure new users aren’t registered with an existing email
```typescript
it('it should not register a new user if the passed email already exists', () => {
      return request(authUrl)
        .post('/register')
        .set('Accept', 'application/json')
        .send(mockUser)
        .expect(HttpStatus.BAD_REQUEST);
    });
```
