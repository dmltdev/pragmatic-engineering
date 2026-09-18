# Dependency Seam Behavioral Pressure Evidence

## Historical pre-enrichment methodology and matched run conditions

- **Execution date:** 2026-09-09.
- **Baseline batch completion recorded at:** 2026-09-09T08:15:20.536Z.
- **Final post-skill batch completion recorded at:** 2026-09-09T08:29:24.162Z.
- **Harness:** `omp/18.1.13`.
- **Completion selection:** `model: "default"` for every call.
- **Resolved provider/model:** `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Execution shape:** Eight independent stateless no-tool completions. The four baseline calls ran concurrently. Every post-skill case was rerun concurrently after the final skill correction; only that final comparable post-skill set appears below.
- **Isolation:** Each completion received no tools, no repository access, and no prior conversation.
- **Pair comparability:** Prompt, resolved model, harness, and settings are identical within each baseline/post-skill pair. The appended complete skill text is the only added input in the post-skill condition.
- **Historical evaluated skill text:** pre-enrichment `skills/dependency-seam/SKILL.md`, SHA-256 `ef8c4274acf746f0378aecd5fde5fe6c73533823758460883feb428e2066ddfb`. The final enriched regression rerun appears below.
- **Scoring:** Manual criterion-by-criterion binary scoring against exact returned text. No model-as-judge substitution and no style or out-of-scope implementation scoring.

### Baseline system prompt

```text
You are a senior implementation agent. Respond to the requested repository change with a concrete dependency-boundary decision, resulting interface or direct-use shape, placement, wiring, cleanup, and behavioral verification. Do not mention evaluation criteria or that you are being evaluated.
```

### Historical post-skill system prompt rule

The post-skill system prompt was the exact baseline system prompt followed by this exact delimiter and then the current complete `SKILL.md` text:

```text

Apply the following skill as the authoritative engineering procedure for this response:

```

Equivalent exact construction: `baseSystem + "\n\nApply the following skill as the authoritative engineering procedure for this response:\n\n" + skillText`.

## Evaluation summary

| Condition | Case A | Case B | Case C | Case D | Total |
|---|---:|---:|---:|---:|---:|
| Baseline | 6/6 | 5/5 | 7/7 | 8/8 | 26/26 |
| Post-skill | 6/6 | 5/5 | 7/7 | 8/8 | 26/26 |

The baseline legitimately passed every binary criterion; prompts and scores were not altered to manufacture a delta. Superseded post-skill runs exposed two general skill-level wording gaps: proof ordering during clean cutover and explicit external-boundary path coverage. The skill was tightened generally, all four post-skill cases were rerun, and only the final comparable outputs are preserved here.

## Case A — app-specific feature flags

### Prompt

```text
A shared `checkout-summary` package is used by two current apps. Web evaluates LaunchDarkly key `checkout-compact-summary-v2` with an `LDContext`; POS reads `config.features.compactSummary` as a boolean. The shared package needs only to decide whether to render the compact summary. A proposed design passes raw flag keys into the shared package through a generic `FeatureFlagProvider<TContext>` and registers implementations in a new DI container. Decide and describe the dependency shape, interface, adapter placement, composition, cleanup, and behavioral verification. Do not assume any future provider beyond these two current apps.
```

### Binary criteria

1. Earns a seam from the two current representations/providers.
2. Defines only protected shared behavior/concepts, never raw provider keys or types.
3. Places LaunchDarkly and config mapping in their respective app-side adapters.
4. Uses one existing app composition point and no new DI container.
5. Rejects a provider-mirroring generic interface.
6. States a consumer-observable verification scenario.

### Baseline exact output

````text
## Dependency-boundary decision

Do not introduce `FeatureFlagProvider<TContext>` or a new DI container.

The shared package does not need feature-flag functionality; it needs one resolved presentation decision. Provider APIs, flag keys, LaunchDarkly contexts, and POS configuration are application infrastructure and should remain outside `checkout-summary`.

The dependency direction should be:

```text
Web LaunchDarkly adapter ──┐
                           ├── boolean ──> checkout-summary
POS config adapter ────────┘
```

## Shared-package interface

Expose a required, semantically named boolean:

```ts
export interface CheckoutSummaryProps {
  compactSummaryEnabled: boolean;
  // existing props...
}

export function CheckoutSummary({
  compactSummaryEnabled,
  ...props
}: CheckoutSummaryProps) {
  return compactSummaryEnabled
    ? <CompactCheckoutSummary {...props} />
    : <StandardCheckoutSummary {...props} />;
}
```

Making the property required ensures each composition root deliberately supplies the decision. The package should not know:

- `checkout-compact-summary-v2`
- `LDContext`
- LaunchDarkly SDK types
- POS configuration structure
- generic feature-flag concepts

If the flag only changes a portion of the component, the boolean can be passed internally as normal component data; it still does not require DI.

## Adapter placement

### Web

Keep the LaunchDarkly key and evaluation logic in the web app’s checkout integration layer, near its other LaunchDarkly adapters or hooks:

```ts
const CHECKOUT_COMPACT_SUMMARY_FLAG = 'checkout-compact-summary-v2';

export function useCheckoutCompactSummary(
  context: LDContext,
): boolean {
  return useBooleanVariation(
    CHECKOUT_COMPACT_SUMMARY_FLAG,
    context,
    false,
  );
}
```

The exact wrapper should follow the LaunchDarkly SDK already used by the web app. Its contract should return a boolean and use `false` as the safe fallback for missing, malformed, or unavailable evaluations.

Composition:

```tsx
function WebCheckoutSummary(props: WebCheckoutSummaryProps) {
  const context = useCheckoutLDContext();
  const compactSummaryEnabled = useCheckoutCompactSummary(context);

  return (
    <CheckoutSummary
      {...props}
      compactSummaryEnabled={compactSummaryEnabled}
    />
  );
}
```

This keeps `LDContext` construction and reevaluation behavior entirely in Web.

### POS

Map the existing configuration value at the POS composition boundary:

```tsx
function PosCheckoutSummary(props: PosCheckoutSummaryProps) {
  return (
    <CheckoutSummary
      {...props}
      compactSummaryEnabled={
        config.features.compactSummary === true
      }
    />
  );
}
```

If POS configuration is already strictly typed as a required boolean, it can be passed directly. Using `=== true` is appropriate if runtime configuration may be missing or malformed and the desired safe default is the standard summary.

No common adapter interface is needed: the two applications obtain the decision differently but produce the same primitive value.

## Wiring and migration

1. Add required `compactSummaryEnabled: boolean` to the shared component’s public props.
2. Change shared rendering to branch only on that value.
3. Add or retain the Web-specific LaunchDarkly evaluation wrapper.
4. Pass the Web evaluation result into `CheckoutSummary`.
5. Pass `config.features.compactSummary` from POS into `CheckoutSummary`.
6. Update any stories, fixtures, tests, and secondary call sites to provide an explicit value.
7. Remove the proposed `FeatureFlagProvider<TContext>`, its implementations, tokens, registrations, and DI container if they were introduced solely for this feature.
8. Remove LaunchDarkly and POS configuration imports or dependencies from the shared package.
9. Search the shared package for the raw flag key and provider-specific types; neither should remain.
10. Keep the raw LaunchDarkly key defined in one Web-owned location rather than duplicating it across call sites.

## Behavioral verification

### Shared package

Test the resolved behavior directly:

```ts
it('renders the compact summary when enabled', () => {
  render(<CheckoutSummary {...props} compactSummaryEnabled />);
  expect(screen.getByTestId('compact-checkout-summary')).toBeVisible();
});

