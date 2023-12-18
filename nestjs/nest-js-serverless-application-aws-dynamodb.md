# NestJS serverless application on AWS with DynamoDB

### What is DynamoDB?
As part of the Amazon Web Services porfolio, Amazon offers DynamoDB, a fully managed, proprietary NoSQL database service that supports key-value and document data formats. DynamoDB presents a data architecture comparable to Dynamo, taking its name from it, although DynamoDB has a distinct underlying implementation.

### What is the Serverless Framework?
The Serverless Framework is an open source CLI that allows you to design, deploy, debug, and protect serverless apps with minimum complexity and expense while providing infrastructure resources from AWS, Azure, and Google.

The Serverless Framework offers out-of-the-box structure, automation, and best practices support, allowing you to focus on developing sophisticated, event-driven, serverless systems made up of functions and events.

### Getting started
To start building and deploying AWS serverless functions, you need to set up an AWS account. Then, set up your credentials by running the command:
```bash
aws configure
```

The setup above is required if you wish to deploy your functions to AWS. Next, install the Serverless Framework globally with the command below:
```bash
npm install -g serverless
```

Install dependencies:
```bash
npm install aws-lambda aws-serverless-express express aws-sdk
```

Configure the Serverless Framework -> Create a serverless.yml file in the root directory
```yaml
service: nest-serverless
plugins:
    - serverless-plugin-optimize
    - serverless-offline
    - serverless-dynamodb-local

functions:
    app:
        handler: dist/main.handler
        events:
            - http:
                method: any
                path: /{any+}

provider:
    name: aws
    runtime: nodejs14.x
    apiGateway:
        shouldStartNameWithService: true

    environment:
        AWS_NODEJS_CONNECTION_REUSE_ENABLED: 1
        NODE_OPTIONS: --enable-source-maps --stack-trace-limit=1000
    iam:
        role:
            statements:
                - Effect: 'Allow'
                  Action: 
                      - 'dynamodb:DescribeTable'
                      - 'dynamodb:Query'
                      - 'dynamodb:Scan'
                      - 'dynamodb:GetItem'
                      - 'dynamodb:PutItem'
                      - 'dynamodb:UpdateItem'
                      - 'dynamodb:DeleteItem'
                  Resource: arn:aws:dynamodb:us-west-2:*table/BlogsTable

custom:
    esbuild:
        bundle: true
        minify: false
        sourcemap: true
        exclue: aws-sdk
        target: node14
        define: 'require.resolve: undefined'
        platform: node
        concurrency: 10
    dynamodb:
        start:
            port: 5000
            inMemory: true
            migrate: true
        stages: dev

resources:
    Resources:
        TodosTable:
            Type: AWS::DynamoDB::Table
            Properties:
                TableName: BlogsTable
                AttributedDefinitions:
                    - AttributeName: id
                      AttributeType: S
                KeySchema:
                    - AttributeName: id
                      KeyType: HASH
                ProvisionedThroughput: 
                    ReadCapacityUnits: 1
                    WriteCapacityUnits: 1
```
We specify which plugins are required to replace or enhance the functionality of our project in `Plugins`. We have 2 plugins, serverless-esbuild and serverless offline, which allow us to execute our code locally. DynamoDB can be operated locally with serverless-dynamodb-local.

In `Provider`, we configure cloud provider for our project. To provide our Lambda functions with reading and writing access to our DynamoDB resource table, we set various cloud provider characteristic, like the `name`, `runtime`, `apiGateway`, and `iam` statements.

In `Resources`, we populate our DynamoDB database using our `cloudFormation` resource templates. In this section, we declare specific characteristics like the `tableName`, `AttributeDefinitions`, where we specify our table's primary key ID, and `ProvisionedThroughput`, where we describe the number of units our database can read and write in one second.

We create our own configuration in `Custom`. For our DynamoDB database, we set port 5000. 

Finally, in `Functions`, we  configure our Lambda functions, routes, and route handlers.

To enable us to run our application locally on our computer, we need to install the following plugins:
- `serverless-dynamodb-local`: Connects to DynamoDB locally on our computer
- `serverless-offline`: Starts the application locally
- `serverless-plugin-optimize`: Enables us to run the application locally

Run the command below to install the plugins above:
```bash
serverless plugin install -n serverless-plugin-optimize
serverless plugin install -n serverless-dynamodb-local
serverless plugin install -n serverless-offline
```

Finally, install DynamoDB locally with the command below:
```
serverless dynamodb install
```

