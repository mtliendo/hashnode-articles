## Serverless Contact Form Using AWS Amplify

When building out applications, sending email is a frequently needed feature. In addition, it's likely that the data sent in the email needs to be stored in a database for record keeping, analytics, or additional processing.

AWS provides a range of services that help make setting up an API, database, and email transport quick, and secure. Specifically, [AWS Amplify](https://docs.amplify.aws/) provides a lot of the functionality we'll need out of the box, and [Amazon SES](https://aws.amazon.com/ses/) will send emails on our behalf.

## Overview of Services

AWS Amplify is a suite of services ranging from UI components and a use case focused CLI to a CI/CD console and backend-scaffolding GUI.

On the other hand, Amazon SES provides a scalable solution for sending email and is used at companies like Reddit and Netflix.

In this post, we'll be using the Amplify CLI along with its JavaScript libraries to create a backend for a contact form, and use Amazon SES to send that information to our email.

## Getting Started

If you don't already have an AWS account or have the Amplify CLI installed, follow [this guide](https://docs.amplify.aws/start/getting-started/installation/q/integration/next#install-and-configure-the-amplify-cli).

> 🚨 This project makes use of lambda environment variables. The ability to do this via the CLI was introduced in version `5.1.0`. You may need to run `npm install -g @aws-amplify/cli` to ensure you're on the latest version.

Once setup, clone this `contact-form-starter` branch from this [github url](https://github.com/mtliendo/amplify-email-recipes/tree/contact-form-starter).

After cloning the project, install the dependencies and run the project. Below are some helpful commands:

```js
// visit: https://github.com/mtliendo/amplify-email-recipes/tree/contact-form-starter

git clone git@github.com:mtliendo/amplify-email-recipes.git
cd amplify-email-recipes
git checkout contact-form-starter
npm install
npm run dev
```

Once the project has started, visit `localhost:3000` and you should be presented with the following screen:

![Main page of contact form site](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995781382/_ELWLNebC.png)

## Understanding Our Backend

![architecture diagram of contact form using appsync, ses, lambda trigger, and dynamodb](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995783231/Cg38XK3Gf.png)

Using the above image as a reference, the Amplify services we'll need for our backend are in the following order:

1. AppSync: Fully managed GraphQL API

2. DynamoDB: NoSQL database

3. Lambda: FaaS/cloud function

In short, when a user fills out their contact form, that information will be stored in our database via our API. When that item is successfully saved, it will automatically trigger a function to send an email.

Sounds like a lot. Let's see what we have to do to get this working.

## Initializing Our Backend 🚀 

We'll start creating our backend by opening up a terminal and making sure we're in the root directory of our project.

From here, we'll initialize Amplify by running the following command:
```js
amplify init
```

We'll give our project a name and when prompted, select `n` to deny the default configuration. This is because we will be deploying our application as a static site. In NextJS, the name of that build directory is called `out`.

In the terminal, accept all of the prompts, except when it comes to the `Distribution Directory Path` enter `out`.

The entire flow should look like the below screenshot:

![Amplify Configure flow for static nextjs apps](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995785111/jRUsWlOxV.png)

Lastly, after selecting `AWS profile` we'll choose the profile we'd like to use.

The flow should look similar to the following screenshot:

![output from going through the Amplify init process](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995787232/pr55F4Cbu.png)

## Adding An API

With our application ready to use the Amplify CLI, we'll create our backend. As mentioned earlier, we are using AWS AppSync, which is a managed GraphQL API.

Traditionally, when sending email, a REST API is used. However, I've found that as needs change, AppSync provides more flexibility when it comes to handling authorization and a few other features.

To add an API in Amplify, we'll simply type the following command in our project's terminal:

```js
amplify add api
```

While in the CLI prompts, choose the following options:

1. GraphQL

2. [enter] to accept the default name

3. API key

4. "Contact form public API"

5. [enter] to accept a default of 7 days

6. [enter] to accept "No, I am done."

7. [enter] to accept the default "N" option

8. [enter] for a Single object with fields

9. "y" to edit the schema now

10. Select your editor of choice

By selecting those options through the prompts, we told Amplify how we would like our API to be built.

At this point, Amplify has opened up a file called `schema.graphql` with a sample Todo object. Replace everything in that file with the following:

```graphql
type Candidate 
  @model 
  @auth(rules: [{ allow: public, operations: [create] }]) {
   id: ID!
   name: String!
   email: AWSEmail!
}
``` 

To break down what's happening here, we are first creating a type called `Candidate`. In our application, a _Candidate_ represents the user submitting their information.

The `@model` text is called a directive. When Amplify sees this, it will automatically create a DynamoDB table *and* create CRUDL operations for the type it's associated with (in this case, Candidate).

The `@auth` directive setups up authorization rules on our API. Here we are saying, "We want our API to be public to anyone with an API key, but we only want them to be able to create entries in out database, they can't read, update, or delete items.

The next few lines are the fields associated with a Candidate. Here it's required that every Candidate has a unique id (automatically created with `ID`), a name, and an email -- AWS has a primitive called AWSEmail that automatically validates an email pattern.

With that, our API and database are ready to be deployed. Before doing so, let's move on to our function.

## Setting Up Our Function Trigger

AWS Lambda is an event-driven function. Meaning, it is called as a response to something. Often times, this is an endpoint like `/pets`, but in our application, we want this function to be called whenever an item is added to our database.

Fortunately, Amplify takes care of this process by allowing us to set this up from the CLI.

In our terminal, let's go through the following prompts:

1. `amplify add function`

2. Lambda function (serverless function)

3. "contactformuploader" as the name of the function

4. NodeJS

5. **Lambda Trigger**

6. Amazon DynamoDB Stream

7. Use API category graphql @model backed DynamoDB table in the current Amplify project

8. [enter] to not configure advanced settings

9. [enter] to edit the local function now

10. Choose your editor of choice

This will open up the function in your editor. Before we remove the contents, let's chat about the generated code.

When a change happens to a record in our database -- a change being either a `INSERT`, `MODIFY`, or `REMOVE` event, that information is sent as a _stream_ of data to our lambda function. 

However, our database can undergo heavy traffic. So instead of firing our lambda for one change at a time, the changes can be sent in batches called _shards_. No need to get too technical, but this is why the generated code is iterating over `event.Records`.

To drive the concept home, here's a diagram to showcase streaming and sharding:

![streams terminology](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995788992/iFSDr6G_m.png)

With that mini-lesson out of the way, let's replace the content in our lambda function, with the following:

```js
const aws = require('aws-sdk')
const ses = new aws.SES()

exports.handler = async (event) => {
  for (const streamedItem of event.Records) {
    if (streamedItem.eventName === 'INSERT') {
      //pull off items from stream
      const candidateName = streamedItem.dynamodb.NewImage.name.S
      const candidateEmail = streamedItem.dynamodb.NewImage.email.S

      await ses
          .sendEmail({
            Destination: {
              ToAddresses: [process.env.SES_EMAIL],
            },
            Source: process.env.SES_EMAIL,
            Message: {
              Subject: { Data: 'Candidate Submission' },
              Body: {
                Text: { Data: `My name is ${candidateName}. You can reach me at ${candidateEmail}` },
              },
            },
          })
          .promise()
    }
  }
  return { status: 'done' }
}

```

This function will be automatically called when a candidate submits their information. The `event` will contain the related stream of data. So from here our job is simple:

Grab the items from the stream, and send an email.

Using the AWS SDK, we call the `sendEmail` function from the `ses` module.

Wit that out of the way, we now have at least touched on all the pieces of our backend. We still however have a couple loose ends.

1. Our function doesn't have permission to interact with SES

2. We need to setup this `process.env.SES_EMAIL` variable

3. We've yet to setup SES

4. Our frontend code isn't setup to interact with our backend.

Let's change gears for a bit and start with the third item and revisit the others.

## Setting Up SES

As mentioned earlier, Amazon Simple Email Service (SES) provides a scalable way to send email. When first setting up SES, AWS will place you in sandbox mode.

This means we'll have the following constraints:

1. We can only send/receive to verified email addresses

2. We can only send 1 email/sec

3. Only 200 emails/day are allowed

Fortunately for our application, this won't matter too much.

To get started, let's hop into our terminal and run the following command:

```js
amplify console
```

When prompted, select "Amplify console".

> 📝 you may be asked to log in to your AWS account

Once logged in, search for "SES" in the top search bar of the console and hit enter.

![AWS SES Getting Started Page](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995790682/AXfWjng8l.png)

You should see a view similar to the one above. If not, you may need to click the top banner to be taken to this newer UI.

From here, perform the following steps:

1. Click the orange "Create identity" button

2. Select the "Email address" option and enter your desired email

3. Click the orange "Create identity" button

![Alt Text](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995792535/IFgObIal5.png)

That's it! Setting up an email for this service is well...simple 😅 

There are two things we'll need before we hop back into our code.

First, copy the ARN for your email by clicking the copy icon on the verified identities screen as show in the screenshot below:

![Copy ARN](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995794540/j1HtF8q12.png)

Store that in a notepad. We'll need it in a bit.

Next, SES sent a confirmation email to the email address that was provided. Click the verification link and we're all set to head back to our code.

## Updating Our Lambda

Recall that we need to both give our function permission to access SES, and add an environment variable to the function called `SES_EMAIL`.

Let's first update the permissions.

In your project directory we'll want to navigate to the following directory:

```js
amplify/backend/function/your-function-name/
```

Inside of this directory, you'll see the `src` directory for lambda, and a file titled

```js
your-function-name-cloudformation-template.json
```

![Alt Text](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995796480/bbKIS8jbG.png)

Select this file.

No need to be intimidated, this JSON code is known as Cloudformation and is what Amplify has been creating for us when we were interacting with the CLI.

It's full of settings and rules and we're about to add one more.

Search for `lambdaexecutionpolicy` (it should be right around line 132). 

This object has a `Statement` array that currently contains a single object. This object lets our function create logs in AWS.

Add the following object to the `Statement` array **and save**:

```js
{
 "Action": ["ses:SendEmail"],
 "Effect": "Allow",
 "Resource": "the-arn-you-copied-from-ses"
}
```

This small addition gives our function the ability to call the `sendEmail` function using the email we verified.

The `lambdaexecutionpolicy` object should look like the below screenshot (note I removed my email in place of a `*` for a bit more flexibility):

![Lambda with SES permissions](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995798712/3ovrehn3f.png)

The next step is to add the environment variable to our function. 

Back in the terminal, run the following command:

```js
amplify update function
```

Enter the following options:
1. Lambda function (serverless function)

2. [Select your function name]

3. Environment variables configuration

4. type `SES_EMAIL`

5. Enter the email that was verified with SES

6. I'm done

7. No -- I don't want to edit the local file now

## Push Up Our Backend

We've done a lot by only running a few commands in the CLI. This templated our resources, but we have yet to push everything up to AWS.

Let's fix that by running the following command in the terminal:

```js
amplify push
```

This will provide a table of the primary resources we created (recall that our database is a secondary resource created by the @model directive).

![amplify status](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995800975/b06o7aEGp.png)

After selecting that you'd like to continue, select the following options:

1. Yes -- generate code for the API 🔥 

2. JavaScript

3. [enter] to allow the default file path

4. Yes -- generate all _possible_ operations (recall we only allow `create` per our schema)

5. [enter] to accept a max depth of 2

It'll take a few minutes for our terminal to finish up, but once that's done, our backend is complete 🎉 

Let's wrap this up by giving our frontend the ability to talk to our backend.

## Configuring Amplify Libraries

We'll start off by installing the AWS Amplify JavaScript package:

```js
npm i aws-amplify
```

Once that is installed, we'll marry our frontend and backend together. In `_app.js`, add the following lines:

```js
import Amplify from '@aws-amplify/core'
import config from '../src/aws-exports'
Amplify.configure(config)
```

Here we bring in the Amplify library, bring in our config (Amplify generated this and put it in `.gitignore`), and then we pass in our config to Amplify.

Next up, in `ContactForm.js`, we'll also bring in the following imports:

```js
import { API } from 'aws-amplify'
import { createCandidate } from '../src/graphql/mutations'
```

> 📝 Feel free to check out the `createCandidate` mutations file that Amplify generated for us.

The API category is how we will talk to our AppSync API. Recall that this should not only store the contact in our database, but send an email to our verified address as well.

The `ContactForm.js` file has the following lines of code:

```js
// TODO: Add code to send email here
console.log('<send to backend here>')
```

Replace the above with this snippet:

```js
await API.graphql({
  query: createCandidate,
  variables: {
    input: {
      name,
      email,
    },
  },
})
```

With that bit out of the way, we can now test our project!

Restart your application on `localhost:3000` and test it out. If all went well, after a few seconds you'll have an email in your inbox 🎉 

> 📝  Because our emails are being sent via SES, they may show up in a spam folder or flagged by your email provider. This is because we haven't setup DKIM with SES. Though not terribly difficult, it's out of scope for this tutorial. However, if interested, you can read more about it [here](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-authentication-dkim.html)

## Hosting Our Project

Having this run locally is great, but for a contact form, chances are we want it to be live on the internet.

Fortunately, Amplify allows us to do this from the CLI as well. 

To get started, in our terminal, we'll run the following command:

```js
amplify add hosting
```

From the prompts, we'll select the following options:

1. Hosting with Amplify Console

2. Manual Deployment

Once selected, we can run the following command to view changes and upon accepting, our application will be deployed and a live on the web:

```js
amplify publish
```

![Amplify hosting flow](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995803017/TLhOpMEBb.png)

> Copy and paste the generated URL in your terminal to browser to view.

As you may have noticed in the CLI prompts, Amplify also supports git-based deployments. More information on setting that up can be found in the [Amplify docs](https://docs.amplify.aws/guides/hosting/git-based-deployments/q/platform/js#n4-deploy-your-app-to-aws-amplify)

## Recap

Using Amplify takes care of a lot of the heavy-lifting when it comes to setting up AWS services so that we can focus on our actual business logic. 

It's also good to remember that Amplify allows us to own the code that we deploy by letting us modify the generated Cloudformation templates.

Be sure to follow this series, or [follow me on Twitter](https://twitter.com/intent/follow?screen_name=mtliendo) to get notified when the next iteration of this series comes out: 

**Sending emails with attachments!** 📧 

Until then 🤖





