---
title: "AWS CDK for Frontend Developers: Databases and Serverless Functions"
datePublished: Sun Mar 19 2023 09:01:48 GMT+0000 (Coordinated Universal Time)
cuid: clff65i5w000809mkeidx6zfa
slug: aws-cdk-for-frontend-developers-databases-and-serverless-functions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679289919996/8677cf10-7a99-4daf-850a-aa59505b8a82.png
tags: startups, aws, full-stack, frontend-development, serverless

---

In the previous post, we scaffolded our CDK project with `context` and additional `props`. In this post, we'll leverage [constructs from the CDK](https://docs.aws.amazon.com/cdk/api/v2/) to create DynamoDB tables, as well as a serverless function. In addition to creating the services themselves, we'll cover roles and policies as necessary.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679203048053/8ddfd4fb-4411-4b8d-b23c-f15f3c50d471.png align="center")

Recall from our architecture diagram that we have two tables:

* **TravelPost**: This will contain information about the trip that was taken. For our application, full CRUD operations will be handled by our API.
    
* **User**: This will contain information about the signed-in user.
    

In addition, we have a serverless function that will populate the `User` table once a user successfully signs up for the first time.

In the next chapter, we'll complete this flow by setting up authentication.

## Going NoSQL with DynamoDB

DynamoDB is a NoSQL database service provided by AWS. Unlike traditional SQL databases that store data in tables with fixed columns and relationships between them, DynamoDB stores data as JSON objects and doesn't rely on strict relationships between tables

From a frontend perspective, DynamoDB can be used to store and retrieve data for web or mobile applications. Moreso, it can help you store and retrieve data for your applications with low latency and high performance, and allows for flexible and scalable data models that can adapt to changing needs.

### Creating the Travel Table

To get started creating our first table, update our `backend-trip-post-stack.ts` file to look like the following:

```typescript
import { CDKContext } from './../cdk.context.d'
import * as cdk from 'aws-cdk-lib'
import { Construct } from 'constructs'
// 👇 this is new
import {createTravelTable} from './database/tables.ts'

export class BackendTripPostStack extends cdk.Stack {
	constructor(
		scope: Construct,
		id: string,
		props: cdk.StackProps,
		context: CDKContext
	) {
		super(scope, id, props)
        // 👇 this is new
		const travelDB = createTravelTable(this, {
			appName: context.appName,
			env: context.environment,
		})
	}
}
```

Note that in addition to the `import` statement, we are calling a function called `createTravelTable`. The function takes in two arguments:

* `this`: This represents the current stack we are in.
    
* `props` : An object that represents values we'd like to pass. In our case, we passing the application name from our `cdk.context.json`file, as well as the current environment.
    

> 🗒️ I personally, like to keep my stack files to where I'm just passing values to services and the actual creation of those services happens in the related file.
> 
> This is a design choice and not a set convention.

With the function in place, let's define our first construct.

From the `lib` directory, create `/database/tables.ts` file and paste in the following:

```typescript

import { Construct } from 'constructs'
import * as awsDynamodb from 'aws-cdk-lib/aws-dynamodb'
import { RemovalPolicy } from 'aws-cdk-lib'
import { envNameContext } from '../../cdk.context'

type BaseTableProps = {
	appName: string
	env: envNameContext
}
type createTravelTableProps = BaseTableProps & {}

export function createTravelTable(
	scope: Construct,
	props: createTravelTableProps
): awsDynamodb.Table {
	const travelTable = new awsDynamodb.Table(scope, 'TravelTable', {
		tableName: `${props.appName}-${props.env}-TravelTable`,
		removalPolicy:
			props.env === 'develop' ? RemovalPolicy.DESTROY : RemovalPolicy.RETAIN,
		billingMode: awsDynamodb.BillingMode.PAY_PER_REQUEST,
		partitionKey: { name: 'id', type: awsDynamodb.AttributeType.STRING },
	})

	return travelTable
}
```

> 🗒️ Since this is our first construct, we'll spend some time going over the arguments and props in a bit more detail. Over time, we'll just focus on the meaningful parts.

The first import is the `Construct`. The `Construct` class is a base class for all constructs in AWS CDK, including the `cdk.Stack` class. By using the `Construct` type for the `scope` parameter, we ensure that the function can accept any construct, not just a `cdk.Stack` instance.

Next, we import `awsDynamodb`. This is the L2 construct from the [CDK library](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_dynamodb-readme.html). Note the import path of `aws-cdk-lib/aws-dynamodb`. All official L2 constructs are bundled under `aws-cdk-lib`.

For the props, we already know we'll have another table, so we're creating a `BaseTableProps` type that both will share, while still allowing each table to define its own.

