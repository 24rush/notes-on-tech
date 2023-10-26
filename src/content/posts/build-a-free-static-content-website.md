---
title: "Build a Free Content-Focused Website with Astro and Vercel"
description: "build a blazing free marketing, blogging, e-commerce website using Astro and Vercel"
date: 2023-10-25T05:00:00Z
image: "/images/posts/0a4ccaea-260e-48b7-9e34-11d909e25c70.png"
categories: ["Programming"]
authors: ["Alin"]
tags: ["Astro", "Vercel", "Cloudflare"]
draft: false
---

If you're a techie and find yourself needing to build a blazing fast static content website (landing page, portfolio, documentation, blog, marketing, etc.) there is a tech stack that can help you do that for free (you will just need to pay for the domain name). 

## Why actually?
While there are many free CMS platforms out there, you might wonder why bother with a custom solution. There are a few reasons why you might need it and ways in which it can be beneficial for you:

1. You want full flexibility over the whole website
    - you can build custom widgets (reactive web components) using *React/Vue/Svelte/etc.*
    - you can customize your website's theme however you want using Tailwind/Sass/Less

2. You want to update the content frequently
    - changing the content in a traditional CMS is a tedious operation which often requires opening up each individual post whereas with this solution every post is a Markdown file which makes updates as easy as find-and-replace
3. You require dynamic routing 
    - you can create dynamic routes at build time which means you can generate multiple similar pages based on templates and parameters passed to those template (read more about routing [here](https://docs.astro.build/en/core-concepts/routing/))
4. You use widgets that rely on *SharedArrayBuffers* or *high-resolution timers* which require a secure context
    - this security requirement involves having your site cross-origin isolated which means your documents (_posts_ or _pages_) need to be served with these headers set:
        ```bash 
        Cross-Origin-Opener-Policy: same-origin
        Cross-Origin-Embedder-Policy: require-corp
        ```
        which is done through a server configuration property that most CMSs don't support changing

## The tech stack
As already hinted in the title, at the core of all this is [**Astro**](https://docs.astro.build/en/getting-started/) which is a "all-in-one web framework for building fast, content-focused websites".

On top of this, there is [**Vercel**](https://vercel.com/) which is "a cloud platform for building, deploying, and scaling serverless applications and static websites" and lastly, the humble kid on the block, [**GitHub**](https://github.com/).

### How they all work together
_the instructions assume that you already have accounts on GitHub and Vercel (free-tier)_

1. Start by choosing a theme for your website from https://astro.build/themes/ or go full cowboy with an empty template by simply running: 
    ```bash
    npm create astro@latest
    ```

2. If you went the 'existing theme' route then go ahead and fork that theme into your GitHub repo. In the full cowboy route, you will just need to push the folder Astro created into a GitHub repo. Bottom line is that in both scenarios, you will have a *GitHub repo containing your Astro project*.
  
2. Test it locally by running the following command in the root of the Astro project:
    ```bash
    npm run dev
    ```
    - verify that it's deployed by navigating to http://localhost:4321/ in your browser

3. Go to Vercel and create a new project then import the GitHub repo containing the Astro project (_you will need to allow Vercel to 'read' your GitHub repo_):
![](/images/posts/d6f3597e-55aa-4ee0-abac-acf86aba501e.png)

4. That's it! Vercel will build the Astro project which will generate the website's content and make that content available at ```<project_name>.vercel.app```

**Bonus tip**

You will most probably want to use a _custom domain_ for your website in which case you will need to buy it first. After you do that, I recommend you use [Cloudflare's (free plan)](https://www.cloudflare.com/) to manage your DNS settings for the newly purchased domain. 

To assign the new domain to your *vercel.app* subdomain, just go to the Vercel project under *Domains*. You will be instructed which DNS entries you need to update/create and these you will need to do in the Cloudflare account under *DNS > Records*. The changes will be visible almost immediately. 

After this step, your website will also be available at your custom domain's address.

## How to write and publish content

Your content (pages and posts) has to be written in [Markdown](https://docs.astro.build/en/guides/markdown-content/) and put under ```src/pages/<pagename>.md``` or ```src/pages/posts/<postname>.md```. Once you run ```npm run dev``` Astro will monitor these folders for changes so the updates you make will be visibile as soon as you hit Save.

When you build the project, Astro will process all the .md files from these folders, create their HTML counterparts and generate static routes for each of them therefore making them accesible at either  _wwwroot/\<pagename>_ (for pages) or _wwwroot/posts/\<postname>_ (for posts). Read more about routing [here](https://docs.astro.build/en/core-concepts/routing/).

Once you are done editing your posts, you can go ahead and push the .md changes back to the GitHub repo. When this happens, Vercel will be notified and will re-deploy the newly made changes thus updating your website.

A diagram of the flow can be seen below:
![](/images/posts/c7306eb1-de73-48dd-ba91-7166797ca559.png)

### Astro islands
You can create also custom widgets by defining them in *.astro* files which you can then include into pages that have *.mdx* extension (this extension signals that you are using custom components in that page). These custom components must be written in a JSX-like language but basically they contain HTML/CSS + Javascript and achieve a certain functionality that can be embedded in every post. Read more about the MDX integration [here](https://docs.astro.build/en/guides/markdown-content/). 

At a higher level, this pattern is part of Astro's [Component Islands](https://docs.astro.build/en/concepts/islands/) concept which refers to these interactive UI components that render among the static content.

If you want to dwelve deeper into Astro, I recommend you start with: [project structure](https://docs.astro.build/en/core-concepts/project-structure/), [components](https://docs.astro.build/en/core-concepts/astro-components/), [UI frameworks](https://docs.astro.build/en/core-concepts/framework-components/) and [plugins](https://docs.astro.build/en/guides/integrations-guide/).

## A few caveats
- writing raw Markdown might not be fun but there are tools that can assist like [Stackedit.io](https://stackedit.io/) or even VS Code's integrated Markdown preview
- if you plan on creating a subscriber based blog (like Substack, Medium), capturing and managing emails might not be that straightforward even though Mailgun should already have embeddings for this
- generally, if you need a more complex web application (something that requires user login/database access) then Astro is probably not the answer

## Showcase
For my usecase, Astro and Vercel have been amazing making my life so much easier and the development so much more enjoyable. 

If you want to see them in action, here are a few of my websites build with Astro: 
- https://primiipasi.info (content is in Romanian)
- this [blog](https://alincodes.vercel.app/) actually 
- my [Strava mapping site](https://the-world-covered.vercel.app/).