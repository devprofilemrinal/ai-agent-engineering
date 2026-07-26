# Capstones — Overview

## Why these exist

Every project embedded within Phases 00 through 11 tested one phase's capability in relative isolation — a RAG pipeline, a memory-backed assistant, a single research agent, a multi-agent pipeline, a production-hardened service. Real systems don't arrive in these clean, single-capability slices. A genuine production AI agent system combines retrieval, memory, tool use, structured reliability, reasoning, orchestration, evaluation, and production hardening simultaneously, with each capability's design decisions constraining and interacting with the others. These four capstones are deliberately scoped to feel like systems built inside a real company, not extended homework — each one is the kind of project a team would actually be assigned, with real users, a real business justification, and genuine cross-cutting production requirements that can't be satisfied by any single phase's techniques in isolation.

## How to use these

Each capstone brief below describes the system, breaks it into components with explicit references to which handbook phase(s) each component draws on, names the cross-cutting production requirements that apply across the whole system (not just one component), describes at least one failure scenario the system must handle gracefully, and suggests a build order grounded in the dependency logic this handbook has followed throughout — not an arbitrary sequence. None of these briefs include starter code or a fully worked solution architecture; per this handbook's own design philosophy, a capstone scopes the problem for you to solve using everything you've built, rather than solving it for you.

You do not need to build all four. Each is substantial enough to stand alone as a genuine demonstration of end-to-end capability. If you're choosing one, consider which most closely resembles work you'd actually want to do professionally — the engineering judgment involved in scoping, evaluating, and hardening any one of these transfers directly to the others.

## What "done" means for a capstone, generally

Across all four, "done" means the same thing this handbook has meant by that phrase throughout: not a demo that worked once, but a system with Phase 10's evaluation harness actually run against it, Phase 11's production hardening actually applied (not just described), and an honest, evidence-based account of where it still falls short — because per Phase 00's foundational lesson, a probabilistic system that "seemed to work" is meaningfully weaker evidence than one you've actually measured.