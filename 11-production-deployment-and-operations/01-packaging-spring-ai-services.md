# Packaging Spring AI Services

## Why this exists

You already know how to package and deploy a Spring Boot service — this file does not re-teach that. What it covers is the small set of genuinely new considerations that arise specifically because this service talks to an LLM provider, wraps Phase 07's agent loops, and depends on Phase 10's observability infrastructure being wired in correctly before traffic ever hits it. Treat this as a checklist of deltas from your existing packaging practice, not a new packaging discipline from scratch.

## Configuration surface: more externalized values than a typical service

An ordinary Spring Boot service's externalized configuration usually covers database connections, a handful of feature flags, and maybe a few tunable timeouts. An agent service's configuration surface is meaningfully larger, because nearly every phase of this handbook introduced its own tunable value, and all of them need to be externalized rather than hardcoded, exactly per the twelve-factor discipline you already apply to database URLs and connection pool sizes:

```yaml
agent:
  llm:
    provider: anthropic
    model: some-model-name
    max-tokens: 1024
    temperature: 0.2
  budget:
    max-iterations: 15
    max-cost-usd: 2.00
    max-wall-clock-seconds: 60
  memory:
    strategy: summarization
    summarize-after-messages: 10
  retrieval:
    top-k: 3
    similarity-threshold: 0.6
  tools:
    confirmation-required: [restart_pod, scale_deployment]
```

The reason to externalize every one of these rather than hardcoding sensible-looking defaults directly in code: Phase 10's regression testing and Phase 11's own canary-release discipline (file 8) both depend on being able to adjust these values — a budget limit, a similarity threshold, a memory strategy — per environment or per rollout stage, without a code change and redeploy for every adjustment. A `max-cost-usd` value that's fine for a staging environment's test traffic might be deliberately tighter or looser in production, and that difference should live in configuration, not in two different code paths.

## Health checks need to mean something different for an agent service

A typical Spring Boot service's health check (`/actuator/health`) verifies the service process is up and its immediate dependencies (database, cache) are reachable. For an agent service, "the process is up" tells you almost nothing about whether it can actually do its job — the LLM provider itself, an entirely external dependency outside your own infrastructure's control, is the dependency that actually determines whether requests will succeed, and Phase 02, file 9's circuit breaker state is a more meaningful signal of service health than process liveness alone:

```java
@Component
public class LlmProviderHealthIndicator implements HealthIndicator {

    private final CircuitBreaker llmCircuitBreaker; // the same circuit breaker from Phase 02, file 9

    @Override
    public Health health() {
        CircuitBreaker.State state = llmCircuitBreaker.getState();
        if (state == CircuitBreaker.State.OPEN) {
            return Health.down()
                .withDetail("reason", "LLM provider circuit breaker is open")
                .withDetail("provider", "anthropic")
                .build();
        }
        return Health.up().withDetail("circuitBreakerState", state.toString()).build();
    }
}
```

Wiring your circuit breaker's state directly into your health check means your existing orchestration tooling (Kubernetes readiness probes, load balancer health checks — infrastructure you already operate) correctly stops routing traffic to an instance whose upstream LLM dependency is currently failing, exactly the way you'd already want a health check to reflect a database outage, applied here to a dependency that lives entirely outside your own infrastructure's boundary.

## Startup-time validation: fail fast on configuration that would otherwise fail silently mid-request

Given how much configuration this phase's opening section externalized, a genuinely valuable practice — more valuable here than for a typical service, because a misconfigured agent can fail in ways that are expensive (Phase 01) rather than just broken — is validating critical configuration at startup, before accepting any traffic, rather than discovering a bad value the first time a real request exercises it:

```java
@Component
public class AgentConfigValidator implements ApplicationRunner {

    private final AgentProperties properties;

    @Override
    public void run(ApplicationArguments args) {
        if (properties.getBudget().getMaxCostUsd() <= 0) {
            throw new IllegalStateException(
                "agent.budget.max-cost-usd must be positive — found: " + properties.getBudget().getMaxCostUsd()
            );
        }
        if (properties.getLlm().getApiKey() == null || properties.getLlm().getApiKey().isBlank()) {
            throw new IllegalStateException("LLM API key is not configured — refusing to start");
        }
        // Additional checks: retrieval threshold within [0,1], confirmation-required tools
        // actually exist in the registered tool set, memory strategy is a recognized value, etc.
    }
}
```

This is directly analogous to any other fail-fast startup validation you already write for a Spring Boot service (a required datasource property, a malformed connection string) — the only thing genuinely new is *which* values are worth validating, since a misconfigured `max-cost-usd` of zero or a missing confirmation-gate entry (Phase 05, file 4) is a production risk specific to this kind of service, not a generic configuration mistake.

## Dependency footprint: what you actually need in the deployable artifact

If you've adopted LangChain4j or Spring AI (Phase 08), your dependency footprint includes the framework itself plus whichever provider integrations and vector store backends (Phase 03) you're actually using — worth auditing deliberately rather than pulling in every available integration "in case it's needed later." This is ordinary dependency hygiene you already practice, but worth calling out specifically here because AI framework dependency trees can be large, and an unused vector-store integration or document-loader dependency adds real image size and attack surface for a capability your service doesn't actually use.

## Trade-offs and when this matters most

- For an internal prototype or a low-traffic tool, the health-check and startup-validation additions in this file are good practice but not urgent — a simpler packaging approach, closer to what you'd already do for any Spring Boot service, is a reasonable starting point.
- For anything serving real production traffic, wiring circuit breaker state into health checks and validating critical configuration at startup are both cheap to implement and meaningfully reduce the chance of a misconfiguration reaching real users as a confusing runtime failure rather than a clear startup error.
- Don't treat this file's checklist as exhaustive — it's the delta from your existing packaging practice specific to what this handbook has built; your organization's own standard packaging requirements (image scanning, base image policy, resource limits) still apply unchanged on top of everything here.

## Why this matters next

Packaging gets the service itself into a deployable, properly health-checked shape. The next file addresses a concern this file's configuration section already gestured at but deliberately deferred: the model API key itself, and the full production-scale secrets and spend-management discipline Phase 02, file 3 first raised in a single-client context.