---
layout: post
title: "Building a Cloud-Agnostic Platform for Dancers While Avoiding Managed Services"
description: "How I designed, built and operate a self-hosted platform as a solo engineer, making deliberate trade-offs around cost, reliability, simplicity and operational ownership."
date: 2026-07-07 17:38:00 +0000
categories: projects dancehub
published: false
---

How I designed, built and operate a self-hosted platform as a solo engineer, making deliberate trade-offs around cost, reliability, simplicity and operational ownership.

<br/>

<div style="background-color: #fff9e6; border-left: 4px solid #f59e0b; color: #78350f; padding: 16px; margin: 20px 0; border-radius: 4px;" markdown="1">
💡 **Work in Progress**  
This page is currently under construction. A few pieces are missing and I might adjust small bits.
</div>

<br/>


# Introduction

What is this about? Let me start with, what it is not. This is not a proof of concept, it's not a pet project that goes to the code graveyard. It's also not AI slop, or vibe coded. So am I not using AI at all? Quite the contrary, I'm heavily using AI, but in a controlled way, I know what is going on under the hood, this is called AI assisted development. In fact, without AI I wouldn't be able to have developed this platform within 6 month, in my free time, while having a full time job, all by myself. In a way, AI enables me to become the 10x engineer that we all want to be. Maybe 10x is slightly exaggerated, proabably more like 2x-3x? My point is, I can move much quicker than before.

**So, what is it then?** In simple words, it's a platform for dancers where they can find all kinds of events in one place. While it's still early stages, this is production grade quality, or can I just say made in Germany? Well that would be a lie, I live in London. What about *made by a German*? You'll get the point, it's German quality.

**Why am I doing this?** For one I love building things, I can put all my knowledge together into one project. From a dancers perspective, this is something to enrich the dance community, not just for salsa and bachata, this is a place for all dance styles.







# Design Principles
- Cloud agnostic
- Open source first
- Self-hosted where practical
- Low cost and able to exist without funding
- Simplicity over complexity
- Operable by one person
- Scale when required

This list has influenced all technical decision in the project.

sustainable

The goal wasn't to build the most sophisticated architecture possible. The goal was to build a useful product while making deliberate trade-offs around cost, reliability, operational ownership and long-term maintainability.


# Architecture Overview

- show diagram
- explain user request flow
- ci cd flow

## Why One VPS Is Enough (For Now)
- Acknowledge SPOF
- Explain trade-offs
- Cost vs complexity
- Avoiding premature optimisation

Let's address the elephant in the room, this runs all one a single server? I know I know, it's a single point of failure, but honestly this is all I need right now.

...

Buuut, just because this is all I need right now, I haven't phantasized about the dream setup, check this out -> Let's Scale Up



## Why I Chose a Monolith
- Simplicity
- Team size
- Operational overhead
- Trade-offs

When I first started thinking about the archtiecture, I was dreaming of microservices, all written in Go, autoscaling with Kubernetes and all that fancy stuff. But do you know the complexity of distributed systems? Just think about deployments, networking, monitoring, or debugging. Every service boundary eventually becomes an operational burden.

I want to move fast, and keep things simple (even though difficult is more fun lol), and I'm the only engineer working on the platform, so let's stay realistic and keep the fancy stuff away. 

