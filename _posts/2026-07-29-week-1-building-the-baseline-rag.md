---
title: "Week 1 — Building the Baseline RAG"
date: 2026-07-29 12:00:00 +0100
categories: [dissertation, build]
tags: [rag, langchain, qdrant, docker, python, sentence-transformers]
---

## Progress since the last post

The first meeting with my supervisor, Dr Nikos Komninos, is booked for later today (Wednesday, 29 July). Between the last post and now, I've moved the project from "planned" to "partially built".

## What I built

### Project structure

Set up a modular Python project on GitHub with clear separation of concerns:

~~~
dissertation-RAG/
├── backend/
│   └── app/
│       ├── ingestion/       # loading and processing documents
│       ├── retrieval/       # semantic search
│       └── vectorstore/     # Qdrant client
├── data/
│   ├── raw/knowledge_base/  # 10 markdown files, one per attack type
│   └── processed/
├── experiments/results/
├── tests/
├── pyproject.toml           # pinned dependency versions for reproducibility
├── DECISIONS.md             # rolling log of key technical decisions
└── .env.example             # template for secrets, no keys committed
~~~

### Environment

- Python 3.12 in a virtual environment (`.venv`)
- Docker installed and used to run a local Qdrant vector database
- Dependency versions pinned in `pyproject.toml` so every experiment is reproducible

### Knowledge base (starter set)

Ten markdown files, each describing one common cyber attack — SQL injection, XSS, DDoS, DNS spoofing, MITM, malware, phishing, ransomware, botnet, and brute force. This is a placeholder knowledge base for learning the pipeline. It will be swapped for structured IoT attack descriptions aligned with the Edge-IIoTset taxonomy once the baseline is validated.

### Retrieval pipeline (working)

The first end-to-end component is running: `search_knowledge_base` embeds a query using `sentence-transformers/all-MiniLM-L6-v2` (the same model as Ikbarieh 2025), performs semantic search against a Qdrant collection, and returns the top-k matches with similarity scores. Tested with the query *"How can attackers manipulate a database through user input?"* — it correctly retrieves the SQL injection markdown file.

## Key technical choices (recorded in `DECISIONS.md`)

- **Vector store — Qdrant**, over ChromaDB and FAISS. Chosen for its metadata filtering, which will matter when I need to tag knowledge base entries as `clean` vs `poisoned` in the attack experiments.
- **Embedding model — `all-MiniLM-L6-v2`**, the same model as the Ikbarieh paper. Keeps the reproduction directly comparable to their baseline.
- **Framework — LangChain** (`langchain_huggingface`, `langchain_qdrant`) using the current 2025 API rather than the deprecated top-level imports.

## Also this week

- Finished all but one module of the TryHackMe *Complete Beginner* path.
- Continued Prof Messer's Security+ SY0-701 videos alongside the build.

## What I'm bringing to today's meeting

- A working retrieval baseline I can demo end-to-end.
- The project structure and `DECISIONS.md` for context.
- Two open questions for Nikos:
  1. Realistic scope on the defence side — one detector done rigorously, or aim for a layered Trust Score approach with multiple signals?
  2. Whether to align the knowledge base with the exact Edge-IIoTset attack taxonomy, or diverge to a broader cybersecurity KB.

More after the meeting.
