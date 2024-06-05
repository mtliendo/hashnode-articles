---
title: "Easily Email Digital Products with Stripe and Amplify Gen 2"
seoTitle: "Email Digital Products with Stripe & Amplify Gen 2"
datePublished: Wed Jun 05 2024 18:38:16 GMT+0000 (Coordinated Universal Time)
cuid: clx2692tn000609kv06beccwa
slug: easily-email-digital-products-with-stripe-and-amplify-gen-2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1717607364520/02d37e6c-7aed-48b1-ac8c-1ef4c6035e6a.png
tags: startups, aws, typescript, serverless, awsamplify, focusotter

---

I'm a huge fan of cutting out the middle-man whenever possible. Don't get me wrong, there are some niceties that come with using a 3rd-party SaaS or wrapper service, but the tradeoff usually means cutting into profits, or working within the constraints of their ecosystem.

For those reasons, in this post, I'll show how to use Stripe and AWS to create an application that allows users to buy a product and have it emailed to them!

%[https://youtu.be/88T0-C1mKFU] 

## Application Overview

All of the code for this project can be found here:

%[https://github.com/focusOtter/simple-stripe-checkout.git] 

The entry point of this application is based on a Stripe payment link that is triggered when a user clicks the "Get Your Song🎵" button.

Once a user makes a purchase, a Lambda function listens to the `checkout.session.completed` event type to perform code. To make sure only Stripe is authorized to process these events, the Stripe webhook signature is validated.

After validation, we can grab our song from Amazon S3 and email it to the customer as a file attachment. As opposed to an inline email, file attachments allow for larger file upload.

> 🗒️ This setup assumes a verified email address. If you're SES account is in the sandbox, ensure a verified email is being used during development. The YouTube video has tips on getting out of the sandbox and moving to production.

The entire application flow looks like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717608003130/e1503e66-1bbe-425d-afe6-c2810de9613b.png align="center")

## Backend Setup

Using [AWS Amplify Gen 2](https://docs.amplify.aws/), I'm able to not only create my AWS services, but also have a full TypeScript experience. This means less errors and a simpler developer experience.

### Creating the Amplify backend

In previous versions of Amplify, a backend was scaffolded and iterated on using the Amplify CLI. In Gen 2, the CLI is only used as an entry point while the iteration happens via TypeScript:

```bash
npm create amplify@latest
```

This will create both a data resource (API + database) and an auth resource (defaults to email sign in).

In this application, we're using neither, so as you can see in the repo, I deleted those respective folders.

### Creating an S3 bucket

The S3 bucket needed to contain our MP3 needs to authorize our to-be-created Lambda function with `read` access. Fortunately, doing so is around 5 lines of code.

Using the GitHub repo as reference, I have following piece of code in the `amplify/storage/resource.ts` file:

```typescript
import { defineStorage } from '@aws-amplify/backend'
import { stripeCheckoutSongFunc } from '../functions/stripe-checkout/resource'

export const storage = defineStorage({
	name: 'amplifyStripeSongCheckout',
	access: (allow) => ({
		'products/*': [allow.resource(stripeCheckoutSongFunc).to(['read'])],
	}),
})
```

### Creating the Lambda function

The Lambda function needing to serve as the webhook for Stripe is similarly simple to setup. In the `amplify/functions/stripe-checkout` file I give it an `name`, an `entry` point for my business logic, and pass in a few `environment` variables:

```typescript
import { defineFunction, secret } from '@aws-amplify/backend'

export const stripeCheckoutSongFunc = defineFunction({
	name: 'stripe-checkout-song',
	entry: './main.ts',
	environment: {
		// STRIPE_SECRET: secret('stripe-secret'),
		// STRIPE_WEBHOOK_SECRET: secret('stripe-webhook'),
		SONG_KEY: 'products/amplify-song.mp3',
		FROM_EMAIL_ADDRESS: 'mtliendo@focusotter.com',
	},
})
```

### Initial sandbox deploy

With our services scaffolded, now is a good time to perform an early deploy of our backend. Deploying early is a great way to catch problems in syntax before they cascade into bigger problems. Before doing so, we have to add these two services to our `backend` stack.

```typescript
import { stripeCheckoutSongFunc } from './functions/stripe-checkout/resource'
import { defineBackend } from '@aws-amplify/backend'
import { storage } from './storage/resource'
import { FunctionUrlAuthType } from 'aws-cdk-lib/aws-lambda'
import { PolicyStatement } from 'aws-cdk-lib/aws-iam'

const backend = defineBackend({
	storage,
	stripeCheckoutSongFunc,
})
```

In addition to adding the `storage` and `stripeCheckoutSongFunc` services that we created, we'll want to both generate a function URL that we can pass to Stripe, and provide permission for our function to use the SES service. To accomplish that, I added the following just below what's currently in that file. Once done, I can deploy my app to AWS:

```typescript
const stripeWebhookUrlObj =
	backend.stripeCheckoutSongFunc.resources.lambda.addFunctionUrl({
		authType: FunctionUrlAuthType.NONE,
	})

backend.addOutput({
	custom: {
		stripeCheckoutSongFunc: stripeWebhookUrlObj.url,
	},
})

backend.stripeCheckoutSongFunc.resources.lambda.addToRolePolicy(
	new PolicyStatement({
		actions: ['ses:SendEmail', 'ses:SendRawEmail'],
		resources: ['*'],
	})
)
```

```bash
npx ampx sandbox
```

This long-running service will continuously watch for backend changes and automatically redeploy our application when those occur.

This is great so far, but the application doesn't make use of any secrets from Stripe. Let's fix that.

### Accessing secret values from Stripe

This application needs 2 secret values: A stripe webhook secret, and the Stripe account secret. The secret key is pretty simple to gather, just make sure you don't share it!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717610599208/a76afe7e-9fa6-4353-9461-f8e7593237dc.png align="center")

The webhook secret can only be given once a webhook has been entered, fortunately, by clicking on the "Webhooks" tab in Stripe, we can do just that:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717610700460/ba5385c1-da9d-41a1-9eb2-88e4e9480a4c.png align="center")

Once complete, you'll want to grab the "signing secret" and make sure you never commit or share it!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717610780904/9291eeda-ca44-43e7-9f01-bf1be3868c38.png align="center")

All that's left is to create your product and get a URL for the payment link. This can be done by clicking the `+` icon in the top-right, or simply pressing `c`, `l` on any page in Stripe(🤯).

I setup mine like this, but feel free to setup yours however you like!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717610898956/5021aceb-43b3-443a-93f8-eaebd1b9cb4a.png align="center")

