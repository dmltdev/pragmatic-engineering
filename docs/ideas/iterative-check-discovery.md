# Iterative check discovery

## Purpose

Create a future skill that runs bounded discovery loops. Each loop searches for small, relevant engineering checks that can improve `pragmatic-catalog`.

The skill should prefer one clear failure condition over broad advice. It should preserve evidence and source details for later review.

## Target users

Catalog maintainers and agents that expand the check catalog from repository evidence, review findings, or approved technical sources.

## Trigger

Use the skill after an explicit request to discover new check candidates from a defined source set or to continue a bounded discovery run.

Automated scheduling is a separate future decision. This idea does not add scheduled discovery to Routed Engineering Checks v2.

## Boundary

The skill discovers candidates. It does not activate checks or create developer-facing findings.

Each loop should:

1. inspect a bounded source set;
2. compare observed failures with existing checks and candidates;
3. produce a new candidate, refine an existing candidate, or report no relevant candidate; and
4. use the result to choose the next bounded search focus.

Candidate output remains outside the installed catalog. Coding and review cannot use it until it passes normalization, conflict checks, examples, fixtures, shadow evaluation, and maintainer activation.

The skill must not create duplicate checks to show progress. It must not turn general principles, syntax matches, or source popularity into failure evidence.

## Core output

Each discovery run should record:

- the inspected sources and loop count;
- each candidate's single failure condition;
- suggested category, phases, tags, and routing signals;
- the evidence needed to prove the failure;
- concise bad and similar good examples when the source supports them;
- duplicate, conflict, and provenance notes;
- unresolved facts; and
- the reason for continuing or stopping the loop.

A valid run may return no candidate.

## Verification

- Repeated evidence for the same failure refines one candidate instead of creating aliases.
- A source with no concrete failure produces no candidate.
- Every candidate stays unavailable to coding and review before promotion.
- Each candidate describes one falsifiable failure and the evidence needed to prove it.
- A bounded run records why it stopped.

## Open questions

- Which source types may a discovery loop inspect by default?
- What limits a run: loop count, candidate count, elapsed time, or a combination?
- Should a later system support scheduled runs, or should every run remain explicit?

## Related context

- [Routed Engineering Checks v2](../specs/routed-engineering-checks-v2.md)
