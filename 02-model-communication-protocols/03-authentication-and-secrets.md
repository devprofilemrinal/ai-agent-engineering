# Authentication and Secrets

## Why this exists

This is the one file in this phase where the honest message is: almost nothing here is new to you, and treating it as if it were novel is itself a risk — engineers sometimes handle AI API credentials more casually than they would a database password or a payment-provider key, precisely because the AI integration "feels" like an experiment rather than production infrastructure. It rarely stays that way for long, and the credential deserves first-class treatment from day one.

## How authentication actually works on the wire

Nearly every provider uses a simple header-based API key, sent on every request:

```
POST /v1/messages HTTP/1.1
Host: api.example-provider.com
x-api-key: sk-ant-yourActualKeyValueHere
Content-Type: application/json
```

Some providers instead use the standard `Authorization: Bearer <token>` header, which will already be familiar from any OAuth2-protected API you've integrated with in a Spring Security context. Either way, the mechanism is exactly the pattern you already know: a secret string attached to every request, proving the caller is authorized, with no session or handshake beyond that single header.

## What's genuinely the same as any other secret you manage

- **Never commit it to source control.** The same rule as a database password or a third-party API key.
- **Never embed it in client-side code** — a mobile app, a browser-side JavaScript bundle, an Android APK. If a key can be extracted from a shipped client, it will be, and unlike a scoped, revocable session token, a raw provider API key extracted this way often has broad billing-level access.
- **Load it from environment variables or a secrets manager**, exactly as you already do for any other credential in a Spring Boot application:

```java
String apiKey = System.getenv("LLM_API_KEY");
if (apiKey == null || apiKey.isBlank()) {
    throw new IllegalStateException("LLM_API_KEY environment variable is not set");
}
```

In a Spring context, this is `application.yml` plus `${LLM_API_KEY}` plus a properly scoped secrets backend (AWS Secrets Manager, Vault, Kubernetes Secrets) — the exact same infrastructure you already run for your database credentials, reused rather than reinvented for this new dependency.

## What's worth calling out as specific to this kind of credential

- **Cost blast radius is unusually direct.** A leaked database credential typically requires further exploitation to cause financial damage. A leaked LLM API key can be used directly, by anyone who has it, to run billable requests against your account — the "damage" and "the credential" are separated by zero steps. This argues for tighter rotation discipline and stricter scoping (separate keys per environment, per service, or per team) than you might apply to a lower-blast-radius secret.
- **Per-key usage limits and monitoring are a first-class mitigation, not a nice-to-have.** Most providers let you set spend limits or rate limits per API key. Setting a conservative limit on a development key is the direct analogue of using a low-privilege IAM role for a dev environment instead of reusing production credentials — cheap insurance against a runaway loop (Phase 07's agent loops, if buggy, can call the API far more times than a human would ever click a button) burning an unexpectedly large bill before anyone notices.
- **Key scoping by environment and by service is worth doing early**, not retrofitting later — a single shared key across dev, staging, and production makes it impossible to answer "which environment caused this cost spike" after the fact, the same diagnostic gap you'd avoid by using separate database credentials per environment.

## A minimal, correct pattern in Java

```java
public class LlmClientConfig {

    private final String apiKey;
    private final String baseUrl;

    public LlmClientConfig() {
        this.apiKey = requireEnv("LLM_API_KEY");
        this.baseUrl = System.getenv().getOrDefault(
            "LLM_API_BASE_URL", "https://api.example-provider.com"
        );
    }

    private static String requireEnv(String name) {
        String value = System.getenv(name);
        if (value == null || value.isBlank()) {
            throw new IllegalStateException(
                "Required environment variable not set: " + name
            );
        }
        return value;
    }

    public String apiKey() { return apiKey; }
    public String baseUrl() { return baseUrl; }
}
```

Note what this deliberately does *not* do: no default fallback key baked into the code, no logging of the key value anywhere (including in exception messages — `requireEnv` reports the missing variable's *name*, never a partial or full credential value), and the base URL is configurable rather than hardcoded, which matters once you're juggling multiple providers (file 7).

## Trade-offs and when this matters most

- For a personal experiment hitting a free tier, strict key scoping and spend-limit discipline is arguably overkill.
- For anything a team shares, anything hitting a paid tier, or anything that will eventually run in a loop (any agent, Phase 07 onward), scoped keys with spend limits and environment separation stop being optional — the cost mechanics from Phase 01 mean a misbehaving loop can generate a meaningful bill in a very short window, far faster than a comparable bug in a traditional API integration typically would.
- Don't over-engineer secret rotation cadence beyond what your existing secrets-management policy already mandates for other credentials — reuse your existing operational maturity here rather than inventing a parallel, AI-specific process.

## Why this matters next

You now know how to prove who's calling and how to keep that credential safe. The next file covers a genuinely new wire-level behavior — streaming responses via Server-Sent Events — which is where your HTTP client code starts looking meaningfully different from a typical single-request/single-response REST integration.