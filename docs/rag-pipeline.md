# RAG Pipeline

This document describes the Retrieval-Augmented Generation (RAG) pipeline used in DocCore, including document ingestion, semantic segmentation, embeddings generation, vector retrieval, contextual assembly, and language model interaction workflows.

---

# Overview

The RAG pipeline is one of the central architectural components of DocCore.

The platform transforms uploaded files into semantically searchable contextual knowledge through:

- Text extraction
- Semantic segmentation
- Embedding generation
- Vector indexing
- Semantic retrieval
- Context assembly
- LLM grounding

The objective is enabling contextual AI interactions over uploaded documents while reducing hallucinations and improving retrieval precision.

---

# High-Level Pipeline

```text
Document Upload
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
pgvector Indexing
    ↓
Semantic Retrieval
    ↓
Top-K Context Selection
    ↓
Prompt Assembly
    ↓
LLM Response Generation
```

---

# Core RAG Goals

The retrieval pipeline was designed around several priorities:

- Contextual accuracy
- Retrieval grounding
- Reduced hallucination
- Semantic continuity
- Long-document understanding
- Tenant-safe retrieval
- Efficient context assembly
- Production-oriented scalability

---

# Why RAG?

Large language models alone do not reliably retain or understand user-specific uploaded documents.

Without retrieval augmentation:

- Context windows become limited
- Hallucination risk increases
- Uploaded knowledge becomes inaccessible
- Responses lose grounding
- Long-document analysis becomes unreliable

RAG enables the model to retrieve relevant contextual information dynamically before generation.

---

# Document Ingestion

The RAG workflow begins during document upload.

High-level ingestion flow:

```text
Upload
    ↓
Validation
    ↓
Extraction
    ↓
Segmentation
    ↓
Embeddings
    ↓
Storage
```

---

# Upload Validation

Before processing, uploads pass through validation rules.

Validation goals include:

- Restricting unsupported formats
- Preventing unsafe ingestion
- Reducing parsing complexity
- Improving extraction reliability

Validation examples:

- MIME validation
- File extension filtering
- File size limits
- Upload restrictions
- Controlled parsing rules

The platform intentionally avoids unrestricted executable uploads.

---

# Text Extraction

Uploaded files are transformed into normalized textual representations.

Extraction responsibilities:

- Preserve semantic meaning
- Maintain contextual continuity
- Remove parsing noise
- Standardize document structure

Extraction quality directly affects retrieval quality downstream.

---

# Normalization

After extraction, documents undergo normalization.

Normalization may include:

- Whitespace cleanup
- Character normalization
- Encoding cleanup
- Structural adjustments
- Text standardization

Goals:

- Improve embedding consistency
- Reduce noisy vectors
- Improve retrieval quality

---

# Semantic Blocking

DocCore explores semantic blocking strategies instead of relying exclusively on naive fixed-size chunking.

---

# Why Traditional Chunking Can Fail

Fixed-size chunking often introduces several retrieval problems:

- Broken legal clauses
- Split semantic meaning
- Context fragmentation
- Incomplete reasoning blocks
- Retrieval inconsistency

Example issues:

```text
Chunk A:
"The contract shall remain valid until..."

Chunk B:
"...termination conditions described in section 5."
```

Meaning becomes fragmented across chunks.

---

# Semantic Blocking Strategy

Semantic blocking attempts to preserve logical continuity before embeddings generation.

Objectives:

- Preserve semantic coherence
- Improve retrieval precision
- Reduce context fragmentation
- Improve long-document understanding
- Better contextual assembly

This is especially important for:

- Legal documents
- Contracts
- Structured reports
- Technical documentation
- Long-form analysis

---

# Embeddings Generation

After segmentation, semantic blocks are transformed into vector embeddings.

Embeddings represent semantic meaning numerically.

The embeddings layer enables:

- Semantic similarity search
- Context ranking
- Vector retrieval
- Retrieval grounding
- Semantic matching

---

# Vector Storage

Embeddings are stored in PostgreSQL using pgvector.

Why pgvector:

- Unified relational + vector storage
- Lower operational complexity
- Easier multi-tenant filtering
- Better SaaS ergonomics
- Simplified infrastructure

Current vector stack:

- PostgreSQL
- pgvector
- SQLAlchemy
- Neon

---

# Retrieval Flow

When a user submits a query:

```text
User Question
    ↓
Query Embedding
    ↓
Vector Similarity Search
    ↓
Top-K Retrieval
    ↓
Context Ranking
    ↓
Prompt Assembly
    ↓
LLM Generation
```

---

# Query Embeddings

User queries are transformed into embeddings using the same semantic space as indexed documents.

This enables:

- Semantic matching
- Contextual similarity
- Intent-aware retrieval

Instead of simple keyword matching.

---

# Semantic Similarity Search

The retrieval layer performs vector similarity search against indexed document embeddings.

Key concepts:

- Vector distance
- Similarity ranking
- Semantic matching
- Retrieval filtering
- Context scoring

The system retrieves the most semantically relevant contextual blocks.

---

