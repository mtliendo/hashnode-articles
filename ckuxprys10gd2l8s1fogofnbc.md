## Fetch both price and product data from stripe in a single api call

# Use Case

 - I want to display products in my web app that show the product's name, description and price. Similar to the below screenshot

![product card that shows plant picture, product name and price](https://cdn.hashnode.com/res/hashnode/image/upload/v1634622991134/1a9vevy3l.png)

- I want to use the Stripe Dashboard as my source-of-truth.

- Upon visiting my simple store, I would like my customers to receive the latest product information.

# The Problem

I tried to fetch all the `product` data, but that didn't work since it doesn't contain the prices for those products.

I tried to fetch the `price` data, but the `product` field only contains an ID for the product and not the rest of the data.

> 🤔 "Do I really have to get all of my price data, then iterate over each price just to get the product data?"

Neither the  [Stripe Price docs](https://stripe.com/docs/api/prices/list?lang=node)  or the  [Product docs](https://stripe.com/docs/api/products/list?lang=node) address this use case. A quick google search confirmed my initial approach:


![StackOverflow answer confirming my initial assumption](https://cdn.hashnode.com/res/hashnode/image/upload/v1634623505257/c1VeNjOHm.png)

But for such a common task, it just wasn't sitting right with me.


%[https://twitter.com/mtliendo/status/1450200714664349699?s=20]

# The Solution

Special shoutout goes to my buddy  [Nick (@dayhaysoos)](https://twitter.com/Dayhaysoos). He let me know that prices can use the `expand` parameter to get the corresponding product information. 

After a quick search through the  [Stripe docs for "expand"](https://stripe.com/docs/expand), the solution became clear: 

```js
import Stripe from 'stripe'

const Home = ({ productPriceData }) => {
	console.log({ productPriceData })

	return <div>display stripe data</div>
}

export async function getServerSideProps() {
	const stripe = new Stripe(process.env.STRIPE_SECRET_KEY)

	const productPriceData = await stripe.prices.list({
		expand: ['data.product'], // 🎉 Give me the product data too
	})
	console.log(JSON.stringify(productPriceData, null, 2))

	return {
		props: { productPriceData },
	}
}

export default Home
```

# 🗒️ Quick Notes

- Expand can be used for just about any API call, but will only go 4 levels deep on what it returns.
- It's showcased here, but just to be clear, you can dig into an object with dot-notation: `data.product.nestedKey.nestedKey2`
- I was going to use the `revalidate` key in `getStaticProps` to be cool, but then I realized that if a user refreshes the page, they should _always_ see the correct price.