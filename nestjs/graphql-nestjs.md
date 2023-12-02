# How to build a GraphQL API with NestJS

GraphQL is a query languages for APIs and a runtime for fulfilling those queries with your existing data. It provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need, makes it easier to evolve APIs over time, and facilitates the use of powerful developer tools.

## Setting up NestJS app
Under the hood, Nest exposes a GraphQL module that can be configured to use the Apollo GraphQL server in Nest applications. To add GraphQL APIs to our Nest project, we need to install Apollo Server and other GraphQL dependencies:

```bash
$ npm i --save @nestjs/graphql graphql-tools graphql apollo-server-express
```

With the dependencies installed, you can now import `GraphQLModule` into `AppModule`

```typescript
@Module({
  imports: [
    GraphQLModule.forRoot({}),
  ],
})
```

`GraphQLModule` is a wrapper over Apollo Server. It provides a static method, forRoot(), for configuring the underlying Apollo instance.

## Building our GraphQL API
Nest offers 2 methods of building GraphQL APIs: code-first and schema-first. The code-first approach involves using TypeScript classes and decorators to generate GraphQL schemas. With this approach, you can reuse your data model class as a schema and decorate it with the `@ObjectType()` decorator. Nest will autogenerate the schema from your model. Meanwhile, the schema-first approach involves defining the schema using GraphQL's Schema Definition Language (SDL) and then implementing a service by matching the definitions in the schema.

### GraphQL components
The GraphQL API is made up of several components that execute the API requests or form the objects for its responses.

**Resolvers**

Resolvers provide instructions for turning a GraphQL operation (a query, mutation, or subscription) into data. They either return the type of data we specify in our schema, or a promise for that data.

The @nestjs/graphql package automatically generates a resolver map using the metadata provided by the decorators used to annotate classes. 

**Object types**
Object types are the most basic components of GraphQL. It is a collection of fields that you can fetch from your service, with each field declaring a type. Each define object type represents a domain object in your API, specifying the structure of the data that can be queried or mutated in the API. 

The Object types are used to define query objects, mutations, and schema for our API.

```typescript
// src/invoice/customer.model.ts

import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, OneToMany } from 'typeorm';
import { ObjectType, Field } from '@nestjs/graphql';
import { InvoiceModel } from '../invoice/invoice.model';

@ObjectType()
@Entity()
export class CustomerModel {
  @Field()
  @PrimaryGeneratedColumn('uuid')
  id: string;
  @Field()
  @Column({ length: 500, nullable: false })
  name: string;
  @Field()
  @Column('text', { nullable: false })
  email: string;
  @Field()
  @Column('varchar', { length: 15 })
  phone: string;
  @Field()
  @Column('text')
  address: string;
  @Field(type => [InvoiceModel], { nullable: true })
  @OneToMany(type => InvoiceModel, invoice => invoice.customer)
  invoices: InvoiceModel[]
  @Field()
  @Column()
  @CreateDateColumn()
  created_at: Date;
  @Field()
  @Column()
  @UpdateDateColumn()
  updated_at: Date;
}
```

> Note: The `ObjectType` decorator also optionally takes the name of the type being created. It is useful to add this name to the decorator when errors like `Error: Schema must contain uniquely named types but contains multiple types named "Item"` are encountered. <br> An alternative solution to this error is deleting the output directory before building and runnig the app.

**Schemas**

A schema in GraphQL is the definition of the structure of the data queried in the API. It defines the fields of the data, the types, and also operations that can be performed. GraphQL Schemas are written in the GraphQL Schema Definition Language (SDL).

Using the code-first approach, the schema is generated using Typescript classes and ObjectType decorators. The schema generated from CustomerModel class above will look like:

```gql
type CustomerModel {
  id: String!
  name: String!
  email: String!
  phone: String!
  address: String!
  invoices: [InvoiceModel!]
  created_at: DateTime!
  updated_at: DateTime!
}
```

**Field**

Each property in our `CustomerModel` class above is decorated with the `@Field()` decorator. Nest requires us to explicitly use the @Field() decorator in our schema definition classes to provide metadata about each field's GraphQL type, optionality, and attributes, like being nullable.

A field's GraphQL type can be either a scalar type or another object type. GraphQL comes with a set of default scalar types out of the box: Int, String, ID, Float, and Boolean. The Field() decorator accepts an option type function (e.g., type -> Int) and, optionally, an options object.

When the field is an array, we must manually indicate the array type in the Field() decorator's type function. Here is an example that indicates an array of InvoiceModels:

```typescript
 @Field(type => [InvoiceModel])
  invoices: InvoiceModel[]
```

**GraphQL special object types**
There are 2 special types in GraphQL: `Query` and `Mutation`. They serve as parents to other object types and define the entry point to other objects. Every GraphQL API has a `Query` type and may or may not have a `Mutation` type.

The `Query` and `Mutation` objects are used to make requests to GraphQL APIs. Query objects are used to make read (i.e., SELECT) requests on GraphQL APIs while Mutation objects are used to make create, update, and delete requests.

Our invoice API should have a Query object that returns our API objects. Here is an example:

```graphQL
type Query {
  customer: CustomerModel
  invoice: InvoiceModel
}
```

Having created the objects that should exist in our graph, we can now define our resolver class to give our client a way to interact with out API.

In the code-first method, a resolver class both defines resolver functions and generates the Query type. To create a resolver, we'll create a class with resolver functions as methods and decorate the class with the `@Resolver()` decorator:

