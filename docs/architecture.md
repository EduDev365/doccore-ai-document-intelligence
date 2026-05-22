# DocCore Architecture

This document describes the high-level architecture, infrastructure, retrieval pipeline, multi-tenant isolation model, and engineering decisions behind DocCore.

---

# System Overview

DocCore is an AI-powered document intelligence platform focused on Retrieval-Augmented Generation (RAG), semantic retrieval, contextual document analysis, and multi-tenant SaaS infrastructure.

The platform transforms uploaded files into searchable semantic knowledge bases through document ingestion pipelines, embeddings generation, vector indexing, retrieval systems, and contextual language model interactions.

The architecture was designed around:

- Production-oriented backend design
- Multi-tenant isolation
- Retrieval quality
- Semantic search
- Async document processing
- Infrastructure simplicity
- Cost-efficient deployment
- Modular AI workflows

---

# High-Level Architecture

```text
                ┌────────────────────┐
                │ Next.js Frontend   │
                │      (Vercel)      │
                └─────────┬──────────┘
                          │
                          ▼
                ┌────────────────────┐
                │ FastAPI Backend    │
                │      (Render)      │
                └─────────┬──────────┘
                          │
         ┌────────────────┴──────────────┐
         │                               │
         ▼                               ▼
┌──────────────────┐          ┌──────────────────┐
│ Redis Queue      │          │ Neon PostgreSQL  │
│ + RQ Workers     │          │ + pgvector       │
└────────┬─────────┘          └────────┬─────────┘
         │                              │
         ▼                              ▼
┌──────────────────┐          ┌──────────────────┐
│ Embeddings       │          │ Semantic Search  │
│ Processing        │          │ Retrieval        │
└──────────────────┘          └──────────────────┘
```

---

# Core Architecture Components

## Frontend Layer

The frontend is built using:

- Next.js
- React
- App Router
- TailwindCSS

Responsibilities include:

- Authentication flows
- Document upload UI
- Retrieval chat interface
- Workspace management
- Multi-tenant navigation
- Artifact visualization

The frontend communicates exclusively through the backend API layer.

---

# Backend API Layer

The backend is implemented with FastAPI.

Main responsibilities:

- Authentication
- Tenant validation
- File ingestion
- Retrieval orchestration
- Context assembly
- AI workflow coordination
- Billing enforcement
- Queue orchestration
- Permissions and memberships

The backend follows layered architectural principles.

Current structure:

```text
application/
core/
domain/
infrastructure/
interfaces/
shared/
workers/
```

Goals of this structure:

- Separation of concerns
- Modular business logic
- Infrastructure abstraction
- Easier maintainability
- Scalable SaaS evolution
- Better testing boundaries

---

# Database Architecture

DocCore uses PostgreSQL with pgvector.

Current infrastructure:

- Neon PostgreSQL
- pgvector extension
- SQLAlchemy ORM
- Alembic migrations

The database stores:

- Tenants
- Users
- Memberships
- Documents
- Embeddings
- Retrieval metadata
- Billing data
- Usage tracking
- Artifact metadata

---

# Why PostgreSQL + pgvector?

Instead of using a dedicated vector database early in development, DocCore uses PostgreSQL + pgvector for several reasons:

- Unified relational + vector storage
- Lower operational complexity
- Easier deployment
- Better cost efficiency
- Faster development iteration
- Simpler infrastructure management

This architecture keeps semantic retrieval tightly integrated with relational tenant-aware queries.

---

# Multi-Tenant Architecture

DocCore was designed from the beginning as a multi-tenant SaaS platform.

Each tenant operates within isolated logical boundaries.

Core concepts include:

- Tenant-scoped documents
- Tenant memberships
- Tenant-aware retrieval
- Scoped permissions
- Usage quotas
- Tenant billing isolation

---

# Tenant Isolation Model

Isolation is enforced at multiple layers.

## Application Layer

Every authenticated request is associated with:

