---
title: "How to Invoke AppSync from a Lambda function"
seoDescription: "Learn how to connect Lambda and AppSync to enable serverless GraphQL integrations using AWS best practices for IAM authorization and signing."
datePublished: Wed Jan 24 2024 17:43:52 GMT+0000 (Coordinated Universal Time)
cuid: clrs2qtft000c08l0dkwh1ajs
slug: how-to-invoke-appsync-from-a-lambda-function
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1706118096196/4c1b4d9b-2402-4a25-91d9-8420b5f064b9.png
tags: aws, graphql, serverless, aws-lambda, appsync, indie-maker, solopreneur, focusotter

---

The only time I drink Starbucks is when I travel through the airport. Last time, I noticed they now allow you to skip the line buy ordering from a QR Code. After making your online order, the baristas still get your order on a screen.

In another use case, I want to create an AI-generated bedtime story for my kids. When it comes to creating the image, audio, and story, these things take time. Instead of polling, I'd like to be notified when it's done.

Both of those scenarios are examples where you'd want to call AppSync from a Lambda function and what the focus of this post is about.

> 📹 This post now has a full video guide!
> 
> %[https://youtu.be/-qogqNXlDKM] 

%[https://github.com/focusOtter/lambda-invoke-appsync] 

> 🗒️ I provided a repo above incase you want to get straight to the good bits!

If you're familiar with Lambda functions but new to AWS AppSync, no worries, I have the video for you!

%[https://www.youtube.com/watch?v=XccNLyZutbU&t=6s] 

## NodeJS Lambda Functions in the AWS CDK

Fortunately, when it comes to the AWS CDK and Lambda functions, we have the ability to create our resources in TypeScript. From the `readme`, this is how we create the Lambda function:

```typescript
export const createInvokeAppSyncFunc = (
	scope: Construct,
	props: InvokeAppSyncFuncProps
) => {
	const invokeAppSyncFunc = new NodejsFunction(
		scope,
		`${props.appName}-invokeAppSyncFunc`,
		{
			functionName: `${props.appName}-invokeAppSyncFunc`,
			runtime: Runtime.NODEJS_18_X,
			handler: 'handler',
			entry: path.join(__dirname, `./main.ts`),
		}
	)

	return invokeAppSyncFunc
}
```

Note that aside from some props being passed in, the core is essentially giving it a name, a runtime, and pointing it to the file location.

## How to allow Lambda to Sign with SigV4

[This file sucks.](https://github.com/focusOtter/lambda-invoke-appsync/blob/main/lib/functions/invokeAppSyncFunc/appsyncAuthUtil.ts) I wish it were easier [(now it is!)](https://blog.focusotter.com/how-aws-appsync-and-amazon-eventbridge-unlock-real-time-data-across-domains). The good news is that you never have to modify it. There are some NPM packages out there that allow you to install it, but it's simple enough to just paste in a project.

Let's break it down:

```typescript
import { SignatureV4 } from '@aws-sdk/signature-v4'
import { Sha256 } from '@aws-crypto/sha256-js'
import { defaultProvider } from '@aws-sdk/credential-provider-node'
import { HttpRequest } from '@aws-sdk/protocol-http'
import { default as fetch, Request } from 'node-fetch'
```

Those imports are needed because the `@aws-sdk` v3 takes a modular approach. So every bit and piece comes from a standalone package.

Next, we have the following:

```typescript
// deconstruct the url and create a URL object
const endpoint = new URL(params.config.url)

// create something that knows how to let Lambda sign AppSync requests
const signer = new SignatureV4({
	credentials: defaultProvider(),
	region: params.config.region,
	service: 'appsync',
	sha256: Sha256,
})
```

Not too bad 😄 This parses the AppSync GraphQL endpoint in an object containing it's various pieces (protocol, pathname, etc).

In addition, we create a signer by passing in details relating to what we're trying to sign. Sigv4 is similar to creating a JWT or hashing a password in my opinion. Not from a cryptography standpoint, but from a "Hey, I'm going to pass you stuff, and you do all the hard work for me" standpoint.

From there, we keep it going by taking that signing mechanism and passing it the request we are trying to sign:

```typescript
// Setup the request that we are wanting to sign  with our URL and signer
const requestToBeSigned = new HttpRequest({
	hostname: endpoint.host,
	port: 443,
	path: endpoint.pathname,
	method: 'POST',
	headers: {
		'Content-Type': 'application/json',
		host: endpoint.host,
	},
	body: JSON.stringify(params.operation),
})

// Actually sign the request
const signedRequest = await signer.sign(requestToBeSigned)

// Create an authenticated request for fetch
const request = new Request(endpoint, signedRequest)
```

With all that in place, we now have a request that is signed, sealed and in the `try/catch` block, delivered!

## Usage

In the `main.ts` file, we actually use the `AppSyncRequestIAM` helper method by invoking it with our AppSync operation:

```typescript
 const res = await AppSyncRequestIAM({
  config: {
    region: process.env.REGION as string,
    url: process.env.APPSYNC_API_URL as string,
  },
  operation: {
    operationName: 'BroadcastMessage',
    query: broadcastMessage,
    variables: {
      msg: event.msg,
    } as BroadcastMessageMutationVariables,
  },
})
```

Note that this only pertains to signing the request. It will never get this far to begin with if the following isn't enabled:

1. The AppSync API doesn't have `IAM` authorization enabled
    
2. The Schema doesn't have the `@aws_iam` directive listed on the operation
    
3. The Lambda was never given persmissions to invoke AppSync (`api.grantMutation(invokeAppSyncFunc`)
    

---

## Conclusion

Calling AppSync from a Lambda function--or any out-of-band service, requires IAM permissions. This *can* be tricky, especially the first time. But I hope this post showed you just how simple things can be when you understand the moving pieces.

What are some use cases you'd like to see covered? Let me know in the comments or on social media.

Until then, Happy Coding 🦦