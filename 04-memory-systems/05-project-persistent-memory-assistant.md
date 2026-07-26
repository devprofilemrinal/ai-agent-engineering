# Phase 04 Project — Persistent Memory Assistant

## Scenario

A personal engineering assistant is only genuinely useful if it doesn't make the user repeat themselves every time they open a new session — "I already told you I use Java 21 and Maven, not Gradle" is exactly the kind of friction that makes an assistant feel disposable rather than something worth returning to. This project builds an assistant that demonstrably remembers specific facts about its user across entirely separate process runs, proving out this phase's central short-term/long-term distinction in a working system rather than leaving it as something you've only read about.

## Functional requirements

1. **Short-term memory within a session**, using either the buffer or summarization strategy from file 2 — your choice, but you must be able to justify which one you picked and why, given the kind of conversation this assistant is meant to support.
2. **A fact-extraction step** (file 3) that runs after each turn and decides whether anything durable was just stated — a technology preference, a standing project constraint, a recurring habit — and, if so, stores it.
3. **Persistent long-term storage that survives a full process restart.** In-memory storage that resets when the JVM stops does not satisfy this requirement — persist the vector store to disk (a simple serialized file is acceptable) or to a real embedded/lightweight database, so memories genuinely outlive the running process.
4. **Correct recall in a brand-new session.** Start the application fresh (a new process, no shared in-memory state with any prior run), ask a question that depends on a fact stored in an earlier run, and confirm the assistant answers correctly using retrieved long-term memory — this is the single most important proof point for the entire phase.
5. **Multi-user correctness**, even if you only test with two synthetic users: confirm that User A's stored facts are never retrieved into User B's conversation, per file 3's metadata-filtering discipline.
6. **A visible distinction in behavior** between something the assistant "remembers" from long-term storage versus something it's only aware of because it's still within the current session's short-term buffer — for example, having the assistant note when it's recalling something from a past session versus the current conversation.

## Constraints

- Long-term memory must use real vector embedding and similarity search (Phase 03's mechanism), not a simple key-value lookup or exact string match — the point is to exercise semantic recall, where a differently-phrased question still correctly retrieves the relevant stored fact.
- The extraction step must be selective, not indiscriminate — logging and embedding every single turn as a "memory" is an explicit anti-pattern this phase warned against; you should be able to show conversation turns that were correctly judged as *not* containing anything worth remembering long-term.
- You may build this using either the hand-rolled approach from files 1 through 3, or a framework from file 4 — but if you use a framework, you must still implement the extraction-step logic yourself, since neither framework provides it out of the box.

## What "done" looks like

- A demo where you: run the assistant, state a few durable facts across a conversation, exit the process entirely, restart it as a genuinely new run, and ask a question that only makes sense if a fact from the earlier run was recalled correctly.
- A log or trace showing at least one conversation turn where the extraction step correctly declined to store anything, alongside at least one where it correctly identified and stored a durable fact — demonstrating the extraction step is discriminating, not indiscriminate.
- A demonstration that User A's facts do not leak into User B's session, using two distinct synthetic user identities.

## Extension

Add a simple forgetting mechanism — an explicit way for the user to say "forget that I mentioned X," which finds and removes the corresponding stored memory rather than leaving stale or outdated facts to accumulate and potentially contradict more recent statements. This also previews a real production concern from Phase 11: long-term memory stores need active management over time, not just a one-way accumulation of everything ever extracted.