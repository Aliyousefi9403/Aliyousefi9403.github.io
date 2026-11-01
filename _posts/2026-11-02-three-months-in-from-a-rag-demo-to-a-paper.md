---
title: "Three months in — from a RAG demo to a research paper"
date: 2026-11-02 18:00:00 +0000
categories: [dissertation, milestones]
tags: [rag, adversarial-ml, pwws, textfooler, relative-entropy, runtime-verification]
---

The last post here was in early August, when the RAG pipeline had just started working end to end. I stopped writing because the project took over, so this is a catch-up. Over the next few posts I'll go through each part in more detail.

## The short version

AegisRAG is now a full experiment rather than a demo:

- a cybersecurity RAG system built from MITRE ATT&CK and CAPEC (18 documents, 200 passages, 14 attack types);
- three versions of the knowledge base: clean, poisoned with PWWS, and a control where only the layout changed;
- a reviewed benchmark of 44 questions, each with reference facts and the exact source section it should be answered from;
- every question run three times per condition, with each record written once and checksummed so nothing can be quietly changed later.

On top of that I built and tested several defences, two attacks, a set of adaptive attackers, and a runtime monitor. The results are written up as a paper that I'm preparing as an arXiv preprint with my supervisors, Dr Hafizul Asad and Dr Nikos Komninos.

## What I found, in one paragraph each

**The attack works, but not the way I expected.** PWWS poisoning dropped judged answer correctness from 0.714 to 0.555. Almost all of that drop came from questions whose source section had been edited. The model rarely repeated the poisoned text as fact (wrong answers went from 0 to 2). Mostly it lost the evidence it needed and said "I don't know": refusals went from 26 to 45 out of 132.

**Relative entropy can find the edited passages, if you use it the right way.** Comparing a whole passage's word distribution with normal text was a weak signal. One term of it worked much better: how much less probable a word is than its most likely synonym. With a small language model as the reference, it reached an AUROC of 0.95 on PWWS, and stayed near chance on the layout-only control.

**Filtering makes the knowledge base safer, not more useful.** Removing flagged passages halved how often edited text reached the model, but correctness didn't come back. The passage that answers the question is usually the one that was edited, so removing it removes the answer too.

**Attackers who know the detector struggle.** An attacker that had to keep each passage as probable under the language model didn't succeed on any of the 57 sections it tried.

**A runtime monitor helps, with caveats.** Following Dr Asad's suggestion, I added a monitor that checks answers against trusted MITRE data and remembers passages that misled it. It cut exposure to edited text from 41% to 26%. On its own it didn't restore correctness; with a trusted MITRE fallback, correctness went up to 0.786.

## What's next

- Finish the preprint with my supervisors' feedback
- Submit the Project Definition Document (due 11 November)
- Write up each part here properly, one post at a time

Next post: what word-substitution poisoning actually does to a RAG system.
