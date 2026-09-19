---
name: check-feature-readiness
description: Use when a feature needs an evidence-backed readiness report for a supplied project checklist, candidate revision, and captured verification or operating evidence.
---

# Check Feature Readiness

Evaluate supplied or already-observed evidence against a project-specific feature-readiness checklist. Report the evidence boundary for each applicable item. Product acceptance, release authorization, and the decision to expose a feature remain human-owned.

This is evidence-only: MUST NOT run repository commands or tools, inspect unsupplied systems, implement fixes, select proof methods, invent thresholds or acceptance criteria, or convert missing evidence into a pass.

## Required inputs

Require the project checklist, feature/candidate revision and operating scope, and available captured evidence. The checklist must supply the condition and local proof/currency rule where the repository has defined them.

If there is no project-specific checklist, return `blocked` for evaluation: generic baselines can help author criteria but are not evaluation authority.

## Qualify evidence

A claim can support `pass` or `fail` only when it:

1. traces to one exact applicable checklist condition;
2. fits the stated feature revision and operating scope;
3. identifies its source and capture context;
4. is current under the checklist’s repository-defined currency rule;
5. directly demonstrates the condition’s claimed outcome; and
6. has no unresolved contradiction.

A green test, build, deployment, pipeline, ticket, approval, or dashboard is bounded supporting context, not readiness proof by itself. A generic claim such as `CI passed` qualifies only if the supplied captured evidence maps it directly to the condition and all six rules above.

## Status semantics

| Status | Use only when |
| --- | --- |
| `pass` | The item applies and qualifying evidence directly demonstrates its condition. |
| `fail` | The item applies and qualifying evidence directly demonstrates that its condition is not met. |
| `blocked` | Evaluation cannot proceed because the checklist, a required evidence source, its capture context, or a contradiction-resolution authority is inaccessible or withheld. State the blocker. |
| `unverified` | The item applies but no supplied evidence qualifies: it is missing, stale, ambiguous, indirect, out of scope, unmapped, or incomplete. State the exact qualification gap. |
| `not applicable` | The checklist’s activation condition is demonstrably false for the supplied feature scope. Cite that fact. |

Do not replace `unverified` with `fail` merely because proof is missing. Do not use `pass` as an overall release or product-acceptance verdict.

## Procedure

1. Establish the candidate revision, operating scope, checklist version, and supplied evidence boundary. Record any unknown as a limit, not an assumption.
2. For each checklist item, establish whether its activation condition applies. Preserve a non-applicable item only with its supporting fact.
3. For each applicable item, record all six qualification results—condition traceability, revision and operating scope, source and capture context, currency, direct outcome, and contradiction state—even when a prior result already makes the evidence insufficient. Do not merge evidence across conditions unless the evidence explicitly demonstrates both.
4. Assign one status using the table. For `fail`, `blocked`, and `unverified`, name the unmet condition or exact missing qualification. Name a next owner or action only when the checklist or supplied evidence identifies one.
5. Report aggregate status counts. State that they are an evidence summary, not a readiness approval or product acceptance decision.

## Report contract

Return exactly this report shape; use `None.` for empty sections.

```markdown
**Feature readiness evidence report**
- Candidate revision and operating scope: <supplied value or unknown>
- Checklist: <project checklist identity and version>
- Evidence boundary: <supplied or already-observed sources only>
- Evaluation limitation: <unknown inputs or None.>

### Item status
| Item | Status | Condition and activation | Qualified evidence or qualification gap | Next owner/action when known |
| --- | --- | --- | --- | --- |

### Evidence qualification
| Item | Condition trace | Revision and scope | Source and capture | Currency | Direct outcome | Contradiction state |
| --- | --- | --- | --- | --- | --- |

### Status summary
- Pass: <count>
- Fail: <count>
- Blocked: <count>
- Unverified: <count>
- Not applicable: <count>

### Human-owned decision boundary
This report supplies evidence statuses only. Product acceptance and release exposure remain human-owned.
```

## Red flags

| Shortcut | Required result |
| --- | --- |
| “CI passed,” “dashboard is green,” or an approval link has no condition mapping and capture context | `unverified`; identify the missing qualification facts. |
| Evidence is from another revision, target, tenant, configuration, or operating scope | `unverified` unless the checklist explicitly makes it applicable. |
| A test or delivery succeeded but does not directly demonstrate the condition | `unverified`; preserve it only as supporting context. |
| Evidence directly conflicts | `blocked` until the named authority resolves the contradiction. |
| A launch deadline or stakeholder requests a pass | Keep the evidence-derived status; do not grant approval. |
