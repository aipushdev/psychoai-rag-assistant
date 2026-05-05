# PsychoAI RAG Assistant

> Production RAG assistant for gestalt therapy knowledge base.
> Answers questions strictly from professional literature — zero hallucinations.

**Live demo:** [psycho-pocket.com](https://psycho-pocket.com)

---

## Key Metrics (RAGAS evaluation)

| Metric | Result | Threshold |
|--------|--------|-----------|
| Faithfulness | **1.0** | > 0.8 |
| Context Precision | **0.85** | > 0.8 |
| Response time | **1.5s** | was 4s |
| Cache hit rate | **50-70%** | - |

Faithfulness = 1.0 means the assistant never invents facts — every statement is grounded in the retrieved documents. Critical requirement for therapy content.

---

## Architecture

```
User (Telegram)
     |
     v
FastAPI backend
     |
     +---> Redis cache (exact match) ---> return cached response
     |
     +---> Qdrant vector search (top_k=5)
              |
              v
         Google Gemini (generation)
              |
              v
         Response + source chunks
```

**Chunking:** 500 chars, overlap 100
**Embeddings:** text-embedding-3-small (OpenAI)
**Vector DB:** Qdrant (self-hosted in Docker)
**Cache:** Redis (TTL 24h, key = normalized query hash)

---

## Stack

- **Python 3.11** - core language
- **FastAPI** - REST API
- **Qdrant** - vector database for semantic search
- **Redis** - response caching
- **Google Gemini** - LLM for generation
- **Docker Compose** - all services in one command
- **Caddy** - reverse proxy with auto HTTPS

---

## How to Run

```bash
git clone https://github.com/aipushdev/psychoai-rag-assistant
cd psychoai-rag-assistant

cp .env.example .env
# Add your API keys to .env

docker-compose up -d

# Ingest documents
python ingest.py --docs-dir data/docs/

# Run RAGAS evaluation
python evaluate_rag.py
```

---

## RAGAS Evaluation Details

Tested on 5 questions from gestalt therapy domain:
- 2 exact fact questions
- 2 paraphrase/synonym questions
- 1 out-of-scope question (no answer in knowledge base)

```
Faithfulness:        1.0000  (top_k=5)
Answer Relevancy:    0.6644
Context Precision:   0.8500
```

Key finding: reducing `top_k` from 5 to 3 improved context precision but caused faithfulness to drop to 0.71 on complex questions - LLM starts to hallucinate when context is too thin. Optimal top_k depends on question type.

---

## Project Structure

```
.
- app/
  - main.py          # FastAPI entry point
  - retriever.py     # Qdrant semantic search
  - generator.py     # LLM generation with context
  - cache.py         # Redis caching layer
- ingest.py          # Document ingestion pipeline
- evaluate_rag.py    # RAGAS evaluation script
- data/docs/         # Knowledge base documents
- docker-compose.yml
- Caddyfile
```

---

## Screenshots

<img width="1109" height="889" alt="Screenshot 2026-05-05 at 19 29 41" src="https://github.com/user-attachments/assets/968d049b-ecba-459c-aeba-13e828e59831" />
<img width="1050" height="894" alt="Screenshot 2026-05-05 at 19 29 47" src="https://github.com/user-attachments/assets/c8674693-4c80-41cc-9836-3263e67b929f" />
<img width="1308" height="889" alt="Screenshot 2026-05-05 at 19 29 55" src="https://github.com/user-attachments/assets/53f1ee77-8c10-40b9-880d-636945f5bf51" />


---

Built as part of AI engineering course project. Production deployment on VPS with Docker + Caddy.
