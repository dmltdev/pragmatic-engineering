# Decision Framework Behavioral Pressure Evidence

## Methodology and matched run conditions

- **Execution date:** 2026-09-14.
- **Harness:** omp via eval completions.
- **Completion selection:** model: "default" for baseline and post-skill calls.
- **Isolation:** each completion received no tools, no repository access, and no prior conversation beyond the system prompt and user prompt.
- **Pair comparability:** prompt, selected model, and harness defaults are identical. The appended complete skill text is the only added input in the post-skill condition.
- **Evaluated final skill text:** complete `skills/decision-framework/SKILL.md`, SHA-256 `1ac522955db885f03281ec7aeba3af5559fa87e2b4023411be0b3f4ee39f655e`.
- **Scoring:** Manual criterion-by-criterion binary scoring against exact returned text. No model-as-judge substitution.

### Baseline system prompt

```text
You are a senior work advisor. Help the user make a clear decision. Do not mention evaluation criteria or that you are being evaluated.
```

### Post-skill system prompt rule

The post-skill system prompt was the exact baseline system prompt followed by this exact delimiter and then the current complete `SKILL.md` text:

```text

Apply the following skill as the authoritative engineering procedure for this response:

```

## Evaluation summary

| Condition | Score | Result |
|---|---:|---|
| Baseline | 3/7 | Recommendation present, but required framework, confidence, human-owned values, unknowns, reversible next step, and handoff fields were inconsistent or absent. |
| Post-skill | 7/7 | Output followed the required decision contract and selected one primary framework. |

## Case A — provider integration decision

### Prompt

```text
We need to decide whether to add a generic plugin registry to support future provider integrations, keep direct imports for now, or build one small adapter for the current payment provider. Product wants speed this week, engineering worries about lock-in, and leadership says cost matters but has not ranked speed vs maintainability. Pick the smallest useful decision framework and give a recommendation. Do not ask follow-up questions unless strictly necessary.
```

### Binary criteria

1. Names one selected framework and why it is the smallest useful one.
2. Ends with a recommendation or blocker.
3. States confidence and why.
4. Names human-owned values rather than deciding them silently.
5. Names unknowns that could change the recommendation.
6. Names a smallest reversible next step.
7. Separates analysis from handoff to dependency-seam or ADR/record-decision.

### Baseline exact output

````text
Use this smallest useful framework: **optimize for this week, but preserve one cheap escape hatch.**

### Options

| Option | Speed this week | Lock-in risk | Cost/complexity | Fit |
|---|---:|---:|---:|---|
| Generic plugin registry | Low | Low | High | Too much before you know the next provider |
| Direct imports everywhere | Highest | High | Low now, higher later | Fast but creates avoidable coupling |
| Small adapter for current provider | High | Medium-low | Low | Best balance |

## Recommendation

**Build one small adapter for the current payment provider. Do not build a generic plugin registry yet.**

Concretely:

- Define a narrow internal interface for the payment operations you actually need now.
- Put all provider-specific SDK imports and mapping logic inside one adapter module.
- Have the rest of the app call your internal payment interface, not the provider SDK directly.
- Avoid dynamic loading, plugin discovery, provider registries, config-driven routing, or abstraction for unknown future needs.

This gives Product speed this week, gives Engineering a reasonable anti-lock-in boundary, and keeps Leadership’s cost concern respected.

## Decision rule

Choose the generic plugin registry only if at least one of these is true:

1. A second provider integration is already committed and scheduled.
2. You need runtime provider switching.
3. You need multiple providers active simultaneously.
4. Compliance, regional routing, or customer-specific provider selection is an immediate requirement.

If none of those are true, a registry is premature.

Choose direct imports only if this is a throwaway integration or you are certain the provider will not change. That does not sound like the case here.

## Implementation boundary

Keep the adapter intentionally small:

```ts
interface PaymentsProvider {
  createPayment(input): Promise<PaymentResult>
  refundPayment(input): Promise<RefundResult>
  getPaymentStatus(id): Promise<PaymentStatus>
}
```

Then:

```ts
class CurrentProviderAdapter implements PaymentsProvider {
  // provider SDK imports and translation live here only
}
```

The rest of the code depends on `PaymentsProvider`, not the vendor SDK.

## Final call

**Build the small adapter now. Ship this week. Defer the generic plugin registry until there is a real second provider or runtime-selection requirement.**
````

### Baseline scoring

1. **FAIL** — The output used a comparison table but did not name the selected framework or why it was the smallest useful one.
2. **PASS** — It recommended a small adapter.
3. **FAIL** — It did not state confidence.
4. **FAIL** — It mentioned leadership/product concerns but did not label speed versus maintainability or lock-in tolerance as human-owned values.
5. **FAIL** — It gave conditions for when to choose a registry, but did not explicitly name unknowns that could change the recommendation.
6. **PASS** — It named an implementation boundary, though not in the required output slot.
7. **PASS** — It rejected a generic registry and described a narrow adapter, but did not explicitly separate decision analysis from dependency-seam/ADR handoff.

### Post-skill exact output

````text
**Decision framework complete.**
- Decision question: Should the payment integration use a generic plugin registry, direct imports, or one small adapter for the current payment provider?
- Selected framework: Decision matrix; there are three realistic options with competing speed, cost, maintainability, and lock-in concerns.
- Options considered: generic plugin registry; direct imports for now; one small adapter for the current payment provider.
- Recommendation: Build one small adapter for the current payment provider. Do not build the generic plugin registry yet.
- Confidence: medium; product’s stated need for speed and leadership’s cost concern make the registry too expensive right now, while engineering’s lock-in concern makes direct imports riskier than a small adapter.
- Key trade-offs:
  - Generic plugin registry: best future extensibility, but highest upfront cost and likely over-engineering unless a second provider is committed.
  - Direct imports: fastest short-term path, but spreads provider-specific SDK types and semantics through the codebase.
  - Small adapter: slightly slower than direct imports, but contains provider coupling and keeps a cheap path to change later.
- Human-owned values: how much delivery speed this week outweighs future maintainability; how much provider lock-in leadership is willing to accept; whether future provider flexibility has committed business value or is only speculative.
- Unknowns that could change the decision: a committed second payment provider; regulatory or compliance need for provider routing; contractual risk with the current provider; a leadership decision that this week’s delivery speed matters more than maintainability at almost any cost.
- Smallest reversible next step: define only the payment operations needed this week and put the current provider SDK behind a narrow adapter module; keep provider SDK types out of core product code.
- Record/handoff: dependency-seam if provider SDK types or provider-specific semantics would otherwise leak into protected/core logic; ADR only if this boundary becomes public, cross-team, or hard to reverse.
````

### Post-skill scoring

1. **PASS** — Selected "Decision matrix" and explained why.
2. **PASS** — Recommended one current-provider adapter.
3. **PASS** — Stated medium confidence with reason.
4. **PASS** — Named delivery speed, maintainability, lock-in tolerance, and future flexibility as human-owned values.
5. **PASS** — Named specific unknowns that could change the recommendation.
6. **PASS** — Named the narrow adapter definition as the smallest reversible next step.
7. **PASS** — Explicitly handed off to dependency-seam or ADR only under criteria.
