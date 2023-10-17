---
title: "Processing 2 Million GPS Points Using Rust"
description: "Visualize 2 million Strava GPS points in a Rust web app; learn data pipeline, data processing, gps track processing, de-duplication"
short_desc: "This article is part of a series in which I explain how I built a web app to visualize all my Strava activities on a single map."
date: 2023-06-30T05:00:00Z
image: "/images/posts/cover_hashnode_3.jpg"
categories: ["Programming"]
authors: ["Alin"]
tags: ["Strava", "Rust", "MongoDB"]
draft: false
---
*This article is part of a series in which I explain how I built a web app to visualize all my Strava activities on a single map. The overarching parent article can be read* [*here*](./mapping-all-my-sports-activities-using-rust-leafletjs-and-openai)*.*

<details data-node-type="hn-details-summary"><summary>What is Strava?</summary><div data-type="detailsContent">"Strava is an American internet service for tracking physical exercise which incorporates social network features. It is mostly used for cycling and running using Global Positioning System data. Strava records data for a user's activities, which can then be shared with the user's followers or shared publicly."</div></details>

Please note that the project is made up of two parts:

1. a localhost Rust application that handles all the data processing and uploads it to MongoDB's Atlas cloud database
    
2. a web application that displays the cloud data on a map
    

In this article, I will go through some of the implementation details of just the Rust app.

## The data pipeline

The Rust "*backend*" responsible for creating the MongoDB database is comprised of several chained modules, each with a specific goal, mostly of picking up from where the previous module left off and improving the data some more.

![image alt ><](/images/posts/8e12955b-8738-4beb-aa2c-5b62dee1a649.avif)

Data enters the system through the StravaAPI which I've talked about in this [article](./how-to-connect-with-strava-api-using-rust) and then it gets stored in MongoDB which I've also talked about [here](./integrating-mongodb-in-a-rust-application).

The startup sequence of the pipeline creates the dependencies of the modules (database handles and Strava API) and specifies which ones should be run:

<script src="https://gist.github.com/24rush/c4ab918e1e549451afa57c72b31156fe.js"></script>

There are 3 pipeline modules, each being configured through the *PipelineOperationType* flag.

## Activity Syncer

This first module is straightforward, it just checks if the last database sync timestamp for the activities is more than 24 hours away and if it's the case it proceeds to ask Strava for activity ids since that timestamp after which it:

* ***downloads*** those activities and their telemetries
    
* runs two ***sanitization*** submodules, one for updating the activity's location city and country which Strava doesn't fill in correctly and a second fixer for dates that are not in the format Mongo expects
    
* runs an ***index remapping*** algorithm for segment polylines. The sub-module is quite interesting and will explain it briefly. Every activity contains a [polyline-encoded](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) map of the track which is an optimized (lower resolution) way of transmitting all the GPS points of that track to the client. The segments (sections) of an activity contain two fields (*start\_index* and *end\_index*) which indicate where they start and end on that activity's GPS track but these indexes are referencing the whole set of telemetry points, not the ones from the optimized polyline. On the client side, I need to be able to draw these segments by using the activity's polyline so I needed some indexes to tell me where in this polyline that segment is. You can image the polyline as a scaled-down version of the whole GPS data set and these new indexes being the start and end positions of the segments in this polyline.
    
    ![image alt ><](/images/posts/e00cf49e-d53e-40a5-8227-a20757e7518b.avif)
    
    ![image alt ><](/images/posts/c5c2f0ff-bd36-4907-ad5d-b209ea4dc725.avif)
    

