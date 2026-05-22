# Serverless Processing Exploration (Ingestion Pipeline)

This document describes the conceptual and architectural exploration of moving DocCore’s ingestion pipeline from an always-on worker model to a serverless/event-driven execution model.

It is not a production migration, but a design study of how the current worker-based ingestion could evolve toward a more elastic architecture.

---

# Overview

DocCore currently processes document ingestion using an always-on worker based on Redis Queue (RQ).

While this model is simple and reliable, it introduces a baseline compute cost due to continuous execution.

This case study explores alternative serverless and event-driven approaches to ingestion processing.

---

# Current Model (Baseline)

```text
Upload → API → Redis Queue → Always-On Worker → PostgreSQL + pgvector
```

Characteristics:

- persistent worker process
- continuous execution loop
- idle time still consumes resources
- simple operational model

---

# Why Explore Serverless Processing

The main motivation for exploring serverless ingestion is:

- reduce idle compute cost
- improve elasticity
- align cost with actual usage
- reduce always-on infrastructure dependency

This becomes more relevant in:

- low-traffic SaaS stages
- irregular ingestion workloads
- portfolio/demo environments
- burst-heavy systems

---

# Target Serverless Concept

A serverless ingestion model would replace the always-on worker with event-driven execution.

```text
Upload → API → Queue/Event → Serverless Function → PostgreSQL + pgvector
```

Instead of a persistent process, each ingestion job runs as an isolated execution unit.

---

# Candidate Architectures

Several serverless patterns can be applied to the ingestion pipeline.

---

## 1. Queue + Serverless Worker

```text
API → Redis / Queue → Serverless Function (Cloud Run / Lambda)
```

Each job triggers a new execution instance.

### Characteristics:

- scale-to-zero behavior
- execution per job
- stateless processing
- external queue dependency

---

## 2. HTTP Triggered Processing (QStash Style)

```text
API → QStash → HTTP Endpoint → Serverless Handler
```

### Characteristics:

- event-driven HTTP calls
- no persistent worker
- retry handled by event system
- strong decoupling from infrastructure

---

## 3. Fully Managed Serverless Jobs

```text
API → Job Queue → Managed Serverless Job Runner
```

Examples:

- Cloud Run Jobs
- AWS Lambda + SQS
- similar managed execution services

---

# Mapping Current Pipeline to Serverless

The existing ingestion pipeline can be decomposed into stateless steps:

---

## Step 1: Document Intake

- receive file metadata
- store raw document reference
- enqueue processing event

---

## Step 2: Extraction

- stateless text extraction
- format-dependent parsing
- no persistent state required during execution

---

## Step 3: Document Blocking

- deterministic transformation step
- operates on extracted text
- no external state dependency

---

## Step 4: Embedding Generation

- external API call (embedding provider)
- stateless computation step
- rate-limited execution

---

## Step 5: Vector Storage

- write embeddings to PostgreSQL (pgvector)
- idempotent inserts preferred

---

# Execution Model Differences

---

## Always-On Worker Model

```text
Process lifecycle:
start → idle loop → job execution → idle → repeat
```

- persistent memory
- low cold start overhead
- idle cost exists

---

## Serverless Model

```text
Execution lifecycle:
trigger → execute pipeline → terminate
```

- no idle cost
- cold start overhead possible
- stateless execution required

---

# Key Architectural Changes Required

Moving to serverless would require several changes:

---

## 1. Stateless Pipeline Design

Each ingestion step must be:

- independent
- restartable
- idempotent

---

## 2. Externalized State Management

State must be fully stored in:

- PostgreSQL (primary state store)
- job metadata tables

No reliance on in-memory worker state.

---

## 3. Retry Strategy Shift

Instead of RQ retries:

- retries handled by queue provider (QStash / SQS)
- or orchestration layer

---

## 4. Execution Time Constraints

Serverless environments impose limits:

- max execution time per function
- memory constraints
- cold start behavior

Pipeline must respect these constraints.

---

# Trade-offs

---

## Advantages of Serverless Model

- scale-to-zero behavior
- cost proportional to usage
- no idle compute cost
- natural elasticity
- better burst handling

---

## Disadvantages

- increased architectural complexity
- harder debugging (distributed execution)
- cold start latency
- more moving parts
- pipeline fragmentation risk
- observability challenges

---

# Impact on DocCore Architecture

Switching to serverless would affect:

---

## 1. Worker System

Current RQ worker would be replaced by:

- event-driven functions
- or managed job runners

---

## 2. Pipeline Orchestration

Current linear worker pipeline would become:

- multi-step distributed execution chain
- event-based transitions

---

## 3. Observability Layer

Would require stronger:

- logging aggregation
- distributed tracing
- job lifecycle tracking

---

## 4. Failure Handling

Failure model shifts from:

- local retry loop (RQ worker)

to:

- distributed retry orchestration
- external queue retry policies

---

# Hybrid Model (Most Realistic Evolution Path)

A more realistic evolution is a hybrid architecture:

```text
API → Queue → Lightweight Worker OR Serverless Burst Execution
```

This allows:

- keep simplicity for baseline load
- use serverless only for spikes
- gradual migration instead of full rewrite

---

# Key Insight

Serverless processing is not just a cost optimization change.

It fundamentally changes:

> execution model, failure handling, and system observability

This makes it a **structural architectural decision**, not just an infrastructure swap.

---

# Why It Was Not Implemented Yet

The current system prioritizes:

- operational simplicity
- predictable debugging
- stable ingestion pipeline
- minimal distributed complexity

Serverless processing introduces:

- more moving parts
- harder local debugging
- distributed execution complexity

At the current stage, this trade-off is not justified.

---

# Future Direction

Potential evolution path:

1. maintain current RQ worker model
2. introduce burst execution experiments
3. test QStash or Cloud Run for ingestion spikes
4. evaluate hybrid ingestion architecture
5. gradually decouple always-on worker dependency

---

# Final Notes

Serverless processing is a valid and scalable evolution path for DocCore ingestion, but it introduces significant architectural complexity.

The current system prioritizes simplicity and reliability, while serverless remains an optional optimization layer for future scaling and cost efficiency improvements.