As far as invoking the actual construct itself, we wrapped it in a function so we could pass in our `appName` and `environment` arguments.

All constructs are created with the `new` command and typically expect a format of `new Construct(scope, id, config)`. In the case of this table, it has the following:

* `scope`: A reference to the current stack or construct.
    
* `id`: A unique identifier for the table.
    
* `config`: An object that configures the properties of the table, including:
    
    * `tableName`: The name of the table. We combine our `id` with our props.
        
    * `removalPolicy`: A `RemovalPolicy` determines whether the table is deleted or retained when the stack is deleted. In `develop` we set it to destroy the table if we destroy our stack. For other branches, it'll remove it from the stack, but keep the table itself.
        
    * `billingMode`: The billing mode for the table. While we can specify the read capacity to optimize our billing and throughput, most applications will benefit from the `PAY_PER_REQUEST` model that will scale up and down automatically based on our needs.
        
    * `partitionKey`: The partition key for the table specifies how data is partitioned and stored in the table. In this case, the partition key is a string `id`. This serves as a unique identifier for all of the items in our database.
        

> 🗒️ A note on pricing:
> 
> Pricing for AWS services can be confusing and I'll do my best to call them out as we go along. Some services have a free tier during the first year of an AWS account opening. Others have it forever.
> 
> DynamoDB offers a generous free tier of 25GB of storage and 100GB of data transfer per month. This is per account, but means for our application, not only are there no servers to manage but there is also no cost.
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679207129333/12411e78-192c-48e4-968c-ed12b755ba30.png align="center")

### Creating our Lambda Function

Our User table will be populated by a Lambda function. A Lambda function is often referred to as a "serverless function" when not referring to just AWS, I'll say Lambda function throughout this series, but just to get us on the same page, for this section I'll say serverless function.

As a frontend/fullstack developer, you may have an idea of what a serverless function is--and for the most part, you're probably correct, but there are some important pieces I want to make sure we understand, specifically the following:

1. **A serverless function executes a bit of code, and then it stops.** As opposed to a server (which can be eternally running), a serverless function is *ephemeral*. It spins up its own environment and when it's done executing and requests have stopped coming in, it'll eventually delete itself.
    
2. **A serverless function takes in an event argument. This is contextual.** Just like in JavaScript anything can call a function, the same is true for a serverless function. Because of that, the `event` object that every function receives will vary based on what is invoking it.
    
3. **A serverless function is not an API route** **but a component of one.** An API route is made up of a gateway (the thing that receives the HTTP request), and the function to invoke (the serverless function). This is important because we often think functions can only be invoked as part of a network request, but again, it can be from just about anything.
    

---

To emphasize that last point about anything being able to call a Lambda function, our function will be triggered everytime a user signs up for our application. While the application will only have one user (you), the same function applies.

However, it's important to remember that multiple services may end up calling our function over time. For that reason, it makes sense to define them in one directory, and then attach them to the services that need them.

To showcase that idea, create a new file called `lib/functions/addUser/main.ts` and paste in the following:

```typescript
import * as AWS from 'aws-sdk'
const docClient = new AWS.DynamoDB.DocumentClient()
import { PostConfirmationConfirmSignUpTriggerEvent } from 'aws-lambda'

exports.handler = async (event: PostConfirmationConfirmSignUpTriggerEvent) => {
	const date = new Date()
	const isoDate = date.toISOString()

	//construct the param
	const params = {
		TableName: process.env.API_HELLO_USERTABLE_NAME as string,
		Item: {
			__typename: 'User',
			id: event.request.userAttributes.sub,
			createdAt: isoDate, // ex) 2023-02-16T16:07:14.189Z
			updatedAt: isoDate,
			username: event.userName,
			email: event.request.userAttributes.email,
		},
	}

	//try to add to the DB, otherwise throw an error
	try {
		await docClient.put(params).promise()
		return event
	} catch (err) {
		console.log(err)
		return event
	}
}
```

> 🗒️ The `PostConfirmationConfirmSignUpTriggerEvent` type is the event that our authentication service will send to us. More on that in the next chapter.

You should be getting TypeScript errors relating to the `aws-sdk` and the `aws-lambda` packages. To fix those, it's important to know two things:

1. The AWS SDK is automatically installed in Lambda functions running in AWS. This means when we install it, we only have to install it as a dev dependency.
    
2. Recall Lambda functions are ephemeral and have their own environment. So when we install dependencies for them, they live in their own `package.json` file.
    

From your terminal, change into the `lib/functions/addUser` directory and run the following commands and the errors should disappear:

```bash
npm init -y && npm i -D aws-sdk @types/aws-lambda
```

While the function itself will be used by our authentication service, we can create the function by calling the `NodejsFunction` L2 construct.

