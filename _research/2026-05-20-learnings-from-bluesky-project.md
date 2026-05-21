---
layout: single
title: "What I learned from spending 2 years building the app for a large-scale field study"
date: 2026-05-20 02:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/2026-05-20-learnings-from-bluesky-project
---

# What I learned from spending 2 years building the app for a large-scale field study

(TODO: workshop name)

- Make things reusable
- *BUT* do NOT spend too much time making things reusable. Know the difference.
- Good code is code that helps you ship product effectively (quickly, accurately, and will continue to do so over time).
- Implementing things from scratch gives you much deeper understanding of how they work (e.g., my "implement queues from scratch" method).
- Education + engineering is better than either on its own.

What am I learning now:

- management
- scalable design
- tradeoffs
- translating to AI agent flows.

Key learnings (things that I can keep in mind):

- From self-doubt to earned confidence: "I started this project unsure whether I could truly build anything end to end, and over two years the project became the proof that I could."
- From doing tasks to developing judgment and product sense: "The biggest thing the project taught me was not a specific stack or tool, but engineering judgment: how to make tradeoffs, simplify problems, and build systems that actually work in the real world."
- From isolated skills to an integrated worldview: "Before this project, I had many fragmented skills, but I lacked a unified way of seeing how research, engineering, infrastructure, and product fit together. Bluesky taught me how those pieces connect."

## Part 0: Before the project

In August 2023 I received news that I was being laid off. My employer was dying a slow death and I was laid off along with most of my team.

It took a few weeks to process the emotional reaction of the layoff. It was relief ("I'm glad I don't have to be up at 3am for meetings anymore"), anger ("I'll become so great and prove them wrong?"), disappointment ("I'm a disgrace of a developer") to renewal ("Now what?"), all in the span of 1-2 months, all at various beach fronts, resorts, cafes, and vacation rentals in Southeast Asia (admittedly not a terrible way to recover from a layoff).

When I had an opportunity to gather my bearings, I was unsure of how I wanted to proceed next. I felt like I didn't know how to develop software and I had little skills beyond executing tickets. I could write one-off code and PRs but I didn't have any vision for how to put things together. I could see pieces of the elephant but not the entire elephant. I also felt like my expertise (AI/ML) was (1) too niche/domain-specific, and (2) didn't give me the skills to build anything "useful". I had some experiences shipping real AI-powered apps at my previous job (e.g., classification models), but I actively advocated to work on more data engineering, backend, and DevOps tasks because I had believed that in doing those, I would learn "real" engineering skills. This left me with task-level skills ("train this model", "fix this bug", "add this new integration"), but without a unified mental framework for how these different concepts fit together and how I could combine them to build a single tangible end product. I felt frustrated that I could write code yet I didn't believe that I could actually "build".

I doubted my potential as a future software engineer. I wondered if I had actually learned anything during my previous job, whether I was actually "skilled" or whether I was hired because of my Yale pedigree, and if I could build anything of substance myself or if I would be relegated to a future of only completing tasks for projects scoped out by other people.

Buoyed by savings and my severance, I took the opportunity to explore other opportunities.

### (Mis)adventure 1: Trying to become a full-stack dev

