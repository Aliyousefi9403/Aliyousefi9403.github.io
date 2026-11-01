---
title: "Catching the poison with relative entropy"
date: 2026-11-10 18:00:00 +0000
categories: [dissertation, results]
tags: [relative-entropy, kl-divergence, detection, language-models, adaptive-attacks]
---

My supervisor, Dr Nikos Komninos, suggested looking at the problem through relative entropy (KL divergence). The idea: compare the words in a passage with a reference distribution of normal text, and the swapped-in synonyms should stand out.

## First attempt: the whole passage

I computed KL divergence, Jensen–Shannon divergence and perplexity over whole passages. They were weak:

- KL over letter frequencies: AUROC 0.33 (worse than chance — letters keep their English statistics when you swap words)
- KL over words, against the rest of the knowledge base: 0.72, but also 0.62 on the control where no word changed

Only a handful of the roughly eighty words in a passage are edited, so their effect gets diluted.

## What worked: one term of the sum

KL is a sum of log-ratio terms. Instead of summing over everything, I looked at that log-ratio only where an attack would act: for each word, how many bits more probable its most likely synonym is. I call this the **synonym gap**.

"Abnegate" scores high, because its likely original, "deny", is far more probable. A passage's score is the mean of its three largest gaps.

| Detector | PWWS | Control (layout only) |
|---|---|---|
| Synonym gap, word-frequency reference | 0.82 | 0.67 |
| Synonym gap, language-model reference (distilgpt2) | **0.95** | **0.53** |
| Whole-passage perplexity | 0.77 | 0.45 |
| Windowed perplexity (top-5 tokens) | 0.91 | 0.56 |

Two things stood out:

- **The reference mattered more than the formula.** A small language model, which knows whether a word fits *this* sentence, did much better than word frequencies.
- **Looking locally is what counts.** The contextual synonym gap clearly beat whole-passage perplexity and was statistically similar to windowed perplexity. What it adds is an explanation: it names the word it thinks was replaced and the likely original (abnegate → deny).

It isn't better everywhere: on the 9 held-out documents the simpler word-frequency version scored higher (0.94 vs 0.88), and on TextFooler the two were about the same.

## Using it as a filter

Run live as a retrieval filter, the contextual detector halved how often edited text reached the model (41% → 20%) and removed both wrong answers, at almost no cost on clean data. But correctness didn't recover: the passage it removes is usually the one that answers the question.

## Attackers who know the detector

I then built attackers that knew about the detector:

- **Frequency-matched**: only swaps to words at least as common. It beat the word-frequency detector, but the language-model detector still flagged 10 of its 13 edits, and it only succeeded on 8 sections instead of 39.
- **Likelihood-constrained**: must also keep the passage as probable under the language model. It succeeded on **0 of 57** sections. Of the 550 swaps normal PWWS used, only 9 passed that rule.

With a looser rule (allowed to lose up to 100 bits of likelihood) it succeeded on 5 sections, and 3 of the 6 edited passages got past the filter. So this is evidence about one family of attackers, not a guarantee.

Next post: a runtime monitor that checks answers and remembers what fooled it.
