# No N+1 Behavioral Pressure Evidence

## Methodology and run conditions

- **Execution date:** 2026-09-18.
- **Harness:** OMP `pressure_completion`.
- **Completion selection:** default model.
- **Resolved provider/model:** `openai-codex/gpt-5.6-sol`.
- **Settings:** harness defaults; no sampling overrides.
- **Isolation:** The post-check completion was stateless, had no tools or repository access, and received only the system prompt and user prompt below.
- **Baseline source:** exact `check-n-plus-one` output from `local://routed-engineering-baselines.json`.
- **Pair comparability:** The prompt and baseline system prompt are identical. The complete check text is the only added post-check input.
- **Evaluated final check:** complete `skills/pragmatic-catalog/checks/data-access/no-n-plus-one.md`, SHA-256 `2fdc6e45c96bcb7fa2ff7b4d61e3dfc8d2acf14af7ca6e0d57b2e82d2946899d`.
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
| Baseline | 5/5 | The baseline already distinguished cardinality-growing reads from the batched in-memory path. |
| Post-check | 5/5 | The output cited cardinality growth, reported only the violating path, and gave a bounded fix. |

## Case — per-order customer reads

### Prompt

```text
A GraphQL resolver loads 500 orders, then awaits db.customer.findById(order.customerId) inside a loop. A comparison path collects unique customer IDs, performs one batched read, builds a map, then loops over in-memory results. Review both.
```

### Binary criteria

1. Reports the first resolver as an N+1 defect.
2. Cites the 500-item collection, the per-item read, and access growth with collection cardinality.
3. Does not treat loop syntax or `await` alone as sufficient evidence.
4. Directs the fix toward batching and an in-memory join.
5. Reports no defect for the batched read followed by in-memory iteration.

### Baseline exact output

```text
- **N+1 query defect in the first resolver:** 500 orders trigger up to 500 additional `findById` calls. Because each call is awaited inside the loop, they execute serially, compounding database round-trip latency and repeating reads when orders share a customer ID.

The batched path avoids this by reading unique customer IDs once and using in-memory lookups; no concrete defect is evident from the scenario.
```

### Baseline scoring

1. **PASS** — It names the first resolver as an N+1 query defect.
2. **PASS** — It cites 500 orders and up to 500 additional reads.
3. **PASS** — It grounds the finding in per-order database calls, not the loop or `await` alone.
4. **PASS** — It identifies one read for unique IDs plus in-memory lookup as the correct shape.
5. **PASS** — It says no concrete defect exists in the batched path.

**Total: 5/5**

### Post-check exact output

```text
- **Defect in the first path:** The resolver loads a collection of 500 orders, then calls `db.customer.findById(order.customerId)` for each order. This produces up to **501 database accesses**—one to load the orders and 500 customer reads—and grows linearly with the number of orders. Batch the unique customer IDs, fetch them in one query, and join customers to orders in memory.

- **Comparison path:** No defect. It performs one batched customer read followed by in-memory map lookups, so database access count does not grow per order.
```

### Post-check scoring

1. **PASS** — It identifies the per-order customer-read path as the defect.
2. **PASS** — It cites 500 orders, 500 customer reads, and linear access growth.
3. **PASS** — It uses the growing database-access count as evidence.
4. **PASS** — It directs the fix to a batched read and in-memory join.
5. **PASS** — It explicitly reports no defect for the comparison path.

**Total: 5/5**

The baseline legitimately passed. The check preserves that precision and states the proof threshold for future cases where a loop performs only in-memory work.

The active check may create a developer-facing finding only when selected and the required cardinality evidence is present.
