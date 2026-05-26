---
layout: single
title: "How I built the infrastructure for a large-scale social media field experiment during the 2024 US election"
date: 2026-05-26 02:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/2026-05-26-bluesky-design-deep-dive
---

## What we built

## Why did we need to build this in the first place?

## How did Bluesky make this possible?

## Architecture at a glance

```mermaid
flowchart LR
  B[Bluesky firehose and APIs] --> S[Sync pipelines]
  S --> P[Preprocessing]
  P --> C[Fan-out to integrations]
  C --> M[ML classifiers]
  C --> SP[Superposter calculation]
  C --> V[Offline FAISS embeddings]
  M --> U[Unify integrations]
  SP --> U
  V --> U
  U --> R[Generate feed ranking algorithms]
  R --> A[Feed API]
  A --> U2[Bluesky users]
```

## Deeper dive into the architecture

### Data ingestion

### Preprocessing

### Enrichment

#### ML classifiers

We used a variety of ML classifiers for the project ...

### Recommendation algorithms

### Serving

### Observability and Ops

### Infra and deployment

## Tradeoffs we made

## Where I'd likely change it up now

### DevOps from the start