it('renders the standard summary when disabled', () => {
  render(
    <CheckoutSummary
      {...props}
      compactSummaryEnabled={false}
    />,
  );
  expect(screen.getByTestId('standard-checkout-summary')).toBeVisible();
});
```

Also verify that totals, actions, accessibility labels, and checkout behavior remain equivalent unless they are intentionally presentation-specific.

### Web

Test the Web adapter/composition separately:

- It evaluates exactly `checkout-compact-summary-v2`.
- It supplies the expected `LDContext`.
- LaunchDarkly `true` produces the compact summary.
- LaunchDarkly `false` produces the standard summary.
- Missing, invalid, or failed evaluation falls back to `false`.
- If the current integration supports live context or flag updates, rerendering propagates the updated boolean to the shared component.

Mock LaunchDarkly only in Web tests, not in shared-package tests.

### POS

Test the POS composition:

- `config.features.compactSummary === true` renders compact.
- `false` renders standard.
- Missing or malformed runtime configuration renders standard if such input is possible.

This shape gives the shared package the smallest dependency it needs—the already-resolved decision—while each current app retains ownership of how that decision is obtained.
````

### Baseline scoring

1. **PASS** — “The shared package does not need feature-flag functionality; it needs one resolved presentation decision.”
2. **PASS** — “Expose a required, semantically named boolean:”
3. **PASS** — “while each current app retains ownership of how that decision is obtained.”
4. **PASS** — “Do not introduce `FeatureFlagProvider<TContext>` or a new DI container.”
5. **PASS** — “Do not introduce `FeatureFlagProvider<TContext>` or a new DI container.”
6. **PASS** — “LaunchDarkly `true` produces the compact summary.”

**Total: 6/6**

**Evaluator note:** The baseline already chose a minimal resolved-decision boundary; the post-skill output made ownership, provider-mirroring rejection, composition, and cleanup explicit. No criterion distinguishes their scores.

### Post-skill exact output

````text
**Dependency seam decision complete.**
- Protected behavior: `checkout-summary` decides whether to render the compact or full summary without knowing how an app obtains that decision.
- Dependency: Web’s LaunchDarkly evaluation using `checkout-compact-summary-v2` and `LDContext`, and POS’s `config.features.compactSummary` boolean.
- Decision: earned seam
- Evidence: Two current apps satisfy the same protected need through materially different representations. Passing a raw key and generic context would spread LaunchDarkly vocabulary into the shared package, while POS has neither keys nor an evaluation context. The generic `FeatureFlagProvider<TContext>` would mirror provider mechanics and expose variability the protected caller does not need.
- Protected interface: Define in `checkout-summary` a single capability such as `interface CompactSummaryDecision { isCompactSummaryEnabled(): Promise<boolean>; }`. Bind any request-specific information before supplying it, so neither `LDContext` nor POS configuration types enter the package. The summary renderer receives this capability through its existing constructor, factory, or function parameters and uses the returned boolean only to select compact versus full rendering.
- Module depth: The small capability hides flag-key selection, context construction, configuration lookup, provider initialization, fallback policy, and error translation. No generic flag lookup, raw key parameter, generic context type, or hypothetical flag operations are exposed.
- Adapter placement: Place a Web-owned adapter beside the Web application’s LaunchDarkly integration. It holds or receives the relevant `LDContext`, evaluates the fixed `checkout-compact-summary-v2` key, and converts the result to the owned boolean contract. Place a POS-owned adapter beside POS configuration code; it reads and validates `config.features.compactSummary` and ignores concepts such as keys and contexts because POS does not have them.
- Injection and composition: At Web’s existing request/page composition root, create or bind the LaunchDarkly adapter with the current `LDContext` and pass it to `checkout-summary`. At the POS composition root, pass the configuration-backed adapter. Use ordinary parameter or constructor injection already supported by the repository; do not add the proposed DI container, registry, or provider factory.
- Boundary semantics: The capability is explicitly asynchronous because LaunchDarkly evaluation may be asynchronous; POS returns `Promise.resolve` around its boolean result. Web owns LD client lifecycle, the fixed key, context validation, and its explicitly configured default or fallback policy. POS validates that the configured value is actually boolean. Invalid data or evaluation failure must not silently enable or disable compact rendering: adapters expose a package-owned decision-unavailable error while preserving the original cause and diagnostics. `checkout-summary` adds no cache. LaunchDarkly cache/freshness behavior remains Web-owned, while POS reads according to the application’s existing configuration freshness semantics.
- Cleanup: Remove the proposed `FeatureFlagProvider<TContext>`, raw-key arguments, shared imports of `LDContext`, provider-shaped errors, generic registrations, and the new DI container. Remove duplicate direct LaunchDarkly/configuration reads after both app adapters are wired; retain the LaunchDarkly key only in Web and the configuration path only in POS.
- Refactor handoff: clean-cutover capability required
- Verification: Exercise each app through its real composition path. Web evaluations returning `true` and `false` must render compact and full summaries respectively, while asserting that only the Web adapter knows the fixed key and `LDContext`; the configured LaunchDarkly fallback must produce the corresponding rendering result. POS boolean values `true` and `false` must produce the same two rendering outcomes. Invalid POS values must fail with the owned decision-unavailable error, and LaunchDarkly actionable and opaque failures must preserve their original causes without an unapproved fallback. Verify provider cache/stale/post-expiry behavior at the Web adapter integration boundary when applicable, verify POS configuration refresh behavior, and confirm no provider key, SDK type, or app configuration type remains reachable from `checkout-summary`.
````

### Post-skill scoring

1. **PASS** — “Two current apps satisfy the same protected need through materially different representations.”
2. **PASS** — “neither `LDContext` nor POS configuration types enter the package.”
3. **PASS** — “Place a Web-owned adapter beside the Web application’s LaunchDarkly integration.”
4. **PASS** — “At Web’s existing request/page composition root”
5. **PASS** — “The generic `FeatureFlagProvider<TContext>` would mirror provider mechanics and expose variability the protected caller does not need.”
6. **PASS** — “Web evaluations returning `true` and `false` must render compact and full summaries respectively”

**Total: 6/6**

**Evaluator note:** The baseline already chose a minimal resolved-decision boundary; the post-skill output made ownership, provider-mirroring rejection, composition, and cleanup explicit. No criterion distinguishes their scores.

## Case B — leaf-local utility

### Prompt

```text
`src/invoices/renderDate.ts` has one call to `date-fns/format`. The formatted string is returned; no date-fns types, errors, configuration, lifecycle, or async behavior escape the file. There is no active replacement or deletion requirement. A reviewer asks for an `IDateFormatter`, `DateFnsAdapter`, factory, and constructor injection because third-party code should always be replaceable and interfaces are easier to mock. Decide the dependency shape and the smallest implementation change, including behavioral verification.
```

### Binary criteria

1. Keeps the dependency direct.
2. Grounds the decision in local use, no semantic leakage, and local deletion.
3. Adds no interface, adapter, factory, registry, or injection mechanism.
4. Rejects third-party status and mock convenience as sufficient evidence.
5. States a consumer-observable formatting verification scenario.

### Baseline exact output

````text
## Decision

Keep `date-fns/format` as a direct, file-local dependency in `src/invoices/renderDate.ts`.

`renderDate` is already the appropriate dependency boundary: callers provide the domain input and receive a string. No third-party types, configuration, state, errors, or asynchronous behavior cross that boundary. Adding `IDateFormatter`, `DateFnsAdapter`, a factory, and constructor injection would create speculative indirection without changing the observable contract.

## Implementation shape

Retain the narrow import and existing formatting pattern:

```ts
import { format } from 'date-fns';

export function renderDate(date: Date): string {
  return format(date, /* existing format pattern */);
}
```

Do not add:

- `IDateFormatter`
- `DateFnsAdapter`
- a formatter factory
- constructor or parameter injection
- mocks for `date-fns`

No new wiring or lifecycle management is required. `date-fns` remains a normal production dependency, and `renderDate.ts` owns the integration.

If any proposed adapter/interface scaffolding has already been added, remove it along with unused exports, factory wiring, and adapter-specific tests.

## Behavioral verification

Test `renderDate` through its public behavior rather than mocking `date-fns`:

```ts
describe('renderDate', () => {
  it('renders an invoice date using the configured format', () => {
    const date = new Date(2024, 0, 15, 12, 0, 0);

    expect(renderDate(date)).toBe(/* expected string for existing pattern */);
  });
});
```

Use a local-calendar date constructed at midday to avoid accidental UTC-to-local date rollover in environments with different time zones. Add representative boundary cases only where they matter to the existing format, such as a single-digit day, year transition, or leap day.

Run the repository’s existing test and type-check commands. No production-code change is necessary if the direct import is already present; the smallest useful change is behavioral coverage around the returned string.

An abstraction should be introduced later only if a concrete second implementation, runtime selection, shared formatter configuration, or externally visible provider behavior appears.
````

### Baseline scoring

1. **PASS** — “Keep `date-fns/format` as a direct, file-local dependency in `src/invoices/renderDate.ts`.”
2. **PASS** — “No third-party types, configuration, state, errors, or asynchronous behavior cross that boundary.”
3. **PASS** — “Do not add:”
4. **PASS** — “Adding `IDateFormatter`, `DateFnsAdapter`, a factory, and constructor injection would create speculative indirection without changing the observable contract.”
5. **PASS** — “Test `renderDate` through its public behavior rather than mocking `date-fns`:”

**Total: 5/5**

**Evaluator note:** Both conditions correctly vetoed speculative indirection. The post-skill output states local deletion directly and keeps verification on the consumer-visible formatted string.

### Post-skill exact output

````text
**Dependency seam decision complete.**
- Protected behavior: `renderDate` must synchronously return the existing formatted date string.
- Dependency: the file-local `date-fns/format` function.
- Decision: direct coupling
- Evidence: there is one local call; no date-fns types, errors, configuration, lifecycle, or asynchronous semantics escape the file; removing date-fns would affect only this implementation; and there is no active replacement, deletion, migration, or multi-provider requirement. Replaceability and easier mocking alone do not justify a seam.
- Protected interface: none
- Module depth: direct/local
- Adapter placement: none
- Injection and composition: keep the direct `date-fns/format` import and call in `src/invoices/renderDate.ts`; do not add constructor injection, a factory, or a DI registration.
- Boundary semantics: preserve the current synchronous formatting pattern and existing invalid-date/error behavior; no validation translation, caching, freshness, lifecycle, or async behavior is introduced.
- Cleanup: no production change is required; reject or remove any proposed `IDateFormatter`, `DateFnsAdapter`, factory, constructor parameter, and related registrations.
- Refactor handoff: none
- Verification: exercise the exported `renderDate` function with a fixed valid date and assert the current literal formatted result, using the repository’s established timezone controls if applicable; also preserve any existing invalid-date behavior test. The test must call the real implementation with `date-fns` rather than mock a formatter, proving the dependency remains local and observable output is unchanged.
````

### Post-skill scoring

1. **PASS** — “Decision: direct coupling”
2. **PASS** — “there is one local call; no date-fns types, errors, configuration, lifecycle, or asynchronous semantics escape the file; removing date-fns would affect only this implementation”
3. **PASS** — “do not add constructor injection, a factory, or a DI registration.”
4. **PASS** — “Replaceability and easier mocking alone do not justify a seam.”
5. **PASS** — “exercise the exported `renderDate` function with a fixed valid date and assert the current literal formatted result”

**Total: 5/5**

**Evaluator note:** Both conditions correctly vetoed speculative indirection. The post-skill output states local deletion directly and keeps verification on the consumer-visible formatted string.

## Case C — existing provider leakage

### Prompt

```text
A core fraud-decision module currently imports `AcmeRiskClient`, accepts `AcmeRiskRequest`, switches on `AcmeRiskErrorCode`, and exposes `AcmeRiskDecision` to six callers. The approved destination is provider-neutral core vocabulary, and repository search can enumerate every caller. There is no external compatibility requirement. A proposal adds `RiskProvider` but keeps the old exports and direct calls as deprecated wrappers for safety. Decide the destination seam, module depth, adapter placement, composition, cleanup, migration handoff, and proof boundary. `refactor-transaction` may or may not be installed in the execution environment.
```

### Binary criteria

1. Defines the provider-neutral destination before migration.
2. Separates module depth from adapter placement.
3. Places Acme types/errors and translation in the provider-side adapter.
4. Rejects permanent/deprecated wrappers and parallel direct/adapted paths without compatibility ownership.
5. Emits a behavior-preserving clean-cutover capability handoff.
6. Names `refactor-transaction` only as an optional current provider and issues no unqualified invocation.
7. Requires caller enumeration and behavior proof before deleting the old path.

### Baseline exact output

````text
## Decision

Create a provider-neutral port owned by the core fraud-decision module. The six callers continue to call the fraud-decision API; they must not call `RiskProvider` directly.

Do **not** retain deprecated `AcmeRisk*` exports or direct-call wrappers. There is no external compatibility requirement, all callers are enumerable, and wrappers would preserve the dependency leak and create two supported paths.

## Destination seam

Use operation-oriented core vocabulary rather than renamed Acme types:

```ts
// core/fraud/risk-assessor.ts

export interface RiskAssessor {
  assess(input: RiskAssessmentInput): Promise<RiskAssessmentResult>;
}

