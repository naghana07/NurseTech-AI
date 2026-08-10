# NurseTech-AI (Nightingale) – Version 2

This folder contains the revised architecture, documentation, implementation plan and materials for Version 2.


# NurseMentor AI (Nightingale — Evidence-Grounded Multi-Agent Nursing Tutor)



## TechCoder Drive Details

https://drive.google.com/drive/folders/1sq4PynlXczjjx7nx7Fws2Ky-uY4fdgUd?usp=sharing



## Project PPT Link

https://github.com/naghana07/NurseTech-AI/tree/main/PPT



## Demo Video Link

https://drive.google.com/file/d/1cKUhxYmjjJbqeIr6ExWZDZUkYt1ehjOJ/view?usp=sharing



An evidence-grounded autonomous multi-agent adaptive nursing education framework in which
a state-aware orchestration agent dynamically coordinates learner modelling, clinical
reasoning, assessment, feedback, planning and progress analytics, while a
retrieval-augmented knowledge layer grounds clinical outputs in traceable nursing evidence
and an independent evidence-validation mechanism estimates response support.

This is the rebuild that incorporates your mentor's revised methodology (see
"What changed" below) — it's now a real Python backend with real RAG, not a single HTML
file calling the LLM directly.

## Architecture (maps directly to the diagram)

```
NURSING STUDENT → USER INTERFACE (frontend/index.html)
                        │
                        ▼
              Secure Backend / API Layer (backend/main.py, FastAPI)
                        │
                        ▼
          AUTONOMOUS AGENT CONTROLLER (backend/controller.py)
     Understand Goal → Decompose Task → Select Agent → Execute →
              Monitor → Critique/Reflect → Decide Next Action
                        │
        ┌───────────────┼────────────────────────────┐
        ▼               ▼                            ▼
  Learner Agent   Clinical Reasoning            Quiz Generator
  Feedback Agent  Learning Planner              Progress Analytics
  (backend/agents/*.py)
                        │
                        ▼
          KNOWLEDGE & RAG LAYER (backend/rag/*.py)
  Documents (backend/documents/) → Chunks → Embeddings (sentence-transformers,
  local, free) → Vector Knowledge Store (Chroma, embedded) → Hybrid Semantic
  Retrieval (vector + BM25) → Re-ranking (cross-encoder) → Evidence Pack
                        │
                        ▼
             Specialist LLM Agent (Anthropic API call)
                        │
                        ▼
     Evidence Validator → Evidence Support Score (computed, not self-reported)
                        │
                        ▼
        Personalized Response → Learner State Updated (SQLite) → Controller
```

## What changed from the earlier single-HTML prototype

| Original limitation | Resolved by |
|---|---|
| Rule-based JS orchestration | `controller.py` — a real Planner→Router→Executor→Critic loop, with retry when quality checks fail |
| Simulated knowledge base (static category list) | `rag/ingest.py` + `rag/retrieval.py` — real documents, chunked, embedded, retrieved, re-ranked |
| No RAG | Full ingestion → embeddings → vector store → hybrid retrieval → re-ranking → evidence pack |
| Self-generated trust score | `rag/evidence_validator.py` — independently embeds the response and the evidence, computes a cosine-similarity-based **Evidence Support Score** |
| JS-only agent controller | `controller.py` running server-side in FastAPI, with state persisted in SQLite (`memory.py`) |

## Why this vector store / stack

For 20–50 documents, a heavyweight vector database (FAISS/Pinecone) buys nothing — see
the trade-off discussion you asked about. This uses **ChromaDB in embedded/local mode**:
zero servers to run, persists to disk in `backend/vector_store/`, and is simple enough to
explain in a methodology write-up. Embeddings and re-ranking run locally via
`sentence-transformers` (no extra API key, no cost, no network dependency beyond the
one-time model download).

## Setup — Anaconda Navigator / Jupyter Notebook (recommended for this project)

Open **`Nightingale_Setup_and_Run.ipynb`** in Jupyter Notebook (launched from Anaconda
Navigator) and run the cells top to bottom — it walks through creating the conda
environment, installing dependencies, adding your API key, building the RAG index, and
starting the server, all with explanations for each step. That notebook is the primary,
novice-friendly setup path for this project.

The backend now serves the frontend itself from the same URL
(`http://127.0.0.1:8000/`) — no separate file to open, no CORS configuration needed.

## Setup — command line (alternative)

```bash
cd backend
conda create -n nightingale python=3.11 -y && conda activate nightingale   # or use venv
pip install -r requirements.txt
cp .env.example .env        # then paste your Anthropic key into .env
uvicorn main:app --reload --port 8000
```

Then open **http://127.0.0.1:8000/** in a browser — that single URL serves both the app
and the API.

First startup automatically ingests everything in `backend/documents/` into the vector
store (downloads the small local embedding model the first time — under 100 MB, one time
only).

To re-index after adding/editing documents:
```bash
curl -X POST http://127.0.0.1:8000/api/admin/reindex
```

## Placeholder content — read before you demo/submit

`backend/documents/*.md` are **original placeholder nursing content** written for this
prototype so RAG has something real to retrieve from immediately — they are not pulled
from any textbook. See `backend/documents/README.md` for how to swap in real, citable
sources (open-access textbooks, published guidelines, your institution's SOPs) before a
real submission — the retrieval, re-ranking, and Evidence Support Score all work the same
way regardless of what's in that folder.

## Project structure

```
backend/
  main.py                 # FastAPI app / Secure Backend-API Layer
  controller.py            # Autonomous Agent Controller (7-step loop)
  memory.py                 # Shared Agent State & Memory (SQLite)
  config.py
  agents/
    learner_agent.py               # Agent 1
    clinical_reasoning_agent.py    # Agent 2
    quiz_generator_agent.py        # Agent 3
    feedback_agent.py              # Agent 4
    learning_planner_agent.py      # Agent 5
    progress_analytics_agent.py    # Agent 6
  rag/
    ingest.py                # chunk + embed + store
    retrieval.py              # hybrid retrieval + re-ranking + evidence pack
    evidence_validator.py     # computed Evidence Support Score
    llm_client.py              # Specialist LLM Agent call wrapper
    models.py                   # shared embedder/reranker
  documents/                 # placeholder nursing knowledge (swap for real sources)
frontend/
  index.html                # UI — talks only to the backend, never to Anthropic directly
                             # (served by the backend itself at http://127.0.0.1:8000/)
Nightingale_Setup_and_Run.ipynb   # guided, step-by-step notebook — start here
```

## Notes / honest limitations for your write-up

- BM25 keyword index and the embedding/reranker models are cached in-process; a
  multi-worker production deployment would need a shared store instead.
- The controller retries once on a failed structural check or a low Evidence Support
  Score — it does not currently re-plan with a different retrieval query, which would be
  a natural next iteration ("Decide Next Action" could trigger a re-query, not just a
  re-generation).
- Evidence Support Score is a semantic-similarity heuristic (embedding cosine similarity
  between response sentences and retrieved evidence), which is simple enough to explain
  and defend in a methodology section, but it is not a formal factual-entailment checker —
  worth naming explicitly as a scoping decision if asked.



