---
title: "Unleash Your Full-Stack Potential: Why Frontend Developers Should Embrace AWS"
datePublished: Fri Mar 17 2023 02:28:02 GMT+0000 (Coordinated Universal Time)
cuid: clfbx7fa100070alb3ml9bch0
slug: unleash-your-full-stack-potential-why-frontend-developers-should-embrace-aws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679019489512/69c9d4e4-4932-409f-8d5f-0ae924b1052c.png
tags: startups, aws, full-stack, serverless, nextjs

---

If there's a particular community I love, it's the indie hacker / small startup / SaaS communities--the ones building beautiful frontend applications powered by a suite of backend services.

They're usually a group of 1-5 individuals with skills that vary across the development stack. From what I've noticed lately, the trend seems to be that individuals are increasingly frontend developers with a knack for backend development.

What's interesting about this group is their tenacity. They build, get stuck, problem-solve, and listen to their customers to figure out what's next in the pipeline. If I had to make a few more generalizations it's that they favor speed, a good DX, and are not afraid of delivering something less than perfect if it still means delivering *something*.

This group is constantly seeking ways to optimize its workflow and create powerful, scalable applications that delight customers. While there are lots of frontend frameworks that help with this, some more than others are standing out on the frontend: **React/NextJS, Stripe, TailwindCSS, and TypeScript**.

The backend is where I see the most decision paralysis. Rightfully so: authN/authZ, file storage, database flexibility, and an API are common to every real-world app, and not always easy to migrate from when needing to. This often leads to picking what is easiest to get started with.

However, while many devs/founders consider the day 1 experience or the first 100 customers, it's important to think about day 100 or how things look when you have 1000 customers. When set up correctly, you sleep all the same.

There are lots of players and opinions in this space, but in this post, we'll discuss why embracing AWS, specifically a core selection of tools and services, can not only unlock your full-stack potential but also elevate your application development so that your customers benefit from it.

**\[image: AWS logo and NextJS logo side by side, illustrating the combination of the two technologies\]**

So, why should you build your next application on AWS? Here are some compelling reasons and services that you should consider:

### Slightly More Technical Overhead, But Worth It

