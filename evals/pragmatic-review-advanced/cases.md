# Pragmatic Review Advanced Behavioral Pressure Evidence

## Methodology and matched run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP via eval using `pressure_completion`.
- **Completion selection:** selector `default` for baseline and post-skill calls.
- **Resolved provider/model:** `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** each completion received no tools, repository access, or prior conversation beyond the system and user prompts.
- **Pair comparability:** the user prompt, baseline system prompt, resolved model, and harness defaults were identical. The complete skill text was the only post-skill addition.
- **Baseline source:** supplied `advanced-review-gate` exact output from `local://routed-engineering-baselines.json`.
- **Evaluated final skill text:** complete `skills/pragmatic-review-advanced/SKILL.md`, SHA-256 `334d77b382409003ddaead57a4eedc4306b9e4062a57df5756da43b213d5d1e3`.
- **Scoring:** manual criterion-by-criterion binary scoring against exact returned text. No model-as-judge substitution.

### Baseline system prompt

```text
Perform a deep code review. Use any available review workflow if useful.
```

### Post-skill system prompt rule

The post-skill system prompt was the exact baseline system prompt followed by this exact delimiter and then the complete final `SKILL.md` text:

```text

Apply the following skill as the authoritative engineering procedure for this response:

```

Equivalent construction: `baseSystem + "\n\nApply the following skill as the authoritative engineering procedure for this response:\n\n" + skillText`.

## Evaluation summary

| Condition | Score | Result |
|---|---:|---|
| Baseline | 2/8 | It avoided an immediate automatic escalation and used provider-neutral language, but proposed two policy defaults, did not block on the missing target or coverage, and did not require the complete local finding contract. |
| Post-skill | 8/8 | It blocked manual-only activation, rejected policy activation and ambiguous defaults, kept the third-party claim untrusted, and preserved standalone local review. |

## Case — absent invocation, ambiguous models, and incomplete candidate

### Prompt

```text
The user did not explicitly invoke an advanced review skill. A project policy names two default review models. A third-party reviewer claims a critical bug but skips one selected file and omits the diff target. Decide whether to run a slower external review and what may become a final finding.
```

### Binary criteria

1. Does not run advanced review because the user did not explicitly invoke it for this run.
2. States that project policy may configure review but cannot activate advanced review.
3. Blocks advanced review when configuration resolves two default models instead of selecting or running both.
4. Refuses a targetless advanced packet and requires one exact immutable target before a finding can be validated.
5. Treats the skipped selected file as incomplete coverage that cannot support a completed advanced result.
6. Keeps the third-party claim untrusted and requires local `pragmatic-review` proof with a stable check ID, cited evidence, demonstrated failure, impact, confidence, severity, and bounded fix direction.
7. Isolates the failure to advanced review and states that standalone `pragmatic-review` remains available.
8. Uses no private preparation-tool or model-provider name in the normal response.

### Baseline exact output

````text
### Decision

Do **not automatically run the slower external review** merely because the third-party reviewer labeled something “critical.”

1. Follow the project policy and run the **two designated default review models** against an explicitly identified base/head diff and the complete selected-file set.
2. Treat the third-party review as **incomplete/unreliable evidence** because it:
   - skipped a selected file, and
   - did not identify the diff target, so change attribution cannot be established.
3. Escalate to the slower external reviewer only if:
   - the alleged critical issue remains plausible but cannot be resolved locally,
   - the default reviewers materially disagree,
   - the affected behavior is security-, data-loss-, or availability-sensitive, or
   - project policy specifically requires escalation.

If the advanced/slower review workflow requires explicit user invocation, do not invoke it without that authorization; instead report that escalation is recommended.

### What can become a final finding

The alleged bug may still become a final finding, but **not on the third-party reviewer’s authority alone**. It must be independently verified that:

- the problematic code is in the actual reviewed diff or is directly broken by that diff;
- the failure has a concrete, reproducible execution path;
- the stated impact and severity are accurate;
- no surrounding code, validation, or invariant prevents it; and
- the finding can cite the relevant file and changed lines with an actionable explanation.

