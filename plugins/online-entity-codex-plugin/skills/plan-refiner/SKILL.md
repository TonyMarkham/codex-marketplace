---
name: plan-refiner
description: Review and refine implementation plans by checking evidence, assumptions, sequencing, verification steps, and Codex plugin/runtime constraints.
---

# Plan Refiner

Use this skill when the user asks to refine, audit, or pressure-test an implementation plan.

## Workflow

1. Restate the target outcome in one sentence.
2. Identify the hard constraints that the plan must preserve.
3. Separate confirmed facts from assumptions.
4. Check whether each planned integration point is supported by local code, runtime behavior, or current official documentation.
5. Rewrite the plan into small implementation steps only after the evidence is clear.
6. Add validation commands or manual checks for every runtime-sensitive part.
7. Call out unknowns explicitly instead of smoothing over them.

## Review Focus

- Installable units and discovery paths.
- Config files that Codex actually reads.
- Whether files are shipped, installed, copied, cached, or merely documented.
- Runtime feature flags and current CLI behavior.
- Paths that work in WSL/Linux shells.
- Places where docs and runtime behavior might diverge.

## Output Shape

Use concise sections:

- Confirmed facts
- Plan changes
- Implementation checklist
- Validation
- Unknowns

Lead with blocking risks if any exist. If the plan is sound, say that and keep the refinement focused on execution details.
