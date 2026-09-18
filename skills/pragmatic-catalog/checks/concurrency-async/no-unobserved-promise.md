---
id: concurrency-async/no-unobserved-promise
category: concurrency-async
phases: [coding, review]
tags: [promise, task-lifecycle, async-failure]
triggers: [discarded-promise, fire-and-forget, response-before-task-completion]
status: active
---

## Failure

Started asynchronous work can reject or outlive its lifecycle owner without any caller or task owner observing its completion.

## Signals

Inspect discarded promise-like results, `void`, missing await or return, async callbacks, and responses sent after starting work. Syntax alone is not proof.

## Evidence

Before finding, trace the call to work that can reject or outlive its owner, then show that no await, return, rejection handler, or lifecycle-managed task owner observes it. A finding must name `concurrency-async/no-unobserved-promise`, impact, confidence, severity, and a bounded fix direction. Otherwise stay silent.

## Bad

```ts
saveAudit(event);
reply.sendStatus(204);
```

## Good

```ts
void tasks.spawn("audit", () => saveAudit(event));
// tasks records failure and drains before shutdown
```

## Allow

Allow awaited or returned work and explicit task ownership that records failure and keeps or drains work for the required lifetime.

## Fix

Await or return the work, handle its failure, or register it with the existing lifecycle-managed task owner.
