---
title: "Week 1 — Choosing a Dissertation Topic: RAG Poisoning"
date: 2026-07-27 22:00:00 +0100
categories: [journey, dissertation]
tags: [rag, llm, cybersecurity, dissertation, langchain, adversarial-ml]
---

## Where I'm at

Four days into my public cybersecurity log. Since the last post:

- **TryHackMe**: 8 rooms completed across the Complete Beginner and Cyber Security 101 paths.
- **Security+ (SY0-701)**: 4 Prof Messer videos in. Slowed the pace so I can focus on the dissertation — exam target now November.
- **Papers**: worked through nine IEEE Xplore papers Dr Nikos Komninos (Senior Lecturer in Cyber Security at City St George's) sent as a reading list on LLMs and adversarial attacks in cybersecurity.
- **RAG and LangChain**: several hours of official LangChain tutorials, hands-on reading with the LangChain docs, and follow-up conversations with GPT to nail down concepts. I now have a working mental model of retrieval-augmented generation — how documents get embedded, indexed in a vector store, retrieved on query, and stitched into an LLM prompt. Ready to start building.

## How I picked the project

The conversation with Nikos shifted my perspective. When he mentioned that the projects he supervises "often lead to publications," it made clear that this doesn't have to be a coursework tick-box — it can be the first real research output on my CV.

The nine papers he sent split roughly into two camps:

- **LLMs as defensive tools** — detecting attacks, analysing alerts, filtering phishing.
- **Attacks on LLMs themselves** — prompt injection, adversarial examples, denial-of-service.

One paper stood out:

**Ikbarieh, Aryal & Gupta (2025), *"RAG-Targeted Adversarial Attack on LLM-Based Threat Detection and Mitigation Framework"*** (IEEE International Conference on Big Data 2025).

The authors show that if an attacker can inject subtle, meaning-preserving word-level changes into a RAG knowledge base — using tools like TextFooler with POS-tag constraints — an LLM-based intrusion detection system starts producing meaningfully worse security recommendations. They test this against ChatGPT-5 Thinking on two real IoT datasets (Edge-IIoTset, CICIoT2023) and quantify the degradation using a purpose-built evaluation rubric.

Crucially, their conclusion leaves **defensive countermeasures as future work.** That's the research gap.

## The dissertation direction

Working title: **Detecting and Mitigating Adversarial Poisoning in RAG-Based Cybersecurity Systems.**

Planned structure:

1. **Reproduce the Ikbarieh attack** on Edge-IIoTset against a small local RAG system. This is the baseline to defend against.
2. **Design and evaluate a poisoning detector.** Candidate approaches include:
   - Embedding-based outlier detection (do new entries cluster with the trusted set?)
   - LLM-as-judge verification against a trusted knowledge source (e.g. MITRE ATT&CK)
   - Statistical anomaly checks (perplexity, token distribution)
3. **Compare against a "no defence" baseline** on detection rate, false positive rate, and computational overhead.
4. **Stretch goal**: propose a hardened RAG architecture — for example voting between multiple retrievers, or cross-referencing with a read-only trusted subset.

Nikos was clear that the innovation should sit in the defence, not in reproducing the attack. That guidance shapes the priorities for the year.

## Next steps this week

- Meeting with Nikos on Wednesday to lock the direction.
- Set up the Python environment (venv, LangChain, Chroma/FAISS, TextFooler, a small OpenAI-compatible LLM).
- Download the Edge-IIoTset dataset.
- Continue Prof Messer + TryHackMe alongside.

Target: a working baseline RAG system by end of next week, and the Ikbarieh attack reproduced by September. That leaves the full academic year to focus on the defensive contribution.