# Tenant-Scoped Retrieval

All retrieval operations are tenant-aware.

Retrieval queries apply tenant filters before similarity ranking.

This prevents:

- Cross-tenant retrieval leakage
- Unauthorized contextual access
- Semantic contamination between organizations

Isolation is enforced throughout the retrieval pipeline.

---

# Top-K Retrieval

DocCore uses Top-K semantic retrieval strategies.

The retrieval engine selects the most relevant contextual blocks according to semantic similarity scores.

Example:

```text
Top 5 most relevant semantic blocks
```

The value of K affects:

- Retrieval quality
- Context density
- Prompt size
- Latency
- Hallucination risk

---

# Retrieval Trade-Offs

Choosing K introduces trade-offs.

---

## Lower K

Advantages:

- Lower latency
- Smaller prompts
- Reduced token usage
- Faster generation

Disadvantages:

- Missing relevant context
- Reduced completeness
- Higher risk of incomplete reasoning

---

## Higher K

Advantages:

- More contextual coverage
- Better reasoning potential
- Improved answer completeness

Disadvantages:

- Larger prompts
- Higher token cost
- More retrieval noise
- Increased hallucination risk if context quality is poor

---

# Context Ranking

Retrieved blocks are ranked before prompt assembly.

Goals:

- Prioritize relevance
- Reduce noisy context
- Improve grounding
- Improve answer quality

Current experimentation areas include:

- Better ranking heuristics
- Hybrid retrieval
- Reranking systems
- Context compression

---

# Context Assembly

After retrieval, contextual blocks are assembled into a prompt for the language model.

Primary goals:

- Preserve semantic continuity
- Maintain contextual relevance
- Reduce hallucination
- Improve answer consistency

The system attempts to provide grounded context rather than relying solely on model memory.

---

# Prompt Construction

The final prompt typically includes:

- User question
- Retrieved contextual blocks
- System instructions
- Retrieval constraints

The language model is instructed to answer primarily using retrieved context.

---

# Grounded Generation

The LLM response generation phase is retrieval-grounded.

This means the model operates with retrieved contextual evidence rather than relying exclusively on pre-trained internal knowledge.

Objectives:

- More reliable answers
- Better contextual accuracy
- Reduced hallucinations
- Improved factual grounding

---

# AI Chat Over Documents

The RAG pipeline powers contextual document chat.

Capabilities include:

- Question answering
- Semantic retrieval
- Multi-block reasoning
- Contextual responses
- Retrieval-grounded conversations

The platform focuses heavily on contextual grounding rather than generic conversational generation.

---

# Multi-Document Retrieval

The architecture supports retrieval across multiple documents within tenant boundaries.

This enables:

- Cross-document reasoning
- Contextual correlation
- Broader semantic retrieval
- Multi-source contextual analysis

---

# Artifact Extraction Workflows

The retrieval system also supports structured extraction workflows.

Examples:

- Summaries
- Structured outputs
- Contextual extractions
- AI-generated artifacts

This allows the platform to operate beyond traditional chat workflows.

---

# Async Processing Pipeline

Document indexing is handled asynchronously.

Current flow:

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
Storage
```

Goals:

- Reduced API latency
- Decoupled ingestion
- Better reliability
- Scalable processing
- Improved operational stability

---

# Retrieval Optimization Areas

Current optimization focus areas include:

- Better semantic segmentation
- Improved Top-K strategies
- Hybrid retrieval
- Reranking systems
- Context-window optimization
- Reduced retrieval noise
- Better context prioritization

---

# Hallucination Reduction

One of the primary goals of the retrieval pipeline is reducing hallucination risk.

Strategies include:

- Retrieval grounding
- Context filtering
- Tenant-scoped retrieval
- Structured context assembly
- Controlled prompt construction

The system intentionally prioritizes contextual grounding over unrestricted generation.

---

# Infrastructure Components

Current infrastructure involved in the RAG pipeline:

| Component | Role |
|---|---|
| FastAPI | API orchestration |
| PostgreSQL | Relational storage |
| pgvector | Vector retrieval |
| Redis | Queue infrastructure |
| RQ Workers | Async processing |
| Neon | Database hosting |
| Render | Backend infrastructure |

---

# Current Limitations

Current experimentation and iteration areas include:

- Retrieval precision
- Better ranking strategies
- Hybrid retrieval systems
- Semantic segmentation refinement
- Long-context optimization
- Context compression
- Better reranking pipelines

---

# Future Exploration

Potential future directions:

- Hybrid BM25 + vector retrieval
- Reranking models
- Event-driven ingestion
- Better semantic segmentation
- Adaptive Top-K retrieval
- Retrieval caching
- Context compression
- Advanced retrieval scoring

---

# Final Notes

The RAG pipeline is one of the most important architectural components of DocCore.

The project focuses heavily on retrieval quality, contextual grounding, semantic continuity, and production-oriented AI document workflows rather than generic chatbot behavior.

The objective is building reliable document-aware AI systems capable of scalable semantic retrieval and contextual reasoning over uploaded knowledge bases.
