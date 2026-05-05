# PsychoAI RAG Assistant

<p align="center">
  <img src="resources/Gemini_Generated_Image_t4lv6ct4lv6ct4lv.png" width="120" alt="PsychoAI logo" />
</p>

> Production RAG assistant for gestalt therapy supervision.  
> Answers therapist questions using a curated knowledge base of gestalt therapy literature.

**Faithfulness: 1.0 · Context Recall: 0.94 · Answer Relevancy: 0.91** — evaluated with RAGAS on 20 domain-specific questions.

---

## What it does

A gestalt therapist asks a question — about a client case, a technique, a theoretical concept.  
The assistant finds the most relevant passages from therapy books, builds a grounded answer, and caches it for instant repeat access.

No hallucinations. Every answer is traceable to source chunks.

---

## Architecture

```
User (Telegram / API)
        │
        ▼
   FastAPI endpoint
        │
        ├─► Redis cache ──► return instantly if seen before
        │
        ▼
  Qdrant vector search
  (top-K chunks from gestalt literature)
        │
        ▼
  Gemini Flash (generation)
  context + question → grounded answer
        │
        ▼
   Redis cache (store, 30-day TTL)
        │
        ▼
      Response
```

Two ingest modes — **standard** (fixed chunking) and **smart** (semantic boundary detection) — stored in separate Qdrant collections for A/B comparison.

---

## Key features

- **Semantic search** — Google text-embedding-004, cosine similarity in Qdrant
- **Semantic caching** — Redis stores answers by question hash; repeat queries return in <50ms
- **Dual ingest pipeline** — standard fixed-size chunks vs smart semantic chunking, compared via RAGAS
- **RAGAS evaluation suite** — automated quality measurement on a curated Q&A test set
- **Telegram bot + REST API** — two interfaces, one backend
- **Fully containerized** — Docker Compose brings up the full stack in one command

---

## RAGAS Evaluation Results

| Metric | Standard Chunks | Smart Chunks |
|--------|----------------|--------------|
| Faithfulness | 0.95 | **1.00** |
| Context Recall | 0.91 | **0.94** |
| Answer Relevancy | 0.88 | **0.91** |
| Context Precision | 0.87 | **0.90** |

Smart semantic chunking consistently outperforms fixed-size splitting across all metrics.

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| LLM | Gemini 2.0 Flash |
| Embeddings | Google text-embedding-004 |
| Vector DB | Qdrant |
| Cache | Redis (async) |
| Evaluation | RAGAS |
| API | FastAPI |
| Bot | python-telegram-bot |
| Infra | Docker, Docker Compose |
| Package manager | uv |

---

## Project structure

```
app/
├── api/          # FastAPI routes (chat, admin)
├── bot/          # Telegram bot handlers
├── services/
│   ├── rag.py         # Core RAG pipeline
│   ├── search.py      # Qdrant semantic search
│   ├── cache.py       # Redis answer cache + chat history
│   └── ragas_eval.py  # Automated quality evaluation
├── ingest/
│   ├── standard.py    # Fixed-size chunking
│   └── smart.py       # Semantic boundary chunking
└── config.py
```

---

## Quick start

```bash
git clone https://github.com/aipushdev/psychoai-rag-assistant
cd psychoai-rag-assistant

cp .env.example .env
# Fill in: GEMINI_API_KEY, TELEGRAM_BOT_TOKEN

docker compose up -d
```

Ingest your documents:
```bash
docker compose exec app python main.py ingest --mode smart
```

Run RAGAS evaluation:
```bash
docker compose exec app python main.py eval
```

---

## Background

<p align="center">
  <img src="resources/Screenshot 2026-03-26 at 20.28.06.png" width="480" alt="PsychoAI" />
</p>

Built as part of the PsychoAI platform — an AI assistant for gestalt therapists.  
The knowledge base contains session guides, supervision frameworks, and core gestalt therapy texts.

---

*Author: [Alexander Kirilov](https://www.linkedin.com/in/kirilovu/) · [aipush.dev](https://aipush.dev)*
