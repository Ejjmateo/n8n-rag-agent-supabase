# Technical Write-Up: AI Knowledge Base Agent (RAG System)

## Overview

This project is a Retrieval-Augmented Generation (RAG) system that turns unstructured documents into a searchable AI knowledge base.

The system has two main workflows:

1. Document Ingestion Pipeline
2. Querying Agent with Citations

The goal is to let users ask natural-language questions and receive accurate answers grounded in uploaded documents, with citations and clickable source links.

## System Design Philosophy

The system is designed with separation of concerns:

- **Ingestion layer** — processes and stores knowledge
- **Retrieval layer** — searches relevant document chunks
- **Reasoning layer** — generates grounded answers
- **Interface layer** — handles user input/output

Because the interface layer is separate, the same AI knowledge base can be connected to Telegram, Slack, WhatsApp, a website chat widget, or any chat software with API access.

## Workflow 1: Document Ingestion Pipeline

### Purpose

The ingestion workflow converts uploaded documents into vector-searchable knowledge base records.

### Flow

```text
Google Drive Trigger
→ Download File
→ Extract Text
→ Split Text into Chunks
→ Generate Section Label
→ Merge Chunk + Metadata
→ Generate Embeddings
→ Store in Supabase
```

### Step-by-Step Breakdown

#### 1. Document Trigger

The workflow watches a Google Drive folder for newly uploaded files.

This trigger can be replaced with other document sources such as:

- Dropbox
- OneDrive
- Email attachments
- API upload endpoint
- Internal file management system

#### 2. File Download and Conversion

The workflow downloads the file from Google Drive. Google Docs can be converted into PDF format before text extraction.

#### 3. Text Extraction

The file content is extracted into plain text so it can be processed by the workflow.

#### 4. Chunking

Long documents are split into smaller overlapping chunks.

Default strategy:

- Chunk size: around 500 words
- Overlap: around 50 words

This improves retrieval accuracy because each chunk is small enough for semantic search but still carries surrounding context.

#### 5. Section Labeling

An LLM generates a short section label for each chunk.

Example labels:

- Cancellation Policy
- Account Setup Process
- Billing Guidelines
- Support Escalation Rules

These labels are stored as metadata and used later for citations.

#### 6. Metadata Attachment

Each chunk is stored with metadata:

```json
{
  "doc_name": "Original document name",
  "section": "Generated section label",
  "doc_url": "Source document link"
}
```

This is what allows the AI assistant to return traceable answers with citations.

#### 7. Embedding Generation

The text chunk is embedded using Google Vertex AI `gemini-embedding-001`.

#### 8. Vector Storage

The chunk, metadata, and embedding are stored in Supabase using pgvector.

## Workflow 2: Querying Agent with Citations

### Purpose

The querying workflow lets users ask questions through a chat interface and receive answers grounded in the knowledge base.

### Flow

```text
Chat Trigger
→ Clean Input
→ AI Agent
→ Supabase Vector Search Tool
→ Format Response
→ Send Reply
```

### Interface Layer

The included workflow uses Telegram as the chat interface, but this is interchangeable.

The same core workflow can be adapted to:

- Slack
- WhatsApp
- Microsoft Teams
- Discord
- Web chat
- REST API
- CRM inbox

Only the trigger and final response node need to change. The AI Agent and retrieval layer can remain the same.

### AI Agent Behavior

The AI Agent is instructed to:

- Always use the knowledge base search tool for document-related questions
- Never guess answers
- Use only retrieved context
- Say when information is not found
- Include citations in every answer

### Retrieval Tool

The Supabase Vector Store is attached to the AI Agent as a retrieval tool.

It searches the `documents` table using the `match_documents()` RPC function.

The retrieved results include:

- Relevant document content
- Document name
- Section label
- Source URL

### Response Format

The assistant returns answers in this structure:

```text
Answer:
<clear answer>

Sources:
1. Document Name — Section
Source Link
```

This makes the output more trustworthy because users can verify where the answer came from.

## Supabase Database Design

### `documents` Table

| Column | Type | Purpose |
|---|---|---|
| id | uuid | Unique record ID |
| content | text | Chunk text |
| metadata | jsonb | Document name, section, source URL |
| embedding | vector(3072) | Semantic vector embedding |

### `match_documents()` Function

The function enables similarity search from n8n.

Expected parameters:

```sql
match_documents(
  query_embedding vector(3072),
  match_count int,
  filter jsonb
)
```

## Key Design Decisions

### 1. Agent-Based Retrieval

Instead of manually retrieving chunks before calling the LLM, the AI Agent is given Supabase Vector Store as a tool.

This creates a cleaner and more extensible architecture.

### 2. Metadata-Driven Citations

Each chunk carries source metadata. This allows the assistant to cite answers instead of acting like a black-box chatbot.

### 3. Interface-Agnostic Design

Telegram is only the sample frontend. The system can be attached to any channel with API access.

### 4. Matching Embedding Dimensions

The system uses `gemini-embedding-001`, so the Supabase vector column is configured as `vector(3072)`.

## Common Issues Solved

### Supabase RPC Not Found

Resolved by creating the `match_documents()` function with the exact parameter names expected by n8n:

- `query_embedding`
- `match_count`
- `filter`

### Vector Dimension Mismatch

Resolved by aligning the embedding model and vector column size.

### Telegram Markdown Errors

Resolved by escaping Markdown characters or disabling parse mode.

### Lost Chat ID After AI Node

Resolved by referencing the original input node when formatting the final response.

## Production Considerations

For production use, consider adding:

- Document update/delete sync
- Access control by user or channel
- Logging of questions and answers
- Feedback buttons
- Confidence thresholds
- Admin review queue
- Error alerts
- Multi-turn conversation memory

## Conclusion

This project demonstrates a reusable AI knowledge base architecture using n8n, Supabase, embeddings, and LLM agents.

It can be adapted for customer support, internal operations, onboarding, HR, technical documentation, client portals, or any environment where users need reliable answers from approved documents.
