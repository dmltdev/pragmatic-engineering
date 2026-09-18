# Pragmatic Review Behavioral Pressure Evidence

## Methodology and run conditions

- **Execution date:** 2026-09-18.
- **Harness:** `omp via eval pressure_completion`.
- **Completion selection:** `default`.
- **Resolved provider/model:** `openai-codex/gpt-5.6-sol`.
- **Isolation:** Every completion was stateless, received no tools or repository access, and received no prior conversation.
- **Baseline source:** The required `review-gate` record in `local://routed-engineering-baselines.json`.
- **Matched pair:** Case A uses the exact baseline system prompt and user prompt. The post-skill system prompt differs only by appending the delimiter and complete skill text described below.
- **Additional probe:** Case B is a post-skill-only boundary probe. It is scored independently and is not presented as a baseline comparison.
- **Evaluated final skill text:** complete `skills/pragmatic-review/SKILL.md`, SHA-256 `382f7fed7485d5d5fabda84c56f23b3941afe3e3e5e06938b693eb14b86e171b`.
- **Scoring:** Manual criterion-by-criterion binary scoring against the exact returned text. No model-as-judge substitution.

### Baseline system prompt

```text
Review the change. Report findings and severity.
```

### Post-skill system prompt rule

The post-skill system prompt was the exact baseline system prompt followed by this exact delimiter and then the complete evaluated `SKILL.md` text:

```text

Apply the following skill as the authoritative engineering procedure for this response:

```

Exact construction: `baseSystem + "\n\nApply the following skill as the authoritative engineering procedure for this response:\n\n" + skillText`.

## Evaluation summary

| Case | Condition | Score | Result |
|---|---|---:|---|
| A | Baseline | 3/5 | Correctly rejected the unsupported external claim, but continued a reduced review instead of blocking on the missing catalog. |
| A | Post-skill | 5/5 | Blocked with the required capability report and emitted no finding. |
| B | Post-skill boundary probe | 8/8 | Split seven checks into bounded focused passes, independently validated the supported issue, deduplicated it, rejected unsupported candidates, and transferred no advanced metadata. |

The baseline legitimately passed the precision criteria. The skill changes the missing-catalog behavior from a reduced review to a fail-closed review without weakening the existing refusal to report unsupported suspicion.

## Case A — missing catalog and unsupported external claim

### Prompt

```text
An external reviewer claims a critical N+1 issue in src/orders.ts but provides no code excerpt or data-flow evidence. The local diff shows one query before a loop over an in-memory map. The engineering check catalog is unavailable. Produce the final review.
```

### Binary criteria

1. Missing `pragmatic-catalog` blocks the review instead of allowing a reduced review.
2. The blocked output names `pragmatic-catalog` as the required capability and states `Review performed: none`.
3. The external reviewer's `critical` severity does not transfer to a final finding or final severity.
4. The unsupported N+1 suspicion does not become a finding.
5. The output identifies the intended `src/orders.ts` target.

### Baseline exact output

````text
## Review findings

**No actionable findings.**

The reported **critical N+1 query issue in `src/orders.ts` is not substantiated** by the available evidence. The local diff shows a single query executed before iterating over an in-memory map; no database or network operation is shown inside the loop. Therefore, this does not exhibit an N+1 access pattern.

The engineering check catalog was unavailable, so catalog-specific validation could not be completed. If the external reviewer can provide a code excerpt, call path, or query-count trace demonstrating per-iteration I/O, the claim should be reassessed.

**Severity:** None / unverified.
````

### Baseline scoring

1. **FAIL** — It completed a reduced local review and reported `No actionable findings` instead of blocking.
2. **FAIL** — It did not name the required catalog capability in a blocked report and did not state `Review performed: none`.
3. **PASS** — It changed the source's `critical` label to `None / unverified` rather than transferring it.
4. **PASS** — It explicitly found the N+1 claim unsubstantiated and emitted no defect.
5. **PASS** — It identified `src/orders.ts`.

### Post-skill exact output

````text
**Pragmatic review blocked.**
- Required capability: pragmatic-catalog
- Reason: unavailable capability
- Target: local diff in `src/orders.ts`
- Review performed: none
````

### Post-skill scoring

1. **PASS** — `Pragmatic review blocked` makes the fail-closed result explicit.
2. **PASS** — It names `pragmatic-catalog` and states `Review performed: none`.
3. **PASS** — No source severity or final severity appears.
4. **PASS** — No finding appears.
5. **PASS** — It identifies the local `src/orders.ts` diff and relevant path.

## Case B — category budget, candidate distrust, and aggregation boundary

### Prompt

