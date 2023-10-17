---
title: "How to Connect with Strava API using Rust"
description: "Connect with Strava API in Rust: Authenticate, access, automate API calls, master manual/automated OAuth with limited crate availability"
date: 2023-06-30T05:00:00Z
image: "/images/posts/a9d311f6-97c1-4c5c-b9b3-ac5f29e69c83.avif"
categories: ["Programming"]
authors: ["Alin"]
tags: ["Strava", "Rust", "OAuth"]
draft: false
---

Implementing this in **Rust** is somewhat challenging since, at the time of writing this article, there were no crates available that could accomplish everything I wanted. As a result, I had to implement most of it myself. However, this was a welcome part of my Rust learning process.

<details data-node-type="hn-details-summary"><summary>What is Strava?</summary><div data-type="detailsContent">"Strava is an American internet service for tracking physical exercise which incorporates social network features. It is mostly used for cycling and running using Global Positioning System data. Strava records data for a user's activities, which can then be shared with the user's followers or shared publicly."</div></details>

The [Strava Developers](https://developers.strava.com/docs/getting-started/) website does an ok job of explaining how to access the API but the article that put all the pieces together for me was this [one](https://jessicasalbert.medium.com/holding-your-hand-through-stravas-api-e642d15695f2). The challenging part was figuring out the **OAuth** procedure for having access to the data and then automating the procedure so I could use it any time I needed new data to be downloaded from Strava.

The gist of it is that you need to take one manual step to authorize the Strava app to read the current athlete's activities (in my case that is me who I'm allowing the app to read my activities) and after you get the authorization tokens from Strava, you will only need to run a refresh procedure for these tokens which I called automated procedure.

## Authorization procedure

### Manual steps:

1. Go to [https://www.strava.com/settings/api](https://www.strava.com/settings/api) and create a new Strava app
    
    > you get the Client ID, Client Secret which should be kept secret
    
2. Go to [http://www.strava.com/oauth/authorize?client_id=**<CLIENT_ID>**&response_type=code&redirect_uri=http://localhost/exchange_token&approval_prompt=force&scope=read,activity:read_all,profile:read_all,read_all](http://www.strava.com/oauth/authorize?client_id=**<CLIENT_ID>**&response_type=code&redirect_uri=http://localhost/exchange_token&approval_prompt=force&scope=read,activity:read_all,profile:read_all,read_all)
    > notice the scope permissions the app is requesting
    
3. Authorize the app
    
4. Strava will redirect to a URL that contains the authorization code under the ***&code*** parameter
    
    > this is your authorization code
    
5. Exchange the authorization code for your first access token by doing a POST request (if using curl, use -F for parameters) to [https://www.strava.com/oauth/token](https://www.strava.com/oauth/token) including the following parameters:
    
    ```bash
    client_id=<CLIENT_ID>    
    client_secret=<CLIENT_SECRET>    
    code=<AUTHORIZATION_CODE> (from step 4)    
    grant_type=authorization_code
    ```
    
    > you will get a JSON response that includes the 'access_token', 'refresh_token' and 'expires_at' fields. Note that the default expiration for the tokens is 6 hours so the expire_at timestamp you get is 6h in advance from the first generation. Subsequent calls will return the same expires_at until they expire.
    
6. You can now make requests to Strava by using the *access\_token* as your Bearer like so:
    ```bash
    curl -X GET "https://www.strava.com/api/v3/activities/8995856626/streams?keys=latlng,altitude" -H "Authorization: Bearer <access_token>"
    ```

The end of the manual procedure consists in having the *access_token, refresh_tokens* and *expires_at* stored somewhere in your app. The automated procedure involves refreshing these tokens when the current timestamp is larger than their *expires_at*.

### Automated steps:

1. Check if the current timestamp (*Datetime.now*) is larger than the *expires\_at* value. If not, then use the *access\_token* as Bearer. If the *expires\_at* field points to the past then go to step 2.
    
2. Request refreshed tokens by doing a POST request (if using curl, use -d for parameters) to [https://www.strava.com/api/v3/oauth/token](https://www.strava.com/api/v3/oauth/token) including the following parameters:
    
    ```bash
    client_id=<CLIENT_ID>    
    client_secret=<CLIENT_SECRET>    
    grant_type=refresh_token
    refresh_token=<refresh_token>
    ```
    
    > you will get a JSON response that includes the 'access\_token', 'refresh\_token' and 'expires\_at' fields. These will need to be updated in your database
    

A few details on the implementation:

1. Use a (*.gitingore*) secrets file to store all the data marked as 'secret'. That is the Client ID, Client Secret. Don't deploy them anywhere in public. As this procedure makes the most sense to be done on a backend, you can store this file safely on your server.
    
2. Store the tokens (access and refresh, alongside their expiration timestamp) in the persistent layer of your choice. As this might need to be checked at runtime every time you make a request maybe it's worth putting them somewhere with fast access.
    
3. Strava imposes [rate limits](https://developers.strava.com/docs/rate-limits/) of 200 per 15 minutes and 1000 requests per day so keep that in mind when implementing your app. Dashboards below are available in your Strava Developers account under *My API Application*.
    ![image alt ><](/images/posts/77bb6b7b-849a-4b27-b8eb-c781abe04713.png)
    

## Trigger API requests

Once you have this procedure set up, it means that you will be ready anytime to trigger requests to Strava's API. The Rust code has a generic GET request implementation (that outputs JSONs) and a multitude of other functions that use it to request a particular endpoint from Strava (like activities, streams, telemetry, etc.)

<script src="https://gist.github.com/24rush/d08bcab0cf635d69d0da447071be5487.js"></script>

By using the function above, you can create additional getter functions for the particular data you might need.

<script src="https://gist.github.com/24rush/0b7e32d59cec5d45160f8e8934e74821.js"></script>

The complete implementation can be found in [src / strava / api.rs](https://github.com/24rush/the-world-covered/blob/develop/src/strava/api.rs) and includes the mechanism for reading the tokens from the secrets TOML file and also the token refresh function.

This article showed how you can authenticate to Strava API and make calls to get the JSON data your app requires. In my [other articles](./mapping-all-my-sports-activities-using-rust-leafletjs-and-openai), I showed how I persisted this data into MongoDB and also how I'm putting it all together in a web app capable also of searching the data using OpenAI's SQL translate API.

Thank you for reading and stay tuned ✌️