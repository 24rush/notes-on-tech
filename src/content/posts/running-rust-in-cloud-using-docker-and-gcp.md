---
title: "Deploying Rust Applications in the Cloud using Docker and GCP"
description: "Optimize Rust apps with Docker and Google Cloud Platform; deploy Rust backend in the cloud, prepare code, and run Docker containers seamlessly on GCP"
short_desc: "My Rust-based Strava visualizer app, which I've explained in detail previously, was lacking an important feature: the ability to automatically save my Strava activities"
date: 2023-07-28T05:00:00Z
image: "/images/posts/cover_hashnode_4.png"
categories: ["Programming"]
authors: ["Alin"]
tags: ["Rust", "GCP", "Docker"]
draft: false
---
My Rust-based Strava visualizer app, which I've explained in detail [previously](./mapping-all-my-sports-activities-using-rust-leafletjs-and-openai), was lacking an important feature: the ability to automatically save my Strava activities in the database as soon as I completed them. This issue existed because my Rust backend was hosted locally, requiring me to manually run it to update the MongoDB database. However, this is no longer the case, as I am proud to have hosted my entire Rust backend in the cloud.

At the time of this writing, the only viable solution for running Rust in the cloud is by wrapping it in a **Docker** container and hosting it on various platforms that support it and I chose **GCP**.

In this updated architecture diagram, you can now see box 7 which depicts the Docker container hosting the Rust backend.

![image alt ><](/images/posts/7f100cfe-27dc-49d7-abeb-a08edfe70113.avif)

## Preparing the Rust code for Cloud

*Prerequisites: these steps assume that you have Docker installed and configured on your host machine (in my case I am using Ubuntu 22.04)*

1. Create a **Dockerfile** and .**dockerignore** in the root of the Rust project:
    
    *The .dockerignore file is very important as it excludes all the unnecessary Rust build files (target folder) from the Docker image.*
    
<script src="https://gist.github.com/24rush/e0e15cbc31659b7df69231b46f94d8af.js"></script>

<script src="https://gist.github.com/24rush/ba29c73ee3cab9181dffec94f1fc62be.js"></script>

1. Build the container:
    

> **docker build .**

You should see in the terminal: *Successfully built <IMAGE_ID>*

1. If the above step fails, you might need to tweak **Cargo.toml** and/or **Dockerfile.**
    
    **In my case**, the docker build had two issues that were not present in the cargo build and the fix was to update these files like so:
    

![image alt ><](/images/posts/4df8ce79-db0b-4cd7-91e7-d33a6a6f3f53.avif)

In line **10**, I had to enable the feature '*serde*' on geo-types as it was causing a build issue saying the *Serialize* trait is not implemented for type **Coord**

![image alt ><](/images/posts/d2e9033c-b4f8-4b27-929b-302d3047b722.avif)

In line **17**, a more complicated issue related to building *openssl* that is explained [here](https://stackoverflow.com/questions/69412947/error-failed-to-run-custom-build-command-for-openssl-sys-v0-9-67) and [here](https://github.com/sfackler/rust-openssl/issues/1021) and how I fixed it is by also installing the build tools required for openssl (line **2** in Dockerfile)

**Takeaway**:

* Rust default features might not be applied consistently and you will need to manually specify them in Cargo.toml
    
* some dependencies might be missing from the docker image which the host already has
    

1. Run the container:
    

> docker run <IMAGE_ID>

At this point, the binary specified in the **CMD** section of the Dockerfile should have run and exited or be listening for connections if you are raising a Rust HTTP server (in which case you can test it by sending requests from the host)

### Other issues you might encounter:

1. Permission errors while trying to interact with the docker daemon require adding your username to the docker group:
    
    > sudo usermod -a -G docker \[user\]    
    > newgrp docker
    
2. If you are starting an HTTP server inside the Docker container but you cannot reach it from the host (even though port forwarding is setup correctly) make sure your server configuration is not set up to listen on (local) 127.0.0.1 instead of 0.0.0.0:
    
    ![image alt ><](/images/posts/6bc02b33-068b-44d0-8bd7-090f0380c112.avif)
    

## Running the Docker container in GCP

Now that you have the Docker image for the Rust code, you need to upload it to GCP and a good starter article is the one [here](https://cloud.google.com/blog/topics/developers-practitioners/lifecycle-container-cloud-run).

You can see there are two entities involved: **CloudRun** and **Artifact Registry**. CloudRun is the Google service that will handle your container and Artifact Registry is the one storing the images for it.

1. Create a **Google Cloud** account if you don't have one
    
    *This will require adding a billing account but you will not be charged anything unless you opt-in for a paid account. The free tier is exhaustive and you will not need a paid account unless you need a lot of firepower.*
    
2. Install the gcloud CLI by following the official instructions [here](https://cloud.google.com/sdk/docs/install#deb)
    
3. Go to [https://console.cloud.google.com/](https://console.cloud.google.com/) and create a new project
    
4. Go to [https://console.cloud.google.com/artifacts/create-repo](https://console.cloud.google.com/artifacts/create-repo) and create a new Artifact Registry repository for the newly created project
    

![image alt ><](/images/posts/3e22004a-2b2a-4192-a902-09af9df35611.avif)

5. Go back to the terminal and create a tag for the docker image and push it. Official instructions are [here](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling) but the commands you need to run are:
    
    > docker tag <IMAGE_ID>\<REGION\>-docker.pkg.dev/<PROJECT_NAME>/<REPO_NAME>/<SOME_IMAGE_NAME>

    > docker push \<REGION\>-docker.pkg.dev/<PROJECT_NAME>/<REPO_NAME>/<SOME_IMAGE_NAME>
    - image_id - the id of the docker image you built     
    - region - the region you selected in the step above     
    - some_image_name - some name you can use for future reference

If you are unable to push to Artifact Registry due to **missing permissions**, check *Policy Troubleshooter* service to see if you indeed are missing those permissions and add them into a role to your Principal if so.

![image alt ><](/images/posts/930c76c3-d4f6-4734-9737-c9e237cf2b59.avif)

1. Go to [https://console.cloud.google.com/run](https://console.cloud.google.com/run) and create a service for running the container:
    * select the **container's image** from Artifact Registry (the one you uploaded in the previous step)
        
    * select **Allow unauthenticated invocations** if you want to access your service from the public internet
        
    * make sure you select the **same region** for the service as you did for the image in Artifact Registry
        

![image alt ><](/images/posts/2f115a92-5d44-4a0e-8db8-e09ed034b248.avif)

2. You should now see your service URL like so 🎉:
    ![image alt ><](/images/posts/a31d9f7b-9b89-466f-bb03-74e3c17bc1f9.avif)
    

Making HTTP requests to this URL will either run the CMD binary you specified in Dockerfile or if that binary is a Rust HTTP server, it will forward it to that server which basically means you can have multiple endpoints specified.

Writing this article, it all seems so easy but I did have some headaches when first doing this process. I hope this material can make your life easier if you plan on taking this route.