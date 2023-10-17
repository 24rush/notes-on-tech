---
title: "Strava Webhook Events API: Get Notified on User Uploaded Activities"
description: "Optimize Strava Webhooks SEO: Receive instant notifications for user activities; effortlessly monitor cycling and running"
short_desc: "In a previous article, I discussed how I put all my Strava activities on a single map"
date: 2023-07-28T05:00:00Z
image: "/images/posts/7f903737-2787-4917-b10e-0e78cc3ab684.avif"
categories: ["Programming"]
authors: ["Alin"]
tags: ["Strava", "Rust", "Vercel", "Typescript"]
draft: false
---
In a [previous](./mapping-all-my-sports-activities-using-rust-leafletjs-and-openai) article, I discussed how I put all my Strava activities on a single map and in this one, I'm gonna show you how I'm getting notified by Strava whenever a user uploads a new activity (*this assumes that you already have a Strava API application created for which the user allowed the read of his activities).*

<details data-node-type="hn-details-summary"><summary>What is Strava?</summary><div data-type="detailsContent">"Strava is an American internet service for tracking physical exercise which incorporates social network features. It is mostly used for cycling and running using Global Positioning System data. Strava records data for a user's activities, which can then be shared with the user's followers or shared publicly."</div></details>

The official documentation can be found [here](https://developers.strava.com/docs/webhooks/) but in summary, what you need to do is the following:

1. Issue a manual subscription request to Strava, specifying the callback URL you want to be notified on whenever the user performs an action on Strava
    
2. Respond to the incoming validation request for that URL
    
3. After the successful validation of the callback, accept incoming requests whenever users perform those actions
    

![](/images/posts/7f903737-2787-4917-b10e-0e78cc3ab684.avif)

To handle the scenario described above, you will need access to a terminal or Postman to make an HTTP request to Strava to initiate the process. Additionally, you will require a server application capable of responding to Strava's HTTP requests; in my case, this is hosted by Vercel and written in Typescript.

## Server-side code

Create a server component that implements the endpoint for the callback URL which Strava will call on:

<script src="https://gist.github.com/24rush/82161b0c62299e5f545e7c7d133eb34d.js"></script>

the code besides creating the ***MY\_SERVER\_URL/api/activity\_update*** endpoint also does the following:

1. **Handles callback validation requests** *(lines 14-21)*

    Immediately after you submit your request for registering a new notification callback, Strava will issue a validation HTTP request to the URL you specified which will contain the query parameter ***hub.challenge***. To successfully validate this callback, in the response body of the request you need to reply with the value of this query parameter.
    
    Strava's request:
    ```bash
    GET https://<my_callback_url> hub.verify_token=STRAVA&hub.challenge=15f7d1a91c1f40f8a748fd134752feb3&hub.mode=subscribe
    ```
    *Your response must be:*
    
    {"**hub.challenge": "15f7d1a91c1f40f8a748fd134752feb3"}**
    
2. **Handles notifications for user-uploaded activities** *(lines 27-38)*
    
    Strava's API supports notifications for multiple actions of the user, like whenever he performs an update on the title or description of an activity but as I'm not interested in this kind of update, just on the creation and deletion of activities, I will not handle it.
    
    The important fields of the request that describe the user action performed:    
    <table><tbody><tr><td colspan="1" rowspan="1"><p><strong>object_type</strong><br>string</p></td><td colspan="1" rowspan="1"><p>Always "activity" or "athlete."</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>object_id</strong><br>long integer</p></td><td colspan="1" rowspan="1"><p>For activity events, the activity's ID. For athlete events, the athlete's ID.</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>aspect_type</strong><br>string</p></td><td colspan="1" rowspan="1"><p>Always "create," "update," or "delete."</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>owner_id</strong><br>long integer</p></td><td colspan="1" rowspan="1"><p>The athlete's ID.</p></td></tr></tbody></table>
    
    In my case, I will only be handling requests that contain the values 'create' and 'delete' for **aspect\_type** and 'activity' for **object\_type**. If the aspect\_type is 'create' then I know the user uploaded a new activity and the id of that activity is stored in **object\_id**. I can then use Strava's API to retrieve the activity based on its id.
    

## Manual registration step

To start the callback registration process, you need to issue a request like so:

```bash
curl -X POST https://www.strava.com/api/v3/push_subscriptions \
      -F client_id=5 \
      -F client_secret=7b2946535949ae70f015d696d8ac602830ece412 \
      -F callback_url=MY_SERVER_URL/api/activity_update \
      -F verify_token=STRAVA
```

notice:

- **client\_id** & **client\_secret**: your Strava API application client ID and client secret which you can see in the app's settings page

- **callback\_url:** the URL where you want to be notified on

- **verify\_token** *(optional)*: extra payload you can specify in case your application has some special logic for registering the callbacks

Once you trigger this request, Strava will start the validation request process specified above so your server needs to be ready to handle it. If the validation goes through, the response of this request should be something like {"id":12345} where the number is your subscription id. If that is the case, you will now see that whenever a user uploads an activity, your server will get notified.

## Deleting a subscription

If you don't want to be notified anymore, issue the following request:

```bash
curl -X DELETE https://www.strava.com/api/v3/push_subscriptions/12345 \
    -F client_id=5 \
    -F client_secret=7b2946535949ae70f015d696d8ac602830ece412
```

Hope this clarifies what you need to do to get notified whenever your application's users perform an action that you are interested in.