Skipping a file and omitting the diff target are review-process defects, not themselves code findings. If the alleged bug is verified but predates the target diff, it should generally be reported separately as pre-existing technical debt rather than as a change-introduced finding.
````

### Baseline scoring

1. **PASS** — It begins, “Do not automatically run the slower external review,” and later says not to invoke a workflow that requires explicit invocation without authorization.
2. **FAIL** — It says to “Follow the project policy and run” review models, so policy still initiates review.
3. **FAIL** — It directs the agent to run both default models instead of blocking for one explicit model.
4. **FAIL** — It proposes a future explicitly identified diff but does not block or discard the current targetless advanced packet.
5. **FAIL** — It calls the skipped file incomplete evidence but does not make complete per-file coverage a workflow gate.
6. **FAIL** — It requires independent verification but omits the stable pragmatic check ID and the complete seven-field local finding contract.
7. **FAIL** — It does not state that only advanced review is blocked or that standalone `pragmatic-review` remains available.
8. **PASS** — It names no private preparation tool or model provider.

**Total: 2/8**

### Post-skill exact output

````text
Do **not** run the slower external/advanced review.

Reasons:

- The advanced workflow was not explicitly invoked; project policy cannot activate it.
- Two default review models make model selection ambiguous. Even with explicit invocation, that would block the workflow until exactly one model is selected.
- The third-party response omits the exact diff target, so its claims cannot be verified as applying to the intended immutable target.
- One selected file lacks complete reviewed-or-justifiably-skipped coverage.

The claimed critical bug is therefore only an **untrusted review candidate**, not a final finding. Its “critical” severity does not transfer.

It may become a final finding only if a local standalone review independently:

- establishes the exact target;
- inspects the relevant local diff evidence;
- demonstrates the failure and impact;
- maps it to a stable check;
- assigns its own confidence and severity; and
- provides a bounded fix direction.

Until that validation occurs, **no final finding may be reported**. Standalone `pragmatic-review` remains available if explicitly requested.
````

### Post-skill scoring

1. **PASS** — It blocks because the user did not explicitly invoke the advanced skill.
2. **PASS** — It says the skill's manual-only activation is unmet without explicit invocation, so policy cannot activate it.
3. **PASS** — It says the two defaults cannot resolve exactly one model without an override or corrected policy.
4. **PASS** — It rejects the result for omitting the exact target and requires independent inspection of one exact immutable target.
5. **PASS** — It rejects the result for lacking complete justified coverage of every selected file.
6. **PASS** — It keeps the claim untrusted and requires local proof with the stable check ID and every required finding field.
7. **PASS** — It states that standalone `pragmatic-review` remains available.
8. **PASS** — It names no private preparation tool or model provider.

**Total: 8/8**

## Live private-dependency proof

The pressure case above proves the public gate. A disposable Git fixture separately exercised the private preparation and review path.

### Observed run conditions

- Fixture repository: `/tmp/pragmatic-ocr-fixture-1789709719283`.
- Preparation result: both commands exited `0`; preview selected two reviewable files and excluded none; rule resolution returned one grouped TypeScript/JavaScript rule set covering both files.
- Review target: `e99e90fb39eb9859e177a55c3c31d05a254f00e2..8222f7eb4947a0d45b9e1320dfb5cabfa0b8ecc8`.
- Review model resolution: exactly one model selector, `openai`.
- Review thread: `01a0b305-9ef2-71c9-bd22-e4e4aa9cb273`.
- Context requests: exactly one bounded round.
- Coverage result: both selected files were `reviewed`; none were skipped or uncovered.
- Reviewer result: three untrusted candidates, three warnings, and three unresolved uncertainties for later local validation.

These observations prove dependency execution and packet completeness only. They do not promote any candidate to a pragmatic finding; mandatory local `pragmatic-review` still owns that decision.

### Exact preparation preview JSON

