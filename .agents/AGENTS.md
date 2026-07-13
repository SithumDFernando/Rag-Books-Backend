# Backend Agent Rules

Welcome to the backend repository of the Self-Help Books RAG system. As an AI Agent working on this repository, you must adhere strictly to the following rules:

## 1. Type Hinting & Validation
- **Strict Typing:** All Python functions, methods, and variables MUST have explicit type hints.
- **Pydantic V2:** Use Pydantic V2 for all data validation, request/response models, and configuration management. Do not use standard `dataclasses` unless absolutely necessary.
- **FastAPI Dependencies:** Always use FastAPI's `Depends()` for database sessions (SQLite) and external service clients (ChromaDB, LLM Factory).

## 2. LLM Agnosticism & Configuration
- **Dual-LLM Support:** The system supports both OpenAI and Google Gemini.
- **Configuration Switch:** You MUST read from the central configuration (`core.config`) to determine which LLM is active.
- **LLM Factory:** Never instantiate `ChatOpenAI` or `ChatGoogleGenerativeAI` directly in your business logic. Always use the `llm_factory` service to obtain the configured LLM instance. This ensures the system remains completely agnostic to the underlying provider.

## 3. RAG Architecture & LangGraph
- **Agentic Workflows:** Use LangGraph to define the orchestration flow (routing between Q&A and Summarization). Do not use basic `SequentialChain` or `RunnableSequence` for top-level logic.
- **Guardrails:** Always enforce the Ingestion Guardrail BEFORE chunking and embedding. The guardrail should be an LLM call that returns a structured Pydantic response (e.g., `is_valid: bool`, `reason: str`).

## 4. Testing & Mocking
- **Test-Driven:** Write robust unit tests using `pytest`.
- **Mock Everything External:** You MUST aggressively mock all calls to the LLM (OpenAI/Gemini) and the vector database (ChromaDB) in your tests. Never make live network calls during tests. Use `unittest.mock.patch` or pytest fixtures.
- **Guardrail Testing:** Specifically test the Guardrails by providing mocked inputs of fictional books to ensure the system correctly raises a 422 error.

## 5. Localhost First
- **No Cloud Dependencies:** Keep all storage local. Use SQLite for relational metadata and ChromaDB in local-persistent mode (writing to a local directory like `/data/chroma`).