- User
- Tenant
- Membership context

Authorization checks validate tenant ownership before retrieval or document access.

---

## Retrieval Layer

Semantic retrieval is always tenant-scoped.

Retrieval queries apply tenant filters before vector similarity operations.

This prevents:

- Cross-tenant retrieval leakage
- Context contamination
- Unauthorized semantic access

---

## Storage Layer

Documents, embeddings, artifacts, and retrieval metadata are tenant-associated.

This allows:

- Logical isolation
- Usage accounting
- Safer retrieval boundaries
- Easier SaaS scaling

---

# Document Processing Pipeline

The ingestion pipeline transforms uploaded files into searchable semantic context.

High-level flow:

```text
Upload
    ↓
Validation
    ↓
Text Extraction
    ↓
Normalization
    ↓
Semantic Blocking
    ↓
Embeddings Generation
    ↓
pgvector Storage
    ↓
Retrieval Ready
```

---

# Upload Validation

Before processing, uploaded files are validated through:

- MIME type validation
- File extension restrictions
- File size limits
- Parsing validation
- Sanitized ingestion rules

The platform intentionally avoids unrestricted executable uploads.

---

# Document Extraction

Supported files are processed into normalized textual representations.

Extraction goals:

- Preserve semantic structure
- Maintain contextual continuity
- Reduce parsing noise
- Improve retrieval quality

---

# Semantic Blocking vs Traditional Chunking

DocCore explores semantic blocking strategies instead of relying exclusively on fixed-size chunking.

Traditional chunking often introduces problems such as:

- Broken clauses
- Context fragmentation
- Retrieval inconsistency
- Incomplete semantic sections

Semantic blocking attempts to preserve logical document continuity before embedding generation.

This becomes especially important for:

- Legal documents
- Contracts
- Technical documentation
- Long-form reports
- Structured textual workflows

---

# Embeddings Pipeline

After semantic segmentation, blocks are transformed into vector embeddings.

The embeddings layer enables:

- Semantic similarity search
- Context ranking
- Retrieval relevance
- Vector-based contextual matching

Embeddings are stored directly in PostgreSQL using pgvector.

---

# Semantic Retrieval System

DocCore uses semantic vector retrieval instead of keyword-only search.

Retrieval flow:

```text
User Query
    ↓
Query Embedding
    ↓
Vector Similarity Search
    ↓
Top-K Retrieval
    ↓
Context Assembly
    ↓
Prompt Construction
    ↓
LLM Generation
```

---

# Top-K Retrieval

The retrieval layer uses Top-K semantic matching to identify the most contextually relevant blocks.

Key retrieval concepts:

- Vector similarity
- Distance ranking
- Similarity scoring
- Context prioritization
- Retrieval filtering
- Tenant-scoped matching

Current optimization areas:

- Better ranking strategies
- Retrieval precision
- Hybrid retrieval experimentation
- Reranking systems
- Reduced hallucination risk

---

# Context Assembly

Retrieved blocks are assembled into contextual prompts before language model generation.

Primary goals:

- Context preservation
- Retrieval grounding
- Reduced hallucination
- Better answer consistency
- Retrieval-aware prompting

---

# AI Chat Architecture

DocCore enables contextual conversations over uploaded documents.

Chat capabilities include:

- Contextual question answering
- Retrieval-grounded responses
- Multi-block reasoning
- Semantic contextualization
- Document-aware conversations

The system focuses heavily on grounding model responses in retrieved document context.

---

# Artifact Extraction Workflows

Beyond conversational retrieval, DocCore supports structured artifact generation.

Examples include:

- Summaries
- Structured extractions
- Contextual outputs
- AI-assisted document workflows
- Structured generation pipelines

These workflows allow the platform to operate beyond traditional chatbot paradigms.

---

# Background Processing Architecture

Document ingestion and embedding generation run asynchronously through worker pipelines.

Current workflow:

```text
API Request
    ↓
Redis Queue
    ↓
Worker Processing
    ↓
Extraction
    ↓
Blocking
    ↓
Embeddings
    ↓
Database Storage
```

Goals of async processing:

- Reduced API latency
- Decoupled ingestion
- Better retry handling
- Scalable processing
- Improved reliability

---

# Queue Infrastructure

Current queue infrastructure:

- Redis
- RQ Workers

Queues are responsible for:

- Document ingestion jobs
- Embedding generation
- Long-running processing tasks
- Async pipeline orchestration

---

# Authentication Architecture

Authentication is JWT-based.

Current characteristics:

- JWT authentication
- HttpOnly cookies
- Membership validation
- Permission-aware APIs
- Tenant-scoped sessions

Authorization checks are integrated into the multi-tenant model.

---

# Security Architecture

Security is treated as a first-class architectural concern.

Current focus areas include:

- Upload validation
- Tenant isolation
- Retrieval isolation
- JWT authentication
- Server-side validation
- Permission enforcement
- Restricted parsing flows
- Environment secret isolation

---

# Malicious Upload Mitigation

The platform intentionally restricts potentially dangerous upload behaviors.

Defensive strategies include:

- Restricted file formats
- Non-executable upload policy
- Validation before extraction
- Controlled parsing surface
- Limited ingestion scope

This reduces risks associated with arbitrary document ingestion.

---

# Billing and Usage Tracking

DocCore uses a credit-oriented billing architecture.

Core concepts:

- Processing credits
- Production credits
- Usage ledger
- Subscription periods
- Tenant quotas
- Usage accounting

This allows flexible accounting for:

- Processing workloads
- Retrieval operations
- AI generation flows

---

# Infrastructure Stack

Current infrastructure:

| Component | Provider |
|---|---|
| Frontend | Vercel |
| Backend API | Render |
| PostgreSQL | Neon |
| Vector Storage | pgvector |
| Queue | Redis |
| Workers | RQ |
| Deployment | Docker Compose |

---

# Why Neon?

The database infrastructure migrated from Render PostgreSQL to Neon.

Reasons included:

- Lower operational costs
- Better serverless-oriented scaling
- Simpler database management
- Improved cost efficiency
- Reduced infrastructure coupling

---

# Why Sync SQLAlchemy?

The backend currently uses synchronous SQLAlchemy patterns.

Reasons include:

- Simpler operational model
- Stable worker integration
- Easier transaction handling
- Lower concurrency complexity
- Better fit for current workload profile

The current bottlenecks are primarily retrieval and processing operations rather than extreme concurrent API throughput.

---

# Engineering Trade-Offs

Several engineering decisions intentionally prioritize:

- Simplicity
- Operational stability
- Iteration speed
- Cost efficiency
- Maintainability

Instead of premature complexity.

Examples include:

- PostgreSQL + pgvector over dedicated vector DBs
- Sync ORM patterns
- Worker-based ingestion
- Modular layered backend architecture

---

# Current Limitations

The platform remains under active evolution.

Current experimentation areas include:

- Retrieval quality improvements
- Better reranking strategies
- Hybrid retrieval
- Event-driven ingestion
- Long-context optimization
- Semantic segmentation refinement
- Observability improvements

---

# Future Architecture Exploration

Potential future directions:

- Event-driven worker pipelines
- Serverless processing
- Advanced reranking systems
- Hybrid retrieval
- Better retrieval scoring
- Enhanced observability
- Distributed ingestion pipelines

---

# Repository Philosophy

This public repository focuses on:

- Architecture
- Engineering decisions
- System design
- Demonstration material
- Technical documentation

The production SaaS implementation remains private.

---

# Final Notes

DocCore is both:

- A production-oriented AI document platform
- An engineering exploration into real-world RAG systems

The project focuses heavily on practical AI infrastructure, semantic retrieval quality, SaaS architecture, and production-oriented AI workflows.
