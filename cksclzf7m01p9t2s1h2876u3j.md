## Intro to AWS Lambda

#### Cover image by Thomas Couillard

---

Hello Again 👋🏽

Let’s get the meme out of the way: Serverless services still have a server. The naming is just used to imply that it’s a server that you no longer have to manage.

This means even though there is a server involved at some point, we don’t have to mess with it. So instead of patching OS updates, setting up scaling rules, etc. we can focus on our business logic and deploy a function.

That’s right. A lambda is just a function that lives on AWS. You put your code in it, and some kind of event calls the function.

## 🗒️  Need to know terms

**Serverless**: A managed service that automatically scales based on your traffic where billing is based on usage.

**Lambda**: A type of serverless service that allows us to write code that executes in a function that is called by some type of event.

**Trigger**: An event that invokes a lambda

![Serverless doesn't mean just lambda](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995696710/7Qdqmc1jc.png)

It’s worth reiterating, that a Lambda is event-driven. That event can be anything from someone coming to your website, to a new user signing up, to some scheduled interval that runs every 5 minutes or so. There’s a lot of flexibility when it comes to what triggers the event.

## 🏗️  Creating our first Lambda

Using a step-by-step process, let’s go ahead and create our lambda function:

1. Log into your AWS account, search for Lambda using the search bar at the top, and select the first option
2. Click the “Create function” button
3. Starting from scratch, name your function pets-get-func
4. Click the “Create function” button

🎉 That’s it! Congratulations on creating your Lambda function!

![Newly created lambda function](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995698544/QpE5zGaLu.png)

## 🟡  Reviewing the Lambda Dashboard

With our function created, let’s take a moment to explore what we’re now presented with.

In the above screenshot, the top portion is known as the “Function Overview”. Since Lambda’s are event-driven, we can configure a trigger as well as the destination for the response.

Further down, we see the “Code Source” panel. While it’s fine to write code in this built-in editor (as we’ll soon do), for bundles of code larger than 50mb, we can also use the “Upload from” dropdown.

In a typical flow however, you’ll most likely use the CLI or another tool to push your function code to AWS.

## 💫  Running our lambda function

As you can see from the code editor, a lambda function can really be as simple as something that just returns some data. A couple of things to note: 

1. Functions are stateless. So you’re not going to want to put any variables outside your function handler. 

2. The event argument contains information relating to whatever triggered the function. Since many things can do just that, this argument is contextual.

Go ahead and update your function so that it looks like the screenshot below and click “Deploy”

![Pets array in a lambda function](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995700440/orVZgybj6.png)

Next up is to create a test event for our event variable: 

1. Click Test 
2. Name your event “exampleEvent” 
3. click Create at the bottom.
Now that we created a test event, we can click the Test button one more time and our function will run.

![pets array response](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995702328/0Y1PaD5ux.png)

🎉 Congratulations on running your lambda function!

While viewing the execution results, we can see the response body as well as meta information such as how long the function took to run. Execution time is something to be mindful of because it relates to how you’ll be billed.

Speaking of billing, let’s wrap this up with a chat on pricing.

## 💸  Pricing

A trend with serverless services is that you’ll typically pay for two things: 

**compute**: How long it takes for something to run 

**requests**: How many times your service is used

Using today's default lambda in us-east-2, the pricing is as follows: 

**Compute**: $0.0000000021 per/ms 

**Requests**: $0.20 per 1M requests

👀  If those penny-fractions get you worried, fear not, AWS also provides a free amount of usage each month:
 
**1M free requests per month and 400,000 GB-seconds of compute time per month**

https://aws.amazon.com/lambda/pricing/

---

As we get along further in understanding some of the core AWS services, I do want to make sure your account is set up correctly to avoid any unexpected charges 😅

So for those interested, I created a video you can checkout to make sure you're covered

{% youtube UnqxiSJEZAk %}