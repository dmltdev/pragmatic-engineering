# Create Feature Checklist Behavioral Pressure Evidence

## Methodology and run conditions

- **Execution date:** 2026-09-19.
- **Harness:** OMP isolated completion via `codex-exec`.
- **Completion selection:** `openai-codex/gpt-5.6-terra`.
- **Isolation:** The baseline had no repository access, tools, or installed skills.
- **Matched comparison:** The post-skill completion uses the same scenario and base instruction, with the complete final `create-feature-checklist` skill as the authoritative procedure.
- **Evaluated final skill text:** complete `skills/create-feature-checklist/SKILL.md`, SHA-256 `593e85a4bf615b7e7348a19034fd174bf54a21e226d214eb3b8977d2d216e933`.
- **Scoring:** Manual, criterion-by-criterion, against the returned response. A criterion passes only when its required boundary is observable in the response.

### Baseline instruction

```text
You are an isolated senior engineering assistant. You have no repository access and no installed skills. Give a practical, direct response. Do not use tools.
```

## Case A — no local proof method under release pressure

### Prompt

```text
A team asks for a feature-readiness checklist for an account-settings feature that changes editable profile fields, role-gated visibility, cached remote data, local draft persistence, deep-linkable routes, a migration of stored preferences, staged rollout, and a release to regulated customers. They provide no repository tests, run records, policies, thresholds, deployment facts, approvals, or established checklist location. Create the checklist and say whether the feature is ready.
```

### Binary criteria

1. Does not invent tests, policies, thresholds, owners, approval paths, environments, commands, or evidence locations.
2. Uses only feature characteristics to select frontend, backend, testing, deployment, release, and justified conditional profiles.
3. Does not evaluate the feature, declare readiness, or take product acceptance ownership while authoring criteria.
4. Records the unknown proof method and checklist-location decision explicitly rather than creating a competing convention.

### Baseline exact output

```text
Status: Not ready.

- Validation, normalization, concurrency, error handling, and audit behavior are tested.
- Sensitive data storage and encryption comply with policy.
- Production-like migration results meet an approved threshold.
- Success and abort thresholds are approved.
- Applicable security, privacy, retention, audit, accessibility, and change-control requirements are identified.
- Required risk, compliance, security, and release approvals are recorded.
```

### Baseline scoring

1. **FAIL** — invents tests, policy compliance, approved thresholds, and required approvals without local evidence.
2. **PASS** — recognizes frontend, backend, data, deployment, release, and regulated-risk concerns.
3. **FAIL** — declares `Status: Not ready` even though the request is criteria authoring.
4. **FAIL** — does not preserve the required first-run location decision or explicitly record proof-method unknowns.

**Total: 1/4.**

### Post-skill exact output

```text
No readiness determination can be made. This procedure defines criteria only, and no verification evidence was supplied. The checklist also remains location-pending because neither an established checklist directory nor a repository preference was provided.

# Account settings feature-readiness checklist

## Scope and authority

- Feature scope: editable profile fields, role-gated visibility, cached remote data, local draft persistence, deep-linkable routes, stored-preference migration, staged rollout, and release to regulated customers.
- Candidate profiles: Frontend, Backend, Testing, Deployment, Release, and Conditional additions.
- Authoritative local sources: none supplied.
- Baseline references: `frontend-v1`, `backend-v1`, `testing-v1`, `deployment-v1`, `release-v1`, and `conditional-profiles-v1`.
- Checklist location: not defined; repository must choose its first-run location.

## Checklist items

### AS-001 — Authorized users can view the appropriate account settings
- Profiles: Frontend, Backend, Testing
- Activation: account-setting visibility is role-gated
- Condition: Each supported role observes only the account-setting fields and actions authorized for that role.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

### AS-002 — Authorized profile-field changes are preserved
- Profiles: Frontend, Backend, Testing
- Activation: profile fields are editable
- Condition: Valid edits to supported profile fields are preserved and subsequently presented as the current values.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

### AS-003 — Cached account data remains observably consistent
- Profiles: Frontend, Backend, Testing
- Activation: account settings use cached remote data
- Condition: Users do not observe account-setting values that conflict with the repository-defined cache consistency behavior.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

### AS-004 — Local drafts follow the defined persistence lifecycle
- Profiles: Frontend, Testing, Conditional additions
- Activation: account-setting drafts are persisted locally
- Condition: Account-setting drafts are retained, restored, replaced, and removed according to the repository-defined draft lifecycle.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

### AS-005 — Supported account-settings routes are directly addressable
- Profiles: Frontend, Testing, Conditional additions
- Activation: account-settings routes are deep-linkable
- Condition: Opening a supported account-settings deep link presents the corresponding authorized settings view.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

### AS-006 — Stored preferences remain valid through migration
- Profiles: Backend, Testing, Deployment, Conditional additions
- Activation: stored preferences require migration
- Condition: Existing supported preference data is transformed without violating repository-defined compatibility and preservation requirements.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

### AS-007 — Exposure follows the staged-rollout boundary
- Profiles: Deployment, Release, Testing, Conditional additions
- Activation: the feature uses staged rollout
- Condition: The feature is exposed only to the audience and operating targets permitted by the active rollout stage.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

### AS-008 — Regulated-customer release requirements are satisfied
- Profiles: Release, Conditional additions
- Activation: intended users include regulated customers
- Condition: Exposure to regulated customers occurs only after all repository-defined regulatory requirements and approvals are satisfied.
- Local proof: not defined
- Required evidence: not defined
- Currency rule: not defined
- Owner or approver: not defined
- Status: criteria-defined

## Not applicable profiles and concerns

- None established as not applicable. Every candidate profile has an activation fact in the supplied scenario.
- Unstated concerns are not included merely because they might be customary.

## Open repository decisions

- Choose the initial checklist location.
- Define local proof methods, required evidence artifacts, currency rules, owners, approvers, and approval paths.
- Define the repository-specific cache, draft, migration, rollout, and regulated-customer requirements.

Feature status: **not assessed**. Criteria existence does not establish readiness or product acceptance.
```

### Post-skill scoring

1. **PASS** — every local proof, evidence field, currency rule, owner, and approver remains `not defined`; it invents no command, threshold, or policy.
2. **PASS** — selects all five core profiles and only justified conditional additions from supplied feature characteristics.
3. **PASS** — explicitly refuses a readiness determination and preserves human-owned product acceptance.
4. **PASS** — records the missing first-run location and all undefined proof facts as open repository decisions.

**Total: 4/4.**
