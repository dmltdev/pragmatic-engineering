---
name: decision-framework
description: Use when choosing between work options, prioritizing engineering or product work, structuring a decision tree, resolving trade-offs, breaking analysis paralysis, or making a recommendation under uncertainty.
---

# Decision Framework

Make the smallest useful work decision under uncertainty without fake certainty, framework soup, endless analysis, or silent theft of human-owned priorities.

> A decision framework earns its keep only when it makes the next commitment clearer. If it adds ceremony without changing the choice, do not use it.

## When to use

Use this skill when:

- a work decision has multiple plausible options;
- engineering, product, delivery, cost, risk, or prioritization trade-offs conflict;
- a decision tree, matrix, prioritization pass, reversibility check, or premortem is requested;
- the user asks to decide, choose, rank, prioritize, evaluate options, or force a recommendation; or
- analysis is looping and a bounded commitment is needed.

## Do not use

Do not use this skill for:

- personal life decisions;
- standalone business strategy where the agent lacks owned business priorities;
- legal, medical, financial, employment, safety, or other regulated professional determinations;
- implementation planning after the decision is already settled;
- durable ADR writing itself; or
- dependency-boundary decisions where `dependency-seam` is the tighter skill.

For product or business implications, reason from stated goals and surface assumptions. The human owns values such as risk tolerance, ethics, opportunity cost, revenue versus trust, and speed versus quality.

## Required inputs

Before deciding, establish:

- the decision question;
- the realistic options, including the boring/default option;
- the decision owner;
- hard constraints and deadlines;
- what happens if no explicit decision is made;
- current evidence and missing evidence;
- human-owned values that could change the answer; and
- whether the decision is reversible, high consequence, or hard to observe after release.

Derive facts from available code, docs, data, and user-provided context before asking. Ask only when the missing value or fact can change the recommendation.

## Framework router

Pick exactly one primary framework. A secondary check is allowed only when labeled `Sanity check` and kept shorter than the primary analysis.

| Situation | Primary framework | Cap |
|---|---|---|
| 2-3 options, reversible, low/medium stakes | Short recommendation | 3 trade-offs |
| 2-5 serious options with competing criteria | Decision matrix | 5 criteria, rough ordinal scores only |
| More than 5 work items need ordering | Prioritization pass | 4 factors: impact, urgency, confidence, effort |
| Branching conditions determine the answer | Decision tree | each branch has condition, action, fallback, owner |
| High-stakes, irreversible, public API, persistence, security, or cross-team consequence | Reversibility gate + premortem | 5 failure modes |
| Team is stuck but facts are unlikely to improve soon | Commitment frame | recommend, delegate, or block |
| Engineering boundary involves app/provider/foreign semantics | Handoff | use `dependency-seam` instead |
| Durable engineering decision evidence is needed after a recommendation | Handoff | use ADR or `record-decision` if available |

### Decision matrix rules

- Use ordinal scores: low, medium, high. Do not compute fake-precise totals unless the inputs are measured.
- State which criterion dominates the recommendation.
- If changing one human-owned weight flips the result, give conditional recommendations instead of one false answer.

### Prioritization rules

- Prioritize problems or outcomes before solutions when possible.
- Use effort as a cost, not a virtue.
- Do not rank long lists item-by-item in prose; group into now / next / later / drop.

### Decision tree rules

Every branch needs:

```text
Condition:
Action:
Fallback:
Owner:
```

Missing data is a branch, not an afterthought.

### Premortem rules

For high-stakes or hard-to-reverse decisions, name:

- what failure looks like;
- the leading indicator;
- the mitigation;
- the rollback or stop condition;
- the owner watching it.

## Output contract

On completion, report exactly:

```markdown
**Decision framework complete.**
- Decision question: <one sentence>
- Selected framework: <framework and why it is the smallest useful one>
- Options considered: <realistic options, including default>
- Recommendation: <one option, or conditional recommendations if human values decide>
- Confidence: <low | medium | high, with reason>
- Key trade-offs: <bounded list>
- Human-owned values: <values/priorities the agent must not decide silently>
- Unknowns that could change the decision: <facts or none>
- Smallest reversible next step: <concrete action>
- Record/handoff: <none | dependency-seam | ADR/record-decision | other named owner>
```

If the skill cannot decide safely, report exactly:

```markdown
**Decision framework blocked.**
- Decision question: <one sentence>
- Missing decision or evidence: <one concrete item>
- Why it is material: <how it changes the recommendation>
- Safe recommendation until resolved: <default, reversible step, or wait>
- Required owner: <user | product | engineering | security | legal | finance | leadership>
```

## Stop conditions

Stop with the blocked report when:

- missing human-owned values would flip the recommendation;
- regulated or professional advice would be required;
- the requested decision owner is absent;
- evidence is unavailable and the wrong call is hard to reverse; or
- the task is actually implementation planning, ADR writing, or dependency-seam design.

Otherwise end with a recommendation. Do not keep asking questions because more context might be nice.

## Example

Prompt: “Should we add a generic plugin registry for future providers, keep direct imports, or build one adapter for the current payment provider? Product wants speed this week; engineering worries about lock-in; leadership says cost matters but has not ranked speed versus maintainability.”

```markdown
**Decision framework complete.**
- Decision question: Should payment integration use a generic registry, direct imports, or a current-provider adapter?
- Selected framework: Decision matrix; three options have competing speed, lock-in, and cost criteria.
- Options considered: generic plugin registry; direct imports; one current-provider adapter.
- Recommendation: Build one current-provider adapter. Do not build the registry yet.
- Confidence: medium; current constraints favor speed and low complexity, but leadership has not ranked speed versus maintainability.
- Key trade-offs: adapter preserves a cheap escape hatch; direct imports maximize immediate speed but spread provider semantics; registry reduces future switching cost only if a second provider or runtime selection is real.
- Human-owned values: speed versus maintainability; how much lock-in risk leadership accepts.
- Unknowns that could change the decision: committed second provider, runtime provider switching, compliance-driven routing, or an explicit business priority for future optionality over this week's delivery.
- Smallest reversible next step: define the narrow payment operations needed this week and keep provider SDK types inside one adapter module.
- Record/handoff: dependency-seam if protected/core logic would consume provider SDK types; ADR only if the boundary becomes public or hard to reverse.
```

## Common rationalizations

| Rationalization | Reality |
|---|---|
| “Use several frameworks to be thorough.” | Pick one primary framework. More frames often hide uncertainty instead of reducing it. |
| “The score says this wins.” | Scores reflect chosen criteria and weights. State who owns those values. |
| “We just need more context.” | Ask only for facts or values that can change the recommendation. |
| “The business answer is obvious.” | Business priorities are inputs from an owner, not agent-owned truth. |
| “An ADR is the decision framework.” | ADRs preserve decisions after analysis; they do not replace analysis. |

## Red flags

- Output has no recommendation or blocker.
- Output mixes matrix, tree, premortem, and scoring without naming one primary framework.
- Confidence is absent.
- Unknowns are absent or phrased as generic caveats.
- Human-owned values are hidden inside numeric weights.
- The agent chooses product/business priorities the user did not state.
- A reversible low-stakes decision receives high-ceremony analysis.
- A hard-to-reverse decision receives only a gut-feel recommendation.

## Verification gate

Do not call the decision complete unless the output names the selected framework, recommendation, confidence, human-owned values, unknowns that could change the recommendation, smallest reversible next step, and any required handoff.
