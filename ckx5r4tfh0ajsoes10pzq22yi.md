## How to Have Fine-Grained Control Over Your Amplify Projects!

## Overview

When adding a backend resource with AWS Amplify, like a Cognito User Pool, a Lambda function, or an S3 bucket, we provide reasonable defaults and best practices so that you can spend more time focusing on your business logic.

At Amazon, we like to say we take care of the  [Undifferentiated Heavy-Lifting](https://www.linkedin.com/pulse/eliminating-undifferentiated-heavy-lifting-jimmy-ray/) ™️.

However, we realize that our users may have specific needs that differ from what we have in place. For example, when adding authentication with `amplify add auth`, we automatically set the password length to be at least 8 characters long without requiring special characters.

This may not work out for every organization, so we allow users to run `amplify update auth` to change some of those attributes from right within the Amplify CLI.

This works great, but AWS Services have many knobs that users can tweak and it isn't practical to add every one in our CLI.

To address that, we recently added a new command to the CLI:

```sh
amplify override <category>
```

This command integrates with the  [AWS CDK](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) to allow your Amplify generated resources to be configured at a much more granular level that is easily reproducible.

In this post, we'll look at examples on how this may be used in your Amplify projects.

## Overriding Authentication

During the process of adding authentication with the `amplify add auth` command, you can enable varying use cases such as adding federated identities from social providers, setting password defaults and more. 

But let's say you wanted to do something really specific like changing the length of time a signup-confirmation email is valid for. Amplify defaults to 30 days. But maybe your organization only allows for 2. That's now possible with only a few commands:

```sh
amplify override auth
```

This command integrates with the CDK to create an `overrides.ts` file that looks like so:

```ts
import { AmplifyAuthCognitoStackTemplate } from '@aws-amplify/cli-extensibility-helper';

export function override(resources: AmplifyAuthCognitoStackTemplate) {
    
}
```

Inside of the `override` function, we have access to the `resources` parameter to reference everything pertaining to our authentication resource. Since this file is written in TypeScript, we can take advantage of intellisense to find the `TemporaryPasswordValidityDays` option that we need:


![autocomplete showing the passwordPolicy CloudFormation link](https://cdn.hashnode.com/res/hashnode/image/upload/v1639460994922/uRX2mSgYJ.png)

> 🗒️ Notice how there is not only information about the property, but a link to learn more.

Here is the complete example:

```ts
import { AmplifyAuthCognitoStackTemplate } from '@aws-amplify/cli-extensibility-helper';

export function override(resources: AmplifyAuthCognitoStackTemplate) {
	resources.userPool.policies = {
		passwordPolicy: {
      ...resources.userPool.policies["passwordPolicy"], // spread in all other policy values
			temporaryPasswordValidityDays: 2,
		},
	}
}
```

## Modifying a DynamoDB Table Created By AppSync

 [AWS AppSync](https://aws.amazon.com/appsync/) is a managed GraphQL service that is used to build production-ready, real-time applications.

Amplify takes this a step further by adding  [directives](https://docs.amplify.aws/cli-legacy/graphql-transformer/directives/) that make working with AppSync even easier.

One such directive is `@model`. Adding this to a `type` will tell Amplify to not only add a DyanamoDB table with the same name, but to prompt is the user would like to automatically have all CRUDL operations scaffolded out as well as websocket subscriptions 🤯

When working with AppSync in Amplify, we abstract away the database so that focus is put on the data model itself. However by running the following command, you can gain access and modify it:

```js
amplify override api
```

Imagine a scenario where you'd like to not only enable a `TTL` on your database, but also take advantage of the new  [DynamoDB Standard-IA table class](https://aws.amazon.com/blogs/aws/new-dynamodb-table-class-save-up-to-60-in-your-dynamodb-costs/) released at re:Invent this year.

> 🗒️ A Time To Live (TTL) specifies the time when an item is to be deleted in a database. For more information and an example walkthrough,  checkout this post 

%[https://blog.focusotter.com/send-an-sms-to-customers-using-react-and-aws-amplify]

These are the steps we'd use to accomplish that:

1. Run `amplify add api` and follow the prompts.
  - change the auth mode from `API` to `Amazon Cognito User Pool`.
  - Select the `Todo` (_Single Object with Fields_) schema template.
2. Once the schema opens up in the editor, paste in the below code:
```js
type Todo @model @auth(rules: [{ allow: owner }]) {
	id: ID!
	name: String!
	description: String
	ttl: Int!
}
```
3. Run `amplify override api` to create an `overrides.ts` file for this resource category.
4. In this file, paste in the following code:
```ts
import { AmplifyApiGraphQlResourceStackTemplate } from '@aws-amplify/cli-extensibility-helper'

export function override(resources: AmplifyApiGraphQlResourceStackTemplate) {
	resources.models['Todo'].modelDDBTable.timeToLiveSpecification = {
		enabled: true,
		attributeName: 'ttl', //what we defined in our schema
	}

	resources.models['Todo'].modelDDBTable.addPropertyOverride(
		'TableClass',
		'STANDARD_INFREQUENT_ACCESS'
	)
}
```

That's it! 🎉

Notice how we can dive into our API to access the underlying database to set the TTL. Also, notice how we can use the `addPropertyOverride` feature of the CDK to change the table class since `Standard-IA` tables are so new, the option isn't typed out in the CDK as of this writing. 😎

Viewing the AWS Console, we can verify our overrides.

![dynamodb ttl set in the AWS Console](https://cdn.hashnode.com/res/hashnode/image/upload/v1639464364531/DOfURLHgV.png)


![DynamoDB table class is standard-IA in the AWS Console](https://cdn.hashnode.com/res/hashnode/image/upload/v1639464440351/r00RvfG6a.png)

## Conclusion

In this post we looked at 3 different ways an Amplify generated resource can be overridden:

1. At the top-level, as shown in overriding authentication.
2. By diving into the category to override underlying resources, as shown with `TTL`.
3. By diving deep to use the CDK to modify the resource at the CloudFormation level, as shown with `addPropertyOverride`.

At the time of this writing, `auth`, `storage`, and `api` are all able to be overridden, with `function` being the next to be added.

If wanting to create your own resources within Amplify, you may want to check out my last post where I showed how to do just that!

%[https://blog.focusotter.com/the-complete-guide-to-adding-aws-resources-to-your-amplify-project]

Thanks for following along, and I can't wait to see what you all have to build! If you have any questions feel free to drop a comment and  [follow me on Twitter](https://twitter.com/mtliendo) to stay up to date with what I have going on!