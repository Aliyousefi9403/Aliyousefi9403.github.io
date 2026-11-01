---
title: "A runtime monitor that remembers what fooled it"
date: 2026-11-14 18:00:00 +0000
categories: [dissertation, results]
tags: [runtime-verification, ltl, mitre-attack, rag, defences]
---

The filters from the last post had two weaknesses. They forget: the same poisoned passage is judged again every time it's retrieved. And they remove the evidence along with the poison. My new consultant, Dr Hafizul Asad, suggested a different angle: runtime verification. Check every answer against rules before the user sees it, feed violations back, and remember what caused them.

## What it checks against

The attacker can edit my knowledge base, so the rules need something the attacker can't edit: the official MITRE STIX files for ATT&CK and CAPEC. Their SHA-256 hashes are written into the code, and the monitor refuses to start if either file has changed.

## The three rules

- **φ1 — no strange words.** The answer must not contain a rare word (Zipf frequency below 3.5) that was copied from the retrieved passages and doesn't appear in MITRE's vocabulary or the question. Poisoned synonyms like "engraft" are exactly that.
- **φ2 — don't give up wrongly.** The answer must not be a refusal when MITRE covers the attack being asked about.
- **φ3 — never trust again.** Written in linear temporal logic: G(quarantined(p) → G ¬inContext(p)). Once a passage is quarantined, it never reaches the model again.

## The loop

1. Retrieve 10 candidates, drop any that are quarantined, keep the top 3.
2. Generate an answer and check φ1 and φ2.
3. If a rule is broken, blame the passages with rare non-MITRE words, add them to an append-only memory, and ask the model again with feedback — at most twice.
4. In *restore* mode, if it still refuses, add the official MITRE entry as a trusted source.

## A mistake and a fix

The first version of φ1 flagged any rare word outside MITRE's vocabulary, including words the model chose itself ("emphasizing"). It wrongly withheld 10 answers. I changed it to count only words copied from the retrieved passages, and re-ran the full experiment. Both runs are kept.

## Results (fixed version)

- Edited text reaching the model: **41% → 26%** on PWWS, **82% → 58%** on TextFooler
- Wrong answers on PWWS: 2 → 0
- Clean data: almost no cost, nothing quarantined
- Correctness on its own: not restored (0.555 → 0.515, not significant)
- With the trusted MITRE fallback: **0.786**, and refusals down from 45 to 7

## The honest details

- The repeated "Engraft Malicious Package" error disappeared with the monitor — but it was already gone the first time the question was asked, so that was the check-and-retry, not the memory.
- The memory did its job (φ3 never failed, and later questions skipped more quarantined passages), but its blame step is rough: on PWWS only 10 of the 18 quarantined passages were actually edited.
- The fallback beats the clean system only because it gets extra trusted text. It shows what a trusted source can add, not what detection can.

## What I learned

Filtering makes a poisoned knowledge base safer; getting the answers back needs another source of truth. And a memory is only as good as the step that decides what to remember.

Next: writing it all up as a paper.
