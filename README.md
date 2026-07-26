# AI Agent Engineering Handbook

A hands-on engineering handbook for building production-grade AI Agents in Java and Spring Boot — written for backend engineers who already know distributed systems, REST APIs, and production operations, and are new only to AI.

This is not a prompt-engineering course, a framework tutorial, or a collection of notes for revision. It's a capability-ordered path from "how does an LLM actually generate text" to "how do I run a multi-agent system in production and know when it's misbehaving."

## Who this is for

You should already be comfortable with:
- Java, Spring Boot, REST APIs
- Distributed systems and microservices
- Docker, Kubernetes, AWS
- Databases, caching, messaging systems

You do not need any prior AI/ML background. Prompt engineering is assumed to be covered elsewhere and is not repeated here.

## How to read this repository

Read phases in numeric order. Within a phase, read files in numeric order. Each phase folder has a `00-overview.md` explaining why the phase exists and what it assumes you already know — read that first every time, even if you're tempted to jump to the interesting file.

Frameworks (LangChain4j, Spring AI) are introduced deliberately late — Phase 08 — after you've hand-built the mechanics they abstract (retrieval, tool-calling, agent loops) without any framework at all. If you're tempted to skip ahead to "the framework part," resist it — the frameworks will make a lot more sense once you know what they're hiding.

## Roadmap at a glance

| # | Phase | Level |
|---|---|---|
| 00 | Foundations | Foundation |
| 01 | LLM Internals and Inference | Foundation |
| 02 | Model Communication Protocols | Foundation |
| 03 | Context Augmentation and Retrieval | Intermediate |
| 04 | Memory Systems | Intermediate |
| 05 | Tool Use and Function Calling | Intermediate |
| 06 | Structured Output and Reliability | Intermediate |
| 07 | Reasoning and Agent Loops | Advanced |
| 08 | Agent Frameworks in Java | Advanced |
| 09 | Orchestration and Multi-Agent Systems | Advanced |
| 10 | Evaluation and Observability | Advanced / Production |
| 11 | Production Deployment and Operations | Production |
| — | Capstones | Production |

## Repository layout

```
ai-agent-engineering-handbook/
├── 00-foundations ... 11-production-deployment-and-operations/   → the 12 phases
├── capstones/                                                    → enterprise-shaped end-to-end projects
├── resources/                                                    → glossary, links, decision log
└── _meta/generation-prompts/                                     → the prompts used to generate this handbook's own content
```

## Status

This repository is under active construction. Content is generated phase by phase, in order, using the reusable prompts in `_meta/generation-prompts/`.
