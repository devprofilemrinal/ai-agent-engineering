# Phase 06 — Structured Output and Reliability

## Why this phase exists

Every phase up to this point has produced systems that ultimately hand something to a human, or to another part of your code, expecting it to be usable. Phase 02 gave you the protocol-level mechanisms for constraining output (JSON mode, forced tool calls). Phase 05 gave you validation and sandboxing specifically for tool-call arguments. This phase is where those two threads combine into a general engineering discipline: **making a fundamentally probabilistic system produce output your code can parse and act on reliably, and building the failure-handling machinery for the cases where it doesn't.**

This phase sits deliberately between Phase 05 (tool use) and Phase 07 (reasoning and agent loops) for a specific structural reason. An agent loop, at its core, is a sequence of decision points — "call this tool," "the task is done," "try a different approach" — and every one of those decisions has to be extracted from generated text and acted upon programmatically. If your parsing and reliability layer is fragile, an agent loop built on top of it doesn't fail occasionally in an obvious way — it fails by silently misinterpreting the model's intent, taking the wrong action, or getting stuck retrying the same malformed step indefinitely. You cannot debug Phase 07's reasoning loops sensibly if you haven't first made this phase's parsing and reliability layer solid, because a bug that looks like "the agent is reasoning badly" is very often actually "the agent's decision was parsed incorrectly downstream of a subtle malformed-output case this phase would have caught."

The core reframe this phase asks you to internalize: **a schema constraint (Phase 02) reduces the frequency of malformed output — it does not eliminate it, and it says nothing at all about whether well-formed output is factually correct.** Those are two entirely separate failure categories, and this phase treats them as such: shape validation (is this parseable and schema-conformant) and content correctness (is this actually true, given what the model was asked and what it was given), with distinct engineering responses for each.

## What this phase covers

1. **`01-schema-driven-generation.md`** — going beyond Phase 02's basic JSON mode and forced tool calls into a full discipline of designing schemas specifically for reliability: how field design, enum constraints, and schema complexity itself directly affect how often the model produces valid output in the first place, with worked examples of good and bad schema designs and why each behaves the way it does.
2. **`02-output-parsers-and-validators.md`** — the actual Java code that sits between "raw text or JSON the model produced" and "a typed object your business logic can trust," including multi-stage validation (syntactic, schema, semantic), and why each stage needs to exist as a genuinely separate pass rather than being conflated into one large try/catch.
3. **`03-retry-on-malformed-output.md`** — what to do when validation fails: a full decision framework for retry-versus-repair-versus-fail-fast, concrete backoff and re-prompting strategies distinct from Phase 02's HTTP-level retry logic, and the specific risk of retry loops that repeat the same failure indefinitely.
4. **`04-project-invoice-extraction-agent.md`** — a complete Java system that extracts structured financial data from realistic, messy invoice text, deliberately engineered to be evaluated against a stress-test set of edge cases designed to break a naive implementation.

## Prerequisites

Phase 02 (the wire-level mechanisms — JSON mode, forced tool calls — that this phase builds a full engineering discipline on top of) and Phase 05 (the validation and sandboxing patterns this phase generalizes beyond tool-call arguments to *any* structured output the model produces).

## What you gain from this phase

The ability to build a decision point in an agent — any moment where the model's output needs to become a concrete, typed value your code branches on — that fails loudly, recoverably, and informatively when the model's output doesn't hold up, instead of failing silently by producing corrupted downstream state. This is the single largest determinant of whether an agent system feels "flaky" or "solid" in practice: nearly every visible agent failure a user encounters, on closer inspection, traces back to a decision point that assumed well-formed output and didn't check.