```typescript
// src/customer/customer.resolver.ts

import { InvoiceModel } from './../invoice/invoice.model';
import { InvoiceService } from './../invoice/invoice.service';
import { CustomerService } from './customer.service';
import { CustomerModel } from './customer.model';
import { Resolver, Mutation, Args, Query, ResolveField, Parent } from '@nestjs/graphql';
import { Inject } from '@nestjs/common';

@Resolver(of => CustomerModel)
export class CustomerResolver {
    constructor(
        @Inject(CustomerService) private customerService: CustomerService, 
        @Inject(InvoiceService) private invoiceService: InvoiceService
    ) {}

    @Query(returns => CustomerModel)
    async customer(@Args('id') id: string): Promise<CustomerModel> {
        return await this.customerService.findOne(id);
    }

    @ResolveField(returns => [InvoiceModel])
    async invoices(@Parent() customer): Promise<InvoiceModel[]> {
        const { id } = customer;
        return this.invoiceService.findByCustomer(id);
    }

    @Query(returns => [CustomerModel])
    async customers(): Promise<CustomerModel[]> {
        return await this.customerService.findAll();
    }
}
```

In our example above, we created the `CustomerResolver`, which defines one query resolver function and one field resolver function. To specify that the method is a query handler, we annotated the method with the @Query() decorator. We also used @ResolveField() to annotate the method that resolves the invoices field of the CustomerModel. And the @Args() decorator is used to extract arguments from a request for use in the query handler.

The @Resolver() decorator accepts an optional argument `of` that is used to specify the parent of a field resolver function. Using our example above, `@Resolver(of => CustomerModel)` indicates that our CustomerModel object is the parent of the field invoices and is passed to the invoices field resolver method.

Our resolver class define above does not hold the logic needed to fetch and return data from our database. Instead, we abstract that logic into a service class, which our resolver class calls. 

```typescript
// src/customer/customer.service.ts

import { Injectable } from '@nestjs/common';
import { CustomerModel } from './customer.model';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { CustomerDTO } from './customer.dto';
@Injectable()
export class CustomerService {
    constructor(
        @InjectRepository(CustomerModel)
        private customerRepository: Repository<CustomerModel>,
      ) {}
      create(details: CustomerDTO): Promise<CustomerModel>{
          return this.customerRepository.save(details);
      }

      findAll(): Promise<CustomerModel[]> {
        return this.customerRepository.find();
      }

      findOne(id: string): Promise<CustomerModel> {
        return this.customerRepository.findOne(id);
      }
}
```

**Mutations**
Mutation methods are used for modifying server-side data in GraphQL.

Technically, a Query could be implemented to add server-side data. But the common convention is to annotate any method that causes data to be written with the @Mutations() decorator. Instead, the decorator tells Nest that such a method is for data modification.

Now, let’s add the new createCustomer() class to our CustomerResolver resolver class:

```typescript
@Mutation(returns => CustomerModel)
async createCustomer(
    @Args('name') name: string,
    @Args('email') email: string,
    @Args('phone', {nullable: true}) phone: string,
    @Args('address', {nullable: true}) address: string, 
): Promise<CustomerModel> {
    return await this.customerService.create({ name, email, phone, address })
}
```

`createCustomer()` has been decorated with @Mutations() to indicate that it modifies or adds new data. If a mutation needs to take an object as an argument, we would need to create a special kind of object called `InputType` and then pass it as an argument to the method. To declare an input type, use the @InputType() decorator:

```typescript
import { PaymentStatus, Currency, Item } from "./invoice.model";
import { InputType, Field } from "@nestjs/graphql";

@InputType()
class ItemDTO {
    @Field()
    description: string;

    @Field()
    rate: number;

    @Field()
    quantity: number;
}

@InputType()
export class CreateInvoiceDTO {
    @Field()
    customer: string;

    @Field()
    invoiceNo: string;

    @Field()
    paymentStatus: PaymentStatus;

    @Field()
    description: string;

    @Field()
    currency: Currency;

    @Field()
    taxRate: number;

    @Field()
    issueDate: Date;

    @Field()
    note: string;

    @Field(type => [ItemDTO])
    items: Array<{ description: string; rate: number; quantity: number }>;
}

@Mutation(returns => InvoiceModel)
async createInvoice(
    @Args("invoice") invoice: CreateInvoiceDTO,
): Promise<InvocieModel> {
    return await this.invoiceService.create(invoice);
}
```

## Testing your GraphQL API using the GraphQL Playground
Now that we've created an entry point to our graph service, we can view our GraphQL API via the playground. The playground is a graphical, interactive, in-browser GraphQL IDE, available by default on the same URL as the GraphQL server itself.

## Benefits of using GrahpQL APIs
GraphQL APIs are popular for providing simplified and efficient communication with the data of a server-side API. Here are some benefits of building and using a GraphQL API:
- **GraphQL requests are faster**: GraphQL allows us to cut down our request and response size by choosing the specific fields you want to query.
- **GraphQL provides flexibility**: One of the advantages of GraphQL over REST is that REST resources usually provide less data than needed (requiring a user to make multiple requests to achieve some functionality), or return unnecessary data where a super resource is built to accommodate multiple use cases. GraphQL solves this by fetching and returning the data fields specified per request.
- **GraphQL structures data hierarchically**: GraphQL structures the relationship between data objects hierarchically, in a graph-like structure.
- **GraphQL is strongly typed**: GraphQL relies on schema, which are strongly typed definitions of the data where each field and level has defined types.
- **With GraphQL, API versioning is not a problem**: Considering that the API user determines the shape of their request and response, it is easier to build on an API to add new functionality and fields without disrupting existing users

>Note: Removing or renaming existing fields will still be disruptive to existing users but it is less disruptive for users when a GraphQL API is expanded than a REST one.