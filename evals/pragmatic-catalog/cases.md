# Pragmatic Catalog Behavioral Pressure Evidence

## Methodology and matched run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP isolated completion via eval.
- **Completion selection:** selector `default` for baseline and post-skill calls.
- **Resolved provider/model:** `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** each completion received no tools, repository access, or prior conversation beyond its system and user prompts.
- **Pair comparability:** user prompts, baseline system prompts, model, and harness defaults were unchanged. Post-skill calls added the complete catalog skill plus a case-specific installed index and eligible resources.
- **Shipping-index cases:** routing-budget-policy, no-match-silence, and conflict-quarantine used final `skills/pragmatic-catalog/index.json`, SHA-256 `e00879eb38133a791ec1689be9ed5ddfb74a7aaabbf46786a79731c21cec7d4c`.
- **Lifecycle fixture:** shadow-candidate-isolation used schema version 1 with only `concurrency-async/no-unobserved-promise`, changed to `status: shadow`; SHA-256 `68f7aeaa7703d46570c5ffdaf8a25a6d32423912c6bf48fefa170d20294c72e2`. Its body was attached to prove it remained unloaded.
- **Evaluated final skill SHA-256:** `fcf18ecabc5c5ec409079dc014f12419656aea3fb6396784d407f3e7bf83c592`.
- **Scoring:** manual criterion-by-criterion binary scoring against exact returned text. No model-as-judge substitution.

## Case — routing-budget-policy

### System prompt

`````text
You are a senior engineering assistant. Select concise engineering guidance for the change. Return a compact selection with reasons and omissions.
`````

### User prompt

`````text
Phase: coding. Risk inventory: TypeScript service parses an external webhook, starts an async audit write, loops over 200 account IDs and queries each account, and catches provider failure by returning an empty list. Candidate guidance: no-impossible-state, no-unobserved-promise, no-n-plus-one, validate-external-data, no-silent-fallback, bound-pagination. Budget: at most 4 checks. Project policy requires validate-external-data. Two same-authority local rules conflict on whether provider failure may return an empty list. Select the guidance.
`````

### Binary criteria

1. Selects no more than four checks
2. Selects validate-external-data because scoped project policy requires it
3. Quarantines the same-authority fallback conflict
4. Records applicable overflow instead of silently dropping it
5. Does not treat selection as a finding

### Baseline exact output

`````text
### Selected checks

1. **validate-external-data** — Required by project policy; validate and narrow the webhook payload before use.
2. **no-unobserved-promise** — The async audit write must be awaited, returned, or explicitly handled with failure reporting.
3. **no-n-plus-one** — Replace 200 per-account queries with batching, a bulk query, or bounded concurrency.

### Omitted

- **no-silent-fallback** — Two same-authority local rules conflict on returning `[]` after provider failure. Do not choose a side implicitly; flag the conflict for resolution.
- **bound-pagination** — No pagination behavior is described.
- **no-impossible-state** — Less directly relevant than the selected concrete risks.

Use only these three checks; the budget need not be filled.
`````

### Baseline scoring

1. **PASS** — Selects no more than four checks
2. **PASS** — Selects validate-external-data because scoped project policy requires it
3. **PASS** — Quarantines the same-authority fallback conflict
4. **PASS** — Records applicable overflow instead of silently dropping it
5. **PASS** — Does not treat selection as a finding

**Total: 5/5**

### Post-skill exact output

`````text
**Pragmatic catalog selection packet**
- Status: selected
- Phase: coding
- Category pass: none
- Budget: check limit of 4
- Effective limit: 4 checks; no token limit supplied

### Selected checks
#### boundary-trust/validate-external-data
- Variant: none
- Reasons: Required by the scoped project policy; the external webhook is a trust boundary whose parsed payload may reach service logic.

##### Core body
## Failure

Untrusted external data reaches trusted domain, persistence, or command logic without runtime validation at or before the trust crossing.

## Signals

Inspect network payloads, files, environment values, IPC messages, and provider responses. Type assertions and static types are routing signals, not validation.

## Evidence

Show the external origin, the data path into trusted logic, and the absence of prior runtime validation. Also show the expected shape or invariant that malformed data can violate.

## Bad

```ts
const event = JSON.parse(body) as PaymentEvent;
charge(event.amount, event.kind);
```

## Good

