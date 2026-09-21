# Retired Void Grimoire review surfaces

## Status

Reviewed as bounded discovery input. No catalog candidate was created.

## Disposition

- `silent-failure-hunter` overlaps the active `failure-recovery/no-silent-fallback` check. Its useful failure condition is already represented: a dependency failure must not be hidden behind an unapproved fallback.
- `type-design-analyzer` overlaps the active `state-modeling/no-impossible-state` check. Its useful failure condition is already represented: invalid states and transitions must not remain reachable.
- `blast-radius-cartographer` describes a planning workflow. It does not provide observed defect evidence for a portable engineering check.
- The archived framework recipes and generic rule files are technical sources. Without a proven behavior failure, counterexample, or repeated review finding, they do not justify catalog candidates.

## Boundary

Keep these dispositions as review evidence only. A future source can enter candidate discovery when it identifies a concrete failure condition and supplies bounded provenance, examples, and counterevidence. Catalog admission remains owned by the existing candidate lifecycle.