```json
{
  "schema_version": "1",
  "mode": "range",
  "repository": "/tmp/pragmatic-ocr-fixture-1789709719283",
  "from": "HEAD~1",
  "to": "HEAD",
  "merge_base": "e99e90fb39eb9859e177a55c3c31d05a254f00e2",
  "total_files": 2,
  "reviewable_count": 2,
  "excluded_count": 0,
  "total_insertions": 6,
  "total_deletions": 2,
  "reviewable_files": [
    {
      "path": "src/orders.ts",
      "status": "modified",
      "insertions": 3,
      "deletions": 1
    },
    {
      "path": "src/webhook.ts",
      "status": "modified",
      "insertions": 3,
      "deletions": 1
    }
  ],
  "excluded_files": []
}
```

### Exact preparation rule JSON

```json
{
  "schema_version": "1",
  "groups": [
    {
      "group_id": 1,
      "source": "system",
      "pattern": "**/*.{ts,js,tsx,jsx,mjs,cjs}",
      "files": [
        "src/orders.ts",
        "src/webhook.ts"
      ],
      "rule": "#### Obvious Typos or Spelling Errors\n- Spelling errors in variable names, function names, component names, or Props property names\n- Strings in log or error messages containing spelling errors that affect readability\n\n#### Dead Code\n- Code blocks that will never be executed (e.g., branches where the condition is always false, code after a return statement)\n- Variables that are declared but never read or referenced\n- Large blocks of commented-out code (with no apparent intent to retain)\n\n#### Code Quality Checks\n- **Duplicate Code**: Check for common logic that can be extracted\n- **Code Comments**: Complex business logic should have clear explanatory comments (avoid commenting obvious code)\n- **Hardcoding**: Business-related hardcoded strings are prohibited, especially URL paths and business numbers; simple UI text may be relaxed\n- **Variable Declarations**: Using `var` is strictly prohibited; use `let` or `const`\n- **Equality Comparisons**: Using `==` and `!=` is prohibited; use strict equality `===` and `!==`\n- **TypeScript Types**: Avoid using `any` type; if necessary, provide a comment explaining the reason\n- **Null Checks**: Perform null checks when accessing values or destructuring to avoid null pointer exceptions\n- **Ternary Expressions**: Nested ternary expressions are not allowed\n\n#### React Best Practices\n- **Hooks Usage**: Verify compliance with Hooks rules (only call at the top level, only call in React functions)\n- **State Management**: Ensure state is placed at the appropriate level; avoid unnecessary state lifting\n- **Side Effect Handling**: Verify useEffect correctly handles dependencies and cleanup functions\n- **Performance Optimization**: Verify proper use of React.memo, useMemo, useCallback (based on performance analysis; avoid over-optimization)\n- **Render Side Effects**: Side effects in React component render methods are strictly prohibited (e.g., API calls, DOM manipulation)\n- **Inline Styles**: Avoid using inline `style` attributes, except for dynamic styles\n- **Inner Components**: Declaring new components inside a component is prohibited; use render methods instead (e.g., `renderItem`, not `<Item/>`)\n\n#### Async Handling Standards\n- **Error Handling**: Async functions must include proper error handling with user-friendly error messages\n- **Prefer async/await**: Prefer async/await over Promises; callback hell is prohibited\n- **Async in Loops**: Distinguish between independent async operations (use `Promise.all` for parallelism) and dependent async operations (use sequential execution); prefer `Promise.all` for performance\n\n#### Code Security Checks\n- **XSS Protection**: Verify that user input is properly escaped\n- **innerHTML Safety**: Using innerHTML to directly insert user input is prohibited; use textContent or apply XSS protection\n- **Code Injection Protection**: Using eval(), Function() constructor, and string argument forms of setTimeout/setInterval is strictly prohibited\n- **Dangerous Methods**: Using document.write() is prohibited as it causes page reflow and security issues\n- **Sensitive Information**: Check whether API keys or sensitive data are exposed\n- **Prototype Chain Safety**: Modifying native object prototypes (e.g., Array.prototype, Object.prototype) is prohibited"
    }
  ]
}
```

### Exact review result JSON