In the `lib/backend-trip-post-stack.ts` file, paste in the following as the first call under `super(scope, id, props)`:

```typescript
//..other imports
import { createTravelTable, createUserTable } from './databases/tables'
import { NodejsFunction } from 'aws-cdk-lib/aws-lambda-nodejs'
import path = require('path')
import { Runtime } from 'aws-cdk-lib/aws-lambda'

const addUserFunc = new NodejsFunction(this, 'addUserFunc', {
	functionName: `${context.appName}-${context.environment}-addUserFunc`,
	runtime: Runtime.NODEJS_16_X,
	handler: 'handler',
	entry: path.join(__dirname, `./functions/addUser/main.ts`),

})
//...creation of db
```

By defining the function here, if there is ever another service that needs to add a user to our table, we can simply pass this function as a prop and update the function's `event` type accordingly.

It's worth noting that the Lambda construct has two different L2 constructs. A [generic construct](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda-readme.html) that allows for code to be written in many different languages, and [this `NodejsFunction` construct](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda_nodejs-readme.html) that only works with JavaScript and TypeScript files, but handles the bundling and transpiling with `esbuild` for us. Since we wrote our function in TypeScript, let's install `esbuild` as a dev dependency so that the CDK can automatically handle the transpilation process for us.

In the root of our project, run the following command:

```bash
npm i -D esbuild
```

> 🗒️ A note on pricing:
> 
> Per the [AWS pricing page for Lambda](https://aws.amazon.com/lambda/pricing/), "The AWS Lambda free tier includes one million free requests per month and 400,000 GB-seconds of compute time per month"
> 
> While the pricing can be a bit confusing due to it being based on both requests and processing time, the gist is that this is a free service for our needs and it cost fractions of a penny for many applications.
> 
> Feel free to checkout the pricing examples in the docs to get a more concrete example.

### Creating the User Table

As for the parts we've already familiar with, let's add those in.

While still in the `lib/backend-trip-post-stack.ts` file, import a `createUserTable` function and add the following underneath our `travelDB` function:

```typescript
//...rest of imports
import { createTravelTable, createUserTable } from './databases/tables'

//...rest of code
//userDB
const userDB = createUserTable(this, {
	appName: context.appName,
	env: context.environment,
	addUserFunc,
})
```

Here we're calling a `createUserTable` function and in addition to our `context` props, we're also passing in the function we just created.

In the `lib/databases/tables.ts` file, we'll create the `createUserTable` which will look similar to what we created earlier. To do so, paste in the following:

```typescript
type CreateUserTableProps = BaseTableProps & {
	addUserFunc: NodejsFunction
}
export function createUserTable(
	scope: Construct,
	props: CreateUserTableProps
): awsDynamodb.Table {
	const userTable = new awsDynamodb.Table(scope, 'UserTable', {
		tableName: `${props.appName}-${props.env}-UserTable`,
		removalPolicy:
			props.env === 'develop' ? RemovalPolicy.DESTROY : RemovalPolicy.RETAIN,
		billingMode: awsDynamodb.BillingMode.PAY_PER_REQUEST,
		partitionKey: { name: 'id', type: awsDynamodb.AttributeType.STRING },
	})

	userTable.grantWriteData(props.addUserFunc)

	return userTable
}
```

The one line that should grab your attention is:

```typescript
userTable.grantWriteData(props.addUserFunc)
```

In AWS, services are deny-by-default. This means unless permissions are defined (either explicitly or implicitly), then they can't talk to one another.

In the case of our Lambda function, we need our DynamoDB table to allow the function to add user data to it. We'll create our own access policies when we talk about S3, but for now, we, fortunately, get to take advantage of a handy utility: `grantWriteData` that will create the access policy for us--all we have to do is pass in the service we want to grant access to.

> 🗒️ While handy, it's important to note the `grantWriteData` function is technically over-permissioned. Intellisense shows the access rights we're permitting.
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679214849275/006f62d0-1109-4d7a-9581-74a56e1dbb8e.png align="center")
> 
> For finer-grained access, there is also the `userTable.grant(props.addUserFunc, "dynamodb:PutItem")` method. Both are fine to use.

## Conclusion

With that in place, we have now successfully provisioned two DynamoDB tables and a Lambda function 🎉

In this section, we dove deep into how CDK constructs are initialized, the arguments they receive and how to add TypeScript types to them. We also discussed what a Lambda function is, and how to create one in TypeScript.

Hopefully, at this point, you're starting to see that creating AWS services in code is part understanding concepts and part calling methods available on L2 constructs. Once those are in place, it's a matter of figuring out how best to organize your code.

In the next chapter, we'll talk all about authentication and authorization with Amazon Cognito.

Until then, happy coding!

🦦