# Phase 02 Project — Dependency-Free Java CLI Chat Client

## Scenario

Your team needs a lightweight internal tool for engineers to quickly test prompts against the production model configuration — no Spring Boot app to stand up, no framework dependency to add to a shared build, just a `.jar` someone can run from a terminal. This project builds that tool, and in doing so, forces every concept from this phase to work together in one real, runnable artifact rather than staying as isolated code snippets in separate files.

## Functional requirements

1. **Interactive loop.** Read a line of user input from stdin, send it to the model along with the full conversation history so far, print the response, and repeat — a real multi-turn conversation, not a single request-response demo.
2. **Correct conversation state management.** Every user message and every assistant reply must be appended to an in-memory history and re-sent in full on every subsequent call, exactly per `02-message-roles-and-statelessness.md` — no server-side session, no shortcuts.
3. **Streamed output.** Responses must be displayed incrementally as they arrive via SSE (`04-streaming-with-sse.md`), not buffered and printed all at once — the user should see text appear progressively, the way a real chat interface would.
4. **Secure credential loading.** The API key must be read from an environment variable, never hardcoded, never logged, per `03-authentication-and-secrets.md`.
5. **At least one working tool call.** Implement one real tool the model can invoke — for example, a `get_current_time` or `read_local_file` tool — following the full round trip from `05-function-calling-wire-protocol.md`: tool definition sent, `tool_use` response parsed, tool executed locally, `tool_result` sent back, final answer displayed.
6. **Resilient error handling.** Distinguish retryable failures (429, 529, transient network errors) from non-retryable ones (context length exceeded, bad request), per `09-resilience-retries-backoff-circuit-breakers.md` — a retryable failure should retry automatically with backoff and inform the user it's retrying; a non-retryable one should fail with a clear, specific error message, not a generic stack trace.
7. **Session-end token/cost summary.** When the user exits (e.g., types `/exit`), print the total input tokens, output tokens, and estimated cost for the entire session, using the real `usage` data returned by the API (per `01-rest-anatomy-of-llm-apis.md`) rather than the Phase 01 heuristic estimates — this is the moment where Phase 01's estimation project and this phase's real client meet.

## Constraints

- **No frameworks.** Plain Java, `java.net.http.HttpClient`, and Jackson for JSON only (Jackson is acceptable as a JSON library, not as a framework — it's the same dependency you'd use in any plain Java project). No Spring Boot, no LangChain4j, no Spring AI — those come in Phase 08, deliberately after this.
- **Must run as a single executable via `java -jar` or `java ClassName`**, reading configuration (API key, base URL, model name) from environment variables only.
- **Must handle at least one full streaming response and one full tool-calling round trip in the same running session** — not as two separate demo modes, but as natural parts of one continuous conversation, since a real usage session could involve either at any point.

## What "done" looks like

- You can have a genuine multi-turn conversation where a later message correctly references something said several turns earlier, proving conversation state is being managed correctly rather than silently dropped.
- Killing your network connection mid-session and restoring it shows an automatic retry with visible backoff, not an immediate crash.
- Asking a question that triggers your implemented tool produces a visibly two-step exchange (tool call, then a final answer incorporating the tool's real result) rather than the model guessing at an answer it has no way to actually know.
- Exiting the session prints an accurate running token/cost total that matches what you'd expect from manually summing the `usage` field across every call made during the session.

## Extension

Once you've built this, deliberately try to break your own resilience handling: simulate a `429` by making rapid-fire requests until you actually hit a real rate limit, and confirm your backoff logic behaves as designed rather than just in a mocked test. This is also a natural point to start informally comparing this hand-built client's behavior against Spring AI's `ChatClient` once you reach Phase 08 — noting, concretely, everything the framework does for you that you had to write by hand here.