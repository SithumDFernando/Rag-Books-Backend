# Backend System Architecture: Self-Help Books RAG

## 1. Overview
The backend is a Python-based REST API built with FastAPI. It handles document ingestion, LLM orchestration via LangChain & LangGraph, and vector storage using ChromaDB. It serves as the intelligent core for the visual dashboard, processing self-help books and answering queries.

## 2. Tech Stack
- **Web Framework:** FastAPI (Asynchronous, fast REST API)
- **AI / Orchestration:** LangChain (LLM wrappers, vector operations) and LangGraph (Agentic routing for QA vs. Summarization workflows)
- **LLM Providers:** OpenAI and Google Gemini (Swappable via configuration)
- **Vector Database:** ChromaDB (Local, memory/file-based vector storage)
- **Relational Database:** SQLite (Local storage for metadata: list of books, query logs, processing status)
- **Document Processing:** PyPDF and Unstructured (for parsing PDFs/EPUBs)
- **Testing:** Pytest

## 3. Core Features
- **Dual-LLM Support:** A centralized LLM factory that dynamically switches between OpenAI and Gemini based on `.env` configuration.
- **Ingestion Guardrails:** Before fully processing a document, an LLM call analyzes the first few pages to verify it is a "non-fiction self-help book". If it is fictional or unrelated, the ingestion is aborted, and an error is returned to the client.
- **RAG & Agentic Workflow:** LangGraph routes user requests. If the user asks a specific question, it triggers a Retriever Node to fetch context from ChromaDB. If the user asks for a summary, it triggers a Summarization Node (Map-Reduce or Refine approach).
- **Knowledge Base API:** Endpoints to list all successfully ingested books, retrieve their metadata, and delete them.

## 4. File & Folder Structure
```text
backend/
├── app/
│   ├── main.py                 # FastAPI application instance and entrypoint
│   ├── api/                    # API Routers
│   │   ├── routes_chat.py      # Q&A and Summarization endpoints
│   │   ├── routes_books.py     # Upload, Guardrails, and Knowledge Base listing
│   ├── core/                   # App-wide configurations
│   │   ├── config.py           # Environment variables and LLM config switch
│   │   ├── guardrails.py       # LLM logic to classify/reject fictional books
│   ├── db/                     # Database connections
│   │   ├── sqlite_manager.py   # SQLite connection and CRUD operations
│   │   ├── vector_store.py     # ChromaDB wrapper for embeddings
│   ├── services/               # Core business logic
│   │   ├── ingestion.py        # PDF parsing, text chunking, and embedding
│   │   ├── langgraph_workflow.py # Complex routing (QA vs Summary)
│   │   ├── llm_factory.py      # Instantiates OpenAI or Gemini models
├── docs/
│   ├── system_architecture.md  # This document
├── tests/                      # Pytest test suite
│   ├── test_guardrails.py      # Mocks for fictional vs non-fiction inputs
│   ├── test_langgraph.py
├── .env                        # Environment variables (not checked into git)
├── requirements.txt            # Python dependencies
```

## 5. API Endpoints (Planned)
- `POST /api/books/upload`: Accepts a PDF file, runs guardrails, chunks, embeds, and saves to ChromaDB & SQLite.
- `GET /api/books`: Returns a list of all books in the knowledge base.
- `POST /api/chat`: Accepts a query, routes via LangGraph, and returns the LLM response.
