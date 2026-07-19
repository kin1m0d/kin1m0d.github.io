---
layout: post
title: "SLI vs SLO vs SLA"
date: 2026-05-12 21:56:00 +0000
categories: blog sre
published: true
---

Site Reliability Engineering is not just a job title, it's a framework with three core concepts
1. SLI: Service Level Indicator
2. SLO: Service Level Objective
3. SLA: Service Level Agreement


## SLI
A good SLI measures outcomes not inputs, it should measure what the users experience. For example
- Requests succeed
- Response is fast
- Data is correct

And **NOT**:
- CPU
- Memory
- Disk IO

While you should monitor infrastructure metrics as well, SLIs are about the user experience metrics.


## SLO
Those are your internal goal that you aim for.


## SLA
This is legal commitment to your customers, a promise with consequences. Violating SLAs means usually paying money/credits to customers, or worst case, customers can leave a contract without penalty.

Ideally your SLAs should be looser than SLOs. That gives you a safety net to detect problems and fix issues before violating customer contracts



## Reliability has exponential cost
Each nine costs ~10x more than the last, exponential more engineering effort.

| % | cost |
| --- | --- |
| 99% | 1x |
| 99.9% | 10x |
| 99.99% | 100x |
| 99.999% | 1000x |


## Error Budget
If you would aim for 100% reliability, it means you can't change anything, innovation stops. But how do you determine if you should deploy new features or work on reliability? The solution is called error budget. Error budget is basically a currency that you can use to deploy freely, when you have consumed the error budget, teams should freeze deployments and focus on reliability.

Let's say we have an SLO of 99.9% availability for 30 days, that gives us an error budget of 0.1% (100% - 99.9% = 0.1%). That gives us an allowed downtime of 43.8 minutes in a month.



## User journey based SLIs
To give your users the best experience we'll focus on the parts that are important for the users experience.

For an e-commerce platform, a user journey would be something like this: Browsing products -> Adding products to cart -> Checking out cart -> Tracking order.

- Checkout: most critical part, as money is involved -> 99.95% availability
- Browsing: important function -> 99.9% availability
- Tracking: order tracking is more nice to have -> 99.5% availability

This is just an example, but as you can see, depending on the function/importance the SLO can be more tight or relaxed.



## To summarise

- SLIs measure the user experience
- SLOs set internal targets
- SLAs create external promises
- Error budget balances velocity vs stability
- Measure user experience over infrastructure
- Reliability has exponential cost