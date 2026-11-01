---
title: "What word-substitution poisoning actually does to a RAG system"
date: 2026-11-06 18:00:00 +0000
categories: [dissertation, results]
tags: [rag, adversarial-ml, pwws, poisoning, evaluation]
---

Most RAG poisoning research adds new, malicious passages to the knowledge base. I wanted to look at something quieter: editing passages that are already there, by swapping words for synonyms.

## The attack

PWWS (Ren et al., 2019) takes a classifier and a piece of text, and swaps words for WordNet synonyms until the classifier changes its answer. The result still reads almost normally. For example, "implants malicious software" became "Engraft Malicious Package".

I trained a DistilBERT classifier that predicts the attack type of a section (DDoS, injection, malware and so on) and used it as a *surrogate*: a section was rewritten whenever PWWS managed to flip its label. Out of 57 sections the classifier got right, PWWS succeeded on 38.

The poisoning script also flattened the layout of the sections it touched. To make sure the layout change wasn't what caused the damage, I built a control knowledge base with the same layout change but the original words. Retrieval on the control was the same as on the clean corpus.

## How I measured it

- 44 reviewed questions, each with reference facts and an exact source section
- each question run 3 times on the clean and the poisoned knowledge base, paired
- an LLM judge scoring correctness against the reference facts, checked against my own blind grades on 75 answers (weighted κ = 0.82)
- question-level bootstrap confidence intervals and Wilcoxon tests

## Results

| Questions | Correctness clean → poisoned | Change [95% CI] |
|---|---|---|
| All 44 | 0.714 → 0.555 | −0.159 [−0.254, −0.068] |
| Source section edited (27) | 0.682 → 0.432 | −0.250 [−0.383, −0.124] |
| Source section not edited (17) | 0.765 → 0.750 | −0.015 [−0.083, 0.069] |

The drop came almost entirely from questions whose source section had been edited.

## The surprise: it's mostly missing evidence, not misinformation

I expected the model to repeat the poisoned words as facts. That happened, but rarely: wrong answers went from 0 to 2, and both were the same question, where the model treated "Engraft Malicious Package" as the name of an attack stage.

What happened much more often was that the model lost its evidence:

- refusals ("the context does not provide…") went from 26 to 45 out of 132
- of the 48 answers that got clearly worse, 27 were refusals and 31 had no relevant passage in the top 3

The edited passage looks less like the question, so it drops out of the top results, and the model, told to answer only from the context, declines.

## Why it matters

A defence that only looks for false statements in the answer would miss most of this damage. That shaped the rest of the project: I focused on finding the edited passages *before* they reach the model.

Next post: catching the edited passages with relative entropy.