export interface RiskAssessmentInput {
  transactionId: string;
  accountId: string;
  amount: Money;
  occurredAt: Date;
  signals: Readonly<Record<string, unknown>>;
}

export type RiskAssessmentResult =
  | {
      kind: "assessed";
      level: "low" | "medium" | "high";
      score?: number;
      reasons: readonly RiskReason[];
    }
  | {
      kind: "failed";
      failure: RiskAssessmentFailure;
    };

export type RiskAssessmentFailure =
  | "temporarily-unavailable"
  | "timed-out"
  | "request-rejected"
  | "invalid-response";
```

Only include fields and failure distinctions that the fraud policy actually uses. Do not mechanically reproduce every `AcmeRiskRequest`, `AcmeRiskDecision`, or `AcmeRiskErrorCode` member under a neutral name.

The decision service depends on that port:

```ts
// core/fraud/decide-fraud.ts

export interface FraudDecisionInput {
  // Provider-neutral caller input.
}

export type FraudDecision =
  | { outcome: "approve" }
  | { outcome: "review"; reasons: readonly string[] }
  | { outcome: "decline"; reasons: readonly string[] };

export function createFraudDecisionService(riskAssessor: RiskAssessor) {
  return {
    async decide(input: FraudDecisionInput): Promise<FraudDecision> {
      const assessment = await riskAssessor.assess(
        toRiskAssessmentInput(input),
      );

      // Switch only on core assessment/failure vocabulary.
      return applyFraudPolicy(input, assessment);
    },
  };
}
```

The six callers should see only `FraudDecisionInput` and `FraudDecision`. `RiskAssessmentInput` and `RiskAssessmentResult` are the internal dependency boundary between the decision service and adapters.

## Module depth

Keep this as one narrow capability port, not a hierarchy of provider abstractions:

```text
core/
  fraud/
    decide-fraud.ts
    fraud-decision-types.ts
    risk-assessor.ts
    fraud-policy.ts
```

`RiskAssessor` is preferable to a generic `RiskProvider`: it describes the operation the core needs and avoids becoming a vendor-shaped service locator. If repository naming conventions require `RiskProvider`, give it the same single-operation shape.

Do not add factories, base classes, provider registries, or generic transport errors unless there is already a demonstrated need for multiple runtime-selected providers.

## Adapter placement

All Acme imports belong outside core:

```text
infrastructure/
  risk/
    acme/
      acme-risk-assessor.ts
      acme-mappers.ts
```

```ts
import {
  AcmeRiskClient,
  AcmeRiskErrorCode,
  type AcmeRiskRequest,
} from "@acme/risk";

import type {
  RiskAssessor,
  RiskAssessmentInput,
  RiskAssessmentResult,
} from "../../../core/fraud/risk-assessor";

export class AcmeRiskAssessor implements RiskAssessor {
  constructor(private readonly client: AcmeRiskClient) {}

  async assess(
    input: RiskAssessmentInput,
  ): Promise<RiskAssessmentResult> {
    try {
      const response = await this.client.assess(toAcmeRequest(input));
      return fromAcmeDecision(response);
    } catch (error) {
      return mapAcmeFailure(error);
    }
  }
}
```

This adapter exclusively owns:

- `RiskAssessmentInput` → `AcmeRiskRequest`
- `AcmeRiskDecision` → `RiskAssessmentResult`
- `AcmeRiskErrorCode` → provider-neutral failure taxonomy
- Acme-specific retry, timeout, authentication, and response-validation details

Unknown Acme codes must map conservatively, normally to `invalid-response` or `temporarily-unavailable`, and should be logged with vendor details at the adapter boundary. Vendor details must not be added to the core type merely to preserve them.

## Composition

Instantiate the concrete client and adapter only in the application composition root:

```ts
const acmeClient = new AcmeRiskClient(config.acmeRisk);
const riskAssessor = new AcmeRiskAssessor(acmeClient);
const fraudDecisionService = createFraudDecisionService(riskAssessor);

const callerA = createCallerA({ fraudDecisionService });
// Repeat for all six callers.
```

No core module should read Acme configuration or construct `AcmeRiskClient`. Callers should receive the fraud-decision service, not the adapter.

For tests, inject a small fake `RiskAssessor` directly:

```ts
const assessor: RiskAssessor = {
  assess: async () => ({
    kind: "assessed",
    level: "high",
    reasons: ["velocity"],
  }),
};
```

## Cleanup and migration transaction

Make this a repository-wide migration in one mergeable change:

1. Enumerate and record all imports and uses of:
   - `AcmeRiskClient`
   - `AcmeRiskRequest`
   - `AcmeRiskDecision`
   - `AcmeRiskErrorCode`
   - the current fraud-decision exports
2. Capture representative existing behavior before changing code, especially every error-code branch.
3. Add the core port and provider-neutral types.
4. Add the Acme adapter and explicit mapping tests.
5. Change the fraud-decision implementation to depend only on `RiskAssessor`.
6. Update the composition root.
7. Migrate all six callers to `FraudDecisionInput` and `FraudDecision`.
8. Delete:
   - old Acme-shaped core exports
   - deprecated wrappers
   - direct client construction outside composition
   - obsolete mapping and switch logic
   - unused imports and dependencies
9. Add an import-boundary rule preventing core from importing the Acme SDK or infrastructure adapter.

Temporary compatibility code may be used locally to sequence edits, but it should not be present in the merged result.

If `refactor-transaction` is installed, it can be used to track the call-site inventory and validate that the migration closes atomically. It must not be required for the implementation. If absent, use repository search, normal source-control commits, and a checked migration list. The handoff should include:

- the original six call sites
- their replacement entry points
- the old-to-new decision and error mapping table
- any intentional behavior changes
- commands and results for build, tests, lint, and boundary checks

## Behavioral proof boundary

The change is complete when all of the following are demonstrated:

### Core behavior

Table-driven decision-service tests cover:

- low, medium, and high assessments
- each provider-neutral failure category
- any score or reason thresholds
- fallback behavior for unavailable or malformed assessments

These tests use a fake `RiskAssessor` and contain no Acme imports.

### Adapter behavior

Adapter tests cover every Acme decision and every Acme error code previously handled, including unknown codes and malformed responses. Tests should assert both request translation and result/error translation.

A mapping table makes equivalence explicit:

| Previous Acme condition | Neutral result | Fraud outcome |
|---|---|---|
| Acme low-risk decision | `assessed/low` | existing outcome |
| Acme high-risk decision | `assessed/high` | existing outcome |
| Acme timeout code | `failed/timed-out` | existing fallback |
| Acme service error | `failed/temporarily-unavailable` | existing fallback |
| Unknown/malformed result | `failed/invalid-response` | conservative fallback |

Use the repository’s actual cases and outcomes rather than assuming those examples.

### Wiring and callers

- Composition tests prove that the production service uses `AcmeRiskAssessor`.
- Tests or type-checking cover all six migrated callers.
- The full repository build, test suite, and lint pass.

### Static dependency boundary

Repository search must show:

- no `AcmeRisk*` references under core
- no `AcmeRiskRequest`, `AcmeRiskDecision`, or `AcmeRiskErrorCode` exports
- no deprecated wrapper path
- `AcmeRiskClient` constructed only in the composition/infrastructure layer
- all six callers use the provider-neutral fraud-decision API

A lint/import rule should enforce this afterward, so the proof is not only a one-time search.

A live Acme smoke test can validate credentials and remote compatibility, but it is not required to prove core behavior. The deterministic proof boundary is the core policy tests, exhaustive adapter mapping tests, composition test, caller migration, and static dependency check.
````

### Baseline scoring

1. **PASS** — “Create a provider-neutral port owned by the core fraud-decision module.”
2. **PASS** — “## Module depth”
3. **PASS** — “This adapter exclusively owns:”
4. **PASS** — “Do **not** retain deprecated `AcmeRisk*` exports or direct-call wrappers.”
5. **PASS** — “Make this a repository-wide migration in one mergeable change:”
6. **PASS** — “If `refactor-transaction` is installed, it can be used to track the call-site inventory and validate that the migration closes atomically. It must not be required for the implementation.”
7. **PASS** — “Capture representative existing behavior before changing code, especially every error-code branch.”

**Total: 7/7**

**Evaluator note:** Both conditions select a provider-neutral clean cutover. The final post-skill run explicitly orders enumeration and current-path characterization before migration, verification of every caller before deletion, and treats `refactor-transaction` as conditional only.

### Post-skill exact output

````text
**Dependency seam decision complete.**
- Protected behavior: Fraud evaluation and its caller-observable decisions, actionable failures, and existing sync/async behavior must remain stable for all six callers.
- Dependency: Acme risk-service API, including `AcmeRiskClient`, `AcmeRiskRequest`, `AcmeRiskDecision`, and `AcmeRiskErrorCode`.
- Decision: earned seam
- Evidence: Provider request, decision, and error-code types currently cross the core boundary and affect six callers; provider-specific branching exists inside core; provider-neutral vocabulary is the approved destination; every caller can be enumerated; and no external compatibility path requires preserving the Acme-shaped API.
- Protected interface: A core-owned, capability-shaped risk assessment operation accepting only fraud-domain input and returning only a core fraud decision using the existing observable async model. It exposes only actionable core outcomes or errors needed by fraud logic; it does not expose Acme requests, decisions, identifiers, or error codes.
- Module depth: A small core interface hides fraud-decision policy, provider invocation semantics, response interpretation, and actionable failure handling. Provider transport and data-shape translation remain outside the core.
- Adapter placement: An Acme adapter belongs in the infrastructure/provider-owned area beside the Acme SDK. It builds and validates `AcmeRiskRequest`, invokes `AcmeRiskClient`, translates `AcmeRiskDecision` into core decisions, maps only actionable `AcmeRiskErrorCode` values into core-language errors, and preserves unknown failures as causes. Acme types must not appear in core exports or caller code.
- Injection and composition: Construct the Acme adapter and inject it into the fraud-decision module at the repository’s existing application composition root, using its existing constructor, factory, or function-parameter convention. Do not add a container, registry, or adapter factory. Migrate all six callers to the provider-neutral fraud API in one bounded cutover.
- Boundary semantics: Validate Acme responses at the adapter edge; reject unknown or malformed decisions rather than silently falling back; translate each actionable provider error explicitly; preserve opaque error causes and diagnostics; retain the currently observed sync/async contract; and preserve existing timeout, retry, caching, freshness, and post-expiry behavior unless an approved behavior change says otherwise.
- Cleanup: Remove the deprecated-wrapper proposal, old Acme-shaped exports, direct `AcmeRiskClient` calls outside the adapter, all caller references to Acme request/decision/error types, duplicate error-code switches, obsolete imports, and stale comments. Do not retain parallel direct and adapted paths because there is no compatibility requirement.
- Refactor handoff: clean-cutover capability required. First enumerate and characterize all six callers through the current path, then migrate and verify every caller before deleting the old path. Use `refactor-transaction` only if the execution host explicitly provides it; otherwise use another available clean-cutover mechanism rather than issuing an unqualified invocation.
- Verification: The proof boundary is the real composed fraud-decision surface used by all six callers. Record current outcomes first, then verify equivalent decisions for representative approve, decline, and review cases; valid and malformed Acme responses; every actionable Acme error mapping; unknown codes and opaque failures with preserved causes; and applicable timeout, retry, cache, freshness, invalidation, and post-expiry paths. Repository searches must then show Acme SDK types and calls confined to the adapter, with all six callers using only core vocabulary.
````

### Post-skill scoring

1. **PASS** — “A core-owned, capability-shaped risk assessment operation accepting only fraud-domain input and returning only a core fraud decision”
2. **PASS** — “Module depth: A small core interface hides fraud-decision policy, provider invocation semantics, response interpretation, and actionable failure handling. Provider transport and data-shape translation remain outside the core.”
3. **PASS** — “An Acme adapter belongs in the infrastructure/provider-owned area beside the Acme SDK.”
4. **PASS** — “Do not retain parallel direct and adapted paths because there is no compatibility requirement.”
5. **PASS** — “Refactor handoff: clean-cutover capability required.”
6. **PASS** — “Use `refactor-transaction` only if the execution host explicitly provides it”
7. **PASS** — “First enumerate and characterize all six callers through the current path, then migrate and verify every caller before deleting the old path.”

**Total: 7/7**

**Evaluator note:** Both conditions select a provider-neutral clean cutover. The final post-skill run explicitly orders enumeration and current-path characterization before migration, verification of every caller before deletion, and treats `refactor-transaction` as conditional only.

## Case D — external data, errors, async behavior, and cache

### Prompt

```text
An async risk SDK returns untrusted JSON. Its documented rate-limit error includes `retryAfterMs`; authentication failures are actionable; other failures are opaque but preserve a cause. Product has approved no silent allow/deny fallback. The caller permits a last-known decision for at most 30 seconds, requires invalidation when the account risk inputs change, and otherwise expects failure to surface. A proposed adapter catches every error and returns `false`, exposes the SDK response type to core, and hides the async call behind an indefinite cache. Define the protected interface and boundary behavior, including validation, error translation, causes, async shape, freshness, invalidation, failure semantics, placement, and verification.
```

### Binary criteria

1. Validates untrusted JSON at the adapter edge.
2. Keeps SDK response/error types out of the protected interface.
3. Translates rate-limit and authentication failures only as actionable core errors and preserves original causes.
4. Rejects catch-all `false` and any unapproved fallback.
5. Makes the interface async and states the approved 30-second freshness bound.
6. States risk-input invalidation and post-expiry failure behavior.
7. Places provider translation outside the protected core.
8. States verification for valid, invalid, actionable-error, opaque-error, fresh-cache, invalidation, and expired-cache paths.

### Baseline exact output

````text
## Dependency-boundary decision

Treat the risk SDK as an infrastructure dependency behind a core-owned asynchronous port. Core and application code must not import the SDK client, response types, or error types.

Use a small anti-corruption adapter for:

- Converting core request data to the SDK request.
- Treating all SDK responses and error payloads as `unknown`.
- Runtime validation.
- Translating errors into stable application-facing errors.
- Preserving the original failure as `cause`.

Apply the 30-second last-known-decision policy in a separate caching decorator. This keeps transport translation and freshness policy independently testable.

## Core-owned interface

```ts
// src/core/risk/RiskDecisionProvider.ts

