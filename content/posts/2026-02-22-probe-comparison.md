---
title: "Why Probe Choice Changes the Result"
date: 2026-02-22
categories: ["research"]
tags: ["probes", "evaluation", "alignment", "interpretability"]
---

## The question
Does the measurement tool change what we think the model can report about itself?

## The setup
We compared two probing methods:
- **8D grid probes** (multi‑emotion, all at once)
- **Single‑emotion probes** (one emotion at a time)

## The result
The 8D grid systematically underestimates self‑report compared to single‑emotion probes.
This is a material effect that changes the apparent size of the introspection gap.

![Probe comparison.](/fig31_probe_comparison.png)

## Why it matters
If different probes yield different levels of self‑report, then evaluation results depend
on measurement design. We should treat probe choice as a first‑class methodological decision.

## What’s next
We’re using probe‑based measurement to bypass output suppression and get closer to the
model’s internal state.

