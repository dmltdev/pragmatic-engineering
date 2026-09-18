# No Unobserved Promise Pressure Evidence

## Methodology and matched run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP via the `pressure_completion` eval tool.
- **Completion selection:** selector `default`; resolved provider/model `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** each completion had no tools, repository access, or prior conversation.
- **Pair comparability:** the prompt and baseline system prompt are identical. The post-check system prompt adds only the delimiter below and the complete final check text.
- **Evaluated final check:** `skills/pragmatic-catalog/checks/concurrency-async/no-unobserved-promise.md`, SHA-256 `6db75a4a203592b84a9cb469b99b2f9403bbe9a927b95574b5fc4b6a6a4cc752`.
- **Scoring:** manual criterion-by-criterion binary scoring against the exact outputs. No model judge or partial credit was used.

### Baseline system prompt

```text
You are a senior engineering assistant. Review the code scenario. Report only concrete defects with evidence; stay silent when safe.
```

### Post-check system prompt rule

The post-check system prompt was the exact baseline system prompt followed by this delimiter and then the complete final check text:

```text

Apply the following check as authoritative engineering guidance for this response:

```

Exact construction: `baseSystem + "\n\nApply the following check as authoritative engineering guidance for this response:\n\n" + checkText`.

## Evaluation summary

| Condition | Score | Result |
|---|---:|---|
| Baseline | 5/7 | Correctly found the unowned work and stayed silent on supervised work, but omitted the stable check ID, confidence, and severity. |
| Post-check | 7/7 | Reported every required finding field and explicitly recognized lifecycle-managed ownership as safe. |

## Case — request-scoped work versus supervised work

### Prompt

```text
TypeScript: an HTTP handler calls saveAudit(event) without await, return, catch, or task ownership, then sends 204. saveAudit can reject. A worker intentionally uses void supervisor.spawn(() => saveAudit(event)); supervisor records failure and drains tasks before shutdown. Review both.
```

### Binary criteria

1. Names stable check ID `concurrency-async/no-unobserved-promise`.
2. Cites that `saveAudit(event)` can reject and has no await, return, handler, or task owner.
3. Demonstrates that the work can outlive the request or reject without observation.
4. States a contextual impact.
5. States confidence and severity.
6. Gives a bounded correction direction.
7. Produces no finding for the supervisor-owned task that records failure and drains before shutdown.

### Baseline exact output

```text
- **HTTP handler:** `saveAudit(event)` creates an unowned promise whose rejection is never observed, causing an unhandled rejection. Sending `204` also allows the request/runtime to finish before the audit save completes, so the audit may be lost during teardown or process termination. Await/return it, catch the rejection, or register it with a lifecycle-managed task supervisor.
```

### Baseline scoring

1. **FAIL** — No stable check ID appears.
2. **PASS** — It identifies the unowned promise and the absence of observation.
3. **PASS** — It states that the request/runtime can finish before the audit save.
4. **PASS** — It names unhandled rejection and lost audit data as consequences.
5. **FAIL** — It states neither confidence nor severity.
6. **PASS** — It bounds the choices to await/return, catch, or lifecycle-managed supervision.
7. **PASS** — It raises no finding against the safe supervised worker.

**Total: 5/7**

### Post-check exact output

```text
- **HTTP handler — `concurrency-async/no-unobserved-promise`**
  - **Defect:** `saveAudit(event)` can reject, but its promise is discarded before the handler sends `204`. No caller or lifecycle owner observes completion or failure, and the work may outlive the request lifecycle.
  - **Impact:** Unhandled rejection, silent audit loss, or process instability; the response can report success before the audit is persisted.
  - **Severity:** High
  - **Confidence:** High
  - **Fix:** Await or return `saveAudit(event)` before responding, explicitly handle rejection if best-effort behavior is intended, or register it with a lifecycle-managed task supervisor that records failures and drains tasks.

The worker is safe: `supervisor.spawn` explicitly owns the task, records failures, and drains outstanding work before shutdown; using `void` on that owned operation does not create an unobserved `saveAudit` promise.
```

### Post-check scoring

1. **PASS** — It names `concurrency-async/no-unobserved-promise`.
2. **PASS** — It cites the rejecting promise and all missing observation or ownership paths.
3. **PASS** — It states that the work can outlive the request and leave rejection unobserved.
4. **PASS** — It names silent audit failure, unhandled rejection, and premature success response.
5. **PASS** — It states high confidence and high severity.
6. **PASS** — It bounds the fix to observation or lifecycle-managed ownership.
7. **PASS** — It explicitly says the supervisor-owned worker is safe.

**Total: 7/7**