I thought "real engineers" did full-stack, so I followed multiple coding tutorials on YouTube (e.g., "Build a Spotify clone from scratch") as well as completing multiple classes on [freeCodeCamp](https://www.freecodecamp.org/) (which is still the way that I recommend people learn to code!). I learned names of frameworks and I can count the lines of Javascript code I wrote in service of copying yet another "Build a (insert app) clone from scratch" app. However, I didn't really internalize deeper principles, both on the engineering side (e.g., "how does XYZ work internally?") and on the product side (e.g., "how do we decide what to build and why?").

### (Mis)adventure 2: Trying my hand at business

Inspired by the entrepreneurial streak of my family (my mother owned multiple restaurants), I decided to learn about running a business.

I read a few Alex Hormozi books as well as other classics such as "How to Win Friends and Influence People". I even bought an entrepreneurship course (before finding out that the way that influencers become rich is through selling you courses on how they became rich).

I didn't make much traction here, mostly because I quickly learned that I was fighting an uphill battle where I would have to start from scratch without any unique branding or offering.

### Figuring out my "unfair advantage"

At some point, I learned about the idea of an "unfair advantage" (likely from sources like [this one](https://www.diannwingertcoaching.com/blog/what-is-your-unfair-advantage)). I had failed at building traction at my misadventures related to full stack development or business. I had also, from my time in the Philippines, learned a bit about outsourced labor and reasoned that the American workforce would become increasingly global-after all, why hire an American dev for $80,000/yr when you can hire an equally qualified, English-speaking dev in the Philippines for $8,000/yr?

With this as the backdrop, I began searching for what skills and experiences I had that gave me my own "unique advantage" in an increasingly competitive workforce. The list I came up with was something like:

- High intelligence
- Work ethic
- Yale pedigree
- Silicon Valley startup experience
- AI/ML theoretical and practical experience

I reasoned that rather than starting a new venture without any unique advantages and building an online brand or reputation from scratch, I could leverage existing privileges and talents that would take others years to accumulate. From a branding perspective, I realized that the signaling conferred from the Yale and Silicon Valley brandings were associations to lean into, rather than affiliations to distance myself from. From a career development perspective, I saw the early rise of ChatGPT (my previous employer went all-in on ideas like AI agents and RAG well before those terms became mainstream) and predicted that AI/ML fundamentals (e.g., math, algorithmic understanding, etc.) were filters that would gatekeep a large proportion of up-and-coming devs interested in AI. I also reasoned that if I leaned into my statistics undergraduate education from Yale, I would further build on my AI/ML fundamentals, given that I had already done the hard part of obtaining years of rigorous math education. I predicted that the investment would exponentially pay off over the years as deeper mathematical and algorithmic understanding of AI/ML would become simultaneously more in-demand and yet be an all-too-rare skill in a world where (1) AI becomes more deeply embedded in traditional workflows but (2) fewer people would pursue the years of rigorous math education to deeply internalize how to build AI models from scratch.

I did not appreciate it then, but my choice to both explicitly state and then lean into my "unfair advantages" would become career-defining and has largely unfolded as I previously predicted, though even I didn't appreciate the degree to which it would be true.

## Part 1: Starting the project

### Learning about Bluesky

Through a series of happenstance events, I reconnected with a postdoc I worked with in college, who was now a professor at Kellogg School of Management at Northwestern University, [Dr. Billy Brady](https://www.kellogg.northwestern.edu/academics-research/faculty/brady_william/). We had chatted after our previous paper was published and I had mentioned to Billy that I was interested in a new opportunity while he had mentioned an interest in more ambitious engineering-related projects. Eventually, this led to an opportunity for me to work full-time at Northwestern.

During our discussions, Billy had mentioned to me this idea for an ambitious project related to the 2024 US election. I knew he was interested in the impact of algorithms on social learning (after all, this was his area of expertise and also was the [subject of the paper we had published](https://www.nature.com/articles/s41562-023-01582-0)). Billy had done numerous experiments demonstrating the impact of social media algorithms on warping people's perceptions of Democrats/Republicans, what issues the average American cared about, and how much people had in common vs. how much they actually differed. However, the scope of many of these experiments were limited to small in-lab applications or field studies in which the underlying algorithms were gatekept by Facebook/Twitter. A direct manipulation to the underlying algorithms to explicitly test the impacts of different modifications on users had never been tested in a live social media app outside of Facebook/Twitter (and understandably, those companies are disinclined to publish academic works against their profit motives). However, a new social media platform called [Bluesky](https://en.wikipedia.org/wiki/Bluesky) had just been announced, an offshoot from Twitter that promised to provide an open-source, open-protocol social media platform, free from deplatforming and censorship. Importantly, Bluesky offered the chance to host our own social media feeds, which meant that we could design our own feed ranking algorithms, populate them with actual live real-time posts from the Bluesky platform, and serve them to real Bluesky users within the app itself. This was an unprecedented opportunity to design algorithms ourselves and see how they would work in a real social media application.

As we talked more about the project, I began to understand the ambitious scope of the project, the impact on academia, and the technical scope of such an endeavor. When I found out about the project, I was simultaneously nervous and excited. I remember telling my partner "this project is really ambitious, but imagine what would happens if I could do it?" I had never built a full end-to-end application on my own, but I took this as an ambitious challenge that would finally allow me to learn what it means to "build" something and to be able to unashamedly identify as a "real" software engineer. I was in equal parts nervous and excited, wondering if I could pull off such an ambitious project, but excited about the outcome if it turned out that I could.

### Planning the project

As I had never built something like this before, I pored into previous literature, books, and GitHub repos to see if I could find some motivating examples. I found examples

(how much did it help me? It's hard to say).

- I read a few books to prepare for this. I also read a bunch of papers.
- I felt nervous, excited, etc., to take on such a large project.

(I also tried my hand at system design for the first time)

(paste some pictures)

(I overcomplicated it because I saw that's how other people did it, without internalizing why they did it that way. I had never built anything from the ground up, so I was left wondering how to do so).

I took a very helpful Coursera course on recommendation algorithms. It gave me a mental framework for understanding recommendation algorithm variations (eg content-based vs. collaborative filtering) and ways to measure performance (eg recall@k). However, many of the examples were related to topics like shopping or movie recommendations, which weren’t directly relatable to building a social media app (though in large part because I didn’t realize that creating and training a recommender system was probably the easiest, least complicated part of developing a fully functioning social media pipeline).

The closest example to my use case that I found was the open-sourced version of the [Twitter algorithm](https://blog.x.com/engineering/en_us/topics/open-source/2023/twitter-recommendation-algorithm). Reviewing its components did give me one framework for designing the application, but because I didn’t have any experience yet building similar apps, the large, complicated, enterprise-heavy version of the Twitter algorithm was my only reference. I focused too much on the deeply complex technical details that needed dedicated technical teams (eg., training custom twin embedding models, scaling the social graph, etc) without understanding a first-principles design of how to build the app (get posts -> generate candidates -> filter candidates -> create feeds -> rerank feeds -> serve feeds). Without a mentor to guide me on the key details to focus on, I was both intimidated by the scale of such a project (“Twitter has hundreds of engineers working on their algorithm, how could I possibly do it?”) and attempting to make an overly complicated replication of Twitter’s version. I shudder now when looking at my first version of the system design.

![Early system design diagram](../assets/images/2026-05-20-learnings-from-bluesky/img1.png)

This version of the design had complexity in the wrong places (e.g., using Kafka, defaulting to a fan-out architecture) and missed important details (e.g., where to store intermediate results, lack of telemetry, no data models). The rough "data -> integrations -> ML" emerged at this stage, which ended up being directionally correct, but it took a lot of learnings along the way.

![Newer system design diagram](../assets/images/2026-05-20-learnings-from-bluesky/img2.png)

This simplified version of the final system design is a bit more tool-agnostic (e.g., it doesn't explicitly mention Kafka) while preserving the core logic and making the pipeline flows explicit.

I overwhelmed myself with the pressure of trying to develop a design as “professional” as the one I had seen for Twitter. I also wanted to pattern-match against the lessons I had learned from my books and Coursera class, so I was introducing concepts like “collaborative filtering” without understanding the core algorithms or what had to be true to support them (eg to do collaborative filtering would require access to social network data, like liked posts, that I did not have).

In hindsight, I would have changed my approach. I would start with the simplest possible approach, in a single script, with max 200 lines of code. I can look at the different components that that script would have (eg a way to load posts, filter them, and then a way to generate the algorithms) and then imagine how each of those pieces would be their own units of work. When I mentor now, I often ask developers what’s their simplest possible single-script implementation of a given task, and then pair design with them to show how their simple implementation can get more complicated over time. A sign of seniority and experience that I can now better appreciate is the ability to look at functions or single lines of code, imagine where and when they’d break, and figure out how to make them more complex (and when something like that is useful versus overkill).

## Part 2: Building the app

### Initial experiments, or "running around in circles"

I started with one-off scripts for individual components of the project, starting with getting the posts themselves. This approach characterized a directionally valid yet unprincipled approach towards developing the platform, where I worked on individual units of work (eg getting the posts, preprocessing them, training the models) without much thought as to how they’d fit together or a top-down vision of the end product. I mostly approached it as “I know I’ll need to get posts” or “I know I’ll need to train a model”, and built individual scripts as needed. It was only later that I’d work towards stitching them together. I describe this as directionally valid because I did end up building the basis for components I’d eventually need, but I also spun my wheels on dead-end projects. I experimented with [what is now known as prompt engineering](https://markptorres.com/research/llm-experiments-pt-i) before it became a popular term. I experimented with the effects of different input formats (eg YAML vs JSON) before formats like BAML became popularized. I also developed the foundations for my data pipeline by building record-specific integration transformations and experimenting with [different forms of language classification](https://markptorres.com/research/llm-experiments-pt-v).

However, for all of these successful experiments, I also had a plethora of failed ones. I discovered early on that LLMs lacked context on current events, which is a problem when our sociopolitical classifier required knowledge of current events. To ameliorate this, I developed a RAG-based classifier that update a database with daily news syncs and then develops a routing-based system to figure out [which queries needed relevant current context](https://markptorres.com/research/llm-experiments-pt-iii). In hindsight, this approach was directionally right and I think a RAG-based classifier works well here here, but we decided to scrap this because (1) I had more pertinent things to build (luckily I was beginning to learn the art of prioritization), (2) the problem wasn’t common enough to merit maintaining the classifier. If I were to implement this today, I think this plan is directionally right as a basis, but I would complement it with an entity classifier and a sentiment classifier. Rather than counting on direct RAG queries to determine sentiment about a topic, which is inefficient at the scales we care about, I could repurpose the news updates pipeline to populate a database with key entities (eg politician names, bill names, etc) and the sentiment linked to each political party (eg democrats support X bill, republicans are against it). Although vastly oversimplified and incomplete (eg a person identifying as a Democrat can be against a post that the Democratic establishment), this simplified approach generally works well (which is more an indictment of the average social media user having very uncomplicated political takes that can be reduced to "Trump is good" or "Trump is bad").

I also had [another series of failed experiments](https://markptorres.com/research/llm-experiments-pt-iv) related to trying to get conservative posts and accounts from Bluesky. This shortcoming was due in large part to platform-specific population distributions, and we eventually accepted that as a limitation, but I spent too long trying to find clever workarounds for this problem, such as upsampling certain accounts or trying (in vain) to scour through the firehose of daily posts to find the conservative needle in the liberal haystack.

What did I learn from this stage of the process?

- (learnings)

### Converging on the "ship what matters" mindset

(slowly learning skills and ways of growing as an engineer, see, e.g.,

- https://markptorres.com/personal/2024-06-24-knowing-how-to-code-isnt-enough
- https://markptorres.com/personal/2024-06-12-best

)

## Part 3: Seeing the app go live

## Part 4: Running the study

## Part 5: Writing up what we did

## Part 6: Aftermath

## Part 7: What I'm working on now

## Part 8: Where I'd like to go next
