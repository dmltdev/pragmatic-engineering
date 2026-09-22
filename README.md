# Pragmatic Engineering

Portable agent skills plugin for pragmatic engineering decisions, prioritization, dependency seams, and evidence-backed quality gates.

Core invariant:

> Agents must make the smallest defensible engineering decision and show the evidence boundary that makes it safe.

## Skills

| Group | Skills | Purpose |
|---|---|---|
| Decisions | `decision-framework` | Choose, prioritize, or structure work trade-offs under uncertainty without fake certainty, framework soup, or silent product/business priority theft. |
| Boundaries | `dependency-seam` | Decide whether core/protected logic should couple directly to app/provider/foreign semantics or earn a seam with owned vocabulary and proof. |
| Routing | `pragmatic-catalog`, `derive-catalog-candidates` | Select active evidence-backed checks from change-specific risk and derive isolated reusable-rule candidates from bounded local evidence. |
| Workflows | `pragmatic-coding`, `pragmatic-review` | Add bounded coding guardrails and run evidence-only engineering review. |
| Optional workflows | `pragmatic-consulting`, `pragmatic-review-advanced` | Challenge designs through approved external consultation or run an explicit, slower advanced review. |
| Feature readiness | `create-feature-checklist`, `check-feature-readiness` | Create repository-specific readiness criteria from versioned fallbacks, then evaluate supplied evidence without executing verification or taking product acceptance ownership. |
| Operations | `install-pragmatic-engineering` | Install or verify this local plugin across Pi, OMP, Claude Code, and Codex with per-target evidence. |

## Routed engineering checks

The routed system uses six installable skills:

1. `derive-catalog-candidates` derives isolated candidate packets from bounded local evidence without changing catalog guidance.
2. `pragmatic-catalog` selects checks and resolves policy, variants, conflicts, and budgets.
3. `pragmatic-coding` adds zero to four preventive checks for non-trivial implementation work.
4. `pragmatic-review` runs focused category passes and requires local evidence for every finding.
5. `pragmatic-consulting` optionally challenges a design after explicit invocation or project opt-in.
6. `pragmatic-review-advanced` runs only after explicit user invocation and adds slower independent analysis before `pragmatic-review`.

`derive-catalog-candidates` accepts a bounded bug-fix, feedback, incident, or technical source set and returns a packet for explicit maintainer import. It never writes a candidate or mutates the installed catalog.

Checks define reusable failure conditions. Categories own review passes. Tags route mechanisms and ecosystems. Variants narrow one check for a specific scope. Project policies can override reusable guidance in an explicit scope.

A catalog candidate is a discovered check that is not installed. It must pass normalization, conflict, example, and fixture gates before shadow use. A review candidate is an untrusted code observation from advanced review; it is not a catalog candidate or a finding. An advanced review packet is the private handoff that records the immutable target, file coverage, warnings, review candidates, rejected candidates, and unresolved uncertainty. A finding exists only after `pragmatic-review` proves the failure under a stable check and supplies evidence, impact, confidence, severity, and a bounded fix direction.

The recommended installation contains `pragmatic-catalog`, `pragmatic-coding`, and `pragmatic-review`. Install `derive-catalog-candidates` when a catalog maintainer needs to turn bounded local evidence into an importable candidate packet. Domain skills remain useful alone. Coding continues with a disclosure when the catalog is missing. Review blocks when the catalog is missing. Install consulting only where external design consultation is permitted. Install advanced review only on hosts that can enforce manual invocation and provide its private dependencies.

## Install

Use [`install-pragmatic-engineering`](skills/install-pragmatic-engineering/SKILL.md) after changing skills or manifests. It defaults to this local checkout, requires aligned manifest versions, treats unavailable harnesses as skips, and reports installation and activation evidence separately for Pi, OMP, Claude Code, and Codex.

### skills.sh

Install the whole skill pack:

```bash
npx skills add dmltdev/pragmatic-engineering --skill '*'
```

Install one skill:

```bash
npx skills add dmltdev/pragmatic-engineering --skill decision-framework
```

Local development install from this repository:

```bash
npx skills add ./pragmatic-engineering --skill '*'
```

Useful options:

- `-g` => install globally.
- `-a claude-code` / `-a codex` => target one agent.
- `--copy` => copy instead of symlinking.
- `-y` => skip prompts.
- `DISABLE_TELEMETRY=1` => opt out of anonymous skills.sh telemetry.

### Claude Code

Install from the plugin marketplace metadata:

```text
/plugin marketplace add dmltdev/pragmatic-engineering
/plugin install pragmatic-engineering@pragmatic-engineering
```

For local development:

```text
/plugin marketplace add /path/to/pragmatic-engineering
/plugin install pragmatic-engineering@pragmatic-engineering
```

### pi

Install from Git:

```bash
pi install git:github.com/dmltdev/pragmatic-engineering
```

Install project-locally:

```bash
pi install -l git:github.com/dmltdev/pragmatic-engineering
```

Run a one-off pi session with the package loaded:

```bash
pi -e git:github.com/dmltdev/pragmatic-engineering
```

### omp

Install from the plugin marketplace metadata:

```bash
omp plugin marketplace add dmltdev/pragmatic-engineering
omp plugin install pragmatic-engineering@pragmatic-engineering
```

For local development:

```bash
omp plugin marketplace add /path/to/pragmatic-engineering
omp plugin install pragmatic-engineering@pragmatic-engineering --force
```

### Codex

Install from the plugin marketplace metadata:

```bash
codex plugin marketplace add dmltdev/pragmatic-engineering --ref main
codex plugin add pragmatic-engineering@pragmatic-engineering
```

For local development:

```bash
codex plugin marketplace add /path/to/pragmatic-engineering
codex plugin add pragmatic-engineering@pragmatic-engineering
```

## Source layout

```text
skills/                         canonical installable skills
skills/registry.json            skill domain index
.claude-plugin/                 Claude Code plugin metadata
.codex-plugin/                  Codex plugin metadata
.agents/plugins/                Codex marketplace metadata
.omp-plugin/                    omp marketplace metadata
package.json                    pi package metadata
evals/                          behavioral pressure evidence
```

## Responsibility boundary

This package owns reusable engineering disciplines, principles, references, and evidence gates, while product discovery/acceptance, lifecycle state, personas, execution profiles, git/forge/deployment/session tooling, and host packaging behavior remain elsewhere.

## Clean-cutover capability

When existing code already leaks provider semantics, `dependency-seam` decides the destination and hands off the behavior-preserving migration as a clean-cutover capability. Void Grimoire’s `refactor-transaction` is the current optional provider; it is not a dependency of this repository and is not invoked automatically.

## License

MIT. See [LICENSE](LICENSE).
