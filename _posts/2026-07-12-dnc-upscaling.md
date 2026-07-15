---
layout: post
title: "Platform Upscaling"
date: 2026-07-12 15:56:00 +0000
categories: projects dancehub
published: true
---

<div style="background-color: #fff9e6; border-left: 4px solid #f59e0b; color: #78350f; padding: 16px; margin: 20px 0; border-radius: 4px;" markdown="1">
💡 **Work in Progress**  
This page is currently under construction. A few pieces are missing and I might adjust small bits.
</div>

<br/>


# Stage 1
As prerequisite for all following steps, we will provision one dedicated node per service.

## Backend
- Scale FastAPI vertical first, maybe 4 cores per instance?
- Add more FastAPI workers, as rule of thumb: (2 * cores) + 1, 9 workers sounds reasonable
- Scale horizontal by adding more FastAPI nodes, let's say 3?

## Load Balancer
- Load balance traffic, add FastAPI nodes to Caddyfile (manually for now)
- Let's go with round robin, as I don't have a large file uploads (it happens in Garage anyway), and I don't have long lasting database queries, if that changes we can still switch to least connection
- Add health checks to Caddyfile to avoid sending traffic in case a node crashes
- Tune Caddy by incresing open file limits to avoid "too many open files" errors
- Add a second Caddy node for high availability

Having multiple Caddy instances introduces a new problem, each instance will try to issue the SSL certificate. I need to store it in a shared place rather then on the individual node, maybe in Redis?

## Object Storage
- Set up Garage 3 way replication for high availability
- Add CDN (Cloudflare?) to reduce Garage load

## Database
- Tune postgres, add connection pooler(PgBouncer): PostgreSQL allocates a full operating system process to every single user connection, which eats up RAM. Placing PgBouncer in front of Postgres to pool and reuse connections can instantly double your database's throughput
- Increase Postgres memory usage
- Add more RAM and CPU cores to the node
- Optimise for reads, by adding read replica nodes
- Configure FastAPI to use replicas for reads
- Partioning, I already partioned events by months, but this could be optimised by moving old data to cold storage
- Sharding? Nah, I don't think this is necessary, as most of the traffic is reading

I feel quite inspired by how OpenAI has scaled their postgres, it's worth reading their blog post https://openai.com/index/scaling-postgresql/

## Caching
- Add Redis cache to take even more load off the database and increase performance

## Service Discovery
- To avoid maintaining Caddyfiles and manually add/remove nodes, I could introduce Consul for dynamic service discovery



<br/>

---

# Stage 2 - Multiple Locations

We're going multi-cloud! 

To allow my applications to talk to each other in a secure way, we have to introduce a VPN mesh, by installing Tailscale on each node. The setup will be the following:

One main location, somewhere central Europe for low latency, where the primary database will live, and two edge locations to absorb heavy read traffic. For the main location I'll choose Frankfurt, Helsinki and Barcelona as edge location.

The main location will get:
- Postgres primary, 1 node (high RAM/CPU)
- Redis primary, 1 node
- Garage storage, 1 node

The edge locations:
- Postgres read replica, 1 node per location (asynchronous replication from main location)
- Redis replica, 1 node per location
- Caddy reverse proxy, 2 nodes per location (for local high availability)
- FastAPI, 3-5 containers per location
- Garage storage: 1 node per location (joined to the master cluster to cache and server images locally)


## Data Flow
How does the data flow? Let's say a user in Barcelona requests event information, and then updates his profile.

The read path, probably 90% of the traffic:
`User -> CDN -> Caddy -> FastAPI -> Redis -> Postgres read replica` (instant response, zero trip to Frankfurt)

The write path, probably 10% of the traffic:
`User -> CDN -> Caddy -> FastAPI -> cross-region network trip -> Frankfurt primary Postgres` (safe write, asynchronously synced back to Barcelona a few milliseconds later)

Or in other words, read requests are served locally, and write requests are sent through a secure VPN mesh from one cloud to another. Most of the image reads will be handled by the CDN. With that setup it will be easy to scale up the platform by adding more edge locations.



<br/>
---


# Further Improvements

## Maximum Transmission Unit
I need to lower the MTU for my multi-cloud network. The standard size for a package is 1500 bytes, and before a package leaves a server it gets intercepted by Tailscale and adds its own data headers to the packet. Now the package is like 1580 bytes and therefore too large, and gets split into two packages, all of a sudden I have double the traffic to deal with, causing extra network overhead and spikes my database query latency. To fix this I can lower the MTU to 1420, if the VPN package overhead is 80 bytes, then I'm back at 1500, no packet fragmentation is triggered.

## Failover
Introduce automated failover, if the primary goes down, a read replica gets promoted to primary

## Queues
Add queues, for example for asynchronous image processing, or in general to absorb request spikes



<br/>
---


# Next steps

<div class="tenor-gif-embed" data-postid="7528020969413377685" data-share-method="host" data-aspect-ratio="1" data-width="100%"><a href="https://tenor.com/view/i-cant-wait-to-see-you-silly-funny-funny-dance-funny-as-hell-gif-7528020969413377685">I Cant Wait To See You Silly GIF</a>from <a href="https://tenor.com/search/i+cant+wait+to+see+you-gifs">I Cant Wait To See You GIFs</a></div> <script type="text/javascript" async src="https://tenor.com/embed.js"></script>
Kubernetes?

<br/>

<div class="tenor-gif-embed" data-postid="19710542" data-share-method="host" data-aspect-ratio="1.52381" data-width="100%"><a href="https://tenor.com/view/meonly-gif-19710542">Meonly GIF</a>from <a href="https://tenor.com/search/meonly-gifs">Meonly GIFs</a></div> <script type="text/javascript" async src="https://tenor.com/embed.js"></script>
But first we should do some math, figure out how much RAM/CPUs are required for each service to run smoothly, and figure out how many users or requests can be served at the same time.


<br/>
---


# Observability
More importantly, we haven't talked about observability, I'll cover this in the next post.