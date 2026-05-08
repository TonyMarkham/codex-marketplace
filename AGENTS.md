# Codex Marketplace Repo Guidance

This repository is a personal Codex plugin marketplace. Keep the structure simple and reviewable.

## Repository Shape

- Use `.agents/plugins/marketplace.json` as the Codex marketplace catalog.
- Treat each `plugins/<name>/` directory as the installable unit.
- Keep each plugin manifest at `plugins/<name>/.codex-plugin/plugin.json`.
- Keep `AGENTS.md` files for repository or project guidance only. Do not use them as base model instruction profiles.
- Put reusable workflows in Codex skills under `skills/<skill-name>/SKILL.md`.
- Put custom base instruction Markdown files in plugin-owned `base-instructions/`, then reference them from Codex config with `model_instructions_file`.
- Keep hook files inert unless plugin-local hook execution is verified for the current Codex version.

## Editing Rules

- Prefer WSL/Linux-friendly relative paths.
- Do not add clever generation layers unless the repo grows enough to justify them.
- When adding plugin metadata, keep names in lower-case kebab-case.
- Keep JSON valid and avoid comments in JSON files.
- Document anything that requires user-level `~/.codex/config.toml` setup in `README.md`.
