## 🌱Post confirmation trigger

> This is a digital garden post. This means I may come back and "water" it by making this look pretty, but for right now it's just a wee sprout 🌱

purpose: I needed to add an authenticated user to a database when they signup but when constructing the event in the lambda, I didn't know the details of what the `event` object contained. So I spent all of 15 minutes to create this example.

The CLI command:

```bash
npx create-next-app trigger-example && cd $_ && npm i aws-amplify @aws-amplify/ui-react && amplify init -y && amplify add auth
```

select `yes` to configure auth with defaults

```bash
amplify push -y
```

Once done pushing:

```bash
amplify update auth
```

Go through _all_ the options, selecting the defaults along the way. 

When asked if I want to update the attributes, I selected `email`, `name`, `family name`, and if I remember right `username` unless that is just a given attribute.

The very last option is to add a trigger. Say yes. Create your own module. Select `edit function`

Note that the code gets put in a `custom.js` file and isn't the `index.js` file.

Put in the following code:

```js
/**
 * @type {import('@types/aws-lambda').APIGatewayProxyHandler}
 */
exports.handler = async (event, context) => {
	// insert code to be executed by your lambda trigger
	console.log('the current user info', JSON.stringify(event, null, 2))
	return event
}
```

Update your `_app.js` file:

```js
import { Amplify } from 'aws-amplify'
import { AmplifyProvider } from '@aws-amplify/ui-react'
import '@aws-amplify/ui-react/styles.css'
import config from '../src/aws-exports'

Amplify.configure(config)
function MyApp({ Component, pageProps }) {
	return (
		<AmplifyProvider>
			<Component {...pageProps} />
		</AmplifyProvider>
	)
}

export default MyApp
```

Update your `index.js` file:

```js
import { Button, withAuthenticator } from '@aws-amplify/ui-react'

function Home({ signOut }) {
	return <Button onClick={signOut}>Signout</Button>
}

export default withAuthenticator(Home)
```

The result is that the default login UI looks like this when signing up for an account:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659591100568/Uciwq5gIH.png align="left")

Once you login, the output for your user looks like the following in CloudWatch:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659591160168/yTUpauDjR.png align="left")

delete it all: `amplify delete`

If wanting to update a dynamoDB table:

```js
const AWS = require('aws-sdk')
const docClient = new AWS.DynamoDB.DocumentClient()

async function main(event) {
	//construct the params
	const params = {
		TableName: process.env.TABLENAME,
		Item: {
			id: event.request.userAttributes.sub,
			firstname: event.request.userAttributes.name,
			lastname: event.request.userAttributes.family_name,
			username: event.userName,
			email: event.request.userAttributes.email,
		},
	}

	//try to add to the DB, otherwise throw an error
	try {
		await docClient.put(params).promise()
		return event
	} catch (err) {
		console.log(err)
		return event
	}
}

module.exports = { main }
```
