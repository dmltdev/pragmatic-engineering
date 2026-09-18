# Routed Engineering Checks v2 — Implementation Tasks

Plan: [`plan.md`](plan.md)

## Phase 1 — Catalog foundation

### Task 1: Build the catalog contract

**Description:** Create the portable catalog skill, empty versioned index, and category ownership document. The catalog owns selection and conflict resolution only.

**Acceptance criteria:**
- [x] Catalog input and selection-packet output match the approved specification.
- [x] Index schema supports stable IDs, routing metadata, status, relationships, and variant paths without embedding full check bodies.
- [x] Categories define exactly the five approved failure domains and one pass owner per check.

**Verification:**
- [x] `skills-ref validate skills/pragmatic-catalog` passes.
- [x] `index.json` parses and contains no dangling path while empty.
- [x] Catalog text contains no implementation, review, or external-model execution path.

**Dependencies:** None

**Files likely touched:**
- `skills/pragmatic-catalog/SKILL.md`
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/categories.md`

**Estimated scope:** Medium: 3 files

### Task 2: Add state and concurrency shadow checks

**Description:** Add `state-modeling/no-impossible-state` and `concurrency-async/no-unobserved-promise` as indexed shadow checks with independent positive and similar negative fixtures.

**Acceptance criteria:**
- [x] Both checks have valid metadata and the seven required sections in order.
- [x] Each check proves one semantic failure and stays silent on its similar safe case.
- [x] The catalog index resolves both files as `shadow` without exposing fixtures.

**Verification:**
- [x] Run isolated positive and negative pressure cases and record exact outputs, scoring, model, harness, and hashes.
- [x] Check prose and examples satisfy the size budgets.
- [x] Confirm shadow results cannot become developer-facing findings.

**Dependencies:** Task 1

**Files likely touched:**
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/checks/state-modeling/no-impossible-state.md`
- `evals/pragmatic-catalog/state-modeling/no-impossible-state/cases.md`
- `skills/pragmatic-catalog/checks/concurrency-async/no-unobserved-promise.md`
- `evals/pragmatic-catalog/concurrency-async/no-unobserved-promise/cases.md`

**Estimated scope:** Medium: 5 files

### Task 3: Add data and boundary shadow checks

**Description:** Add `data-access/no-n-plus-one` and `boundary-trust/validate-external-data` as indexed shadow checks with independent fixtures.

**Acceptance criteria:**
- [x] Query cardinality, not loop syntax alone, is required for the N+1 finding.
- [x] External trust crossing without runtime validation is required for the boundary finding.
- [x] Similar bounded access and already-validated data remain finding-free.

**Verification:**
- [x] Record isolated positive and negative evidence for both checks.
- [x] Check metadata, section order, prose budget, and example budget.
- [x] Confirm index paths and one-category ownership.

**Dependencies:** Task 2

**Files likely touched:**
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/checks/data-access/no-n-plus-one.md`
- `evals/pragmatic-catalog/data-access/no-n-plus-one/cases.md`
- `skills/pragmatic-catalog/checks/boundary-trust/validate-external-data.md`
- `evals/pragmatic-catalog/boundary-trust/validate-external-data/cases.md`

**Estimated scope:** Medium: 5 files

### Task 4: Add failure-recovery shadow check

**Description:** Add `failure-recovery/no-silent-fallback` as an indexed shadow check with explicit approved-fallback and hidden-failure boundaries.

**Acceptance criteria:**
- [x] The check requires a hidden dependency failure and unapproved observable fallback.
- [x] Explicitly approved fallback policy with preserved diagnostics remains valid.
- [x] Index and fixture paths resolve without loading fixture content into catalog prompts.

**Verification:**
- [x] Record isolated positive and negative pressure evidence.
- [x] Check metadata, section order, prose budget, and example budget.
- [x] Confirm shadow non-emission.

**Dependencies:** Task 3

**Files likely touched:**
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/checks/failure-recovery/no-silent-fallback.md`
- `evals/pragmatic-catalog/failure-recovery/no-silent-fallback/cases.md`

