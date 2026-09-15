# Pragmatic Engineering

Portable agent skills plugin for pragmatic engineering decisions, prioritization, dependency seams, and evidence-backed quality gates.

Core invariant:

> Agents must make the smallest defensible engineering decision and show the evidence boundary that makes it safe.

## Skills

| Group | Skills | Purpose |
|---|---|---|
| Decisions | `decision-framework` | Choose, prioritize, or structure work trade-offs under uncertainty without fake certainty, framework soup, or silent product/business priority theft. |
| Boundaries | `dependency-seam` | Decide whether core/protected logic should couple directly to app/provider/foreign semantics or earn a seam with owned vocabulary and proof. |

## Install

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
