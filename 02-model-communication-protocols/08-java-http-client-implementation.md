# Java HTTP Client Implementation

## Why this exists

Every file before this one has been building toward a single deliverable: a real, working, dependency-free Java client. This file writes it. Deliberately, this uses only `java.net.http.HttpClient` — no Spring `RestTemplate`/`WebClient`, no third-party SDK — because the goal here is to see, in your own code, exactly what's happening at the HTTP layer before any framework does it for you. Once you've written this by hand, Spring AI's `ChatClient` (Phase 08) will look like a convenience wrapper around something you already fully understand, not an opaque black box.

## Why `java.net.http.HttpClient`, specifically

It's been part of the JDK since Java 11, requires zero external dependencies, and supports everything this phase needs natively: synchronous and asynchronous requests, HTTP/2, and — critically for file 4 — a `BodyHandler` that exposes the response as a stream of lines rather than forcing you to wait for a fully buffered body. If you've only used `RestTemplate` or `WebClient` in Spring projects, this is a good opportunity to see the lower-level client Spring's abstractions are themselves often built on top of.

## The request/response model, as real Java records

Building directly on file 1's anatomy:

```java
package com.handbook.llmclient;

import java.util.List;

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

## The client itself

```java
package com.handbook.llmclient;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.DeserializationFeature;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;

public class AnthropicLlmClient {

    private final HttpClient httpClient;
    private final ObjectMapper mapper;
    private final String apiKey;
    private final String baseUrl;

    public AnthropicLlmClient(String apiKey, String baseUrl) {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
        this.httpClient = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .version(HttpClient.Version.HTTP_2)
            .build();
        this.mapper = new ObjectMapper()
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }

    public ChatResponse send(ChatRequest request) throws Exception {
        String requestBody = mapper.writeValueAsString(request);

        HttpRequest httpRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/v1/messages"))
            .header("x-api-key", apiKey)
            .header("anthropic-version", "2023-06-01")
            .header("Content-Type", "application/json")
            .timeout(Duration.ofSeconds(60))
            .POST(HttpRequest.BodyPublishers.ofString(requestBody))
            .build();

        HttpResponse<String> httpResponse = httpClient.send(
            httpRequest, HttpResponse.BodyHandlers.ofString()
        );

        if (httpResponse.statusCode() != 200) {
            throw new LlmApiException(
                httpResponse.statusCode(), httpResponse.body()
            );
        }

        return mapper.readValue(httpResponse.body(), ChatResponse.class);
    }
}
```

A few implementation details worth calling out, since they're easy to get subtly wrong:

- **The connect timeout and the overall request timeout are configured separately**, and deliberately generous on the request timeout — recall from Phase 01 that output length (and therefore total latency) is the dominant cost driver; a short, REST-typical timeout tuned for a fast CRUD call will spuriously kill a legitimate, longer generation.
- **`FAIL_ON_UNKNOWN_PROPERTIES` is disabled deliberately** — providers add new response fields over time, and a client that throws on any unrecognized field is fragile against routine, non-breaking API evolution; this is standard defensive JSON deserialization practice, not something specific to LLM APIs.
- **Non-200 responses are converted into a typed exception immediately**, rather than letting a caller discover a problem by inspecting a null or malformed `ChatResponse` — this is the hook the next file's retry logic attaches to.

## A minimal custom exception, carrying what you actually need for retry logic

```java
public class LlmApiException extends RuntimeException {
    private final int statusCode;
    private final String responseBody;

    public LlmApiException(int statusCode, String responseBody) {
        super("LLM API returned status " + statusCode);
        this.statusCode = statusCode;
        this.responseBody = responseBody;
    }

    public int statusCode() { return statusCode; }
    public String responseBody() { return responseBody; }
}
```

Carrying the status code as a typed field (rather than parsing it back out of an exception message string later) is what lets the resilience layer in the next file distinguish a `429` (rate limited, worth retrying) from a `400` (malformed request, retrying won't help) without fragile string matching.

## Building the request from conversation state

Tying back to file 2's `ConversationState`:

```java
ConversationState conversation = new ConversationState();
conversation.addUserMessage("What does idempotency mean for a REST API?");

ChatRequest request = new ChatRequest(
    "claude-model-name",
    1024,
    0.3,
    "You are a precise backend engineering assistant.",
    conversation.currentHistory()
);

ChatResponse response = client.send(request);
String replyText = response.content().get(0).text();

conversation.addAssistantMessage(replyText);
```

Notice this final snippet is where files 1 and 2 of this phase stop being separate concepts and become one working call: the request shape from file 1, populated with the ever-growing history from file 2, sent through the client built in this file.

## Trade-offs and when this matters most

- A hand-rolled client like this is the right choice while you're learning the protocol (this phase) and for small, single-provider tools where you want zero dependencies and full control over every detail (like the CLI project at the end of this phase).
- For a larger production system integrating multiple providers, handling complex streaming/tool-calling flows, and needing battle-tested edge-case handling, adopting Spring AI or LangChain4j (Phase 08) is the right call — but only once you've built this by hand at least once, so you can recognize what those frameworks are doing internally and debug past their abstractions when something goes wrong in production.
- Don't add speculative configuration (connection pool tuning, retry policies, circuit breakers) into this class prematurely — the next file covers resilience patterns deliberately separately, so this client stays a clean, single-responsibility HTTP mapping layer, and resilience logic wraps around it rather than being tangled into it.

## Why this matters next

You have a working client that can send a request and get a response — but nothing here handles the specific ways this call can fail: rate limits, transient overload, and the "succeeded but malformed" cases from earlier in this phase. The next file adds exactly that resilience layer, using patterns that will feel very familiar from any other unreliable-dependency work you've already done, adapted to this API's particular failure modes.