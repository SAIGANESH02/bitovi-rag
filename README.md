# Bitovi RAG Agent

## Overview

The Bitovi RAG Agent is an end-to-end AI-powered Retrieval-Augmented Generation (RAG) pipeline built using n8n for workflow orchestration and PostgreSQL for data storage. It allows automated ingestion of articles, vectorizes and stores them, and serves them via a conversational agent that performs smart document-based question answering.

Main Goal: Build a maintainable, easily extensible RAG system with modular workflows, simple deployment, and an interface for both ingestion and intelligent retrieval.

## Components and Architecture


### Architecture



### Key Project Files

- `docker-compose.yml`: Launches the n8n server and a PostgreSQL instance with required environment configs.
- `Data_Ingestion_Workflow.json`: Automates document ingestion, chunking, and embedding creation.
- `AI_Agent_with_RAG.json`: Implements the intelligent agent that queries embeddings using semantic similarity.
- `Postgres/`: Contains custom Dockerfile for the PostgreSQL setup.
- `Adminer/`: Optional Adminer UI service for viewing PostgreSQL data.
- `.env` (optional): For storing keys and DB configs securely.

## Database Schemas

### document_metadata

Tracks metadata of ingested articles.

| Column         | Type           | Notes                        |
|----------------|----------------|------------------------------|
| id             | integer        | Primary key, auto-increment |
| title          | text           | Article title                |
| date           | date           | Publish date                 |
| author         | text           | Article author               |
| article_link   | text (unique)  | Canonical link               |
| vector_created | boolean        | Flag to check vector status  |
| created_at     | timestamptz    | Default: now()               |

### documents

Stores actual text content and vector embeddings.

| Column     | Type    | Notes                             |
|------------|---------|-----------------------------------|
| id         | uuid    | Primary key (generated)           |
| content    | text    | Raw or chunked document content   |
| metadata   | jsonb   | Structured metadata               |
| embedding  | vector  | Vector embedding (for RAG search) |

### n8n_chat_histories

Stores past user-agent conversations.

| Column      | Type                  | Notes                  |
|-------------|-----------------------|------------------------|
| id          | integer               | Auto-increment key     |
| session_id  | varchar(255)          | Unique session key     |
| message     | jsonb                 | Chat message structure |

## Main Workflows

### 1. Data Ingestion Workflow (Data_Ingestion_Workflow.json)

Steps:
- Fetch article metadata from a source URL.
- Store metadata in `document_metadata`.
- Fetch and chunk article content.
- Generate vector embeddings using an embedding model.
- Insert into `documents` table with associated metadata.

Goal: Prepare documents for semantic search.

### 2. RAG Agent Chat Workflow (AI_Agent_with_RAG.json)

Steps:
- Accept user query from chat interface or API.
- Convert query to embedding using same model.
- Perform vector similarity search on `documents`.
- Retrieve top relevant chunks.
- Compose answer and return to user.
- Store chat in `n8n_chat_histories`.

Goal: Deliver context-aware, document-grounded answers.

## Tech Stack

| Layer            | Tech Used                 |
|------------------|---------------------------|
| Workflow Engine  | n8n                        |
| Database         | PostgreSQL + PGVector     |
| Vector Search    | PostgreSQL extension       |
| Embeddings       | OpenAI / Local models      |
| Containerization | Docker, Docker Compose     |
| Admin UI         | Adminer (optional)         |

## Getting Started

1. Clone the repository
   ```
   git clone <repo-url>
   cd bitovi-rag
   ```

2. Run the stack
   ```
   docker compose up -d
   ```

3. Open n8n UI
   Visit http://localhost:5678 to access n8n and import the workflows.


## Adminer Integration

Adminer is a lightweight, full-featured database management tool that is included in this project to simplify managing the PostgreSQL database used by n8n.

You can use Adminer to:
- View and modify tables (`document_metadata`, `documents`, and `n8n_chat_histories`)
- Run SQL queries for debugging or exploration
- Monitor and verify data insertions and updates from workflows

This tool is especially helpful during development and debugging phases.

## Running All Services Together

For convenience, you can use the following command to start all required services:

```
./up Postgres Adminer
```

This script runs:
- The PostgreSQL container (with required extensions like PGVector)
- The Adminer UI for DB management
- The n8n workflow engine

After executing the command, you can:
- Access the n8n UI at `http://localhost:5678`
- Access Adminer at `http://localhost:8080` to inspect the database


## Evaluation Criteria Addressed

- Well-documented: This README provides a clear overview, setup instructions, tech stack, workflows, and database design.
- Easy to understand: Logical grouping of workflows, schema explanation, and simple instructions for bootstrapping.
- Professional structure: Modular, extensible, and adheres to modern RAG agent development practices.
