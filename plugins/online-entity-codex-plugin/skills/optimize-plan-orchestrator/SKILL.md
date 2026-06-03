---
name: optimize-plan-orchestrator
description: Stabilize Markdown implementation plans by running repeated optimize-plan passes until three clean passes, flip-flop, a blocker, or twenty passes. Use when the user asks to optimize until stable or run the old multi-pass plan loop.
---
# Optimize Plan Orchestrator

Stabilize a Markdown implementation plan by running repeated `$optimize-plan` passes and tracking convergence.

Use this skill when the user asks to optimize, harden, stabilize, or repeatedly refine a plan until it is clean. For a single audit-and-edit pass, use `$optimize-plan`. For audit-only review, use `$colab-audit-plan`.

## Modes

Support exactly one mode per invocation:

- **In-place mode:** one Markdown plan path. Optimize that file in place through repeated passes.
- **Output-plan mode:** input Markdown path plus output Markdown path. Each pass reads the input/current output state and writes the complete refined plan only to the output path.

If the paths or mode are ambiguous, ask one direct question before starting.

## Subagent Policy

Prefer one fresh `optimize-plan` custom agent or subagent per pass when the user explicitly authorizes subagents, parallel agents, or fresh-context agents. Fresh means blind independent review of the current plan file, not merely a new process.

If subagents are not explicitly authorized:

1. Ask whether to use fresh subagents, or
2. Run a clearly labeled single-agent fallback only if the user requested no subagents or accepts the fallback.

Do not hide the fallback. It is lower isolation than the opencode command harness.

## Independence Boundary

The parent orchestrator may track pass history, clean streak, flip-flop state, pass count, and final reporting. Pass workers must not see that state.

A pass worker receives only:

- mode
- input plan path
- output plan path, when applicable
- the instruction to perform one blind independent `$optimize-plan` pass on the current plan contents

A pass worker must not receive prior pass summaries, prior findings, prior changes, prior clean/dirty status, clean-pass streak, flip-flop state, or instructions to suppress previously reported issues.

## Orchestrator Rules

1. Parse mode and paths before launching any pass.
2. Do not edit files directly; only pass workers may edit the named plan/output file.
3. Launch or perform one `$optimize-plan` pass at a time.
4. Do not pass accumulated pass history, prior outcomes, clean streak, or flip-flop state to pass workers.
5. Track pass history privately in the orchestrator after each pass returns.
6. Run at most twenty passes.
7. Continue until at least three consecutive blind passes return `ZERO_CHANGES_REQUIRED: yes`.
8. Reset the clean-pass streak to zero whenever a pass returns `ZERO_CHANGES_REQUIRED: no` or `unclear`.
9. Stop on flip-flop only after the orchestrator privately compares a completed worker report with prior pass history. Do not ask workers to detect flip-flop.
10. Stop early only for missing input, denied required permission, unrecoverable blocker, user interruption, context contamination, or flip-flop.
11. Treat missing or malformed `ZERO_CHANGES_REQUIRED` as not clean unless the pass clearly says no changes were required and no edits were made.
12. Preserve every pass's `CHANGES_SUMMARY`, `CHAIN_SUMMARY`, `CHANGES_MADE`, and `REMAINING_CONCERNS` in orchestrator-private state. Do not request or expose hidden chain-of-thought.

## Pass Prompt Shape

Use this shape for each pass worker:

```text
Run one optimize-plan pass.

MODE: in-place | output-plan
INPUT_PLAN: <input path>
OUTPUT_PLAN: <same as input for in-place, or output path>

You are a blind independent reviewer. Review only the current plan file. Do not ask for, infer, or use prior pass summaries, prior findings, prior changes, clean-pass state, flip-flop state, or suppression instructions. If any prior-review context is accidentally included, ignore it and report `CONTEXT_CONTAMINATION` under `REMAINING_CONCERNS`.

Review only genuine issues that would concretely affect implementation correctness, repo-pattern alignment, verification quality, production behavior, or executable dependency order.

Pay particular attention to dependency ordering: every step must depend only on artifacts, decisions, credentials, or behavior created earlier in the plan or already present in the repository.

Return ZERO_CHANGES_REQUIRED: yes only if this pass found no verified required plan changes and made no edits. Return ZERO_CHANGES_REQUIRED: no if this pass made edits or found required changes.

Return CHANGES_SUMMARY with the concrete plan edits made in this pass, or none if no edits were made.

Return CHAIN_SUMMARY with a concise visible workflow summary: audit result, key evidence checked, decision made, and edit outcome. Do not include hidden reasoning or private chain-of-thought.
```

## Output

```text
PLAN:
- <input path>

OUTPUT:
- <same as input for in-place mode, or output path>

MODE:
- in-place | output-plan

SUBAGENT_MODE:
- fresh-subagents | single-agent-fallback

PASSES_RUN:
- <number>

FINAL_CLEAN_STREAK:
- <number>/3

STOP_REASON:
- stable-after-three-clean-passes | flip-flop-detected | context-contaminated | max-passes-reached | blocked | user-stopped

FLIP_FLOP_ITEM:
- <oscillating item, or "none">

PASS_SUMMARY:
- pass <N>: ZERO_CHANGES_REQUIRED: yes | no | unclear; CHANGES_SUMMARY: <summary or "none">; CHAIN_SUMMARY: <visible workflow summary>

REMAINING_CONCERNS:
- <unresolved uncertainty, or "none">
```
