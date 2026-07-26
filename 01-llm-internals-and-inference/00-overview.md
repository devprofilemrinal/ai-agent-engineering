# Phase 01 — LLM Internals and Inference

## Why this phase exists

Phase 00 told you *that* LLMs are probabilistic and *that* they're API-shaped but not API-identical to what you know. It didn't tell you why. Without the "why," every rule you memorize later — "keep prompts short," "watch your token count," "lower the temperature for factual tasks" — is just folklore you follow without being able to reason about edge cases, extend it to new situations, or know when it stops applying.

This phase gives you the mechanism. Not the full mathematics of transformer architecture — you don't need to derive backpropagation to engineer good agents, any more than you needed to understand TCP's congestion control algorithm in detail to build reliable REST services. You need the *conceptual* internals: what a token is, what an embedding represents, what "attention" and "context window" actually mean, how text gets generated one piece at a time, why that process hallucinates, and why it costs what it costs. Everything from Phase 02 onward assumes this.

## What this phase covers

1. `01-tokens-and-tokenization.md` — the actual unit of everything: cost, limits, and truncation
2. `02-embeddings-and-vector-space.md` — how "meaning" becomes something a computer can compare
3. `03-attention-and-context-window.md` — what the model can "see" at any point, and why that's limited
4. `04-autoregressive-generation-and-sampling.md` — how text actually gets produced, one token at a time
5. `05-why-hallucination-happens.md` — why confident wrong answers are a structural property, not a bug
6. `06-cost-and-latency-mechanics.md` — turning the above into real dollar and millisecond numbers
7. `07-project-token-cost-estimator.md` — a small Java CLI that proves you understand the mechanism, without calling any API

## Prerequisites

Phase 00 — specifically, you should already accept that LLM output is probabilistic and that the transport layer around it is ordinary REST engineering. This phase explains *why* the output is probabilistic.

## What you gain from this phase

The ability to look at a prompt and conversation history and predict, roughly, its token count, its likely cost, where it's at risk of truncation, and where the model is likely to guess rather than know. That predictive ability is what Phase 02 needs in order to build a client that handles limits and errors correctly, and what Phase 03 needs to explain why retrieval — feeding the model facts instead of relying on its memorized knowledge — is often the right fix rather than a bigger context window.