# RAG
Локальный RAG (BGE+Qdrant+Qwen)
# Local n8n Documentation RAG

**A customizable, fully local Retrieval-Augmented Generation (RAG) pipeline built with n8n, Qdrant, BGE-M3 embeddings, and Qwen.**

This project is a baseline implementation of a local AI knowledge assistant for n8n documentation. It provides a foundation that can be customized, extended, and adapted to different use cases, including technical support, internal knowledge bases, documentation search, and AI-powered workflow assistants.

The goal is to retrieve relevant documentation from a local vector database and use a local language model to generate context-aware answers without relying on a cloud-based LLM API.

## Features

- **Local document ingestion** — load documentation files from disk.
- **Text extraction** — extract readable content from source files.
- **Recursive text chunking** — split documents into smaller passages for retrieval.
- **Local embeddings** — use BGE-M3 to convert text into vector representations.
- **Vector storage and retrieval** — store and search document embeddings in Qdrant.
- **Local AI generation** — use Qwen through Ollama for answering questions.
- **Source-grounded responses** — retrieve relevant documentation before generating answers.
- **Hallucination checks** — test whether the assistant distinguishes missing information from documented facts.
- **Extensible architecture** — replace models, adjust chunking, add sources, or integrate additional workflows.

## Architecture

The project consists of two main workflows: document ingestion and question answering.

### 1. Document ingestion and indexing

`Manual Trigger → Read Files from Disk → Extract from File → Qdrant Vector Store`

Supporting components:

- **Default Data Loader** — prepares document content and metadata.
- **Recursive Character Text Splitter** — divides documents into smaller chunks.
- **BGE-M3 Embeddings** — generates vector embeddings.
- **Qdrant** — stores document vectors and associated payloads.

The ingestion workflow can be executed again when the knowledge base needs to be refreshed. For repeatable imports, point ID management and duplicate prevention should be configured according to the chosen ingestion strategy.

### 2. Question answering

`Chat Trigger → AI Agent → Local Qwen Chat Model`

The AI Agent uses a Qdrant Vector Store tool to retrieve relevant passages. The local chat model then generates a response based on the retrieved information and its system instructions.

## Technology Stack

| Component | Technology |
|---|---|
| Workflow orchestration | n8n |
| Vector database | Qdrant |
| Embedding model | BGE-M3 |
| Chat model | Qwen |
| Local model runtime | Ollama |
| Document splitting | Recursive Character Text Splitter |
| Interface | n8n Chat Trigger |

All inference components are designed to run locally. No external LLM API is required for the core embedding and generation pipeline.

## Chunking Configuration

The initial chunking configuration uses the following values:

| Parameter | Value |
|---|---:|
| Chunk size | 1,200 characters |
| Chunk overlap | 180 characters |
| Splitter | Recursive Character Text Splitter |

The overlap preserves some context between adjacent chunks, while the chunk size balances retrieval precision and the amount of context supplied to the language model.

These values are starting points rather than universal optima. Different document types may require different settings. Markdown headings, tables, code blocks, and source metadata should be preserved wherever possible.

## Vector Database Configuration

The initial indexed collection is `n8n_docs_v2`.

| Parameter | Configuration |
|---|---|
| Collection | `n8n_docs_v2` |
| Vector dimensions | 1,024 |
| Distance metric | Cosine |
| Embedding model | BGE-M3 |
| Recorded point count | 9,657 |

The collection was confirmed to have a green status and no pending updates at the time of inspection.

**Important:** the recorded point count describes the state of the collection at that time, not a permanent dataset size. The point count and collection configuration should be checked after each re-indexing operation.

The embedding model used for indexing must remain compatible with the model used for query embeddings. Changing embedding models may require re-embedding the collection.

## Testing and Validation

The prototype was tested with questions targeting n8n documentation and retrieval behavior.

### Test 1 — Webhook documentation retrieval

**Goal:** retrieve documentation about Webhook URLs and response handling.

**Observed result:** the assistant returned information about test and production URLs and referenced the Respond to Webhook node.

### Test 2 — Specific node parameters

**Goal:** identify the available `Respond With` options.

**Observed result:** Qdrant retrieved a relevant documentation passage containing the eight listed options: All Incoming Items, Binary File, First Incoming Item, JSON, JWT Token, No Data, Redirect, and Text.

This verified that the indexing and retrieval pipeline could return a relevant section containing a structured parameter list.

### Test 3 — Hallucination resistance

**Goal:** search for the deliberately fictional parameter `imaginaryWebhookQuantumMode`.

**Observed result:** the assistant reported that the parameter was not found in the retrieved documentation rather than inventing a configuration option.

This is a basic negative-retrieval test, not proof that the parameter is absent from every n8n version.

### Test 4 — Practical workflow generation

**Goal:** generate a webhook workflow that validates an email field and returns different HTTP status codes.

**Observed result:** the assistant proposed the general workflow structure, but some configuration details were not verified against retrieved documentation. This exposed an area for improvement in retrieval coverage and technical answer validation.

### Evaluation Summary

The tests confirmed that the system can retrieve relevant documentation passages and answer basic documentation questions. They also showed that complex implementation instructions require additional validation.

The current implementation is a working baseline, not a claim of complete documentation coverage or production-grade accuracy.

## Customization and Extension

The project is intentionally designed as a starting point rather than a fixed solution.

Possible extensions include:

- **Better retrieval:** experiment with chunk sizes, overlap, metadata filters, and reranking.
- **Markdown-aware splitting:** preserve section hierarchy and heading context.
- **Source citations:** include document paths or URLs in generated answers.
- **Evaluation datasets:** test retrieval relevance, answer correctness, and unsupported claims.
- **Additional document sources:** ingest API references, internal documentation, PDFs, or Markdown repositories.
- **Alternative local models:** compare embedding and chat models without redesigning the entire workflow.
- **Production hardening:** add scheduled ingestion, duplicate prevention, error handling, monitoring, and automated tests.

## Setup Overview

1. Install and configure n8n, Qdrant, and Ollama.
2. Make the documentation directory accessible to n8n.
3. Download the selected local embedding and chat models.
4. Configure BGE-M3 embeddings with the Qdrant ingestion workflow.
5. Run document extraction, chunking, embedding, and indexing.
6. Configure the AI Agent to use Qwen and the Qdrant retrieval tool.
7. Test with known documentation questions and questions that should return no supporting evidence.

Model names, service URLs, collection names, and directory paths may need to be adjusted for the local environment.

Never commit API keys, passwords, private paths, or other sensitive configuration values.

## Project Status

**Status: Functional baseline / experimental**

The ingestion and retrieval pipeline has been exercised with real n8n documentation. Initial tests demonstrate relevant retrieval and basic hallucination resistance. More comprehensive evaluation, source attribution, and workflow-level validation remain opportunities for improvement.

The purpose of this repository is to provide a reproducible foundation for experimenting with local RAG and n8n automation.
