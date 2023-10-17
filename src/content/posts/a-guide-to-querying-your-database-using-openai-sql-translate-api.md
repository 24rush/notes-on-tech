---
title: "How to Use OpenAI SQL Translate API to Query your Own Database"
description: "Effortlessly analyze data with OpenAI SQL Translate API for MongoDB. Query sports activities, generate stats, and visualize data"
date: 2023-06-30T05:00:00Z
image: "/images/posts/cover_hashnode.png"
categories: ["Programming"]
authors: ["Alin"]
tags: ["OpenAI", "MongoDB", "Typescript", "Vercel"]
draft: false
---
I've always been fascinated by the idea of querying data using natural language, effortlessly exploring and analyzing data without worrying about the technicalities. 

Redirecting the mental energy spent on crafting queries towards truly understanding the data can lead to more valuable insights and with the advent of OpenAI, I was excited to make this idea a reality. 

In this endeavor, the dataset I had available was my collection of sports activities from _Strava_ - not a big data set, around 550 activities and maybe 300MB of data.

<details data-node-type="hn-details-summary"><summary>Where can I see this running?</summary><div data-type="detailsContent">The website is called <a target="_blank" rel="noopener noreferrer nofollow" href="https://the-world-covered.vercel.app/">The World Covered</a>.</div></details>

## OpenAI SQL Translate API

After exploring OpenAI's website, I discovered this exciting API called [SQL Translate](https://platform.openai.com/examples/default-sql-translate). At first, I assumed (based on the name and the example provided for its usage) that it would only work with SQL-based databases and since my database used MongoDB, I considered creating a converter to transform the SQL response into a MongoDB equivalent. However, this wasn't necessary, as when I prompted the API to generate a query for MongoDB, it worked remarkably well, returning a correctly formatted MongoDB JSON query. With this, the stage was set for creating something truly impressive.

### API key

OpenAI's platform for developers is not free as ChatGPT is and it requires an API key for each developer wanting to use it. This key has a paid quota associated with it and it must be kept private to avoid abuse. I discussed in a [previous](./mapping-all-my-sports-activities-using-rust-leafletjs-and-openai) article how I accomplished that using Vercel so I will not go into details here.

### Database structure

For the intention of this feature, I wanted to query only the data from one of my collections in the Mongo database and that is the *Activities* collection which is the summary of a Strava activity (either running, cycling, etc.). It contains a bunch of interesting fields about the activity:

![image alt ><](/images/posts/447b39be-66e9-4753-9652-0dafd56b2b84.png)

The search capabilities I wanted it to have were related to statistics mainly so from this big set of fields, I would be needing just a small subset.

### Writing the prompt

OpenAI is all about prompts so let's see how I formed mine.

![image alt ><](/images/posts/df8c88d5-0894-4878-ae5b-c518369d2780.png)

Notice how I am mentioning:

* that this is a **MongoDB** database so I want my query responses to be JSON (MongoDB format) and not SQL.
    
* the collection's properties which are just the subset I am interested in for the statistics I want to create on the data and the kind of queries I want it to be able to handle.
    
* that it should be using the *aggregate* function of MongoDB instead of what other querying facilities it might come up to (like find, etc.) - *you will see later why I insisted on this.*
    
* the fields' units (meters & seconds) so when I'm asking about kilometers it knows to convert them from meters.
    
* the request I am making followed by the 'natural language' string the user inputs (the search request).
    

Let's see it in action (in the playground) and what it responds when I'm asking it to show me the kilometers I've ridden every month in 2022.

![image alt ><](/images/posts/9b088f1d-c688-4a85-9a92-81f47ee370ff.png)

Pasting the query in the MongoDB playground, results in this (correct) response 👏:

![image alt ><](/images/posts/f7e79d35-7dc5-4bfe-a264-fd8ff4056ade.png)

Proving that it works, I could move to integrate this into my app.

## Architecture

![image alt ><](/images/posts/0ffa5c4f-8c05-4f20-8e12-e3b748a04574.png)

