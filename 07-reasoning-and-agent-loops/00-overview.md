# Phase 07 — Reasoning and Agent Loops

## Why this phase exists

Every phase up to this point has built a distinct capability: sending a well-formed request (Phase 02), grounding it in retrieved facts (Phase 03), managing what it remembers (Phase 04), letting it call real code (Phase 05), and making its output reliably parseable (Phase 06). This phase is where those five capabilities stop being independent pieces and become one thing: an **agent loop** — a running process that repeatedly asks the model what to do next, does it, observes what happened, and decides whether to continue, all without a human driving each individual step.

This is the conceptual center of gravity for the entire handbook, and it's worth being explicit about why it comes exactly here, and not earlier. A "loop" implies iteration, and iteration implies every earlier phase's concerns compound rather than apply once: Phase 01's cost mechanics, which were about a single call or a single conversation, now apply per iteration, potentially many times over one task — a runaway loop is not a hypothetical, it's a real, common failure mode this phase has to engineer against directly. Phase 04's memory-growth problem, previously about a human conversation lengthening over many turns, now happens *faster*, since every tool call and its result (Phase 05) adds to the context that has to be carried into the next iteration. Phase 06's reliability discipline, previously applied to one extraction call, now has to hold at every single decision point in a loop that might run for ten or twenty steps — one unvalidated step is one opportunity for the entire loop to go somewhere wrong and, without a stopping check, keep going.

This phase is deliberately, uncompromisingly framework-free. You will build a complete, working agent loop using nothing but plain Java, the client from Phase 02, the tools from Phase 05, and the validation from Phase 06. This is not an academic exercise — it's the single highest-leverage thing you can do before Phase 08 introduces LangChain4j's `AiServices` and Spring AI's tool-calling `ChatClient`, both of which implement an agent loop internally and hand you the *result* of running one. If you've built this loop by hand first, a framework's agent abstraction is legible: you know exactly what it's doing on every iteration, you can predict its cost and failure modes, and when it misbehaves in production, you know precisely which part of the loop to look at. If you skip straight to the framework, you're debugging a black box the first time something goes wrong — and with a probabilistic system (Phase 00), something eventually will.

## What this phase covers

1. **`01-react-pattern-from-scratch.md`** — the foundational reasoning pattern: Reason, Act, Observe, repeat. What it actually is mechanically, why it works, and a first, minimal hand-built implementation.
2. **`02-plan-and-execute-pattern.md`** — a different reasoning structure, where the model commits to a multi-step plan upfront rather than deciding one step at a time, and the specific situations where this outperforms ReAct.
3. **`03-reflection-and-self-critique-loops.md`** — adding a step where the agent evaluates its own intermediate output before proceeding, and the real cost/benefit trade-off of doing so.
4. **`04-building-an-agent-loop-in-plain-java.md`** — the complete, production-shaped implementation: the actual `while` loop, termination conditions, cost and iteration budgets, and every earlier phase's safeguards wired in together.
5. **`05-project-hand-rolled-research-agent.md`** — a full research agent, built with zero framework dependency, that plans, searches, reasons over results, and produces a synthesized answer — the most substantial project in the handbook so far.

## Prerequisites

Phase 02 (the client and resilience layer the loop calls on every iteration), Phase 04 (memory strategies, since a loop's growing context is a memory-management problem), Phase 05 (tool use, since acting is calling tools), and Phase 06 (structured output and reliability, since every reasoning step's output needs to be reliably parsed before the loop can act on it).

## What you gain from this phase

A complete, working, hand-built agent — the first genuinely autonomous multi-step system in this handbook, capable of deciding its own sequence of actions rather than following a single fixed request/response exchange. More importantly, you gain the ability to *debug* one: to look at a misbehaving agent and know whether the problem is in its reasoning, its tool execution, its memory management, or its termination logic, because you've built and reasoned through each of those parts by hand, one at a time, before ever combining them.