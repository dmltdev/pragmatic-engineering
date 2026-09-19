# Feature readiness checklists

Status: idea

## Purpose

Define two complementary skills:

- `create-feature-checklist` creates and maintains a project-specific source of truth for feature-readiness checks.
- `check-feature-readiness` evaluates a feature against the applicable project checklist and reports evidence, gaps, and blockers.

The generic checklists provide a safe baseline. Project-specific checklist entries define the actual tools, thresholds, environments, evidence, and ownership used by that repository.

## Target users

Engineers and agents preparing a feature for review, release, or acceptance in repositories whose readiness criteria differ by stack and operating environment.

## Trigger

Use `create-feature-checklist` when a repository has no durable feature-readiness checklist, when its verification tooling changes, or when a new checklist type is needed.

Use `check-feature-readiness` when a feature needs a release, review, or acceptance gate grounded in supplied or already-observed evidence.

## Boundary

`create-feature-checklist` authors readiness criteria. It does not evaluate a feature or implement missing work.

`check-feature-readiness` evaluates supplied or already-observed evidence. It does not execute repository commands or tools, implement fixes, invent acceptance criteria, select local proof methods, or mark missing evidence as passed.

Each `pass` traces to an applicable checklist condition and qualifying evidence. Missing, inaccessible, stale, ambiguous, or unmapped evidence remains `unverified` or `blocked`.

Product acceptance remains human-owned. Existing repository policy, specifications, and domain rules remain authoritative over generic checklist defaults.

## First-run location

On first use, `create-feature-checklist` inspects existing repository conventions and documentation. It selects one established location when present:

- `docs/features/checklists/`
- `docs/checklists/`

If neither location is established, it asks the user once for the repository preference. Creating the selected path persists that convention. Later runs reuse the existing location rather than asking again or creating a competing structure.

## Common frontend baseline

- Accessibility: semantic structure, keyboard and focus behavior, contrast, labels, announcements, and reduced-motion behavior where relevant.
- Performance: explicit budgets and evidence for loading, rendering, interaction latency, network use, and asset size.
- State management: clear ownership, server/client boundaries, valid transitions, loading/error/empty states, persistence, and stale-data behavior.
- User experience: responsive behavior, supported browsers, forms, error recovery, localization, and destructive-action safeguards.
- Security and operations: untrusted input, sensitive data, authorization-visible states, observability, rollout, and rollback.

## Common backend baseline

- Contracts: API or event schemas, validation, compatibility, error semantics, pagination, and idempotency where required.
- Security: authentication, authorization, secrets, untrusted input, privacy, audit needs, and dependency boundaries.
- Data: schema changes, migrations, transactions, integrity, retention, backfills, and rollback behavior.
- Performance and reliability: capacity, bounded queries, caching, timeouts, retries, concurrency, background jobs, and failure recovery.
- Operations: structured telemetry, dashboards and alerts, deployment sequencing, configuration, runbooks, and rollback proof.

## Additional checklist types

Select only types justified by the feature and repository:

- Full-stack and integration: end-to-end contracts, cross-service behavior, compatibility, and ownership at boundaries.
- Data and storage: migrations, backfills, integrity, retention, privacy, and recovery.
- Asynchronous processing: delivery guarantees, retries, idempotency, ordering, dead letters, and operator recovery.
- Release and operations: configuration, observability, capacity, rollout, rollback, support, and incident readiness.
- Platform-specific UI: mobile, desktop, offline, permissions, device constraints, or store requirements when applicable.

## Project-specific derivation

`create-feature-checklist` derives concrete checks from repository evidence: documentation, scripts, test configuration, deployment configuration, observability setup, and user-provided operating knowledge.

Each project entry maps a concern to the local proof method. For example, one application may verify production performance through Grafana, while another may use k6 or Artillery. Generic tool examples never override repository evidence.

The skill must not invent tools, thresholds, owners, environments, or commands. Missing information remains explicit and can be resolved by the user.

## Core output

The durable project checklist records:

- applicable checklist type and concern;
- project-specific acceptance condition;
- verification method, tool, environment, or command;
- required evidence and evidence location;
- owner or approver when the repository defines one.

The readiness report classifies each applicable item as `pass`, `fail`, `blocked`, `not applicable`, or `unverified`. Each result cites the checklist condition and observed evidence or the explicit reason that evidence is unavailable. It includes the unmet condition and the next owner or action when known.

## Verification

- First use follows an existing repository convention or records the user's chosen checklist location.
- Later runs reuse that location and update the established checklist instead of creating a second convention.
- Frontend, backend, and justified additional checklist types can be selected independently.
- Project-specific proof methods override generic examples; Grafana, k6, and Artillery are examples, not defaults.
- Missing proof remains `unverified` or `blocked`; it never becomes an inferred pass.
- Readiness output traces every status to a checklist condition and observed evidence.

## Accepted decisions

- **Evaluator name:** `check-feature-readiness`. It describes a status-based evidence report. `is-feature-ready` implies a boolean verdict and understates the report.
- **Execution boundary:** Evidence-only. The evaluator reads supplied or already-observed evidence, including captured command output and CI artifacts. It does not run repository verification tools or commands.
- **Rationale:** This preserves evidence-only findings and keeps deployment tooling and host-specific workflow orchestration outside the package boundary.

- **Baseline catalog:** Versioned baseline references live inside `skills/create-feature-checklist/`. The authoring skill uses them as fallbacks only. The generated project checklist remains the repository-specific source of truth.

## Open questions

- When neither path exists and the user has no preference, should one directory be the documented default?
- Should repositories maintain reusable readiness profiles, per-feature checklist instances, or both?

## Source

Captured from the requested workflow for reusable common frontend, backend, and optional readiness checklists that become project-specific through repository evidence and a persisted local directory convention.
