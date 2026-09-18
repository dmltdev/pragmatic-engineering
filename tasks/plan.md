# Implementation Plan: Routed Engineering Checks v2

**Status:** Implemented; authenticated Claude Code runtime verification deferred by maintainer on 2026-09-18

## Plan source

- Normative specification: [`docs/specs/routed-engineering-checks-v2.md`](../docs/specs/routed-engineering-checks-v2.md)
- Package boundary and install-source rules: [`AGENTS.md`](../AGENTS.md)
- Current public package contract: [`README.md`](../README.md)
- Current skill index: [`skills/registry.json`](../skills/registry.json)
- Advanced-review source idea: [`docs/ideas/consult-backed-ocr-delegation.md`](../docs/ideas/consult-backed-ocr-delegation.md)
- External contracts: Agent Skills specification and `skills-ref`; Claude Code, Codex, Pi, and OMP manual-invocation controls; `ocr delegate`; `consult-llm --task review`.

The approved specification owns product meaning and architecture. This plan translates it into repository changes without changing its acceptance criteria.

## Repository facts

- `skills/` is the only canonical installable source for Pi, OMP, Claude Code, Codex, and `npx skills`.
- The repository contains Markdown skills and evidence artifacts; it has no application runtime, build script, or committed test harness.
- Existing pressure evidence records isolated baseline/post-skill prompts, exact outputs, binary criteria, model identity, and the evaluated skill hash.
- Plugin manifests already load the repository or `./skills/`; new skill directories do not require a second install source or manifest-specific copies.
- `omp`, `pi`, `codex`, `claude`, `ocr`, `consult-llm`, and `npx` are locally available. `skills-ref` is not installed and must be used from a throwaway environment outside the worktree.
- No prior `tasks/` plan exists, and no prior Memorix project history exists for this package.

## Architecture decisions

1. **Keep v2 skill-only.** Add Markdown skills, JSON catalog metadata, check resources, and pressure evidence. Do not add a CLI, generated bundle, runtime loader, or host-specific orchestration layer.
2. **Start with five core checks.** One check per approved failure-domain category gives end-to-end routing coverage without creating a broad unproved catalog.
3. **Add one scoped variant.** A TypeScript refinement for unobserved promises proves variant resolution and narrowing without introducing a second semantic invariant.
4. **Promote explicitly.** Author checks as `shadow`, run independent positive and negative fixtures, then pause for a maintainer activation decision. Promotion occurs only after that decision and reruns the final-file fixtures.
5. **Use capability handoffs.** Workflow skills refer to `pragmatic-catalog`, `pragmatic-review`, and `consult-llm` capabilities by name and stable contracts, never by another skill's filesystem path.
6. **Keep advanced review generic and manual-only.** Its public surface is `pragmatic-review-advanced`; OpenCodeReview and `consult-llm` remain private implementation dependencies inside the skill body and dependency diagnostics.
7. **Do not change plugin manifests unless discovery proves a defect.** Their current root/`skills/` source configuration already satisfies the one-source invariant. Public descriptions remain sufficiently broad.

## Initial catalog inventory

| Check ID | Phases | Primary proof target |
|---|---|---|
| `state-modeling/no-impossible-state` | design, coding, review | A representation permits an invalid state combination or transition. |
| `concurrency-async/no-unobserved-promise` | coding, review | Started asynchronous work can fail or outlive its owner without observation. |
| `data-access/no-n-plus-one` | design, coding, review | Per-item access causes request/query count to grow with collection cardinality. |
| `boundary-trust/validate-external-data` | design, coding, review | Untrusted external data reaches trusted logic without runtime validation. |
| `failure-recovery/no-silent-fallback` | design, coding, review | A dependency failure is hidden behind an unapproved fallback or default. |

Initial scoped variant:

- `concurrency-async/no-unobserved-promise/typescript`: narrows signals and examples to TypeScript promise expressions, async callbacks, `void`, and explicitly owned fire-and-forget work. It cannot weaken the core evidence requirement.

Every core check starts with `status: shadow`. No candidate or empty directory is created.

## Public, schema, and file contracts

### Catalog index

`skills/pragmatic-catalog/index.json` will contain:

- `schemaVersion`;
- category IDs and category-document location;
- one entry per check with stable ID, relative resource path, category, phases, tags, triggers, status, relationships, and matching variant paths;
- no prompt bodies, fixtures, provenance narratives, or project policy content.

Paths are resolved only by `pragmatic-catalog`. Consumers receive stable IDs and a selection packet.

### Core check file

Each check has required metadata followed by exactly these sections:

1. `Failure`
2. `Signals`
3. `Evidence`
4. `Bad`
5. `Good`
6. `Allow`
7. `Fix`

Prose stays within 180 words excluding metadata and code unless the file records a justified exception. Bad and good examples normally stay within six lines.

### Selection packet

The catalog returns only selected check IDs and bodies, matching variants, selection reasons, applicable policies, quarantined conflicts, and budget omissions. It never performs coding, review, or an external call.

### Advanced review packet

The advanced workflow records review-model identity, exact target and fingerprint, selected and excluded files, per-file reviewed/skipped state, applicable rule groups, warnings, normalized candidates, pragmatic findings, rejected candidates, and unresolved uncertainty. Provider chain-of-thought is excluded.

## Affected modules

- `skills/pragmatic-catalog/`: routing contract, taxonomy, index, checks, and one variant.
- `skills/pragmatic-coding/`: fail-open implementation enrichment.
- `skills/pragmatic-review/`: fail-closed evidence gate and category-pass aggregation.
- `skills/pragmatic-consulting/`: optional external design challenge.
- `skills/pragmatic-review-advanced/`: explicit-only deeper review plus Codex sidecar.
- `skills/dependency-seam/`: optional stable-ID enrichment while retaining standalone behavior.
- `evals/`: catalog fixtures and isolated pressure evidence for every new workflow.
- `skills/registry.json`, `README.md`, `AGENTS.md`, and the superseded idea document: public index, installation combinations, governance, and authority reconciliation.

## Data and migration plan

No persisted data, API, schema, or runtime migration exists. Changes are additive until explicit check activation. Rollback is deletion of the new skill/evaluation directories plus removal of their registry and documentation entries. Existing `decision-framework` and `dependency-seam` standalone behavior must remain usable throughout.

## Permission and security impact

- `pragmatic-consulting` may disclose selected non-secret context only after explicit invocation or project opt-in.
- `pragmatic-review-advanced` requires explicit user invocation every run. Project policy may configure a model and disclosure scope but cannot invoke it.
- Secret, credential, environment, unrelated proprietary, and generated-noise files are excluded before external calls.
- The advanced skill never installs tools, configures providers, selects credentials, posts comments, or edits source.
- Missing authorization or missing private dependencies blocks only the optional external workflow that needs them.

## Telemetry and observability

No production telemetry is added. Durable evidence is repository-local:

- exact pressure prompts and outputs;
- binary criterion scoring;
- resolved model and harness;
- final skill SHA-256;
- per-file advanced-review coverage and warnings;
- explicit skipped, blocked, and unresolved states.

## Implementation slices

The detailed acceptance criteria, file lists, dependencies, and verification steps are in [`tasks/todo.md`](todo.md).

| Task | Slice | Dependency |
|---:|---|---|
| 1 | Catalog contract, empty index, and taxonomy | None |
| 2 | State and concurrency shadow checks with fixtures | 1 |
| 3 | Data-access and boundary shadow checks with fixtures | 2 |
| 4 | Failure-recovery shadow check with fixtures | 3 |
| 5 | TypeScript variant and catalog routing pressure evidence | 4 |
| 6 | Maintainer-approved state/concurrency activation | Human activation decision after Task 5 |
| 7 | Maintainer-approved data/boundary activation | 6 |
| 8 | Maintainer-approved failure-recovery activation | 7 |
| 9 | Coding gate and pressure evidence | 8 |
| 10 | Review gate and pressure evidence | 8 |
| 11 | Consulting gate and pressure evidence | 8 |
| 12 | Manual-only advanced review and pressure evidence | 10 |
| 13 | Optional `dependency-seam` enrichment | 8 |
| 14 | Registry, docs, and governance reconciliation | 9–13 |
| 15 | Cross-host discovery and final acceptance audit | 14 |