export type RiskOutcome = "allow" | "deny";

export interface RiskInputs {
  readonly country: string;
  readonly accountAgeDays: number;
  readonly recentChargebackCount: number;
  // Only domain fields that affect the decision.
}

export interface RiskDecisionRequest {
  readonly accountId: string;

  /**
   * Changes whenever any risk-relevant account input changes.
   * Prefer a persisted monotonic revision or immutable snapshot ID.
   */
  readonly riskInputRevision: string;

  readonly inputs: RiskInputs;
}

export interface RiskDecision {
  readonly outcome: RiskOutcome;
}

export interface RiskDecisionProvider {
  decide(
    request: RiskDecisionRequest,
    options?: { readonly signal?: AbortSignal },
  ): Promise<RiskDecision>;
}

/**
 * Called by the account-risk-input update path.
 */
export interface RiskDecisionInvalidator {
  invalidateAccount(accountId: string): void;
}
```

The `Promise` remains visible. Do not make an asynchronous network dependency appear synchronous.

Use a named outcome rather than a bare boolean so callers cannot confuse `false` with “denied,” “unavailable,” or “invalid response.”

## Core-facing failures

```ts
// src/core/risk/RiskDecisionErrors.ts

export abstract class RiskDecisionError extends Error {
  abstract readonly kind:
    | "authentication"
    | "rate_limited"
    | "invalid_response"
    | "provider_failure";
}

export class RiskAuthenticationError extends RiskDecisionError {
  readonly kind = "authentication" as const;

  constructor(options: { cause: unknown }) {
    super(
      "Risk provider authentication failed; verify provider credentials and configuration.",
      { cause: options.cause },
    );
    this.name = "RiskAuthenticationError";
  }
}

export class RiskRateLimitedError extends RiskDecisionError {
  readonly kind = "rate_limited" as const;

  constructor(
    readonly retryAfterMs: number,
    options: { cause: unknown },
  ) {
    super(`Risk provider rate limit exceeded; retry after ${retryAfterMs} ms.`, {
      cause: options.cause,
    });
    this.name = "RiskRateLimitedError";
  }
}

export class InvalidRiskResponseError extends RiskDecisionError {
  readonly kind = "invalid_response" as const;

  constructor(options: { cause: unknown }) {
    super("Risk provider returned an invalid response.", {
      cause: options.cause,
    });
    this.name = "InvalidRiskResponseError";
  }
}

export class RiskProviderError extends RiskDecisionError {
  readonly kind = "provider_failure" as const;

  constructor(options: { cause: unknown }) {
    super("Risk provider request failed.", {
      cause: options.cause,
    });
    this.name = "RiskProviderError";
  }
}
```

Authentication and rate limiting are separately actionable. All other provider failures remain intentionally opaque to callers, while the original exception remains available through `error.cause` for diagnostics.

Do not put credentials, raw response bodies, or arbitrary provider messages in public error messages.

## SDK adapter and validation

```ts
// src/infrastructure/risk-sdk/SdkRiskDecisionProvider.ts

import { z } from "zod";
import type {
  RiskDecision,
  RiskDecisionProvider,
  RiskDecisionRequest,
} from "../../core/risk/RiskDecisionProvider";
import {
  InvalidRiskResponseError,
  RiskAuthenticationError,
  RiskProviderError,
  RiskRateLimitedError,
} from "../../core/risk/RiskDecisionErrors";
import type { VendorRiskClient } from "@vendor/risk-sdk";

const successfulResponseSchema = z
  .object({
    decision: z.enum(["allow", "deny"]),
  })
  .strict();

const rateLimitPayloadSchema = z
  .object({
    code: z.literal("RATE_LIMITED"),
    retryAfterMs: z.number().int().nonnegative().finite(),
  })
  .passthrough();

const authenticationPayloadSchema = z
  .object({
    code: z.enum(["UNAUTHENTICATED", "INVALID_API_KEY", "FORBIDDEN"]),
  })
  .passthrough();

export class SdkRiskDecisionProvider implements RiskDecisionProvider {
  constructor(private readonly client: VendorRiskClient) {}

  async decide(
    request: RiskDecisionRequest,
    options?: { readonly signal?: AbortSignal },
  ): Promise<RiskDecision> {
    let raw: unknown;

    try {
      // Ignore the SDK's compile-time response declaration at this boundary.
      raw = (await this.client.evaluateRisk(
        {
          accountId: request.accountId,
          riskInputRevision: request.riskInputRevision,
          ...request.inputs,
        },
        { signal: options?.signal },
      )) as unknown;
    } catch (cause: unknown) {
      throw translateSdkFailure(cause);
    }

    const parsed = successfulResponseSchema.safeParse(raw);

    if (!parsed.success) {
      throw new InvalidRiskResponseError({ cause: parsed.error });
    }

    return { outcome: parsed.data.decision };
  }
}
```

Error classification must inspect unknown values safely rather than relying on an SDK error type assertion:

```ts
function translateSdkFailure(cause: unknown): Error {
  const payload = extractProviderErrorPayload(cause);

  const rateLimit = rateLimitPayloadSchema.safeParse(payload);
  if (rateLimit.success) {
    return new RiskRateLimitedError(rateLimit.data.retryAfterMs, { cause });
  }

  const authentication = authenticationPayloadSchema.safeParse(payload);
  if (authentication.success || hasAuthenticationStatus(cause)) {
    return new RiskAuthenticationError({ cause });
  }

  return new RiskProviderError({ cause });
}

function extractProviderErrorPayload(cause: unknown): unknown {
  if (typeof cause !== "object" || cause === null) {
    return undefined;
  }

  // Property reads may themselves be unsafe on arbitrary objects/proxies.
  try {
    const value = cause as Record<string, unknown>;
    return value.response ?? value.body ?? value.data;
  } catch {
    return undefined;
  }
}

function hasAuthenticationStatus(cause: unknown): boolean {
  if (typeof cause !== "object" || cause === null) {
    return false;
  }

  try {
    const status = (cause as Record<string, unknown>).status;
    return status === 401 || status === 403;
  } catch {
    return false;
  }
}
```

The exact classifiers should use the SDK’s documented status/code locations. A claimed rate-limit error with a missing, negative, non-finite, fractional, or incorrectly typed `retryAfterMs` must not become a `RiskRateLimitedError`; translate it to `RiskProviderError` or a dedicated malformed-error/protocol error. Never invent a retry delay.

Cancellation should retain its established application semantics if the codebase has one. For example, map the SDK cancellation error to the platform’s cancellation error rather than misclassifying it as a provider outage.

## Bounded last-known-decision decorator

```ts
// src/application/risk/CachedRiskDecisionProvider.ts