![Vintage Balance Scale Scales of Justice by JoieDeCleve on Etsy](https://imgs.search.brave.com/rvbw2t1OTukWRUOguEmg_j2aykJ_tU2OfgESHl0WZXE/rs:fit:1200:1200:1/g:ce/aHR0cDovL2ltZzEu/ZXRzeXN0YXRpYy5j/b20vMDAwLzAvNTg4/MDE4NC9pbF9mdWxs/eGZ1bGwuMTc2NTc1/MTg1LmpwZw align="left")

Adopting this AWS-based stack might come with slightly more technical overhead compared to your current workflow. However, the numerous benefits it offers far outweigh this factor. By diving into the AWS ecosystem, you'll gain access to a world of powerful services and tools, enabling you to build highly-scalable, cost-effective applications with ease. It's important to note that you developing on AWS doesn't mean using all of the 200+ services they offer. Instead, it's about picking the right service for your particular need.

The added technical overhead will be a valuable investment in your skillset, making you a more versatile developer. As you become proficient in AWS services, you'll find that the initial learning curve pays off in the long run, helping you create more sophisticated applications and positioning you for greater opportunities in the tech industry.

### Cost-effective compared to 3rd-party low-code tools

\[image: Graph comparing the cost of AWS vs. low-code tools with monthly subscriptions\]

While low-code tools with monthly subscriptions may seem tempting due to how easy they are to get started, AWS offers a more cost-effective and flexible solution when making the most of serverless/managed services. With those, you only pay for the resources you consume, allowing you to optimize costs as your application scales.

That isn't to say low-code tools don't have their place or benefit. Rather, it's about retaining ownership, and thus having more control over what is happening in your application. Recall that AWS is an ecosystem of services. Having services that know how to talk to and integrate is a huge benefit. In the end, low-code tools remain extremely viable (AWS has several), and when considering a 3rd-party tool, there is of "Better-Together" workflow as well.

%[https://twitter.com/focusotter/status/1628053794691899393?s=20] 

### Bias Towards Serverless & Managed Services

Touched on above, but focusing on serverless and managed services offered by AWS enables you to build scalable, cost-effective applications without managing servers. I think it seems fair: If I'm not using a service, then don't charge me.

AWS offers a range of serverless services like AWS Lambda, Amazon API Gateway, and Amazon DynamoDB, allowing you to build and deploy your applications quickly, with built-in scalability and cost optimization.

As an indie creator or SaaS founder, cost and durability are likely your biggest factors. This is where a managed service comes into play. *Managed* means less time provisioning throughput, less code you have to write, and more time focusing on your product and customers.

When I tell folks that, they often think this means more Lambda functions. I'm saying *fewer* Lambda functions. A large benefit of serverless, fully-managed services on AWS is there are often direct integrations available so they can talk to one another. Less Lambda functions mean fewer cold starts.

AWS StepFunctions, EventBridge Pipes, and AWS AppSync are great examples of this.

%[https://twitter.com/focusotter/status/1614040726320418816?s=20] 

### Increased earning potential

I've been speaking from the perspective of an indie creator or SaaS founder, and while that's intentional, I wanted to add this blurb before getting into the stack itself.

Maybe the product is *you* and you're consulting or offering time-based services. Learning AWS can significantly boost your earning potential. As one of the most in-demand cloud platforms, having AWS skills on your resume can open up lucrative opportunities in the job market. You don't need to have every certification, or even be some AWS guru. You can still position yourself as an invaluable full-stack developer, ready to tackle complex projects and contribute to the growth of startups and SaaS businesses.

---

Ok, so if you've made it this far, you may be wondering what a fullstack project on AWS looks like and of the 200+ services, which ones are core for most SaaS apps and indie creators.

Let's discuss it!

### Modern Frontend Development with NextJS and Tailwind

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679004658458/632c4f73-5613-402a-b32f-c2b1e15412c5.png align="center")

> With almost half of all respondents in the 2022 State of JS survey using NextJS and a 90% retention rate, NextJS has become the staple when it comes to application development.

Embracing modern frontend development means adopting powerful tools like NextJS and Tailwind CSS. NextJS, a popular framework built on top of React, brings numerous benefits, such as built-in TypeScript support, which enables robust type-checking and code autocompletion, and file-based routing for effortless page creation and navigation. With its hybrid rendering, you choose between server-side rendering (SSR), static site generation (SSG), or client-side rendering, depending on your application's needs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679004998489/e6b724f9-9d7b-4ebe-8bf4-b617bc192c4e.png align="center")

> With 46% of the State of CSS survey respondents using Tailwind, and the highest retention rate (79%) of CSS frameworks, Tailwind should be your default CSS framework when building new applications.

Tailwind CSS is a utility-first CSS framework that streamlines the styling process, making it faster and more efficient. It promotes consistency, maintainability, and readability, allowing you to create responsive, modern designs without writing custom CSS classes. By [combining NextJS and Tailwind CSS](https://tailwindcss.com/docs/guides/nextjs), you can build high-performance, visually appealing applications while benefiting from an enhanced development experience that keeps you focused on what truly matters: delivering innovative solutions for your customers.

### Low Barrier-to-Entry with AWS CDK and TypeScript

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679005525077/685c3714-64e9-4341-8fed-8bd221640372.png align="center")

> While not showcasing every option available, the above image shows everything required to create a database and an S3 bucket that stores files.

As a TypeScript enthusiast, you'll love working with the AWS Cloud Development Kit (CDK). Before, creating your AWS Services by clicking through the AWS Console and writing down the steps you took so you didn't forget them next time. When scripting came along, it was either writing your own Bash scripts to use the AWS CLI or writing verbose templates using AWS CloudFormation.

> If you haven't used CloudFormation before, just know defining a database like the above was with [a YAML file](https://s3.us-west-2.amazonaws.com/cloudformation-templates-us-west-2/DynamoDB_Table.template) several times longer.

With the AWS CDK, you can create, test, and deploy AWS resources a lot easier, streamlining your development process and taking advantage of the full power of AWS services since it exists as a wrapper around CloudFormation.

> The AWS CDK ranks 3rd when it comes to building out fully serverless applications, has the highest retention (84%), and has the best positive/negative split.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679007451045/bdc328c3-51de-486c-8807-bebe3df3bf8d.png align="center")

### Simple Frontend-Backend Integration with AWS Amplify

AWS Amplify is a suite of services that include a CLI, hosting platform, UI Component Library and more. In particular, the Amplify JavaScript libraries provide a seamless way to connect your frontend and backend, offering pre-built UI components, authentication, and storage solutions.

As an example, developers often have to figure out an easy way to pass a large file from the frontend to some backend storage service. Using the Amplify libraries, files can several GB large and can be sent as a multi-part upload, paused, resumed and canceled.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679012559797/493638af-0c6b-46db-b5a7-9183ee2186e5.png align="center")

