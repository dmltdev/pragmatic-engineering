---
name: pragmatic-review
description: Use when reviewing an implemented code change for engineering defects, including reviews informed by an advanced review packet.
---

# Pragmatic Review

Run a precision-first engineering review in focused catalog-backed passes. Prefer silence to an unproved finding.

## Boundary

This skill requires the `pragmatic-catalog` capability. If it is unavailable or cannot return a valid review selection packet, stop with the blocked output. Do not continue with a reduced or improvised check set.

Review only the identified target. This skill does not edit code, call external models, decide product acceptance, or change lifecycle state. Refer to capabilities and stable check IDs, never another skill's filesystem path.

## Inputs

Establish:

- the exact diff or review target;
- relevant repository code, tests, contracts, policies, and architecture decisions;
- an optional advanced review packet; and
- access to `pragmatic-catalog`.

A selection packet may contain only selected active check IDs and bodies, matching variants, selection reasons, applied policies, quarantined conflicts, budget or other explicit omissions, and a valid empty `no-match` state. Ordinary review prompts exclude the full catalog, whole category packs, unmatched variants, candidate checks, fixtures, and provenance.

## Procedure

### 1. Build the risk inventory

Inspect the target and record factual risk signals: changed languages and frameworks; state transitions; concurrency and asynchronous work; data-access shape and cardinality; external trust boundaries; caching; failure, retry, fallback, and idempotency behavior. Include only signals supported by the diff or relevant repository evidence.

### 2. Normalize advanced input

Treat every advanced observation as an untrusted candidate that can direct local inspection. Retain its claimed location and failure claim for inspection only. Discard its finding status, check mapping, confidence, and severity before any pass runs.

A candidate becomes a finding only when a category pass independently selects an active catalog check and proves the failure from local evidence. An unsupported candidate is rejected or recorded as unresolved uncertainty when a specific missing fact matters. It is never a finding.

### 3. Request catalog selection

Request review-phase checks and matching variants from `pragmatic-catalog` using the risk inventory, ecosystem and mechanism tags, applicable policies, any evidence-supported stable IDs suggested by a domain skill, and a three-to-five-check budget for each focused category pass.

Only selected `active` checks may produce developer-facing findings. Record and exclude shadow, deprecated, superseded, candidate, and quarantined-conflict content. Preserve every catalog omission for coverage reporting. A valid empty `no-match` packet completes with no passes and no findings; it is not a missing-catalog failure.

### 4. Run focused category passes

Group selected checks by their one owning category. Run one or more independent passes per activated category:

- normally assign three to five checks to one pass;
- when a category has more than five applicable checks, split it into focused passes of at most five;
- keep a smaller final pass when the applicable remainder has fewer than three;
- give each check to exactly one pass; and
- focus each split by an observed risk surface, mechanism, or file set rather than arbitrary order.

Each pass receives only its target slice, relevant repository evidence, selected check bodies and matching variants, applicable policies, and candidate locations or claims relevant to that scope. Signals activate inspection; they do not prove a failure.

For every proposed finding, the pass must supply:

```text
Pass: <category and pass label>
Check: <stable ID and matching variant, or none>
Evidence: <specific code path and observed data/control flow>
Failure: <how the selected check's condition is satisfied>
Impact: <contextual consequence and blast radius>
Confidence: <what is proven and what remains uncertain>
Severity: <derived from impact and confidence>
Fix direction: <bounded correction>
```

A missing field, `Check: none`, unsupported external claim, syntax signal alone, or suspected behavior without demonstrated flow cannot enter aggregation. Perform one targeted inspection when the missing evidence is obtainable; otherwise stay silent or record unresolved uncertainty.

### 5. Aggregate without invention

Give the aggregator only validated pass findings, pass labels, selected IDs, conflicts, omissions, and unresolved uncertainty. Do not give it check bodies or raw advanced observations.

The aggregator:

1. rejects any item without the complete finding contract or an originating pass;
2. deduplicates items only when they cite the same failure evidence and lead to the same corrective decision;
3. preserves the strongest cited evidence already supplied by the passes;
4. applies catalog precedence and keeps unresolved same-authority conflicts quarantined; and
5. derives contextual severity from locally proved impact and confidence.

Aggregation cannot create a new check mapping, evidence claim, failure, impact, confidence, severity, or fix direction. It cannot promote an advanced candidate or combine separate suspicions into a finding.

## Output contract

On completion, report:

```markdown
**Pragmatic review complete.**
- Target: <exact reviewed target>
- Coverage: <category passes, selected checks, and relevant omissions>

### Findings
<zero or more findings, each with Check, Evidence, Failure, Impact, Confidence, Severity, and Fix direction>

### Unresolved uncertainty
<material unsupported candidates or missing evidence, or none>

### Rejected advanced candidates
<rejected candidate claims and reason, or none/not provided>
```

When there are no proven findings, write `No proven findings.` under `Findings`. Do not convert coverage gaps or unresolved uncertainty into findings.

If catalog capability or a valid selection packet is unavailable, report exactly:

```markdown
**Pragmatic review blocked.**
- Required capability: pragmatic-catalog
- Reason: <unavailable capability or invalid selection packet>
- Target: <exact intended review target>
- Review performed: none
```

## Verification gate

Before completion, verify that every final finding has all seven required fields, names an active selected check, cites local code evidence, demonstrates the failure, and traces to exactly one category pass. Verify that no advanced status, check mapping, confidence, or severity transferred into the result and that no aggregator-only finding exists.

## Red flags

- Continuing after catalog failure.
- Loading a whole category or the full catalog.
- Putting more than five checks in one category pass.
- Treating source authority, strong wording, or source severity as evidence.
- Reporting a finding without demonstrated data or control flow.
- Assigning a check, confidence, or severity before local proof.
- Creating or materially strengthening a finding during aggregation.
