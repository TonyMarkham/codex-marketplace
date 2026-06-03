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
- **Output-plan mode:** input Markdown path plus output Markdown path. Pass 1 reads the input path and writes the complete refined plan to the output path. Later passes read and write the output path as the current plan. Do not keep rereading the original input after the output file exists.

If the paths or mode are ambiguous, ask one direct question before starting.

## Subagent Policy

Explicit invocation of `$optimize-plan-orchestrator` authorizes the workflow's required fresh `optimize-plan` custom agents or subagents unless the current user turn asks for no subagents, fallback, single-agent mode, or otherwise restricts delegation.

Do not ask an extra authorization question merely because the prompt omitted wording such as "with subagents authorized."

Fresh means blind independent review of the current plan file, not merely a new process.

Ask before launching subagents only when the user's wording conflicts with the default fresh-subagent workflow, delegation availability is unclear, or the runtime cannot launch the required fresh agents.

If the user requests or accepts fallback, do not hide it. Single-agent fallback is lower isolation than the opencode command harness and must not be reported as blind independent convergence.

## Independence Boundary

The parent orchestrator may track pass history, clean streak, flip-flop state, pass count, and final reporting. Pass workers must not see that state.

A pass worker receives only static workflow instructions plus:

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
7. In output-plan mode, pass 1 uses `INPUT_PLAN: <input path>` and `OUTPUT_PLAN: <output path>`; later passes use `INPUT_PLAN: <output path>` and `OUTPUT_PLAN: <output path>`.
8. Continue until at least three consecutive blind passes return `ZERO_CHANGES_REQUIRED: yes`.
9. Reset the clean-pass streak to zero whenever a pass returns `ZERO_CHANGES_REQUIRED: no` or `unclear`.
10. Stop on flip-flop only after the orchestrator privately compares a completed worker report with prior pass history. Do not ask workers to detect flip-flop.
11. Stop early only for missing input, denied required permission, unrecoverable blocker, user interruption, nonrecoverable worker-prompt contamination, or flip-flop.
12. Treat missing or malformed `ZERO_CHANGES_REQUIRED` as not clean unless the pass clearly says no changes were required and no edits were made.
13. Preserve every valid pass's `CHANGES_SUMMARY`, `CHAIN_SUMMARY`, `CHANGES_MADE`, and `REMAINING_CONCERNS` in orchestrator-private state. Do not request or expose hidden chain-of-thought.
14. Track invalid attempts separately from valid passes. Invalid attempts include contaminated worker prompts, denied required permissions before review, malformed worker outputs that cannot be interpreted, or worker runs that did not inspect the target plan.

## Context Contamination Adjudication

Do not treat every `CONTEXT_CONTAMINATION` mention as an automatic stop. First classify what happened:

- **Worker-prompt contamination:** prior pass history, prior findings, prior changes, clean streak, flip-flop state, or suppression instructions were included in the worker prompt or parent-provided instructions outside the plan file. This invalidates the pass. Do not count it as clean. Rerun once with a clean blind prompt if possible. Stop only if the contamination cannot be removed or repeats.
- **Plan-contained process text:** the current plan file itself contains status notes, audit notes, provenance, or text such as "repository audit found...". This is plan content, not parent-context contamination. If the worker ignored it as authority and verified against current repo evidence, the pass remains valid. If the text is stale, misleading, or likely to bias future implementation/review, treat it as a plan-content issue to report or patch; do not hard-stop solely because it exists.
- **Ambiguous contamination or reliance:** if the worker relied on prior-review/process text as authority, or the report does not make clear whether contamination came from the prompt or from the plan file, mark the pass `unclear`, do not count it as clean, and rerun once with an explicit clean blind prompt. Stop only if ambiguity/reliance repeats or cannot be resolved.

Use an actionable stop reason that names the source. Do not use generic contamination wording without stating exactly what leaked and why rerun could not recover.

## Valid Pass And Attempt Accounting

- A **valid pass** is a worker run that inspected the current plan, returned an interpretable structured report, and was not invalidated by worker-prompt contamination or denied required permission before review.
- `PASSES_RUN` counts valid passes only.
- `ATTEMPTS_RUN` counts valid passes plus invalid attempts.
- Invalid attempts do not count as clean and do not advance convergence.
- Invalid attempts reset no state except their own retry allowance.
- The 20-pass safety limit applies to valid passes. Also stop if invalid attempts exceed three total or if the same invalid-attempt cause repeats after one clean rerun attempt.
- Record invalid attempts under `INVALID_ATTEMPTS`; do not hide them inside `PASS_SUMMARY`.
- Single-agent fallback passes are valid only for a fallback run, but they are not blind independent passes. Do not use `stable-after-three-clean-passes` for fallback convergence; use `single-agent-fallback-complete` or `single-agent-fallback-stopped`.

## Pass Prompt Shape

Use this shape for each pass worker:

```text
Run one optimize-plan pass.

MODE: in-place | output-plan
INPUT_PLAN: <input path>
OUTPUT_PLAN: <same as input for in-place, or output path>

You are a blind independent reviewer. Review only the current plan file. Do not ask for, infer, or use prior pass summaries, prior findings, prior changes, clean-pass state, flip-flop state, or suppression instructions. If prior-review context appears in this prompt outside the plan file, ignore it and report `CONTEXT_CONTAMINATION` under `REMAINING_CONCERNS`. If the plan file itself contains status notes, audit notes, provenance, or prior-review wording, treat that as plan content: do not rely on it as authority, verify against current repo evidence, and report or patch it only if it is stale, misleading, or materially likely to bias implementation/review.

Review only genuine issues that would concretely affect implementation correctness, repo-pattern alignment, verification quality, production behavior, or executable dependency order.

Pay particular attention to dependency ordering: every step must depend only on artifacts, decisions, credentials, or behavior created earlier in the plan or already present in the repository.

Do not write patch provenance, pass notes, audit summaries, status notes, change logs, reviewer comments, or "this pass changed..." explanations into the plan file. The plan file should contain only the implementation plan. Put patch/change reporting only in the structured worker report.

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

ATTEMPTS_RUN:
- <number>

FINAL_CLEAN_STREAK:
- <number>/3

STOP_REASON:
- stable-after-three-clean-passes | single-agent-fallback-complete | single-agent-fallback-stopped | flip-flop-detected | contaminated-worker-prompt | invalid-attempt-limit | max-passes-reached | blocked | user-stopped

CONTEXT_CONTAMINATION_ASSESSMENT:
- none | plan-contained-process-text-valid-pass | plan-contained-process-text-patched | worker-prompt-contaminated-rerun | contaminated-worker-prompt | ambiguous-rerun | unresolved-ambiguous

INVALID_ATTEMPTS:
- <attempt N>: <cause and recovery result>
- or "none"

FLIP_FLOP_ITEM:
- <oscillating item, or "none">

PASS_SUMMARY:
- pass <N>: ZERO_CHANGES_REQUIRED: yes | no | unclear; CHANGES_SUMMARY: <summary or "none">; CHAIN_SUMMARY: <visible workflow summary>

REMAINING_CONCERNS:
- <unresolved uncertainty, or "none">
```