```ts
const raw: unknown = JSON.parse(body);
const event = PaymentEventSchema.parse(raw);
charge(event.amount, event.kind);
```

## Allow

Allow internal values that do not cross a trust boundary. Allow external input already validated at an earlier boundary when only the validated value continues. Opaque forwarding without interpretation is not trusted use.

## Fix

Parse into `unknown`, validate the required shape and domain invariants, reject invalid data, and pass only the validated value into trusted logic.

##### Variant body
None.

#### concurrency-async/no-unobserved-promise
- Variant: concurrency-async/no-unobserved-promise/typescript
- Reasons: The TypeScript service starts an asynchronous audit write; this directly matches the task-lifecycle and fire-and-forget inspection triggers. The language scope exactly matches the TypeScript variant.

##### Core body
## Failure

Started asynchronous work can reject or outlive its lifecycle owner without any caller or task owner observing its completion.

## Signals

Inspect discarded promise-like results, `void`, missing await or return, async callbacks, and responses sent after starting work. Syntax alone is not proof.

## Evidence

Before finding, trace the call to work that can reject or outlive its owner, then show that no await, return, rejection handler, or lifecycle-managed task owner observes it. A finding must name `concurrency-async/no-unobserved-promise`, impact, confidence, severity, and a bounded fix direction. Otherwise stay silent.

## Bad

```ts
saveAudit(event);
reply.sendStatus(204);
```

## Good

```ts
void tasks.spawn("audit", () => saveAudit(event));
// tasks records failure and drains before shutdown
```

## Allow

Allow awaited or returned work and explicit task ownership that records failure and keeps or drains work for the required lifetime.

## Fix

Await or return the work, handle its failure, or register it with the existing lifecycle-managed task owner.

##### Variant body
## Failure

In TypeScript, a promise-producing expression or async callback satisfies the core failure only when its work can reject or outlive its lifecycle owner without observation.

## Signals

Inspect discarded `Promise` expressions, `void promise`, async callbacks passed to APIs that ignore returned promises, and handlers that finish before started work. These forms only trigger inspection.

## Evidence

Keep the complete core threshold: trace work that can reject or outlive its owner and prove no await, return, rejection handler, or lifecycle-managed task owner observes it. Also prove the expression returns a promise or the callback consumer ignores its promise. Name the core ID and this matching variant in any finding. `void` alone cannot prove a finding.

## Bad

```ts
items.forEach(async (item) => {
  await save(item);
});
```

## Good

```ts
void supervisor.spawn(() => saveAudit(event));
// supervisor records failures and drains tasks before shutdown
```

## Allow

Owned `void` work is safe when the called supervisor registers it immediately, observes rejection, and retains, drains, or cancels it for the required lifetime. Async callbacks are safe when their consumer awaits returned promises.

## Fix

Await or return the promise, use a promise-aware collection operation, attach rejection handling, or register work with the existing owned task supervisor.

#### data-access/no-n-plus-one
- Variant: none
- Reasons: The service loops over 200 account IDs and queries each account, directly matching query-in-loop and collection-cardinality evidence.

##### Core body
## Failure

A collection path performs database or remote access per item, so the access count grows with collection cardinality.

## Signals

Inspect per-item resolvers, repository calls in iteration, lazy relation loading, and repeated remote reads. Signals only select this check.

## Evidence

Show the collection source, the access inside its item path, and how the number of accesses grows as the collection grows. A loop, recursion, or `await` alone is not evidence.

## Bad

```ts
const orders = await db.order.findMany();
for (const order of orders) {
  order.customer = await db.customer.findById(order.customerId);
}
```

## Good

```ts
const orders = await db.order.findMany();
const ids = [...new Set(orders.map(order => order.customerId))];
const customers = await db.customer.findManyById(ids);
const byId = new Map(customers.map(customer => [customer.id, customer]));
for (const order of orders) order.customer = byId.get(order.customerId);
```

## Allow

Allow iteration over data already in memory. Allow access whose count is fixed and independent of input cardinality. Do not report a batched read followed by in-memory lookup.

## Fix

Batch or prefetch the needed records, then join in memory. If batching is unavailable, bound the access count explicitly and preserve required ordering and failure behavior.

##### Variant body
None.

#### failure-recovery/no-silent-fallback
- Variant: none
- Reasons: Provider failure is caught and converted to an empty list, directly matching catch-default and empty-on-error triggers. The contradictory local authorization rules are quarantined, so they cannot establish whether this fallback is approved.

