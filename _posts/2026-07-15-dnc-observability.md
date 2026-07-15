---
layout: post
title: "Platform Obeservability"
date: 2026-07-15 15:40:00 +0000
categories: projects dancehub
published: true
---

<div style="background-color: #fff9e6; border-left: 4px solid #f59e0b; color: #78350f; padding: 16px; margin: 20px 0; border-radius: 4px;" markdown="1">
💡 **Work in Progress**  
This page is currently under construction. A few pieces are missing and I might adjust small bits.
</div>

<br/>


# Stage 1

Tech stack, traditional, battle-tested open-source

Grafana: Your unified visualization frontend.
Prometheus: Pulls and stores metrics from your single-node services.
Grafana Loki + Promtail: Promtail sits on your host, ships application logs, and Loki stores them.




Each node/service will run a leightweight exporter on the same node to scrape metrics/logs etc.
New centralised node for Grafana, Prometheus
On the new node we introduce a new service Loki for logs



## Metrics
To collect metrics we need to add a few exporters to the service nodes
- FastAPI
- Postgres
- Redis

Garage and Caddy already have an internal endpoint to scrape metrics
Additionally each node gets a node_exporter to track server metrics (CPU, RAM, etc.)