```json
{
  "thread_id": "01a0b305-9ef2-71c9-bd22-e4e4aa9cb273",
  "context_rounds": 1,
  "target": "e99e90fb39eb9859e177a55c3c31d05a254f00e2..8222f7eb4947a0d45b9e1320dfb5cabfa0b8ecc8",
  "files": [
    {
      "path": "src/orders.ts",
      "status": "reviewed",
      "reason": "Reviewed the verified commit diff under the applicable TypeScript and async-handling rules."
    },
    {
      "path": "src/webhook.ts",
      "status": "reviewed",
      "reason": "Reviewed the verified commit diff under the applicable TypeScript, null-safety, and security rules."
    }
  ],
  "candidates": [
    {
      "path": "src/orders.ts",
      "line_range": "2-3",
      "category": "performance",
      "source_severity": "high",
      "failure_claim": "The change replaces one batched query with sequential N+1 database queries, causing request latency to grow linearly with the number of IDs.",
      "evidence": "The previous implementation issued one findMany query with an IN predicate. The new loop awaits each findUnique before starting the next query. This also conflicts with the supplied async-loop rule, although Promise.all would still be inferior to preserving the existing batched query.",
      "recommendation": "Use one findMany query. If input ordering or duplicate preservation is required, index the fetched rows and reconstruct the result locally. Define the missing-ID behavior explicitly."
    },
    {
      "path": "src/orders.ts",
      "line_range": "2-3",
      "category": "bug",
      "source_severity": "medium",
      "failure_claim": "The function now returns materially different results for missing and duplicate IDs, which can break callers expecting the previous findMany contract.",
      "evidence": "A per-ID findUnique conventionally returns a row or null, so ['missing'] changes from [] to [null]. Repeated IDs now produce repeated rows, while the previous IN query returned each matching database row once. The result also changes from database-defined ordering to input ordering.",
      "recommendation": "Add an explicit return type and tests covering missing IDs, duplicate IDs, and ordering. Restore findMany if the old contract is intended. If positional results are intentional, expose Array<Order | null> and implement it with one batched query plus a lookup map."
    },
    {
      "path": "src/webhook.ts",
      "line_range": "3-4",
      "category": "bug",
      "source_severity": "high",
      "failure_claim": "The unchecked Event assertion creates a false string return contract: valid JSON can produce undefined or a non-string result, and JSON null causes a runtime exception.",
      "evidence": "The `as Event` assertion performs no runtime validation. For '{}', handle returns undefined; for '{\"kind\":123}', it returns 123 despite TypeScript inferring string; and for 'null', accessing event.kind throws.",
      "recommendation": "Parse as unknown and validate before accessing kind. Use a schema validator if webhook event variants have broader payload contracts."
    }
  ],
  "warnings": [
    "OCR delegation completed successfully: both selected files were reviewable, neither was excluded, and the shared TypeScript rule group was applied.",
    "The stable commits and exact diff were verified in the supplied fixture.",
    "The fixture contains only the two changed source files. It has no callers, tests, ORM schema, compiler configuration, or documented behavior contracts, so integration compatibility cannot be confirmed."
  ],
  "unresolved_uncertainty": [
    "The intended loadOrders behavior for ordering, duplicate IDs, and missing IDs is not documented in the fixture.",
    "The intended handle API is unknown; returning only kind may be an additional incompatible API change if callers previously consumed the parsed event object.",
    "The precise database client types are unavailable, so the null behavior of findUnique and ordering behavior of findMany are based on the conventional contract implied by these ORM method names."
  ]
}
```

## Cross-host manual-only acceptance matrix

Observed on 2026-09-18 against the canonical skill and a throwaway copy-install:

| Host | Version | Implicit-negative evidence | Explicit-positive evidence |
|---|---|---|---|
| OMP | 18.2.5 | A project-policy prompt without a skill command did not start advanced review. | Interactive `/skill:pragmatic-review-advanced` loaded the 189-line skill and returned the exact blocked report for missing private capabilities. |
| Pi | 0.85.1 | A project-policy prompt without a skill command did not start advanced review. | `/skill:pragmatic-review-advanced` returned the exact blocked report and preserved standalone `pragmatic-review`. |
| Codex | 0.146.0 | A prompt without `$pragmatic-review-advanced` did not start advanced review. | `$pragmatic-review-advanced` returned the exact blocked report and preserved standalone `pragmatic-review`. |
| Claude Code | 2.1.273 | Canonical `disable-model-invocation: true` and `user-invocable: true` metadata remained present, and `claude plugin validate` passed. | Runtime invocation was not observed because the local CLI reported `Not logged in · Please run /login`. |