Tasks 9, 10, 11, and 13 are independent after Task 8. Task 12 depends on the final `pragmatic-review` contract. One writer must own shared registry and documentation files in Task 14.

## Checkpoints

### Checkpoint A — shadow catalog

After Task 5:

- every core check and variant passes structural validation;
- every positive fixture produces the required evidence shape;
- every similar negative fixture stays finding-free;
- routing, budgets, conflicts, candidates, and shadow silence have recorded pressure evidence;
- a maintainer reviews the evidence and explicitly decides which checks may become active.

Implementation must not perform Tasks 6–8 before that decision.

### Checkpoint B — routed workflows

After Tasks 9–13:

- coding remains fail-open without catalog and never loads more than four checks;
- review remains fail-closed without catalog and cannot invent findings;
- consulting remains design-only and authorization-gated;
- advanced review remains explicit-only, coverage-checked, and locally validated;
- `dependency-seam` still works when installed alone.

### Checkpoint C — package complete

After Task 15:

- all supported surfaces discover the canonical `skills/` source;
- manual-only controls pass on Claude Code, Codex, Pi, and OMP;
- all 25 acceptance criteria have observed evidence;
- no empty directories, temporary fixtures, generated copies, or validation environments remain in the worktree.

## Verification commands and scenarios

Run commands from `pragmatic-engineering/` unless stated otherwise.

### Structural validation

Use the official `skills-ref` project in a throwaway directory outside the worktree, following its documented virtual-environment installation, then run:

```bash
skills-ref validate skills/pragmatic-catalog
skills-ref validate skills/pragmatic-coding
skills-ref validate skills/pragmatic-review
skills-ref validate skills/pragmatic-consulting
```

The canonical advanced skill intentionally carries the required host extensions `disable-model-invocation` and `user-invocable`, which the base Agent Skills schema does not define. Copy that skill to a throwaway directory, remove only those two extension fields from the copy, and run `skills-ref validate` on the portable projection. Separately verify that the canonical file still contains both fields and that every supported host enforces the manual-only boundary.

Also parse `skills/pragmatic-catalog/index.json`, verify every indexed relative path exists, enforce unique IDs, enforce one primary category, check section order and prose/example budgets, and reject dangling or weakening variants with a throwaway audit script. Remove the script after the audit.

### Local discovery and installation

```bash
npx skills add . --list
```

From a temporary directory outside the repository, copy-install the complete package for all supported agents:

```bash
npx skills add /absolute/path/to/pragmatic-engineering --skill '*' --agent '*' --copy -y --json
```

Confirm the installed skill set contains the five routed-system skills and the two existing skills, all sourced from `skills/`.

### Catalog and workflow pressure evidence

Use the existing isolated baseline/post-skill method. Record exact prompts, outputs, binary scoring, model/harness identity, and final skill hash. Required scenarios include:

- relevant routing and no-match silence;
- four-check coding cap and deferred remainder;
- category split at more than five review checks;
- same-authority conflict quarantine;
- candidate normalization and shadow non-emission;
- coding fail-open versus review fail-closed;
- external authorization and minimal disclosure;
- unsupported advanced candidate rejected by pragmatic validation.

### Advanced-review dependency proof

Against a disposable Git fixture repository:

```bash
ocr delegate preview --repo <fixture> --from <base> --to <head> --format json
ocr delegate rule --repo <fixture> --from <base> --to <head> --format json <selected-paths...>
consult-llm --task review --model <approved-model> --prompt-file <normalized-packet>
```

Verify that OCR preparation succeeds without an OCR-managed LLM endpoint, one resolved external model reviews the normalized packet, every selected file is reviewed or skipped with a reason, one bounded context request is supported, target drift invalidates the packet, and local `pragmatic-review` independently accepts or rejects every candidate.

### Manual-only host proof

For Claude Code, Pi, and OMP, verify `disable-model-invocation: true` hides the advanced skill from model discovery while `/skill:pragmatic-review-advanced` still works. For Codex, verify `agents/openai.yaml` sets `policy.allow_implicit_invocation: false`, implicit selection fails, and explicit `$pragmatic-review-advanced` works. Run negative prompts that resemble an advanced review request, project-policy preconfiguration, scheduled-discovery text, subagent instructions, and model initiative; none may load the skill body.

