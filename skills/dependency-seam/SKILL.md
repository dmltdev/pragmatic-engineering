---
name: dependency-seam
description: Use when implementing critical or core logic that would consume app-specific configuration, provider APIs, SDK types, external services, or third-party semantics, or when asked for adapters, dependency injection, replaceability, or deletability.
---

# dependency-seam

Make the smallest correct seam decision when protected or core logic meets app-owned variability or foreign dependency semantics.

> The protected module owns its vocabulary. A dependency gets a seam only when isolating it reduces real future edit surface more than the seam increases current cognitive surface.

This is a decision-and-implementation discipline, not an adapter-pattern mandate.

## When to use

Use this skill when:

- critical, shared, or core behavior would consume app-specific configuration;
- provider APIs, SDK types, external services, or third-party semantics could enter protected logic;
- current apps, providers, or data shapes satisfy the same protected need differently;
- replacement, deletion, migration, adapters, dependency injection, or replaceability are explicit concerns; or
- a repository already owns a seam for the dependency relationship.

A third-party import alone is insufficient. A leaf-local utility whose semantics, usage, and deletion remain local should normally stay direct.

## Required inputs

Before deciding, establish:

- the protected behavior that must remain stable;
- the dependency or external capability under evaluation;
- current consumers and their representations;
- foreign types, identifiers, keys, errors, validation, lifecycle, and sync or async semantics;
- explicit replacement, deletion, and migration constraints;
- the repository's existing composition and injection convention; and
- the observable behavior or verification surface that can prove the result.

Derive these facts from repository code, tests, configuration, and current documentation before asking. Stop only when an unavailable fact materially changes the boundary decision.

## Decision procedure

Make these decisions in order. Do not collapse module depth, adapter placement, and injection into one architecture label.

### 1. Direct coupling or earned seam

Keep the dependency direct when all relevant facts support locality:

- usage is narrow and local;
- no semantic translation is required;
- foreign types, keys, errors, or lifecycle neither escape through the protected interface nor spread through protected logic;
- removing the dependency changes only the local implementation; and
- no explicit replacement, deletion, active migration, or current multi-provider requirement exists.

Otherwise, earn a seam only from strong current semantic evidence. Record the evidence before defining an interface.

### 2. Protected-module-owned interface

When a seam is earned, define the smallest interface in the protected module's vocabulary. Expose only operations and semantics its callers need. Do not mirror an SDK, expose provider keys or types, or add operations for hypothetical providers.

If direct coupling was selected, the protected interface is `none`; keep the foreign dependency confined to its local implementation point.

### 3. Module depth

Assess depth independently from adapter placement. A module is deep when a small owned interface hides meaningful policy, behavior, or integration complexity. Translation alone does not make a module deep.

A deep owned module may have its interface implemented by one or more app or provider adapters. “Deep module” and “adapter” are not competing outcomes.

### 4. Adapter placement

Add an adapter when app or provider representations differ from the protected interface. Keep foreign types, identifiers, keys, errors, validation, lifecycle, and data-shape translation on the volatile or consumer-owned side. Place each adapter beside that side, not inside the protected core.

One adapter is not proof of useful variability. A seam with one adapter still needs semantic mismatch, an explicit replacement or deletion requirement, an active migration, or another strong current fact.

### 5. Injection and composition

Wire the selected implementation at the repository's existing composition root. Prefer the lightest repository-native mechanism: a function parameter, constructor, or existing factory.

Do not introduce a DI container, service locator, registry, plugin system, event bus, or adapter stack solely for this seam. Injection is a wiring choice made after the boundary, interface, depth, and adapter decisions; it does not itself justify a seam.

## Evidence hierarchy

Strong current evidence for an earned seam includes:

- two or more current apps, providers, or data shapes satisfying the same protected need;
- app-owned configuration or provider types otherwise entering a shared or protected interface;
- provider vocabulary, identifiers, errors, lifecycle, or async model otherwise spreading through protected callsites;
- an explicit replacement, deletion, or active migration requirement; or
- existing repository architecture that already owns the dependency relationship.

The following are insufficient alone because they describe ownership or hypothetical convenience, not current semantic pressure:

- “It is third-party.”
- “We might replace it someday.”
- “Interfaces are easier to mock.”
- “Dependency injection is best practice.”
- “Another provider could exist.”
- “A wrapper only costs one file.”

Do not compute an abstraction score. Numeric scoring creates fake precision; use recorded semantic facts.

## Boundary semantics

At an earned boundary:

- validate untrusted external or provider data at the adapter edge;
- translate only errors the protected module can act on, with actionable protected-language meaning;
- preserve original causes and useful diagnostics when translating errors;
- never hide dependency failure behind an unapproved silent fallback;
- make sync and async behavior explicit;
- state cache, prefetch, freshness, invalidation, and failure behavior whenever they change what callers observe; and
- avoid unnecessary allocation, copying, and representation conversion in hot paths.

These semantics belong in the protected contract when callers depend on them; provider mechanics stay behind the adapter.

## Feature-flag example

A shared package owns protected concepts, not every consumer app's flag registry. It must not accept raw LaunchDarkly, Unleash, or custom-provider keys or types.

For several related protected decisions, a closed union can be appropriate:

```ts
export type CheckoutDecision = "new-checkout" | "compact-summary";

export interface CheckoutDecisions {
  isEnabled(
    decision: CheckoutDecision,
    context: CheckoutContext,
  ): Promise<boolean>;
}
```

Each app-side adapter maps those concepts to its own provider keys, evaluation context, fallback policy, and data shape:

```text
Checkout feature logic
        ↓ depends on
CheckoutDecisions
        ↑ implemented by
Web LaunchDarkly adapter
POS configuration adapter
```

The app composition root supplies its adapter. Raw keys and provider types remain app-side. Adding or replacing a provider changes that app's adapter and composition, not protected checkout logic.

If the protected caller needs only one decision, prefer a capability-shaped interface such as `isCompactSummaryEnabled(context)` when that is smaller and harder to misuse. Choose union versus capability shape from the actual protected caller surface, never from the provider's registry shape.

## Existing leakage and clean cutover

When existing code already leaks provider semantics, first decide the destination interface, module depth, adapter placement, and composition point. Then require a soft handoff to a behavior-preserving clean-cutover capability; do not duplicate a migration workflow inside this skill.

`refactor-transaction` is an optional current provider of that capability. It is not a dependency of this repository. Never issue an unqualified invocation: the host may not provide it, and a future execution profile may bind another provider.

## Cleanup and proof

Prove the selected behavior through the real verification surface before cleanup. Then, within the changed scope, delete:

- duplicate translation;
- obsolete direct provider or configuration access;
- stale provider-shaped types, keys, registrations, and imports; and
- comments that describe the removed path or misstate ownership.

Do not retain accidental direct and adapted paths in parallel. A compatibility path remains only when it is an explicit, owned requirement with a removal condition.

## Overengineering vetoes

Reject:

- SDK- or provider-mirroring interfaces;
- generic `Provider<T>` or `AdapterFactory` abstractions without current need;
- interfaces created only to make mocking easier;
- methods for hypothetical providers;
- new DI containers, service locators, registries, plugin systems, event buses, or adapter stacks for one local dependency; and
- transparent local wrappers whose usage, semantics, and deletion remain local.

## Output contract

On completion, report exactly:

```markdown
**Dependency seam decision complete.**
- Protected behavior: <what must remain stable>
- Dependency: <app/provider/library/external capability>
- Decision: direct coupling | earned seam
- Evidence: <current facts supporting the decision>
- Protected interface: none | <caller-owned operations and semantics>
- Module depth: direct/local | <behavior or complexity hidden>
- Adapter placement: none | <translation location and provider/app ownership>
- Injection and composition: <wiring mechanism and location>
- Boundary semantics: <validation, errors, async, caching/freshness>
- Cleanup: <duplicate/direct paths removed or none>
- Refactor handoff: none | clean-cutover capability required
- Verification: <observed scenario and result>
```

If the skill cannot decide safely, report exactly:

```markdown
**Dependency seam decision blocked.**
- Missing decision or evidence: <one concrete item>
- Why it is material: <product, architecture, risk, or reversibility impact>
- Safe work completed: <bounded progress that does not pre-decide the seam>
- Required owner: <product | architecture | engineering | security>
```

Neither report commits lifecycle state, records acceptance, or performs a state transition.

## Stop conditions

Stop and use the blocked report when unavailable evidence makes any of these material:

- product meaning or required behavior;
- acceptance criteria or the observable proof surface;
- trust or risk posture for external data, errors, or failure;
- a hard-to-reverse public, persistence, security, or cross-application boundary;
- sync or async, cache, freshness, invalidation, or failure behavior that callers observe; or
- an existing leakage migration whose blast radius cannot be bounded from repository evidence.

Complete any safe inspection or bounded work that does not pre-decide the seam, and name the owner required to resolve the missing fact.

## Verification gate

Do not call the decision complete without an observed behavior scenario through the real selected surface. Prove that protected behavior remains correct and that the dependency is either confined locally or translated and wired at the earned boundary.

Interface aesthetics, compilation alone, and a mock that merely echoes configured values are not proof.