import type {
  RiskDecision,
  RiskDecisionInvalidator,
  RiskDecisionProvider,
  RiskDecisionRequest,
} from "../../core/risk/RiskDecisionProvider";

export interface MonotonicClock {
  nowMs(): number;
}

interface CacheEntry {
  readonly decision: RiskDecision;
  readonly expiresAtMs: number;
  readonly generation: number;
}

export class CachedRiskDecisionProvider
  implements RiskDecisionProvider, RiskDecisionInvalidator
{
  private readonly entries = new Map<string, CacheEntry>();
  private readonly generations = new Map<string, number>();

  constructor(
    private readonly delegate: RiskDecisionProvider,
    private readonly clock: MonotonicClock,
    private readonly ttlMs: number = 30_000,
  ) {
    if (!Number.isFinite(ttlMs) || ttlMs < 0 || ttlMs > 30_000) {
      throw new RangeError("Risk decision TTL must be between 0 and 30 seconds.");
    }
  }

  async decide(
    request: RiskDecisionRequest,
    options?: { readonly signal?: AbortSignal },
  ): Promise<RiskDecision> {
    const key = cacheKey(request);
    const now = this.clock.nowMs();
    const cached = this.entries.get(key);

    if (cached && now < cached.expiresAtMs) {
      return cached.decision;
    }

    if (cached) {
      this.entries.delete(key);
    }

    const generation = this.generationOf(request.accountId);

    // Deliberately do not catch. Authentication, rate-limit, protocol, and
    // provider failures must surface when no current decision is available.
    const decision = await this.delegate.decide(request, options);

    // An input-change event may have occurred while the request was in flight.
    // Never repopulate the cache with a decision from the old generation.
    if (generation === this.generationOf(request.accountId)) {
      this.entries.set(key, {
        decision,
        expiresAtMs: this.clock.nowMs() + this.ttlMs,
        generation,
      });
    }

    return decision;
  }

  invalidateAccount(accountId: string): void {
    this.generations.set(accountId, this.generationOf(accountId) + 1);

    const prefix = `${accountId}\u0000`;
    for (const key of this.entries.keys()) {
      if (key.startsWith(prefix)) {
        this.entries.delete(key);
      }
    }
  }

  private generationOf(accountId: string): number {
    return this.generations.get(accountId) ?? 0;
  }
}

function cacheKey(request: RiskDecisionRequest): string {
  // Revision must identify the complete set of decision-affecting inputs.
  return `${request.accountId}\u0000${request.riskInputRevision}`;
}
```

If `riskInputRevision` cannot be guaranteed to represent all risk-affecting fields, include a canonical digest of the complete `RiskInputs` snapshot in the key. Do not use ordinary `JSON.stringify` unless ordering and normalization are controlled.

### Freshness semantics

- A successful decision is reusable for no more than 30 seconds from receipt.
- Cache hits do not renew or slide the expiration.
- At exactly the expiry instant, the entry is expired.
- Only successful, validated decisions are cached.
- Errors are never cached.
- An expired decision is never returned because a refresh failed.
- No error is converted to `allow`, `deny`, `true`, `false`, or `undefined`.
- Invalidation removes all decisions for that account, regardless of revision.
- The revision in the key additionally prevents a request using new inputs from receiving an old decision.
- The generation check prevents an old in-flight request from repopulating the cache after invalidation.

Request coalescing may be added for identical keys, but it must preserve cancellation behavior and the invalidation-generation check. It must not turn failures into cached results.

## Wiring

```ts
// src/bootstrap/risk.ts

const sdkClient = new VendorRiskClient({
  apiKey: config.riskApiKey,
});

const sdkProvider = new SdkRiskDecisionProvider(sdkClient);

const riskDecisionProvider = new CachedRiskDecisionProvider(
  sdkProvider,
  monotonicClock,
  30_000,
);

container.bind<RiskDecisionProvider>(
  RISK_DECISION_PROVIDER,
  riskDecisionProvider,
);

container.bind<RiskDecisionInvalidator>(
  RISK_DECISION_INVALIDATOR,
  riskDecisionProvider,
);
```

The account update path must invalidate after the risk-relevant update is committed and before subsequent work is allowed to reuse a decision:

```ts
await accountRepository.updateRiskInputs(accountId, newInputs);
riskDecisionInvalidator.invalidateAccount(accountId);
```

Prefer publishing a transactional `AccountRiskInputsChanged` event containing the new revision and handling invalidation synchronously in-process where possible. The new revision remains the correctness backstop if event delivery is delayed.

## Placement and cleanup

Recommended placement:

```text
src/
  core/risk/
    RiskDecisionProvider.ts
    RiskDecisionErrors.ts
  application/risk/
    CachedRiskDecisionProvider.ts
  infrastructure/risk-sdk/
    SdkRiskDecisionProvider.ts
  bootstrap/
    risk.ts