(Well.. for now, because knowing me, I would happily move to Kuberentes and make evertyhing even more complicated, and of course I'll manage the cluster myself, everything else would be boring)



## No managed services?

I'm not fully avoiding them, for example I use GitHub Actions or Docker Hub (free tier) to make my life easier, but the actual platform is free of managed services.

And just to clarify this, I'm not against managed services, they're great and in many cases it's a smart move to use them! For example using GKE or EKS over a self hosted Kubernetes cluster. With those services you eliminate the operational burden of managing complex control planes, scaling infrastructure, and other things that I might not know about yet.

So why am I not using Supbase, Appwrite, Clerk, Auth0, Firabase etc. and push everything to Vercel and I'm done? Everyone does that? Doing everything myself is so much more operational overhead? Exactly, this is the whole point, this is where the fun begins.

![this is where the fun begins](https://tenor.com/uqNk.gif)

<div class="tenor-gif-embed" data-postid="4830492" data-share-method="host" data-aspect-ratio="2.08333" data-width="100%"><a href="https://tenor.com/view/this-is-where-the-fun-begins-star-wars-anakin-sassy-excited-gif-4830492">This Is Where The Fun Begins Star Wars GIF</a>from <a href="https://tenor.com/search/this+is+where+the+fun+begins-gifs">This Is Where The Fun Begins GIFs</a></div> <script type="text/javascript" async src="https://tenor.com/embed.js"></script>

Yes I want to move fast and this slows me down, but that's okay, I'm not a startup that has to reach milestones for more funding.

A few other reasons why I choose to go the much-more-work way:
- Unpredictable costs, once you leave the free tier it can get quite expensive
- Vendor lock-in, they can change their rules at anytime
- Data sovereignty, I want to be in control of the data and eventualy be GDPR compliant
- Architectural control, while I'm being far away from optimising every bit, this could be a limitation if you're working with a black box

## Tech Stack

### Database (Postgres)

When I started prototyping I was using MongoDB, just because I thought with NoSQL I can stay flexible and not be restricted by any schema, as I'm changing things constantly. The truth is, you just move that contract onto the app layer, now everytime my app reads from the database, I have to do all kinds of checks, if, else, are you a list? what data are you? There is no guarantee documents are all the same, there are so many ways to mess this up.

Now with Postgres, I'll define the schema upfront, the data that goes in has to follow that contract, that means the data that comes out is consistent, I don't need to do any crazy validations. As long as the app knows how to read the data, there is no way to get it wrong.

And one thing that made me really happy as well, was the native UUID v7 support coming along with Postgres 18.


### Mobile app (Flutter)
That was an easy choice to make, I needed something that makes app development for Android **and** iOS simple. I had never used Flutter or written a single line of Dart, so I had to learn this from scratch, but it's very similar to Java or C#.

### Backend (FastAPI)
My initial goal was to go with Go (lol), I'm just not familiar with the language. I had written some Go before, but it was more like "I want to do x, let me google how to do x". To speed things up, I decided to start prototyping with Python and switch later to Go. Well I sticked to my beloved Python code as I'm already learning Flutter.

But why FastAPI and not Django or Flusk? I was looking for simplicity and performance, FastAPI is build on Starlette and Uvicorn, which gives me `async`. Why is that so good? In synchronous coding, when a request waits for a slow operation, like a database query or an external API call, the entire execution thread stops and waits. With async, the execution thread immediately moves on to serve the next request while waiting for the slow operation to finish.

[Click here for more FastAPI fundamentals](https://dev.to/kfir-g/understanding-fastapi-fundamentals-a-guide-to-fastapi-uvicorn-starlette-swagger-ui-and-pydantic-2fp7)


### Object storage (Garage)
This is something I completely overlooked, I forgot that I need images for my events, or user profiles, but where do I store them? Object storage of course, but still, wheeere? There are too many solutions, following my design principles, I boiled it down to 
- MinIO, sounded like the perfect solution until I realised they're not open source anymore
- Ceph (RGW), enterprise-grade, massive-scale storage and notoriously complex to set up and manage, no thanks
- RustFS, quite new but it's in alpha stage, might be not production ready?
- SeaweedFS, that seemed quite promising actually, until I found out that it's more complicated to deploy
- Garage, hugely popular in the selfhosted community, easy and lightweight, s3 compatible -> yeap Garage it is




### Reverse proxy (Caddy)
Which proxy do I choose, this was a battle between Traefik, HAProxy, Nginx and Caddy. I'm not going into details here, this blog post [reverse-proxy-showdown](https://hostim.dev/blog/reverse-proxy-showdown/) describes the differences quite well, and I choose simplicity.



### Cloud Service Provider
This research took a long time, there are plenty of options and so many differences in terms of pricing and what they have to offer in general. I actually had my prototype running on GCP, but went for Hetzner, simply because of the low compute and storage cost, infrastructure in Germany -> GDPR check. And lastly Germany is central Europe, that should keep the latency low for everyone (well only for Europeans of course).


### Grafana/Prometheus
Industry standard, we use it at work, I already know how to use it, it does what I need, I'll use it, me happy.


## Why not Kubernetes?




While I'm not using any specific servers from any cloud provider, 
While Terraform makes it easy to switch between providers, I would still need to adjust the code to make it work for that specific provider. But that is only a minor annoyance, and the main goal to avoid vendor lock in is achieved.



## Security