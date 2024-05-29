---
layout: single
title:  "Accidentally deleting prod data (and the importance of backups)"
date:   2024-05-29 04:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/importance-of-backups-2024-05-28
---
# Accidentally deleting prod data (and the importance of backups).

## What went wrong

## Steps to fix it

### Looking at existing third-party options
There are a few third-party options that I saw (via ChatGPT), such as [DBeaver](https://github.com/dbeaver/dbeaver) and [DB Browser](https://sqlitebrowser.org/). But, it looks like these are just wrappers that would automate the task of writing the data as some converted and compressed file format on a certain schedule, so given how straightforward the task is, I'd rather just built it myself instead of relying on more dependencies.

### Create automated script to back up databases and restore from backups

#### Backing up the databases

#### Restoring from backups


### Running backups as a cron job

