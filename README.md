# AI Knowledge Base Agent (RAG + n8n + Supabase)

A production-ready Retrieval-Augmented Generation (RAG) system that ingests documents, stores embeddings in a vector database, and allows users to query the knowledge base through any chat interface with citations and source links.

The included example uses Telegram, but the trigger/interface layer can be replaced with Slack, WhatsApp, Microsoft Teams, a website chat widget, or any chat software with API access.

## Overview

This project demonstrates a complete AI knowledge base system:

- Automated document ingestion from Google Drive
- PDF/text extraction and document chunking
- LLM-generated section labels for better metadata
- Vector storage in Supabase using pgvector
- AI Agent with tool-based retrieval
- Chat-based querying interface
- Citation-aware answers with clickable source links

## Architecture

```text
Document Ingestion:
Google Drive → n8n → File Download → Text Extraction → Chunking → Section Labeling → Embeddings → Supabase Vector Store

Querying:
Chat Interface/API → n8n → AI Agent → Supabase Vector Tool → LLM Answer → Response with Citations
```

## Interface Flexibility

The querying workflow is interface-agnostic. Telegram is only one possible frontend.

You can replace the trigger and response nodes with:

- Slack
- WhatsApp API
- Microsoft Teams
- Discord
- Web chat UI
- Custom REST API
- Internal dashboard
- CRM chat widget

As long as the interface can send a message into n8n and receive a response, the same RAG logic can be reused.

## Tech Stack

- **n8n** — workflow orchestration
- **Supabase + pgvector** — vector database
- **Google Vertex AI** — embeddings (`gemini-embedding-001`)
- **Google Gemini / OpenRouter / OpenAI / Claude** — LLM response generation
- **Google Drive** — document source
- **Telegram Bot API** — example chat interface

## Repository Structure

```text
.
├── README.md
├── WRITEUP.md
├── workflows/
│   ├── document-ingestion.json
│   └── query-agent-chat-with-citations.json
├── supabase/
│   └── schema.sql
└── docs/
```

## Workflows Included

### 1. Document Ingestion Workflow

This workflow watches a Google Drive folder for new files, extracts document text, splits it into chunks, labels each chunk, generates embeddings, and stores everything in Supabase.

Main flow:

```text
Google Drive Trigger
→ Download File
→ Extract Text
→ Split Text into Chunks
→ Generate Section Label
→ Attach Metadata
→ Store in Supabase Vector Store
```

Each stored chunk includes:

```json
{
  "content": "Document chunk text...",
  "metadata": {
    "doc_name": "Original document name",
    "section": "Generated section label",
    "doc_url": "Source document link"
  },
  "embedding": "vector"
}
```

### 2. Query Agent Workflow

This workflow receives a user question, lets an AI Agent call the Supabase vector search tool, then returns an answer grounded only in retrieved documents.

Main flow:

```text
Chat Trigger
→ Clean Input
→ AI Agent
→ Supabase Vector Store Tool
→ Format Response
→ Send Chat Reply
```

Response format:

```text
Answer:
<answer based only on retrieved context>

Sources:
1. Document Name — Section
https://source-link.example
```

## Setup Instructions

### 1. Supabase Setup

Open Supabase SQL Editor and run:

```text
supabase/schema.sql
```

This creates:

- `documents` table
- `embedding vector(3072)` column
- `match_documents()` RPC function for similarity search

### 2. n8n Setup

Import both workflows from the `workflows/` folder.

Configure credentials for:

- Google Drive
- Supabase
- Google Vertex AI
- Your chosen LLM provider
- Your chosen chat interface

### 3. Configure Document Source

In the ingestion workflow, update the Google Drive folder ID to the folder that contains your knowledge base documents.

### 4. Configure Chat Interface

The provided workflow uses Telegram, but you can replace it with Slack, WhatsApp, Teams, or any API-based frontend.

For another interface, replace:

- Trigger node
- Input cleaning node fields if needed
- Final send message node

Keep the AI Agent and Supabase Vector Store tool unchanged.

## Key Features

- RAG-based AI answers
- Source-grounded responses
- Clickable source citations
- Metadata-aware retrieval
- Modular chat interface
- Reusable across different industries
- Production-style n8n architecture

## Use Cases

- Customer support knowledge base
- Internal SOP assistant
- HR policy assistant
- Sales enablement assistant
- Training and onboarding assistant
- Documentation chatbot
- Client-facing FAQ automation

## Important Notes

- The embedding model used during ingestion and querying must match.
- This setup uses `gemini-embedding-001`, which requires `vector(3072)` in Supabase.
- The `match_documents()` function must match the parameters expected by n8n: `query_embedding`, `match_count`, and `filter`.
- Citations depend on proper metadata: `doc_name`, `section`, and `doc_url`.

## Future Improvements

- Multi-turn memory
- Confidence scoring
- Result re-ranking
- Admin dashboard
- User feedback collection
- Multi-channel deployment
- Document deletion/update sync
- Usage analytics

## Author

Built as a reusable AI automation architecture for document search, support automation, and internal knowledge base assistants.
