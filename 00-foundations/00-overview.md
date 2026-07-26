# Phase 00 — Foundations

## Why this phase exists

Every debugging instinct you've built over six years of Java backend work rests on one assumption: **given the same input, a function returns the same output.** A `GET /users/42` call, a Kafka consumer processing a message, a cache lookup — all of it is deterministic, or treated as a bug when it isn't (a flaky test, a race condition, a stale cache entry). You've spent years getting good at hunting down the *causes* of non-determinism because it's always been the exception.

Large language models flip that assumption. Non-determinism isn't a bug you hunt down — it's the normal operating mode of the system. Every phase after this one — cost estimation, retrieval quality, tool-calling reliability, agent loops, evaluation — is, underneath, a strategy for managing that one fact. If you carry your deterministic-systems instincts into agent engineering unexamined, you'll misdiagnose a huge share of the problems you hit: chasing a "bug" that's actually expected model variance, or trusting a single test run the way you'd trust a unit test.

This phase has no code. Its only job is to reset that assumption before you touch an API.

## What this phase covers

1. `01-deterministic-vs-probabilistic-systems.md` — what actually changes in your engineering approach when a system is probabilistic
2. `02-why-llms-are-not-traditional-apis.md` — where an LLM API genuinely behaves like a REST API you already know, and where it doesn't
3. `03-project-brief.md` — a short prediction exercise, revisited once Phase 01 is complete

## Prerequisites

None beyond your existing backend background.

## What you gain from this phase

Not a skill exactly — a recalibration. By the end, you should stop asking "why did it give a different answer" as if it's an anomaly, and start asking "how much variance should I expect here, and what do I do about it" as an engineering question with a real answer. That question is what Phases 01 through 11 answer, piece by piece.