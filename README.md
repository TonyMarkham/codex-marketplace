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

To verify that the `optimize-plan` skill is installed, check for:

Linux/WSL:

```bash
test -f ~/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/skills/optimize-plan/SKILL.md
```

Windows PowerShell:

```powershell
Test-Path "$env:USERPROFILE\.codex\plugins\cache\online-entity-codex-marketplace\online-entity-codex-plugin\<version>\skills\optimize-plan\SKILL.md"
```

## Plugin

| Name | Description |
| --- | --- |
| `online-entity-codex-plugin` | Codex skills for iterative plan optimization, architecture review, and plugin-building review. |

## Skills

### `optimize-plan`

Optimize a plan document through iterative Codex review/edit passes until stable.

File loop mode reviews and edits a file in place each pass:

```text
Use $optimize-plan on <file> with subagents authorized.
```

Plan mode reads from an input file and writes the complete refined plan to an output file each pass:

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

High-risk review with a stronger first pass:

```text
Use $optimize-plan on <file> with subagents authorized. Use gpt-5.5 medium as orchestrator, gpt-5.5 high for pass 1, and gpt-5.4-mini high for later reviewer subagents.
```

Cheaper smoke review:

```text
Use $optimize-plan on <file> with subagents authorized. Use gpt-5.4-mini medium for orchestrator and reviewer subagents.
```

This is the Codex-supported port of the Claude Code `/optimize-plan` workflow. Codex CLI slash commands are currently built-in commands; this marketplace exposes the workflow as a skill instead.

The fresh-context loop requires explicit subagent authorization in current Codex runtimes. If the prompt does not authorize subagents, the skill may run in single-agent fallback mode and should label that fallback in its final report.

### `plan-refiner`

Review and refine implementation plans by checking evidence, assumptions, sequencing, verification steps, and Codex plugin/runtime constraints.

## Profile Instructions

The plugin ships base instruction Markdown files under:

```text
plugins/online-entity-codex-plugin/base-instructions/
```

Codex does not automatically use those files just because the plugin is installed. Reference them from an active Codex config using `model_instructions_file`.

Example user-level config in `~/.codex/config.toml`, after replacing `<version>` with the installed plugin version:

```toml
[profiles.optimize-plan-orchestrator]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/optimize-plan-orchestrator.md"
model_reasoning_effort = "high"

[profiles.plan-review-subagent]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/plan-review-subagent.md"
model_reasoning_effort = "high"

[profiles.strict-auditor]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/strict-auditor.md"
model_reasoning_effort = "high"

[profiles.architect]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/architect.md"
model_reasoning_effort = "high"

[profiles.plugin-builder]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/<version>/base-instructions/plugin-builder.md"
model_reasoning_effort = "high"
```

The `profiles/` directory in this repo contains editable examples for a local checkout. Adjust paths after install if you want to use the cached plugin copy.

Run a profile with:

```bash
codex --profile optimize-plan-orchestrator
```

The `optimize-plan` skill still includes its review contract inline because current Codex runtimes may not support assigning a separate profile or `model_instructions_file` to each spawned subagent.

## Hooks

`plugins/online-entity-codex-plugin/hooks/hooks.json` is intentionally inert. Current Codex docs describe plugin-local lifecycle config, but plugin hook runtime support should be verified on the installed Codex version before adding active handlers.

## Current Limits

- Codex CLI slash commands are built-in commands. This marketplace does not expose `/optimize-plan`; use `Use $optimize-plan ...`.
- The Codex desktop app may not share WSL2 plugin state. For Windows desktop testing, install and verify the marketplace from Windows PowerShell so the plugin lands under the Windows Codex home.
- Base instruction files are shipped with the plugin, but Codex uses them only when an active `config.toml` profile points at them with `model_instructions_file`.

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
        optimize-plan-orchestrator.md
        plan-review-subagent.md
      skills/
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
    optimize-plan-orchestrator.toml
    plan-review-subagent.toml
```