**Estimated scope:** Medium: 3 files

### Task 5: Add TypeScript variant and catalog pressure evidence

**Description:** Add the TypeScript refinement for unobserved promises and prove catalog routing, budgets, authority, conflict quarantine, candidate normalization, and shadow silence.

**Acceptance criteria:**
- [x] The variant narrows TypeScript detection and examples without weakening core evidence.
- [x] Catalog selection loads only matched checks/variants and records reasons, policy, conflicts, and omissions.
- [x] Candidate and shadow inputs cannot produce developer-facing findings.

**Verification:**
- [x] Run positive and negative variant fixtures.
- [x] Run general catalog pressure cases for routing, no match, budgets, conflicts, candidates, and status behavior.
- [x] Audit every index path and reject a deliberately weakening variant fixture.

**Dependencies:** Task 4

**Files likely touched:**
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/variants/concurrency-async/no-unobserved-promise/typescript.md`
- `evals/pragmatic-catalog/concurrency-async/no-unobserved-promise/typescript/cases.md`
- `evals/pragmatic-catalog/cases.md`

**Estimated scope:** Medium: 4 files

## Checkpoint A — Shadow evidence and activation

- [x] All five core checks and the TypeScript variant pass structural and behavioral fixtures.
- [x] Negative fixtures remain finding-free.
- [x] Shadow checks remain unable to affect coding guidance or final findings.
- [x] Maintainer reviews recorded evidence and explicitly selects checks for activation.

Do not start Tasks 6–8 without the explicit maintainer decision.

### Task 6: Activate approved state and concurrency checks

**Description:** Apply the maintainer decision to the first two checks, update the index, and rerun final-file evidence after status changes.

**Acceptance criteria:**
- [x] Only explicitly approved checks become `active` in both file metadata and index.
- [x] Final hashes in evidence match the activated files.
- [x] Unapproved checks, if any, remain `shadow` without special casing.

**Verification:**
- [x] Rerun both positive and negative fixture pairs.
- [x] Audit index/file status consistency.

**Dependencies:** Task 5 and explicit maintainer activation decision

**Files likely touched:**
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/checks/state-modeling/no-impossible-state.md`
- `evals/pragmatic-catalog/state-modeling/no-impossible-state/cases.md`
- `skills/pragmatic-catalog/checks/concurrency-async/no-unobserved-promise.md`
- `evals/pragmatic-catalog/concurrency-async/no-unobserved-promise/cases.md`

**Estimated scope:** Medium: 5 files

### Task 7: Activate approved data and boundary checks

**Description:** Apply the maintainer decision to the data-access and boundary checks and refresh their final evidence.

**Acceptance criteria:**
- [x] File and index statuses match the maintainer decision.
- [x] Final hashes and exact outputs match the activated text.
- [x] Evidence thresholds remain semantic after activation.

**Verification:**
- [x] Rerun both positive and negative fixture pairs.
- [x] Audit index/file status consistency.

**Dependencies:** Task 6

