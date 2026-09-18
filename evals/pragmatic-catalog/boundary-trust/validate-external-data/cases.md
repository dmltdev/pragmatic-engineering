# Validate External Data Behavioral Pressure Evidence

## Methodology and run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP `pressure_completion`.
- **Completion selection:** default model.
- **Resolved provider/model:** `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** The post-check completion was stateless, had no tools or repository access, and received only the system prompt and user prompt below.
- **Baseline source:** exact `check-external-data` output from `local://routed-engineering-baselines.json`.
- **Pair comparability:** The prompt and baseline system prompt are identical. The complete check text is the only added post-check input.
- **Evaluated final check:** complete `skills/pragmatic-catalog/checks/boundary-trust/validate-external-data.md`, SHA-256 `9536134538c618b3dd4c74ecf41a6fc54cf252d95105cdbc16d46ec31489d7da`.
- **Scoring:** Manual criterion-by-criterion binary scoring against the exact output. No model judged the output.

### Baseline system prompt

```text
Review the code scenario. Report only concrete defects with evidence; stay silent when safe.
```

### Post-check system prompt rule

The post-check system prompt was the exact baseline system prompt followed by this delimiter and the complete final check text:

```text

Apply the following check as authoritative review guidance for this response:

```

## Evaluation summary

| Condition | Score | Result |
|---|---:|---|
| Baseline | 4/5 | It found the trust-boundary defect and safe comparison, but added an unrelated parse-error finding. |
| Post-check | 5/5 | It proved the trust crossing, rejected static assertion as validation, and kept the validated path finding-free. |

## Case — asserted webhook payload

### Prompt

```text
A webhook handler does JSON.parse(body) as PaymentEvent and immediately branches on event.amount and event.kind. A comparison handler parses JSON, validates it with a runtime schema, rejects invalid input, then passes the validated value to domain logic. Review both.
```

### Binary criteria

1. Reports the first handler because external webhook data reaches trusted logic without runtime validation.
2. Explains that a type assertion does not validate data at runtime and cites a shape or invariant that malformed data can violate.
3. Reports only the demonstrated missing-validation failure under this check.
4. Directs the fix toward runtime validation, rejection, and use of only the validated value.
5. Reports no defect for the handler that validates before domain use.

### Baseline exact output

```text
- **Unsafe type assertion:** `JSON.parse(body) as PaymentEvent` performs no runtime validation. Valid JSON such as `{"amount":"100","kind":null}` is accepted despite violating expected types, so branching and payment logic can behave incorrectly.
- **Unhandled parse failure risk:** Malformed JSON causes `JSON.parse` to throw. If the handler does not catch this, invalid requests become internal errors rather than controlled client rejections.
- **Missing domain validation:** Fields may be absent, unknown, or invalid (for example, negative amounts or unsupported `kind` values) unless explicitly checked.

The comparison handler avoids these defects by validating parsed data and passing only the validated value to domain logic.
```

### Baseline scoring

1. **PASS** — It traces the asserted webhook value into branching and payment logic without validation.
2. **PASS** — It explains that the assertion performs no runtime validation and gives concrete invalid field types.
3. **FAIL** — It adds an unhandled JSON parse failure. That failure does not demonstrate unvalidated data reaching trusted logic.
4. **PASS** — It requires explicit validation before the data is used.
5. **PASS** — It says the comparison handler avoids the defect through validation and validated-value flow.

**Total: 4/5**

### Post-check exact output

```text
- **Webhook handler: Defect.** The request body is untrusted external data. `JSON.parse(body) as PaymentEvent` performs no runtime validation, yet `event.amount` and `event.kind` immediately influence command/domain logic. Malformed payloads can violate the expected `PaymentEvent` shape or invariants—for example, a string/negative `amount` or unsupported/missing `kind`. Parse to `unknown`, validate with a runtime schema, reject failures, and branch only on the validated value.

- **Comparison handler: No defect.** It validates the parsed external payload at the trust boundary, rejects invalid input, and passes only the validated value into domain logic.
```

### Post-check scoring

1. **PASS** — It identifies the request body, the trust crossing, and the unvalidated branch into domain logic.
2. **PASS** — It distinguishes a compile-time assertion from runtime validation and names malformed fields.
3. **PASS** — It reports only the missing-validation failure.
4. **PASS** — It requires parsing to `unknown`, runtime validation, rejection, and validated-value use.
5. **PASS** — It explicitly reports no defect for the validated comparison path.

**Total: 5/5**

The close negative fixture stays finding-free because runtime validation occurs before the value enters trusted logic.

The active check may create a developer-facing finding only when selected and the required trust-crossing evidence is present.
