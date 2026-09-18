---
id: state-modeling/no-impossible-state
category: state-modeling
phases: [design, coding, review]
tags: [state, lifecycle, transition]
triggers: [independent-state-flags, invalid-state-combination, unchecked-transition]
status: active
---

## Failure

A reachable representation or transition permits a state that violates a stated domain or lifecycle invariant.

## Signals

Inspect independent flags, correlated nullable fields, public setters, partial updates, and transitions that bypass one authoritative state change. These signals are not proof.

## Evidence

Before finding, cite the construction or transition path and the incompatible values it can produce. Show the violated invariant and consumer impact. A finding must name `state-modeling/no-impossible-state`, impact, confidence, severity, and a bounded fix direction. Otherwise stay silent.

## Bad

```ts
job.isRunning = true;
job.isComplete = true;
```

## Good

```ts
type JobState = "queued" | "running" | "complete";
job.transitionTo("complete");
```

## Allow

Allow independent values when every combination is valid, and closed representations whose construction and checked transitions preserve the invariant.

## Fix

Use the smallest representation that excludes the invalid combination, and centralize checked transitions when transition order matters.
