# Pragmatic Coding Behavioral Pressure Evidence

## Methodology and run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP via the `pressure_completion` eval tool.
- **Completion selection:** selector `default`; resolved provider/model `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** Each post-skill completion was independent, stateless, had no tools or repository access, and received no prior conversation.
- **Matched comparison:** The recorded `coding-gate` baseline prompt and exact output came from the approved baseline artifact. The post-skill comparison used the identical baseline system prompt and user prompt.
- **Post-skill construction:** Every post-skill system prompt was `baseSystem + "\n\nApply the following skill as the authoritative engineering procedure for this response:\n\n" + skillText`, where `skillText` was the complete final `skills/pragmatic-coding/SKILL.md`.
- **Additional conditions:** The routed-budget case supplied a bounded, hypothetical catalog selection packet because an isolated completion cannot invoke repository capabilities. Its `active` labels test the consumer contract; they do not assert or change the current repository index status. The missing-catalog case explicitly withheld that capability. No case supplied catalog filesystem paths or check-resource contents outside its selected packet.
- **Evaluated final skill text:** complete `skills/pragmatic-coding/SKILL.md`, SHA-256 `0a8b402805acdb9e9adfbf43acb415f975448b0a378eec319a97f88aaf78fbc8`.
- **Scoring:** Manual criterion-by-criterion binary scoring against the exact returned text. No model-as-judge substitution.

### Baseline system prompt

```text
Give implementation guidance for the requested change. Be concise.
```

### Post-skill delimiter

```text

Apply the following skill as the authoritative engineering procedure for this response:

