---
name: install-pragmatic-engineering
description: Use when installing, reinstalling, or verifying the local pragmatic-engineering plugin in Pi, OMP, Claude Code, or Codex after its skills or manifests change.
---

# Install Pragmatic Engineering

Install this plugin from its local checkout and return complete installation and activation evidence for every supported harness.

## Boundary and invariant

This skill owns only `pragmatic-engineering` installation. It does not publish the repository, install missing harnesses, alter unrelated plugins, or change a remote marketplace unless the user authorizes the local-source switch.

The five version-bearing manifests must agree before any installation. A missing harness is **skipped**. An available target that was not run is **not attempted**. A successful install command is not activation evidence.

## Workflow

### 1. Resolve and validate the local source

Locate this skill's owning repository and resolve its absolute path as `root`; do not assume the current working directory is the plugin checkout. Confirm the package and plugin names are `pragmatic-engineering` and read all five versions:

```bash
jq -r '.name, .version' package.json
jq -r '.name, .version' .claude-plugin/plugin.json
jq -r '.name, .plugins[0].name, .plugins[0].version' .claude-plugin/marketplace.json
jq -r '.name, .plugins[0].name, .plugins[0].version' .omp-plugin/marketplace.json
jq -r '.name, .version' .codex-plugin/plugin.json
```

Stop before installation when a name is unexpected, a version is empty, or the five versions differ. Confirm every `skills/*/SKILL.md` has `name` and `description`, and that `name` matches its directory.

### 2. Detect targets and current registrations

Check `command -v pi`, `command -v omp`, `command -v claude`, and `command -v codex` independently. Record all four targets. For each available CLI, read current help when its command differs from this skill; use the supported equivalent rather than guessing flags.

Inspect current state before mutation:

| Harness | Inspect |
|---|---|
| Pi | `pi list --approve` from the intended consumer workspace |
| OMP | `omp plugin list --json`; inspect the `pragmatic-engineering` marketplace source |
| Claude Code | `claude plugin marketplace list --json`; `claude plugin list --json` |
| Codex | `codex plugin marketplace list --json`; `codex plugin list --available --json` |

Preserve an intentionally remote marketplace unless the user explicitly requested this local checkout as its replacement. Refresh or remove only the `pragmatic-engineering` registration; never clear shared caches or unrelated marketplaces.

### 3. Validate packaging

Run available native checks from `root` before installation:

```bash
claude plugin validate . --strict --json
claude plugin validate .claude-plugin/plugin.json --strict --json
omp plugin install ./ --dry-run --json
```

A failed available check blocks that harness. A dry run proves only the proposed install, never installed state or activation.

### 4. Install available targets

Use the resolved absolute `root`. If a listed command is unsupported, read that CLI's current help and report the supported command or the exact blocker.

| Harness | Local installation | Required evidence |
|---|---|---|
| Pi | From the intended consumer workspace: `pi install "$root" -l --approve`; `pi list --approve` | Project-local package entry names `root`; report the consumer workspace. |
| OMP | `omp plugin marketplace add "$root"`; `omp plugin install pragmatic-engineering@pragmatic-engineering --force`; `omp plugin list --json`; `omp plugin discover pragmatic-engineering` | Local source, intended version, and expected skill inventory. |
| Claude Code | `claude plugin marketplace add "$root" --scope user`; `claude plugin install pragmatic-engineering@pragmatic-engineering --scope user -y --json`; `claude plugin list --json` | Intended plugin ID/version and local marketplace source. |
| Codex | `codex plugin marketplace add "$root" --json`; `codex plugin add pragmatic-engineering@pragmatic-engineering --json`; `codex plugin list --available --json` | Intended plugin ID/version enabled through the local marketplace. |

If Pi's consumer workspace is unspecified, use `root` for a development-scoped install and report that limitation. Continue collecting evidence for the other harnesses after one target blocks. Preserve each exact error.

### 5. Report activation separately

An installed target is active only when the host exposes and confirms a reload or a fresh session observes the expected plugin. Otherwise report `restart needed` or `unverified`. Do not claim the current agent session refreshed its own plugin inventory.

## Quick reference

| Observation | Status |
|---|---|
| CLI absent | `skipped` |
| Precondition or attempted command failed | `blocked` |
| CLI available but no install command ran | `not attempted` |
| Install plus post-install listing confirms source/version | `installed` |
| Current session still holds old inventory | `restart needed` or `unverified` |

## Output contract

```text
Plugin: pragmatic-engineering
Checkout: resolved absolute local source
Version: intended version and five-manifest agreement
Pi: installed | skipped | blocked | not attempted; command and evidence; workspace
OMP: installed | skipped | blocked | not attempted; command and evidence
Claude Code: installed | skipped | blocked | not attempted; command and evidence
Codex: installed | skipped | blocked | not attempted; command and evidence
Activation: observed reload | restart needed | unverified for each installed target
Checks: observed validation results and remaining limitations
```

Example: `Pi: skipped; command not run; pi absent from PATH` is complete for Pi, but does not imply that the plugin is installed elsewhere. Every report still includes OMP, Claude Code, Codex, activation, and checks.

## Common mistakes

| Rationalization | Required response |
|---|---|
| "The install command exited zero." | Inspect the post-install source, version, and skill inventory before using `installed`. |
| "The README command is probably current." | Read current CLI help when command shape is uncertain. |
| "The remote marketplace is in the way." | Preserve it until the user authorizes replacing this plugin's source. |
| "Three harnesses are enough." | Report all four; absent tools are skips and available unrun tools are not attempted. |
| "The active session already sees the change." | Report activation separately and require observed reload evidence. |

## Red flags

- Any manifest version differs.
- A command was inferred without checking available help.
- A marketplace source changed without authorization.
- One target's success is used as evidence for another.
- A dry run, process presence, or stale listing is described as installation proof.
- The report omits a harness, source path, version, workspace, blocker, or activation state.

## Verification gate

Do not call the local installation complete until the five versions agree, every available native check has an observed result, every target has one allowed status with evidence, each installed target shows the intended local source and version, and activation is reported separately.
