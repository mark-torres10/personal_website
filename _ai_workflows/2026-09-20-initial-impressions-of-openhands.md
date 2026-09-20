---
layout: single
title: "Initial impressions of OpenHands Cloud, as a Cursor Cloud Agents user"
date: 2026-09-20 16:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2026-09-20-initial-impressions-of-openhands
---

# Initial impressions of OpenHands Cloud, as a Cursor Cloud Agents user

I do all my work on Cursor, especially the cloud environments, but I am also interested in seeing what other options there are. For some work also, I am interested in a BYOK approach so that I can link my coding setup to some models that we have on AWS Bedrock.

The workflow that I am accustomed to is something like:

1. Define the task that I want to do: to write a GitHub issue.
2. Spin up a new remote agent to implement the issue.
3. Review the implementation.

I was interested in seeing what OpenHands Cloud would have to offer. I have been using it for a few days now, and here are some initial thoughts. This is specifically with regards to the cloud environment, not the IDE itself, as most of my work is done in the cloud environment.

Here's an example conversation with OpenHands:

![Screenshot of an OpenHands Cloud conversation](/assets/images/2026-09-20-initial-impressions-of-openhands/openhands-convo-screenshot.png)

Same UI in Cursor:

![Screenshot of a Cursor Cloud conversation](/assets/images/2026-09-20-initial-impressions-of-openhands/cursor-convo-screenshot.png)

## Cursor is much easier to use out of the box, but OpenHands offers a lot more customizability

I found the Cursor onboarding to be straightforward, and I could ship code within 2 to 3 minutes of onboarding to the cloud environment. For OpenHands, it was not difficult, but it still took a little bit more setup. I did find it to be more customizable, especially with regards to the coding agents used by OpenHands.

## OpenHands lets you configure more, while Cursor "just works"

OpenHands poses a lot more information and configuration to you than Cursor does, but Cursor's configuration tends to be more abstracted away from the model layer and more related to the part itself. For Cursor, you can swap between the different models, but you can't bring a model yourself or use one that Cursor doesn't itself support. Cursor also takes care of a lot of the details, like compaction and managing the cost. These are things that OpenHands can do, but it requires a little bit more setup.

## OpenHands lets you use your own models

OpenHands explicitly allows you to use your own models. I wanted to use some models hosted on AWS Bedrock, so I'm using that, but in trying to set that up, I saw that OpenHands allows Ollama and other ways to self-host models, as well as a variety of inference providers. In contrast, Cursor's model choices are much more limited and opinionated, but it does remove the need to make these decisions yourself.

That being said, it is nice to be able to bring my own model into OpenHands and to use the model setup that I would like. It's also been eye-opening to see the amount of usage that I am incurring and the token cost. Cursor, especially with regards to their own in-house models, definitely subsidizes the cost a lot better. OpenHands, in contrast, charges tokens at the market rate.

## User experience

I found the interface on Cursor to be snappier and more responsive. The load time for the sandbox on Cursor was quicker than on OpenHands. I also found the streaming rate to be faster on Cursor than on OpenHands. It's unclear how much of this is UI/UX experience as opposed to a model-serving characteristic. Cursor likely has partnerships with their models that they serve, such that the latency is probably lower and well suited for Cursor's demand and load. Whenever I start a new sandbox, it takes up to a half minute on OpenHands, whereas on Cursor it feels snappier.

I also found the Cursor UI to be much simpler, especially as OpenHands seems to have two interfaces: a primary one and a new canvas UI. I do like the canvas UI much more, and it definitely seems quite similar to the Cursor or Claude Code interface. It would be nice if OpenHands would route to the canvas directly, but it's likely something that will come with time.

## Features from Cursor that I'd like in OpenHands

Some features that I would love to see on OpenHands that I was expecting coming from Cursor include:

1. **Prebuilt environments**. This is the biggest one for me, as it takes a while to spawn a new sandbox environment on OpenHands, but on Cursor, it feels so instantaneous and snappy. Environments can spawn in a few seconds on Cursor, when it can take 3-4x as long on OpenHands.
2. **Easy-to-configure environment setup**. In addition to sandbox spawning, I wish that OpenHands allowed you to configure how environments are set up as easily as Cursor does. Cursor allows you to very easily and seamlessly update secrets and create a setup script. I think OpenHands allows you to do both of these, but it's not as blatantly obvious in the UI that these exist.

If OpenHands did these, my experience would probably be on par with what I already do in Cursor, and I would be more apt to move more of my everyday work onto OpenHands.

## Some takeaways and when I'll use each

Right now, it seems like Cursor is more feature complete out of the box. It's easier to use from the get-go. I find the interface simpler and snappier compared to OpenHands. That being said, OpenHands does allow you a lot more customizability. I also do like the number of configuration details, especially on the model side, that OpenHands supports. I think OpenHands will probably lag in terms of product features as compared to Cursor, but depending on the use case, that will likely be okay. I think I'll continue to use Cursor for more daily and everyday work, given how easy the user experience is, especially on the cloud agent side. I do see myself using OpenHands if I want more control over the model layer myself, if I'm working with private or sensitive data that I don't want made available to Cursor servers, or if I want to just own the LLM stack and use models hosted on AWS Bedrock.

The larger trend of open-source models being slightly behind proprietary frontier models is likely going to continue to be true when comparing Cursor to OpenHands, but that's also completely okay because the feature set is still quite good in both. In the same way that I oftentimes use open-source models or smaller, cheaper models because they work well enough for my use case, I see myself increasingly doing that with alternatives to Cursor.
