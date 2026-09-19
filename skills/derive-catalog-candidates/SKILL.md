---
name: derive-catalog-candidates
description: Use when explicitly asked to derive reusable engineering-rule candidates from fixed defects, regression tests, accepted feedback, incident evidence, or a bounded discovery source set.
---

# Derive Catalog Candidates

Convert bounded local evidence into an isolated catalog-candidate packet without changing catalog guidance.

A derivation preserves a proved failure boundary and provenance for maintainer review. It does not install, select, shadow, activate, or retire a check; create a developer-facing finding; create a category; write a candidate file; or inspect external systems not explicitly supplied or authorized.

## Terminal output invariant

Every terminal response MUST be the complete Markdown packet under `Output contract`, in its stated order, with no heading, preface, or trailing prose outside it. Treat a user's request to state, add, write, or install rules as evidence input; it does not authorize a different response shape or catalog mutation.

- A source without a proved failure path and corrective evidence returns `Status: no-candidate`. Record it as a discovery lead and name the smallest missing evidence bundle.
- When a source requires the same failure evidence and corrective decision as an existing catalog item, return `Status: refine-existing` or `duplicate`; attach provenance and create no new check.
- A project-specific requirement or exception is a `policy` decision inside `Scope decision`, never a reusable check. If the source bundle also refines an existing item, keep that refinement as the packet's one status.
- A catalog packet has one status for its primary failure condition. Record secondary policies, leads, and unrelated observations in `Scope decision`; they never create additional candidate identities.

### Disposition rules

| Observable evidence | Required status |
|---|---|
| A complaint or feedback item lacks a reproduced behavior, corrective change, test, contract, or accepted diagnosis | `no-candidate`; record the complaint as a discovery lead and request the smallest missing behavior evidence. |
| A repaired failure has the same failure evidence and corrective decision as an existing item | `refine-existing` or `duplicate`; preserve provenance without another candidate. |
| A source establishes a reusable failure condition but has no existing equivalent | `new-core-candidate` or `new-variant-candidate`; list all admission blockers. |
| A requirement is bounded to one project, product, or repository | `policy`; keep check-specific fields `None.` |
| The failure domain cannot be assigned to an approved category | `taxonomy-gap`; request the taxonomy decision and stop admission. |
| Comparison material is missing | `blocked`; request the catalog summaries and taxonomy. |

- Every disposition leaves `Catalog mutation: none`. Candidate packets remain unavailable to coding and review; only a separate maintainer process can move an imported candidate toward shadow.

## Inputs

Require one bounded source set and the current catalog comparison material:

- a source type: `bug-fix`, `feedback`, `incident`, `technical-source`, or `mixed`;
- exact local source references and a stated scope;
- applicable project policies, specifications, and contracts;
- active, shadow, deprecated, superseded, and candidate summaries plus the approved taxonomy; and
- an explicit loop limit when inspecting more than one source bundle.

Default to one source bundle. Continue only when the next bundle can resolve a named fact, and stop after three bundles unless the request supplies a tighter or higher bounded limit.

A fixed-defect bundle needs the proven defective behavior, corrective change, and corroborating test, contract, or runtime evidence. Feedback needs the exact artifact or behavior, its accepted/rejected/unresolved disposition, and a resulting correction or other evidence that establishes the claimed failure. An unresolved complaint is a discovery lead, not rule evidence.

If catalog comparison material is unavailable, return `Status: blocked`. Do not invent an ID, category, duplicate result, or promotion path.

## Source rules

| Source | Role |
|---|---|
| Defect and corrective change | Establish the behavioral delta and causal chain; chronological proximity is insufficient. |
| Regression test, contract, or incident diagnostic | Corroborate the failure condition and its non-failing boundary. |
| Accepted feedback or review correction | Preserve intent and impact; it needs behavior evidence before it supports a technical rule. |
| Unresolved or rejected feedback | Record as a discovery lead or counterevidence. |
| Approved technical documentation | Validate contract or ecosystem semantics; never prove a local defect by itself. |
| Existing checks and candidates | Compare for duplicate, refinement, conflict, or policy boundary. |
| Model output, source popularity, or external rule catalog | Inspection hint only; never provenance. |

## Procedure