##### Core body
## Failure

A dependency failure is hidden behind an unapproved fallback that callers observe as a normal result.

## Signals

Inspect broad catches that return empty collections, defaults, cached data, or success states. Also inspect fallback paths that omit freshness markers or original-cause diagnostics. Signals are not proof.

## Evidence

Before reporting, prove all four facts: the dependency can fail on this path; code converts that failure into an outwardly valid result; no applicable contract or approved policy authorizes the substitution; and the caller cannot distinguish it from a normal result. Cite the path, failure, impact, confidence, severity, and bounded fix direction under this stable check ID.

## Bad

```ts
try {
  return await pricing.fetch();
} catch {
  return [];
}
```

## Good

```ts
catch (cause) {
  diagnostics.dependencyFailure(cause);
  if (!cache.hasPrices()) throw cause;
  return { prices: cache.prices(), stale: true };
}
```

## Allow

An approved degraded or stale-data policy is safe when the result is distinguishable, original-cause diagnostics remain available, and an unusable fallback fails explicitly.

## Fix

Propagate an explicit failure, return a distinguishable failure result, or apply an approved marked fallback while preserving diagnostics.

##### Variant body
None.

### Applied policies
- Project policy requiring `boundary-trust/validate-external-data`; scope: external webhook handling; authority: scoped project policy; effect: prioritizes and requires selection of the validation check.

### Quarantined conflicts
- IDs not supplied: the local rule permitting provider failure to return an empty list and the local rule forbidding that fallback; shared authority: same-authority local rules; overlapping scope: provider-failure handling; contradiction: whether an empty-list substitution is approved. Neither rule was applied.

### Budget omissions
None.

### Other omissions
- `state-modeling/no-impossible-state`: suggested by the `no-impossible-state` label, but no independent flags, invalid state combination, unchecked transition, or other current state-modeling evidence was supplied.
- `bound-pagination`: unknown ID; no installed check has this stable ID.
`````

### Post-skill scoring

1. **PASS** — Selects no more than four checks
2. **PASS** — Selects validate-external-data because scoped project policy requires it
3. **PASS** — Quarantines the same-authority fallback conflict
4. **PASS** — Records applicable overflow instead of silently dropping it
5. **PASS** — Does not treat selection as a finding

**Total: 5/5**

**Evaluator note:** Both outputs satisfy the behavioral criteria. The final catalog additionally loads the exact installed active bodies and TypeScript variant under the required packet contract.

## Case — no-match-silence

### System prompt

`````text
You are a senior engineering assistant. Select concise engineering checks for the change and return the selection result.
`````

### User prompt

`````text
Phase: coding. Change: fix spelling in README. Risk inventory: no production behavior, state, async work, data access, boundaries, caching, or failure semantics change. Suggested ID: data-access/no-n-plus-one. Budget: 4. The installed check is active but has no applicable evidence. Return the catalog result.
`````

### Binary criteria

1. Returns no-match
2. Loads no check body
3. Rejects the irrelevant suggested ID
4. Does not pad the result

### Baseline exact output

`````text
{
  "selected": [],
  "skipped": [
    {
      "id": "data-access/no-n-plus-one",
      "reason": "Not applicable: README-only spelling change has no data-access behavior or supporting evidence."
    }
  ]
}
`````

### Baseline scoring

1. **FAIL** — Returns no-match
2. **PASS** — Loads no check body
3. **PASS** — Rejects the irrelevant suggested ID
4. **PASS** — Does not pad the result

**Total: 3/4**

### Post-skill exact output

`````text
**Pragmatic catalog selection packet**
- Status: no-match
- Phase: coding
- Category pass: none
- Budget: 4 checks
- Effective limit: 4 checks; no token limit supplied

### Selected checks
None.

### Applied policies
None.

### Quarantined conflicts
None.

### Budget omissions
None.

