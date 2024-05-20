---
title: "AWS Amplify in 2024 is not the Amplify you grew up with"
seoTitle: "AWS Amplify 2024: A New Era"
seoDescription: "Discover the evolution of AWS Amplify in 2024, featuring enhanced flexibility, TypeScript integration, and improved developer experience"
datePublished: Mon May 20 2024 18:15:43 GMT+0000 (Coordinated Universal Time)
cuid: clwfaegar000i09l93k396ejc
slug: aws-amplify-in-2024-is-not-the-amplify-you-grew-up-with
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1716228841311/fb992f21-9cdb-43de-8d5b-81aa8bae70fb.png
tags: fullstack, typescript, awsamplify, focusotter

---

In 2019 I got into learning AWS as a software engineer at John Deere. I took a course on Frontend Masters taught by [Steve Kinney](https://twitter.com/stevekinney) about how frontend developers could use a tool called AWS Mobile Hub.

Through the lens of Mobile Hub, I learned the foundations of AWS and serverless. However, the tool was buggy, soon deprecated, and shortly after, [AWS Amplify](https://docs.amplify.aws/) came to be.

Through this tool, I didn't have to setup Cloudfront or S3. As a frontend/fullstack developer, I wanted to use something like what [Zeit (now Vercel)](https://vercel.com/blog/zeit-is-now-vercel), or Netlify had to offer. AWS Amplify Hosting allowed that.

%[https://www.youtube.com/watch?v=ucVK6Z55PZY&t=1s] 

In addition, the Amplify CLI meant I didn't have to *know* AWS. I could just use the CLI. From easy use cases where I wanted an S3 bucket:

```bash
amplify add storage
```

To more complex integrations likes creating a GraphQL API with a database and authentication rules:

```bash
amplify add api
```

```graphql
type Todo @model @auth(rules: [{allow: private}]) {
    id: ID!
    name: String!
}
```

I felt empowered to build *anything*.

Through the [AWS Community Builders](https://aws.amazon.com/developer/community/community-builders/) program, I was able to get direct feedback to and from the AWS Amplify team until the Spring of 2021 when I was hired as a developer advocate on the AWS Amplify team.

%[https://twitter.com/focusotter/status/1398046515189399554] 

## Learning How to DA and Icarus

As a developer advocate for the Amplify team I loved challenging myself to see how I could build out seemingly-complex applications using the Amplify CLI. I wasn't interested in todo-style or simple CRUD-type applications.

I instead wanted to build social media sites like Instagram, apps that have to pass data to one another like Starbucks, and multi-tenant SaaS applications like Slack.

I would often hit a roadblock, talk to the engineering team, find a workaround, and keep going forward. It wasn't that what I was doing couldn't be done, but there were times when the solution was so far-fetched that I couldn't believe that was the best form of DX we could have.

Naturally, I gave feedback and as expected, the Amplify team delivered. In 2022, the team announced [*extensibility* with AWS Amplify](https://aws.amazon.com/blogs/mobile/use-aws-cdk-v2-with-the-aws-amplify-cli-extensibility-features-beta/). This feature allowed customers to leverage the AWS CDK to provision services that the Amplify CLI didn't support.

> 🗒️ This was absolutely the right move. AWS has over 200 services. It'd be ridiculous to have a CLI tool that had all of those services baked in!

Through extensibility, I felt I had better control. The ceiling to what I could build as a sole developer was certainly raised and to this day, I'd say the majority of solo devs could use this and be profitable in both revenue and usage.

The problem was that I -- a solo developer -- had to know:

* A frontend framework
    
* AWS Amplify's way of doing things (CLI, libraries, and patterns)
    
* AWS CDK
    
* TypeScript
    

Not just that but what often gets overlooked, is knowing *when* to use those tools.

Looking back, I put out a lot of tutorials that talked about how to build Complicated App™️ and the solution was filled with various workarounds to what should have been a simple process.

I forgot that most developers don't have direct access to an AWS engineering team. That most don't get a perpetually free AWS account to work in. And that most don't know when an enhanced experience is just around the corner so they shouldn't invest in hackySolution™️.

> 🗒️ One of my greatest realizations at AWS was when the director of our org spoke on the value of DA's by saying, "They \[engineers\] often will perform the workarounds without realizing that they're workarounds."
> 
> Nothing against our amazing engineers of course. His statement was simply about the impact of DA's being an important link the chain. Between our customers and product.

I had become [Icarus](https://en.wikipedia.org/wiki/Icarus). Through my various solutions and content pieces, I flew high without realizing I was advocating for the product instead of the developers I was trying to help. And with each YouTube video that showed what was possible, my wings melted in the sun of what I thought was good enough.

## Switching teams and going in on AWS

Soon after, my personal interests shifted towards learning AWS services and I switched to the AWS AppSync team.

%[https://twitter.com/focusotter/status/1570574512395255808] 

This is where I learned how to use the AWS CDK. I no longer had the luxury (and it is a luxury!) of letting Amplify do all the work in creating an API, adding authentication, or configuring a database. I also had to learn how to connect my frontend (NextJS) with my CDK backend.

It took months of learning and I'm fortunate my manager trusted me to figure out a DX that met my high bar.

I finally settled on a solution that worked well for the majority of small to medium-sized applications, and provided enough inspiration for medium to large applications.

%[https://github.com/focusOtter/fullstack-nextjs-cdk-starter] 

My pitch to fullstack cloud developers was, "I'm trying to make your life 15% harder for unlimited flexibility".

Looking back, that pitch was pretty awful 😅

For context, this was when Supabase, SST, Vercel, and tRPC captivated crowds on what an amazing DX could look like. My solution works -- and many customers find success/inspiration from it, but it was a heavy sell for those outside of the niche.

So there I was. A DA no longer directly tied to the Amplify team or it's crowd of frontend developers. However, in building out fullstack applications, I wanted more flexibility than what Amplify had at the time, and I wanted frontend developers to have an easier onboarding than what the AWS CDK offers.

> 🗒️ I hate to keep stressing this, but the Amplify CLI can get you *really* far. And the AWS CDK is my favorite IaC tool.
> 
> %[https://www.youtube.com/playlist?list=PLiLHsu3XjGfMgGxbjK59tgGt0NYywpcDd] 

## Enter Amplify Gen2

The amazing thing about AWS is that our teams (the ones I interact with) are structured like a startup -- albeit a well funded one. What I mean is that we never really rest on *good enough*. We are always challenging each other and [Thinking Big](https://www.amazon.jobs/content/en/our-workplace/leadership-principles).

So when my colleague and mentor [René Brandel](https://twitter.com/renebrandel) asked for my feedback on an API concept for Amplify, I took a look not expecting much of it.

One of our personal rules is that we don't do "company speak" or put fluff between the lines. We just say what we feel. So if something sucks, then we say it sucks and not stumble for the politically correct way of saying it. I encourage everyone to have a friend like this at work.

His solution sucked.

```typescript
import { type ClientSchema, a, defineData } from '@aws-amplify/backend'

// a wordsearch API
const schema = a.schema({
    WordSearch: a
	    .model({ // a database
			id: a.id().required(),
			name: a.string().required(),
			columns: a.integer().required(),
			rows: a.integer().required(),
			wordBank: a.string().array().required(),
		})
        .authorization([a.allow.owner()]), // an auth rule
})
```

It was all TypeScript based. There was no clear separation for when the frontend started and the backend began. The API was completely changed from what I was familiar with, and his presentation was filled with "what ifs", and "now imagine if you could"s.

I gave him my feedback on there being too much magic, how it changed how things were done, and how the problems customers faced when building applications would be the same and moved on with my day.

A few days later we met again to cover some minor changes he made. I remember telling him, "The API just doesn't look or *feel* like GraphQL". I'll never forget his reply:

**"Who said this is GraphQL?"**

Granted, it ***was*** GraphQL -- AppSync specifically. I knew that. But I only knew it because I was close to the product/engineering team. I realized I was being an Icarus -- flying high advocating for a product instead of what fullstack developers want.

> 🗒️ In hindsight, his idea didn't suck. I just couldn't hit pause on what I was familiar with long enough to see the benefits of what was in front of me.

He sent an early build for me to play around with and provide more feedback on, and this was in fact our cycle for the next several months.

In the end, thanks to him and the his team's engineering efforts, Amplify Gen 2 (preview) was announced during the re:invent 2023 season.

%[https://aws.amazon.com/blogs/mobile/introducing-amplify-gen2/] 

Admittedly, it wasn't until recently that I began trying to move my NextJS + CDK applications over to Amplify Gen 2. Most recently, [this](https://github.com/focusOtter/sample-wordsearch) wordsearch app built with the CDK being ported over to this one using Gen 2 :

%[https://github.com/focusOtter/wordsearch-gen2] 

## Amplify Gen 2 Takeaways

The rule I have with René is the same rule I have with all of you -- no fluff.

Amplify Gen 2 is nice...not perfect...but *very* nice!

### Frontend Developers

From a frontend DX perspective, everything is in TypeScript. If you want to setup authentication, you do so using TypeScript. If you want to create an data resource (API), it's done in TypeScript as shown above.

This end-to-end type-safety is visible throughout the application. In a NextJS application for example, all of your CRUD operations are now fully-typed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712725690739/446d9d13-b291-474a-bfb5-65aa76fdd485.png align="center")

This means less context-switching and mental overhead. In 2024, it's all but expected, but a nice DX with nonetheless.

### Backend Developers

For backend developers, the DX is even better IMO.

This may have flown under the radar for most people, but a big PR was merged in the CDK last year that the Amplify team was able to capitalize on: Making GraphQL schemas hot-swappable.

%[https://github.com/aws/aws-cdk/pull/27197] 

This means whenever a schema is changed, the running process would redeploy those assets to AWS. In relatable terms, this means when a customer runs the following command:

```bash
npx amplify sandbox
```

Then a stack (completely separated from your team/org) will get deployed and this process will watch for changes. When Amplify detects a change to your data file, for example, then it will redeploy your API, and run codegen on your behalf so that your get live feedback across your application.

> 🗒️ Much like the `cdk watch` command, sandbox environments are used during development.

---

Simple CRUD apps are great to showcase because they cover common scenarios. However, it's important to remember real-world applications are often much more involved. For example, adding AI generated content often needs another service or custom ability.

In the CDK, a custom mutation that called out to [Amazon Bedrock](https://aws.amazon.com/bedrock/?gclid=Cj0KCQjwztOwBhD7ARIsAPDKnkB6On-TOqL6nKQ-fSdIA1Q_dNgEdNdDTX7Ok12EshM0CCfz5la2g1kaAvwEEALw_wcB&trk=0eaabb80-ee46-4e73-94ae-368ffb759b62&sc_channel=ps&ef_id=Cj0KCQjwztOwBhD7ARIsAPDKnkB6On-TOqL6nKQ-fSdIA1Q_dNgEdNdDTX7Ok12EshM0CCfz5la2g1kaAvwEEALw_wcB:G:s&s_kwcid=AL!4422!3!692006004688!p!!g!!bedrock!21048268554!159639952935) would look like this:

```graphql
type Mutation {
	generateWordSearchWords(theme: String!): String!
}
```

While simple at first, it's important to remember you still have to create the HTTP datasource:

```typescript
// add bedrock as a datasource
const bedrockDataSource = api.addHttpDataSource(
	'bedrockDS',
	'https://bedrock-runtime.us-east-1.amazonaws.com',
	{
		authorizationConfig: {
			signingRegion: 'us-east-1',
			signingServiceName: 'bedrock',
		},
	}
)
```

Then give the datasource permissions to invoke the foundation model (Claude V2 in this case):

```typescript

// Allow datasource to invoke claude
bedrockDataSource.grantPrincipal.addToPrincipalPolicy(
	new PolicyStatement({
		resources: [
			'arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-v2',
		],
		actions: ['bedrock:InvokeModel'],
	})
)
```

Create and configure the custom resolver for your API:

```typescript
// create a unit resolver that connects to bedrock and returns a string of words
const generateWordSearchWordsResolver = api.addResolver(
	'generateWordSearchWordsResolver',
	{
		dataSource: bedrockDataSource,
		typeName: 'Mutation',
		fieldName: 'generateWordSearchWords',
		code: Code.fromAsset(path.join(__dirname, 'generateWordSearchWords.js')),
		runtime: FunctionRuntime.JS_1_0_0,
	}
)
```

And only then can you write your business logic for your application:

```typescript
export function request(ctx) {
	const assistant = ``
	const theme = ctx.args.theme
	const prompt = `Generate 10 words related to ${theme}. Put the words in an array. For example, if the theme was "animals", you would return ["monkey", "tiger", "bear", "lion", "gorilla", "bird", "penguin", "dolphin", "wolf", "dog"].`

	return {
		resourcePath: '/model/anthropic.claude-v2/invoke',
		method: 'POST',
		params: {
			headers: {
				'Content-Type': 'application/json',
			},
			body: {
				prompt: `\n\nHuman:${prompt}\n\nAssistant:${assistant}`,
				max_tokens_to_sample: 300,
				temperature: 0.5,
				top_k: 250,
				top_p: 1,
				stop_sequences: ['\\n\\nHuman:'],
			},
		},
	}
}

export function response(ctx) {
	console.log('the bedrock response', ctx.result.body)
	return ctx.result.body
}
```

In a TypeScript environment powered by Amplify Gen 2, this becomes much easier:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712727156756/af553ee6-b76f-4219-afde-3a2a78d1d435.png align="center")

> 🗒️ You still have to [assign the correct permissions](https://github.com/focusOtter/wordsearch-gen2/blob/main/amplify/backend.ts), and your custom business logic is still up to you to decide.

What I love about this model isn't just the type-safety, but that I am able to copy over the permissions from the CDK project because at the end of the day, it's all TypeScript.

I love frontend frameworks that use APIs and concepts from browsers and push developers to [MDN](https://developer.mozilla.org/en-US/). Amplify does the same by taking care of the expected and telling customers that want additional services to view the [CDK docs](https://docs.aws.amazon.com/cdk/api/v2/).

### Continuity

Amplify Gen 2 in your CLI and code editor also brings enhancements to the Hosting platform as well! For starters, it's now possible to have [wildcard subdomains](https://aws.amazon.com/blogs/mobile/wildcard-subdomains-for-multi-tenant-apps-on-aws-amplify-hosting/) on your domain name. This, along with enhanced server-side rendering support means it's now possible to have true multi-tenant applications hosted on Amplify.

Additionally, because of the closer connection to the CDK, Amplify will `cdk bootstrap` your account/region on your behalf!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712728250591/e9b82fe4-2f68-4e9c-9606-41e5d03c08c2.png align="center")

Also, for teams that may incrementally adopt Amplify, have a large team, or have an existing CDK backend, it's possible to keep the applications separated while taking advantage of all the Amplify goodness.

%[https://docs.amplify.aws/gen2/deploy-and-host/fullstack-branching/mono-and-multi-repos/] 

## Conclusion

A major part of developer advocacy is what [my manager](https://twitter.com/BillFine) calls "selling what's on the truck". Meaning there are times when we know a big feature release is coming, but we have to help the folks that are trying to build *today*.

I'm happy with the current state of Amplify Gen 2, but any DX will need constant refinement. Myself and the team will continue to make improvements -- but we can only do it with your feedback.

My hope is that I gave you a glimpse into what it took to get t o this point, and inspired you to build with [Amplify Gen 2](https://docs.amplify.aws/gen2/) on your next project!

As always, let me know if you have any questions by hitting me up on [X](https://twitter.com/focusotter) or [LinkedIn](https://www.linkedin.com/in/focusotter/).

Until next time,

Happy Coding 🦦