**Files likely touched:**
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/checks/data-access/no-n-plus-one.md`
- `evals/pragmatic-catalog/data-access/no-n-plus-one/cases.md`
- `skills/pragmatic-catalog/checks/boundary-trust/validate-external-data.md`
- `evals/pragmatic-catalog/boundary-trust/validate-external-data/cases.md`

**Estimated scope:** Medium: 5 files

### Task 8: Activate approved failure-recovery check

**Description:** Apply the maintainer decision to the failure-recovery check and refresh its final evidence.

**Acceptance criteria:**
- [x] File and index status match the maintainer decision.
- [x] Final hash and exact outputs match the activated text.
- [x] Approved fallback policy remains a negative case.

**Verification:**
- [x] Rerun the positive and negative fixture pair.
- [x] Audit complete index/file status consistency.

**Dependencies:** Task 7

**Files likely touched:**
- `skills/pragmatic-catalog/index.json`
- `skills/pragmatic-catalog/checks/failure-recovery/no-silent-fallback.md`
- `evals/pragmatic-catalog/failure-recovery/no-silent-fallback/cases.md`

**Estimated scope:** Medium: 3 files

## Phase 2 — Routed workflows

### Task 9: Build the coding enrichment gate

**Description:** Add `pragmatic-coding` as a risk-triggered, zero-to-four-check implementation enrichment that fails open with explicit disclosure when catalog capability is absent.

**Acceptance criteria:**
- [x] Low-risk changes add no pragmatic context; non-trivial changes receive at most four applicable checks.
- [x] No match returns exactly `no pragmatic enrichment required`.
- [x] Missing catalog preserves self-contained domain skills and reports `catalog enrichment unavailable`.

**Verification:**
- [x] Record pressure cases for low risk, exact match, over-budget selection, no match, and missing catalog.
- [x] `skills-ref validate skills/pragmatic-coding` passes.

**Dependencies:** Task 8

**Files likely touched:**
- `skills/pragmatic-coding/SKILL.md`
- `evals/pragmatic-coding/cases.md`

**Estimated scope:** Small: 2 files

### Task 10: Build the pragmatic review gate

**Description:** Add `pragmatic-review` as a catalog-required, category-pass review that independently proves every final finding and treats advanced observations as untrusted candidates.

**Acceptance criteria:**
- [x] Missing catalog blocks review; category passes normally load three to five checks and split when larger.
- [x] Every final finding follows the stable evidence contract and is backed by one pass.
- [x] Aggregation deduplicates but cannot invent findings or inherit advanced severity/confidence.

**Verification:**
- [x] Record pressure cases for missing catalog, category splitting, unsupported suspicion, deduplication, and advanced-candidate rejection.
- [x] `skills-ref validate skills/pragmatic-review` passes.

**Dependencies:** Task 8

**Files likely touched:**
- `skills/pragmatic-review/SKILL.md`
- `evals/pragmatic-review/cases.md`

**Estimated scope:** Small: 2 files

### Task 11: Build the consulting design gate

**Description:** Add `pragmatic-consulting` as an optional design-only workflow using `consult-llm --task plan`, local design-phase checks, and human-owned decisions.

**Acceptance criteria:**
- [x] It runs only after explicit invocation or project opt-in and sends only authorized, relevant, non-secret context.
- [x] Missing `consult-llm` blocks consultation without affecting catalog, coding, review, or domain skills.
- [x] Output contains alternatives, trade-offs, unresolved facts, recommendation, and human-owned decisions.

**Verification:**
- [x] Record pressure cases for authorization, minimal disclosure, one context request, dependency failure, and consultant/project-authority disagreement.
- [x] `skills-ref validate skills/pragmatic-consulting` passes.

**Dependencies:** Task 8

**Files likely touched:**
- `skills/pragmatic-consulting/SKILL.md`
- `evals/pragmatic-consulting/cases.md`

**Estimated scope:** Small: 2 files

### Task 12: Build manual-only advanced review

**Description:** Add the generic `pragmatic-review-advanced` public skill, Codex sidecar, private OCR/consult preparation, normalized coverage packet, and mandatory pragmatic validation.

**Acceptance criteria:**
- [x] Pi and OMP frontmatter blocks model invocation; the Codex sidecar blocks implicit invocation; explicit invocation remains available. Claude Code retains the same canonical controls, but its authenticated runtime probe is deferred by maintainer decision.
- [x] One stable target produces complete selected/excluded/per-file coverage with exactly one resolved external model.
- [x] Dependency, authorization, schema, coverage, target-drift, and multi-model failures block only advanced review.

**Verification:**
- [x] Run host-specific implicit-negative and explicit-positive invocation cases for the current delivery scope: OMP, Pi, and Codex. Claude Code runtime is deferred.
- [x] Run actual `ocr delegate preview/rule --format json` against a disposable Git fixture without an OCR LLM endpoint.
- [x] Run one authorized `consult-llm --task review` flow and prove local `pragmatic-review` accepts or rejects each candidate independently.

**Dependencies:** Task 10

**Files likely touched:**
- `skills/pragmatic-review-advanced/SKILL.md`
- `skills/pragmatic-review-advanced/agents/openai.yaml`
- `evals/pragmatic-review-advanced/cases.md`

**Estimated scope:** Medium: 3 files

### Task 13: Add optional domain-skill enrichment

**Description:** Extend `dependency-seam` with optional stable check suggestions while preserving its complete standalone contract and existing evidence.

**Acceptance criteria:**
- [x] Enrichment names only `pragmatic-catalog`, stable IDs, and evidence conditions; it references no skill filesystem path.
- [x] Missing catalog leaves the existing dependency-seam decision unchanged.
- [x] Irrelevant suggested IDs are omitted by catalog applicability.

**Verification:**
- [x] Rerun existing dependency-seam pressure cases after the edit.
- [x] Add standalone-no-catalog and optional-enrichment cases with final skill hash.

**Dependencies:** Task 8

**Files likely touched:**
- `skills/dependency-seam/SKILL.md`
- `evals/dependency-seam/cases.md`

**Estimated scope:** Small: 2 files

## Checkpoint B — Workflow behavior

- [x] Coding is bounded and fail-open.
- [x] Review is evidence-only and fail-closed.
- [x] Consulting is authorization-gated and design-only.
- [x] Advanced review is explicit-only, coverage-checked, and isolated.
- [x] Existing domain skills remain independently usable.

## Phase 3 — Package integration

### Task 14: Reconcile registry, docs, and governance

**Description:** Publish the five routed-system skills and capability combinations, reconcile the narrow external-tool exception, and mark the source idea as superseded by the approved generic skill design.

**Acceptance criteria:**
- [x] Registry indexes every skill and evaluation document without becoming a second install source.
- [x] README explains the five skills, install combinations, capability failures, and domain terms.
- [x] AGENTS preserves the package boundary while requiring local discovery and pressure evidence for the isolated external workflow.

**Verification:**
- [x] Check every local documentation and registry path.
- [x] Confirm manifests still point only at the repository/`skills/` source.
- [x] Scan public descriptions for stale provider-branded skill names and accidental auto-invocation language.

**Dependencies:** Tasks 9–13

**Files likely touched:**
- `skills/registry.json`
- `README.md`
- `AGENTS.md`
- `docs/ideas/consult-backed-ocr-delegation.md`

**Estimated scope:** Medium: 4 files

### Task 15: Run cross-host discovery and acceptance audit

**Description:** Validate the final repository through the official reference implementation, local install surfaces, host invocation controls, pressure evidence, and all 25 acceptance criteria.

**Acceptance criteria:**
- [x] All seven skills install from canonical `skills/` through supported surfaces; the four portable routed skills validate directly and the manual-only advanced skill validates through a throwaway projection with only its required host extensions removed.
- [x] Every in-scope acceptance criterion has observed evidence with no unresolved semantic conflict. The authenticated Claude Code portion of AC-21 is explicitly deferred.
- [x] Worktree contains no empty directories, generated copies, temporary fixtures, validation environments, or implementation outside v2 scope.

**Verification:**
- [x] Run `skills-ref validate` for the four portable new skills and for a throwaway advanced-skill projection that removes only the two required host-extension fields.
- [x] Run `npx skills add . --list` and a temporary all-agent copy install.
- [x] Run the in-scope host manual-only matrix and a throwaway acceptance audit over IDs, paths, metadata, budgets, links, and evidence hashes.

**Dependencies:** Task 14

**Files likely touched:** None unless a failed gate exposes a source defect; fix the owning task rather than weakening the gate.

**Estimated scope:** Medium: verification-only

## Checkpoint C — Ready for review

- [x] All acceptance criteria in the current delivery scope are satisfied; authenticated Claude Code runtime proof remains deferred.
- [x] Final pressure evidence hashes match shipped skill files.
- [x] Standalone existing skills still pass their pressure cases.
- [x] Public documentation exposes generic skills, not private provider tooling.
- [x] Adversarial review is complete with no unresolved implementation or evidence finding. The maintainer accepted the Claude Code runtime deferral.
