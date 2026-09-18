# Routed Engineering Checks v2

**Status:** Approved — amended 2026-09-13 for manual-only advanced review

**Date:** 2026-09-13

## Authority and sources

This document is the durable authority for the routed engineering-check system in `pragmatic-engineering`. It derives from:

- the current package responsibility in [`README.md`](../../README.md);
- the canonical install-source and verification rules in [`AGENTS.md`](../../AGENTS.md);
- the current skill domain index in [`skills/registry.json`](../../skills/registry.json);
- the approved 2026-09-13 design discussion covering catalog routing, conflicts, noise control, coding, review, external consultation, and advanced review;
- the local [consult-backed OCR delegation idea](../ideas/consult-backed-ocr-delegation.md) and the official OpenCodeReview [delegation skill contract](https://github.com/alibaba/open-code-review/blob/main/plugins/open-code-review/skills/open-code-review-delegate/SKILL.md);
- the official manual-invocation controls for [Claude Code](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill), [Codex](https://learn.chatgpt.com/docs/build-skills#optional-metadata), and [Pi](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/skills.md#frontmatter); and
- the shipped `dependency-seam` contract as evidence that domain skills must remain useful without optional enrichment.

When sources conflict, explicit current human decisions control product meaning and scope; this specification controls the routed-check architecture; `AGENTS.md` controls repository workflow until implementation intentionally updates it; executable code and evaluations control observed behavior.

This specification supersedes the first-release prohibition on a rule catalog and umbrella-like engineering gates. It does not supersede the package boundary against product discovery, acceptance ownership, lifecycle state, deployment tooling, or personas. The advanced review skill is an explicit, isolated exception to the previous no-cross-plugin-dependency posture; its external tools remain private implementation details rather than public skill identities.

## Feature summary

`pragmatic-engineering` will provide a precision-first system of concise engineering checks that can expand quickly without loading a large rule collection into every coding or review prompt.

Checks are dormant reference material. A router selects them from change-specific evidence. Coding receives a very small set of preventive guardrails; review receives broader coverage through focused category passes; consultation may apply design-phase checks after receiving an independent external design opinion. A manually invoked advanced review may add slower, independent, coverage-checked analysis before local pragmatic validation.

The system optimizes first for trustworthy, actionable findings. A check cannot create a developer-facing finding without concrete code evidence. Discovery may create candidates automatically, but candidates cannot affect coding or review until they pass quality gates and promotion.

Automated discovery scheduling is outside this specification. This specification begins when discovered material enters the candidate lifecycle.

## Users

- **Implementation agent:** needs a small risk-specific guardrail packet without losing context to the catalog.
- **Review agent:** needs thorough category coverage, concrete evidence requirements, conflict handling, finding prioritization, and an explicitly requested deeper review option.
- **Designing agent or engineer:** may request an independent external design challenge before implementation.
- **Catalog maintainer:** normalizes candidates, maintains taxonomy and precedence, reviews shadow evidence, and promotes or retires checks.
- **Domain-skill author:** may suggest stable check IDs without making the domain skill depend on the catalog.

## Responsibility boundary

The routed-check system owns:

- the check contract, taxonomy, scoped variants, lifecycle, and provenance;
- metadata-based routing and context budgets;
- precedence, conflict quarantine, supersession, and deprecation;
- coding guardrail selection;
- evidence-backed review passes and finding aggregation;
- normalization and coverage accounting for manual advanced review; and
- explicit or project-approved external design consultation.

It does not own:

- automated discovery schedules or agent-run infrastructure;
- product discovery, product meaning, acceptance, or lifecycle state;
- implementation execution, source-control operations, deployment, or release orchestration;
- personas or generic agent-role definitions;
- deterministic host integration beyond the portable Agent Skills contract; or
- automatic external disclosure of repository content.

`pragmatic-coding` enriches implementation; it does not replace the host's implementation workflow. `pragmatic-review-advanced` may collect candidate observations but cannot create final findings. `pragmatic-review` is the mandatory engineering evidence gate and does not change acceptance state. `pragmatic-consulting` advises on design; the primary agent and human retain decision ownership.

## Domain model

### Check

One evidence-backed engineering failure condition. A check is dormant until selected and cannot emit a finding without proof.

### Policy

A project-specific instruction that may narrow, require, or override a check within an explicit scope. Policies have higher authority than reusable catalog guidance.

### Category

The one stable failure domain that owns a check's review pass. Categories answer what kind of failure can occur.

### Tag

A non-owning mechanism, ecosystem, or signal label used for routing. Examples include `promise`, `orm`, `graphql`, `react`, and `postgres`.

### Variant

A language-, framework-, or ecosystem-specific refinement of one core check. A variant may add syntax signals, data-flow signals, examples, and narrower applicability. It cannot silently weaken the core check.

### Candidate

Discovered content that is not available to coding or review. A candidate remains untrusted until normalized, conflict-checked, fixture-validated, and promoted to shadow.

### Fixture

A realistic evaluation case used for promotion and regression. Every check requires at least one positive fixture that must be detected and one similar-looking negative fixture that must not produce a finding.

### Shadow check

A fixture-valid check that may run against real diffs but cannot create developer-facing findings. Shadow evidence supports promotion or revision.

### Risk inventory

A compact factual description of a proposed or implemented change: languages, frameworks, state transitions, concurrency, asynchronous work, data access, external boundaries, caching, failure behavior, and other observed risk signals.

### Selection packet

The catalog's output: selected check IDs and bodies, variants, selection reasons, applied policies, quarantined conflicts, and budget omissions.

### Finding

A proven occurrence of a selected check. It contains the check ID, cited code evidence, demonstrated failure condition, contextual impact, evidence confidence, contextual severity, and a bounded fix direction.

### Review candidate

An untrusted observation from the advanced reviewer. It may direct local inspection but cannot inherit final severity, confidence, or finding status.

### Advanced review packet

The normalized handoff from advanced review: review-model identity, exact target, selected and excluded files, per-file reviewed or skipped status, applicable rule groups, warnings, unresolved uncertainty, and candidate observations. Provider-specific reasoning and hidden chain-of-thought are excluded.

## Architecture

### `pragmatic-catalog`

`pragmatic-catalog` is the shared lookup module and source of truth for active checks.

It accepts:

- phase: `design`, `coding`, or `review`;
- risk inventory;
- ecosystem and mechanism tags;
- optional check IDs suggested by a domain skill;
- applicable project policies; and
- a check or token budget.

It returns a selection packet. It resolves internal paths and variants itself. Consumers use stable check IDs and the catalog capability, never relative paths into another skill.

`pragmatic-catalog` does not implement changes, perform code review, or call external models.

### `pragmatic-consulting`

`pragmatic-consulting` is an optional pre-coding design gate with a hard dependency on the separately provided `consult-llm` capability.

It runs only after explicit user invocation or explicit project opt-in. It:

1. gathers focused repository evidence and the real decision question;
2. sends a neutral question and only relevant, non-secret files to `consult-llm --task plan`;
3. resolves at most one consultant context request according to the provider contract;
4. keeps the external opinion independent by not preloading a preferred solution or catalog conclusion;
5. applies design-phase catalog checks locally to the returned alternatives; and
6. returns alternatives, trade-offs, unresolved facts, a recommendation, and human-owned decisions.

If `consult-llm` is unavailable, consultation blocks with an explicit dependency report. Catalog, coding, review, and domain skills remain unaffected.

### `pragmatic-review-advanced`

`pragmatic-review-advanced` is the optional, deliberately slower review workflow. Its public contract is generic: accept a review target and optional model override; return coverage, evidence-backed final findings, rejected candidates, and unresolved uncertainty. OpenCodeReview delegation and `consult-llm` are private implementation dependencies, not concepts consumers must understand.

The skill is manual-only. It must never activate from prompt similarity, automatic skill selection, project opt-in, scheduled discovery, or another model's initiative. Its canonical `SKILL.md` frontmatter includes:

```yaml
name: pragmatic-review-advanced
description: Run a deliberate, slower, coverage-checked independent review before pragmatic validation. Invoke explicitly when deeper review is worth additional latency and model cost.
disable-model-invocation: true
user-invocable: true
```

Claude Code, Pi, and OMP enforce `disable-model-invocation`. Codex requires `agents/openai.yaml` beside the skill:

```yaml
policy:
  allow_implicit_invocation: false
```

The base Agent Skills standard does not define manual-only invocation, so these supported-host extensions are required behavior, not portable metadata. Project policy may configure defaults but cannot invoke the skill. If a supported host cannot prove implicit invocation is disabled, it must not advertise the advanced skill as available.

#### Internal implementation contract

After explicit user invocation, the skill:

1. records the review mode, refs, target identity, and diff fingerprint;
2. runs `ocr delegate preview --format json` for that target to obtain every reviewable and excluded file without an OCR LLM call;
3. runs `ocr delegate rule --format json <paths...>` to resolve and group the applicable rules;
4. assembles the selected diffs, rule groups, business context, and required coverage manifest;
5. invokes one resolved model through `consult-llm --task review` and requires every selected file to be marked `reviewed` or `skipped` with a concrete reason;
6. applies the bounded context-request loop, validates result schema and coverage, and normalizes supported claims as review candidates; and
7. passes the advanced review packet to `pragmatic-review` for independent catalog-backed validation.

Preferred execution lets a tool-capable review session run the deterministic preparation commands in the target repository. When that session cannot access local binaries, the host runs them and attaches exact JSON, diffs, and resolved rules. Both modes must produce the same advanced review packet and coverage guarantees. The deterministic preparation tool never invokes or configures an LLM endpoint; the independent review-model boundary may still use credentials or incur provider cost.

Model selection order is an explicit model override supplied with the invocation, scoped project policy, then one configured review default. If configuration resolves multiple defaults, the workflow blocks for one explicit model; comparative multi-model review is outside the MVP. The packet records the resolved model.

The internal reviewer must identify the target, account for every selected file, and return candidate path, line range when known, category, source severity, failure claim, cited evidence, and recommendation. Missing coverage, malformed output, target drift, unavailable dependencies, or unauthorized external disclosure block advanced review. Standalone `pragmatic-review` remains available.

The workflow does not install or configure dependencies, post PR comments, apply fixes, import external rules into the pragmatic catalog, or promote reviewer claims into catalog checks.

### `pragmatic-coding`

`pragmatic-coding` is a risk-triggered implementation enrichment gate. It activates automatically for non-trivial production changes involving state, concurrency, asynchronous behavior, data access, external boundaries, caching, or failure semantics. It skips documentation-only, formatting-only, mechanical rename, and equivalent low-risk changes.

It:

1. builds a compact risk inventory from the request and repository evidence;
2. selects applicable self-contained domain skills;
3. requests zero to four coding-phase checks from `pragmatic-catalog`;
4. produces a concise guardrail packet before implementation; and
5. verifies the selected risks after implementation.

If no domain skill or check applies, it returns `no pragmatic enrichment required` and adds no further context. If the catalog is unavailable, coding continues with self-contained domain skills and an explicit `catalog enrichment unavailable` disclosure.

`pragmatic-coding` does not own the implementation plan, edit loop, tests, commits, or delivery lifecycle.

### `pragmatic-review`

`pragmatic-review` is a precision-first engineering review gate. It requires `pragmatic-catalog`; missing catalog capability blocks the review rather than silently reducing coverage. It may accept an advanced review packet, but treats every advanced-review observation as untrusted inspection input.

It:

1. builds a risk inventory from the diff and relevant repository evidence;
2. requests matching review-phase checks and scoped variants;
3. uses any normalized review candidates only to prioritize relevant local inspection;
4. runs one focused pass per activated category, normally with three to five checks per pass;
5. independently requires concrete code evidence for every final finding; and
6. aggregates, deduplicates, resolves precedence, and reports contextual severity.

Category passes may run in parallel. The aggregator cannot invent a finding absent from category-pass evidence. An advanced-review candidate without local proof is rejected or left as unresolved uncertainty, never reported as a pragmatic finding.

### Self-contained domain skills

Existing and future domain skills, including `dependency-seam`, remain complete when installed alone. They may include an optional enrichment section naming:

- capability: `pragmatic-catalog`;
- suggested stable check IDs; and
- the condition under which each ID is relevant.

A domain skill never references another skill's filesystem path and never fails because optional catalog enrichment is absent.

## Check contract

Each active or shadow core check has stable metadata and one concise body.

### Required metadata

```yaml
id: data-access/no-n-plus-one
category: data-access
phases: [coding, review]
tags: [query, collection, cardinality]
triggers: [query-in-loop, per-item-resolver, repeated-remote-read]
status: active
```

Rules:

- `id` is stable and uses `<category>/<slug>`.
- `category` is exactly one approved failure domain.
- `phases` contains only phases where the check changes behavior.
- `tags` describe mechanisms and ecosystems; they do not create pass ownership.
- `triggers` are routing signals, not proof of a violation.
- `status` is `shadow`, `active`, `deprecated`, or `superseded` for installed catalog content. Candidates remain outside the installed catalog.
- Optional relationships may identify `refines`, `conflicts-with`, or `supersedes` IDs.
- Provenance and discovery confidence are retained for governance but are not loaded into ordinary coding or review prompts.

### Required body

Every check contains these sections in order:

1. `Failure` — one falsifiable failure condition.
2. `Signals` — concise syntax, data-flow, and semantic indicators.
3. `Evidence` — the minimum code facts required for a finding.
4. `Bad` — at least one concise violating example.
5. `Good` — at least one concise similar non-violating example.
6. `Allow` — legitimate exceptions or bounded cases.
7. `Fix` — a bounded correction direction, not a complete implementation tutorial.

The prose target is at most 180 words, excluding metadata and code. Each required example should normally fit within six lines. A check covers one failure mode; broad principles must be split before activation.

Signals only activate inspection. Syntax such as `await` inside a loop is never sufficient proof by itself.

## Scoped variants

A core check owns the semantic invariant. A scoped variant may:

- narrow applicability to a language, framework, library, or runtime;
- add syntax or data-flow signals;
- replace generic examples with ecosystem-specific examples; and
- add ecosystem-specific legitimate exceptions.

A variant cannot remove the core evidence requirement or permit the core failure condition. A true project exception is represented as a higher-authority policy, not hidden inside a variant.

Only the matching variant is loaded. Unmatched ecosystem variants consume no model context.

## Taxonomy

The initial primary categories are:

| Category | Failure domain | Example checks |
|---|---|---|
| `state-modeling` | Invalid state representation or transition | `no-impossible-state`, `make-transitions-explicit` |
| `concurrency-async` | Ordering, concurrency, task, or promise correctness | `no-racy-check-then-act`, `no-unobserved-promise` |
| `data-access` | Query cardinality, access shape, and data-volume behavior | `no-n-plus-one`, `bound-pagination` |
| `boundary-trust` | Untrusted input, external representation, and boundary translation | `validate-external-data`, `preserve-error-cause` |
| `failure-recovery` | Retry, fallback, idempotency, and failure visibility | `no-silent-fallback`, `design-idempotent-retry` |

Each check has one primary category. Mechanisms such as promises, ORMs, queues, React, and databases are tags or variants. Adding or splitting a primary category requires an explicit taxonomy decision because category changes alter routing and review-pass ownership.

## Routing and context budgets

Routing is two-stage progressive disclosure:

1. inspect the change and lightweight metadata to produce candidate IDs;
2. load only the selected check bodies and matching variants.

The full catalog, full category, unmatched variants, candidate checks, fixtures, and provenance are never loaded into an ordinary coding or review prompt.

### Coding budget

- Zero to four checks.
- Skill-suggested IDs still require applicable evidence.
- If more than four checks appear applicable, prioritize by current risk and defer the remainder to review.
- A no-match result adds no rule context.

### Review budget

- Three to five checks per activated category pass under normal conditions.
- Multiple category passes may run independently and in parallel.
- A category with more applicable checks is split into focused passes rather than receiving a larger prompt.
- The aggregator receives findings and essential metadata, not all check bodies.

### Selection order

Within the applicable set, selection prefers:

1. explicit project policy requirements;
2. domain-skill suggestions supported by current evidence;
3. exact ecosystem variant matches;
4. direct semantic and data-flow trigger matches; and
5. broader category matches.

Popularity, discovery recency, source count, and self-declared severity cannot override applicability or authority.

## Authority and conflicts

The authority ladder is:

1. explicit current human decision and approved product or engineering specification;
2. scoped project policy or accepted architecture decision;
3. verified public contract, schema, or behavioral invariant;
4. matching ecosystem variant;
5. general core check; and
6. discovered candidate.

Higher authority wins only when scopes genuinely overlap. A more specific variant wins over the core only for detection detail, not by weakening the invariant.

Same-authority contradictions are quarantined. Neither side becomes active or produces findings until a maintainer resolves the scope, supersession, or policy boundary. Conflicts are never resolved by load order, recency, popularity, or model preference.

Deduplication treats two checks as duplicates when they require the same failure evidence and lead to the same corrective decision, even when their wording differs.

## Finding contract

A developer-facing finding must include:

```text
Check: <stable ID and variant>
Evidence: <specific code path and observed data/control flow>
Failure: <how the check's condition is satisfied>
Impact: <contextual consequence and blast radius>
Confidence: <what is proven and what remains uncertain>
Severity: <derived from impact and confidence>
Fix direction: <bounded correction>
```

A check defines possible risk; the occurrence determines severity. The same check may be minor in a strictly bounded internal path and critical in a high-cardinality or irreversible production path. A project policy may impose a severity floor in an explicit scope.

Unproven suspicion does not become a finding. The reviewer may request one targeted inspection when missing evidence is obtainable; otherwise it remains silent.

## Candidate lifecycle

```text
discovered
  → candidate
  → normalized and deduplicated
  → conflict-checked
  → inline bad/good examples validated
  → positive and negative fixtures passed
  → shadow
  → active
  → deprecated or superseded
```

Rules:

- Scheduled discovery may create candidates but cannot create active checks.
- Candidates do not appear in the installed catalog index.
- Inline examples teach the check boundary but do not substitute for independent fixtures.
- A positive fixture must demonstrate the defect and required evidence.
- A negative fixture must closely resemble the signal while remaining valid.
- A shadow check cannot create developer-facing findings or alter coding guidance.
- The MVP automatically moves fixture-valid, conflict-free candidates into shadow.
- MVP activation is a batchable maintainer decision informed by shadow evidence.
- Automatic precision-threshold activation is deferred until real-diff labels are trustworthy.
- Deprecated and superseded IDs remain resolvable for historical references and point to their replacement when one exists.

## Precision-first quality target

The system prefers silence over an unproven finding. Quality is demonstrated by:

- no finding without cited failure evidence;
- negative fixtures remaining finding-free;
- no active unresolved same-authority conflict;
- bounded prompt context in every phase;
- disclosed omissions when a budget excludes an applicable check; and
- shadow evidence reviewed before activation.

Recall may improve through better risk inventories, tags, variants, and focused passes. It must not improve by lowering the evidence threshold.

## External tools and privacy

`pragmatic-consulting` may send repository context externally after explicit user invocation or existing project opt-in. `pragmatic-review-advanced` requires explicit user invocation for every run; project policy may preauthorize its model and disclosure scope but cannot invoke it.

Every external path must:

- identify the resolved review model, exact diff target, and files eligible for disclosure before the external call;
- expose only selected diffs, resolved review guidance, and business context required for review;
- exclude secrets, credentials, environment files, unrelated proprietary context, and generated noise;
- preserve skipped files, coverage gaps, warnings, and unresolved context requests as uncertainty;
- avoid automatic installation, provider configuration, credential selection, PR posting, or source modification; and
- leave design decisions and final pragmatic findings under local agent and human ownership.

`pragmatic-consulting` uses an external planning model for design. `pragmatic-review-advanced` uses deterministic local preparation and one external review model internally. Those dependencies do not become public skill identities or transitive requirements for catalog, coding, standalone review, or domain skills. Normal reports remain provider-neutral; dependency diagnostics may name the missing internal capability needed to recover.

## Proposed repository shape

```text
pragmatic-engineering/
├── skills/
│   ├── pragmatic-catalog/
│   │   ├── SKILL.md
│   │   ├── index.json
│   │   ├── categories.md
│   │   ├── checks/<category>/<check>.md
│   │   └── variants/<category>/<check>/<scope>.md
│   ├── pragmatic-consulting/SKILL.md
│   ├── pragmatic-review-advanced/
│   │   ├── SKILL.md
│   │   └── agents/openai.yaml
│   ├── pragmatic-coding/SKILL.md
│   ├── pragmatic-review/SKILL.md
│   ├── decision-framework/SKILL.md
│   └── dependency-seam/SKILL.md
├── candidates/checks/<candidate>.md
├── evals/
│   ├── pragmatic-catalog/<check>/cases.md
│   ├── pragmatic-consulting/cases.md
│   ├── pragmatic-review-advanced/cases.md
│   ├── pragmatic-coding/cases.md
│   └── pragmatic-review/cases.md
└── docs/specs/routed-engineering-checks-v2.md
```

Directories are created only when their first real artifact exists. No empty category, variant, candidate, or evaluation directories are added.

`skills/` remains the canonical installable source for every supported harness. Candidates and promotion fixtures remain outside installed skill directories. Active and shadow check resources live inside `pragmatic-catalog` so that the installed catalog can resolve them without relying on repository-root files.

## Installation and capability behavior

The recommended local system includes `pragmatic-catalog`, `pragmatic-coding`, `pragmatic-review`, and any desired domain skills. `pragmatic-consulting` is installed only where external design consultation is permitted. `pragmatic-review-advanced` is installed only where its private deterministic preparation and review-model dependencies are available; it does not make those capabilities mandatory for standalone review.

Individual behavior remains explicit:

- domain skill alone: complete behavior, no enrichment;
- coding without catalog: continue with disclosure;
- standalone review without catalog: block;
- advanced review without `pragmatic-review`: block;
- advanced review without an internal dependency: block with a recovery diagnostic while standalone review remains available;
- advanced review without proven manual-only host controls: do not advertise or install it for that host;
- consulting without its external model capability: block; and
- catalog alone: lookup capability only, no coding or review action.

Because Agent Skills provide no universal dependency loader, v2 uses convention-based capability handoffs. Deterministic cross-skill loading, generated bundles, and a dedicated CLI remain future options.

## Edge cases

- **A syntax signal is present but behavior is safe:** load the check if relevant, then produce no finding when evidence or the good/allow condition applies.
- **More than four coding checks appear relevant:** select the highest current risks and disclose that review owns the remainder.
- **Many review checks match one category:** split the category into focused passes; do not enlarge one prompt.
- **A domain skill suggests an irrelevant check:** applicability evidence wins; omit the check and record the reason in the selection packet.
- **A framework variant contradicts the core invariant:** quarantine the variant; it cannot weaken the core.
- **A project intentionally accepts a normally risky pattern:** record a scoped policy with rationale; do not alter the reusable check.
- **Two candidates restate one concern:** merge provenance into one candidate rather than growing aliases.
- **A shadow check finds a severe-looking issue:** record shadow evidence only; it cannot emit a developer-facing finding before activation.
- **Catalog is missing during coding:** continue with explicit disclosure and self-contained domain skills.
- **Catalog is missing during pragmatic review:** block and name the required capability.
- **External consultation is not authorized:** skip `pragmatic-consulting`; coding and review remain available.
- **Consultant advice conflicts with project authority:** project authority wins; record the disagreement as a trade-off, not a check conflict.
- **An advanced-review candidate lacks local proof:** reject it or preserve it as uncertainty; do not report it as a pragmatic finding.
- **Advanced preparation selects no reviewable files:** skip the external review call, preserve exclusion reasons, and let standalone pragmatic review handle any separately applicable scope.
- **The advanced reviewer skips a selected file:** require a concrete reason and treat the workflow as partial until pragmatic review covers or explicitly accounts for that file.
- **The diff changes during advanced review:** invalidate the advanced review packet and restart against one stable target.
- **The review session cannot run deterministic preparation:** run preparation in the host and attach exact outputs, diffs, and guidance; do not silently change coverage.
- **Review configuration resolves multiple default models:** block for one explicit model; do not multiply cost implicitly.
- **The model attempts implicit invocation:** host policy must reject it and keep the skill body out of model context.

## Non-goals

- Designing the scheduled discovery runner, cadence, or hosting.
- Building a deterministic AST linter or static-analysis engine in v2.
- Treating syntax signals as sufficient proof for semantic failures.
- Loading the complete catalog or complete categories into one model context.
- Turning every check into an Agent Skill.
- Replacing domain skills with generic checks.
- Making coding depend on catalog availability.
- Letting review silently proceed without its catalog contract.
- Automatically promoting discovered checks directly to active.
- Automatically sending repository files to external models.
- Owning product acceptance, lifecycle state, implementation execution, source control, or deployment.
- Adding a CLI, generated skill bundles, or deterministic cross-skill runtime in the MVP.
- Calling an internally managed LLM review path instead of delegated preparation.
- Building a generic multi-provider review gateway or comparative multi-model review in the MVP.
- Assuming every review backend can execute local binaries.
- Exposing internal dependency names as the public skill name or ordinary report vocabulary.
- Importing external rules, reviewer severities, or candidate findings as trusted catalog content.
- Letting the workflow install tools, configure providers, select credentials, post PR comments, or apply source fixes.
- Allowing prompt similarity, project policy, schedules, subagents, or model initiative to invoke advanced review.

## Acceptance criteria

| ID | Observable criterion | Source |
|---|---|---|
| AC-01 | The repository contains installable `pragmatic-catalog`, `pragmatic-coding`, and `pragmatic-review` skills plus optional `pragmatic-consulting` and manual-only `pragmatic-review-advanced` skills; `skills/` remains the canonical install source. | Current `AGENTS.md`; approved architecture |
| AC-02 | `pragmatic-catalog` exposes the specified input and selection-packet output without performing coding, review, or external consultation. | Approved catalog boundary |
| AC-03 | Every active or shadow check has one stable ID, one primary category, valid phase/tag/trigger/status metadata, and the seven required body sections in order. | Approved check contract |
| AC-04 | Every active or shadow check contains at least one concise bad and one concise good example, and its prose and examples satisfy the stated size targets or document a justified exception. | User-approved linter-like extension |
| AC-05 | Every shadow or active check has independent positive and negative promotion fixtures; inline examples alone cannot satisfy promotion. | Approved promotion gate |
| AC-06 | Each check belongs to one stable failure-domain category; mechanisms and ecosystems are tags or scoped variants, not competing primary categories. | Approved taxonomy model |
| AC-07 | A scoped variant narrows detection and examples without weakening the core evidence or failure condition; true exceptions are policies. | Approved variant contract |
| AC-08 | Catalog selection applies the authority ladder, quarantines unresolved same-authority conflicts, and never resolves conflicts by load order, recency, popularity, or model preference. | Approved conflict policy |
| AC-09 | Coding loads zero to four applicable checks, reports `no pragmatic enrichment required` on no match, and continues with explicit disclosure when catalog enrichment is unavailable. | Approved coding model and missing-catalog behavior |
| AC-10 | Review runs focused category passes with normally three to five checks each, blocks when the catalog is unavailable, treats advanced-review candidates as untrusted inspection input, and aggregates without inventing findings. | Approved review execution and missing-catalog behavior |
| AC-11 | No developer-facing finding is emitted without a stable check ID, cited code evidence, demonstrated failure, contextual impact, confidence, contextual severity, and bounded fix direction. | Approved evidence and severity contracts |
| AC-12 | The full catalog, whole category packs, unmatched variants, candidates, fixtures, and provenance are absent from ordinary coding and review prompts. | Approved routed progressive disclosure |
| AC-13 | Existing and future domain skills remain complete alone and reference only optional catalog capability plus stable IDs, never another skill's filesystem path. | Approved optional enrichment boundary |
| AC-14 | `pragmatic-consulting` runs only after explicit invocation or project opt-in, uses `consult-llm` as an isolated hard dependency, sends minimal non-secret context, and leaves final decisions human-owned. | Approved consultation boundary |
| AC-15 | Discovery output enters as inactive candidates; passing normalization, conflict, example, and fixture gates permits shadow execution but not developer-facing findings. | Approved lifecycle |
| AC-16 | MVP activation is a maintainer decision informed by shadow evidence; automatic precision-threshold promotion is not implemented before trustworthy labels exist. | Precision-first lifecycle decision |
| AC-17 | Pressure evidence proves routing relevance, negative-fixture silence, context budgets, conflict quarantine, coding fail-open disclosure, review fail-closed behavior, external authorization, advanced-review coverage, and candidate normalization. | Repository verification rule; approved architecture |
| AC-18 | Scheduled discovery infrastructure, deterministic AST linting, a new CLI/runtime, generated bundles, and implementation execution remain outside v2 scope. | Explicit non-goals |
| AC-19 | Repository documentation explains the five routed-system skills, capability dependencies, installation combinations, and the distinctions among checks, policies, categories, tags, variants, review candidates, advanced review packets, and findings. | Public interface requirement |
| AC-20 | `AGENTS.md` and the skill registry are updated during implementation so the new evidence gates and isolated external-tool exceptions do not contradict repository instructions. | Current authority reconciliation |
| AC-21 | `pragmatic-review-advanced` is user-invocable but absent from model discovery and implicit invocation on every supported host: `disable-model-invocation: true` for Claude Code, Pi, and OMP, plus Codex `policy.allow_implicit_invocation: false`. | Approved manual-only boundary |
| AC-22 | After explicit invocation, advanced review composes private deterministic preparation, exactly one resolved external review model, and mandatory `pragmatic-review` validation without exposing those dependencies as its public identity. | Approved advanced review boundary |
| AC-23 | Every advanced review packet records model identity, exact target, selected and excluded files, applicable guidance, per-file coverage, warnings, normalized candidates, pragmatic findings, rejected candidates, and unresolved uncertainty while excluding provider chain-of-thought. | Advanced review packet contract |
| AC-24 | An advanced-review candidate cannot inherit final finding status, check mapping, confidence, or severity; those fields require local evidence under `pragmatic-review`. | Precision-first evidence boundary |
| AC-25 | Missing dependencies, authorization failure, target drift, malformed reviewer output, implicit multi-model selection, uncovered files, or unavailable manual-only controls block advanced review while leaving standalone `pragmatic-review` available. | Failure isolation contract |

## Verification stages

### Specification verification

1. Confirm every acceptance criterion is observable and falsifiable.
2. Check all local links.
3. Scan for unresolved placeholders, conflicting terminology, and accidental auto-discovery scope.
4. Verify the module boundaries match current `AGENTS.md` or explicitly identify required reconciliation.
5. Confirm every approved design decision has one normative owner and a traceable acceptance criterion.

### Implementation verification

1. Validate every new skill against the Agent Skills reference implementation.
2. Verify local discovery and installation from `skills/` through each supported package surface.
3. Run positive and negative fixtures for each initial check and variant.
4. Run behavioral pressure cases for routing, budgets, conflicts, missing capabilities, evidence-only findings, and external authorization.
5. Confirm domain skills remain executable when installed without the catalog.
6. Confirm coding does not exceed its selected-check budget.
7. Confirm review category passes do not exceed their per-pass check budget.
8. Confirm shadow checks cannot reach developer-facing findings.
9. Confirm advanced review cannot activate implicitly, from project policy, schedules, subagents, or model initiative on Claude Code, Codex, Pi, and OMP.
10. Confirm explicit user invocation still works on each supported host and the advanced skill body is absent from model context beforehand.
11. Verify deterministic preparation coverage without configuring it with an LLM endpoint.
12. Verify one-model review normalization, including one bounded context-request round.
13. Verify in-session preparation and host-prepared fallback produce equivalent target, guidance, and coverage packets.
14. Verify target drift and uncovered selected files invalidate completion.
15. Verify source severity and category cannot bypass pragmatic evidence, check mapping, or severity derivation.
16. Confirm missing internal dependencies block only advanced review and leave standalone pragmatic review available.
17. Review all changed public documentation and package metadata for one-source install consistency.

## Open questions

None blocking implementation planning. Exact initial check inventory, exact initial scoped variants, advanced-review pressure fixtures, and implementation sequencing belong to the repository-grounded implementation plan.

## Next gate

Repository-grounded implementation planning. This approval does not authorize implementation; implementation begins only after the plan is reviewed and approved.