To safely store the API key, I am using Vercel as an API host which means the HTTP requests to OpenAI don't go directly from my client as that would expose the key but instead are forwarded to Vercel which holds the key in an environment variable. Vercel then uses the key, forms the request to OpenAI and sends it. The response is then sent back to the client.

The response the website gets back is a JSON containing the MongoDB query OpenAI created. This query is then processed and sent to MongoDB's Atlas through their WebSDK.

## Interpreting the response

One of the challenges of implementing this feature was to handle the responses OpenAI might send back. Depending on the questions the users pose, you could get a query that will have MongoDB return a list of activities or a list of objects containing some data (as we saw in the example above). I wanted to be able to handle this as reliably as possible so I've created two resolution scenarios depending on the content returned:

1. **list of activities**, in which case I would just display those activities on the map which my web app already knows how to do for other features
2. **list of objects**:
    
    - a list containing more than 1 element which means we will need to create a chart        
    - a single object list in which case we must have gotten back an aggregated value (a sum, count, etc.) in which case we will just show the number
        

### 1. Response consists of a list of activities

The listing below determines what kind of response we got depending on the presence of the *length* key and the *content* of such an object, if present.

The ***canParseFromObject*** function is checking if the object can be considered an *Activity*, e.g. it contains at least these fields: *'map', 'type', 'distance', 'location\_city', 'location\_country'* in which case will allow it to be displayed as an activity. This is scenario #1 from the above.

<script src="https://gist.github.com/24rush/9c9bf6d55e89b40d2585451cc0c022f0.js"></script>

![image alt ><](/images/posts/0011ae7e-d869-4df4-9052-cb17e1a7902d.png)

### 2. Response consists of a list of objects that do not resemble activities

For this scenario, we send the data to the charting component which decides itself how to display it, either as a chart (if it can extract a list of (x,y) pairs from the response) or in a tabular way (for one-dimensional objects).

I will explain briefly how this component works as it's a bit cumbersome to explain it in writing but please check it out in the repo ▶️ [src / components / GPTChart.vue](https://github.com/24rush/the-world-covered-web/blob/develop/src/components/GPTChart.vue)

Once the charting component receives the list of non-activity objects, it will:

* check if the size of the data is 1 in which case it will iterate over all the object's (key, value) pairs and display them in a table or if there is only one pair, as a single value.
    

![image alt ><](/images/posts/1fd79577-437f-44ac-8096-572e56a5f9da.png)

![image alt ><](/images/posts/fd84c52d-d227-4898-b323-7080b3b79273.png)

* if the size of the data is larger than 1 then it means we have something that can be put on a chart. It then proceeds to figure out who x and y can be depending on the names of the keys the object contains. It will try to put the key that has a name indicating time (like year, month, etc.) on the x-axis and rest on y.
    

![image alt ><](/images/posts/1303b195-281b-45b2-bdb7-c317541df184.png)

Because the response contained the *year* key, it decided that this is a good time-based key for the x-axis and used it and left the remaining *count* value for the y-axis.

![image alt ><](/images/posts/7263a0ad-9a3d-4773-bac1-8ef4d1b0ec97.png)

## Optimizing the use of OpenAI

There is one last thing I want to mention about this feature and that is the very crude query optimizer. I noticed how some queries, especially the ones requesting activities in a particular location, are very easy to parse without the need of going to OpenAI. I did this as I wanted to reduce the load on my quota and also because if you are using some city or country names, OpenAI will not deduce correctly what you are talking about and it would fail.

Below is the listing for the first implementation of this mechanism. It tries to determine if the user is looking for *(runs|rides)* in *location* in which case it goes and formats some known locations accordingly (like NL -&gt; Netherlands or Bucuresti -&gt; Bucharest).

<script src="https://gist.github.com/24rush/e4e402575d15668f89f5b631be48bd69.js"></script>

I am extremely proud of this feature, even though there is still more to explore and refine in the future. I eagerly look forward to the potential new features.

Thank you for reading and stay tuned ✌️