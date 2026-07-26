# Phase 11 Project — Production-Hardened Agent Service

## Scenario

The Phase 09 multi-agent research pipeline works, and Phase 10 gave you real, calibrated evidence of exactly how well — pass rates, cost breakdowns, latency percentiles, a working evaluation harness. This project is the culmination of the entire handbook: taking that evaluated system and hardening it with every production concern from this phase, so it becomes something you could genuinely hand to an operations team and trust running unattended, serving real traffic, with a safe path for making future changes to its behavior.

## Functional requirements

1. **Package the service** (file 1) with a health check that reflects actual LLM-provider reachability (via your circuit breaker's state), not just process liveness, and startup-time validation that fails fast on any critically misconfigured value (a zero or negative cost budget, a missing API key, a confirmation-required tool that doesn't exist in the registered tool set).
2. **Implement scoped secrets management** (file 2): separate API keys for at least two distinct environments (a "staging" and a "production" configuration, even if both point at the same underlying provider for this project's purposes), each with its own configured spend limit, and a cost report that correctly attributes spend to the correct environment.
3. **Add a semantic cache** (file 3) in front of the summarizer agent's final-synthesis step specifically — this is the component most likely to receive genuinely similar queries worth caching — with a tuned similarity threshold and freshness window, evaluated against a small manually-judged set of "should cache-hit" and "should not cache-hit" query pairs.
4. **Implement layered guardrails** (file 4): both an input guardrail (screening for injection attempts before the pipeline runs) and an output guardrail (screening the final synthesized answer before delivery), using a smaller, cheaper model for both checks, and demonstrate at least one deliberately crafted injection attempt being correctly flagged and blocked.
5. **Implement shared rate limiting** (file 5) in front of all outbound LLM calls across the entire multi-agent pipeline (supervisor, search agent, and summarizer agent all routing through the same limiter), with a demonstrated fail-fast response when the limiter is deliberately configured tight enough to be exceeded during a test run.
6. **Convert the pipeline to an async, queue-based architecture** (file 6) for at least the search-and-summarize portion of the workflow — task submission returns a task ID immediately, a worker pool processes tasks from a queue, and a status endpoint reports progress or the final result.
7. **Version the pipeline's full configuration** (file 7) — system prompts for all three agents, tool definitions, and budget values — as a single coherent `AgentConfigVersion`-style artifact, with at least one real version-2 change (a prompt wording adjustment, or a budget adjustment) recorded with a change description, distinct from version 1.
8. **Run a full canary evaluation** (file 8) of your version-2 change against version-1 using Phase 10's regression suite, and report the outcome — approved for rollout, or rejected due to a detected regression — based on actual pass-rate comparison, not assumption.

## Constraints

- The health check must genuinely reflect circuit breaker state, not just return a hardcoded "UP" — demonstrate this by deliberately forcing the circuit breaker open (e.g., by pointing at an invalid endpoint temporarily) and showing the health check correctly reports degraded status.
- The async worker pool's autoscaling configuration (even if only conceptually specified, if you don't have a live Kubernetes cluster available for this project) must scale on a queue-depth-style metric, not CPU or memory, with a written justification referencing file 6's argument about why this workload is latency-bound on an external dependency rather than compute-bound.
- The canary evaluation in requirement 8 must use freshly-run regression suite results for both the candidate and baseline versions, per file 8's explicit guidance against comparing a fresh candidate against a stale baseline number.

## What "done" looks like

- A complete demonstration of the health check correctly degrading when the LLM provider circuit breaker opens, and recovering once it closes again.
- A cost report showing distinct, correctly attributed spend across your two configured environments, with each environment's spend limit respected.
- A semantic cache hit demonstrated on a genuinely differently-worded but semantically equivalent query, and a demonstrated cache miss (or correct non-match) on a genuinely different query that a naive, overly-loose threshold would have incorrectly matched.
- A blocked injection attempt, shown end to end: the crafted input, the input guardrail's classification, and confirmation that the pipeline did not proceed to process it normally.
- A rate-limiter-triggered fail-fast response under a deliberately tight test configuration, with a clear, honest error message rather than an unhandled exception or an indefinite hang.
- A working async submission-and-status-polling flow for at least one real multi-hop research task, run through the queue-based architecture rather than synchronously.
- A canary evaluation report comparing your version-1 and version-2 configurations, with an explicit approve-or-reject decision backed by the actual regression-suite pass-rate numbers for each test case.

## Reflection

This project, and this phase, complete the handbook's core arc: from understanding how an LLM generates text (Phase 01) through building, evaluating, and now safely operating a genuinely production-grade multi-agent system. Before moving to the capstones, it's worth writing a short, honest retrospective: of every discipline this phase introduced, which one would you have been most likely to skip if you'd built this system without having gone through this handbook first, and what's the most likely real-world failure that omission would eventually have caused? This is not a rhetorical exercise — it's the same kind of retrospective judgment a principal engineer applies when reviewing any system's operational readiness, and it's worth having a genuine, specific answer before considering yourself done with this phase.