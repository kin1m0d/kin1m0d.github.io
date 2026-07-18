---
layout: post
title: "SLI vs SLO vs SLA"
date: 2026-07-01 21:56:00 +0000
categories: blog sre
published: true
---

<div style="background-color: #fff9e6; border-left: 4px solid #f59e0b; color: #78350f; padding: 16px; margin: 20px 0; border-radius: 4px;" markdown="1">
💡 **Work in Progress**  
This page is currently under construction. A few pieces are missing and I might adjust small bits.
</div>

<br/>

This is your SRE crashcourse! Site Reliability Engineering is not just a job title, it's a framework with three core concepts
1. SLI: Service Level Indicator
2. SLO: Service Level Objective
3. SLA: Service Level Agreement


## SLI
A good SLI measures outcomes not inputs, it should measure what the users experience. For example
- Requests succeed
- Response is fast
- Data is correct

And **NOT**:
- CPU utilisation
- Memory usage
- Disk IO

While you should monitor infrastructure metrics as well, SLIs are about the user experience metrics.


## SLO
Internal goal, what you aim for **internal**
teams goal
guides decisions, deployments


## SLA
Legal commitment to customers, promise with consequences, 
violating SLAs, pay money to customer or customer can leave




SLA must be looser then SLO, measured by SLIs
Example

SLA: 99.5% -> SLO: 99.9% -> SLI 99.95% 
safety margin of 0.45 (99.95-99.5)

gives time to detect problems and fix issues before violating customer contracts