### Convert our application to AWS Lambda
We'll convert our NestJS app to and AWS Lambda function in our src/main.ts file. First, we need to map our application to an Express app with the code snippet below:
```ts
import { NestFactory } from '@nestjs/core';
import { ExpressAdapter } from '@nestjs/platform-express';
import { INestApplication } from '@nestjs/common';
import { Express } from 'express';
import { Server } from 'http';
import { Context } from 'aws-lambda';
import { createServer, proxy, Response } from 'aws-serverless-express';
import * as express from 'express';
 
import { AppModule } from './app.module';

let cachedServer: Server;

async function createExpressApp(expressApp: Express): Promise<INestApplication> {
    const app = await NestFactory.create(
        AppModule, 
        new ExpressAdapter(expressApp)
    );
    return app;
}
// ...
```

Then, we'll convert our Express app into an AWS Lambda function so that we can run our app as Lambda functions:
```ts
async function bootstrap(): Promise<Server> {
    const expressApp = express();
    const app = await createExpressApp(expressApp);
    await app.init();
    return createServer(expressApp);
}

export async function handler(event: any, context: Context): Promise<Response> {
    if(!cachedServer) {
        const server = await bootstrap();
        cachedServer = server;
    }
    return proxy(cachedServer, event, context, 'PROMISE').promise;
}
```
At this point, we can run our NestJS application as an AWS Lambda function. 

### Create a blog service
Let’s create the CRUD operations for our blog API.
```ts
// src/app.service.ts
import { Injectable, InternalServerErrorException } from '@nestjs/common';
import { v4 as uuid } from 'uuid';
import * as AWS from 'aws-sdk'; // we imported the AWS SDK to interact with our DynamoDB
import Blog from './interface';

// We’ll use the AWS.DynamoDB.DocumentClient method to create our CRUD operations
const dynamoDB = process.env.IS_OFFLINE 
    ? new AWS.DynamoDB.DocumentClient({
        region: "localhost",
        endpoint: process.env.DYNAMO_ENDPOINT
    })
    : new AWS.DynamoDB.DocumentClient();

@Injectable()
export class AppService {
    async getBlogs(): Promise<any> {
        try {
            return dynamoDB.scan({ TableName: "BlogsTable" }).promise();
        } catch (e) {
            throw new InternalServerErrorException(e);
        }
    }

    async createBlog(blog: Blog): Promise<any> {
        const blogObj = {
            id: uuid(),
            ...blog
        };
        try {
            return await dynamoDB.put({
                TableName: "BlogsTable",
                Item: blogObj,
            }).promise();
        } catch (e) {
            throw new InternalServerErrorException(e)
        }
    }

    async getBlog(id: string): Promise<any> {
        try {
            return await dynamoDB.get({
                TableName: process.env.USERS_TABLE_NAME,
                Key: { id },
            }).promise();
        } catch (e) {
            throw new InternalServerErrorException(e);
        }
    }

    async deleteBlog(id: string): Promise<any> {
        try {
            return await dynamoDB.delete({
                TableName: "BlogsTable",
                Key: {
                    todosId: id,
                }
            }).promise();
        } catch (e) {
            throw new InternalServerErrorException(e);
        }
    }
}
```

Now, create an interface.ts file in the src folder and add the code snippet below:
```ts
export default interface Blog {
  title: string;
  coverImage: String;
  body: string;
  createdBy: string;
  dateCreated: string;
}
```

### Create blog controllers
```ts
import { Controller, Get, Post, Delete, Body, Param } from '@nestjs/common';
import { AppService } from './app.service';
import Blog from './interface';

@Controller('blogs')
export class AppController {
    constructor(private readonly appService: AppService) {}

    @Get()
    async getTodos(): Promise<Blog[]> {
        return await this.appService.getBlogs();
    }

    @Post()
    async createTodo(@Body() blog: Blog): Promise<Blog> {
        return await this.appService.createBlog(blog);
    }

    @Post(':id')
    async getTodo(@Param() id: string): Promise<Blog> {
        return await this.appService.getBlog(id);
    }

    @Delete(':id')
    async deleteTodo(@Param() id: string): Promise<any> {
        return await this.appService.deleteBlog(id);
    }
}
```

### Test the application
Run our application with the command below. Since our handler points to the dist folder, we need to build the application before running it:
```bash
npm run build && serverless offline start
```

### Deploy the application
**Deploy all** <br>
This is the main and most commonly used method for deploying serverless applications. When you’ve updated your function, event, or resource configuration in serverless.yml and want to deploy that change (or multiple changes at once) to Amazon Web Services, use this method. To deploy the application using this method, run the command below:
```
serverless deploy
```

**Direct CloudFormation deployments** <br>
This is a faster method of deploying serverless functions. Add the configuration below to serverless.yml file:
```yml
provider: 
    ...
    deploymentMethod: direct
```
This method’s default settings are dev stage and us-east-1 region. You can change the default stage and region by passing the following flags to the command:
```
serverless deploy --stage beta --region eu-central-1
```

**Deploy function** <br>
This is the fastest method of deploying serverless functions. It simply overwrites the current function’s zip file on AWS without modifying the AWS CloudFormation stack:
```
serverless deploy function --function funtionToDeploy
```

