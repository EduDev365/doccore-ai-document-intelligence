# System Decisions

This document describes the major architectural, infrastructure, retrieval, and operational decisions made during the development of DocCore, including the trade-offs behind each choice.

---

# Overview

DocCore was designed as a production-oriented AI document intelligence platform focused on:

- Retrieval-Augmented Generation (RAG)
- Semantic retrieval
- Multi-tenant SaaS architecture
- Async document processing
- AI document workflows
- Operational simplicity
- Cost-efficient infrastructure

Many architectural choices intentionally prioritize:

- Simplicity
- Maintainability
- Iteration speed
- Operational stability
- Lower infrastructure complexity

Instead of premature optimization or unnecessary distributed complexity.

---

# Architectural Philosophy

The platform architecture follows several core principles:

- Build operationally simple systems first
- Optimize retrieval quality before infrastructure complexity
- Prioritize tenant isolation
- Reduce moving parts
- Keep AI workflows modular
- Prefer evolutionary architecture over premature scaling

The project intentionally focuses on practical production engineering rather than overly theoretical AI infrastructure.

---

# Why Multi-Tenant Architecture?

DocCore was designed from the beginning as a multi-tenant SaaS platform.

Reasons:

- Shared infrastructure efficiency
- SaaS scalability
- Centralized deployment
- Easier operational management
- Lower infrastructure costs

The challenge of this approach is ensuring proper tenant isolation across:

- Retrieval systems
- Embeddings
- Documents
- AI workflows
- Usage tracking

The architecture intentionally treats tenant isolation as a first-class concern.

---

# Why Shared Infrastructure Instead of Dedicated Databases Per Tenant?

The platform currently uses shared infrastructure with logical isolation.

Reasons:

- Lower operational complexity
- Easier deployment
- Better cost efficiency
- Faster development iteration
- Simpler infrastructure management

Trade-offs:

- Stronger isolation logic required
- More careful authorization enforcement
- Retrieval isolation complexity

The current scale and workload profile do not justify dedicated infrastructure per tenant.

---

# Why PostgreSQL + pgvector?

One of the most important architectural decisions was using PostgreSQL with pgvector instead of adopting a dedicated vector database early.

Reasons:

- Unified relational + vector storage
- Reduced operational complexity
- Easier tenant-aware querying
- Simpler infrastructure
- Faster iteration speed
- Better developer ergonomics
- Lower operational cost

This architecture allows semantic retrieval while preserving traditional relational SaaS workflows.

---

# Why Not a Dedicated Vector Database?

Dedicated vector databases introduce additional complexity:

- More infrastructure components
- Synchronization complexity
- Higher operational cost
- Additional deployment overhead
- Multi-system consistency concerns

At the current stage of the platform, PostgreSQL + pgvector provides sufficient retrieval capabilities while keeping infrastructure manageable.

---

# Why Neon PostgreSQL?

The database infrastructure migrated from Render PostgreSQL to Neon.

Reasons:

- Lower operational cost
- Better free-tier ergonomics
- Serverless-oriented scaling behavior
- Simpler operational management
- Easier cost optimization

The migration reduced unnecessary always-on database costs while maintaining PostgreSQL compatibility.

---

# Why Render for Backend Hosting?

The backend infrastructure currently uses Render.

Reasons:

- Simpler deployment model
- Fast iteration speed
- Easy Docker deployment
- Good developer ergonomics
- Operational simplicity

Trade-offs:

- Always-on worker costs
- Less aggressive scale-to-zero behavior
- Limited serverless worker flexibility

The infrastructure remains intentionally simple during early-stage evolution.

---

# Why Redis + RQ Workers?

Background processing is handled through Redis queues and RQ workers.

Reasons:

- Async ingestion support
- Reduced API latency
- Better processing separation
- Easier retry workflows
- Simpler queue architecture

The worker system processes:

- Extraction
- Embeddings generation
- Semantic indexing
- Long-running document workflows

---

# Why Not Serverless Workers Yet?

Serverless/event-driven processing is still under evaluation.

Reasons for delaying migration:

- Simpler operational model
- Lower engineering overhead
- Faster implementation
- Easier debugging
- Existing worker stability

Trade-offs of always-on workers:

- Higher idle cost
- Persistent infrastructure billing
- Less efficient scale-to-zero behavior

Potential future exploration includes:

- QStash
- Event-driven processing
- Cloud Run
- Serverless ingestion workflows

---

# Why Sync SQLAlchemy Instead of Async?

The backend currently uses synchronous SQLAlchemy patterns.

Reasons:

- Simpler transaction handling
- Easier operational behavior
- Stable worker integration
- Lower concurrency complexity
- Easier debugging
- Better fit for current workload profile

Current workload bottlenecks are primarily:

- Retrieval operations
- Embedding generation
- Processing workflows

Rather than extreme concurrent API throughput.

---

# Why Layered Backend Architecture?

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

Reasons:

- Separation of concerns
- Cleaner boundaries
- Easier maintainability
- Infrastructure abstraction
- Better testing structure
- Scalable project organization

The architecture attempts to avoid tightly coupled monolithic business logic.

---

# Why Semantic Blocking Instead of Only Fixed Chunking?

