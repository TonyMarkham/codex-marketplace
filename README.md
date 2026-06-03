# codex-marketplace

A personal Codex plugin marketplace for reusable skills, workflow agents, and base-instruction profiles.

This repository is the Codex counterpart to `TonyMarkham/cc-marketplace`. It uses the Codex marketplace catalog path:

```text
.agents/plugins/marketplace.json
```

The installable unit is the plugin folder:

```text
plugins/online-entity-codex-plugin/
```

## Install

Add this repository as a marketplace:

```bash
codex plugin marketplace add TonyMarkham/codex-marketplace
```

For local development from this checkout:

```bash
codex plugin marketplace add .
```

Then open Codex and use `/plugins` to install `online-entity-codex-plugin` from `Online Entity Codex Marketplace`.

Codex installs plugin bundles into the Codex plugin cache. It does not copy plugin files into the working project repository.

Typical cache location:

```text
~/.codex/plugins/cache/<marketplace-name>/<plugin-name>/<version>/
```

Windows PowerShell equivalent:

```text
$env:USERPROFILE\.codex\plugins\cache\<marketplace-name>\<plugin-name>\<version>\
```

## Update And Verify

After this marketplace repo changes, refresh the registered marketplace:

```bash
codex plugin marketplace upgrade online-entity-codex-marketplace
```

Then open Codex and use `/plugins` to upgrade, reinstall, or verify `online-entity-codex-plugin`.

Some Codex plugin views show only install/uninstall state, not the installed plugin version. Verify the installed version from the plugin cache. The installed directory should match the `version` in:

```text
plugins/online-entity-codex-plugin/.codex-plugin/plugin.json
```

Linux/WSL:

```bash
ls ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin
```

Windows PowerShell:

```powershell
Get-ChildItem "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin"
```

To verify that key skills are installed, check for:

Linux/WSL:

```bash
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/optimize-plan-orchestrator/SKILL.md
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/optimize-plan/SKILL.md
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/colab-audit-plan/SKILL.md
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/guided-implement/SKILL.md
```

Windows PowerShell:

```powershell
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\optimize-plan-orchestrator\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\optimize-plan\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\colab-audit-plan\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\guided-implement\SKILL.md"
```

## Plugin

| Name | Description |
| --- | --- |
| `online-entity-codex-plugin` | Codex skills for plan audits, single-pass and multi-pass plan optimization, guided implementation, architecture review, and plugin-building review. |

Current plugin version:

```text
0.25.3
```

## Skills

### `optimize-plan-orchestrator`

Stabilize a Markdown implementation plan by running repeated `$optimize-plan` passes until the plan reaches three consecutive clean passes, a flip-flop is detected, a blocker appears, or the 20-pass safety limit is hit.

This is the Codex mapping of the opencode `/optimize-plan-orchestrator` command. Codex does not need a repo-local slash command for this workflow; invoke the skill directly:

```text
Use $optimize-plan-orchestrator on <plan-file>.
```

Output-plan mode:

```text
Use $optimize-plan-orchestrator on <input-plan.md> and write the result to <output-plan.md>.
```

The orchestrator does not edit the plan directly. Invoking `$optimize-plan-orchestrator` authorizes its required fresh blind `$optimize-plan` subagents unless the prompt asks for no subagents, fallback, or single-agent mode. Fresh means the worker sees only static workflow instructions and the current plan path/mode, not prior pass summaries, findings, changes, clean streak, flip-flop state, or suppression instructions. Single-agent fallback is used only when requested or accepted and is not reported as blind independent convergence.

Stop conditions:

- three consecutive passes with `ZERO_CHANGES_REQUIRED: yes`
- flip-flop between passes
- unrecoverable blocker or denied required permission
- 20 total passes

### `optimize-plan`

Run one evidence-backed audit-and-edit pass on a Markdown implementation plan. This is the Codex mapping of the opencode `/optimize-plan` command.

In-place mode:

```text
Use $optimize-plan on <plan-file> for one evidence-backed pass.
```

Output-plan mode:

```text
Use $optimize-plan on <input-plan.md> and write the refined plan to <output-plan.md>.
```

The pass must:

- read the plan before editing
- audit concrete claims against repo evidence
- use an independent `audit-plan` custom agent when subagents are authorized, otherwise label the audit as single-agent fallback
- edit only the named plan/output Markdown file
- avoid source, test, config, or unrelated documentation edits
- change only verified material implementation issues, such as wrong paths/APIs, stale assumptions, dependency-order defects, unsupported runtime assumptions, duplicate greenfield implementation, or missing required steps
- avoid style-only rewrites and optional hardening
- return `ZERO_CHANGES_REQUIRED`, `CHANGES_SUMMARY`, `CHAIN_SUMMARY` as visible workflow summary only, `CHANGES_MADE`, and `REMAINING_CONCERNS`

