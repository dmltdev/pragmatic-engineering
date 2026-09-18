# AGENTS.md

## Skills

- When asked to create, change, or delete any skill in this repository, verify the affected skill through local discovery and pressure evidence before reporting completion.
- Keep `skills/` as the canonical installable source for every supported harness: pi, omp, Claude Code, and Codex.
- Do not add a second install source beside `skills/`; plugin manifests must point at this repository and load that directory.
- Routed checks and workflow skills must preserve the catalog budgets, evidence-only finding contract, shadow/candidate isolation, and documented missing-capability behavior.
- `pragmatic-review-advanced` is the only approved external-tool workflow exception. It must remain manual-only, keep provider tooling private, and leave final findings to `pragmatic-review`.

## Package boundary

- This package owns reusable engineering decision disciplines, prioritization lenses, dependency-seam judgment, and evidence gates.
- It does not own product discovery/acceptance lifecycle state, personas, deployment tooling, session tooling, or host-specific workflow orchestration.
- The advanced-review exception does not authorize generic host orchestration, automatic external disclosure, provider configuration, source modification, or product acceptance.
