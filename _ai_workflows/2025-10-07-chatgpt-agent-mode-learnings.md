---
layout: single
title: What I learned watching ChatGPT Agent Mode try to automate lecture notes for me
date: 2025-10-07 10:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-10-07-chatgpt-agent-mode-learnings
---

# What I learned watching ChatGPT Agent Mode try to automate lecture notes for me

I have a task that I typically do where I take lecture videos from my class in my master's program, I go and I download the transcripts and I pass them in a ChatGPT so that then I can teach myself with my own AI agent prompt that I created for myself that is optimized to how I learn. And then I just do that instead of trying to watch the lectures, for me it's a lot more of active and engaging way to learn the material in a way that makes sense to me. Typically it's a manual process where I have to get the transcripts for myself and upload them to ChatGPT. It's very easy, but I wanted to see if AI agent mode on ChatGPT could automate this on my behalf. So below are some of the steps that I did to do that as well as some of the adventures that I had and some of the findings I had along the way.

Here's a [Loom link](https://www.loom.com/share/435b0ed30d8941239a21b811dbab7fa8?sid=bdaa0dd5-1363-4dfd-b809-05c3532dcf71) of my (mis)adventures

Here are the exact instructions I gave to ChatGPT:

```text
I need you to go to each of the videos in this playlist: https://utexas.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=31158508-1d81-420b-b88a-b1cb013c9abc&pid=de246629-aa7d-48ac-a3d3-b1ba004b8830&start=0.938977

Then, do the following:
1. Go to "captions".
2. Save the captions and download them to my local downloads.

Then, for each of the text files saved, I want you to, one at a time:
3. Then, I want you to open a new ChatGPT window. 
4. Go to the "School + Learning Stuff" project.
5. Start a new query. Attach the text file with the captions and then ask "Teach me this content".
6. Move on to the next text file and start at step #3 again.

Tell me when you're done.
```

It had a hard time opening the correct window to authenticate to the class-specific window, so I just went and downloaded the files myself. That's okay. After I did that, I added that to the next prompt. At this point, I used a speech-to-text app, so perhaps the translation was a little bit off. But this is what I essentially said.

```
I've downloaded the transcripts. Here are those text files. Continue with the steps provided.

Then, when you have created the three ChatGPT chat windows, one for each of the text files, I want you to go back to the projects in the folder that called School plus learning stuff. And then I want to change the name to each of those conversations. I want to give them this name pattern that you see in the other lectures that are prepended with brackets of Fall 2025. There's a particular syntax of how I named those, and I wanted to decipher what that is, interpret it, and then rename each of the three subsequent chats that you created in the same way. 
```

## What actually happened

I had to log in to ChatGPT manually because the agent runs in a virtual sandbox environment. Beyond that initial authentication, I didn't have to intervene.

### What didn't work well

The agent spent considerable time making clumsy mistakes:

- Repeatedly misclicked the `+` button to attach files
- Clicked on wrong folders
- Misclicked text boxes
- Got confused when pages reloaded
- Sometimes clicked the wrong links and had to backtrack
- Got stuck in loops trying to correct its own mistakes

These are mistakes that a human wouldn't make. After 15 minutes, it was still stuck on some misclicking tasks. These edge case failures will hopefully improve as computer use agents mature. Eventually I just stopped the agent and gave up after 25 minutes because I couldn't stand to watch it get stuck trying to click the ellipses that you had to click on the UI to be able to change the name. It knows that it should be doing it, but it's been stuck on that step for 10 minutes now.

### What actually worked

On the positive side, the agent demonstrated surprising competence in several areas:

- Intelligently waited for pages to load before proceeding
- Correctly typed in the right information
- Identified which text boxes to populate
- Successfully uploaded the correct files
- Read page content and recognized the titles of my existing chats
- Correctly navigated to the specific folder I wanted it to reference

Eventually, after 10-15 minutes, it completed the task, whereas that would have taken me 2-3 minutes manually.

## What this means for AI automation

The fact that a computer can do this at all is impressive. You can imagine these agents operating in the background for tasks that don't require human intervention: uploading forms, filling out repetitive information, organizing files, setting up recurring workflows. The current implementation fumbles in ways that feel distinctly non-human, but the core capability is there.

This isn't replacing human work yet. It's slower and more error-prone. But it's a glimpse of what's coming. Tasks that are tedious but don't require judgment or creativity are exactly where these agents will shine once the reliability improves.
