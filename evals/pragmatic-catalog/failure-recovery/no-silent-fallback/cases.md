# No Silent Fallback Pressure Evidence

## Method and conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP via eval `pressure_completion` for baseline and post-text runs.
- **Completion selector:** `default` for both runs.
- **Resolved model:** `openai-codex/gpt-5.6-sol` for both runs.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** each run was a stateless no-tool completion with no repository access or prior conversation.
- **Pair comparability:** system prompt, user prompt, selector, resolved model, and harness settings were identical. Appending the complete final check text was the only post-text change.
- **Baseline evidence:** exact output from supplied baseline `check-silent-fallback` in `local://routed-engineering-baselines.json`.
- **Post-text construction:** the exact baseline system prompt, then `\n\nApply the following routed engineering check as the authoritative review procedure for this response:\n\n`, then the complete final check text. The user prompt was unchanged.
- **Evaluated final file:** `skills/pragmatic-catalog/checks/failure-recovery/no-silent-fallback.md`, SHA-256 `46f76fbc7c1a1f612f908e194c82e663386d76945df08d6c071975e3e8dbacc6`.
- **Scoring:** manual criterion-by-criterion binary scoring against the exact outputs. The check was `active`; the isolated evaluation completion was not emitted to a developer workflow.

## Prompt

### System

```text
Review the code scenario. Report only concrete defects with evidence; stay silent when safe.
```

### User

```text
A pricing client catches every provider error and returns [] with no log or signal, so callers treat failure as no prices. A comparison client uses an approved stale-cache policy, emits a dependency-failure event with the original cause, marks the result stale, and fails when no cache exists. Review both.
```

## Binary criteria

1. Names stable check ID `failure-recovery/no-silent-fallback`.
2. Cites the dependency failure path and its conversion to `[]`.
3. Demonstrates the unapproved, normal-looking fallback and hidden failure.
4. States the contextual impact.
5. States evidence confidence.
6. States contextual severity.
7. Gives a bounded fix direction.
8. Produces no finding for the approved stale fallback with diagnostics, a stale marker, and explicit failure when no cache exists.

## Summary

| Condition | Score | Result |
|---|---:|---|
| Baseline | 5/8 | Detected only the bad client, but omitted the stable ID, confidence, and severity. |
| Post-text | 8/8 | Reported the complete evidence contract only for the bad client. |

## Baseline exact output

```text
- **Pricing client:** Defect—catching all provider errors and returning `[]` converts dependency failures into a valid “no prices” response. Callers cannot distinguish outages from genuine empty results, and the lack of logging or signaling removes observability. Propagate the error or return an explicit failure result and emit the original cause.
```

## Baseline scoring

1. **FAIL** — It named the client, not the stable check ID.
2. **PASS** — It cited caught provider errors converted to `[]`.
3. **PASS** — It showed callers cannot distinguish outage from an empty result.
4. **PASS** — It stated that callers can mistake failure for no prices.
5. **FAIL** — It stated no confidence.
6. **FAIL** — It stated no severity.
7. **PASS** — It bounded the correction to propagation or an explicit failure result with diagnostics.
8. **PASS** — It made no finding against the comparison client.

**Total: 5/8**

## Post-text exact output

```text
### `failure-recovery/no-silent-fallback` — Pricing client

- **Path:** Provider request error → broad catch → returns `[]`.
- **Failure:** Any provider failure is converted into an empty price collection.
- **Evidence:** No approved fallback policy, log, freshness marker, error status, or preserved cause is present. Callers interpret `[]` as a successful “no prices” response and cannot distinguish dependency failure from a legitimate empty result.
- **Impact:** Provider outages or errors silently appear as absent prices, potentially causing incorrect pricing decisions and preventing detection or recovery.
- **Severity:** High
- **Confidence:** High
- **Fix direction:** Propagate the provider error or return a distinguishable failure result. If fallback is required, use an approved policy that preserves the original cause in diagnostics, marks degraded/stale data explicitly, and fails when no usable fallback exists.
```

## Post-text scoring

1. **PASS** — The heading names the stable check ID.
2. **PASS** — Evidence identifies the provider failure catch and replacement `[]`.
3. **PASS** — Evidence proves there is no approved policy and callers observe apparent success.
4. **PASS** — Impact identifies incorrect no-price decisions and hidden outage blast radius.
5. **PASS** — Confidence is high.
6. **PASS** — Severity is high.
7. **PASS** — Fix direction is bounded to explicit failure or an approved marked fallback.
8. **PASS** — It makes no finding against the approved stale-cache client.

**Total: 8/8**
