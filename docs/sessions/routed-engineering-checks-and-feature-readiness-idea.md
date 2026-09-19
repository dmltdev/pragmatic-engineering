# Routed Engineering Checks and Feature Readiness Idea

**Date:** 2026-09-19

## TL;DR

This session completed, committed, and pushed the routed engineering checks implementation to `origin/main` as commit `50346f4468ba6784c47716c93fd2731274ccd654`. It also captured an uncommitted idea for project-specific feature-readiness checklist creation and evaluation. The evaluator name and execution boundary are now accepted; resolve checklist storage before implementing the two proposed skills.

## Decisions

### Keep one canonical installable skill source

- **Chose:** Keep `skills/` as the canonical source for pi, OMP, Claude Code, and Codex.
- **Options considered:** One shared source / harness-specific copies.
- **Trade-offs:** A shared source avoids drift and duplicate maintenance, but runtime-specific behavior must remain explicit and cannot be claimed without runtime evidence.

### Route bounded checks instead of loading one large checklist

- **Chose:** Use a routed catalog with stable IDs, category and variant selection, strict loading budgets, and evidence-only findings.
- **Options considered:** Routed catalog / load every check / embed checks in each workflow.
- **Trade-offs:** Routing controls context size and noise, but it requires registry integrity, missing-capability behavior, and pressure evidence that proves irrelevant checks remain unloaded.

### Keep advanced review manual-only

- **Chose:** Allow `pragmatic-review-advanced` as the sole external-tool workflow exception, keep provider tooling private, and leave final findings to `pragmatic-review`.
- **Options considered:** Automatic external review / manual-only review / no advanced review path.
- **Trade-offs:** Manual invocation limits accidental disclosure and ownership drift, but users must opt in and configure the provider explicitly.

### Defer unobserved Claude runtime verification explicitly

- **Chose:** Retain canonical Claude controls while recording Claude runtime verification as deferred.
- **Options considered:** Remove Claude support / claim parity without evidence / retain controls with an explicit verification gap.
- **Trade-offs:** The repository remains structurally ready for Claude, but runtime parity is not proven until the deferred scenario is exercised.

### Separate checklist authoring from readiness evaluation

- **Chose:** Propose `create-feature-checklist` for durable project criteria and `check-feature-readiness` for evidence-backed evaluation.
- **Options considered:** One combined skill / two complementary skills / generic static checklist only.
- **Trade-offs:** Separate responsibilities keep authoring and evaluation clear and reusable, but their shared schema and execution boundary must be decided before implementation.

### Keep readiness evaluation evidence-only

- **Chose:** Keep `check-feature-readiness` as the public evaluator name. It assesses supplied or already-observed evidence and does not execute repository verification tools or commands.
- **Options considered:** `check-feature-readiness` / `is-feature-ready` / audit or trace names; evidence-only evaluation / a bounded command runner.
- **Trade-offs:** `check-feature-readiness` describes a status-based report without implying a boolean acceptance verdict. Evidence-only evaluation preserves the package boundary and traceability, but fresh verification must occur outside this skill and be supplied as evidence.

## Discussion

- The canonical repository and plugin name is `pragmatic-engineering`; there is no separate `pragmatic-engineer` directory.
- The readiness idea uses common frontend, backend, integration, data, asynchronous-processing, operations, and platform-specific baselines only as fallbacks. Repository evidence controls actual tools, thresholds, environments, and owners.
- On first checklist creation, the workflow should reuse an existing `docs/features/checklists/` or `docs/checklists/` convention. If neither exists, it should ask once and persist the chosen path by creating it.
- Project-specific proof methods can differ: one application may use Grafana, while another may use k6 or Artillery. These are examples, not defaults.
- Exact-output fixtures in `evals/pragmatic-coding/cases.md` contain intentional two-space Markdown hard breaks. `git show --check` reports these as trailing whitespace even though they preserve fixture formatting.

### Stable facts

- `skills/` is the only approved installable source for all supported harnesses.
- Routed checks must preserve catalog budgets, stable IDs, shadow and candidate isolation, evidence-only findings, and documented missing-capability behavior.
- Product acceptance, deployment tooling, session tooling, and host-specific orchestration remain outside this plugin's package boundary.
- `pragmatic-review-advanced` is manual-only and cannot transfer final-finding ownership away from `pragmatic-review`.

## Accomplished

- Added and verified the routed catalog, coding, review, consulting, and manual advanced-review workflows.
- Added activation and pressure evidence for the routed workflows and enriched dependency-seam routing.
- Reconciled package documentation, governance, registry data, task records, and the routed-engineering specification.
- Removed the obsolete global `implement` skill from supported harness paths through a clean cutover.
- Closed all review findings and completed all 16 implementation tasks.
- Committed the implementation as `50346f4468ba6784c47716c93fd2731274ccd654` and pushed it to `origin/main`.
- Saved `docs/ideas/feature-readiness-checklists.md` with the proposed authoring and evaluation workflows.
- Resolved and received approval for the readiness evaluator name and execution boundary.

## Unfinished / Next Steps

1. [high] Decide whether checklist storage uses reusable project profiles, per-feature instances, or both, and choose a default path only if the user wants one.
2. If implementation is approved, create both skills under `skills/` and verify them through local discovery and pressure evidence as required by `AGENTS.md`.
3. Run the explicitly deferred Claude Code runtime verification for the routed workflows.
4. Commit the idea note and this session summary when their content is accepted.

## Files Changed

- `AGENTS.md` - records package boundaries and routed-check invariants.
- `README.md` - documents the published workflows and usage.
- `docs/specs/routed-engineering-checks-v2.md` - source specification for the routed implementation.
- `docs/ideas/consult-backed-ocr-delegation.md` - related future workflow idea updated in the pushed commit.
- `docs/ideas/iterative-check-discovery.md` - related future workflow idea updated in the pushed commit.
- `docs/ideas/feature-readiness-checklists.md` - uncommitted idea for project-specific readiness checklist creation and evaluation.
- `docs/sessions/routed-engineering-checks-and-feature-readiness-idea.md` - this uncommitted continuation handoff.
- `skills/dependency-seam/` - enriched dependency-seam guidance and routing.
- `skills/pragmatic-catalog/` - catalog, stable check definitions, registry, and variants.
- `skills/pragmatic-coding/`, `skills/pragmatic-consulting/`, `skills/pragmatic-review/`, `skills/pragmatic-review-advanced/` - routed workflow skills.
- `evals/` - activation, pressure, and behavior evidence for the affected skills.
- `tasks/plan.md`, `tasks/todo.md` - completed implementation plan and task state, including deferred Claude runtime verification.
