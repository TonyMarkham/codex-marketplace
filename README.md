# codex-marketplace

A personal Codex CLI plugin marketplace for reusable skills, workflows, and profile instruction files.

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
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/optimize-plan/SKILL.md
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/colab-audit-plan/SKILL.md
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/guided-implement/SKILL.md
```

Windows PowerShell:

```powershell
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\optimize-plan\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\colab-audit-plan\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\guided-implement\SKILL.md"
```

## Plugin

| Name | Description |
| --- | --- |
| `online-entity-codex-plugin` | Codex skills for iterative plan optimization, guided implementation, architecture review, and plugin-building review. |

Current plugin version:

```text
0.19.0
```

## Skills

### `optimize-plan`

Optimize a plan document through iterative Codex review/edit passes until stable.

File loop mode reviews a file and applies only gated material fixes in place:

```text
Use $optimize-plan on <file> with subagents authorized.
```

Plan mode reads from an input file and writes a complete refined plan to an output file:

```text
Use $optimize-plan on <input_file> and write the result to <output_file> with subagents authorized.
```

Optional GPT/Codex preferences can be included in plain language:

```text
Use $optimize-plan on <file> with subagents authorized, gpt-5.5, and high reasoning.
```

Recommended efficient model split:

```text
Use $optimize-plan on <file> with subagents authorized. Use the current model as orchestrator and use gpt-5.4-mini high for reviewer subagents.
```

High-risk review with a stronger first reviewer:

```text
Use $optimize-plan on <file> with subagents authorized. Use gpt-5.5 medium as orchestrator, gpt-5.5 high for the first reviewer, and gpt-5.4-mini high for later reviewer subagents.
```

Cheaper smoke review:

```text
Use $optimize-plan on <file> with subagents authorized. Use gpt-5.4-mini medium for orchestrator and reviewer subagents.
```

This is the Codex-supported port of the Claude Code `/optimize-plan` workflow. Codex CLI slash commands are currently built-in commands; this marketplace exposes the workflow as a skill instead.

The fresh-context loop requires explicit subagent authorization in current Codex runtimes. If the prompt does not authorize subagents, the skill may run in single-agent fallback mode and should label that fallback in its final report.

#### Optimize-plan behavior

The orchestrator controls the loop. Reviewer subagents perform individual review passes.

Reviewers receive neutral workflow context: target path, mode, baseline state, prior reviewer outcomes, and permission patterns. They should not be told their ordinal pass number or whether they are early or late in the loop.

Each reviewer pass must:

- read the plan before judging it
- verify concrete claims against the repository, docs, APIs, and runtime behavior when needed
- identify the existing repo surface for each major planned behavior before accepting the plan's implementation shape
- classify planned work as modifying, extending, replacing, or newly adding implementation
- treat duplicate greenfield implementations of existing behavior as blocking unless replacement is justified with repo evidence
- check that the plan follows repo-local patterns for modules, APIs, data flow, errors, tests, and ownership
- classify proposed edits before editing as `BLOCKING`, `MATERIAL`, `MINOR`, or `OPTIONAL`
- apply only `BLOCKING` and `MATERIAL` changes
- apply `MINOR` changes only when directly adjacent to an applied material fix
- skip `OPTIONAL` changes
- report skipped minor/optional proposals under `SKIPPED_CHANGES`
- report runtime approval prompts under `PERMISSIONS_REQUESTED`

Before the first reviewer, the orchestrator records the target file baseline: path, existence, VCS status when available, and a diff stat or fingerprint. The final report compares the final target state to that baseline, so a modified file is reported as one of:

- changed during optimization
- already modified before optimization
- reporting mismatch, if the file changed but all reviewers reported `CHANGES_MADE: none`

The orchestrator must forward each raw reviewer report unchanged before summarizing or adjudicating it, then close that completed reviewer before starting the next one. After forwarding the raw report, the orchestrator separately classifies the pass as:

```text
BLOCKING | MATERIAL | MINOR_ONLY | CLEAN | FLIP_FLOP
```

Only orchestrator-classified `CLEAN` passes count toward convergence. `CLEAN` means the reviewer found no material issue and made no file edits; `MINOR_NOTES` and `SKIPPED_CHANGES` may still be present. `MINOR_ONLY` does not count as a no-change pass.

Convergence rules:

- Stop after three consecutive `CLEAN` orchestrator classifications.
- Stop immediately on `FLIP_FLOP`.
- Stop at the safety limit of 20 reviewer passes.
- `BLOCKING`, `MATERIAL`, and `MINOR_ONLY` reset the no-change streak.

Permission handling is advisory, not a bypass. If a reviewer reports an approved reusable command pattern, the orchestrator passes that pattern to later reviewers so they can keep command shapes stable, but Codex may still ask for runtime approval.

If a runtime clearly supports a same-thread two-phase reviewer pass, the orchestrator may ask for proposed changes first, adjudicate those proposals, then send the same reviewer back to apply approved material fixes. Otherwise, the reviewer-side pre-edit gate is the supported default.

### `plan-refiner`

Review and refine implementation plans by checking evidence, assumptions, sequencing, verification steps, and Codex plugin/runtime constraints.

### `guided-implement`

Guide a human through implementing an approved plan one small manual coding step at a time. Codex acts as a teacher and reviewer, but does not edit files.

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

The skill must:

- read the plan before selecting the next step
- read every existing target file before proposing edits to it
- present exactly one implementation step at a time
- avoid summarizing future steps after presenting the current step
- avoid writing, patching, formatting, or deleting files
- never infer edit permission from a plan, prior approval, or surrounding context
- avoid authority performance, lecturing, scolding, and defensive replies
- include the target path relative to the repo root
- prefer `Find` / `Replace` instructions for existing files
- use exact `Insert After`, `Insert This`, and `Insert Before` landmarks for insertions
- stop after each step and wait for the user
- stop immediately if the user says no, wait, stop, or challenges whether Codex should proceed

This is intentionally separate from `optimize-plan`: `optimize-plan` hardens the plan, while `guided-implement` teaches the user through applying the already-approved plan manually.

### `colab-audit-plan`

Audit an implementation plan against the current repo without editing by default.

```text
Use $colab-audit-plan on <plan-file>. Do not edit.
```

Patch only explicitly approved findings:

```text
Use $colab-audit-plan to patch findings A1 and B2 only, then stop.
```

The skill is intentionally separate from `optimize-plan`. `optimize-plan` is an iterative review/edit loop. `colab-audit-plan` is a permission-gated audit checkpoint for proving whether a plan matches existing repo behavior and patterns before any patching happens.

The audit reports:

- existing repo surface for each major planned behavior
- whether the plan modifies, extends, replaces, or newly adds implementation
- repo-pattern violations versus intentional new behavior
- existing behavior compatibility risks
- actual plan defects
- unverifiable assumptions

By default it must not edit files, compress the plan, replace implementation detail with prose, or patch unapproved findings.

For a one-off audit inside a normal Codex session, invoke the skill directly:

```text
Use $colab-audit-plan on <plan-file>. Do not edit.
```

For an entire collaborative planning session without editing config, start Codex with the base instruction file directly. This form works in both PowerShell and POSIX shells because both expand `$HOME` inside double quotes; forward slashes are accepted by Windows path handling and by Linux/macOS:

```bash
codex -c model_instructions_file="$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/colab-audit-plan.md"
```

Do not use `~` in this quoted `-c` value. Shells do not expand it consistently inside quoted strings.

## Base Instruction Startup

The plugin ships base instruction Markdown files under:

```text
plugins/online-entity-codex-plugin/base-instructions/
```

Codex does not automatically use those files just because the plugin is installed. Start Codex with `-c model_instructions_file=...` when you want the whole session to use one of them.

For a session that defaults to manual, teacher-style implementation guidance, replace `<version>` with the installed plugin version:

```bash
codex -c model_instructions_file="$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/guided-implementation-teacher.md"
```

For a session that defaults to collaborative plan auditing and permission-gated plan patching:

```bash
codex -c model_instructions_file="$HOME/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/colab-audit-plan.md"
```

The `optimize-plan` skill still includes its review contract inline because current Codex runtimes may not support assigning a separate profile or `model_instructions_file` to each spawned subagent.

## Hooks

`plugins/online-entity-codex-plugin/hooks/hooks.json` is intentionally inert. Current Codex docs describe plugin-local lifecycle config, but plugin hook runtime support should be verified on the installed Codex version before adding active handlers.

## Current Limits

- Codex CLI slash commands are built-in commands. This marketplace does not expose `/optimize-plan`; use `Use $optimize-plan ...`.
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
  plugins/
    online-entity-codex-plugin/
      .codex-plugin/
        plugin.json
      AGENTS.md
      base-instructions/
        strict-auditor.md
        architect.md
        plugin-builder.md
        guided-implementation-teacher.md
        colab-audit-plan.md
        optimize-plan-orchestrator.md
        plan-review-subagent.md
      skills/
        colab-audit-plan/
          SKILL.md
        guided-implement/
          SKILL.md
        optimize-plan/
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
    strict-auditor.toml
    architect.toml
    plugin-builder.toml
    colab-audit-plan.toml
    guided-implementation-teacher.toml
    optimize-plan-orchestrator.toml
    plan-review-subagent.toml
```
