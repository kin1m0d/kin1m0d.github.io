---
layout: post
title: "Platform Observability"
date: 2026-07-15 13:40:00 +0000
categories: projects dancehub
published: true
---

<div style="background-color: #fff9e6; border-left: 4px solid #f59e0b; color: #78350f; padding: 16px; margin: 20px 0; border-radius: 4px;" markdown="1">
💡 **Work in Progress**  
This page is currently under construction. A few pieces are missing and I might adjust small bits.
</div>

<br/>


## Stage 1

### Metrics
To collect metrics we need to add a few exporters to the specific services (FastAPI, Postgres, Redis). Garage and Caddy already have an internal endpoint to scrape metrics. Additionally each node gets a node_exporter to track server metrics (CPU, RAM, and other things). To host Grafana and Prometheus we'll provision a dedicated observability node.

### Logs
My initial plan was to use Loki with Promtail, but Promtail has reached end-of-life in March, and Grafana Alloy is the official successor. I don't have experience with either tools, but I assume they have to run on the same node where it scrapes the logs? Loki will definitely join the observability node, everything else remains to be seen.

### High Availability
Is it worth over engineering the observability stack to achieve high availability? For example Prometheus is not designed to do that. I will just say, no, it's not worth it, we're not operating in the finance sector. I'm sure there are a few ways to make this more reliable, but for now the setup is good enough, especially for the size of the platform. Let's move on.

<br/>
---

## SLIs and SLOs

Before we move to the next stage, let's define some service level indicators and objectives.


And to keep things simple, we'll start only with two critical user facing metrics.

### Defining Indicators
For a period of 30 days:
- Availability, 99.5% of successful requests or all non-5xx status codes (successful requests / total requests)
- Latency, 99.0% of requests completed under 500ms


What are the most critical user journes?
- User logs in
- User filters for events
- User opens event
- User creates event
- User uploads image
- User loads image


## Defining Indicators

- Login success rate: successful logins / total login attempts
- Event filter performance: percentage of filter requests completed within 500ms
- Event load performance: percentage of event loads completed within 1 second
- Event creation success rate: successful event creations / total creation attempts
- Image upload success rate: successful uploads / total upload attempts
- Image load success rate: successful image requests / total image requests

## Defining Objectives

To keep this simple for a one man-army, I'll aim for 99% availability, which allows for roughly 7.3 hours downtime per month.





<br/>
---

## Stage 2

<div class="tenor-gif-embed" data-postid="16839780121924734769" data-share-method="host" data-aspect-ratio="0.871486" data-width="100%"><a href="https://tenor.com/view/marmalady-loading-cat-loading-no-thoughts-head-empty-orange-cat-gif-16839780121924734769">Marmalady Loading Cat GIF</a>from <a href="https://tenor.com/search/marmalady+loading-gifs">Marmalady Loading GIFs</a></div> <script type="text/javascript" async src="https://tenor.com/embed.js"></script>


<br/>