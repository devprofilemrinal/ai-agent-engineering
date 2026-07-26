# REST Anatomy of LLM APIs

## Why this exists

Before you can implement anything in this phase — auth, streaming, tool calling, structured output — you need to know the actual shape of the request and response body you're sending and receiving. Every other file in this phase assumes you know this shape cold. This file is intentionally the most detailed, JSON-heavy file in the phase, because everything else builds directly on top of it.

## The endpoint itself is unremarkable

Nearly every major LLM provider exposes a single primary endpoint for generating a response, conventionally something like `POST /v1/messages` or `POST /v1/chat/completions`. That's it — one endpoint, one HTTP verb, for the overwhelming majority of what you'll do. This is a much smaller surface area than a typical CRUD API you've built, and it should feel almost anticlimactic: there's no `/v1/conversations/{id}` resource, no `PATCH`, no pagination of "past responses." One endpoint, doing one thing — generate the next message given everything you send it.

```
POST https://api.example-provider.com/v1/messages
Content-Type: application/json
Authorization: Bearer <api-key>

{ ... request body ... }
```

## Anatomy of the request body

A realistic request body looks like this (field names vary slightly by provider — Phase 02.07 covers those differences — but the shape below is representative):

```json
{
  "model": "some-model-name",
  "max_tokens": 1024,
  "temperature": 0.3,
  "system": "You are a backend engineering assistant. Be precise and concise.",
  "messages": [
    { "role": "user", "content": "What does idempotency mean for a REST API?" },
    { "role": "assistant", "content": "It means repeating the same request produces the same effect as making it once." },
    { "role": "user", "content": "Give a concrete Java example." }
  ]
}
```

Field by field, and why each one exists:

- **`model`** — which specific model version to run against. Unlike a typical API where the endpoint or path implies the "version" of behavior you get, here the model identifier itself is the version selector, and different models can have wildly different cost, latency, and capability — treat this the way you'd treat pinning a specific library version in a `pom.xml`, not as a cosmetic string.
- **`max_tokens`** — the output-length cap discussed in Phase 01's autoregressive generation file. This is a request parameter, not a server-side default you can ignore; forgetting to set it sensibly is a direct cost and latency risk.
- **`temperature`** — the sampling parameter from Phase 01, sent as a plain request field like any other configuration value.
- **`system`** — a special instruction channel, separate from the conversational `messages` array, used to set persistent behavior/persona/constraints for the whole call. Conceptually similar to setting up a base configuration or interceptor that applies to every request in a session, rather than being "just another message" the user could have typed.
- **`messages`** — the actual conversation, as an ordered array of role-tagged entries. This is the single most important field in the entire request, and the next file (`02-message-roles-and-statelessness.md`) is dedicated entirely to it.

## Anatomy of the response body

A non-streaming response (streaming is covered in file 4) looks roughly like:

```json
{
  "id": "msg_01AbC...",
  "model": "some-model-name",
  "role": "assistant",
  "content": [
    { "type": "text", "text": "Idempotency means..." }
  ],
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 187,
    "output_tokens": 94
  }
}
```

Field by field:

- **`content`** — an *array* of content blocks, not a single string. This surprises engineers coming from simpler APIs: a response can contain multiple blocks of different types in sequence — for example, a text block followed by a tool-call block (file 5 covers this exact shape). Always code against "an array of typed blocks," never against "a single string field," even for today's simple text-only case — the array shape is there specifically because richer response types (tool calls, structured content) share the same envelope.
- **`stop_reason`** — why generation ended: it reached a natural stopping point (`end_turn`), it hit the `max_tokens` cap (commonly `max_tokens` as the value here too), or it stopped specifically to let you execute a tool call (`tool_use`, covered in file 5). This field is not optional to check — code that only reads `content` and ignores `stop_reason` cannot distinguish "the model finished its thought" from "the model got cut off mid-sentence by your own `max_tokens` setting," which is exactly the kind of silent truncation bug Phase 01 warned about.
- **`usage`** — the actual input/output token counts for this specific call, which is what you use in production (Phase 11) instead of the character-based heuristics from Phase 01 — those heuristics were for planning ahead of time; `usage` is the ground truth after the fact.

## Mapping this to a Java representation

Even before writing any HTTP code (file 8), it's worth seeing how naturally this maps onto plain Java records — this is the exact shape you'll build in file 8's client:

```java
public record ChatRequest(
    String model,
    int maxTokens,
    double temperature,
    String system,
    List<Message> messages
) {}

public record Message(String role, String content) {}

public record ChatResponse(
    String id,
    String model,
    String role,
    List<ContentBlock> content,
    String stopReason,
    Usage usage
) {}

public record ContentBlock(String type, String text) {}

public record Usage(int inputTokens, int outputTokens) {}
```

Jackson (or any JSON library you already use in Spring projects) maps directly onto these records with standard `@JsonProperty` annotations for the snake_case-to-camelCase mapping — nothing about JSON (de)serialization here is different from any other REST integration you've built.

## Trade-offs and when this matters most

- Treating `content` as a single string instead of an array works today, for simple text-only prompts, and will silently break the moment your agent starts using tools (Phase 05) or structured output (Phase 06) — model the array from day one even if you're not using its full richness yet.
- Ignoring `stop_reason` is fine for quick experiments and actively dangerous in production — Phase 06's reliability patterns depend on knowing whether output was truncated versus naturally complete.
- Hardcoding a request body as a raw JSON string (instead of typed records/DTOs) is tempting for a five-minute test script and a real liability the moment you need to add a field, handle a provider difference (file 7), or unit test request construction — use typed objects from the start, exactly as you would for any other REST client.

## Why this matters next

You now know the shape of a single request/response pair. The next file zooms into the single most important field in that shape — the `messages` array — and explains precisely why it has to contain the *entire* conversation on every call, tying directly back to the statelessness and cost-compounding behavior introduced in Phase 00 and Phase 01.