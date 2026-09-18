---
name: pragmatic-catalog
description: Use when a design, coding, or review workflow needs a bounded set of pragmatic engineering checks selected from current risks, ecosystem signals, project policy, or suggested stable check IDs.
---

# Pragmatic Catalog

Select concise engineering checks without turning the catalog into an implementation or review workflow.

The catalog owns metadata routing, scoped variant resolution, budgets, authority, and conflict quarantine. It returns reference material only. It does not edit code, review code, create findings, decide acceptance, or invoke another model or external tool.

## Non-negotiable gates

- Select only checks whose installed status is exactly `active`.
- Never select or load a `shadow` check or discovered candidate. If all applicable material is shadow, candidate, deprecated, superseded, conflicted, or otherwise ineligible, return `Status: no-match` and record each omission.
- A quarantined conflict never changes the output shape. Remove the conflicted material from selection, record it under `Quarantined conflicts`, and return the complete packet.
- Always return the exact Markdown packet under `Output contract`. JSON, free-form advice, a shortened result, and a direct finding are invalid.

## Inputs

Accept one request with:

- `phase`: `design`, `coding`, or `review`;
- `riskInventory`: factual change evidence, including relevant state, concurrency, async work, data access, external boundaries, caching, and failure behavior;
- `tags`: ecosystem and mechanism signals;
- `suggestedCheckIds`: optional stable IDs from domain skills;
- `policies`: applicable project policies with their authority, scope, and instruction; and
- `budget`: a check limit or token limit. A token limit never overrides a check-count cap.

Treat suggested IDs and tags as routing hints, not proof. Require current risk evidence before selecting a check.

## Installed index contract

Open the sibling `index.json`. Schema version 1 contains category metadata and `checks`. Each future check entry has:

- stable `id`, relative `path`, one `category`, `phases`, `tags`, `triggers`, and `status`;
- `relationships` using optional `refines`, `conflicts-with`, and `supersedes` ID lists; and
- `variants`, each with a stable `id`, relative `path`, matching `scope`, and `tags`.

Check bodies, fixtures, provenance narratives, and project policy content stay outside the index.

## Selection procedure

Follow these steps in order.

1. **Validate the request.** Require a valid phase, risk inventory, tags, policies, and non-negative budget. For coding, the effective check limit is the smaller of the requested limit and four. For review, route one category pass at a time with a three-to-five-check limit. For design, obey the supplied check or token limit.
2. **Read lightweight metadata.** Read `index.json` and, when ownership needs resolution, `categories.md`. Resolve every resource path relative to this skill directory. Reject absolute paths, traversal, and paths that escape the catalog. Paths are internal and never appear in the packet.
3. **Build the applicable set.** Consider only `active` entries whose phase and current evidence match. A suggested ID still needs evidence. Use tags and triggers to activate inspection, never to claim a violation.
4. **Apply lifecycle boundaries.** Candidates are not installed and cannot be selected. Shadow checks cannot change coding guidance or enter an ordinary review packet. Deprecated or superseded IDs remain historically resolvable but are not selected; use an active replacement only when its own evidence matches.
5. **Apply authority and conflicts.** Use the authority rules below. Quarantine unresolved same-authority contradictions before ranking. Deduplicate checks that require the same failure evidence and lead to the same corrective decision.
6. **Rank applicable checks.** Prefer, in order: scoped project-policy requirements; evidence-supported domain-skill suggestions; exact ecosystem variant matches; direct semantic or data-flow trigger matches; broader category matches. Recency, popularity, source count, and self-declared severity do not affect selection.
7. **Enforce the budget.** Coding selects zero to four checks and records applicable overflow for review. Review uses three to five applicable checks per focused category pass and splits a category above five. Never pad a pass with irrelevant checks when fewer than three apply. Design stays within its supplied budget.
8. **Load progressively.** Only after selection, load selected core bodies and matching variants. Keep unmatched checks, unmatched variants, the full category, candidates, fixtures, evaluations, and provenance unloaded.
9. **Return the packet.** Emit the exact ordered contract below only after the verification gate passes.

## Authority and conflict rules

Apply authority only where scopes overlap:

1. explicit current human decision and approved product or engineering specification;
2. scoped project policy or accepted architecture decision;
3. verified public contract, schema, or behavioral invariant;
4. matching ecosystem variant;
5. general core check;
6. discovered candidate.

A variant may refine detection details but cannot weaken the core invariant. A policy override must identify its scope. Quarantine both sides of a same-authority contradiction; do not resolve it by order, recency, popularity, or model preference. Record the quarantined IDs, shared authority, overlapping scope, and contradiction.

## Output contract

Return the exact Markdown skeleton below, with these sections in this order and no additional sections. Do not substitute JSON, prose advice, or a shortened status report.

```markdown
**Pragmatic catalog selection packet**
- Status: selected | no-match
- Phase: design | coding | review
- Category pass: none | <category ID>
- Budget: <supplied unit and limit>
- Effective limit: <check and token limits that apply>

### Selected checks
#### <stable check ID>
- Variant: none | <stable variant ID>
- Reasons: <policy, supported suggestion, scope, and evidence matches>

##### Core body
<selected core check body>

##### Variant body
None. | <matching variant body>

### Applied policies
- <policy, scope, authority, and effect>

### Quarantined conflicts
- <IDs, shared authority, overlapping scope, and contradiction>

### Budget omissions
- <stable ID, applicability reason, and why the effective budget deferred it>

### Other omissions
- <suggested or historically resolved ID and reason: unknown ID, not applicable, non-active status, or duplicate>
```

Use `None.` for every empty section. A `no-match` packet contains no check or variant body. It remains a complete packet so the caller can apply its own no-match contract.

## Missing and no-match behavior

- **No match:** return the complete packet with `Status: no-match`, `Selected checks: None.`, and no loaded body. For coding, the consuming capability maps this result to exactly `no pragmatic enrichment required`.
- **Catalog capability absent:** no packet exists. Coding may continue under its own contract with `catalog enrichment unavailable`; pragmatic review blocks and names `pragmatic-catalog` as required. Self-contained domain skills remain usable.
- **Missing or invalid installed resource:** return no partial packet. Report exactly:

```markdown
**Pragmatic catalog unavailable**
- Reason: <missing or invalid index, category metadata, selected check, or selected variant>
- Safe result: no checks selected
- Required repair: <one bounded catalog repair>
```

Do not broaden the search to candidates, fixtures, repository-root files, another install source, or external assistance.

## Finding boundary

Selection means “inspect with this guidance,” not “a defect exists.” The catalog never emits a finding, confidence, or severity. A downstream review may report a finding only when it independently provides the stable check ID, cited code evidence, demonstrated failure, contextual impact, evidence confidence, contextual severity, and bounded fix direction. Unproven suspicion remains silent.

## Verification gate

Before returning a selected packet, verify all of the following:

- schema version is `1`, every selected ID is unique, and every selected check is `active`;
- every selected phase, category, tag, trigger, suggestion, and policy claim is supported by the request and installed metadata;
- every selected path resolves inside this catalog and every loaded variant matches the current scope;
- authority has been applied only to overlapping scopes and every unresolved same-authority conflict is quarantined;
- coding contains at most four checks, each review pass contains at most five, and every applicable overflow appears under budget omissions; and
- no unmatched resource, fixture, candidate, provenance record, external model, implementation action, review action, or developer-facing finding entered the packet.

If any item fails, return the unavailable report instead of a partial selection.
