# Candidate Derivation Behavioral Pressure Evidence

## Methodology and run conditions

- **Execution date:** 2026-09-19.
- **Harness:** OMP isolated completion via eval.
- **Completion selection:** selector `default`; resolved provider/model `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** The baseline received no tools, repository access, or installed skills.
- **Matched comparison contract:** A valid post-skill run retains the baseline prompt and base system prompt, and adds the complete final `derive-catalog-candidates` skill as authoritative procedure.
- **Post-skill completion:** The final `skills/derive-catalog-candidates/SKILL.md`, SHA-256 `0c6cfdb08f2949e65adb9b6831c07d0633dae74a016fe072eb00c8ee009c4035`, was supplied to three fresh `default` completions under the matched comparison contract.
- **Scoring:** Manual, criterion-by-criterion, against the returned response. A criterion passes only when the response states the required observable disposition.

### Baseline system prompt

```text
You are a senior engineering assistant. Give a practical direct answer. You have no repository access or installed skills.
```

## Case A — repaired failure, accepted feedback, and local policy

### Prompt

```text
You maintain a reusable engineering-rule catalog. Derive catalog rules from this evidence.

A TypeScript request handler used `void supervisor.spawn(() => saveAudit(event))`. A bug report said that an audit write failed after the response, but the correction made the supervisor record rejection and drain it on shutdown. The added regression test proves rejection is observed and the task is drained. A user feedback comment says, "Do not block request completion on audit persistence." The repository already has an active rule whose failure condition is: started work can reject or outlive its lifecycle owner without a caller or task owner observing completion. The supplied catalog comparison reports no matching shadow, candidate, deprecated, or superseded item. Separately, a project requirement says customer-facing error text must use the product's approved terminology.

State what catalog rules to add and what happens next.
```

### Binary criteria

1. Treats the repaired audit failure as evidence for the existing check or a refinement candidate; it creates no duplicate check.
2. Treats accepted feedback as supporting context, not independent technical proof.
3. Classifies the terminology requirement as a project policy or local convention, not a reusable catalog candidate.
4. Emits a candidate packet or an explicit no-candidate/refinement disposition rather than installing or activating a check.
5. Leaves the candidate outside coding and review, and states the admission blockers required before shadow.
6. Preserves the non-blocking request requirement as a legitimate scoped case, not a failure signal.

### Baseline exact output

```text
- **Do not add a new async-lifecycle rule.** The audit failure is already covered by the active rule: started work must not be allowed to reject or outlive its lifecycle owner without completion being observed by a caller or task owner.
  - Record this incident as supporting evidence/example for that rule.
  - The approved implementation pattern is to hand off the audit work to a lifecycle-owning supervisor that records rejection and drains owned tasks during shutdown.
  - Do **not** require awaiting audit persistence in the request handler; the non-blocking response requirement is intentional.

- **Add a distinct customer-error terminology rule** (if not already present):
  **Rule:** Customer-facing error messages must use the product-approved terminology.
  **Failure condition:** Error text exposed to customers uses unapproved, internal, legacy, or inconsistent terms.
  This is separate from task supervision and should not be folded into the async rule.

**Next:** deduplicate the async finding against the existing active rule, attach the regression test and bug fix as evidence, and ensure future handlers use supervised detached work rather than `void`-discarded promises. Add the terminology rule to the catalog and validate customer-visible error strings against the approved terminology source.
```

### Baseline scoring

1. **PASS** — avoids a duplicate async check.
2. **FAIL** — treats feedback as an instruction but does not distinguish its evidentiary role.
3. **FAIL** — promotes a project-specific terminology requirement to a reusable catalog rule.
4. **FAIL** — directs adding a catalog rule rather than emitting an isolated candidate disposition.
5. **FAIL** — omits candidate isolation and the admission gates before shadow.
6. **PASS** — retains non-blocking, lifecycle-owned work as valid.

**Total: 2/6.**

### Post-skill scoring

1. **PASS** — returns `Status: blocked`, identifies the active lifecycle rule as the likely refinement or duplicate target, and creates no duplicate check because exact provenance and the stable existing ID are absent.
2. **PASS** — identifies the accepted feedback as intent supporting non-blocking audit persistence, not technical proof of the lifecycle failure.
3. **PASS** — puts the customer-terminology requirement under the project-specific `policy` scope decision.
4. **PASS** — returns the complete candidate-derivation packet with `Catalog mutation: none`.
5. **PASS** — keeps the result outside coding and review and names exact provenance, catalog comparison, and fixture-related admission blockers.
6. **PASS** — preserves supervisor-owned, non-blocking audit work as a legitimate non-failing boundary.

**Total: 6/6.** The intentionally incomplete source bundle produced `blocked`, not an invented candidate.

## Case B — unresolved feedback

### Prompt

```text
A support message says, "The export sometimes loses fields." No reproduced path, corrective change, test, contract, or accepted diagnosis is available. The supplied catalog comparison reports no matching active, shadow, candidate, deprecated, or superseded item. Derive a reusable catalog candidate.
```

### Binary criteria

1. Records the support message as a discovery lead with unresolved facts.
2. Emits `no-candidate`; it does not invent a failure condition, category, check ID, or rule text.
3. Requests the smallest missing evidence bundle and stops the bounded run.

### Post-skill scoring

1. **PASS** — records the report as an unresolved discovery lead with the missing behavior facts.
2. **PASS** — returns `Status: no-candidate`, uses `None.` for every check-specific field, and invents no category, ID, or rule text.
3. **PASS** — requests the smallest behavior, contract, corrective, and corroborating-evidence bundle, then stops after the one supplied source.

**Total: 3/3.**

## Case C — framework-only observation

### Prompt

```text
A fixed defect exists only because an undocumented adapter method in Framework X mutates a request after a callback returns. The evidence supports no cross-framework semantic invariant, and the supplied catalog comparison reports no parent core or matching active, shadow, candidate, deprecated, or superseded item. Derive a reusable catalog candidate.
```

### Binary criteria

1. Does not invent a general core check merely to create a framework variant.
2. Classifies the result as a scoped candidate blocked from admission, a local convention/policy, or `no-candidate`, with the reason stated.
3. Does not add a category; `taxonomy-gap` remains an explicit disposition when the failure domain cannot be assigned.

### Post-skill scoring

1. **PASS** — returns `Status: no-candidate`; it does not manufacture a universal parent or framework variant.
2. **PASS** — classifies the undocumented adapter behavior as a discovery lead lacking a reusable semantic invariant and identifies the evidence needed to change that result.
3. **PASS** — assigns no category or check; taxonomy remains unmodified.

**Total: 3/3.**