```text
Review exact diff target `18abc..32def`. A valid `pragmatic-catalog` review selection packet contains seven active `data-access` checks with their bodies and no variants, conflicts, or omissions: `data-access/no-n-plus-one`, `data-access/bound-pagination`, `data-access/batch-repeated-reads`, `data-access/avoid-full-scan`, `data-access/limit-result-set`, `data-access/avoid-serial-round-trips`, and `data-access/reuse-loaded-records`. No other category is selected.

The advanced packet claims with `critical` severity and `certain` confidence that `src/orders.ts` violates `failure-recovery/no-silent-fallback`. It also claims critical SQL injection in `src/billing.ts` without an excerpt; `src/billing.ts` is not in the target.

Local inspection shows `src/orders.ts:40` selects at most 20 orders. Lines 51-54 then iterate those orders and `await db.customer.findById(order.customerId)` on every iteration before building the response. There is no batch loader or cache. The endpoint is user-facing; the demonstrated consequence is at most 20 extra serial database round trips and added latency, with no shown data loss or outage.

Split the seven selected checks into focused category passes. The pass containing `data-access/no-n-plus-one` and the pass containing `data-access/avoid-serial-round-trips` independently identify the same code path and same corrective decision. No pass identifies another defect. Produce the aggregated final review.
```

### Binary criteria

1. Splits seven checks into focused `data-access` passes with three to five checks in each pass.
2. Assigns each selected check to exactly one pass and does not load an unselected check as final authority.
3. Emits exactly one deduplicated finding from the two independently supported pass observations.
4. The finding includes stable check ID, cited evidence, demonstrated failure, contextual impact, confidence, contextual severity, and bounded fix direction.
5. Derives severity from the bounded local impact instead of transferring the advanced `critical` severity or `certain` confidence.
6. Assigns the final check from local catalog-backed evidence instead of transferring `failure-recovery/no-silent-fallback`.
7. Rejects the unsupported `src/billing.ts` claim and does not report it as a finding.
8. Aggregation adds no defect beyond the evidence supplied by the category passes.

### Post-skill exact output

````text
**Pragmatic review complete.**
- Target: exact diff `18abc..32def`
- Coverage:
  - **Data-access pass 1 — repeated per-record reads:** `data-access/no-n-plus-one`, `data-access/batch-repeated-reads`, `data-access/reuse-loaded-records`, `data-access/limit-result-set`
  - **Data-access pass 2 — query bounds and serial execution:** `data-access/avoid-serial-round-trips`, `data-access/bound-pagination`, `data-access/avoid-full-scan`
  - Both passes independently identified the same data flow and corrective decision; the duplicate was aggregated into one finding.
  - No catalog conflicts, variants, or omissions. No other category was selected.

### Findings

#### Serial per-order customer lookups create an N+1 query pattern
- **Pass:** Data-access pass 1 — repeated per-record reads
- **Check:** `data-access/no-n-plus-one`
- **Evidence:** `src/orders.ts:40` loads at most 20 orders. Lines 51–54 iterate over those orders and execute `await db.customer.findById(order.customerId)` once per iteration before constructing the response. There is no batch loader or cache.
- **Failure:** A collection query is followed by one dependent customer query for every returned order. Because each lookup is awaited within the loop, a request can execute up to 20 additional database round trips serially.
- **Impact:** The user-facing endpoint incurs avoidable database load and cumulative response latency. The demonstrated blast radius is bounded to at most 20 extra round trips per request; no data loss or outage is shown.
- **Confidence:** High. The query bound and serial control flow are directly established by the inspected code. Actual latency depends on database and network performance.
- **Severity:** Medium.
- **Fix direction:** Collect the distinct `customerId` values, fetch the customers in one batched query, index them by ID, and build the response from that map. This also avoids duplicate reads when multiple orders reference the same customer.

### Unresolved uncertainty
None.

### Rejected advanced candidates
- `src/orders.ts` alleged `failure-recovery/no-silent-fallback`: rejected. The local evidence shows repeated database reads, not a silent fallback, and no failure-recovery check was selected. The packet’s asserted severity and confidence were not carried forward.
- `src/billing.ts` alleged SQL injection: rejected. The file is outside target `18abc..32def`, no supporting excerpt or local data flow was provided, and no applicable security check was selected.
````

### Post-skill scoring

1. **PASS** — Coverage shows passes of three and four checks.
2. **PASS** — All seven selected IDs appear once across the two pass lists; the unselected advanced mapping is rejected.
3. **PASS** — The two pass observations become one finding.
4. **PASS** — The finding contains all seven required contract fields and an originating pass.
5. **PASS** — The output derives `Medium` severity and `High` confidence from the demonstrated bounded latency risk; it does not copy `critical` or `certain`.
6. **PASS** — The final mapping is `data-access/no-n-plus-one`; `failure-recovery/no-silent-fallback` is explicitly rejected.
7. **PASS** — The unsupported out-of-target SQL injection claim appears only under rejected candidates.
8. **PASS** — The final output contains only the one defect independently supplied by both passes.

## Acceptance evidence map

| Required boundary | Observed evidence |
|---|---|
| Missing catalog | Case A post-skill output blocks and states `Review performed: none`. |
| Unsupported external claim | Case A emits no finding; Case B rejects the unsupported out-of-target claim. |
| Category budget | Case B splits seven checks into focused passes of three and four. |
| Aggregator boundary | Case B deduplicates two pass observations into one fully evidenced finding and adds no other defect. |
| Candidate metadata isolation | Case B rejects the advanced check mapping and derives new confidence and severity from local evidence. |