### Other omissions
- data-access/no-n-plus-one: not applicable; the README spelling change has no data-access behavior or evidence matching query-in-loop, per-item-resolver, or repeated-remote-read triggers.
`````

### Post-skill scoring

1. **PASS** — Returns no-match
2. **PASS** — Loads no check body
3. **PASS** — Rejects the irrelevant suggested ID
4. **PASS** — Does not pad the result

**Total: 4/4**

**Evaluator note:** The baseline omitted an explicit no-match status; the final catalog returns the complete no-match packet without loading a body.

## Case — conflict-quarantine

### System prompt

`````text
You are a senior engineering assistant. Apply the engineering catalog to the stated policy and risk evidence.
`````

### User prompt

`````text
Phase: design. Risk: provider failure behavior. Two scoped project policies of equal authority and identical scope conflict: policy A requires fail closed; policy B requires return stale data. Candidate check no-silent-fallback otherwise applies. Budget: 3. Return the catalog result.
`````

### Binary criteria

1. Quarantines both same-authority policies
2. Does not resolve by order or preference
3. Does not emit a finding
4. Records the overlapping scope and contradiction

### Baseline exact output

`````text
- **Catalog result:** **BLOCKED — unresolved policy conflict**
- **Reason:** Equal-authority, identical-scope policies require incompatible provider-failure behaviors: fail closed vs. return stale data. `no-silent-fallback` does not resolve this conflict.
- **Required action:** Escalate for explicit policy precedence or reconciliation; do not approve or implement either behavior until resolved.
`````

### Baseline scoring

1. **PASS** — Quarantines both same-authority policies
2. **PASS** — Does not resolve by order or preference
3. **PASS** — Does not emit a finding
4. **PASS** — Records the overlapping scope and contradiction

**Total: 4/4**

### Post-skill exact output

`````text
**Pragmatic catalog selection packet**
- Status: selected
- Phase: design
- Category pass: none
- Budget: 3 checks
- Effective limit: 3 checks; no token limit supplied

### Selected checks
#### failure-recovery/no-silent-fallback
- Variant: none
- Reasons: The provider-failure risk and contemplated stale-data response directly match dependency-failure and stale-on-error signals. The suggested no-silent-fallback check resolves to this active installed check. Selection does not resolve the conflicting policies or determine whether fail-closed or stale-data behavior is authorized.

##### Core body
## Failure

A dependency failure is hidden behind an unapproved fallback that callers observe as a normal result.

## Signals

Inspect broad catches that return empty collections, defaults, cached data, or success states. Also inspect fallback paths that omit freshness markers or original-cause diagnostics. Signals are not proof.

## Evidence

Before reporting, prove all four facts: the dependency can fail on this path; code converts that failure into an outwardly valid result; no applicable contract or approved policy authorizes the substitution; and the caller cannot distinguish it from a normal result. Cite the path, failure, impact, confidence, severity, and bounded fix direction under this stable check ID.

## Bad

```ts
try {
  return await pricing.fetch();
} catch {
  return [];
}
```

## Good

```ts
catch (cause) {
  diagnostics.dependencyFailure(cause);
  if (!cache.hasPrices()) throw cause;
  return { prices: cache.prices(), stale: true };
}
```

## Allow

An approved degraded or stale-data policy is safe when the result is distinguishable, original-cause diagnostics remain available, and an unusable fallback fails explicitly.

## Fix

Propagate an explicit failure, return a distinguishable failure result, or apply an approved marked fallback while preserving diagnostics.

##### Variant body
None.

### Applied policies
- Policy A; provider failure behavior scope; scoped project-policy authority; requires fail-closed behavior but is quarantined because an equal-authority policy contradicts it in the identical scope.
- Policy B; provider failure behavior scope; scoped project-policy authority; requires returning stale data but is quarantined because an equal-authority policy contradicts it in the identical scope.

### Quarantined conflicts
- Policy A and Policy B; shared scoped project-policy authority; identical provider failure behavior scope; Policy A requires fail closed while Policy B requires returning stale data. Neither policy controls selection or resolves the permitted fallback behavior until the conflict is settled.

### Budget omissions
None.