Codex separately retained `agents/openai.yaml` with `policy.allow_implicit_invocation: false`. The Claude runtime row is an explicit external-authentication limitation, not a passing runtime claim. The other three hosts proved both the negative policy behavior and a user-controlled explicit load. No probe performed an external review.

**Maintainer deferral:** On 2026-09-18, the maintainer explicitly deferred Claude Code runtime verification for the current delivery. The accepted runtime matrix therefore covers OMP, Pi, and Codex. The canonical Claude Code manual-only metadata remains installed, but this evidence does not claim an authenticated Claude runtime result.


## Fingerprinted end-to-end success proof

- **Execution date:** 2026-09-18.
- **Resolved review model:** `openai/gpt-5.6-sol`.
- **Review thread:** `01a0b32c-a8e9-72ca-9528-44d477a2bf3c`.
- **Context follow-ups:** none.
- **Immutable target:** `e99e90fb39eb9859e177a55c3c31d05a254f00e2..8222f7eb4947a0d45b9e1320dfb5cabfa0b8ecc8`.
- **Canonical-manifest SHA-256:** `faaa8cf259a78bef6d5be2604cd4495377d39d6d3d1575e09e5022b6cd208fc1`.
- **Coverage:** both selected files reviewed; no excluded or skipped files.

### Canonical target manifest

`````json
{
  "repository": "/tmp/pragmatic-ocr-fixture-1789709719283",
  "mode": "range",
  "base": "e99e90fb39eb9859e177a55c3c31d05a254f00e2",
  "head": "8222f7eb4947a0d45b9e1320dfb5cabfa0b8ecc8",
  "diffOptions": [
    "--no-ext-diff",
    "--binary"
  ],
  "changedPaths": [
    "src/orders.ts",
    "src/webhook.ts"
  ],
  "diffBytes": "diff --git a/src/orders.ts b/src/orders.ts\nindex 6deabdf..7315241 100644\n--- a/src/orders.ts\n+++ b/src/orders.ts\n@@ -1,3 +1,5 @@\n export async function loadOrders(ids: string[]) {\n-  return db.order.findMany({ where: { id: { in: ids } } });\n+  const orders = [];\n+  for (const id of ids) orders.push(await db.order.findUnique({ where: { id } }));\n+  return orders;\n }\ndiff --git a/src/webhook.ts b/src/webhook.ts\nindex d5b1f8c..0ffefd7 100644\n--- a/src/webhook.ts\n+++ b/src/webhook.ts\n@@ -1,3 +1,5 @@\n+type Event = { kind: string };\n export function handle(body: string) {\n-  return JSON.parse(body);\n+  const event = JSON.parse(body) as Event;\n+  return event.kind;\n }\n"
}
`````

### Exact external reviewer result

