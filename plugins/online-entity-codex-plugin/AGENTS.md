# Online Entity Codex Plugin Guidance

This plugin packages reusable Codex workflows. Keep reusable behavior in skills and keep base model personas in `base-instructions/`.

## Plugin Boundaries

- Do not assume plugin install copies files into the current project repository.
- Keep skill instructions self-contained and portable.
- Reference deeper docs from `references/` instead of expanding `SKILL.md` without limit.
- Keep scripts optional and WSL/Linux-friendly.
- Keep lifecycle hooks inert until plugin-local hook execution is verified in the active Codex runtime.
