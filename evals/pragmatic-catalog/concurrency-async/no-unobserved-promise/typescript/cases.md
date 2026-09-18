# TypeScript No Unobserved Promise Variant Pressure Evidence

## Method and matched run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP via eval `pressure_completion` for baseline and post-text runs.
- **Completion selector:** `default` for both runs.
- **Resolved model:** `openai-codex/gpt-5.6-sol` for both runs.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** each run was a stateless no-tool completion with no repository access or prior conversation.
- **Pair comparability:** system prompt, user prompt, selector, resolved model, and harness settings were identical. Appending the complete final core and matching variant text was the only post-text change.
- **Baseline evidence:** exact output from supplied baseline `check-unobserved-promise` in `local://routed-engineering-baselines.json`.
- **Post-text construction:** the exact baseline system prompt, then `\n\nApply the following routed engineering core check and matching TypeScript variant as the authoritative review procedure for this response:\n\n`, then the complete final core text, `\n\n`, and the complete final variant text. The user prompt was unchanged.
- **Evaluated final variant:** `skills/pragmatic-catalog/variants/concurrency-async/no-unobserved-promise/typescript.md`, SHA-256 `d98c3143f182895dd3d176a5fd4c81ef67f0838ff1b5c24d43a3abf7d0c45700`.
- **Inherited final core:** `skills/pragmatic-catalog/checks/concurrency-async/no-unobserved-promise.md`, SHA-256 `6db75a4a203592b84a9cb469b99b2f9403bbe9a927b95574b5fc4b6a6a4cc752`.
- **Scoring:** manual criterion-by-criterion binary scoring against the exact outputs. The parent check was `active`; the isolated evaluation completion was not emitted to a developer workflow.

## Prompt

### System

```text
Review the code scenario. Report only concrete defects with evidence; stay silent when safe.
```

### User

```text
TypeScript: an HTTP handler calls saveAudit(event) without await, return, catch, or task ownership, then sends 204. saveAudit can reject. A worker intentionally uses void supervisor.spawn(() => saveAudit(event)); supervisor records failure and drains tasks before shutdown. Review both.
```

## Binary criteria

1. Names core ID `concurrency-async/no-unobserved-promise` and matching variant `concurrency-async/no-unobserved-promise/typescript`.
2. Cites the promise-producing call and absence of await, return, rejection handling, or task ownership.
3. Demonstrates that the promise can reject or outlive its request owner without observation.
4. States the contextual impact.
5. States evidence confidence.
6. States contextual severity.
7. Gives a bounded TypeScript-appropriate fix direction.
8. Produces no finding for `void supervisor.spawn(...)` when the supervisor records failure and drains owned tasks before shutdown.

## Summary

| Condition | Score | Result |
|---|---:|---|
| Baseline | 5/8 | Detected only the unowned handler promise, but omitted stable IDs, confidence, and severity. |
| Post-text | 8/8 | Preserved the core threshold, named the variant, and left supervised work finding-free. |

## Baseline exact output

```text
- **HTTP handler:** `saveAudit(event)` creates an unowned promise whose rejection is never observed, causing an unhandled rejection. Sending `204` also allows the request/runtime to finish before the audit save completes, so the audit may be lost during teardown or process termination. Await/return it, catch the rejection, or register it with a lifecycle-managed task supervisor.
```

## Baseline scoring

1. **FAIL** — It named the HTTP handler, not the core and variant IDs.
2. **PASS** — It cited the unowned promise and missing observation mechanisms.
3. **PASS** — It proved possible rejection and request-lifecycle escape.
4. **PASS** — It identified unhandled rejection and lost audit impact.
5. **FAIL** — It stated no confidence.
6. **FAIL** — It stated no severity.
7. **PASS** — It bounded the fix to await/return, rejection handling, or task supervision.
8. **PASS** — It made no finding against the owned supervisor case.

**Total: 5/8**

## Post-text exact output

```text
- **Finding:** `concurrency-async/no-unobserved-promise` (`concurrency-async/no-unobserved-promise/typescript`)
  - **Location:** HTTP handler calling `saveAudit(event)` before sending `204`.
  - **Evidence:** `saveAudit(event)` returns a promise that can reject, but the handler neither awaits/returns it nor attaches a rejection handler or registers it with a lifecycle owner. The response completes while the audit operation may still be running.
  - **Impact:** Audit failures are unobserved, and the work may outlive the request or be lost during shutdown.
  - **Severity:** Medium
  - **Confidence:** High
  - **Fix:** Await or return `saveAudit(event)` before completing the response, attach explicit rejection handling if completion is intentionally decoupled, or submit it to the lifecycle-managed `supervisor`.
```

## Post-text scoring

1. **PASS** — The finding names the core and matching TypeScript variant IDs.
2. **PASS** — Evidence cites the promise and all absent observation mechanisms.
3. **PASS** — Evidence proves rejection and request-lifecycle escape.
4. **PASS** — Impact identifies unobserved audit failure and shutdown loss.
5. **PASS** — Confidence is high.
6. **PASS** — Severity is medium.
7. **PASS** — Fix is bounded to direct observation or the existing supervisor.
8. **PASS** — It makes no finding against the owned supervisor case.

**Total: 8/8**
