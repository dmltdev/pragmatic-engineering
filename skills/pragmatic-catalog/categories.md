# Pragmatic Catalog Categories

A category is the one stable failure domain that owns a check's review pass. Tags and variants improve routing but never transfer ownership.

| Category | Failure domain | Review ownership |
|---|---|---|
| `state-modeling` | Invalid state representation or transition | The state-modeling pass owns representation invariants, reachable states, and transition correctness. |
| `concurrency-async` | Ordering, concurrency, task, or promise correctness | The concurrency-and-async pass owns ordering, races, task lifetime, cancellation, and promise observation. |
| `data-access` | Query cardinality, access shape, and data-volume behavior | The data-access pass owns query growth, batching, pagination bounds, and access-volume behavior. |
| `boundary-trust` | Untrusted input, external representation, and boundary translation | The boundary-trust pass owns runtime validation, representation translation, and preservation of boundary semantics. |
| `failure-recovery` | Retry, fallback, idempotency, and failure visibility | The failure-and-recovery pass owns retry safety, fallback policy, idempotency, and observable failure. |

## Ownership rules

- Every check has exactly one category and one owning review pass.
- Mechanisms and ecosystems such as promises, ORMs, queues, React, and databases are tags or variant scopes, not categories.
- A focused review pass loads only applicable checks owned by its category.
- Cross-category symptoms do not duplicate ownership. Route the check by its falsifiable failure condition.
- Adding, renaming, merging, or splitting a category requires an explicit taxonomy decision because it changes routing and review ownership.
