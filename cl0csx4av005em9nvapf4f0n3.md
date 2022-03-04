## Trigger AppSync Subscriptions with EventBridge targets

[AWS AppSync](https://aws.amazon.com/appsync/) is a managed graphQL service that comes with built-in websockets so applications can subscribe to real-time updates. Oftentimes, this means something like, "When someone adds a todo, let any subscriber see the new todo."

But the actual use case can be much more interesting. In this post, we'll explore the following scenario: 

> "When a customer is close to a café, update their order  from `PROCESSED` to `ARRIVING` and notify a store in real-time that they can begin making it."

We'll explore the location tracking in the next post, but this post will assume that the customer is close enough to the store that it triggered the event and it's now our job to notify the store.

## Understanding our services

![iam-appsync.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1646421175749/g9In4louZC.png)

The above image shows an event matching a rule on [Amazon EventBridge](https://aws.amazon.com/eventbridge/). 

The matched rule is then sent to an event bus in our account. The event bus routes the event to a Lambda function that calls an `update` mutation on our AppSync API. 

The mutation (from AppSync), calls DynamoDB to update a field on our table.

> 🗒️ If needing a refresher on AppSync and it connecting to DynamoDB, I have a beginner-friendly video that's just for you!

%[https://www.youtube.com/watch?v=OK2B8cp1EyE]

## Modeling our AppSync Schema

Let's take a look at what an AppSync schema would need to look like if wanting to do something like this.

> 🗒️Running `amplify add api` in an Amplify project will walk you through the steps to create an AppSync API😎

The below AppSync schema is decorated with directives from AWS Amplify, but the same principles apply:

```rb
type Order
	@model
	@auth(
		rules: [
			{ allow: private, operations: [create] }
			{ allow: groups, groups: ["employee"], operations: [read] }
			# EventBridge will automatically call this lambda function to update orders
			{ allow: private, provider: iam, operations: [read, update] }
		]
	) {
	id: ID!
	products: [String!]! #They already paid, so we just need the product name(s)
	status: ORDER_STATUS!
	customerName: String!
}

enum ORDER_STATUS {
	PROCESSED # The customer just clicked the "buy" button
	ARRIVING # The customer just entered a geofence
	FULFILLED #The employee gave the customer their order
}
```

The `@model` directive will automatically create a DynamoDB table with CRUD resolvers. 

The `@auth` directive is where things get interesting. By specifying rules on our `Order` type, we can have fine-grained authorization on our API.

* `{ allow: private, operations: [create] }`: Signed in users can create orders.
* `{ allow: groups, groups: ["employee"], operations: [read] }`: Users in the "employee" cognito group can read orders.
* `{ allow: private, provider: iam, operations: [update] }`: AWS services with IAM permissions to access are restricted to updating  an order.

As you can see, AppSync's integration with Amazon Cognito makes setting up authZ (authorization) rules very easy. Amplify takes it a step further by managing the DynamoDB connections as well.

> 🔥Tip: When creating an AppSync API with Amplify, the CLI will ask if you'd like code generated for you. Selecting `yes` will add a `graphql` folder to your project with `queries.js`, `mutations.js`, and `subscriptions.js` files. We'll take advantage of this shortly.

## Creating a Lambda function

We want to create a Lambda function that gets called by EventBridge but also has permission to update our AppSync API by calling a mutation.

This can be done from the AWS console, but we'll again leverage AWS Amplify to handle this for us.

By leveraging the Amplify CLI, creating a new function that is associated with our AppSync API is as easy as answering a few questions.

![create a function with IAM role containing permission to mutate appsync](https://cdn.hashnode.com/res/hashnode/image/upload/v1646417921763/KJgX6h1ic.png)

As you can see, by running `amplify add function`, we're prompted to give our function a name, and runtime. As an advanced setting, you can associate the function with existing services in our application.

Selecting `api` will allow us to select the type of change we'd like to enable. This will update the function's IAM role with [a policy that allows us to call a mutation](https://docs.aws.amazon.com/appsync/latest/devguide/security-authz.html#aws-iam-authorization).

---

As for the code in our Lambda function, because we told Amplify that we need access to our AppSync API, it will automatically provide a `process.env` variable for our AppSync endpoint.

The code to call our API looks like this:

```js
/* Amplify Params - DO NOT EDIT
    API_ORDERPROJECT_GRAPHQLAPIENDPOINTOUTPUT
    API_COFFEESHOPTECH_GRAPHQLAPIIDOUTPUT
    ENV
    REGION
Amplify Params - DO NOT EDIT */

/**
 * @type {import('@types/aws-lambda').APIGatewayProxyHandler}
 */

const AWS = require('aws-sdk')
const urlParse = require('url').URL
const fetch = require('node-fetch')
const updateOrder = require('./graphql/mutations').updateOrder

exports.handler = async (event) => {
	const appsyncUrl = process.env.API_ORDERPROJECT_GRAPHQLAPIENDPOINTOUTPUT
	const region = process.env.REGION

	//same as appsyncUrl but without the "https://"
	const endpoint = new urlParse(appsyncUrl).hostname
	const httpRequest = new AWS.HttpRequest(appsyncUrl, region)

	httpRequest.headers.host = endpoint
	httpRequest.headers['Content-Type'] = 'application/json'
	httpRequest.method = 'POST'

	//request to update an order with AppSync start
	const updateOrderById = (id) => {
		const updateOrderBody = {
			query: updateOrder,
			operationName: 'UpdateOrder',
			variables: {
				input: {
					id,
					status: 'ARRIVING',
				},
			},
		}

		httpRequest.body = JSON.stringify(updateOrderBody)

		signer = new AWS.Signers.V4(httpRequest, 'appsync', true)
		signer.addAuthorization(AWS.config.credentials, AWS.util.date.getDate())

		const options = {
			method: httpRequest.method,
			body: httpRequest.body,
			headers: httpRequest.headers,
		}
		return fetch(appsyncUrl, options).then((res) => res.json())
	} //request to update an order with AppSync end

	//make the call to update an order, sending the updated order back to the client
	try {
		const { orderId } = event.detail
		const updatedOrder = await updateOrderById(orderId)

		return {
			statusCode: 200,
			body: updatedOrder,
		}
	} catch (e) {
		console.log({ error: e })
		return { statusCode: 404, body: { error: e } }
	}
}
```
While it may look complicated, let's break it down.

1. Using the graphql endpoint Amplify provided to us, we pass it to the native `URL` package in NodeJS. This returns an object that contains the parts of our API such as `host`, `hostname`, `href`, etc. 

We're mainly using it to go from `https://our-api.com` to just `our-api.com`.

2. We attach the host as well as a couple of other properties to our headers.
3. We create a function that will call our AppSync API by taking in the ID of the order we're updating.
4. Inside of that function, we stub out the body of our request and stringify it.
5. We sign the request so that AWS knows it's trusted and can be used for AppSync.
6. Using those now-signed headers, we pass the URL and the headers to our `fetch` request.

> 🗒️Because `fetch` has landed in `nodev17`, I'm using the `node-fetch` package to make modifying this function in the future simpler.

> 🔥 Remember in the API section above that I mentioned Amplify will automatically generate the `queries`, and `mutations`? Notice that I simply copied that file over to this Lambda function's directory hence the line:
> `const updateOrder = require('./graphql/mutations').updateOrder`

Now when this function is triggered, it will update our AppSync API and any subscribed listeners will get notified

## Listening for API updates

In a frontend application, you can use the `API` module from the `aws-amplify` package [to subscribe to events](https://docs.amplify.aws/lib/graphqlapi/subscribe-data/q/platform/js/). An example for doing so in a react app would look like the following:

```js
useEffect(() => {
	const subscription = API.graphql({
		query: onUpdateOrder,
	}).subscribe({
		next: ({ provider, value }) => {
			console.log({ provider, value })
			if (value.data.onUpdateOrder.status === 'ARRIVING') {
				setOrders((currentOrders) => {
					return [value.data.onUpdateOrder, ...currentOrders]
				})
			}
		},
	})

	return () => {
		subscription.unsubscribe()
	}
}, [])
```

## Setting up EventBridge

Let's go the extra mile on this post and show how one would set up EventBridge to send the event.

Again, this can be done from the AWS console, but we'll use Amplify to create the EventBridge rule, add our function as a target.

Running the command `amplify add custom` and selecting `CDK` will scaffold out a CDK application that is aware of our Amplify project.

During scaffolding, we also import a `AmplifyDependentResourcesAttributes` object that contains all the references to our Amplify-generated services. We'll use that to grab our function:

```ts
import * as cdk from '@aws-cdk/core'
import * as AmplifyHelpers from '@aws-amplify/cli-extensibility-helper'
import { AmplifyDependentResourcesAttributes } from '../../types/amplify-dependent-resources-ref'
import * as events from '@aws-cdk/aws-events'
import * as targets from '@aws-cdk/aws-events-targets'
import * as lambda from '@aws-cdk/aws-lambda'

export class cdkStack extends cdk.Stack {
	constructor(
		scope: cdk.Construct,
		id: string,
		props?: cdk.StackProps,
		amplifyResourceProps?: AmplifyHelpers.AmplifyResourceProps
	) {
		super(scope, id, props)
		/* Do not remove - Amplify CLI automatically injects the current deployment environment in this input parameter */
		new cdk.CfnParameter(this, 'env', {
			type: 'String',
			description: 'Current Amplify CLI env name',
		})

		const rule = new events.Rule(this, 'cafeCustomerEnter', {
			eventPattern: {
				source: ['aws.geo'],
				detailType: ['Location Geofence Event'],
				detail: {
					EventType: ['ENTER'],
				},
			},
		})
		
		//grab a reference to our Amplify function
		const retVal: AmplifyDependentResourcesAttributes =
			AmplifyHelpers.addResourceDependency(
				this,
				amplifyResourceProps.category,
				amplifyResourceProps.resourceName,
				[{ category: 'function', resourceName: 'sampleFuncToAppSync' }]
			)

		const coffeeOrderReceivedFunc = lambda.Function.fromFunctionArn(
			this,
			'sampleFuncToAppSync',
			//when the app synthesizes, grab the actual ARN of the function
			cdk.Fn.ref(retVal.function.sampleFuncToAppSync.Arn)
		)

		// add this rule to the default event bus
		rule.addTarget(new targets.LambdaFunction(sampleFuncToAppSync))
	}
}
```

As you can see, we create our rule, grab our function, and then add the function to the rule as a target.

The rule itself is an event that comes from [Amazon Location Service](https://aws.amazon.com/location/), specifically whenever a customer enters a geofence.

If you'd like to see what that piece looks like, let me know down in the comments!

## Conclusion

While it's certainly possible to [use AppSync and EventBridge without a Lambda](https://aws.amazon.com/blogs/mobile/appsync-eventbridge/) if your API is public, when working with IAM permissions, this is a really elegant solution that can be put in version control and reused.



