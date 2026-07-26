# Provider Differences: Anthropic vs. OpenAI (and Why This Matters for Your Client Design)

## Why this exists

Every file so far in this phase has described "the" LLM API protocol as if it were one universal standard. It isn't — it's a *family* of very similar but not identical protocols, and the differences are exactly the kind that silently break code if you don't design for them deliberately. This file exists so that when you write your Java client (next file) and, later, adopt a framework (Phase 08), you understand precisely which parts of your mental model are provider-specific rather than universal — the same discipline you'd apply if you were writing code that needed to work against both PostgreSQL and MySQL and had learned, the hard way, exactly where "SQL" stops being portable.

## Where the two major providers genuinely diverge

**System instructions.** Anthropic's API takes `system` as a separate top-level field, distinct from the `messages` array (as shown throughout this phase). OpenAI's API instead represents the system instruction as a message with `role: "system"` inside the same `messages` array as everything else. Functionally similar in intent, structurally different in JSON shape — code that hardcodes "system prompt is always its own field" will need adjusting for the other provider.

```json
// Anthropic-style
{ "system": "You are a support agent.", "messages": [ {"role": "user", "content": "..."} ] }

// OpenAI-style
{ "messages": [ {"role": "system", "content": "You are a support agent."}, {"role": "user", "content": "..."} ] }
```

**Tool/function-calling field names.** The concept from file 5 is universal — every major provider supports the model requesting a function call — but field names differ: `tool_use` / `input_schema` in one convention versus `function_call` / `parameters` in another. The *shape* of the idea (schema-defined tool, model requests it, you execute and return a result) is portable; the exact JSON keys are not.

**Response envelope structure.** Whether content arrives as an array of typed blocks (as emphasized in file 1) or as a single message string with function calls represented in a sibling field varies by provider — this is precisely why file 1 insisted on modeling `content` as an array from the start: it makes adapting to a provider that uses a different envelope shape a matter of changing a mapping layer, not rewriting your entire request/response model.

**Streaming event types and framing.** The SSE mechanism itself (file 4) — newline-delimited `data:` events over a long-lived HTTP response — is common across providers, but the specific event `type` values, and how a text delta versus a tool-call delta versus a stream-end marker are represented, differ enough that a streaming parser hardcoded to one provider's event vocabulary will not work unmodified against another.

**Pricing granularity and cache-related token categories.** Some providers introduce additional token categories beyond simple input/output — for example, distinguishing tokens that hit a prompt cache versus tokens processed fresh, priced differently again. Phase 01's two-part (input/output) cost model is the right *starting* mental model; providers can add finer-grained categories on top of it, and your cost-estimation code (Phase 01's project, revisited in Phase 11) should be built to accommodate additional categories rather than assuming exactly two will always be the full picture.

## The engineering response: an abstraction seam, not a rewrite

None of this means you need to support every provider from day one, or that the differences are so deep as to make portability pointless. It means your Java client (next file) should have one clear seam — an interface, not a scattering of `if (provider.equals("anthropic"))` checks throughout your business logic — between "provider-specific request/response shape" and "the rest of your application's logic":

```java
public interface LlmClient {
    ChatResponse send(ChatRequest request);
    Stream<StreamEvent> sendStreaming(ChatRequest request);
}

public class AnthropicLlmClient implements LlmClient {
    // maps ChatRequest -> Anthropic's specific JSON shape,
    // and Anthropic's response JSON -> the shared ChatResponse record
}

public class OpenAiLlmClient implements LlmClient {
    // same responsibility, different mapping
}
```

This is exactly the same discipline as a repository interface sitting in front of a specific JDBC driver or a specific NoSQL client — your business logic (agent loops in Phase 07, orchestration in Phase 09) depends on the interface, `ChatRequest`/`ChatResponse`, never on a provider's raw JSON shape directly. Later, when Phase 08 introduces LangChain4j and Spring AI, you'll recognize this exact seam — that's precisely the abstraction those frameworks provide for you, and understanding why you'd hand-build it yourself here is what makes the framework's version legible instead of magical.

## Trade-offs and when this matters most

- If you know with certainty you'll only ever integrate one provider, a thin abstraction layer might feel like unnecessary ceremony — but even single-provider projects benefit from the seam, because provider API versions themselves evolve, and isolating the mapping logic means an API version bump touches one class instead of scattered call sites.
- If multi-provider support (or even the *option* to switch later, for cost or capability reasons) is a real possibility, build the seam from the start — retrofitting it after business logic has directly coupled itself to one provider's JSON shape is a much larger refactor than designing for it up front.
- Don't over-abstract to the point of hiding genuinely provider-specific *capabilities* (a feature one provider has and another doesn't) behind a lowest-common-denominator interface that quietly drops functionality you actually need — the seam should normalize *shape*, not erase real capability differences you depend on.

## Why this matters next

You now have the complete conceptual and structural picture: request/response anatomy, statelessness, auth, streaming, tool calling, structured output, and where providers diverge. The next file is where all of this becomes real, working Java code — a dependency-free client built directly on `java.net.http.HttpClient`, implementing exactly the concepts covered across this entire phase so far.