```

Remove:

- The catch-all `return false`.
- Any imports of SDK response or error types outside `infrastructure/risk-sdk`.
- The indefinite cache.
- Type assertions that treat SDK JSON as trusted domain data.
- Call-site logic that interprets provider failure as a decision.
- Duplicate provider-specific status/code handling outside the adapter.

## Behavioral verification

Unit and boundary tests should cover:

1. Valid `allow` and `deny` responses map to domain outcomes.
2. `null`, arrays, missing fields, unknown decisions, and wrong field types produce `InvalidRiskResponseError`.
3. A documented rate-limit error yields `RiskRateLimitedError` with the exact validated `retryAfterMs`.
4. Invalid `retryAfterMs` values do not produce a retryable rate-limit error.
5. Authentication codes and documented 401/403 failures yield `RiskAuthenticationError`.
6. Unknown rejection values—including strings, plain objects, and `Error` instances—yield `RiskProviderError`.
7. Every translated provider error retains the exact original value in `cause`.
8. Synchronous SDK throws and asynchronous rejections are both translated.
9. A successful decision is returned from cache before 30 seconds without another SDK call.
10. A cache hit does not extend expiry.
11. At 30 seconds, the provider is called again.
12. An expired entry followed by provider failure surfaces the failure rather than returning the old decision.
13. Failures are not cached; a subsequent call retries the provider.
14. A changed revision cannot reuse the previous decision.
15. `invalidateAccount` removes every cached revision for the account.
16. Invalidation during an in-flight request prevents that old result from repopulating the cache.
17. Other accounts are unaffected by invalidation.
18. Abort behavior remains observable and is not translated into allow/deny.
19. Architecture checks ensure core and application packages cannot import the SDK package.
20. End-to-end wiring confirms account risk-input changes invoke invalidation and that callers handle authentication, rate limiting, and opaque provider failures explicitly.
````

### Baseline scoring

1. **PASS** — “Treating all SDK responses and error payloads as `unknown`.”
2. **PASS** — “Core and application code must not import the SDK client, response types, or error types.”
3. **PASS** — “Authentication and rate limiting are separately actionable. All other provider failures remain intentionally opaque to callers, while the original exception remains available through `error.cause` for diagnostics.”
4. **PASS** — “No error is converted to `allow`, `deny`, `true`, `false`, or `undefined`.”
5. **PASS** — “A successful decision is reusable for no more than 30 seconds from receipt.”
6. **PASS** — “An expired decision is never returned because a refresh failed.”
7. **PASS** — “Use a small anti-corruption adapter for:”
8. **PASS** — “Unit and boundary tests should cover:”

**Total: 8/8**

**Evaluator note:** Both conditions cover the complete trust, error, async, cache, invalidation, and expiry boundary. The final post-skill run explicitly enumerates the valid and invalid response paths plus actionable, opaque, fresh-cache, invalidation, and expired-cache proof paths.

### Post-skill exact output

````text
**Dependency seam decision complete.**
- Protected behavior: Produce an account risk decision without exposing provider vocabulary, while enforcing bounded freshness, explicit invalidation, and surfaced failures.
- Dependency: Async risk SDK returning untrusted JSON and provider-specific errors.
- Decision: earned seam
- Evidence: The SDK response requires runtime validation and translation; provider response types and errors would otherwise enter core; rate-limit and authentication failures have protected-language meaning; cache freshness and invalidation are caller-observable; and product explicitly forbids silent allow/deny fallback.
- Protected interface: Core owns the following capability and types:
  ```ts
  export interface AccountRiskInputs {
    accountId: string;
    revision: string; // Changes whenever risk-relevant inputs change.
    // Core-owned risk fields required for evaluation.
  }

  export type RiskDecision = "allow" | "deny";

  export interface RiskDecisionResult {
    decision: RiskDecision;
    source: "live" | "last-known";
    decidedAt: Date;
  }

  export interface AccountRiskEvaluator {
    evaluate(inputs: AccountRiskInputs): Promise<RiskDecisionResult>;
    invalidate(accountId: string): void;
  }

  export class RiskRateLimitedError extends Error {
    constructor(
      readonly retryAfterMs: number,
      options: { cause: unknown },
    ) {
      super("Risk evaluation was rate limited", options);
    }
  }

  export class RiskAuthenticationError extends Error {
    constructor(options: { cause: unknown }) {
      super("Risk service authentication failed", options);
    }
  }

  export class RiskEvaluationError extends Error {
    constructor(
      message: "Risk service failed" | "Risk service returned an invalid response",
      options: { cause: unknown },
    ) {
      super(message, options);
    }
  }
  ```
  `evaluate` is explicitly asynchronous. No SDK response, request, error, status, or identifier type appears in this interface.
- Module depth: The small `evaluate`/`invalidate` surface hides SDK request translation, runtime response validation, error classification, cause preservation, bounded last-known storage, expiry, invalidation, and protection against invalidated in-flight requests repopulating the cache.
- Adapter placement: Put the SDK implementation beside the provider integration in the application/infrastructure layer. It translates `AccountRiskInputs` into the SDK request and validates the returned JSON against a private runtime schema. Only documented decision variants are mapped to `"allow"` or `"deny"`; missing fields, wrong types, unknown variants, and malformed JSON reject with `RiskEvaluationError("Risk service returned an invalid response", { cause })`. A documented rate-limit error is translated only when `retryAfterMs` is a finite, non-negative integer; otherwise it is an opaque `RiskEvaluationError`. Authentication failures become `RiskAuthenticationError`. Every other failure becomes `RiskEvaluationError("Risk service failed", { cause: originalError })`; its provider details remain opaque to core.
- Injection and composition: Construct the provider adapter at the existing application composition root and inject it into the protected risk workflow as an `AccountRiskEvaluator`, using the repository-native constructor or function parameter convention. Keep one appropriately scoped adapter instance wherever its last-known state must be shared; do not add a container, registry, or service locator.
- Boundary semantics: A successful validated decision may be reused only when its age is less than 30,000 ms and both `accountId` and `revision` match. Cached returns are marked `source: "last-known"` and retain the original `decidedAt`; live returns use `source: "live"`. At 30,000 ms the entry is expired and cannot be returned. `invalidate(accountId)` removes all decisions for that account and advances an internal generation so an older in-flight request cannot repopulate the cache after invalidation. A changed `revision` is always a cache miss, providing defense even if explicit invalidation is delayed. Only validated successes populate the cache; invalid responses and SDK failures never populate or extend it. On a cache miss or expiry, the SDK is awaited and any translated failure rejects the promise. There is no synthesized `allow` or `deny`, no indefinite cache, and no stale-on-error use beyond the explicitly valid 30-second last-known window.
- Cleanup: Reject/remove the catch-all `return false` path, SDK response types from core signatures, indefinite cache behavior, duplicate provider error handling, and direct SDK imports from protected code. Do not retain parallel direct and adapted paths.
- Refactor handoff: none
- Verification: Exercise the composed protected workflow through the real adapter surface with a controllable SDK transport and clock: valid provider allow/deny JSON maps correctly; malformed JSON, missing fields, wrong types, and unknown decision values reject as invalid responses; valid rate-limit errors expose `retryAfterMs` and preserve the original cause; malformed rate-limit metadata becomes an opaque evaluation error; authentication errors translate distinctly with their cause; unknown failures remain opaque while preserving cause identity; a successful decision is returned as last-known before 30 seconds but not at or after 30 seconds; changed revisions and explicit invalidation force live evaluation; invalidation during an in-flight request prevents stale repopulation; failures do not populate or refresh cache; and an expired or invalidated entry followed by SDK failure rejects rather than returning `false` or another decision.
````

### Post-skill scoring

1. **PASS** — “validates the returned JSON against a private runtime schema.”
2. **PASS** — “No SDK response, request, error, status, or identifier type appears in this interface.”
3. **PASS** — “valid rate-limit errors expose `retryAfterMs` and preserve the original cause”
4. **PASS** — “product explicitly forbids silent allow/deny fallback.”
5. **PASS** — “A successful validated decision may be reused only when its age is less than 30,000 ms”
6. **PASS** — “an expired or invalidated entry followed by SDK failure rejects rather than returning `false` or another decision.”
7. **PASS** — “Put the SDK implementation beside the provider integration in the application/infrastructure layer.”
8. **PASS** — “valid provider allow/deny JSON maps correctly; malformed JSON, missing fields, wrong types, and unknown decision values reject as invalid responses”

**Total: 8/8**

**Evaluator note:** Both conditions cover the complete trust, error, async, cache, invalidation, and expiry boundary. The final post-skill run explicitly enumerates the valid and invalid response paths plus actionable, opaque, fresh-cache, invalidation, and expired-cache proof paths.

## Routed checks v2 supplemental pressure evidence

### Supplemental methodology and run conditions

- **Execution date:** 2026-09-18.
- **Harness:** isolated stateless `pressure_completion` calls under OMP; the tool reported no harness version.
- **Completion selection:** default model for every call.
- **Resolved provider/model:** active resolved model `openai-codex/gpt-5.6-sol`.
- **Settings:** no sampling overrides.
- **Execution shape:** Six independent no-tool completions. Two baseline calls ran concurrently before the skill edit. Two initial post-skill calls exposed a handoff-naming gap. The skill wording was corrected, both post-skill cases were rerun concurrently, and only the final comparable post-skill outputs appear below.
- **Isolation:** Each completion received no tools, repository access, or prior conversation.
- **Pair comparability:** Prompt, resolved model, and settings are identical within each pair. The complete final skill text is the only post-skill addition.
- **Post-skill system prompt construction:** the baseline system prompt, then `"\n\nApply the following skill as the authoritative engineering procedure for this response:\n\n"`, then the complete final `skills/dependency-seam/SKILL.md`.
- **Evaluated final skill text:** complete `skills/dependency-seam/SKILL.md`, SHA-256 `94ed1d391624eee4029d746203cc4490e327286d6325e1e59e62570ab58770fc`.
- **Scoring:** Manual criterion-by-criterion binary scoring against the exact returned text. A baseline pass remains a pass.

### Supplemental baseline system prompt

```text
You are a senior implementation agent. Respond to the requested repository change with a concrete dependency-boundary decision, resulting interface or direct-use shape, placement, wiring, cleanup, and behavioral verification. Do not mention evaluation criteria or that you are being evaluated.
```

## Case E — standalone without catalog

### Prompt

```text
The `pragmatic-catalog` capability is not installed. `src/invoices/renderDate.ts` makes one local call to `date-fns/format`; no foreign types, keys, errors, configuration, lifecycle, or async behavior escape the file, and there is no replacement, deletion, or migration requirement. A reviewer asks for an `IDateFormatter`, adapter, factory, and injection solely because the library is third-party. Apply the dependency-seam discipline and complete the decision. Catalog absence must not reduce the standalone decision.
```

### Binary criteria

1. Completes the dependency-seam decision despite unavailable `pragmatic-catalog`.
2. Keeps the local dependency direct and adds no interface, adapter, factory, or injection.
3. Introduces no catalog hard dependency or cross-skill filesystem path.
4. Uses the existing complete output contract without an enrichment field.
5. Suggests no stable check ID or optional capability request when the catalog is absent.
6. States a consumer-observable behavioral verification scenario.

### Baseline exact output

````text
## Decision

Keep the direct `date-fns/format` import in `src/invoices/renderDate.ts`.

`date-fns` is a private implementation detail of that module: it is used once, synchronously, and does not expose library-owned types, errors, configuration, lifecycle, or behavior across the module boundary. Third-party ownership alone does not justify a dependency seam.

## Resulting shape

```ts
// src/invoices/renderDate.ts
import { format } from 'date-fns';

export function renderDate(/* existing parameters */) {
  return format(/* existing arguments */);
}
```

Preserve the existing exported signature and format string.

## Placement and wiring

- No `IDateFormatter`
- No adapter class
- No factory
- No dependency-injection token or container registration
- No constructor or function parameter added for formatting
- No new module or public export
- Keep `date-fns` as the normal production dependency already used by this file

If any proposed seam artifacts were added during review, remove them and restore callers to the existing `renderDate` API.

## Behavioral verification

Verify behavior at the `renderDate` boundary rather than mocking `date-fns`:

- Assert representative dates produce the expected rendered strings.
- Cover any existing locale, timezone, or invalid-date behavior that the function promises.
- Run the repository’s type-check, unit tests, and lint checks.
- Do not add tests asserting adapter/factory invocation details.

A seam should be reconsidered only if a concrete boundary need appears—for example multiple formatter implementations, runtime selection, escaping foreign contracts, or an actual replacement/migration requirement. The missing catalog capability does not alter this standalone decision.
````

### Baseline scoring

1. **PASS** — “The missing catalog capability does not alter this standalone decision.”
2. **PASS** — “Keep the direct `date-fns/format` import”
3. **PASS** — No catalog dependency or filesystem path appears.
4. **FAIL** — The response uses free-form headings instead of the required completion report.
5. **PASS** — It suggests no stable check ID or capability request.
6. **PASS** — “Assert representative dates produce the expected rendered strings.”

**Total: 5/6**

### Post-skill exact output

````text
**Dependency seam decision complete.**
- Protected behavior: `renderDate` consistently renders invoice dates using the existing format.
- Dependency: `date-fns/format`
- Decision: direct coupling
- Evidence: The dependency is used by one local call in `src/invoices/renderDate.ts`; no foreign types, keys, errors, configuration, lifecycle, or asynchronous semantics escape the file; removing it would affect only this local implementation; and no replacement, deletion, migration, or multi-provider requirement exists. Third-party status alone does not earn a seam.
- Protected interface: none
- Module depth: direct/local
- Adapter placement: none
- Injection and composition: Keep the direct `date-fns/format` import and call in `src/invoices/renderDate.ts`; no factory, injection mechanism, or composition-root wiring is warranted.
- Boundary semantics: Formatting remains synchronous with existing `date-fns` behavior. No additional validation, error translation, caching, freshness, or fallback policy is required.
- Cleanup: Do not add `IDateFormatter`, an adapter, a factory, or injection plumbing; remove such artifacts if introduced solely for this review.
- Refactor handoff: none
- Verification: The real `renderDate` path makes its existing local `date-fns/format` call and returns the formatted invoice-date result without exposing dependency semantics outside the file; the requested abstraction would not change observable behavior.
````

### Post-skill scoring

1. **PASS** — It completes the decision without requiring the catalog.
2. **PASS** — “Decision: direct coupling” and “Do not add `IDateFormatter`, an adapter, a factory, or injection plumbing”
3. **PASS** — No catalog dependency or filesystem path appears.
4. **PASS** — It uses the unchanged completion report and adds no field.
5. **PASS** — It suggests no stable check ID or capability request.
6. **PASS** — “The real `renderDate` path makes its existing local `date-fns/format` call and returns the formatted invoice-date result”

**Total: 6/6**

**Evaluator note:** The baseline already preserved standalone behavior. The final skill also preserves the exact output shape and stays silent about optional enrichment when the capability is absent.

## Case F — evidence-bounded optional enrichment

### Prompt

```text
The optional `pragmatic-catalog` capability is available. A provider adapter does `JSON.parse(body) as QuoteResponse` and passes that value into protected pricing logic without runtime validation. It also catches provider request failure and returns `[]`; no contract or approved policy authorizes that substitution, no failure signal or original cause is preserved, and callers cannot distinguish it from a normal empty result. The same change uses a closed status union, awaits all asynchronous work, and performs one batch query before an in-memory loop. Apply the dependency-seam discipline, decide the boundary, and state any optional catalog capability handoff with its evidence. Do not suggest irrelevant checks.
```

### Binary criteria

1. Names `pragmatic-catalog` as the only optional enrichment capability.
2. Suggests `boundary-trust/validate-external-data` only with the external origin, path into trusted logic, missing runtime validation, and violated-shape evidence.
3. Suggests `failure-recovery/no-silent-fallback` only with failed dependency operation, valid-value conversion, absent authorization, and caller-indistinguishability evidence.
4. Suggests no other stable ID for the safe state, async, query, or loop facts.
5. Keeps the handoff optional and leaves applicability and status to `pragmatic-catalog`.
6. Uses one existing output-contract shape without a new enrichment field.

### Baseline exact output

````text
## Boundary decision

Treat the provider response as untrusted at the provider adapter. The adapter must:

1. Parse the body into `unknown`.
2. Runtime-validate the provider payload.
3. Map the validated provider DTO into the protected pricing model.
4. Propagate transport and contract failures as failures, preserving their original causes.

The protected pricing logic must not receive `QuoteResponse` directly or depend on the provider’s wire contract.

```ts
export interface QuoteProvider {
  getQuotes(request: QuoteRequest): Promise<readonly Quote[]>;
}
```

`Quote` is the protected/domain type. `QuoteResponse` and its runtime schema remain private to the provider integration.

## Placement and direct-use shape

Place the decoder beside the provider adapter, for example:

```text
infrastructure/providers/<provider>/
  provider-adapter.ts
  quote-response.schema.ts
  provider-errors.ts
