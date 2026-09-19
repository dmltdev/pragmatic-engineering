# Deployment Feature-Readiness Baseline v1

## Always-considered outcomes

| ID | Concern | Observable outcome |
| --- | --- | --- |
| DEP-BASE-01 | Release identity and target integrity | Evidence identifies the exact revision, delivery attempt, intended change scope, and target scope. |
| DEP-BASE-02 | Transition integrity | Delivery preserves required behavior, contracts, and data integrity through supported, repeated, interrupted, or partial states. |
| DEP-BASE-03 | Running-state confirmation | Evidence distinguishes a completed delivery workflow from the intended change actually running in the applicable scope. |
| DEP-BASE-04 | Failure containment and disposition | Failed or partial delivery remains visible and bounded with a known state for the local response. |

## Conditional concerns

| Activate when | Concern | Observable outcome |
| --- | --- | --- |
| Runtime values, flags, credentials, or environment settings change. | Configuration delivery | Required configuration reaches the intended scope with compatible values. Missing, invalid, or partial configuration fails visibly. |
| Delivery selects or promotes an artifact. | Artifact identity and integrity | The target receives the identified candidate without mutation during delivery. |
| Data, schemas, contracts, infrastructure state, or components have order-dependent compatibility. | Migration and sequencing | Coexistence, interruption, restart, and retry states have a defined safe sequence. |
| Exposure is staged, target-specific, mixed-version, or traffic-controlled. | Rollout control | Promotion, hold, stop, and mixed-state behavior are defined. Partial rollout does not become unknown state. |
| Material behavior or failure is observable only after delivery. | Post-delivery observation | Evidence covers required outcomes and failure modes for the local observation scope. Delivery success alone is insufficient. |
| Returning to a previous executable or configuration state is applicable. | Rollback safety | Restoration preserves compatible data, configuration, contracts, and dependencies. Residual effects remain visible. |
| Effects are durable, external, or irreversible. | Recovery readiness | The local repair, reconciliation, replay, restoration, or containment path can reach an acceptable state. |

## Boundary and evidence

Pipelines, registries, manifests, CI, dashboards, and deployment commands are evidence context. They do not prove deployment readiness. Rollback restores release state; recovery repairs effects that restoration cannot undo. Every derived project item names an observable delivery condition and local evidence. Missing, stale, ambiguous, inaccessible, or unrelated evidence remains `unverified` or `blocked`.
