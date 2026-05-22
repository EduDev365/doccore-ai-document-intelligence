# Scaling Decisions (DocCore Architecture)

This document describes the scaling strategy, constraints, and architectural decisions related to how DocCore handles growth in usage, data volume, and processing demand.

It focuses on practical scaling considerations across API, ingestion pipeline, retrieval system, and infrastructure layers.

---

# Overview

DocCore is designed as a production-oriented AI document intelligence system with:

- multi-tenant SaaS architecture
- RAG-based retrieval system
- asynchronous ingestion pipeline
- PostgreSQL + pgvector as core data layer

Scaling decisions are intentionally guided by simplicity and operational stability rather than premature distributed complexity.

---

# Current Scaling Model

```text
Users → FastAPI → PostgreSQL + pgvector → Redis Queue → Worker
```

The system scales along four main axes:

- API layer (request handling)
- Database layer (storage + retrieval)
- Worker layer (ingestion processing)
- Embedding layer (external API dependency)

---

# API Layer Scaling

The FastAPI layer is stateless.

## Scaling Characteristics:

- horizontally scalable
- no session affinity required (JWT-based auth)
- stateless request handling
- depends on database + Redis

## Bottlenecks:

- database query load
- vector retrieval latency
- connection pooling limits

---

# Database Scaling (PostgreSQL + pgvector)

The database is the most critical scaling component.

## Current Approach:

- single PostgreSQL instance (Neon)
- pgvector used for semantic retrieval
- tenant-scoped queries

## Scaling Constraints:

- vector search cost grows with dataset size
- index performance depends on data distribution
- connection limits become relevant under concurrency

## Key Decision:

> prioritize query optimization before horizontal scaling

---

# Retrieval Scaling (RAG Layer)

Retrieval is one of the most sensitive scaling areas.

## Scaling Behavior:

- embeddings grow linearly with document ingestion
- vector search cost increases with dataset size
- Top-K retrieval helps control prompt size

## Main Constraints:

- similarity search latency
- embedding dimensionality
- index performance (HNSW / IVF depending on configuration)

---

# Worker Scaling (Ingestion Pipeline)

The worker system is currently a single always-on RQ consumer.

## Scaling Characteristics:

- vertical scaling model
- single process execution (or limited concurrency)
- queue-based workload distribution

## Bottlenecks:

- embedding API rate limits
- CPU-bound text processing
- sequential pipeline stages

## Scaling Limitation:

> no automatic horizontal scaling is currently implemented

---

# Redis Queue Scaling

Redis acts as the coordination layer.

## Characteristics:

- lightweight queue system
- supports burst workloads
- decouples API from processing

## Scaling Constraints:

- memory limits (job backlog)
- connection limits
- single-node dependency (current setup)

---

# Embedding Layer Scaling

Embeddings are generated via external API.

## Scaling Constraints:

- provider rate limits
- network latency
- batching efficiency

## Key Bottleneck:

> external dependency rather than internal compute

---

# Key Scaling Philosophy

DocCore follows a **progressive scaling strategy**:

---

## 1. Optimize Before Distributing

Before introducing distributed systems:

- optimize queries
- reduce payload size
- improve retrieval quality
- minimize unnecessary complexity

---

## 2. Keep System Monolithic Where Possible

Core system remains:

- single API service
- single database
- single worker process

This reduces:

- operational overhead
- debugging complexity
- infrastructure fragmentation

---

## 3. Scale Vertically First

Initial scaling strategy prioritizes:

- vertical scaling (CPU / memory)
- query optimization
- indexing improvements

Before:

- horizontal worker scaling
- distributed ingestion
- microservices decomposition

---

# Scaling Bottlenecks Summary

| Layer | Primary Bottleneck |
|------|------------------|
| API | DB query load |
| Database | vector search complexity |
| Worker | embedding + CPU pipeline |
| Retrieval | similarity search at scale |
| Redis | memory + queue backlog |
| Embeddings | external API limits |

---

# Multi-Tenant Scaling Impact

Multi-tenancy introduces scaling considerations:

## Positive Effects:

- shared infrastructure efficiency
- centralized optimization
- unified caching strategies

## Challenges:

- tenant-level data growth imbalance
- noisy neighbor risk
- uneven retrieval load distribution

---

# RAG Scaling Challenges

Retrieval-Augmented Generation introduces unique scaling constraints:

---

## 1. Vector Growth

Each document increases:

- embedding count
- index size
- retrieval search space

---

## 2. Context Window Pressure

As retrieval improves:

- more context is selected
- prompt size increases
- token usage increases

---

## 3. Latency Trade-offs

Better retrieval often means:

- more computation per query
- higher latency risk

---

# Worker Scaling Strategy (Current State)

Current system uses:

- single worker process
- queue-based job distribution
- sequential ingestion pipeline

## Implication:

- ingestion throughput is bounded
- scaling requires vertical improvement or additional workers

---

# Why Not Horizontal Scaling Yet

Horizontal scaling introduces complexity:

- distributed job coordination
- duplicate processing risk
- state synchronization challenges
- increased operational overhead

At current stage:

> system simplicity is prioritized over elastic scaling

---

# Scaling Trade-off Philosophy

DocCore explicitly accepts the trade-off:

| Goal | Priority |
|-----|--------|
| simplicity | high |
| cost optimization | medium |
| elastic scaling | future |
| distributed architecture | deferred |

---

# Scaling Evolution Path

Likely future progression:

---

## Phase 1 (Current)

- monolithic API
- single DB
- single worker
- RQ queue

---

## Phase 2

- worker concurrency improvements
- query optimization
- partial caching layer

---

## Phase 3

- optional horizontal workers
- ingestion parallelization
- improved queue scaling

---

## Phase 4 (Optional Future)

- event-driven ingestion
- serverless workers
- fully decoupled processing pipeline

---

# Key Insight

DocCore scaling strategy is intentionally incremental:

> optimize locally before distributing globally

This avoids premature architectural complexity while maintaining production readiness.

---

# Final Notes

Scaling in DocCore is driven by practical constraints of a RAG-based multi-tenant SaaS system.

The architecture prioritizes:

- predictable behavior
- simple operational model
- controlled complexity growth
- incremental scaling improvements

over aggressive distributed system design.