### Other omissions
None.
`````

### Post-skill scoring

1. **PASS** — Quarantines both same-authority policies
2. **PASS** — Does not resolve by order or preference
3. **PASS** — Does not emit a finding
4. **PASS** — Records the overlapping scope and contradiction

**Total: 4/4**

**Evaluator note:** Both outputs refuse to choose between equal-authority rules. The final catalog preserves its required packet shape and records both quarantined policies.

## Case — shadow-candidate-isolation

### System prompt

`````text
You are a senior engineering assistant. Apply the engineering catalog to the supplied lifecycle states.
`````

### User prompt

`````text
Phase: review. Risk: one unobserved promise. The only matching installed check is status shadow. A discovered candidate also matches but is outside the installed catalog. Budget: 5. Return the catalog result.
`````

### Binary criteria

1. Selects neither shadow nor candidate
2. Returns no-match
3. Loads no bodies
4. Explains lifecycle omission
5. Does not create a finding

### Baseline exact output

`````text
{
  "phase": "review",
  "risk": "one unobserved promise",
  "checks": ["status shadow"],
  "budget": 5,
  "excluded": ["matching candidate outside the installed catalog"]
}
`````

### Baseline scoring

1. **PASS** — Selects neither shadow nor candidate
2. **FAIL** — Returns no-match
3. **PASS** — Loads no bodies
4. **FAIL** — Explains lifecycle omission
5. **PASS** — Does not create a finding

**Total: 3/5**

### Post-skill exact output

`````text
**Pragmatic catalog selection packet**
- Status: no-match
- Phase: review
- Category pass: concurrency-async
- Budget: 5 checks
- Effective limit: 5 checks; no token limit supplied

### Selected checks
None.

### Applied policies
None.

### Quarantined conflicts
None.

### Budget omissions
None.

### Other omissions
- concurrency-async/no-unobserved-promise: applicable evidence exists, but installed status is `shadow`; non-active checks cannot be selected or loaded.
- Discovered candidate (ID not supplied): applicable but outside the installed catalog and therefore ineligible for selection.
`````

### Post-skill scoring

1. **PASS** — Selects neither shadow nor candidate
2. **PASS** — Returns no-match
3. **PASS** — Loads no bodies
4. **PASS** — Explains lifecycle omission
5. **PASS** — Does not create a finding

**Total: 5/5**

**Evaluator note:** The baseline did not identify the result as no-match and did not explain the shadow lifecycle omission. The final catalog does both.



## Candidate lifecycle pressure evidence

- **Execution date:** 2026-09-18.
- **Harness/model:** OMP isolated completion, `openai-codex/gpt-5.6-sol`, default settings, no tools.
- **Authoritative source:** `docs/specs/routed-engineering-checks-v2.md`, Candidate lifecycle rules, plus the shipped catalog's candidate and shadow exclusion boundary.
- **Lifecycle-rules system prompt SHA-256:** `983dda6d42d3efc4828360335f7cd42e8ad12579a7e3389d12ce3effc81822c2`.

### Baseline system prompt

`````text
Assess the proposed engineering-rule candidates and state what should happen to each.
`````

### Lifecycle-rules system prompt

`````text
Assess the proposed engineering-rule candidates and state what should happen to each.

Apply these authoritative lifecycle rules:
- discovered -> candidate -> normalized and deduplicated -> conflict-checked -> inline bad/good examples validated -> independent positive and similar negative fixtures passed -> shadow -> maintainer-approved active.
- Candidates are not installed and cannot create findings.
- Reject or quarantine wording that weakens the semantic failure threshold.
- Signals alone are not failures. A loop or void expression alone cannot prove a defect.
- Fixture-valid, conflict-free normalized candidates may move only to shadow.
- Activation requires a separate maintainer decision informed by shadow evidence.
`````

### User prompt

`````text
Raw candidate batch:
1. A: "Every discarded Promise is a defect, including void supervisor.spawn()."
2. B: "An async operation is defective when it can reject or outlive its owner without await, return, rejection handling, or lifecycle ownership."
3. C: duplicate of B from a second source.
4. D: "Any await inside a loop is an N+1 defect."

Candidate B has a concise bad example with an unawaited audit write and a good example with a supervisor that records failure and drains tasks. Its independent positive fixture proves rejection escapes after the request finishes. Its similar negative fixture proves supervisor-owned work is observed and drained. Candidate D's negative fixture has a fixed two-query loop independent of collection size. No candidate is installed. Return the lifecycle disposition without creating an active check or developer-facing finding.
`````

### Binary criteria

1. Rejects candidate A because syntax-only wording weakens the semantic threshold.
2. Normalizes candidate B to the failure condition supported by evidence.
3. Deduplicates candidate C into B.
4. Rejects candidate D because its similar negative fixture disproves the universal rule.
5. Requires concise bad/good examples plus independent positive and negative fixtures.
6. Moves eligible B only to shadow, never active.
7. Creates no installed check or developer-facing finding.

### Baseline exact output

`````text
| Candidate | Lifecycle disposition | Rationale |
|---|---|---|
| **A** | **Reject as overbroad; supersede with B** | Discarding a Promise is not inherently defective when a lifecycle owner, such as `supervisor.spawn()`, observes failures and drains the work. |
| **B** | **Accept as the canonical validated candidate, but keep inactive** | The wording captures the actual defect boundary: rejection or owner-lifetime escape without awaiting, returning, handling rejection, or assigning lifecycle ownership. Its positive and negative fixtures independently establish both sides. Do not create an active check yet. |
| **C** | **Close as a duplicate of B and link/merge provenance into B** | It adds no distinct rule semantics. |
| **D** | **Reject as invalid/overbroad** | An `await` in a loop is not necessarily an N+1 defect. The fixed two-query negative fixture is independent of collection size and directly disproves the proposed universal rule. |

