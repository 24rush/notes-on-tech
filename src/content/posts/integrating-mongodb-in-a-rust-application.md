---
title: "Integrating MongoDB in a Rust application"
description: "MongoDB integration with official Rust driver, VSCode extension, and data mapping for efficient database management"
short_desc: "Rust has excellent support for interacting with MongoDB through the official Rust MongoDB driver and if you are using Visual Studio Code, I highly recommend installing the following extension before proceeding."
date: 2023-06-30T05:00:00Z
image: "/images/posts/cover_hashnode_1.png"
categories: ["Programming"]
authors: ["Alin"]
tags: ["MongoDB", "Rust"]
draft: false
---
Rust has excellent support for interacting with MongoDB through the [official Rust MongoDB driver](https://www.mongodb.com/docs/drivers/rust/) and if you are using Visual Studio Code, I highly recommend installing the following extension before proceeding.

![image alt ><](/images/posts/14c5a889-e40f-46a7-ab25-dc80a8e72d8e.png)

The documentation on MongoDB's website is more than explanatory on how to configure your environment and it's also something straightforward to do so I won't go into details. I will focus on how I used it to build a viewer for all my Strava activities instead.

In my [main article](./mapping-all-my-sports-activities-using-rust-leafletjs-and-openai), I am explaining why I'm using two databases - one local database handled by the mongod service on my Linux machine and a remote one handled by MongoDB's Atlas. I created connections for both of them in VSCode using the extension mentioned above and this is what it looks like:

![image alt ><](/images/posts/377aa62e-5c6b-41c8-a62e-946c6e8ed695.png)

Notice the two playgrounds below, they are a powerful tool which I recommend you give it a try. They are javascript files that can be run natively against your databases to test different queries before implementing them in Rust - *not only that but you can also do more complicated data processing on the results and then write the results back in MongoDB which I will show later in the article.*

Now, a **quick reminder** on the structure of the project which for what this article is concerned is just a web application requesting data from MongoDB.

![image alt ><](/images/posts/3a41a512-0d23-455d-ae0f-1234bed80b7e.png)

There are two databases in my project (*Static DB* and *GC DB*), one meant for static (read-only) data and the second one for data that I am producing through my pipeline. Both databases' collections store JSON objects that I either retrieve from Strava API or I am creating from the Rust pipeline.

<details data-node-type="hn-details-summary"><summary>Where can I see the complete Rust code?</summary><div data-type="detailsContent">The code for the database handling in Rust can be found on Github ▶️ <a target="_blank" rel="noopener noreferrer nofollow" href="https://github.com/24rush/the-world-covered/tree/develop/src/database" style="pointer-events: none">src / database</a></div></details>

## Data mapping

The way these collections are meant to be used is by mapping their JSON contents to **Rust data types** which you can later use in your application. In my case, the JSON object I got from StravaAPI contained more data than I cared for (at least at the beginning of the project) but I did store the complete JSON in the database even if the corresponding mapped Rust datatype (the data actually used) was just a subset of that structure.

<script src="https://gist.github.com/24rush/0b7e32d59cec5d45160f8e8934e74821.js"></script>

And below you can see how the mapping is done between the Rust data types and JSON fields :

![image alt ><](/images/posts/515fe083-1376-44a6-83b1-ddf640195db9.png)

struct *Activity* is the parent type encapsulating the top-level JSON object and structs *Effort* and *Segment* are nested structures accessed from the *segment\_efforts* keys. Notice the use of ***Option&lt;&gt;*** for fields that might be null or unexistent in the JSON structure.

Things to keep in mind:

1. you don't need to map the complete JSON database object to a Rust data field, what is not mapped gets ignored
    
2. if something you mapped in the Rust data type is not found in the JSON object then Rust will panic (unless you specify a default value for that field by using *#\[serde(default)\]* or make it *Option&lt;T&gt;* instead of *T*)
    

## Application design

Every application will have one database which will contain different collections with different structures. In my case, I have two DBs with 2 or 3 collections each (Activities, Telemetry, Athletes). I decided to structure my application around the collections the databases contain and their corresponding operations.

![image alt ><](/images/posts/87fcd313-e29d-43a2-a94f-52d20e30f3fc.png)

### MongoDatabase

This is the creator of the connection to the Database and the distributor of collection handles. It also has a few generic operations to be used on any collection type like *upsert\_one, set\_field, exists*, etc. Notice that is not aware of any particular document structure.

<script src="https://gist.github.com/24rush/c72543719e01b4b90bbe0fc8c9ff8625.js"></script>

### Specific collections

The *&lt;Activities|Telemetry|Athletes&gt;Collection* classes are responsible for executing specific operations (querying, inserting, aggregating) using their collection's data and being aware of the structure of this data.
 
<script src="https://gist.github.com/24rush/d1edc4b3673fdd377c7517c9d938bd61.js"></script>

Things to note:

* *line 6:* they define the name of the collection they work on
    
* *lines 14-18*: they request from the *MongoDatabase* class collections for typed objects from the collection or raw documents
    
    *Because I am retrieving data from StravaAPI which is a verbose JSON, I need to also have the raw documents collection available so I can write/read this complete JSON not just the subset I am mapping through my data types.*
    
* *lines 31-45*: specific operation on the collection's data of returning results sorted by a particular field. Notice how the function is aware that the field corresponding to the athlete's ID is stored in the JSON structure under athlete.id
    

All the collections for a particular database (*strava\_db* in this case) are then placed together in this class:

<script src="https://gist.github.com/24rush/5645fbd39a605cdaebfd3aaf7358e9b8.js"></script>

Putting it all together, retrieving an activity from Strava's API, storing it in MongoDB and then getting it back in a typed Rust object looks like this:

<script src="https://gist.github.com/24rush/2c2c22417c0a29eb56a735918e311c4a.js"></script>

Few things to notice:

* *line 1*: the connection string for the database, in my case the *localhost* one but it might as well be the remote (Atlas) one
    
* *line 8*: *new\_activity* is a JSON object just returned from the Strava API. It's not the *Activity* Rust type as I want to store the complete JSON in my DB (for future use cases) and not just the part I mapped so far.
    
* *line 13:* Notice how I am mentioning which collection I want to query through the *activities* member of *StravaDB* and also that *db\_activity* is an actual Activity instance
    

## Playgrounds

As I was mentioning at the beginning of this article, playgrounds are powerful debugging tools (for visualizing the data, testing queries, etc) but also they can be a good option for implementing features altogether. In the gist below, I'm showing how I used such a playground to compute [VO2MAX](https://en.wikipedia.org/wiki/VO2_max) for every year since 2014 and store those results back into a MongoDB collection.

<script src="https://gist.github.com/24rush/28f00724007c306fed6eb925791e40d4.js"></script>

I hope this provides some clarity on how I utilized Rust in my project. While it's not a comprehensive step-by-step guide, it serves as a general outline and should be sufficient to give you a better understanding of how you can implement your own solution.

Thank you for reading and stay tuned ✌️