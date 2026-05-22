# Worker Cost & Architecture Analysis (RQ Pipeline)

This document describes the architecture and operational role of the DocCore background worker system using Redis Queue (RQ) and an always-on worker process.

It focuses on how ingestion is processed, how the pipeline is structured, and the architectural implications of the current execution model.

---

# Overview

DocCore uses a background worker system to process document ingestion asynchronously.

The worker is responsible for transforming uploaded documents into structured representations used later by the RAG system.

Main responsibilities include:

- document ingestion execution
- text extraction
- document blocking
- embedding generation
- vector persistence (pgvector)
- pipeline state updates

The worker is focused entirely on ingestion and indexing, not query-time retrieval.

---

# Architecture

```text
FastAPI API
    ↓
Redis Queue (RQ)
    ↓
Worker Process (RQ consumer)
    ↓
PostgreSQL + pgvector
```

---

# Worker Role in the System

The worker is the execution layer of the ingestion pipeline.

It is triggered whenever a document is uploaded and enqueued by the API layer.

Flow:

```text
Upload → API → Queue → Worker → Processing Pipeline → Storage
```

---

# Ingestion Pipeline (Code-Based)

The worker executes the pipeline defined in:

- `process_ingestion_pipeline.py`
- `run_document_ingestion_job.py`

---

## 1. Document Ingestion

Upload requests are converted into background jobs.

The worker consumes these jobs asynchronously.

---

## 2. Text Extraction

Documents are parsed into raw text depending on format and structure.

---

## 3. Document Blocking

The system applies a blocking strategy defined in `blocking.py`.

Important clarification:

DocCore no longer uses naive chunking as the primary abstraction; it uses document blocks, with chunking as an internal fallback/splitting strategy.

Blocks are created using:

- structural signals
- size constraints
- overlap windows
- layout heuristics

The goal is to produce stable units for embedding and retrieval.

---

## 4. Embedding Generation

Each block is transformed into a vector embedding using an external embedding provider.

This step is asynchronous and rate-limited depending on provider constraints.

---

## 5. Vector Storage

Embeddings are persisted in PostgreSQL using pgvector.

Each entry includes:

- tenant_id
- document_id
- block_id
- embedding vector

This enables tenant-scoped semantic retrieval later.

---

## 6. Pipeline State Updates

The worker updates document processing state during execution:

- processing
- completed
- failed

This allows the API to expose ingestion status.

---

# Role in RAG System

The worker does NOT participate in query-time retrieval.

Its role is strictly building the index used by the RAG system.

At runtime:

```text
User Query → API → Vector Search → Context → LLM
```

At ingestion:

```text
Upload → Worker → Embeddings → Vector DB
```

---

# Execution Model

The worker runs as a long-lived RQ consumer process.

```text
start-worker.sh → listens to queue → processes jobs → idle wait
```

This means:

- the worker is always running
- it continuously polls or waits for jobs
- execution is not event-scaled

---

# Cost Model (Architectural View)

This section describes structural cost behavior, not measured billing data.

---

## Always-On Model

The worker runs continuously regardless of workload.

This implies:

- compute is allocated 24/7
- idle time still counts as runtime
- cost is not directly proportional to job volume

---

## Cost Characteristics

| Factor | Impact |
|------|--------|
| uptime | high |
| idle time | high |
| ingestion volume | medium |
| embedding workload | variable |
| queue activity | low impact |

---

## Key Insight

The main cost driver is:

> the always-on execution model, not the actual workload

---

# Trade-offs

The current architecture prioritizes simplicity over elasticity.

---

## Advantages

- simple pipeline architecture
- predictable execution model
- easy debugging
- stable ingestion flow
- clear separation between API and worker

---

## Limitations

- always-on compute cost
- no scale-to-zero behavior
- limited elasticity
- idle resource usage
- manual scaling only

---

# Why RQ

RQ was chosen for its simplicity:

- Redis-based queueing
- minimal infrastructure overhead
- straightforward integration with FastAPI
- simple worker lifecycle

This aligns with DocCore’s approach of keeping the system understandable and easy to operate.

---

# Important Clarification on Chunking

DocCore no longer uses naive chunking as the primary abstraction; it uses document blocks, with chunking as an internal fallback/splitting strategy.

This shift is important because:

- it improves consistency in retrieval units
- it decouples ingestion logic from simple text splitting
- it aligns better with embedding structure
- it reduces fragmentation issues in long documents

---

# Reliability Model

The worker system includes basic reliability mechanisms:

- RQ retry support
- job re-execution on failure
- error handling in pipeline steps
- ingestion state tracking in database

It is reliable for ingestion workflows but is not a full workflow orchestration system.

---

# System Role Separation

| Component | Responsibility |
|----------|----------------|
| API | request handling + enqueue jobs |
| Worker | ingestion + indexing pipeline |
| RAG runtime | query-time retrieval |
| PostgreSQL + pgvector | storage + vector search |

---

# Optimization Opportunities

Several directions exist for improving cost efficiency and execution model:

---

## Burst Execution

Run worker only when queue has jobs.

- reduces idle cost
- increases operational complexity

---

## Scheduled Processing

Batch ingestion using cron-style execution.

- simpler runtime cost
- higher latency

---

## Serverless Workers

Event-driven execution model:

- Cloud Run
- Lambda
- QStash

- better elasticity
- higher architectural complexity

---

## Hybrid Model

Combination of:

- lightweight always-on worker
- burst/serverless ingestion for peaks

---

# Key Insight

The worker is best understood as:

> a stable ingestion pipeline engine rather than an elastic compute system

It prioritizes:

- simplicity
- reliability
- predictable processing

over:

- cost optimization
- dynamic scaling
- serverless execution

---

# Final Notes

The worker is a core part of DocCore’s ingestion system and is essential for building the RAG index.

Its current design reflects a deliberate trade-off: keeping the system simple and reliable while accepting a non-optimal cost profile due to the always-on execution model.
