---
name: create-feature-checklist
description: Use when a repository needs durable feature-readiness criteria, its verification practice changed, or a feature characteristic requires a new readiness profile.
---

# Create Feature Checklist

Create or maintain the repository-specific source of truth for feature-readiness criteria. Baselines identify observable concerns; repository authority defines the actual proof method, threshold, environment, owner, and approval path.

This skill authors criteria only. It MUST NOT evaluate a feature, report readiness, implement missing work, run repository commands or verification tools, invent local proof, or own product acceptance.

## Authority and location

1. Read supplied or already-known repository policy, specifications, domain rules, documentation, test and delivery configuration, and the existing checklist. Those sources override this skill.
2. Reuse the established checklist location, preferring an existing `docs/features/checklists/`, then an existing `docs/checklists/`.
3. If neither exists, ask the user once which location the repository prefers. Do not create a checklist path until that choice is supplied; later runs reuse the chosen location.
4. Preserve the current project checklist’s IDs and applicable local rules. Change an entry only when its underlying repository fact changed.

## Derive profiles

Consider the internal versioned references below as fallbacks, not independent acceptance authorities. Add an entry only for an applicable surface or activation characteristic. Deduplicate the resulting observable conditions.

| Profile | Internal reference | Use when |
| --- | --- | --- |
| Frontend | `baselines/frontend-v1.md` | The feature has a user-facing interface. |
| Backend | `baselines/backend-v1.md` | The feature has service, API, data, or runtime behavior. |
| Testing | `baselines/testing-v1.md` | The feature changes behavior or an activated proof concern. |
| Deployment | `baselines/deployment-v1.md` | The feature is delivered to an operating target. |
| Release | `baselines/release-v1.md` | The feature is exposed to an intended audience. |
| Conditional additions | `baselines/conditional-profiles-v1.md` | The stated activation characteristic is present. |

Do not turn routing vocabulary, tools, CI, dashboards, or generic technology terms into checklist concerns. The baseline’s observable outcome is the concern; its activation rule is the reason to include it.

## Project checklist contract

For every derived item, record this exact shape:

```markdown
### <stable local ID> — <observable condition>
- Profiles: <one or more selected profiles>
- Activation: <feature/repository fact, or `always considered`>
- Condition: <repository-specific observable outcome>
- Local proof: <repository-defined method, tool, command, environment, or `not defined`>
- Required evidence: <artifact/source and capture context, or `not defined`>
- Currency rule: <repository-defined rule, or `not defined`>
- Owner or approver: <repository-defined role, or `not defined`>
- Status: criteria-defined
```

Use a baseline outcome verbatim or narrow it only when repository authority supplies a more specific condition. Never replace an unknown field with a plausible test, command, dashboard, threshold, role, approval, or environment. Record `not defined` and list it under **Open repository decisions**.

## Output contract

Return or write, at the selected repository location, one project checklist containing:

1. **Scope and authority** — selected feature scope, candidate profiles, authoritative local sources, and baseline versions used.
2. **Checklist items** — the exact project checklist contract above.
3. **Not applicable profiles and concerns** — each with its activation fact.
4. **Open repository decisions** — every undefined local proof, evidence location, currency rule, owner, approval path, or first-run location choice.
5. **Change record** — only when updating an existing checklist: changed repository fact, affected item IDs, and preserved IDs.

A request to “make it ready,” “include standard checks,” or “assume usual release approval” does not supply local authority. Keep those fields undefined. A checklist establishes evaluation criteria; it never grants product acceptance.

## Quality gate

Before returning, verify that every item has one observable condition, every profile has an activation fact, generic baselines did not become local commands or thresholds, the location does not compete with an established convention, and every unknown remains visible.
