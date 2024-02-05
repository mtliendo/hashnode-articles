---
title: "Hosting a Headless Hashnode UI on AWS Amplify"
seoDescription: "Optimize blog using Hashnode's Headless UI on AWS Amplify for customizations, integrations, and seamless CMS article publishing"
datePublished: Mon Feb 05 2024 16:35:51 GMT+0000 (Coordinated Universal Time)
cuid: cls95lkoa000e08l2g0jz6pu6
slug: hosting-a-headless-hashnode-ui-on-aws-amplify
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1707145608874/450ff0ff-7691-460b-93c1-24c24944ab72.png
tags: aws, hashnode, serverless, headless-cms, awsamplify

---

I've been blogging on Hashnode for several years. I know and have met several members of the team, and for the most part, can say I'm genuinely impressed with it. However, up until recently, I'd also say I'm not your typical Hashnode user. I've always wanted the ability to add my own customizations and integrations while still having a dedicated CMS to easily publish articles.

Fortunately, the release of Hashnode's headless UI makes this all possible.

This post will focus on my journey enabling the [Hashnode Starter Kit](https://github.com/Hashnode/starter-kit) to run on AWS Amplify, why I decoupled it from its monorepo setup, and what my plans are for the future of my blog.

## Where standard Hashnode fell short

Hashnode has done an excellent job at delivering incremental value. I remember when it was just blogging. Then [custom domains](https://support.hashnode.com/en/articles/5755362-how-to-map-a-custom-domain) launched, followed by [custom CSS](https://hashnode.com/post/style-your-hashnode-blog-with-custom-css-ckfwpesyg00ut0es10jgk5uwl). Each iteration felt like it was from the voice of their audience while [technical details were made public](https://engineering.hashnode.com/hashnodes-overall-architecture) for folks to learn from.

However, my needs were diverging from the the platforms core audience. I didn't want [another AI tool](https://hashnode.com/ai) to offer over embellished ways of writing. I wanted to insert sponsor links. I wanted a customized Focus Otter store that integrated with Stripe, and if I'm being honest, I felt the [newsletter experience](https://townhall.hashnode.com/publishing-a-newsletter-is-now-as-easy-as-blogging) offered by Hashnode was too limiting for what I'd like to put out to the audience.

At the same time, my friend Allen Helton shared [details around his custom blogging solution](https://www.readysetcloud.io/blog/allen.helton/how-i-built-a-serverless-automation-to-cross-post-my-blogs/), so I migrated my blog to a custom setup using GitHub Pages and mimicked his setup.

> I failed incredibly hard.

[Allen's setup was customized](https://www.readysetcloud.io/blog/allen.helton/how-i-built-a-serverless-automation-to-cross-post-my-blogs/) to his needs, goals, and technical ability. It was the culmination of small improvements over time. So I tried to build my own custom setup and it went exactly as you might expect:

%[https://twitter.com/focusotter/status/1709583746226114719?s=20] 

I felt stuck. What I wanted was a launch pad. A way to easily get a custom site up and running...but not too custom where I would spend days/weeks trying to make it my own. Well as always, Hashnode listened and as the author Mark Twain said,

> "History doesn't repeat itself, but it often rhymes."

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707147963721/6967a42a-ec8d-4f91-917a-b1385b1f1561.png align="center")](https://hashnode.com/headless)

Hashnode had once again delivered a feature that felt straight from the voice of their audience.

## Customizing the Headless UI Experience

A headless UI is a term used to describe a feature where backend functionality is given and exposed to the frontend via API's. For example, a React UI component that creates a table is nice, but probably limiting in how you can customize it. What's better, is to [give developers all the tools, hooks, and methods to build their own table and they provide the UI themselves](https://tanstack.com/table/latest).

Hashnode does the same thing by providing 3 starter repos that come integrated with all the utility functions, API calls, etc for building out a Hashnode-like experience.

Using [their GraphQL API](https://gql.hashnode.com/), developers can craft queries that best match the UI experience they're trying to deliver.

However, there were a two problems I had with their setup.

1. It was tailored to Vercel
    
2. It was setup as a monorepo
    

The first one made sense in a way because the 3 starter themes offered (`personal`, `hashnode`, `enterprise`) are NextJS apps, but going back to my needs, I like the integration and customization offered by [AWS Amplify](https://aws.amazon.com/amplify/?gclid=CjwKCAiAq4KuBhA6EiwArMAw1LVvjrCGSjS2-WFXWIuU0bOO75QZdOcIJjJ-B3DW0ZKaP0waSwTcPBoCTOcQAvD_BwE&trk=66d9071f-eec2-471d-9fc0-c374dbda114d&sc_channel=ps&ef_id=CjwKCAiAq4KuBhA6EiwArMAw1LVvjrCGSjS2-WFXWIuU0bOO75QZdOcIJjJ-B3DW0ZKaP0waSwTcPBoCTOcQAvD_BwE:G:s&s_kwcid=AL!4422!3!646025317188!e!!g!!aws%20amplify!19610918335!148058249160). It provides full support for NextJS, while allowing me to build out custom solutions using serverless services

%[https://twitter.com/AllenHeltonDev/status/1601316734278901761?s=20] 

As for the second option, this is more of a personal gripe. The monorepo setup is made with `pnpm`, and while [AWS Amplify has support for monorepos](https://docs.aws.amazon.com/amplify/latest/userguide/monorepo-configuration.html), I didn't fee like the [Hashnode starter kit](https://github.com/Hashnode/starter-kit) *should* be one, but rather standalone packages.

My reasoning is that if the goal is to deploy only one of the themes, then despite their being shared resources, the starter templates should live on their own with shared packages and styles in NPM packages. This is similar to [how Vercel allows for prebuilt templates](https://vercel.com/templates?framework=next.js&utm_source=google&utm_medium=cpc&utm_campaign=18576682555&utm_campaign_id=18576682555&utm_term=nextjs%20example&utm_content=141035138526_665293501587&gad_source=1&gclid=CjwKCAiAq4KuBhA6EiwArMAw1E3OZAa3GJcmmvtV5Eo2-1CydW7CnRQD9ssHOpTjR8u_Bev82ENh_RoC1wIQAvD_BwE) to be used.

Fortunately, deleting the packages for the monorepo and moving things into a single project repo was very straightforward. If wanting to view what that kind of repo looks like, I have a branch setup that you can take a peek at:

%[https://github.com/focusOtter/hashnode-enterprise-standalone/tree/standalone-enterprise] 

> 🗒️ The great thing about this setup, is that it's just a NextJS app using the pages directory and SSG. This means it can be deployed *anywhere*.

## Setting up my project on AWS Amplify

%[https://www.youtube.com/watch?v=ucVK6Z55PZY&t=20s] 

Gone are the days of uploading a project to an S3 bucket, setting up a Cloudfront distribution, and creating a hosted zone in Route53 just to get a site up on the web using AWS.

As you'd come to expect, you just tell Amplify which Git provider you're using (GitHub), which repo you'd like to upload, and it does the work for you.

Because I purchases my domain on AWS, setting up a subdomain was as simple as writing `blog`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707149614104/d4d558a8-1e94-4927-88d9-542d922f1cfd.png align="center")

This is great because I already use Linktree with Amplify to setup a custom domain for `focusotter.com`

%[https://youtube.com/shorts/OSLwLTdjHfg?si=gcM5uMtTNl1AivLe] 

From there, the only thing left to go to Hashnode and tell it to enable headless UI mode.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707149857352/543c5549-8875-47a2-8eef-6bbf283da965.png align="center")

This setup allows me to write my blogs using Hashnode's amazing CMS, schedule them, view analytics, and more, while still having complete control over what the user sees and experiences on my site.

## Where I plan on taking my blog

At the time of this writing, I still have the base `enterprise` starter plan. The "Book a demo" link doesn't do anything and the newsletter doesn't go anywhere.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707149972251/d990cce5-f52c-4265-8fe8-b2f29d442115.png align="center")

However, it will now be very easy to customize this experience to my personal needs. A few things I'll be adding in the short term:

1. **Audio Blogs:** Hashnode removed this feature due to non-usage, but I would love to bring this back!
    
2. **Translations:** I'd love to use the power of AWS to automatically translate my site into a language that best suits the reader.
    
3. **Store front:** Folks often ask me where they can get Focus Otter apparel and swag. It's coming soon! 🦦
    
4. **Premium blog posts:** Converting my blog into a SaaS product where users can signup, and select a *very* small donation plan to get access to premium blog posts is something I've been wanting to add for a while.
    
5. Much more!
    

## Conclusion

This post showed you how I setup my blog with a custom domain on AWS Amplify while still having it backed by Hashnode's headless UI.

But that wasn't really the point.

I want to highlight what's possible when a company genuinely strives to make life simpler for its users while still allowing them to creatively expand on what they've built.

Hashnode does this very well and for that, I look forward to blogging with them for many years to come.

\-- Focus Otter 🦦