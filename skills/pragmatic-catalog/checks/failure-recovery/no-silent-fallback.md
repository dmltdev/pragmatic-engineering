---
id: failure-recovery/no-silent-fallback
category: failure-recovery
phases: [design, coding, review]
tags: [fallback, dependency-failure, stale-cache]
triggers: [catch-default, empty-on-error, stale-on-error]
status: active
---

## Failure

A dependency failure is hidden behind an unapproved fallback that callers observe as a normal result.

## Signals

Inspect broad catches that return empty collections, defaults, cached data, or success states. Also inspect fallback paths that omit freshness markers or original-cause diagnostics. Signals are not proof.

## Evidence

Before reporting, prove all four facts: the dependency can fail on this path; code converts that failure into an outwardly valid result; no applicable contract or approved policy authorizes the substitution; and the caller cannot distinguish it from a normal result. Cite the path, failure, impact, confidence, severity, and bounded fix direction under this stable check ID.

## Bad

```ts
try {
  return await pricing.fetch();
} catch {
  return [];
}
```

## Good

```ts
catch (cause) {
  diagnostics.dependencyFailure(cause);
  if (!cache.hasPrices()) throw cause;
  return { prices: cache.prices(), stale: true };
}
```

## Allow

An approved degraded or stale-data policy is safe when the result is distinguishable, original-cause diagnostics remain available, and an unusable fallback fails explicitly.

## Fix

Propagate an explicit failure, return a distinguishable failure result, or apply an approved marked fallback while preserving diagnostics.