The payment link is what is put on the button on our landing page.

### Storing Secret Values

It's worth noting that the `secret` function provided by Amplify (as seen in our Lambda function resource) will take in a secret name and automatically handle grabbing the secret value. However, setting the secret in the first place is handled via the CLI:

```bash
npx ampx sandbox secret set stripe-secret

# enter the secret value:
```

I ran that command for both the `stripe-secret` and the `stripe-webhook` values. Afterwards, make sure to uncomment the environment variables in the resource file.

## Handling Backend Logic

With our resources created, and secrets added, it's time to setup the business logic for our Lambda function.

While the `main.ts` file is essentially the "glue" that puts the logic together, the logic is really divided into 3 sections:

1. Verifying the Stripe signature
    
2. Getting the song from S3
    
3. Sending the email as an attachment
    

### Verifying a Stripe signature

As shown in `amplify/functions/stripe-checkout/helpers/verifyStripeWebhook.ts`, this function is generic and ready to be reused across applications:

```typescript
import Stripe from 'stripe'
import { env } from '$amplify/env/stripe-checkout-song'

type Event = {
	headers: {
		'stripe-signature': string
	}
	body: string
}
export const verifyWebhookSig = async (event: Event) => {
	const stripe = new Stripe(env.STRIPE_SECRET)
	const sig = event.headers['stripe-signature']
	try {
		const stripeEvent = stripe.webhooks.constructEvent(
			event.body,
			sig,
			env.STRIPE_WEBHOOK_SECRET
		)
		return stripeEvent
	} catch (err) {
		console.log('uh oh', err)
		throw Error('Invalid signature')
	}
}
```

It simply expects the Lambda `event` object and from there pulls off the headers while using the Stripe secret we created to perform the validation.

### Getting an Object from S3

This is another file that I templated so that it can be easily reused across projects:

```typescript
import { GetObjectCommand, GetObjectCommandOutput } from '@aws-sdk/client-s3'
import { S3Client } from '@aws-sdk/client-s3'
const s3Client = new S3Client()

export async function getObjectFromS3({
	bucketName,
	objKey,
}: {
	bucketName: string
	objKey: string
}) {
	let res: GetObjectCommandOutput | undefined
	try {
		res = await s3Client.send(
			new GetObjectCommand({ Bucket: bucketName, Key: objKey })
		)
	} catch (e) {
		console.log('error getting object', e)
	}

	return res
}
```

Essentially, you pass it the name of the S3 bucket, and the key of the object. Both of these items are stored as environment variables, making this a simple solution for grabbing any item from S3!

### Sending the Email as an attachment with SES

Lastly, and still in a reusable fashion, this we want the ability to send an email with both text, and an attachment. As opposed to sending an `html` formatted email, this involves dropping down to a `raw` email format:

```typescript
import { SESv2Client, SendEmailCommand } from '@aws-sdk/client-sesv2'
import { env } from '$amplify/env/stripe-checkout-song'
const sesClient = new SESv2Client()

export const sendSESEmailWithAttachment = async (
	fileBuffer: Buffer,
	toEmailAddresses: string[],
	fileContentType: string,
	fileName: string
) => {
	const myEmail = env.FROM_EMAIL_ADDRESS
	const rawMessage = `From: ${myEmail}
To: ${toEmailAddresses.join(', ')}
Subject: Email with MP3 Attachment
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="boundary_string"

--boundary_string
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 7bit

This email has an MP3 attachment.

--boundary_string
Content-Type: ${fileContentType}
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="${fileName}"

${fileBuffer.toString('base64')}
--boundary_string--`

	const sendEmailParams = {
		Content: {
			Raw: {
				Data: Buffer.from(rawMessage),
			},
		},
	}

	const sendEmailCommand = new SendEmailCommand(sendEmailParams)

	try {
		const result = await sesClient.send(sendEmailCommand)
		console.log(`Email sent successfully. Message ID: ${result.MessageId}`)
	} catch (err) {
		console.error('Error sending email:', err)
	}
}
```

> 🗒️ It's ugly. But I think it's just the long string that makes it look bad.

### Conclusion

With those 3 pieces in place, the `amplify/functions/stripe-checkout/main.ts` file simply makes use of them to create our application flow.

If wanting to clean up and delete the resources, running `npx ampx sandbox delete` will do just that (or cancelling out of the running sandbox will trigger a prompt). Also worth noting is that any secrets we added in the sandbox must be manually removed with `npx ampx sandbox secret remove [secret-name]`.

What I love about this project is that it can be reused anytime you want to sell *any* digital item online! Making it a super fast an easy solution!

Hope this helped and if you enjoyed this piece let me know by dropping a comment!

Until next time, Happy Coding 🦦