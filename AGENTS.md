# AGENTS.md

## Skills

- When asked to create, change, or delete any skill in this repository, verify the affected skill through local discovery and pressure evidence before reporting completion.
- Keep `skills/` as the canonical installable source for every supported harness: pi, omp, Claude Code, and Codex.
- Do not add a second install source beside `skills/`; plugin manifests must point at this repository and load that directory.

## Package boundary

- This package owns reusable engineering decision disciplines, prioritization lenses, dependency-seam judgment, and evidence gates.
- It does not own product discovery/acceptance lifecycle state, personas, deployment tooling, session tooling, or host-specific workflow orchestration.
