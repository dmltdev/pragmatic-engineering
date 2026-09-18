---
name: pragmatic-consulting
description: Use when an explicit user request or project opt-in authorizes an independent external opinion on a pre-coding engineering design decision.
---

# Pragmatic Consulting

Challenge a proposed engineering design before coding, then test the external opinion against local project authority and design checks.

This skill is design-only. It advises; it does not edit source, execute a plan, accept work, or change lifecycle state. The human retains the final decision.

**Terminal output invariant:** Use only the exact complete or blocked field list under Output contract, including when the user asks for a report or another format.

## Entry gate

Start only when one of these authorizations exists:

- the user explicitly invokes or requests `pragmatic-consulting`; or
- a current project policy explicitly opts into external design consultation and defines an applicable disclosure scope.

Prompt similarity, model initiative, and a general request to implement or review code are not authorization. If neither authorization exists, make no external call and return the blocked report.

Authorization to consult is not blanket disclosure permission. Apply all user and project confidentiality limits. When their scope is unclear or conflicting, disclose nothing and return the blocked report.

## Required capabilities

The workflow requires:

- `consult-llm` with the `--task plan` contract; and
- `pragmatic-catalog` for local `design`-phase selection.

Check capability availability before preparing an external call. A missing capability blocks only this optional consultation. Catalog, coding, review, domain skills, and ordinary local design work remain available. Do not install, configure, or substitute a dependency.

## Workflow

### 1. Frame the decision locally

Establish from authorized repository evidence:

- the exact design question and realistic alternatives;
- current project requirements, accepted specifications, policies, and architecture decisions;
- observed constraints, risks, and unresolved facts; and
- the human or team that owns the decision.

Do not treat the consultant as project authority.

### 2. Build a disclosure allowlist

Select the smallest excerpts or files required to understand the decision. Before the call, record the exact eligible context and why each item is necessary.

Eligible context is relevant, authorized, and non-secret. Exclude credentials, secrets, tokens, environment files, personal data, unrelated proprietary material, generated output, vendored content, build artifacts, dumps, and noisy logs. Exclude whole files when a bounded excerpt is enough. Never broaden scope merely because the consultant may benefit from more context.

If a material fact exists only in excluded or unauthorized context, preserve it as an unresolved fact or ask the disclosure owner for a narrower safe artifact.

### 3. Request one independent opinion

Invoke the `consult-llm` capability in `--task plan` mode with:

- one neutral design question;
- the allowlisted context only;
- the project constraints the answer must respect; and
- a request for alternatives, trade-offs, unresolved facts, and a recommendation.

Do not preload a preferred solution, catalog conclusion, or desired verdict. Record the resolved model and the disclosed context. Treat the response as advice, not authority.

### 4. Resolve at most one context request

If the consultant makes a material context request, collect all requested items into one bounded follow-up. Reapply the authorization and disclosure allowlist to every item. Attach only approved additional context and state which requested items remain unavailable.

Perform no second context follow-up. Preserve any repeated request or remaining gap as unresolved uncertainty.

### 5. Apply local design checks

After the consultant's final response, build a compact design risk inventory and request applicable `design`-phase checks from `pragmatic-catalog`. Apply only the returned active checks and matching scoped policies. Shadow or candidate checks cannot change guidance or appear as findings.

Assess each alternative against current repository evidence. A check signal prompts inspection; it does not prove a failure. When a design concern is proven, cite its stable check ID, relevant evidence, failure condition, impact, confidence, severity, and bounded fix direction.

### 6. Reconcile authority

Authority applies in this order: explicit current human decisions and approved specifications, scoped project policy or accepted architecture decisions, verified contracts and invariants, then reusable checks and consultant advice.

Project authority wins a conflict with consultant advice. Record the disagreement and its trade-off; do not rewrite project authority, hide the conflict, or misclassify it as a catalog conflict. Separate facts from human-owned priorities such as risk tolerance, cost, schedule, and reversibility.

### 7. Return advice and stop

Return the report below. Make no implementation edit or acceptance claim. A later workflow may implement only after the human-owned decision is made.

## Output contract

Every terminal response is one of the following reports, with no heading, preface, or trailing prose outside it.

On completion, report exactly:

```markdown
**Pragmatic consultation complete.**
- Authorization: <explicit invocation | project opt-in, including disclosure scope>
- Decision question: <one sentence>
- Consultant: <resolved model>
- Disclosed context: <exact files or bounded excerpts and why each was needed>
- Alternatives: <realistic options, including the local/default option>
- Trade-offs: <bounded comparison>
- Local design checks: <none applicable, or stable ID, evidence, failure, impact, confidence, severity, and bounded fix direction for each proven concern>
- Project-authority disagreements: <consultant claim, controlling authority, and trade-off, or none>
- Unresolved facts: <facts or context gaps that could change the recommendation, or none>
- Recommendation: <advice consistent with controlling authority>
- Human-owned decisions: <choices the agent and consultant cannot make>
- Status: design advice only; not implemented or accepted
```

When blocked, report exactly:

```markdown
**Pragmatic consultation blocked.**
- Reason: <missing authorization | unclear disclosure scope | missing capability>
- Missing requirement: <one concrete authorization, scope, or capability>
- Disclosed context: none
- Unaffected work: catalog, coding, review, domain skills, and local design work remain available
- Required owner: <user | project owner | security | capability operator>
- Status: no external call; not implemented or accepted
```

## Verification gate

Do not call consultation complete unless authorization is recorded, disclosed context is an exact minimal allowlist, no excluded material was sent, at most one context follow-up occurred, local design checks were applied, project authority controlled conflicts, and the report leaves the decision with the human owner.
