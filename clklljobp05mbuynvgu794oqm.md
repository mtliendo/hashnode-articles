---
title: "Automate Home Lights with the Fetch API"
datePublished: Thu Jul 27 2023 20:17:52 GMT+0000 (Coordinated Universal Time)
cuid: clklljobp05mbuynvgu794oqm
slug: automate-home-lights-with-the-fetch-api-1
canonical: https://focusotter.cloud/posts/undefined
cover: https://res.cloudinary.com/dgtvzkmvu/image/upload/f_auto,q_auto/v1689496355/home-automation/cover_ocgv2i.png

---


I read that [interior lighting](https://www.google.com/search?q=indoor+color+lighting&tbm=isch) can boost your mood, the same way colors on a web page can. So I decided to purchase some [LIFX Mini bulbs](https://www.lifx.com/collections/lights/products/lifx-mini-color)--no hub required, and can say after a couple days, it's definitely helped keep me sane during these times.

<!--more-->

## Automate Home Lights With The Fetch API

However, after a bit, my developer itch was kicking in. So in this project, we'll use the [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) web API to first list our lights, and from there, we will send a request to a particular light so that it's color changes.

> 🗒️ It's worth mentioning that if you don't want to take the coding route, the service [IFTTT has many 3rd party integrations](https://ifttt.com/lifx) that you can take advantage of to enhance your lighting experience.

---

🚨 If you just want to look at the code and fill in the blanks, there is a codeSandbox down below. Just make sure to update the `super-secret-token.js` file with your keys if wanting to make it work 😉

## Getting an API Key

I was pleased to find that LIFX has an API for their bulbs, and upon signing up, you can access your secret token that we'll need later on.

Admittedly, it took me longer in figuring where to find my token than it did to write the code, so to help out, here's the easy path:

1. Head over to the [cloud portal](https://cloud.lifx.com/settings) and sign in **with the same username and password that you used to sign in with on your mobile app**

2. Once you're signed it, you should be redirected to a page where you can "Generate New Token" (note that I already have a few tokens created).

Click the "Generate New Token" button and give your token a name.

![generate token](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995413862/ze-_wTb7t.png)

That's it 🎉 Now you should see your secret token.

> 🗒️ As is mentioned on the site, you'll want to copy that somewhere safe because once you navigate away from the page, you won't be able to access the token again and can only generate a new one. In codeSandbox, the easiest way to do this, is to make your project private or an a minimum, unlisted

![sandbox](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995415846/wiFxXV9Dv.png)

## Reading the Docs

Now that I had an API Key, I was able to [checkout the docs](https://api.developer.lifx.com/docs/authentication) and figure out what endpoints were available to me.

In particular the [List Lights](https://api.developer.lifx.com/docs/list-lights) and the [Set State](https://api.developer.lifx.com/docs/set-state) endpoints are what we'll be using.

### Authentication

It's worth reading over the [Authentication](https://api.developer.lifx.com/docs/authentication) and [Rate Limits](https://api.developer.lifx.com/docs/rate-limits) sections to understand how the API keeps us secure and prevents spamming. Here are the main points:

- If using `Basic` auth, as we are, we'll need to provide our token as the username
- Our requests have to have the `Content-Type: application/json` header
- Our token allows us to make 120 requests every 60 seconds. We can use the `X-RateLimit-Remaining` response header to see how many request we have left.

---

### Setting Up Our Project

As mentioned above, if you have your secret token, feel free to plug that into the `super-secret-token.js` file's `TOKEN` area and click the _List Light ids_ button to get your lights. From there, pick and id, plug it into the other secret value, and it should Just Work™️

{% codesandbox 6uqyc %}

For those still with me, let's use the sandbox above and take a tour of the code.

Starting with the `index.html` file, you'll notice it's fairly boring. It's vanilla JavaScript and we just have a couple of elements with `id`'s assigned so that we can target them in our code. The most appealing part is that we have `<input id="color-picker" type="color" />`. Giving an `input` a `type` attribute of "color" will give us a nice color picker on both desktop and mobile!

Let's get to the star of the show.

## _Finally_ Making Our Request With Fetch

In `index.js` we kick things off by bringing in our tokens, and grabbing a few elements that we created in our `index.html` file.

Next up we add an event-listener to our button so we can trigger a request whenever a user clicks the _List Light ids_ button.

Now on to our request.

### Fetch With Basic Auth

`fetch` takes in an endpoint as a first argument. Which we are passing as a string

```js
fetch(`https://api.lifx.com/v1/lights/all`
```

Additionally, `fetch` can take an object used for configuration as a second argument. This is where we can specify any required headers, as well as set the auth type.

```js
headers: {
  "Content-Type": "application/json",
  Authorization: `Basic ${btoa(TOKEN)}`
}
```

> 🔥 Note the use of the `btoa` function. This is the browser's built-in method for converting a _normal_ string to a base64 encoded string. If wanting to convert the other way around, there's also `atob()`.

From there, we continue as usual:

1. Fetch returns a _promise_ so we call `.then` to run code when we get a response from the server
2. We take the response from the server and parse the data as JSON via `res.json()`
3. We update the DOM to show the list of lights in the subsequent `.then` block.

### Fetch With A `Put` Verb and Hidden Headers

Things get even more interesting with our "change color" button.

After listening for a click event, we grab the value from the input and log it out. What this reveals is that the value is in fact a hex color: `#00ffff` for example.

This is great because to change a light's color to something more that just a generic "green", we'll have to pass in a hex code value.

Now within our `fetch` request, we have a new endpoint. This one ends in `id:${LIGHT_ID}/state` where the `LIGHT_ID` as you may have guessed, is one of the lights that we got back from our _List Light ids_ button.

However, let's take a moment to dissect the second argument: the configuration object

```js
{
    method: "put",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Basic ${btoa(TOKEN)}`
    },
    body: JSON.stringify({
      color: colorValue
    })
```

Here, we say the method is `put` because we are _sending_ data. The headers are the same as before. Next up the `body` is the data that we are sending. Servers typically expect JSON data to be passed to them, so we use the built-in `JSON.stringify()` method to convert our object to that format for us.

🎉All done🎉

That alone is enough to get the color to change! If all went well, you should see something like the below tweet

%[https://twitter.com/focusotter/status/1250285107077771265]

---

### 🔥Extra Credit🔥

Recall, that the API only allows us to send 120 requests per minute. So the question becomes: How can we make sure we don't go over our limit so that our application doesn't break or we get flagged for spamming?

The answer is in the final lines of our code:

```js
  .then(res => {
      console.log(res.headers.get("x-ratelimit-remaining"));
      return res.json();
    })
    .then(data => console.log(data)); // display to user
```

Normally, when using `fetch`, only a small amount of headers are actually available for us to access. So saying `res.headers["x-ratelimit-remaining"]` is going to give us `undefined`. However, using the special `res.headers.get()` function, we can target the header that lets us know how many requests are remaining.

There are actually plenty of other headers to checkout as well! I included an award-winning screenshot of how to find them:

![network tab](https://cdn.hashnode.com/res/hashnode/image/upload/v1628995417865/hSkZSgvWs.png)
