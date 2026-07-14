---
layout: post
title: "Platform upscaling"
date: 2026-07-12 15:56:00 +0000
categories: projects dancehub
published: true
---

How I designed, built and operate a self-hosted platform as a solo engineer, making deliberate trade-offs around cost, reliability, simplicity and operational ownership.

<br/>

<div style="background-color: #fff9e6; border-left: 4px solid #f59e0b; color: #78350f; padding: 16px; margin: 20px 0; border-radius: 4px;" markdown="1">
💡 **Work in Progress**  
This page is currently under construction. A few pieces are missing and I might adjust small bits.
</div>

<br/>



# Stage 1



- add redis cache to take load from the database
- one node for each service
- scale fastapi vertical first, add more CPU cores, let's say 4 cores, then add more workers, as rule of thumb: (2 * cores) + 1, so 9 workers
- scale horizontal by adding more nodes for fastapi, let's say 3
- load balance traffic, hardcode nodes into config for for now, add health checks to stop sending traffic in case a node crashes, 
    round robin vs least connection?
    let's go with round robin, as I don't have a file large uploads (it happens in Garage anyway), and I don't think I have long lasting db queries?
    if that changes we can still switch to least_conn

- add CDN for Garage
The CDN: Your storefront. It intercepts 99% of user download traffic so your 3 Garage nodes can sit quietly, using very little CPU and RAM.
- use 3 nodes for Garage
The 3 Garage Nodes: Your vault. They guarantee that your images are safely written to disk and never lost, even if one server explodes.

- for Garage we enable 3 way replication


Caddy
- scale vertical first, by also incresing open file limits to avoid "too many open files" errors
- scale horizontal, 3 nodes, configure CDN (maybe Cloudflare?), enable DNS round-robin by creating multiple A records for my domain, each pointing to one Caddy node IP
- That introduces a new problem, each instance will try to issue the SSL certificate, so I need to find a way to store it in shared place? Redis?


Database

Phase 1: Optimise first performance
- add a Connection Pooler (PgBouncer): PostgreSQL allocates a full operating system process to every single user connection, which eats up RAM. Placing PgBouncer in front of Postgres to pool and reuse connections can instantly double your database's throughput
- Tune postgresql.conf Settings: Default Postgres settings are intentionally ultra-conservative. Optimize these three key variables to match your system's RAM:shared_buffers: Set this to 25% of your total server RAM.work_mem: Increase this (e.g., 64MB–256MB) so complex sorts happen in RAM instead of spilling onto the slow hard drive.effective_cache_size: Set this to 75% of total server RAM.

https://www.tinybird.co/blog/postgresql-horizontal-scaling
https://www.youtube.com/watch?v=0siW6E6eEgg A Roadmap To Scaling Postgres | Scaling Postgres 361




Phase 2: Vertical Scaling (Scale-Up)

The simplest, most reliable way to scale Postgres is to give it a larger virtual machine.The Action: Move to a cloud instance with more CPU cores, higher RAM, and ultra-fast NVMe SSD storage.Why it's preferred: It requires zero changes to your FastAPI application code. Postgres loves RAM because it allows the database to cache your entire index and active dataset directly in memory, reducing slow disk reads

https://www.velodb.io/glossary/ways-to-scale-postgresql


Phase 3: Horizontal Scaling (Scale-Out Options)

When a single massive machine runs out of resources, or your write/read volumes become too heavy, you must split the database across multiple machines. Your exact option depends entirely on what your bottleneck is:

Option A: Read Replicas (For Read-Heavy Applications)If your primary bottleneck is users executing heavy search queries or loading dashboards, use Streaming Replication
- How it works: You run one Primary node (handles all INSERT, UPDATE, DELETE operations) and one or more Read Replicas (constantly mirror the primary node via WAL logs).
- FastAPI Integration: You configure your FastAPI backend or database ORM (like SQLAlchemy) to send all write queries to the Primary database IP, and distribute all read queries across the Read Replica IPs.


Option B: Table Partitioning (For Massive Tables)
If your database handles millions of rows but data is predictable (e.g., logs or orders organized by date), use Postgres's native Table Partitioning.
- How it works: A massive 500GB table is transparently split into smaller, hyper-efficient monthly chunks (e.g., orders_2026_06, orders_2026_07) on the same machine.
- Extension to look at: Use the pg_partman extension to completely automate the creation and maintenance of these historical data chunks


I definitely need to optimise for data reads, rather then writes, so sharding is not interesting yet

I would go the same path that open has gone, https://openai.com/index/scaling-postgresql/





# Optimising Service Discovery
To avoid maintaining Caddyfiles
- I could use an internal DNS server to map my FastAPI nodes to a single domain name
- I could use a Docker Swarm cluster, which is essentially an internal DNS as well, just on the Docker side
- Caddy has an admin API that I could use to dynamically register the API on startup

At some point I have used Consul to add Service Discovery, I guess I would have to do some more research on which option is the best. But this optimisation is optional anyway.



Summary
- We have introduced Redis to take load of the database
- Each service on it's own node
- Vertically upscaled nodes (where it makes sense)
- Added CDNs to take load of Garage
- Garage 3 way replication
- Scaled FastAPI horizontally, optimised workers
- Optimised Postgres connection pooling
- Introduced read replica to take load of the primary instance
- Optimised Caddy by increasing open file limits
- Scaled Caddy horizontally
- Introduced service discovery




# Stage 2 - Multiple locations

The CDN setup stays the same, through Anycast the client will be routed to the nearest datacentre

https://www.cloudflare.com/learning/cdn/glossary/anycast-network/



Okay we're going multi-cloud! To allow my applications to talk to each other in a secure way, we have to introduce a VPN mesh, by installing Tailscale on each node.

The setup will be the following:

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


How does the data flow

Let's say a user in Barcelona requests event information, and then updates his profile.

The read path, probably 90% of the traffic:
User -> CDN -> Caddy -> FastAPI -> Redis -> Postgres read replica (instant response, zero trip to Frankfurt)

The write path, probably 10% of the traffic:
User -> CDN -> Caddy -> FastAPI -> cross-region network trip -> Frankfurt primary Postgres (safe write, asynchronously synced back to Barcelona a few milliseconds later)

Or in other words, read requests are served locally, and write requests are sent through a secure VPN mesh from one cloud to another.

Most of the image reads will be handled by the CDN.






Further improvements:

I need to lower the MTU for my multi-cloud network. The standard size for a package is 1500 bytes, and before a package leaves a server it gets intercepted by Tailscale and adds its own data headers to the packet. Now the package is like 1580 bytes and therefore too large, and gets split into two packages, all of a sudden I have double the traffic to deal with, causing extra network overhead and spikes my database query latency. To fix this I can lower the MTU to 1420, if the VPN package overhead is 80 bytes, then I'm back at 1500, no packet fragmentation is triggered.

Introduce automated failover, if the primary goes down, a read replica gets promoted to primary

Add queues, for example for asynchronous image processing

With that setup it's easy to add more edge locations to absorb more traffic.