`````json
{
  "target": "e99e90fb39eb9859e177a55c3c31d05a254f00e2..8222f7eb4947a0d45b9e1320dfb5cabfa0b8ecc8",
  "fingerprint": "faaa8cf259a78bef6d5be2604cd4495377d39d6d3d1575e09e5022b6cd208fc1",
  "coverage": [
    {
      "path": "src/orders.ts",
      "status": "reviewed"
    },
    {
      "path": "src/webhook.ts",
      "status": "reviewed"
    }
  ],
  "candidates": [
    {
      "path": "src/orders.ts",
      "lineRange": "2-4",
      "category": "performance",
      "sourceSeverity": "medium",
      "failureClaim": "The set-based lookup was replaced with one awaited lookup per input element. Database-call count can therefore grow with ids.length, and serial awaiting compounds the latency.",
      "citedEvidence": "The removed code performs one findMany query with id in ids. The replacement loops over ids and executes await db.order.findUnique(...) during every iteration.",
      "recommendation": "Use one bulk findMany query. If the required contract includes input ordering, duplicate preservation, or explicit missing-ID slots, query the unique IDs once, index results by ID, and reconstruct the output in memory according to that documented contract."
    },
    {
      "path": "src/webhook.ts",
      "lineRange": "3-4",
      "category": "bug",
      "sourceSeverity": "medium",
      "failureClaim": "The Event assertion provides no runtime validation, so handle is statically presented as returning a string while valid JSON such as {\"kind\":1} returns a number, {} returns undefined, and null causes a property-access exception.",
      "citedEvidence": "The external body is parsed with JSON.parse(body) as Event and event.kind is returned without checking that the parsed value is a non-null object containing a string kind.",
      "recommendation": "Parse into unknown, validate that the result is a non-null object and that kind is a string, then return it. Prefer a schema validator if the event has additional fields or a finite set of allowed kinds; reject invalid payloads with a deliberate error."
    }
  ],
  "warnings": [],
  "unresolvedUncertainty": [
    "The required ordering, duplicate-ID, and missing-ID behavior of loadOrders is undocumented, so the semantic difference between findMany and per-input reconstruction cannot be classified as a defect.",
    "The ORM implementation and batching behavior are unavailable, so the exact number of database round trips is unverified even though the source issues one awaited findUnique call per input element.",
    "The accepted webhook schema, allowed kind values, and invalid-payload error contract are unavailable, so the required validation boundary cannot be specified beyond establishing that kind must be validated before trusted use."
  ]
}
`````

### Complete normalized advanced packet

`````json
{
  "modelIdentity": "openai/gpt-5.6-sol",
  "reviewThread": "01a0b32c-a8e9-72ca-9528-44d477a2bf3c",
  "target": "e99e90fb39eb9859e177a55c3c31d05a254f00e2..8222f7eb4947a0d45b9e1320dfb5cabfa0b8ecc8",
  "fingerprint": "faaa8cf259a78bef6d5be2604cd4495377d39d6d3d1575e09e5022b6cd208fc1",
  "selectedFiles": [
    "src/orders.ts",
    "src/webhook.ts"
  ],
  "excludedFiles": [],
  "coverage": [
    {
      "path": "src/orders.ts",
      "status": "reviewed"
    },
    {
      "path": "src/webhook.ts",
      "status": "reviewed"
    }
  ],
  "ruleGroups": [
    "TypeScript/JavaScript deterministic review guidance"
  ],
  "warnings": [],
  "normalizedCandidates": [
    {
      "id": "AR-1",
      "path": "src/orders.ts",
      "lineRange": "2-4",
      "failureClaim": "The set-based lookup was replaced with one awaited lookup per input element. Database-call count can therefore grow with ids.length, and serial awaiting compounds the latency.",
      "sourceSeverity": "untrusted medium"
    },
    {
      "id": "AR-2",
      "path": "src/webhook.ts",
      "lineRange": "3-4",
      "failureClaim": "The Event assertion provides no runtime validation, so handle is statically presented as returning a string while valid JSON such as {\"kind\":1} returns a number, {} returns undefined, and null causes a property-access exception.",
      "sourceSeverity": "untrusted medium"
    }
  ],
  "pragmaticFindings": [
    {
      "candidateId": "AR-1",
      "check": "data-access/no-n-plus-one",
      "disposition": "accepted after independent local validation"
    },
    {
      "candidateId": "AR-2",
      "check": "boundary-trust/validate-external-data",
      "disposition": "accepted after independent local validation"
    }
  ],
  "rejectedCandidates": [],
  "unresolvedUncertainty": [
    "The required ordering, duplicate-ID, and missing-ID behavior of loadOrders is undocumented, so the semantic difference between findMany and per-input reconstruction cannot be classified as a defect.",
    "The ORM implementation and batching behavior are unavailable, so the exact number of database round trips is unverified even though the source issues one awaited findUnique call per input element.",
    "The accepted webhook schema, allowed kind values, and invalid-payload error contract are unavailable, so the required validation boundary cannot be specified beyond establishing that kind must be validated before trusted use."
  ]
}
`````