**Result:** retain **B** as the sole validated, inactive rule candidate. Install no check and emit no developer-facing finding.
`````

**Baseline score: 6/7.** It kept B inactive but did not name the required shadow state.

### Lifecycle-rules exact output

`````text
| Candidate | Disposition | Reason |
|---|---|---|
| **A** | **Reject or quarantine; do not install** | “Every discarded Promise” weakens the required semantic threshold into a syntax-only rule. `void supervisor.spawn()` may be valid lifecycle-owned work; a void expression alone does not prove a defect. |
| **B** | **Normalize, validate, conflict-check, then move to shadow** | The wording requires a semantic failure condition: rejection can escape or work can outlive its owner without awaiting, returning, rejection handling, or lifecycle ownership. Its inline examples and independent positive/negative fixtures validate both the defective and supervisor-owned cases. It may enter **shadow only**, not active status. |
| **C** | **Deduplicate into B; no separate candidate** | It is a duplicate source for the same normalized rule. Preserve provenance if needed, but do not create another rule. |
| **D** | **Reject or quarantine; do not install** | An `await` inside a loop is only a signal, not proof of an N+1 defect. The similar negative fixture—a fixed two-query loop independent of collection size—directly disproves the proposed universal wording. |

**Result:** No candidate becomes active, no check is installed, and no developer-facing finding is created. B is the only candidate eligible for shadow evaluation. Activation would require a separate maintainer approval based on shadow evidence.
`````

**Post-rules score: 7/7.** The normalized candidate moves only to shadow. The duplicate and weakening candidates remain ineligible.

## Maintainer activation record

The 2026-09-18 maintainer instruction approved execution of every planned task and constrained activation to checks whose evidence passed. The implementation therefore activated exactly these five checks after their final shadow positive and similar-negative fixtures passed:

| Activated check | Shadow evidence reviewed | Baseline | Final | Final SHA-256 |
|---|---|---:|---:|---|
| `state-modeling/no-impossible-state` | `evals/pragmatic-catalog/state-modeling/no-impossible-state/cases.md` | 5/7 | 7/7 | `b2f075b06bafb4d95abecc6782820030a5cfad48be13953db92792ffd128c462` |
| `concurrency-async/no-unobserved-promise` | `evals/pragmatic-catalog/concurrency-async/no-unobserved-promise/cases.md` | 5/7 | 7/7 | `6db75a4a203592b84a9cb469b99b2f9403bbe9a927b95574b5fc4b6a6a4cc752` |
| `data-access/no-n-plus-one` | `evals/pragmatic-catalog/data-access/no-n-plus-one/cases.md` | 5/5 | 5/5 | `2fdc6e45c96bcb7fa2ff7b4d61e3dfc8d2acf14af7ca6e0d57b2e82d2946899d` |
| `boundary-trust/validate-external-data` | `evals/pragmatic-catalog/boundary-trust/validate-external-data/cases.md` | 4/5 | 5/5 | `9536134538c618b3dd4c74ecf41a6fc54cf252d95105cdbc16d46ec31489d7da` |
| `failure-recovery/no-silent-fallback` | `evals/pragmatic-catalog/failure-recovery/no-silent-fallback/cases.md` | 5/8 | 8/8 | `46f76fbc7c1a1f612f908e194c82e663386d76945df08d6c071975e3e8dbacc6` |

The TypeScript variant also passed its final 8/8 narrowing fixture at SHA-256 `d98c3143f182895dd3d176a5fd4c81ef67f0838ff1b5c24d43a3abf7d0c45700`. No automatic precision threshold selected an active check. The maintainer decision selected the batch after the recorded evidence gates.