```

Use the repository’s existing runtime-validation library if one is already established. Otherwise, add a focused decoder at this seam rather than introducing validation into protected pricing code.

```ts
function decodeQuoteResponse(body: string): QuoteResponse {
  let value: unknown;

  try {
    value = JSON.parse(body);
  } catch (cause) {
    throw new ProviderContractError(
      "Provider returned malformed JSON",
      { cause },
    );
  }

  const result = quoteResponseSchema.safeParse(value);

  if (!result.success) {
    throw new ProviderContractError(
      "Provider response does not satisfy the quote contract",
      { cause: result.error },
    );
  }

  return result.data;
}
```

Scope transport error handling narrowly so schema failures are not mislabeled:

```ts
async getQuotes(request: QuoteRequest): Promise<readonly Quote[]> {
  let body: string;

  try {
    body = await this.client.fetchQuotes(request);
  } catch (cause) {
    throw new ProviderRequestError(
      "Quote provider request failed",
      { cause },
    );
  }

  const response = decodeQuoteResponse(body);
  return response.quotes.map(mapProviderQuote);
}
```

Do not include the raw response body in error messages if it may contain sensitive or high-volume data.

## Wiring and cleanup

- Inject the adapter through the protected-side `QuoteProvider` port.
- Keep provider DTOs, schemas, and provider-specific errors in the infrastructure integration.
- Remove `JSON.parse(body) as QuoteResponse`.
- Remove the catch-and-`return []` substitution.
- A valid provider response containing no quotes may still resolve to `[]`.
- A request failure or invalid payload must reject. If fallback behavior is ever approved, implement it explicitly in an orchestration/policy layer rather than hiding it inside the adapter.

No changes are warranted for the closed status union, awaited asynchronous work, or the single batch query followed by an in-memory loop.

## Behavioral verification

Add focused adapter tests proving:

- Valid non-empty payloads are validated and mapped.
- A valid empty provider result resolves to `[]`.
- Malformed JSON rejects with `ProviderContractError` and preserves the parse error as `cause`.
- Structurally invalid JSON rejects with `ProviderContractError` and preserves validation details as `cause`.
- Provider request rejection produces `ProviderRequestError` and preserves the original request error as `cause`.
- Protected pricing logic is not called with an unvalidated provider payload.
- Request failure is observably different from a successful empty result.

## Optional `pragmatic-catalog` handoff

Use `pragmatic-catalog` only as a non-blocking discovery handoff for:

- an existing runtime schema/decoder for `QuoteResponse`;
- canonical provider request/contract error classes and `cause` conventions;
- an explicitly approved provider fallback policy.

Accept the handoff only with concrete evidence such as an exported symbol, defining file, and production call site. The supplied evidence establishes that the current code has only an unchecked cast and an unauthorized `[]` substitution; it provides no approved fallback contract. Therefore, absent a catalog result identifying such a policy, failure propagation is the correct behavior and the change must not retain the empty-array fallback.
````

### Baseline scoring

1. **PASS** — It names only `pragmatic-catalog` as an enrichment capability.
2. **FAIL** — It does not suggest the stable ID `boundary-trust/validate-external-data`.
3. **FAIL** — It does not suggest the stable ID `failure-recovery/no-silent-fallback`.
4. **PASS** — It suggests no irrelevant stable ID.
5. **FAIL** — It treats the catalog as source-code and policy discovery rather than check applicability and status resolution.
6. **FAIL** — It adds a separate optional-handoff section instead of using the completion report.

**Total: 2/6**

### Post-skill exact output

````text
**Dependency seam decision blocked.**
- Missing decision or evidence: An observed behavioral verification result through the real provider adapter and application composition path.
- Why it is material: The boundary decision is clear, but completion cannot be claimed until malformed provider JSON and request failures are proven not to enter protected pricing as valid quote data or a normal empty result.
- Safe work completed: The boundary is an **earned seam** because provider DTOs, unchecked external data, and provider failure semantics otherwise leak into protected pricing. Pricing should own a minimal asynchronous capability such as `QuoteSource.getQuotes(request): Promise<readonly Quote[]>`, expressed only in protected request and quote types. The provider/app-side adapter should batch-fetch once, parse and runtime-validate the response, map valid provider records and the closed status union into protected values, and await all asynchronous work before resolving. Invalid JSON or schema-invalid data must fail at the adapter edge; actionable failures should become protected-language errors with the original cause preserved, while opaque failures must remain failures. Returning `[]` on request failure must be removed. Wire the adapter through the repository’s existing constructor, function parameter, or factory at the composition root; no container or registry is warranted. Cleanup should remove the unchecked `as QuoteResponse` path, silent fallback, provider-shaped types from pricing, and any duplicate direct access. Optional handoff to `pragmatic-catalog`: suggest `boundary-trust/validate-external-data` because externally sourced `body` reaches trusted pricing through `JSON.parse(body) as QuoteResponse` without runtime validation, allowing malformed data to violate `QuoteResponse` invariants; suggest `failure-recovery/no-silent-fallback` because provider request failure is converted to indistinguishable `[]` success without authorization, a failure signal, or the original cause. The closed status union, fully awaited work, and one batch query before the in-memory loop require no additional catalog checks.
- Required owner: engineering
````

### Post-skill scoring

1. **PASS** — The optional handoff names only `pragmatic-catalog`.
2. **PASS** — It cites externally sourced `body` reaching trusted pricing through an unchecked assertion and malformed data violating `QuoteResponse` invariants.
3. **PASS** — It cites provider request failure converted to indistinguishable `[]` success without authorization, failure signal, or original cause.
4. **PASS** — “The closed status union, fully awaited work, and one batch query before the in-memory loop require no additional catalog checks.”
5. **PASS** — The handoff is explicitly optional and claims neither applicability nor status.
6. **PASS** — It uses the existing blocked-report shape and adds no enrichment field.

**Total: 6/6**

**Evaluator note:** The baseline made a generic discovery request and omitted both stable IDs. The final skill produces only the two evidence-supported suggestions, omits irrelevant checks, and keeps domain reasoning inside an unchanged output-contract shape.


## Final enriched regression rerun for Cases A-D

- **Execution date:** 2026-09-18.
- **Harness/model:** OMP 18.2.5 isolated completions, `openai-codex/gpt-5.6-sol`, default settings, no tools.
- **Pairing:** Each original Case A-D prompt above was rerun with the same baseline system prompt plus the complete final enriched skill.
- **Evaluated final skill SHA-256:** `94ed1d391624eee4029d746203cc4490e327286d6325e1e59e62570ab58770fc`.

### Final Case A exact output

`````text
**Dependency seam decision complete.**
- Protected behavior: The shared `checkout-summary` package must choose compact versus standard rendering without knowing how an app obtains that decision.
- Dependency: Web’s LaunchDarkly evaluation using `checkout-compact-summary-v2` and `LDContext`, and POS’s `config.features.compactSummary` boolean.
- Decision: earned seam
- Evidence: Two current apps satisfy the same protected need through materially different representations. Direct use would leak a LaunchDarkly key, `LDContext`, and asynchronous/provider failure semantics into shared code while also forcing POS through an irrelevant generic provider abstraction. The proposed `FeatureFlagProvider<TContext>` mirrors provider mechanics and exposes capabilities the package does not need.
- Protected interface: A package-owned, capability-shaped interface:
  ```ts
  export interface CheckoutSummaryDecisions {
    isCompactSummaryEnabled(): Promise<boolean>;
  }
  ```
  The shared render entry point receives this dependency through a parameter or constructor. It does not accept raw flag keys, generic provider contexts, `LDContext`, or app configuration. Any checkout-specific data needed to construct an `LDContext` remains app-owned and is captured by the Web implementation.
