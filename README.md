# codex-marketplace

A personal Codex plugin marketplace for reusable workflows, skills, and profile instruction files.

This repository is the Codex counterpart to `TonyMarkham/cc-marketplace`. It uses the Codex-native marketplace path:

```text
.agents/plugins/marketplace.json
```

## Install

Add this repository as a Codex marketplace:

```bash
codex plugin marketplace add TonyMarkham/codex-marketplace
```

For local development from a clone:

```bash
codex plugin marketplace add .
```

Then open Codex and use `/plugins` to install `online-entity-codex-plugin` from `Online Entity Codex Marketplace`.

Codex installs plugin bundles into its plugin cache, not into the working project repository. Local plugins are loaded from a cache path like:

```text
~/.codex/plugins/cache/<marketplace-name>/<plugin-name>/local/
```

## Plugin

| Name | Description |
| --- | --- |
| `online-entity-codex-plugin` | Codex workflows and commands for iterative plan refinement, architecture review, and plugin-building review. |

## Commands

### `/optimize-plan`

Optimize a plan document through iterative fresh-context Codex review/edit passes until stable.

File loop mode reviews and edits a file in place each pass:

```text
/optimize-plan <file>
```

Plan mode reads from an input file and writes the complete refined plan to an output file each pass:

```text
/optimize-plan <input_file> <output_file>
```

Optional GPT/Codex controls:

```text
/optimize-plan <file> --model gpt-5.5 --reasoning high
```

This is the Codex-native port of the Claude Code `/optimize-plan` command. It uses Codex/GPT model terminology and requires Codex subagent support for the fresh-context pass loop.

## Profile Instructions

The plugin ships base instruction Markdown files under:

```text
plugins/online-entity-codex-plugin/base-instructions/
```

Codex does not automatically use those files just because the plugin is installed. Reference them from an active Codex config using `model_instructions_file`.

Example user-level config in `~/.codex/config.toml`:

```toml
[profiles.strict-auditor]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/local/base-instructions/strict-auditor.md"
model_reasoning_effort = "high"

[profiles.architect]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/local/base-instructions/architect.md"
model_reasoning_effort = "high"

[profiles.plugin-builder]
model_instructions_file = "/home/tony/.codex/plugins/cache/online-entity-codex-marketplace/online-entity-codex-plugin/local/base-instructions/plugin-builder.md"
model_reasoning_effort = "high"
```

The `profiles/` directory in this repo contains editable examples for a local checkout. Adjust paths after install if you want to use the cached plugin copy.

Run a profile with:

```bash
codex --profile strict-auditor
```

## Hooks

`plugins/online-entity-codex-plugin/hooks/hooks.json` is intentionally inert. Current Codex docs describe plugin-local lifecycle config, but plugin hook runtime support should be verified on the installed Codex version before adding active handlers.

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
      commands/
        optimize-plan.md
      skills/
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
```
