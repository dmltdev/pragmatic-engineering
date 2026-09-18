---
name: pragmatic-coding
description: Use when implementing a non-trivial production change involving state, concurrency, asynchronous work, data access, external boundaries, caching, or failure behavior.
---

# Pragmatic Coding

Add a small, risk-specific guardrail packet to an existing implementation workflow without taking ownership of that workflow.

## Boundary

This skill enriches implementation. It does not own or replace the implementation plan, edits, tests, commits, acceptance, or delivery lifecycle state. The host workflow remains responsible for those activities and their normal verification.

Use `pragmatic-catalog` only as an optional capability handoff. Refer to domain skills by skill name and catalog checks by stable ID. Never read another skill's filesystem path.

## Gate

First classify the requested change from the request and repository evidence.

Skip enrichment when the entire change is confirmed to be documentation-only, formatting-only, a mechanical rename, or otherwise behavior-equivalent and low risk. Return exactly:

```text
no pragmatic enrichment required
```

Do not inspect catalog availability or add any other context after this result.

Continue when production behavior can change and the change involves at least one of:

- state representation or transitions;
- concurrency, asynchronous work, task ownership, or ordering;
- data access shape, query cardinality, or data volume;
- untrusted or external boundaries;
- caching, freshness, or invalidation; or
- failure, fallback, retry, or recovery behavior.

A syntax signal alone does not establish risk. Ground the decision in the changed behavior and data or control flow.

## Procedure

### 1. Build the risk inventory

Record only facts needed for routing:

- changed production behavior and affected path;
- languages, frameworks, and mechanisms;
- relevant state transitions and ownership;
- asynchronous work and lifetime;
- data-access cardinality and volume;
- trust boundaries and validation;
- cache, freshness, and invalidation behavior; and
- failure, fallback, retry, and recovery behavior.

Omit absent dimensions. Do not turn the inventory into an implementation plan.

### 2. Select optional domain guidance

Load a self-contained domain skill only when its own trigger matches current evidence. Apply that skill as complete guidance even when the catalog is absent. Collect any stable check IDs it suggests as routing hints, not mandatory selections.

Do not load a domain skill merely to obtain catalog suggestions. An unavailable optional domain skill does not block coding.

### 3. Request catalog enrichment

When the `pragmatic-catalog` capability is available, request a selection packet with:

- phase `coding`;
- the compact risk inventory;
- observed ecosystem and mechanism tags;
- evidence-supported stable IDs suggested by selected domain skills;
- applicable project policies; and
- a hard budget of four checks.

Accept zero to four applicable active checks and only their selected bodies and matching variants. Let `pragmatic-catalog` resolve its own resources.

Never load or request the full catalog, a whole category, candidates, fixtures, provenance, shadow checks, or unmatched variants. Candidate and shadow material must not change coding guidance or produce developer-facing output.

Within the applicable active set, preserve catalog authority and selection order. Domain-skill suggestions still require current evidence. If more than four active checks apply, use the four selected by current risk and disclose the omitted stable IDs or risk areas as deferred to `pragmatic-review`; do not load their bodies.

If the catalog capability is unavailable, continue with any selected self-contained domain guidance and include this exact disclosure:

```text
catalog enrichment unavailable
```

Do not infer catalog contents, IDs, variants, status, or guidance. Missing catalog enrichment never blocks the host implementation workflow.

### 4. Produce the guardrail packet

Before implementation, keep the packet concise:

```text
Pragmatic coding guardrails
Risk inventory: <compact facts>
Domain guidance: <skill name and applicable constraint, or none>
Catalog checks:
- <stable ID and matching variant if any>: <preventive constraint>; prove <observable condition>
Deferred to pragmatic-review: <stable IDs or risk areas omitted by the budget, or none>
Catalog status: available | catalog enrichment unavailable
```

Include no more than four catalog checks. Summarize only the selected constraints and proof obligations. Do not copy unrelated check prose into the packet.

If the available catalog returns no applicable active check and no domain skill applies, return exactly:

```text
no pragmatic enrichment required
```

Add no heading, explanation, risk inventory, or generic advice to that result.

### 5. Require post-change proof

After the host workflow makes the change, inspect the changed behavior and use its observed verification evidence. For every selected catalog check and domain constraint, record:

```text
Post-change proof
- <stable check ID or domain skill>: <satisfied | unresolved>; <specific changed path and observed behavior or missing evidence>
```

Proof must address the routed risk, not merely compilation, test existence, or a mock echo. If evidence is missing, mark the item `unresolved` and return the bounded proof obligation to the host workflow. Do not claim that tests passed, the change is complete, or lifecycle state changed on the host workflow's behalf.

If a demonstrated failure is reported as a developer-facing finding, it must include the stable check ID, cited evidence, demonstrated failure, impact, confidence, severity, and a bounded fix direction.

## Non-negotiable rules

- Zero checks is valid; silence is better than irrelevant context.
- Four catalog checks is a hard maximum, not a target.
- Project policy and current repository evidence outrank reusable guidance.
- Signals route inspection; they do not prove a defect.
- Missing catalog fails open. Missing proof does not become fake certainty.
- This skill supplies guardrails and proof obligations only; it never takes over planning, implementation, testing, source control, acceptance, or delivery.