- Module depth: A narrow seam hiding provider-key mapping, context handling, configuration lookup, runtime validation, and dependency failure semantics. It is intentionally not a generic feature-flag module.
- Adapter placement: Place a Web adapter beside the Web app’s LaunchDarkly integration. It owns the `checkout-compact-summary-v2` key, `LDContext`, SDK calls, result validation, and LaunchDarkly diagnostics. Place a POS adapter beside POS configuration code; it reads and validates `config.features.compactSummary`. Neither adapter belongs in `checkout-summary`.
- Injection and composition: At the Web composition root, construct the Web adapter with the existing LaunchDarkly client and the current request/session `LDContext`, then pass it to the shared checkout-summary entry point. At the POS composition root, construct the POS adapter with the validated POS configuration and pass it through the same entry point. Use the repository’s ordinary function-parameter or constructor wiring; do not add the proposed DI container, registry, or generic provider.
- Boundary semantics: The shared contract is asynchronous because Web evaluation may be asynchronous; POS can return `Promise.resolve(config.features.compactSummary)`. Both adapters must produce an actual boolean. Malformed configuration or provider results reject rather than being coerced. Dependency failures must not silently become `false` unless an existing, explicitly approved application fallback policy requires that behavior; any such policy remains in the owning app adapter. Preserve original causes and provider diagnostics on failures. The shared package adds no cache, prefetch, freshness, or invalidation policy; LaunchDarkly lifecycle and freshness remain Web-owned, while POS follows its existing configuration lifecycle.
- Cleanup: Remove the proposed `FeatureFlagProvider<TContext>`, raw-key parameters, shared references to `checkout-compact-summary-v2`, shared `LDContext` imports, generic registrations, and the new DI-container setup. Keep exactly one dependency path per app, with provider/configuration details confined to its adapter.
- Refactor handoff: none
- Verification: Exercise the real shared render entry point with each composed app implementation. Verify Web provider results `true` and `false` render compact and standard summaries respectively, and verify the exact key and current `LDContext` are used only inside the Web adapter. Verify POS configuration values `true` and `false` produce the same two render outcomes. Verify malformed/non-boolean data is rejected, actionable provider failures follow the app’s explicit policy, opaque failures retain their original cause, and no shared cache causes stale results after the underlying app-owned value changes. Also verify that the shared package graph contains no LaunchDarkly imports, raw flag keys, POS configuration types, or DI-container dependency.
`````

#### Final Case A scoring

1. **PASS** — Earns a seam from the two current representations/providers.
2. **PASS** — Defines only protected shared behavior/concepts, never raw provider keys or types.
3. **PASS** — Places LaunchDarkly and config mapping in their respective app-side adapters.
4. **PASS** — Uses one existing app composition point and no new DI container.
5. **PASS** — Rejects a provider-mirroring generic interface.
6. **PASS** — States a consumer-observable verification scenario.

**Total: 6/6.**

### Final Case B exact output

`````text
**Dependency seam decision complete.**
- Protected behavior: `renderDate` returns the established formatted representation of a supplied date.
- Dependency: The local `date-fns/format` function.
- Decision: direct coupling
- Evidence: Usage is confined to one call in one file; no date-fns types, errors, configuration, lifecycle, or async semantics escape; no semantic translation is needed; deletion would affect only this implementation; and there is no active replacement, migration, or multi-provider requirement. Replaceability and easier mocking alone do not justify a seam.
- Protected interface: none
- Module depth: direct/local
- Adapter placement: none
- Injection and composition: Keep the direct `date-fns/format` import and invocation in `src/invoices/renderDate.ts`; do not add `IDateFormatter`, `DateFnsAdapter`, a factory, or constructor injection.
- Boundary semantics: Preserve the current synchronous return value and existing date-fns behavior. No validation, error translation, caching, or freshness policy is introduced.
- Cleanup: No production-code change is required. If any proposed formatter interface, adapter, factory, or wiring was added, remove it and restore the single direct import.
- Refactor handoff: none
- Verification: Exercise the exported `renderDate` function with a fixed date and assert the exact established formatted string without mocking `date-fns`. Also retain or add a test for any currently documented invalid-date behavior. Run the repository’s focused test command for `renderDate`; the direct integration, rather than an echoing formatter mock, is the behavioral proof.
`````

#### Final Case B scoring

1. **PASS** — Keeps the dependency direct.
2. **PASS** — Grounds the decision in local use, no semantic leakage, and local deletion.
3. **PASS** — Adds no interface, adapter, factory, registry, or injection mechanism.
4. **PASS** — Rejects third-party status and mock convenience as sufficient evidence.
5. **PASS** — States a consumer-observable formatting verification scenario.

**Total: 5/5.**

### Final Case C exact output

`````text
**Dependency seam decision complete.**
- Protected behavior: Fraud decisions and their caller-visible outcomes must remain stable while provider-specific requests, decisions, and failures are removed from the core API.
- Dependency: `AcmeRiskClient`, `AcmeRiskRequest`, `AcmeRiskDecision`, and `AcmeRiskErrorCode`.
- Decision: earned seam
- Evidence: Acme types currently cross the protected boundary into six callers, provider error codes drive core branching, the approved destination is provider-neutral vocabulary, every caller can be enumerated, and no external compatibility requirement justifies retaining the old path.
- Protected interface: A core-owned capability such as `evaluate(FraudRiskInput): Promise<FraudRiskDecision>`, using only fraud-domain inputs, outcomes, and actionable failures. It must expose neither Acme types nor a provider-shaped operation catalog; rename `RiskProvider` if necessary so the interface describes the core capability rather than the implementation role.
- Module depth: The small core interface hides remote risk evaluation, provider response interpretation, actionable failure semantics, and lifecycle details while the fraud-decision module retains fraud policy in its own vocabulary.
- Adapter placement: Place an Acme implementation beside the Acme/infrastructure side. It translates core input to `AcmeRiskRequest`, validates Acme responses, maps `AcmeRiskDecision` to core outcomes, and translates only actionable `AcmeRiskErrorCode` values into core errors while preserving original causes and diagnostics. Unknown provider failures remain explicit failures.
- Injection and composition: At each existing repository composition root, construct `AcmeRiskClient`, wrap it with the Acme adapter, and inject the core-owned capability into the fraud-decision module using the repository’s existing constructor, factory, or function-parameter convention. Do not add a container, registry, or provider factory.
- Boundary semantics: Keep async behavior explicit in the core contract; preserve currently observable timeout, retry, and failure behavior unless separately approved. Validate untrusted Acme data at the adapter edge, do not silently substitute an approval or denial on dependency failure, keep Acme lifecycle and credentials outside core, and introduce no cache or freshness behavior during the cutover.
- Cleanup: Migrate all six enumerated callers in one behavior-preserving cutover, then delete the deprecated wrappers, old exports, direct `AcmeRiskClient` calls, Acme-shaped public types, error-code switches outside the adapter, stale imports, and duplicate translations. Do not retain parallel direct and adapted paths because there is no compatibility requirement.
- Refactor handoff: clean-cutover capability required; use `refactor-transaction` if available, otherwise use an equivalent behavior-preserving clean-cutover workflow without issuing an unqualified dependency on that capability.
- Verification: Before removal, characterize consumer-observable behavior through the current path for all six callers. Prove the new composed path against the same scenarios: each valid decision, malformed Acme data, every actionable mapped error code, unknown or opaque provider errors with preserved causes, and applicable timeout, retry, lifecycle, cache, freshness, invalidation, and post-expiry behavior. The cutover is complete only when every caller exercises the provider-neutral API with matching observed outcomes and repository search finds no Acme references outside the adapter and composition code.
`````

#### Final Case C scoring

1. **PASS** — Defines the provider-neutral destination before migration.
2. **PASS** — Separates module depth from adapter placement.
3. **PASS** — Places Acme types/errors and translation in the provider-side adapter.
4. **PASS** — Rejects permanent/deprecated wrappers and parallel direct/adapted paths without compatibility ownership.
5. **PASS** — Emits a behavior-preserving clean-cutover capability handoff.
6. **PASS** — Names `refactor-transaction` only as an optional current provider and issues no unqualified invocation.
7. **PASS** — Requires caller enumeration and behavior proof before deleting the old path.

**Total: 7/7.**

### Final Case D exact output

`````text
**Dependency seam decision blocked.**
- Missing decision or evidence: Access to the repository’s composition root and executable verification surface is unavailable, so placement, wiring, cleanup, and behavioral results cannot be observed.
- Why it is material: The boundary is clear, but completion requires proving the real adapter does not leak SDK types, silently convert failures to `false`, retain decisions beyond 30 seconds, or repopulate invalidated cache entries from an older in-flight request.
- Safe work completed: An earned seam is required because untrusted SDK data, provider errors, asynchronous behavior, and cache policy would otherwise enter protected logic. The protected module should own a capability such as `RiskDecisions` with `evaluate(inputs: AccountRiskInputs): Promise<RiskDecisionResult>` and `invalidate(accountId: AccountId): void`. `AccountRiskInputs`, `AccountId`, `RiskDecision`, and `RiskDecisionResult` must be protected-domain types, never SDK response types. `RiskDecisionResult` should expose `{ decision, decidedAt, source: "live" | "last-known" }` so a cached result is distinguishable rather than a silent allow/deny fallback. The provider-side adapter must runtime-validate the SDK’s JSON before translation, including required fields, field types, decision values, and relevant identifiers; malformed success data becomes `RiskUnavailableError` with the validation error retained as its cause and is never cached. A documented rate-limit failure becomes `RiskRateLimitedError` with a validated finite, non-negative `retryAfterMs` and the original SDK error as `cause`; malformed rate-limit metadata becomes `RiskUnavailableError`. Authentication failure becomes `RiskAuthenticationError` with its original cause. Every other failure becomes opaque `RiskUnavailableError` while preserving the original cause. No error may be converted to `false`, `allow`, `deny`, or another successful value. `evaluate` remains explicitly asynchronous. Only successfully validated decisions may be cached. A cached decision may be returned only while its age is at most 30 seconds and must be marked `"last-known"`; once older than 30 seconds, the adapter must call the SDK and surface any resulting failure. No indefinite cache, silent stale-on-error fallback, or background refresh with hidden failures is allowed. `invalidate(accountId)` must immediately evict that account’s decision and advance an account generation/version so SDK calls started before invalidation cannot repopulate the cache after risk inputs change. The adapter belongs beside the SDK integration in the app/infrastructure layer; the protected module contains only the owned interface, result types, and actionable error types. The concrete adapter should be supplied through the repository’s existing constructor, factory, or function-parameter composition mechanism, without a new container or registry. Verification must use the real adapter surface with controlled SDK responses and a controllable clock: valid JSON maps to a live domain decision; a second call within 30 seconds returns a marked last-known decision without another SDK call; a call after expiry invokes the SDK; invalidation forces reevaluation; invalidation during an in-flight request prevents stale repopulation; malformed JSON fails and is not cached; valid rate limiting exposes `retryAfterMs` and preserves cause; malformed rate-limit data becomes unavailable; authentication and opaque failures retain causes; and failures with no eligible cached decision surface rather than returning `false`. If the rejected adapter already exists, migration requires a behavior-preserving clean cutover followed by removal of SDK-shaped core types, catch-all fallback code, indefinite-cache code, and obsolete direct SDK access.
- Required owner: engineering
`````

#### Final Case D scoring

1. **PASS** — Validates untrusted JSON at the adapter edge.
2. **PASS** — Keeps SDK response/error types out of the protected interface.
3. **PASS** — Translates rate-limit and authentication failures only as actionable core errors and preserves original causes.
4. **PASS** — Rejects catch-all `false` and any unapproved fallback.
5. **PASS** — Makes the interface async and states the approved 30-second freshness bound.
6. **PASS** — States risk-input invalidation and post-expiry failure behavior.
7. **PASS** — Places provider translation outside the protected core.
8. **PASS** — States verification for valid, invalid, actionable-error, opaque-error, fresh-cache, invalidation, and expired-cache paths.

**Total: 8/8.**

Case D correctly used the blocked output because the isolated completion had no repository verification surface. Its safe-work field still states every required boundary and verification path. Cases A-C used the complete output. The final enriched file therefore preserves every original behavioral criterion while adding optional catalog enrichment.
