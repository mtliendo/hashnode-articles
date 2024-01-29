---
title: "Event-Driven Architectures: Real-time Data Across Decoupled Applications"
seoDescription: "Enable real-time apps by having AWS EventBridge directly invoke AppSync mutations, triggering data updates via websockets."
datePublished: Mon Jan 29 2024 21:24:54 GMT+0000 (Coordinated Universal Time)
cuid: clrzfubvw000009jwbw1667of
slug: event-driven-architectures-real-time-data-across-decoupled-applications
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1706563476039/5a920439-8480-4dc5-94fb-c1ce533622bc.png
tags: aws, amazon-web-services, serverless, appsync, event-driven-architecture, aws-eventbridge, focusotter

---

I recently wrote about [how to invoke an AppSync API from a Lambda function](https://hashnode.com/post/clrs2qtft000c08l0dkwh1ajs). While useful and a valid approach, I had 2 gripes with that approach:

1. Manually signing SigV4 requests
    
2. The coupling between the application that sent the request, and the application that consumed it.
    

Fortunately, that all changed when [EventBridge announced the addition of AWS AppSync as a direct target](https://aws.amazon.com/about-aws/whats-new/2024/01/amazon-eventbridge-appsync-target-buses/)🎉

In this post, I'll show how to build out this integration from the position of an event that just got sent to an EventBridge bus. We'll use the AWS CDK to construct this, and show how to test the sample.

> 🗒️ The code for this blog post can be found here:
> 
> %[https://github.com/focusOtter/eventbridge-target-appsync] 

## Project Overview

The repo has all the code, and the readme is detailed, so I'll be going over the services in more detail.

### Amazon EventBridge

When building out API's there's a subtly contract made between the creator and the consumer. That is: Whenever the creator makes a change, the consumer needs to update to meet that change.

A message bus solves this by serving as an intermediary between the two. Put simply, when application events happen on application 1 like "An order is placed", instead of calling the `RewardsAPI` , and `FulfillmentAPI`, a message is put on the message bus:

> 🗣️ "Hey, order `abc123` was placed "

Now, those downstream teams can grab that event and do whatever they want. Teams do this by setting a matching criteria on the event:

"If the message has the word `order` in it."

On AWS, the bus is called EventBridge, and the matching criteria is an EventBridge Rule. Those downstream services are called *targets*.

### AWS AppSync

%[https://www.youtube.com/watch?v=XccNLyZutbU&t=12s] 

Ok, I have recorded, streamed, and written *a lot* of content around AppSync. So definitely checkout my channel and previous blogs to get up to speed.

## Application Overview

The application in the provided repository uses the AWS CDK to create and provision the mentioned services. The CDK allows us to write TypeScript to version our services instead of clicking through the AWS Console.

```typescript
const appName = 'eventbridge-to-appsync'

const auth = createCognitoAuth(this, { appName })

const api = createAppSyncAPI(this, { appName, userpool: auth.userPool })

const cfnAPI = api.node.defaultChild as CfnGraphQLApi

const eventBridge = createEventBridge(this, {
	busName: 'eb-appsync-bus',
	appsyncApiArn: api.arn,
	appsyncEndpointArn: cfnAPI.attrGraphQlEndpointArn,
	graphQlOperation: publishMsgFromEB,
})
```

When it comes to creating the AppSync API, there are one main point of interest:

**Schema Definition**

```graphql
type Mutation {
	publishMsgFromEB(msg: String!): String! @aws_iam
}

type Subscription {
	onPublishMsgFromEb: String
		@aws_cognito_user_pools
		@aws_subscribe(mutations: ["publishMsgFromEB"])
}
```

This tells AppSync to protect the `publishMsgFromEB` mutation with IAM permissions, and that consumers can subscribe to that mutation if they are in a Cognito user pool.

However, the focus is really the EventBridge rule. This forms the connection between the EventBridge bus and the target (AppSync):

```typescript
const mycfnRule = new events.CfnRule(scope, 'cfnRule', {
		eventBusName: bus.eventBusName,
		name: 'mycfnRule',
		eventPattern: {
			source: ['sample.source'],
		},
		targets: [
			{
				id: 'myAppsyncTarget',
				arn: props.appsyncEndpointArn,
				roleArn: ebRuleRole.roleArn,
				appSyncParameters: {
					graphQlOperation: props.graphQlOperation,
				},
				inputTransformer: {
					inputPathsMap: {
						msg: '$.detail.msg',
					},
					inputTemplate: JSON.stringify({
						msg: '<msg>',
					}),
				},
			},
		],
	})
```

This is the L1 construct for creating a rule. It specifically listens for the `sample.source` source on the event payload and invokes our AppSync API in response.

## Conclusion

For detailed instructions on how to test this event, refer to the readme in the repository! In the end, this new integration unlocks a new streamlined approach to creating real-time, event-driven applications that are decoupled from one another.

I'll be posting a bunch more on what you this unlocks so stay tuned for more to come, until then,

Happy coding🦦