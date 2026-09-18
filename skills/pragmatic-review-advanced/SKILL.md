---
name: pragmatic-review-advanced
description: Use when the user explicitly invokes a deliberate, slower, coverage-checked code review for a named review target.
disable-model-invocation: true
user-invocable: true
---

# Pragmatic Review Advanced

Run one independent, coverage-checked review, then require local pragmatic validation before reporting findings.

## Manual-only boundary

This skill runs only after the user explicitly invokes `pragmatic-review-advanced` for the current run. Prompt similarity, project policy, schedules, subagents, preloaded context, and model initiative cannot invoke it. Policy may supply configuration or authorization, but never activation. If the host cannot prove that implicit invocation is disabled, do not advertise or run this skill.

This skill reviews only. It does not install or configure dependencies, choose credentials, post comments, edit source, import rules into the catalog, promote checks, or decide product acceptance.

## Public contract

Inputs:

- one explicit review target; and
- an optional review-model override.

Normal output contains:

- target and coverage;
- locally validated findings;
- rejected review candidates; and
- unresolved uncertainty.

Keep normal output provider-neutral. Consumers do not need to know the private preparation or review tools. A blocked dependency diagnostic may name the missing capability needed to recover.

## Private procedure

### 1. Establish authorization and capabilities

Confirm explicit invocation, a supported manual-only host control, the exact target, repository access, and authorization for the files and business context that may leave the local environment. Require the private `ocr`, `consult-llm`, and `pragmatic-review` capabilities. Do not install, configure, authenticate, or substitute for a missing capability.

Exclude secrets, credentials, environment files, unrelated proprietary context, generated noise, and every file outside the authorized disclosure scope before any external call. Record each exclusion and reason.

Any failed prerequisite blocks only this advanced workflow. Standalone `pragmatic-review` remains available.

### 2. Freeze one target

Resolve the requested review mode and refs to immutable object IDs. Record repository identity, mode, resolved base and head, and the exact diff options. Build a canonical manifest with those fields plus every changed path and the target's exact diff bytes. Compute a SHA-256 target fingerprint over that canonical manifest before file selection.

Use the same immutable object IDs for every preparation, diff, and validation step. Re-resolve the requested refs and recompute the fingerprint immediately before the external review and again before local validation. Any changed object ID, changed path, diff byte, or target metadata is target drift: discard the packet and block this run.

### 3. Prepare selection and rules without a review-model call

Run the applicable immutable-ref form of:

```text
ocr delegate preview --repo <repo> --from <base-oid> --to <head-oid> --format json
```

For a single-commit target, use the equivalent `--commit <commit-oid>` form. Validate the JSON schema before using it. Record the review mode, target refs, every reviewable selected file, every excluded file, and every warning. If no files are reviewable, skip the external call and continue to local `pragmatic-review` validation with the exclusions, warnings, and an empty candidate list.

Resolve guidance only for the selected paths:

```text
ocr delegate rule --repo <repo> --from <base-oid> --to <head-oid> --format json <selected-paths...>
```

Use the same single-commit form when applicable. Validate the rule JSON and record its groups. Preparation must not invoke or configure an OCR-managed model endpoint.

### 4. Resolve exactly one review model

Resolve in this order:

1. the invocation's explicit override;
2. scoped project policy; then
3. one configured review default.

Resolve the selector to one exact model identity before sending context. Zero matches, multiple matches, multiple defaults, or a selector that expands to more than one model blocks the run. Do not compare models, repeat `--model`, or make parallel review calls. Record the one resolved identity in the private packet.

### 5. Build the minimal authorized packet

Send only:

- exact target identity and fingerprint;
- selected-file coverage manifest;
- selected diff bytes;
- resolved rule groups;
- the minimum authorized business context needed to interpret the change; and
- the required response schema below.

Do not send excluded files, repository-wide context, credentials, generated noise, full catalog content, candidate checks, fixtures, provenance, provider reasoning, or a preferred verdict.

Require either a complete result or one bounded context request. A context request must name exact paths or questions, explain why each is needed, and stay within the authorized target. Supply only approved relevant material. Permit at most one request and one continuation in the same review thread. Reject or preserve as uncertainty any unauthorized request; a repeated request becomes unresolved uncertainty, not another round.