### Scalable, Real-Time GraphQL API with AWS AppSync

As a frontend developer, you're probably familiar with the benefits of GraphQL, offering a flexible and efficient way to fetch data.

AWS AppSync is a fully managed GraphQL service. You supply it with a schema, give it data sources, and tell it how to connect to the data sources. As more requests come in, the service scales appropriately. If you've ever had [Taco Bell delivered to you](https://aws.amazon.com/solutions/case-studies/taco-bell/), [watched a show on HBO Max](https://aws.amazon.com/blogs/media/watch-now-hbo-max-uses-canaries-on-aws-for-outside-in-validation/), [saw stats on a live Ferrari F1 race](https://www.youtube.com/watch?v=1qYBEcG-fV4&t=1s), or [purchased a ticket from Ticketmaster](https://twitter.com/AWSstartups/status/1040844414627864576), then you've already interacted with AppSync.

> As a managed GraphQL service, there isn't a GrapQL server to set up. Also, AppSync doesn't have its own client library and instead integrates with clients as part of the Amplify libraries package. This makes it hard to evaluate when compared to other GraphQL offerings.
> 
> However, if interested in a comparison, check out the [State of GraphQL survey](https://2022.stateofgraphql.com/en-US/).

To resolve the data, you write JavaScript (native TypeScript support coming soon). Note that this isn't in a server, or even a serverless function like Lambda, this is a mapping that tells AppSync how to fetch the data. For example, the following is how a user would grab an item from DynamoDB:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679014855591/007ba567-2808-4c6d-8bd6-5236569ed62d.png align="center")

To emphasize, when a user on the frontend makes a request, that request goes to AppSync, which then, gets the data directly from DynamoDB. No connections to manage, to extra service to invoke. This is the power of having various resources as part of the same ecosystem of services.

To go in the other direction, by specifying the `@aws_cognito_user_pools` directive on your schema, AppSync will automatically inspect the JWT of the request to make sure the user is authenticated with Amazon Cognito before allowing the request.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679015109437/1f92d8a5-0399-4fbd-866d-1e12d141e610.png align="center")

Indie creators and SaaS founders of all sizes often have changing requirements and unforeseen access patterns to account for. AppSync's flexible schema model means less stress and more innovation.

I've written *a lot* about AppSync on this blog and [the official AWS blog](https://aws.amazon.com/blogs/mobile/category/mobile-services/aws-appsync/page/2/), so feel free to check out that content if interested in a deeper discussion!

---

## Conclusion

Adopting AWS for your application development brings a lot of benefits, including a more streamlined and efficient development process. By leveraging the AWS CDK for TypeScript, serverless and managed services, AWS AppSync for GraphQL, and AWS Amplify for frontend-backend integration, you can fully take advantage of your full-stack potential and build innovative, scalable applications.

This is the first post in a series where I'll walk you through building a full-stack application using the tools and services we discussed. I'll show you how to get started, from setting up your AWS environment to deploying your NextJS application backed by core AWS services. By the end, you'll have the confidence and experience to scaffold out your next great idea on AWS,

See you there!

\- 🦦