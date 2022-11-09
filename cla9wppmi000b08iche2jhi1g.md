# The case for fullstack teams having dedicated frontends and backends

> 🎉 This post has been officially written and [published to the AWS Mobile Blog](https://aws.amazon.com/blogs/mobile/secure-aws-appsync-with-api-keys-using-the-aws-cdk/) as a getting started guide!
> 
> Feel free to read that to get a walkthrough or follow along as this post will explain the important bits!

## Overview

In the past, I've shared how teams can quickly get their applications from idea to production using the AWS Amplify Framework. And for good reason--Amplify instills a long list of best practices and automation so that policy management, scaling, and provisioning can safely fall into the role of _undifferentiated-heavy-lifting._

However, while it's possible to use the entire suite of Amplify features to build your application, Amplify can also be used piece-by-piece (or _à la carte_ as I like to say 😅).

Thinking of the Amplify framework as a multi-headed dragon is a great way to tackle your next set of application requirements!

![hydra going against a swordsman. The base of the hydra is AWS Amplify while the heads represent the UI components, Service libraries, Studio, CLI, and Hosting. The swordsman represents your application requirements](https://pbs.twimg.com/media/FhHQZLcXoAMD1z0?format=jpg&name=small align="center")

This allows Amplify to remain flexible and adapt to a lot of different use cases and team setups.

In this post, we'll see how frontend teams can use the [Amplify JavaScript libraries](https://docs.amplify.aws/lib/q/platform/js/) in their repo, and a backend team can create their AWS services using the AWS CDK.

## The case for split teams

%[https://www.youtube.com/watch?v=GBPDeic5fPE&t=8s] 

The larger the project, the more difficult it is to have one set of developers running the end-to-end stack.

Instead, many startups and enterprises adopt a strategy where frontend developers manage that entire stack, and backend developers manage the deployment pipeline and service provisioning.

> 🗒️ The frontend team **integrates** with the backend, but is not in charge of provisioning it.

This allows each team to scale independently and team members carry less cognitive load.

This doesn't mean the two teams will never communicate. As the saying goes, "The right hand needs to know what the left hand is doing." This means those discussions can take place in weekly meetings and such.

## TypeScript enables cross-team communication

![Thumbnail for version as of 17:33, 6 May 2021](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4c/Typescript_logo_2020.svg/120px-Typescript_logo_2020.svg.png?20210506173343 align="left")

To assist in effective communication, agreeing on a widely understood programming language will go a long way. Fortunately, modern application development for both frontend and backend teams can take advantage of TypeScript.

NextJS is a react framework developed by Vercel and with the release of NextJS version 13 has opted to make TypeScript the default programming language (over JavaScript).

In addition, the AWS CDK has first-class support for TypeScript by simply specifying it as the language when scaffolding an application:

```bash
npx aws-cdk init my-project --language typescript
```

## GraphQL as a data hub for your API contract

%[https://snappify.io/view/30dd5f9c-3fea-42b7-a293-999b3828f1b7] 

> 🗒️ Hover on the embed above and hover over each code editor to copy the code.

AWS AppSync is a managed GraphQL service. So there are no servers to manage and it includes built-in integrations with services such as AWS Cognito, Amazon DynamoDB, and more. Another compelling feature is its out-of-the-box WebSocket support.

However, an often overlooked feature stems from the fact that it's GraphQL: Teams now have a binding contract on what fields are available on an API and the types that they can assume.

With this, frontend teams get quick feedback on what is considered valid data before the request is even submitted.

## Conclusion

This post is the first in a series where we'll dive deeper into how teams can take advantage of this stack and the benefits it provides when factoring in speed, maintainability, and collaboration.

If there are certain parts you'd like to be emphasized, feel free to leave a comment on engage with me on any of my social channels to stay in touch!

Until next time, stay creative and happy building 🦦