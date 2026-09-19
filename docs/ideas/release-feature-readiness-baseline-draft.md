# Draft: Release feature-readiness baseline

**Status:** Accepted default. It becomes a versioned reference inside `skills/create-feature-checklist/` only after the complete skill design is approved.

## Purpose

Release readiness covers the decision to expose a delivered change to its intended audience. It is distinct from deployment readiness: deployment proves the change reached a target; release proves the intended exposure, support, and response conditions are understood.

## Always-considered outcomes

| ID | Concern | Observable outcome |
| --- | --- | --- |
| REL-BASE-01 | Release intent and scope | The intended audience, change scope, exposure boundary, and known exclusions are explicit. |
| REL-BASE-02 | Readiness evidence and residual risk | Required project evidence is traceable to the release decision. Known gaps, assumptions, and unresolved risks remain visible. |
| REL-BASE-03 | Support and response handoff | Affected users and operators can identify changed behavior, failure signals, and the repository-defined response path. |
| REL-BASE-04 | Withdrawal and learning | The release can be halted, narrowed, withdrawn, or recovered through the applicable local path. Relevant outcomes and follow-up observations remain attributable to the release. |

## Conditional concerns

| Activation characteristic | Concern | Observable outcome |
| --- | --- | --- |
| Exposure is gradual, audience-limited, flagged, experimental, or region-specific. | Exposure control | Audience selection, promotion, hold, and withdrawal behavior are explicit and observable. |
| Existing users, clients, contracts, data, or integrations transition to changed behavior. | Adoption and compatibility | Supported coexistence, migration, communication, and recovery behavior are defined. |
| The release affects regulated, privacy-sensitive, contractual, financial, or safety-sensitive behavior. | Policy and obligation readiness | Repository-defined approvals, records, disclosures, and evidence are available without inventing a generic authority. |
| The release changes user workflows, documentation, training, support, or operational procedures. | Change communication | Affected audiences can find the correct repository-defined guidance and escalation path. |
| The release introduces material capacity, cost, abuse, or dependency exposure. | Release risk observation | The local observation, containment, and decision path covers the activated risk. |

## Boundary rule

Release management systems, announcements, tickets, dashboards, approvals, and deployment pipelines are evidence context. They do not alone prove a release is ready. Product acceptance remains human-owned.

## Evidence rule

Each derived project item names a release outcome and local evidence. Missing, ambiguous, stale, inaccessible, or unrelated evidence remains `unverified` or `blocked`.