Traditional chunking strategies often damage retrieval quality.

Problems include:

- Broken contextual continuity
- Split legal clauses
- Incomplete semantic meaning
- Weak retrieval grounding

DocCore explores semantic blocking approaches to preserve:

- Logical continuity
- Semantic cohesion
- Retrieval quality
- Contextual integrity

This becomes especially important for:

- Legal documents
- Structured reports
- Long-form textual analysis

---

# Why Retrieval-Grounded Generation?

The platform intentionally prioritizes retrieval-grounded AI generation.

Reasons:

- Reduced hallucination
- Better contextual accuracy
- More reliable outputs
- Safer document-aware AI workflows

The system attempts to avoid unrestricted language model generation without contextual evidence.

---

# Why Top-K Retrieval?

Top-K retrieval provides a practical balance between:

- Retrieval quality
- Prompt size
- Token usage
- Latency
- Contextual coverage

The architecture intentionally focuses on retrieving the most relevant contextual blocks before generation.

---

# Why Tenant-Scoped Retrieval?

Tenant-scoped retrieval is one of the most important security decisions in the platform.

Without proper retrieval isolation:

- Semantic leakage may occur
- Context contamination becomes possible
- AI workflows become unsafe

Retrieval queries apply tenant filters before vector similarity ranking.

This prevents:

- Cross-tenant retrieval
- Unauthorized contextual access
- Embedding contamination

---

# Why Retrieval Safety Matters in AI Systems

Traditional SaaS applications mainly protect CRUD operations.

AI systems additionally require protection for:

- Retrieval context
- Prompt construction
- Embeddings
- Semantic similarity operations
- Grounded generation

This creates significantly larger security surfaces.

---

# Why Restrictive Upload Policies?

Document upload pipelines intentionally use restrictive validation rules.

Reasons:

- Reduced attack surface
- Safer ingestion
- Controlled parsing complexity
- Reduced malicious upload risk

The platform intentionally avoids unrestricted executable uploads.

---

# Why Controlled Parsing Matters

Document parsing itself can become a security risk.

Potential concerns include:

- Parser vulnerabilities
- Malicious payloads
- Unsafe execution paths
- Excessive parsing complexity

Controlled ingestion strategies reduce these risks.

---

# Why Credit-Based Billing?

The billing system evolved toward a credit-oriented architecture.

Reasons:

- Flexible resource accounting
- Easier SaaS monetization
- Better workload abstraction
- More scalable usage modeling

The architecture separates:

- Processing credits
- Production credits
- Usage tracking
- Subscription periods

This provides more flexibility than rigid feature-based quotas.

---

# Why Async Document Processing?

Document ingestion is computationally expensive.

Examples:

- Extraction
- Embedding generation
- Semantic indexing
- Vector storage

Async processing allows:

- Reduced API latency
- Better scalability
- Improved reliability
- Decoupled workflows

---

# Why Operational Simplicity Is Prioritized

Many engineering decisions intentionally prioritize operational simplicity.

Examples:

- PostgreSQL + pgvector
- Shared infrastructure
- Sync ORM
- Worker-based ingestion
- Modular monolithic backend

Reasons:

- Faster iteration
- Easier debugging
- Lower operational burden
- Better maintainability

The platform intentionally avoids premature distributed complexity.

---

# Why Infrastructure Cost Optimization Matters

The project intentionally considers infrastructure cost efficiency.

Examples:

- Migration to Neon
- Shared infrastructure
- Unified storage systems
- Controlled worker architecture

The objective is maintaining a production-oriented architecture without excessive operational cost.

---

# Current Trade-Offs

Several trade-offs are intentionally accepted.

Examples include:

| Decision | Trade-Off |
|---|---|
| Shared DB | Requires stronger isolation logic |
| Sync ORM | Lower theoretical concurrency |
| Worker model | Always-on infrastructure cost |
| PostgreSQL + pgvector | Less specialized vector features |
| Simpler infra | Fewer advanced distributed capabilities |

These trade-offs are considered acceptable for the current platform stage.

---

# Current Architectural Limitations

Current areas under active evolution include:

- Retrieval precision
- Better ranking strategies
- Hybrid retrieval
- Event-driven processing
- Worker cost optimization
- Long-context optimization
- Better observability
- More advanced retrieval pipelines

---

# Future Exploration Areas

Potential future directions include:

- Hybrid retrieval systems
- Reranking pipelines
- Event-driven ingestion
- Serverless processing
- Better retrieval observability
- Advanced prompt orchestration
- Adaptive retrieval strategies
- More advanced authorization models

---

# Engineering Philosophy

DocCore intentionally prioritizes:

- Practical engineering
- Retrieval quality
- Operational simplicity
- Cost-aware infrastructure
- Modular architecture
- Tenant-safe AI workflows

Instead of prematurely pursuing maximum distributed complexity.

---

# Final Notes

The system architecture continues evolving through practical experimentation and production-oriented iteration.

The project focuses heavily on:

- Real-world RAG systems
- Retrieval quality
- Multi-tenant AI workflows
- SaaS architecture
- Production engineering trade-offs

The goal is building scalable AI document systems while maintaining operational simplicity, retrieval safety, and engineering maintainability.
