---
id: concurrency-async/no-unobserved-promise/typescript
refines: concurrency-async/no-unobserved-promise
scope:
  kind: language
  value: typescript
tags: [typescript, promise]
---

## Failure

In TypeScript, a promise-producing expression or async callback satisfies the core failure only when its work can reject or outlive its lifecycle owner without observation.

## Signals

Inspect discarded `Promise` expressions, `void promise`, async callbacks passed to APIs that ignore returned promises, and handlers that finish before started work. These forms only trigger inspection.

## Evidence

Keep the complete core threshold: trace work that can reject or outlive its owner and prove no await, return, rejection handler, or lifecycle-managed task owner observes it. Also prove the expression returns a promise or the callback consumer ignores its promise. Name the core ID and this matching variant in any finding. `void` alone cannot prove a finding.

## Bad

```ts
items.forEach(async (item) => {
  await save(item);
});
```

## Good

```ts
void supervisor.spawn(() => saveAudit(event));
// supervisor records failures and drains tasks before shutdown
```

## Allow

Owned `void` work is safe when the called supervisor registers it immediately, observes rejection, and retains, drains, or cancels it for the required lifetime. Async callbacks are safe when their consumer awaits returned promises.

## Fix

Await or return the promise, use a promise-aware collection operation, attach rejection handling, or register work with the existing owned task supervisor.