Use `$optimize-plan-orchestrator` when the requested behavior is repeated passes until stable.

### `plan-refiner`

Review and refine implementation plans by checking evidence, assumptions, sequencing, verification steps, and Codex plugin/runtime constraints.

### `guided-implement`

Guide a human through implementing an approved plan one small manual step at a time. Codex acts as a teacher and reviewer, but does not edit files or run mutating commands.

Use it after a plan has been reviewed or optimized:

```text
Use $guided-implement on <plan-file>.
```

Useful variations:

```text
Use $guided-implement on <plan-file>, starting at step <n>.
Use $guided-implement on <plan-file>. Verify my last edit, then give me the next step.
Use $guided-implement on <plan-file>. Focus on the parser section first.
```

The skill preserves the plan's implementation method:

- if the next approved step is a CLI step, it gives exact commands for the user to run manually
- if the next approved step is a file edit, it gives exact manual edit instructions
- it does not convert CLI steps into direct file edits, or direct file edits into CLI steps, unless the plan or repo instructions require it
- it reads existing target files before proposing edits
- it presents exactly one command, edit, verification, or blocked step at a time
- it stops after the step and waits for the user

### `colab-audit-plan`

Audit an implementation plan against the current repo without editing by default.

```text
Use $colab-audit-plan on <plan-file>. Do not edit.
```

Patch only explicitly approved findings:

```text
Use $colab-audit-plan to patch findings A1 and B2 only, then stop.
```

The skill is intentionally separate from `$optimize-plan`. `$colab-audit-plan` is a permission-gated audit checkpoint; `$optimize-plan` is a single audit-and-edit pass; `$optimize-plan-orchestrator` is the repeated stabilization loop.

The audit reports:

- existing repo surface for each major planned behavior
- whether the plan modifies, extends, replaces, or newly adds implementation
- type/file inventory for planned structs, enums, modules, public APIs, and helper types
- implementability gaps where the plan uses broad prose instead of concrete repo-local API, control-flow, data-flow, error-flow, or test-shape guidance
- repo-pattern violations versus intentional new behavior
- existing behavior compatibility risks
- actual plan defects
- unverifiable assumptions

By default it must not edit files, compress the plan, replace implementation detail with prose, or patch unapproved findings.

## Codex Agents And Command Mapping

The opencode harness has first-class command and agent files. The Codex mapping in this repo is:

| Opencode surface | Codex surface in this repo |
| --- | --- |
| `commands/optimize-plan-orchestrator.md` | `$optimize-plan-orchestrator` skill |
| `commands/optimize-plan.md` | `$optimize-plan` skill |
| `commands/audit-plan.md` | `$colab-audit-plan` skill and local `audit-plan` custom agent |
| `commands/guided-implement.md` | `$guided-implement` skill and local `guided-implement` custom agent |
| `agents/*.md` | `.codex/agents/*.toml` for local development, plus plugin `base-instructions/` for session profiles |

Current Codex custom prompts are local to `~/.codex/prompts/` and deprecated in favor of skills, so this marketplace exposes command-like workflows as skills instead of adding repo-local slash commands.

Project-local custom agents live under:

```text
.codex/agents/
```

They are useful when working in this checkout after the project `.codex/` layer is trusted. They are not claimed as plugin-distributed until Codex documents plugin packaging for custom agents.

## Base Instruction Startup

The plugin ships base instruction Markdown files under:

```text
plugins/online-entity-codex-plugin/base-instructions/
```

Codex does not automatically use those files just because the plugin is installed. For a whole-session behavior change, Codex currently needs `model_instructions_file` to point at a real file.

Installed plugin cache paths are versioned. Do not hard-code `<version>` unless you already verified the installed version. Use a shell snippet that locates the installed instruction file first.

PowerShell, collaborative plan auditing:

```powershell
$pluginRoot = "$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin"

$instructions = Get-ChildItem $pluginRoot -Directory |
    ForEach-Object {
        $path = Join-Path $_.FullName "base-instructions/colab-audit-plan.md"
        if (Test-Path $path) { $path }
    } |
    Sort-Object -Descending |
    Select-Object -First 1

codex -c model_instructions_file="$instructions"
```

Linux/macOS/WSL, collaborative plan auditing:

```bash
instructions="$(
  find "$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin" \
    -mindepth 2 \
    -maxdepth 2 \
    -path '*/base-instructions/colab-audit-plan.md' \
    -type f |
  sort -V |
  tail -n 1
)"

codex -c model_instructions_file="$instructions"
```

