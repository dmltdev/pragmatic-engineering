# No Impossible State Pressure Evidence

## Methodology and matched run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP via the `pressure_completion` eval tool.
- **Completion selection:** selector `default`; resolved provider/model `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** each completion had no tools, repository access, or prior conversation.
- **Pair comparability:** the prompt and baseline system prompt are identical. The post-check system prompt adds only the delimiter below and the complete final check text.
- **Evaluated final check:** `skills/pragmatic-catalog/checks/state-modeling/no-impossible-state.md`, SHA-256 `b2f075b06bafb4d95abecc6782820030a5cfad48be13953db92792ffd128c462`.
- **Scoring:** manual criterion-by-criterion binary scoring against the exact outputs. No model judge or partial credit was used.

### Baseline system prompt

```text
You are a senior engineering assistant. Review the code scenario. Report only concrete defects with evidence; stay silent when safe.
```

### Post-check system prompt rule

The post-check system prompt was the exact baseline system prompt followed by this delimiter and then the complete final check text:

```text

Apply the following check as authoritative engineering guidance for this response:

```

Exact construction: `baseSystem + "\n\nApply the following check as authoritative engineering guidance for this response:\n\n" + checkText`.

## Evaluation summary

| Condition | Score | Result |
|---|---:|---|
| Baseline | 5/7 | Correctly found the reachable contradiction and left the safe representation finding-free, but omitted the stable check ID, confidence, and severity. |
| Post-check | 7/7 | Reported every required finding field and explicitly left the closed representation finding-free. |

## Case — reachable contradictory job state

### Prompt

```text
A job record stores isQueued, isRunning, and isComplete as independent booleans. Public setters allow isRunning=true and isComplete=true at the same time. A nearby version uses a closed status union: queued | running | complete, changed only by checked transition functions. Review both.
```

### Binary criteria

1. Names stable check ID `state-modeling/no-impossible-state`.
2. Cites the public mutation path and the incompatible reachable values.
3. Demonstrates the violated single-lifecycle-state invariant.
4. States a contextual consumer impact.
5. States confidence and severity.
6. Gives a bounded correction direction.
7. Produces no finding for the closed union with checked transitions.

### Baseline exact output

```text
- **Defect:** Independent public boolean setters allow contradictory job states, such as `isRunning === true` and `isComplete === true` simultaneously. Consumers cannot reliably determine the job’s lifecycle state, and invalid transitions cannot be centrally enforced. The closed status union with checked transition functions avoids this defect.
```

### Baseline scoring

1. **FAIL** — No stable check ID appears.
2. **PASS** — It cites independent public setters and the simultaneous `isRunning` and `isComplete` values.
3. **PASS** — It identifies the values as contradictory lifecycle states.
4. **PASS** — It explains that consumers cannot reliably determine lifecycle state.
5. **FAIL** — It states neither confidence nor severity.
6. **PASS** — It points to a closed union and checked transitions.
7. **PASS** — It says the comparison representation avoids the defect and raises no finding against it.

**Total: 5/7**

### Post-check exact output

```text
### Finding: Independent booleans permit contradictory lifecycle states

- **Rule:** `state-modeling/no-impossible-state`
- **Affected version:** `isQueued`, `isRunning`, and `isComplete` booleans with public setters
- **Construction/transition path:** Call `setIsRunning(true)`, then `setIsComplete(true)`.
- **Reachable incompatible values:** `{ isRunning: true, isComplete: true }`; similarly, all flags may be false or multiple flags may be true.
- **Violated invariant:** A job must occupy exactly one lifecycle state—`queued`, `running`, or `complete`.
- **Impact:** Consumers can interpret the same job as both active and terminal, causing duplicate execution, incorrect scheduling, misleading UI, or invalid transition decisions.
- **Severity:** High
- **Confidence:** High
- **Bounded fix:** Replace the three booleans with one closed `status: "queued" | "running" | "complete"` field and mutate it only through centralized transition functions that validate allowed lifecycle changes.

The nearby closed-union version preserves the stated invariant through checked transitions; no defect is identified there.
```

### Post-check scoring

1. **PASS** — It names `state-modeling/no-impossible-state`.
2. **PASS** — It cites sequential public setter calls and the resulting incompatible values.
3. **PASS** — It states the exactly-one-lifecycle-state invariant.
4. **PASS** — It names duplicate work, inconsistent UI, and invalid scheduling as contextual consequences.
5. **PASS** — It states high confidence and high severity.
6. **PASS** — It bounds the fix to one closed status and checked transition functions.
7. **PASS** — It explicitly says the nearby closed representation has no defect.

**Total: 7/7**
