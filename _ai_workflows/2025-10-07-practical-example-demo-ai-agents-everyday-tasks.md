---
layout: single
title: "A Practical Example and a Demo: I'm Using AI Agents for Everyday Tasks"
date: 2025-10-07 10:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-10-07-practical-example-demo-ai-agents-everyday-tasks
---

# A Practical Example and a Demo of how I'm uusing AI Agents for everyday work

I have a task that I typically do where I take lecture videos from my class in my master's program, I go and I download the transcripts and I pass them in a ChatGPT so that then I can teach myself with my own AI agent prompt that I created for myself that is optimized to how I learn. And then I just do that instead of trying to watch the lectures, for me it's a lot more of active and engaging way to learn the material in a way that makes sense to me. Typically it's a manual process where I have to get the transcripts for myself and upload them to ChatGPT. It's very easy, but I wanted to see if AI agent mode on ChatGPT could automate this on my behalf. So below are some of the steps that I did to do that as well as some of the adventures that I had and some of the findings I had along the way.

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

I had to log in to `ChatGPT` because I suppose that this is a run in a virtual sandbox environment. Beyond that, I didn't have to intervene. It did spend a lot of time trying to goof around. It kept misclicking the + to attach files. It clicked on the wrong folders. It sometimes misclicked the text box. So there were some mistakes that hopefully are going to be cleared up as the computer use agents get better over time. On the plus side, it was smart enough to be able to intelligently allow a page to load. It would give the page time to load. It would type in the correct thing. It would know what text box was and be able to populate it correctly. It also knew enough to be able to upload the correct files. It was able to read what was on the page and know the titles of other chats that I had mentioned. It was also able to correctly know the file on the folder that I wanted it to reference in the first place. So a bit of work on the edges, but actually pretty good. And eventually after 10-15 minutes, it was a complete task. It would have taken me like 2-3 minutes to do, but it's cool to see what it could and couldn't do. It kind of got confused and if it goofed around a little bit and it fumbled in ways that a human would not do. But the fact that a computer can even do this in the first place is pretty impressive. And you can imagine having something like this operate in the background and do tasks that don't require human intervention and just have it go upload forms, go and fill things out and things like that that normally a person wouldn't have to do. It oftentimes appears to also get messed up by pages having to reload it sometimes misclicks the wrong link and therefore has to spend time trying to correct itself and they get stuck in these weird loops even after 15 or so minutes it got stuck on like a misclicking task so it still can't really check the right thing but again the fact that a computer can do this is pretty impressive as it were.
