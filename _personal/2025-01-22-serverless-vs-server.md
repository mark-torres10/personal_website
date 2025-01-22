---
layout: single
title:  "Why do we teach developers that serverless is always better?"
date:   2025-01-22 05:00:00 +0800
classes: wide
toc: true
categories:
- personal
- all_posts
permalink: /personal/2025-01-22-serverless-vs-server
---

## Why do we teach developers that serverless is always better?

I'm not sure why we always teach that "serverless is always better" and "don't manage your own servers". A better way to think about it is that "serverless" (or managed) solutions offer convenient abstractions, but at a cost. There's no free lunch here, as anyone who has seen an exorbitantly large cloud bill can attest. It's generally the case that "serverless is better", sure, but it's important to know why. After all, AWS isn't running a charity, they're such a profitable business and for good reason.

## Cases where serverless is better, and why it's probably taught that it is better

Serverless (or managed) solutions do offer convenient abstractions. For example, systems like SQS and Lambda are very easy to set up and use, especially as compared to doing it yourself. It's also very likely that the smart engineers at AWS have optimized their offerings to be highly scalable and performant and resilient, in much better ways than a single engineer can likely do themselves (unless they're absolutely cracked). Serverless solutions are also great for dealing with variable workloads, so it's especially good for client-facing web traffic, for example. Invoking lambda functions is very cheap, for example, and can scale to an arbitrary number of calls. It's very painful to manage scale-up and scale-down policies for servers, but it's easy to do so with serverless.

As cloud offerings give more and more generalized abstractions, they make it easier for developers to focus time on building product features and less time on repeatable infrastructure. I'm a big fan of the AWS ecosystem and how trivial they make it to, for example, deploy apps or spin up a backend or build a large-scale data pipeline. There is good reason why they're the market leader in this space after all, and I think every engineer should be comfortable and performant in the cloud.

## Caveats with defaulting to serverless

However, it's important to also know when serverless isn't ideal. Serverless is great, until you get that $10,000 AWS bill and you realize that you could've gotten a Digital Ocean droplet for $5/month and saved a lot of money and have gotten 80% of the quality. Using a lambda as a backend can be fine, but if you need real-time responses, you'll have to wait for the warmup time. If your server is doing a low-powered job that can run on a single server and has no risk of traffic spikes, it's probably better to just run it on a single server and have a backup available. There are plenty of reasons to consider managing your own servers, and it's important to know when to do so as opposed to always defaulting to using a managed solution. With sufficient scale (e.g., enough traffic), it often is much cheaper to use a managed solution as opposed to hiring an expensive SRE. But for smaller scale applications, there's no need to run up the AWS bill unnecessarily when you could just wire up a few EC2 instances.

This argument also more broadly extends to the "cloud vs. on-prem" argument. DHH (of Ruby on Rails fame) has a piece on why he moved his company off the cloud [(link)](https://world.hey.com/dhh/we-have-left-the-cloud-251760fb), and many large companies also host their own on-prem infrastructure.

There's no free lunch here. Just like how using higher-level languages like Python as opposed to C++ makes it easier for development at the expense of performance, using managed solutions makes it easier for development at the expense of cost.

## Conclusion

Critical thinking is a good skill to have as an engineer, and it's important to know why a "best practice" is like so, and when to "break the rules". I think it's OK to generally teach that cloud and serverless is better, especially for beginners, but as you do more software architecture and engineering, it's important to know when to break the rules.