```

## Evaluation summary

| Condition | Score | Result |
|---|---:|---|
| Recorded matched baseline | 2/5 | Identified the relevant runtime risks and stayed within an advisory boundary, but added low-risk context, exceeded the four-check budget, and omitted the required missing-catalog disclosure. |
| Matched post-skill completion | 5/5 | Skipped the low-risk change, inventoried the relevant risks, capped the requested catalog areas at four, failed open, and retained the standalone domain guide. |
| Standalone low-risk branch | 1/1 | Returned the exact no-enrichment result with no added context. |
| Routed relevant-risk and budget branch | 5/5 | Used four selected active checks, one matching variant, deferred the fifth risk, ignored forbidden resources, and supplied bounded proof obligations. |
| Standalone catalog no-match branch | 1/1 | Returned the exact no-match result with no added context. |
| Missing-catalog domain branch | 4/4 | Disclosed unavailable enrichment, retained domain guidance, invented no checks, and did not take over implementation lifecycle work. |

## Case A — matched `coding-gate` baseline

### Prompt

```text
Change A only fixes spelling in README. Change B adds a TypeScript webhook handler that parses external JSON, starts an audit promise, queries per item, and silently falls back on provider failure. State which engineering checks to load for each change. The catalog is unavailable for a third change that already has a self-contained dependency-boundary guide.
```

### Binary criteria

1. Adds no pragmatic context for Change A and gives the no-enrichment result.
2. Inventories Change B's external-data, asynchronous-work, per-item-access, and failure-fallback risks.
3. Requests no more than four catalog check areas and invents no stable ID without a catalog packet.
4. Fails open for the third change, includes `catalog enrichment unavailable`, and retains the self-contained dependency-boundary guidance.
5. Does not take ownership of plans, edits, tests, commits, acceptance, or lifecycle state.

### Baseline exact output

````text
- **Change A — README spelling only:** Load the **documentation/spelling** check. No implementation, security, or runtime checks are needed.
- **Change B — TypeScript webhook handler:** Load checks for:
  - **TypeScript correctness**
  - **Untrusted/external JSON validation**
  - **Promise lifecycle and async error handling** (the started audit promise must be awaited or deliberately managed)
  - **Per-item query / N+1 performance**
  - **Provider failure handling and observability** (avoid silent fallback; log/measure failures appropriately)
  - **Webhook security**, including authentication/signature verification and replay protection, if applicable
- **Third change:** Since the catalog is unavailable and the dependency-boundary guide is self-contained, use that guide directly; do not block on or attempt to infer catalog checks.
````

### Baseline scoring

1. **FAIL** — It loaded a documentation/spelling check instead of adding no pragmatic context.
2. **PASS** — It named untrusted JSON, promise lifecycle, per-item queries, and provider-failure handling.
3. **FAIL** — It proposed six check areas, exceeding the hard budget of four.
4. **FAIL** — It retained the self-contained guide and failed open, but omitted the exact disclosure `catalog enrichment unavailable`.
5. **PASS** — It stayed advisory and did not claim ownership of implementation or lifecycle work.

**Total: 2/5**

### Post-skill exact output

````text
**Change A**

no pragmatic enrichment required

**Change B**

Pragmatic coding guardrails  
Risk inventory: TypeScript webhook; untrusted external JSON parsing; audit promise ownership/lifetime; per-item queries with request-dependent cardinality; silent provider-failure fallback.  
Domain guidance: none identified.  
Catalog checks: none loaded; request up to four checks covering boundary validation, asynchronous task ownership, query amplification, and explicit fallback/observability. Stable IDs must come from the catalog.  
Deferred to pragmatic-review: those four risk areas if they remain unverified.  
Catalog status: catalog enrichment unavailable

**Third change**

Pragmatic coding guardrails  
Risk inventory: dependency-boundary behavior; further implementation details not provided.  
Domain guidance: apply the self-contained dependency-boundary guide in full; no catalog is required for it to remain authoritative.  
Catalog checks: none loaded.  
Deferred to pragmatic-review: any dependency-boundary proof obligations not demonstrated after the change.  
Catalog status: catalog enrichment unavailable
````

### Post-skill scoring

1. **PASS** — The Change A branch contains only the required no-enrichment result after the prompt-required case label; Case B below proves the standalone exact output.
2. **PASS** — The risk inventory names all four evidenced runtime risk areas.
3. **PASS** — It requests four areas and refuses to invent stable IDs without a catalog packet.
4. **PASS** — It uses the exact missing-catalog disclosure and applies the self-contained dependency-boundary guide in full.
5. **PASS** — It supplies guardrails only and makes no plan, edit, test, commit, acceptance, or lifecycle claim.

**Total: 5/5**

**Condition note:** The prompt states that the catalog is unavailable for the third change, but the isolated completion has no callable catalog for any branch. The response therefore fails open for Change B rather than fabricating check IDs. Case C supplies a real bounded selection packet and tests the available-catalog path.

## Case B — standalone low-risk skip

### Prompt

```text
The entire requested change corrects one misspelled word in README. Repository evidence confirms no code, configuration, generated output, or behavior changes. Return the pragmatic coding enrichment result.
```

### Binary criterion

1. Returns exactly `no pragmatic enrichment required` with no heading or other context.

### Exact output

````text
no pragmatic enrichment required
````

### Scoring

1. **PASS** — The output is the exact required string and nothing else.

**Total: 1/1**

## Case C — relevant risks, hard budget, and forbidden resources

### Prompt

```text
Prepare pragmatic coding guardrails for a TypeScript webhook handler that receives untrusted JSON, starts an audit promise whose lifetime must be owned, reads one customer per payload item, silently returns [] on provider failure, and updates status through independent booleans. The available pragmatic-catalog selection packet for phase coding and budget 4 selects active checks boundary-trust/validate-external-data, concurrency-async/no-unobserved-promise with its matching TypeScript variant, data-access/no-n-plus-one, and failure-recovery/no-silent-fallback. It omits active state-modeling/no-impossible-state because of the budget. Nearby catalog resources also contain shadow checks, candidates, fixtures, a whole category pack, and unmatched variants. Do not implement the change; produce only the enrichment packet and proof obligations.
```

### Binary criteria

1. Loads exactly the four selected active stable IDs and no fifth check body.
2. Loads only the matching TypeScript variant and no unmatched variant.
3. Defers `state-modeling/no-impossible-state` to `pragmatic-review` because of the budget.
4. Does not use or expose shadow checks, candidates, fixtures, a whole category, or unmatched variants.
5. Supplies risk-specific proof obligations without planning or implementing the change.

### Exact output

````text
Pragmatic coding guardrails  
Risk inventory: TypeScript webhook path accepts untrusted JSON; starts an audit promise requiring explicit lifetime ownership and rejection observation; performs one customer read per payload item, creating query cardinality proportional to payload size; converts provider failure to silent `[]`; represents status with independent booleans that can form impossible combinations.  
Domain guidance: none  
Catalog checks:
- `boundary-trust/validate-external-data`: Validate and narrow the webhook body at ingress before business logic, audit work, or data access; prove malformed, missing, oversized, and unexpected values are rejected or safely normalized.
- `concurrency-async/no-unobserved-promise` (TypeScript variant): Explicitly await, return, or transfer the audit promise to a documented owner, with rejection handling; prove request completion cannot leave an unintentionally unobserved promise or rejection.
- `data-access/no-n-plus-one`: Replace per-item customer reads with bounded bulk access or justified deduplicated batching; prove datastore call count does not grow one-for-one with payload items.
- `failure-recovery/no-silent-fallback`: Do not silently translate provider failure into an indistinguishable empty result; prove callers or telemetry can distinguish genuine `[]` from provider failure and that the chosen recovery behavior is explicit.
Deferred to pragmatic-review: `state-modeling/no-impossible-state`—independent status booleans permit contradictory or impossible states.  
Catalog status: available

Post-change proof
- `boundary-trust/validate-external-data`: unresolved; inspect the changed webhook ingress and exercise invalid JSON shapes, missing fields, unexpected types, excessive collections or values, and valid payloads to show validation occurs before trusted use.
- `concurrency-async/no-unobserved-promise` (TypeScript variant): unresolved; inspect the audit promise’s owner and force audit rejection across handler success and failure paths to show the promise is awaited, returned, or explicitly supervised without an unhandled rejection.
- `data-access/no-n-plus-one`: unresolved; observe query counts for payloads with multiple and duplicate customer references to show customer reads are bounded or batched rather than one read per item.
- `failure-recovery/no-silent-fallback`: unresolved; force provider failure and compare it with a successful empty response to show the outcomes are observably distinct through the API contract, propagated error, structured result, or required telemetry.
````

### Scoring

1. **PASS** — The packet contains exactly four selected active IDs.
2. **PASS** — It names only the selected TypeScript variant.
3. **PASS** — It defers the fifth active ID without loading its body.
4. **PASS** — None of the forbidden nearby resources appears in the output.
5. **PASS** — It records bounded observable proof obligations and performs no implementation work.

**Total: 5/5**

**Status note:** This isolated case does not activate any repository check. It tests behavior for a packet that explicitly labels four entries active; the prompt's separate shadow resources remain unused and absent from developer-facing output.

## Case D — standalone catalog no match

### Prompt

```text
A production change replaces one pure, local string concatenation with an equivalent standard-library join. Repository evidence confirms no state, async work, data access, external boundary, cache, failure behavior, or matching domain skill. The available pragmatic-catalog selection packet has status no-match and contains no check bodies. Return the pragmatic coding enrichment result.
```

### Binary criterion

1. Returns exactly `no pragmatic enrichment required` with no heading or generic advice.

### Exact output

````text
no pragmatic enrichment required
````

### Scoring

1. **PASS** — The empty catalog selection maps to the exact required output.

**Total: 1/1**

## Case E — missing catalog with self-contained domain guidance

### Prompt

```text
A production change moves payment-provider SDK calls out of protected checkout logic. The self-contained dependency-seam skill applies and requires a caller-owned interface, provider translation at the app-side adapter, existing composition-root wiring, and observed boundary behavior. The pragmatic-catalog capability is unavailable. Produce only the pragmatic coding enrichment; do not plan, edit, test, commit, or mark lifecycle state.
```

### Binary criteria

1. Includes the exact disclosure `catalog enrichment unavailable` and does not block coding.
2. Retains the supplied self-contained `dependency-seam` constraints.
3. Loads or invents no catalog check or stable ID.
4. Does not take ownership of plans, edits, tests, commits, acceptance, or lifecycle state.

### Exact output

````text
Pragmatic coding guardrails  
Risk inventory: Payment-provider SDK calls move outside protected checkout logic, changing dependency ownership and an external failure boundary; affected behavior includes provider request/response translation and checkout success/failure propagation.  
Domain guidance: dependency-seam — define a caller-owned interface, translate provider types and failures in the app-side adapter, wire the adapter through the existing composition root, and preserve observable boundary behavior.  
Catalog checks: none  
Deferred to pragmatic-review: none  
Catalog status: catalog enrichment unavailable
````

### Scoring

1. **PASS** — The exact disclosure is present and the response continues with available guidance.
2. **PASS** — It retains all four supplied dependency-seam constraints.
3. **PASS** — It reports no catalog checks and invents no ID.
4. **PASS** — It provides only an enrichment packet.

**Total: 4/4**
