# Phase 11 — Production Deployment and Operations

## Why this phase exists

This is the final regular phase of the handbook, and it's worth being explicit about what it is and isn't, given your existing six-plus years of Java backend and infrastructure experience. This phase does not re-teach Docker, Kubernetes, or AWS — you already run production services on this infrastructure, and re-explaining container orchestration or load balancing to you would waste your time and dilute a phase that has real, genuinely new material to cover. What this phase covers is precisely the set of concerns that are specific to agentic AI workloads running on top of infrastructure you already know how to operate: what changes about secrets management when the secret is a model API key with a very different blast radius than a database credential (a concern first raised in Phase 02, file 3, and given full production treatment here); what caching means when the thing you're caching is a semantically fuzzy match rather than an exact key lookup; what "rate limiting" means when the thing you're rate-limited by is a third-party model provider rather than your own infrastructure; and what "safely rolling out a change" means when the change isn't to code but to a model's *behavior*, which can't be verified by a type checker or a deterministic test suite the way an ordinary code change can.

This phase builds directly on Phase 10's observability foundation, and that dependency is not incidental — every production concern in this phase assumes you can already see what your agent system is doing (Phase 10, file 1's tracing), already know what it costs (Phase 10, file 5's dashboards), and already have a way to verify a change didn't regress behavior (Phase 10, file 4's regression suite). Deploying and operating an agent system without that foundation in place is deploying blind, regardless of how solid your Kubernetes manifests or your Terraform are — the infrastructure layer this phase covers is only as trustworthy as the observability layer underneath it.

## What this phase covers

1. **`01-packaging-spring-ai-services.md`** — packaging a Spring AI-based agent service for deployment, and the specific things that differ from packaging an ordinary Spring Boot service, given everything else this phase will build on top of it.
2. **`02-secrets-and-config-management.md`** — a full production treatment of the concern Phase 02, file 3 first raised: model API key scoping, rotation, and spend-limit discipline, now at the scale of a real deployed system with multiple environments and services.
3. **`03-semantic-caching-for-cost-control.md`** — a caching strategy with genuinely no analogue in ordinary REST service caching, exploiting Phase 03's embedding-similarity machinery to cache "close enough" requests, not just exact ones.
4. **`04-guardrails-and-safety-filters.md`** — a production-grade defense-in-depth layer against harmful, off-policy, or manipulated agent output and input, extending Phase 05's tool-validation discipline to the full input/output surface of a deployed agent.
5. **`05-rate-limiting-and-backpressure.md`** — managing the specific situation where your own service is rate-limited by an upstream model provider (Phase 02, file 9's resilience patterns, now applied at the scale of your entire service's traffic, not one client's retry logic).
6. **`06-scaling-async-agent-workloads.md`** — scaling agent processing given the specific latency and cost profile of LLM calls (Phase 01's mechanics), including async, queue-based architectures for workloads that don't need synchronous, interactive response times.
7. **`07-prompt-and-agent-versioning.md`** — treating prompts, tool definitions, and agent configuration as versioned artifacts with the same discipline you already apply to API versioning and database migrations.
8. **`08-canary-releases-for-agent-behavior.md`** — safely rolling out a change to agent *behavior* using Phase 10's evaluation harness as the gate, since a canary release here needs a different kind of health check than an ordinary code deployment's.
9. **`09-project-production-hardened-agent-service.md`** — taking the Phase 09 multi-agent research pipeline, already evaluated in Phase 10, and hardening it with every concern in this phase into something genuinely ready to run in production.

## Prerequisites

Phase 10 in full (this phase assumes tracing, safe logging, evaluation, and cost/latency visibility are already in place) and your existing Docker/Kubernetes/AWS operational experience, which this phase builds on rather than re-teaches.

## What you gain from this phase

The ability to run an agent system in production with the same operational rigor you already apply to any other microservice — plus the specific, genuinely new handful of concerns (API key blast radius, semantic caching, behavioral guardrails, provider-side rate limiting, and behavioral canary releases) that don't have a direct analogue in your existing infrastructure experience and would otherwise be gaps you'd discover the hard way, in production, rather than having engineered for deliberately in advance.