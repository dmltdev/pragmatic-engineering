# Conditional Feature-Readiness Profiles v1

These profiles are conditional additions, not mandatory standalone checklists. They add only activated concerns, do not duplicate accepted baseline items, and never create a second acceptance authority. Project evidence owns local proof methods, tools, thresholds, environments, owners, and approval.

| Profile | Activate when | Focus |
| --- | --- | --- |
| Integration | A feature crosses frontend, backend, service, partner, or consumer boundaries. | End-to-end contracts, ownership, compatibility, and observable cross-boundary recovery. |
| Data and storage | A feature changes persistent representation, retention, access, migration, backfill, or recovery behavior. | Integrity, transition ordering, privacy lifecycle, reconciliation, and restoration. |
| Asynchronous processing | A feature uses queues, streams, schedules, workers, deferred effects, or replay. | Delivery, ordering, duplication, cancellation, dead work, and operator recovery. |
| Platform-specific UI | A feature depends on mobile, desktop, offline, device permissions, store policy, or hardware capability. | Platform behavior, capability loss, permissions, device constraints, and required fallback. |
| High-risk operations | A feature changes operational capacity, control planes, regulated behavior, or incident response obligations. | Containment, observability, recovery, and local policy evidence. |