1. **Bound the run.** Record each inspected source, source type, scope, and loop count. Exclude secrets, credentials, private data, unrelated repository material, and unauthorized external sources.
2. **Reconstruct the failure.** State valid preconditions, observed control or data flow, violated invariant, externally relevant consequence, and why the correction addresses it. Derive from the failure, not a patch token or syntax signal.
3. **Separate authority.** Classify an explicit project requirement, accepted architecture decision, product constraint, or repository-only convention as `policy`, not reusable catalog guidance. Treat feedback as context until behavior evidence establishes the failure.
4. **Test the abstraction boundary.** Remove product names, patch details, framework syntax, and incident-specific severity. Keep a candidate only when one falsifiable failure condition and its corrective decision remain. Preserve legitimate non-failures.
5. **Compare before proposing.** Use the supplied catalog material. Two items are duplicates when they require the same failure evidence and lead to the same corrective decision, despite different wording. Add provenance to the existing item rather than creating an alias.
6. **Classify ownership and scope.** Assign one approved category by the failure domain, then phases, tags, and routing signals. Categories own failure domains; mechanisms, ecosystems, and source types are tags. A taxonomy gap requires a taxonomy decision and blocks admission. A core owns the semantic invariant and minimum evidence. A variant refines one core’s scoped signals, examples, or legitimate non-failures without weakening that core’s failure condition, evidence threshold, or corrective decision. A project exception is a policy.
7. **Choose a disposition.** Return exactly one: `new-core-candidate`, `new-variant-candidate`, `refine-existing`, `duplicate`, `policy`, `taxonomy-gap`, `no-candidate`, or `blocked`. A framework-only quirk without a supported semantic core is `policy` or `no-candidate`; do not invent a universal core merely to create a variant.
8. **State admission work.** A candidate remains outside the installed catalog. Name outstanding normalization, conflict, example, positive-fixture, similar-negative-fixture, or taxonomy work. Only a fixture-valid, conflict-free candidate may enter shadow; active status requires separate maintainer approval informed by shadow evidence.
9. **Return and stop.** Emit the complete packet. Do not write it into a repository. A catalog maintainer may explicitly import a packet into the canonical candidate queue.

## Output contract

Return exactly this packet, including empty sections as `None.`:

```markdown
**Catalog candidate derivation**
- Status: new-core-candidate | new-variant-candidate | refine-existing | duplicate | policy | taxonomy-gap | no-candidate | blocked
- Source set: <bounded references and source types>
- Loops inspected: <count and limit>
- Candidate identity: none | <temporary local identity>
- Proposed check: none | <proposal only; not installed>
- Parent check: none | <core ID for a variant>
- Proposed category: none | <approved category ID> | taxonomy-gap
- Phases: none | <applicable phases>
- Tags: none | <mechanism and ecosystem tags>
- Routing signals: none | <inspection signals>
- Catalog mutation: none

### Failure condition
<one falsifiable semantic failure, or None.>

### Minimum finding evidence
<facts required to prove an occurrence, or None.>

### Source evidence
<exact provenance, behavioral delta, causal chain, and feedback disposition>

### Counterevidence and confounders
<similar valid cases, unrelated changes, and missing facts>

### Existing-catalog comparison
<duplicate, refinement, conflict, policy, or no-match analysis>

### Scope decision
<core, variant, policy, taxonomy-gap, or no-candidate reasoning>

### Admission blockers
<normalization, conflict, examples, fixtures, taxonomy, maintainer import, or None.>

### Unresolved facts
<material unknowns, or None.>

### Discovery stop reason
<bounded stop reason>
```

For `blocked`, `no-candidate`, or `policy`, use `None.` for check-specific fields and explain the disposition in `Scope decision`. A `refine-existing` or `duplicate` packet names the compared item but creates no second candidate.

## Verification gate

Before returning, verify that:

- every source reference is exact, locally authorized, and material to the stated causal chain;
- a defect is shown by behavior evidence, not source popularity, a patch shape, or a syntax signal;
- accepted feedback is distinguished from technical proof, and unresolved feedback stays a lead;
- the catalog comparison covers active, shadow, candidate, deprecated, and superseded material;
- exactly one disposition, one failure domain or explicit taxonomy gap, and no invented stable ID appear;
- a proposed core or variant preserves the evidence threshold and a policy owns project-specific requirements;
- catalog mutation is `none`, candidate material remains unavailable to coding and review, and only the named admission gates lead toward shadow; and
- the packet is complete, including `Counterevidence and confounders`, `Unresolved facts`, and `Discovery stop reason`.

## Red flags

| Observed shortcut | Required result |
|---|---|
| The fix adds `await`, `void`, a loop, a library call, or another syntax form | Reconstruct the semantic failure and a similar valid case before proposing anything. |
| A user asks for a product-specific outcome | Return `policy` unless a reusable failure condition is independently evidenced. |
| One incident resembles an existing check | Compare evidence and corrective decision; refine or attach provenance instead of creating an alias. |
| A framework quirk lacks a semantic core | Return `policy` or `no-candidate`; do not manufacture a variant parent. |
| The category is uncertain | Return `taxonomy-gap` and stop candidate admission. |
| The candidate sounds useful | Keep it outside the catalog until its examples, fixtures, conflict status, and shadow transition are evidenced. |