PowerShell, general collaboration contract:

```powershell
$pluginRoot = "$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin"

$instructions = Get-ChildItem $pluginRoot -Directory |
    ForEach-Object {
        $path = Join-Path $_.FullName "base-instructions/collaboration-contract.md"
        if (Test-Path $path) { $path }
    } |
    Sort-Object -Descending |
    Select-Object -First 1

codex -c model_instructions_file="$instructions"
```

Linux/macOS/WSL, general collaboration contract:

```bash
instructions="$(
  find "$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin" \
    -mindepth 2 \
    -maxdepth 2 \
    -path '*/base-instructions/collaboration-contract.md' \
    -type f |
  sort -V |
  tail -n 1
)"

codex -c model_instructions_file="$instructions"
```

PowerShell, teacher-style implementation guidance:

```powershell
$pluginRoot = "$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin"

$instructions = Get-ChildItem $pluginRoot -Directory |
    ForEach-Object {
        $path = Join-Path $_.FullName "base-instructions/guided-implementation-teacher.md"
        if (Test-Path $path) { $path }
    } |
    Sort-Object -Descending |
    Select-Object -First 1

codex -c model_instructions_file="$instructions"
```

Linux/macOS/WSL, teacher-style implementation guidance:

```bash
instructions="$(
  find "$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin" \
    -mindepth 2 \
    -maxdepth 2 \
    -path '*/base-instructions/guided-implementation-teacher.md' \
    -type f |
  sort -V |
  tail -n 1
)"

codex -c model_instructions_file="$instructions"
```

These startup snippets are the current required startup procedure for whole-session base instructions from an installed plugin: installed plugin files exist on disk, installed paths are versioned, and `model_instructions_file` does not appear to support a stable plugin-resource alias.

Do not use `~` in quoted `-c` values. Shells do not expand it consistently inside quoted strings.

The optimize-plan skills keep their pass contracts inline. Role-specific base instructions are also shipped for sessions or custom agents that can apply them explicitly.

## Profile Templates

`profiles/*.config.toml` use the current Codex profile-file shape: top-level config keys, not legacy `[profiles.<name>]` tables. They are templates for user-level Codex profiles such as:

```text
~/.codex/strict-auditor.config.toml
```

If you copy one outside this repository, replace `model_instructions_file` with an absolute path to the installed plugin cache or to this checkout. Installed plugin paths are versioned, so the startup snippets above are safer for ad hoc use.

## Hooks

`plugins/online-entity-codex-plugin/hooks/hooks.json` is intentionally inert. Current Codex docs describe plugin-local lifecycle config, but plugin hook runtime support should be verified on the installed Codex version before adding active handlers.

## Current Limits

- Repo-distributed command workflows are exposed as skills, not custom prompts. Codex custom prompts are local to `~/.codex/prompts/` and deprecated in favor of skills.
- Project-local custom agents in `.codex/agents/` are for this checkout after the project is trusted. The installed plugin ships skills and base instruction files; do not assume custom agents are plugin-distributed unless current Codex docs/runtime support is verified.
- The Codex desktop app may not share WSL2 plugin state. For Windows desktop testing, install and verify the marketplace from Windows PowerShell so the plugin lands under the Windows Codex home.
- Base instruction files are shipped with the plugin, but Codex uses them for a session only when started with `-c model_instructions_file=...` or another explicit Codex configuration mechanism.

## Structure

```text
codex-marketplace/
  AGENTS.md
  README.md
  .agents/
    plugins/
      marketplace.json
  .codex/
    agents/
      audit-plan.toml
      collaborator.toml
      guided-implement.toml
      optimize-plan.toml
      optimize-plan-orchestrator.toml
    config.toml
    base-instructions.md
  plugins/
    online-entity-codex-plugin/
      .codex-plugin/
        plugin.json
      AGENTS.md
      base-instructions/
        strict-auditor.md
        architect.md
        plugin-builder.md
        collaboration-contract.md
        guided-implementation-teacher.md
        colab-audit-plan.md
        optimize-plan-orchestrator.md
      skills/
        colab-audit-plan/
          SKILL.md
        guided-implement/
          SKILL.md
        optimize-plan/
          SKILL.md
        optimize-plan-orchestrator/
          SKILL.md
        plan-refiner/
          SKILL.md
          scripts/
          assets/
          references/
      hooks/
        hooks.json
      assets/
  profiles/
    strict-auditor.config.toml
    architect.config.toml
    plugin-builder.config.toml
    colab-audit-plan.config.toml
    guided-implementation-teacher.config.toml
    optimize-plan-orchestrator.config.toml
```