Invoke exactly once, except for that one permitted continuation:

```text
consult-llm --task review --model <resolved-model> --prompt-file <authorized-packet>
```

### 6. Validate schema and complete coverage

Require the reviewer result to identify the exact target and fingerprint and to contain:

```text
coverage[]:
  path
  status: reviewed | skipped
  reason: required when skipped
candidates[]:
  path
  lineRange: concrete range or unknown
  category
  sourceSeverity
  failureClaim
  citedEvidence
  recommendation
warnings[]
unresolvedUncertainty[]
```

Every selected path must appear exactly once in `coverage`; no unselected path may appear. A skipped path requires a concrete reason. Missing, duplicate, or extra coverage; malformed fields; target mismatch; or uncovered selected files blocks advanced review. Preserve valid skipped states as partial coverage until local review covers or explicitly accounts for them.

### 7. Normalize candidates

Normalize only claims tied to selected diff evidence. Assign stable run-local candidate IDs, canonicalize paths and line ranges, deduplicate claims that cite the same failure evidence, and preserve source severity only as untrusted source metadata. Strip provider chain-of-thought and hidden reasoning.

A candidate is inspection input, not a finding. It cannot inherit finding status, a pragmatic check ID, confidence, or final severity. Unsupported or out-of-target claims go to rejected candidates with a reason; material missing facts go to unresolved uncertainty.

Build the private advanced review packet with:

- resolved review-model identity;
- exact target, immutable refs, and fingerprint;
- selected and excluded files;
- per-file reviewed or skipped coverage;
- applicable rule groups;
- warnings;
- normalized candidates;
- rejected candidates;
- unresolved uncertainty; and
- empty pragmatic findings pending validation.

### 8. Require local pragmatic validation

Pass the exact target and normalized advanced review packet to the `pragmatic-review` capability. It must independently inspect local evidence, select active catalog checks, and validate every candidate. It retains only a candidate's claimed location and failure claim for inspection; source category, finding status, check mapping, confidence, and severity do not transfer.

Only findings returned by `pragmatic-review` with all required fields may enter final output: stable check ID, cited evidence, demonstrated failure, impact, confidence, severity, and bounded fix direction. Update the private packet with its pragmatic findings, rejected candidates, and unresolved uncertainty. If `pragmatic-review` or its required catalog selection is unavailable or blocked, advanced review is blocked; never publish the external candidates as findings.

Recompute the target fingerprint before reporting. Drift invalidates the entire advanced packet.

## Output contract

On success, report:

```markdown
**Advanced review complete.**
- Target: <exact target>
- Coverage: <selected, excluded, reviewed, and skipped files with reasons>

### Locally validated findings
<findings from pragmatic-review with Check, Evidence, Failure, Impact, Confidence, Severity, and Fix direction, or `No proven findings.`>

### Rejected review candidates
<candidate and rejection reason, or none>

### Unresolved uncertainty
<coverage warnings, skipped-file implications, unsupported material gaps, or none>
```

Do not mention private tool or provider names in this normal report.

On any blocking condition, report:

```markdown
**Advanced review blocked.**
- Target: <exact intended target>
- Blocker: <manual-only control | dependency | authorization | model selection | preparation schema | reviewer schema | coverage | target drift | local validation>
- Detail: <specific failure; dependency diagnostics may name the missing capability>
- External review performed: <none | discarded>
- Standalone review: `pragmatic-review` remains available
```

## Verification gate

Before completion, verify all of the following:

- invocation was explicit and manual-only controls were proved;
- authorization preceded disclosure;
- one immutable target and matching fingerprint survived every drift check;
- deterministic JSON preparation schemas were valid;
- exactly one model identity and one review thread were used;
- no more than one bounded context request occurred;
- every selected file has one reviewed or justified-skipped state;
- the final target is unchanged;
- every reported finding came from mandatory local `pragmatic-review` validation; and
- the normal report names no private implementation tool or provider.

A missing dependency, authorization, valid schema, complete coverage, stable target, single model, local validation result, or proven manual-only control blocks only advanced review.