## Acceptance-to-evidence map

| Acceptance criteria | Planned evidence |
|---|---|
| AC-01 | Tasks 9–12 create five routed skills; Tasks 14–15 prove canonical discovery and installation. |
| AC-02 | Tasks 1 and 5 validate catalog-only input/output and absence of coding, review, or external calls. |
| AC-03–AC-06 | Tasks 2–5 run metadata, section-order, size, example, fixture, and taxonomy audits. |
| AC-07 | Task 5 tests the TypeScript variant against narrowing and invariant-preservation cases. |
| AC-08 | Tasks 1 and 5 record authority-order and same-authority quarantine pressure cases. |
| AC-09 | Task 9 records zero-to-four selection, no-match silence, and missing-catalog disclosure. |
| AC-10–AC-12 | Task 10 records category budgets, fail-closed catalog behavior, evidence-only aggregation, and prompt-content exclusion. |
| AC-13 | Task 13 proves `dependency-seam` standalone behavior and optional stable-ID enrichment without path coupling. |
| AC-14 | Task 11 proves explicit/project-approved consultation, isolated dependency failure, minimal disclosure, and human ownership. |
| AC-15–AC-16 | Tasks 2–8 prove candidate/shadow gates and require an evidence-informed maintainer activation decision. |
| AC-17 | Tasks 2–5 and 9–12 create the required pressure evidence; Task 15 audits its completeness. |
| AC-18 | Task 15 verifies absence of a CLI/runtime, generated bundles, scheduled runner, and implementation execution. |
| AC-19–AC-20 | Task 14 updates public documentation, governance, registry, and the superseded idea status. |
| AC-21 | Tasks 12 and 15 prove host-specific implicit-invocation blocking and explicit invocation. |
| AC-22–AC-23 | Task 12 proves private preparation, one-model review, packet completeness, and mandatory pragmatic validation. |
| AC-24 | Tasks 10 and 12 prove candidates cannot inherit check mapping, confidence, severity, or finding status. |
| AC-25 | Task 12 exercises each blocking condition while Task 15 confirms standalone review remains available. |

## Risks and rollback

| Risk | Impact | Mitigation / rollback |
|---|---|---|
| Agent Skills has no portable dependency loader. | A workflow may assume a missing capability. | Every skill defines explicit block/fail-open behavior; installation combinations are documented and pressure-tested. |
| Manual-only metadata differs by host. | Advanced review could enter model context implicitly. | Use host-native controls, negative activation tests, and refuse installation where the control cannot be proved. Delete only the optional advanced skill if a host regresses. |
| Check prose becomes broad or noisy. | False positives and prompt bloat. | One failure mode per check, hard evidence threshold, budgets, negative fixtures, and explicit shadow promotion. Revert status to `shadow` without deleting history. |
| External tool output changes. | Coverage or normalization may become unsound. | Validate JSON shape and target fingerprint before model use; block advanced review on mismatch while standalone review remains available. |
| Provider context leaks. | Repository confidentiality risk. | Pre-call allowlist, secret exclusions, exact eligible-file record, explicit authorization, and no automatic external call. |
| Eval evidence becomes stale after edits. | Claimed behavior no longer matches shipped skill text. | Record final hashes and rerun affected cases after every skill-body or status change. |

## Detected conflicts and reconciliation

- `AGENTS.md` currently excludes host-specific workflow orchestration. The approved specification creates one narrow external-tool exception for advanced review. Task 14 updates the package boundary without broadening it to generic orchestration.
- The base Agent Skills standard has no manual-only field. The approved host-specific metadata is therefore required and tested; unsupported hosts must not advertise the advanced skill.
- `skills-ref` is not installed locally. This is a verification prerequisite, not a product dependency; use its official throwaway installation and leave no repository artifact.
- QMD is indexed to an unrelated repository and was not used as authority. Memorix was rebound to this package and reported no prior project history.

No unresolved semantic conflict blocks implementation.

## Ready for implementation

**Ready for `implement-feature`: yes. Human approval was recorded in this conversation.**

Implementation must stop at Checkpoint A for the evidence-informed maintainer activation decision before changing any initial check from `shadow` to `active`.