The Rust code for this module can be found in [src / processors / sync\_from\_strava.rs](https://github.com/24rush/the-world-covered/blob/develop/src/processors/sync_from_strava.rs)

## Route Matcher (*track de-duplication*)

As I take the same route again and again, especially for running, I wanted to implement a mechanism for finding such "duplicated" tracks. *The outcome seen below is the tool figuring out that I've been on the blue track 52 times (note the 52x badge) and I would then display it as just one track instead of 52 independent ones.*

![image alt ><](/images/posts/ab18c637-6b69-404f-83bc-f9ea80c43978.avif)

The task laying ahead meant I had to process around 22000 kilometers of GPS tracks sampled in 1300 hours of data. The GPS coordinates are returned by Strava's API in the form of arrays of latitude and longitude pairs which were sampled at random time intervals (around 2-10 seconds depending on the sport and the speed).

![image alt ><](/images/posts/31baa25f-d579-459d-907e-ef368871f76f.avif)

The idea I came up with was a mechanism of reducing slightly the accuracy of the coordinates paired with the use of two HashMaps for storing these normalized latitudes and longitudes.

### Reducing the accuracy of sampled GPS points

The GPS data coming in from Strava had an accuracy of 11cm (6 digits) but if I wanted to match similar tracks I had to go lower as:

* I didn't always move on the same path (consider running on the opposite sidewalk of a street or different lanes of a running track)
    
* the sampled GPS points between two identical paths are anyway intertwined, there is no guarantee that they will fall on the exact same point every time
    
* my Garmin's GPS might introduce noise.
    

Therefore, I went with a more permissive allowance of 50m which means I would only pick the first 3 digits from a coordinate and then add 5 x 10^-4. We can think of this as splitting the Earth into 50m x 50m squares and all my coordinates would then fall under one of the corners of such a square. *I am aware of how naive this solution is as the Earth is not a perfect sphere but considering that the length of these tracks is a few kilometers, this doesn't matter.*

The example below shows two tracks that share just one common section:

![image alt ><](/images/posts/d1846b7b-e5cf-47a8-93b3-854c22502bd2.avif)

All the GPS samples from a 50x50m square get snapped to the bottom left corner of that square (*snapping point*), therefore any points falling in there are considered equal - so basically a single point.

The function for reducing the accuracy of a GPS coordinate can be seen below and you may spot that it also returns an integer value for reasons I will explain in the next section.

<script src="https://gist.github.com/24rush/d88653e38cb47ed1831b62226e9ca586.js"></script>

*Ex. 45.473928 gets converted to 45.473****5*** *(3.5 digits) and then to its integer value*

### Identifying similar points

Now that I knew how to merge *close-enough* points, I had to put in place a registry of points and the activities in which they appeared. For this, I used a 2-level HashMap, a fancy way of saying HashMap of HashMap in which the key of the first level map is the latitude and the value is the second level HashMap. This second level map had as a key the longitude and as value the list of unique activities in which the point appeared (so a HashSet). As ***float values aren't hashable in Rust***, I used their integer representation.

![image alt ><](/images/posts/49dd2b4a-1f06-4e86-8298-8e47402f8a87.avif)

The underlying structure, would in my mind look like this.

![image alt ><](/images/posts/1de37972-9cac-47b8-817f-7ddde4f0c90e.avif)

A (lat, long) point would enter the algorithm and get looked up first by its latitude and then by its longitude. The resulting value of this 2nd level HashMap would be a set of activities to which we would add the activity containing this point.

In parallel to the lookup, I would also be counting the number of shared points between the activities already processed. I used a structure that mimics an unoriented graph but is similar to the one above, connecting two activity ids to a number that represents how many GPS points the two have in common.

The resulting structure looks like this:

![image alt ><](/images/posts/1ea8633e-8082-43c7-836a-82f233587fbb.avif)

*Ex: Activity 3 has 3 points in common with Activity 4*

and the associated Rust code:

<script src="https://gist.github.com/24rush/af98c21bfbac3045807435f39b0d58ee.js"></script>

After having processed all the points, I would end up with a list of activity pairs and the number of points they share. I would then use that number to compute a percentage by dividing the number of common points by the number of samples the source activity contained and only if that number would be higher than 85%, I would consider it a 'match'. An extra check that their lengths are proportional was made so I would account for situations where a smaller loop contained in a larger one would not be considered equal.

![image alt ><](/images/posts/dcc4bed1-58c9-4977-9898-4f8881f4f543.avif)

<script src="https://gist.github.com/24rush/020713041ede46d3a4d571f2eb3c932a.js"></script>

The next step was to merge the results above into groups of similar activities. For this, I would try to allocate a pair of incoming activities to an already existing group that contained one of them or create a new group if the activities were not present in any groups yet. In the example below you can see how activity number 3 was allocated in Group 2 because activity 1 (source activity) was already present in that group.

![image alt ><](/images/posts/b9cfcc94-bc9d-493f-b297-3c427b919e55.avif)

The result of the merge would be a list of unique routes and each would contain an array of activity ids representing the duplicated activities they encompass.

For all my Strava activities, the code had to process `496 activities and 2088188 points.`

The complete code for the matching algorithm can be found in ▶️ [src / processors / commonality.rs](https://github.com/24rush/the-world-covered/blob/develop/src/processors/commonality.rs)

## Route Processor

This is the last module in the chain and is responsible for adding the final pieces of information on the generated unique routes by the *Route Matcher*. It consists of the following functions:

* determining the ***center point*** of the activity
    
* determining the ***distance*** from this center point to the map's ***reference point*** (the capital city) - *this is necessary so that the website can incrementally download routes as the user moves the map. I explained this* [*here*](./mapping-all-my-sports-activities-using-rust-leafletjs-and-openai) *in the web app chapter.*
    
* ***finding gradients*** using the *Gradient Finder.* This is a neat little feature that uses telemetry data (altitude, distance, grade) to find climbs (road gradient over 7%) and descents (gradient below -3%) from the tracks I've been on. It's a basic algorithm that traverses the telemetry data and tries to find out the longest sections of the road where the gradients follow the criteria for ascent or descent but it allows for some slack (the road can flatten for a maximum of 600 m along the way) so it doesn't break the chain. This feature is particularly interesting for cycling where you are interested in consistent climbing - the gradient doesn't fluctuate (goes up and down) or in descents...to have some fun.
    
    ![image alt ><](/images/posts/bc49debd-c1b1-4e0a-850a-6325dccb3bf7.avif)
    
    The code for the GradientFinder can be seen in [src / processors / gradient\_finder.rs](https://github.com/24rush/the-world-covered/blob/develop/src/processors/gradient_finder.rs)
    

I hope you enjoyed the article and found it informative. Feel free to share your thoughts in the comments section below and don't forget to follow me on Hashnode and [LinkedIn](https://www.linkedin.com/in/aliniordache/) for more.