# Iterative check discovery

## Status

Implemented as [`derive-catalog-candidates`](../../skills/derive-catalog-candidates/SKILL.md).

The installed skill accepts an explicit bounded local source set and returns an isolated packet. It has no write, catalog-selection, finding, category-creation, or activation authority. A catalog maintainer explicitly imports a packet into `candidates/checks/`; the existing candidate lifecycle then owns normalization, conflict handling, examples, fixtures, shadow evaluation, and activation.

## Retained constraints

- Each run inspects a bounded source set, compares existing catalog material, and returns one disposition: a candidate, refinement, duplicate, policy, taxonomy gap, no candidate, or blocked result.
- Candidate packets preserve the failure condition, provenance, category proposal, tags, routing signals, counterevidence, unresolved facts, admission blockers, and bounded stop reason.
- Product requirements and project-only exceptions remain policies or local conventions. Syntax, source popularity, and model assertions do not establish a failure condition.
- Candidates remain unavailable to coding and review until promotion.

## Deferred

Automated scheduling and project-specific source adapters remain separate future decisions. They must supply bounded authorized evidence to the portable skill rather than duplicating rule derivation or catalog admission.

## Related context

- [Routed Engineering Checks v2](../specs/routed-engineering-checks-v2.md)
