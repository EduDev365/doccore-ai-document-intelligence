# DocCore — AI Document Intelligence Platform

AI-powered document intelligence platform focused on Retrieval-Augmented Generation (RAG), semantic retrieval, document analysis, contextual chat, and multi-tenant SaaS architecture.

---

## Overview

DocCore is a production-oriented AI document platform designed to process, index, retrieve, and analyze unstructured files using modern LLM and vector retrieval pipelines.

The system was built around a document-centric AI architecture where uploaded files become searchable semantic knowledge bases through embeddings, retrieval pipelines, contextual prompting, and structured extraction workflows.

Unlike simple chatbot wrappers, DocCore focuses heavily on:

* Multi-tenant isolation
* Secure document processing
* Semantic retrieval
* Context-aware RAG pipelines
* Background processing
* Credit-based SaaS architecture
* Production backend patterns
* Infrastructure cost optimization

The platform is designed as both a SaaS product and an engineering case study around applied AI systems.

---

# Live Demo

[https://doccore.com.br](https://doccore.com.br)

---

# Screenshots

> Screenshots and architecture diagrams will be added in the `/media` folder.

Planned showcase:

* Dashboard
* Upload flow
* Retrieval chat
* Processing pipeline
* Multi-tenant workspace
* Architecture diagrams
* Embedding + retrieval flow

---

# Core Features

## AI Document Intelligence

* Retrieval-Augmented Generation (RAG)
* Semantic document search
* Contextual document chat
* Embedding-based retrieval
* AI artifact extraction
* Structured document analysis
* Long-context retrieval workflows
* Multi-document contextual reasoning

---

## Multi-Tenant SaaS Architecture

DocCore was designed from the beginning as a multi-tenant platform.

Key characteristics:

* Tenant-scoped document ownership
* Tenant-isolated retrieval
* Membership and permission systems
* Per-tenant usage tracking
* Tenant-aware billing and quotas
* Isolated retrieval pipelines
* Scoped vector search

Isolation is enforced at both:

* Application layer
* Retrieval/query layer

This prevents semantic leakage between organizations and ensures retrieval operations are always tenant-aware.

---

## Retrieval-Augmented Generation (RAG)

The retrieval pipeline is one of the central architectural components of DocCore.

High-level flow:

```text
Document Upload
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
Top-K Semantic Retrieval
    ↓
Context Assembly
    ↓
LLM Response Generation
```

The platform avoids naive document processing strategies and instead focuses on retrieval quality and contextual consistency.

---

# Semantic Blocking Strategy

Instead of relying exclusively on fixed-size chunking, DocCore explores semantic blocking approaches.

The objective is improving:

* Retrieval precision
* Context cohesion
* Semantic continuity
* Long-document understanding
* Reduced context fragmentation

Traditional chunking strategies often create:

* Broken legal clauses
* Incomplete sections
* Context loss
* Retrieval inconsistencies

Semantic blocking attempts to preserve logical structure and contextual continuity before embedding generation.

This becomes especially important for:

* Legal documents
* Contracts
* Structured reports
* Technical documentation
* Long-form textual analysis

---

# Semantic Retrieval

DocCore uses embedding-based semantic retrieval instead of keyword-only search.

Key retrieval concepts:

* Vector similarity search
* Embedding indexing
* Context ranking
* Top-K retrieval
* Similarity scoring
* Retrieval filtering
* Tenant-scoped search

The retrieval layer uses Top-K semantic matching to retrieve the most relevant contextual blocks before assembling prompts for the language model.

Current retrieval focus areas:

* Better context ranking
* Retrieval precision optimization
* Hybrid retrieval experimentation
* Context-window efficiency
* Reduced hallucination risk

---

# pgvector + PostgreSQL

The vector layer is built on PostgreSQL with pgvector.

Reasons for this architectural choice:

* Operational simplicity
* Unified relational + vector storage
* Lower infrastructure complexity
* Easier SaaS deployment
* Cost efficiency
* Good developer ergonomics

Current database stack:

* PostgreSQL
* pgvector
* SQLAlchemy
* Alembic migrations
* Multi-tenant relational modeling

The infrastructure recently migrated from Render PostgreSQL to Neon for lower operational costs and simpler scaling.

---

# AI Chat Over Documents

DocCore enables contextual conversations over uploaded files.

Chat capabilities include:

* Contextual question answering
* Retrieval-grounded responses
* Multi-block context assembly
* Conversation continuity
* Document-aware reasoning
* Retrieval-constrained prompting

The platform focuses on reducing hallucinations by grounding model responses in retrieved contextual data.

---

# Artifact Extraction

Beyond conversational retrieval, the system also supports structured artifact generation workflows.

Examples include:

* Structured summaries
* Key information extraction
* Contextual outputs
* AI-assisted document workflows
* Structured response generation

This enables workflows beyond simple chat interfaces.

---

# Background Processing Architecture

Document processing runs asynchronously through worker-based pipelines.

Current workflow:

```text
Upload
    ↓
Queue
    ↓
Worker Processing
    ↓
Extraction
    ↓
Blocking
    ↓
Embeddings
    ↓
Vector Storage
```

Key goals:

* Decoupled ingestion
* Better reliability
* Async processing
* Scalable indexing
* Reduced request latency

Current infrastructure:

* Redis queues
* Background workers
* Async document ingestion pipeline

Future roadmap includes event-driven processing experimentation.

---

# Security Considerations

Security and isolation are considered first-class architectural concerns.

Current security-focused areas:

* JWT authentication
* HttpOnly cookies
* Tenant-scoped access
* Upload validation
* Restricted file formats
* Retrieval isolation
* Permission validation
* Environment secret isolation
* Server-side validation

---

# File Upload Restrictions

To reduce abuse and malicious uploads, document ingestion applies restrictions and validation rules.

Examples include:

* Allowed MIME type validation
* Restricted file extensions
* File size limits
* Upload validation
* Parsing restrictions
* Sanitized processing flow

The platform intentionally avoids unrestricted executable uploads.

Accepted document categories are focused on structured textual analysis workflows.

---

# Virus and Malicious Content Prevention

DocCore avoids unsafe upload behavior through controlled ingestion strategies.

Current defensive approaches include:

* Restricted upload formats
* Non-executable document policy
* Server-side validation
* Controlled extraction pipelines
* Limited parsing surface

The architecture intentionally minimizes direct execution risks from uploaded content.

---

# Authentication and Access Control

Authentication is handled through JWT-based flows.

Current approach includes:

* JWT authentication
* HttpOnly cookies
* Tenant membership validation
* Permission-aware endpoints
* Session-aware APIs

The authorization layer is integrated with the multi-tenant model.

---

# Billing Architecture

The platform uses a credit-oriented billing strategy.

Core concepts:

* Usage tracking
* Processing credits
* Production credits
* Subscription periods
* Usage ledger
* Tenant-level quotas

This architecture allows flexible resource accounting for:

* Processing pipelines
* Retrieval workloads
* AI generation workflows

---

# Backend Architecture

The backend follows layered architectural principles.

Current structure includes:

```text
application/
core/
domain/
infrastructure/
interfaces/
shared/
workers/
```

Goals:

* Separation of concerns
* Infrastructure abstraction
* Easier maintainability
* Modular AI workflows
* Scalable SaaS architecture

---

# Technology Stack

## Backend

* FastAPI
* SQLAlchemy
* Alembic
* PostgreSQL
* pgvector
* Redis
* RQ Workers

---

## Frontend

* Next.js
* App Router
* TailwindCSS
* React

---

## Infrastructure

* Render
* Neon
* Vercel
* Redis
* Docker Compose

---

# Engineering Decisions

## Why PostgreSQL + pgvector?

Instead of adopting a fully separate vector database early, DocCore uses PostgreSQL + pgvector to:

* Reduce operational complexity
* Keep relational and vector data unified
* Simplify deployments
* Reduce infrastructure costs
* Accelerate iteration speed

---

## Why Sync SQLAlchemy?

The backend currently uses synchronous SQLAlchemy patterns.

Reasons include:

* Simpler operational behavior
* Easier worker integration
* Lower concurrency complexity
* Stable development ergonomics
* Better fit for current workload profile

The current bottlenecks are retrieval and processing workloads rather than extreme concurrent API traffic.

---

## Why Background Workers?

Embedding generation and retrieval preparation are computationally heavier operations.

Using worker pipelines allows:

* Async ingestion
* Reduced API latency
* Better processing separation
* Retry strategies
* Decoupled document indexing

---

## Why Multi-Tenant Isolation Matters

AI document systems handling organizational data must avoid contextual leakage.

DocCore treats tenant isolation as a core architectural requirement instead of an afterthought.

Retrieval queries, memberships, document ownership, and contextual assembly are tenant-aware.

---

# Current Limitations

As an evolving platform, several areas are still under active iteration.

Current limitations and experimentation areas:

* Retrieval precision improvements
* Better ranking strategies
* Advanced hybrid retrieval
* Event-driven workers
* Retrieval latency optimization
* Embedding strategy experimentation
* Long-context optimization

---

# Future Roadmap

Planned exploration areas:

* Event-driven processing pipelines
* Hybrid retrieval systems
* Better ranking and reranking
* Advanced artifact generation
* Improved semantic segmentation
* Enhanced observability
* More advanced tenant analytics
* Workflow automation

---

# Architecture Diagram

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

# Repository Structure

This public repository intentionally focuses on:

* Architecture
* Engineering decisions
* System design
* Showcase material
* Screenshots and demonstrations

The production SaaS codebase remains private.

---

# Project Goals

The primary engineering goals behind DocCore include:

* Building production-oriented AI systems
* Exploring real-world RAG architectures
* Understanding retrieval quality trade-offs
* Designing scalable document pipelines
* Combining SaaS architecture with applied AI workflows

---

# Status

Active development and experimentation.

The platform continues evolving around retrieval quality, document intelligence workflows, infrastructure simplification, and production-oriented AI engineering.
