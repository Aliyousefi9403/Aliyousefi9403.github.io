---
title: "Week 2 — Milestone: full RAG pipeline running end-to-end"
date: 2026-08-04 00:00:00 +0100
categories: [dissertation, milestones]
tags: [rag, langchain, qdrant, adversarial-ml, pwws, mitre-attack]
---

## Milestone reached

The core RAG pipeline is now running end-to-end: user question → semantic retrieval from Qdrant → generation with `gpt-4o-mini` → grounded response with cited sources. From "planned" to "working" in about a week.

## What went in this week

### Source-first knowledge-base builder

Rather than hand-writing attack descriptions or asking an LLM to invent them, I built an extraction pipeline that pulls structured attack content directly from authoritative STIX 2.1 sources:

- **MITRE ATT&CK Enterprise** — for adversary techniques
- **CAPEC** (Common Attack Pattern Enumeration and Classification) — for attack mechanisms
- A YAML source-mapping file defines which authoritative source each attack maps to

Every generated knowledge-base document is fully traceable to a specific external identifier. No LLM-generated content in the corpus. The generator marks each document with provenance metadata (`generated_from_authoritative_sources`, `human_review_required`, `LLM-generated_technical_content: no`).

The current corpus covers 14 attack types across DDoS, reconnaissance, spoofing, injection, and malware categories, plus IoT device specifications.

### Full RAG stack

- **Backend**: FastAPI + LangChain + Qdrant (vector store) + `sentence-transformers/all-MiniLM-L6-v2` embeddings + `gpt-4o-mini` for generation.
- **Frontend**: Next.js with a purpose-built interface (AegisRAG) for demonstrating retrieval + evidence traces.
- **Confidence and trust metrics** wired end-to-end from retrieval scores through to the UI.
- **Response format**: grounded text with numbered `[Source N]` citations and a live evidence chain sidebar.

### Adversarial attack proof of concept

Word-level adversarial attacks are functional in the project's environment. I initially targeted TextFooler (matching the paper I'm building on) but hit persistent Apple-Silicon compatibility issues around TensorFlow / Universal Sentence Encoder. As a pragmatic engineering call, the POC uses **PWWS (Ren et al. 2019, ACL)** — another peer-reviewed word-level attack with meaning-preservation constraints, that doesn't require TensorFlow.

Verification: 100% attack success rate on a pre-trained classifier, demonstrating that the attack tooling works end-to-end in my environment. This isn't the final research attack — it's proof the tools function — but it means the adversarial pipeline can move forward.

## What I've been reading

Working through a set of IEEE Xplore papers on LLM security shared by my supervisor. The paper anchoring the dissertation is Ikbarieh, Aryal & Gupta (2025), *"RAG-Targeted Adversarial Attack on LLM-Based Threat Detection and Mitigation Framework"* (IEEE Big Data 2025). Their key contribution: showing that meaning-preserving word-level perturbations to a RAG knowledge base can measurably degrade LLM output quality for cybersecurity recommendations. Their conclusion leaves defensive countermeasures as future work — that's the gap my dissertation addresses.

## Next up

- Fine-tune a BERT surrogate on the KB attack descriptions
- Run PWWS against the fine-tuned BERT to generate perturbed KB entries
- Ingest perturbed entries into Qdrant with `origin: "poisoned"` metadata
- Measure the impact of retrieval-time poisoning on LLM output quality

That's the reproduction of the paper's attack in my own setup. It's the first real research milestone.

## Meta

TryHackMe: paused for the week to focus on the RAG build. Will pick back up alongside the BERT fine-tuning phase. Prof Messer / Security+ study continues in the background.