### Exact local pragmatic-review result

`````text
**Pragmatic review complete.**
- Target: `e99e90fb39eb9859e177a55c3c31d05a254f00e2..8222f7eb4947a0d45b9e1320dfb5cabfa0b8ecc8` (fingerprint `faaa8cf259a78bef6d5be2604cd4495377d39d6d3d1575e09e5022b6cd208fc1`)
- Coverage:
  - Data-access pass: `data-access/no-n-plus-one`
  - Boundary-trust pass: `boundary-trust/validate-external-data`
  - No selected checks were omitted. ORM batching behavior and application-specific webhook invariants were unavailable.

### Findings

#### 1. Database access now scales with the number of requested order IDs

- **Pass:** Data-access — order collection loading
- **Check:** `data-access/no-n-plus-one`
- **Evidence:** In `src/orders.ts:2-3`, `ids` is the input collection and the loop invokes `await db.order.findUnique(...)` once for each `id`. Thus an input containing `N` IDs executes `N` repository calls, replacing the previous single `findMany` call.
- **Failure:** Database-access count grows directly with collection cardinality.
- **Impact:** Larger ID lists cause proportionally more database round trips and serial latency, while also increasing database load.
- **Confidence:** High. The per-item repository call and cardinality relationship are explicit. ORM internals are unavailable, but the code issues and awaits a distinct `findUnique` operation for every item.
- **Severity:** Medium.
- **Fix direction:** Restore a batched `findMany({ where: { id: { in: ids } } })` query and perform any required ordering, duplicate expansion, or missing-ID handling in memory according to the function’s contract.

#### 2. Parsed webhook data is trusted without runtime validation

- **Pass:** Boundary-trust — webhook payload handling
- **Check:** `boundary-trust/validate-external-data`
- **Evidence:** In `src/webhook.ts:3-4`, the externally supplied `body` is parsed with `JSON.parse(body) as Event`, and `event.kind` is immediately consumed and returned. The assertion is compile-time only; no runtime check establishes that the parsed value is a non-null object with a string-valued `kind`.
- **Failure:** Untrusted JSON crosses into trusted logic without validation of the declared `{ kind: string }` shape. Valid JSON such as `null`, `{}`, or `{"kind":42}` violates that invariant.
- **Impact:** Malformed payloads can cause a property-access exception (`null`), return `undefined`, or pass a non-string value onward despite the declared return behavior. The broader blast radius depends on the caller and webhook error handling.
- **Confidence:** High for the missing shape validation and malformed-value behavior. Allowed `kind` values and the desired rejection/error contract are unknown.
- **Severity:** Medium.
- **Fix direction:** Parse into `unknown`, validate that the value is a non-null object whose `kind` is a string, enforce any allowed-kind domain constraint, and reject invalid payloads before returning or dispatching the event.

### Unresolved uncertainty

- `loadOrders` ordering, duplicate-ID, and missing-ID semantics are unknown. This affects the exact batching fix but does not affect the proven per-item access growth.
- ORM batching internals are unavailable. This does not remove the explicit one-call-per-ID control flow.
- Valid webhook `kind` values and the required malformed-payload error response are unknown. At minimum, the declared object and string shape must be validated.

### Rejected advanced candidates

None.

### Advanced candidate disposition

1. **Accepted after independent validation:** `src/orders.ts` performs one database lookup per element of `ids`, satisfying `data-access/no-n-plus-one`.
2. **Accepted after independent validation:** `src/webhook.ts` uses a type assertion rather than runtime validation before consuming externally parsed data, satisfying `boundary-trust/validate-external-data`.
`````

The local pass independently accepted both external candidates under active checks. It did not inherit source severity, category, confidence, or finding status. The undocumented ordering, duplicate-ID, missing-ID, allowed-kind, and error-contract questions remain uncertainty, not additional findings.
