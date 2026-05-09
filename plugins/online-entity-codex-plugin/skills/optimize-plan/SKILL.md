---
name: optimize-plan
description: Optimize a plan document through iterative review/edit passes until stable. Use when the user asks to optimize, harden, pressure-test, or repeatedly refine a plan file, especially when they expect the old /optimize-plan workflow from Claude Code.
---

# Optimize Plan

Optimize a plan document through iterative review/edit passes until stable.

This is the Codex-supported version of the Claude Code `/optimize-plan` workflow. Invoke it as a skill:

```text
Use $optimize-plan on <file>
Use $optimize-plan on <input_file> and write the result to <output_file>
```

## Modes

Parse the user's request before doing anything else.

- File loop mode: one file path is provided. Review and edit that file in place each pass.
- Plan mode: input and output paths are provided. Read the input file and write the complete refined plan to the output file each pass.
- Model/reasoning preferences: if the user asks for a specific GPT model or reasoning effort, use those preferences only where the active Codex runtime supports them.
- Distinguish orchestrator and reviewer preferences when the user provides them. The orchestrator controls the loop; reviewer subagents perform pass audits.

Do not map Anthropic model names such as Sonnet or Opus. This workflow runs on Codex with GPT models.

Recommended efficient defaults when the user does not specify models:

- Orchestrator: current session model and reasoning effort.
- Reviewer subagents: `gpt-5.4-mini` with `high` reasoning when the runtime supports model selection for subagents.
- High-risk escalation: use `gpt-5.5` with `high` reasoning for the first pass or final adjudication only when the user asks for a higher-confidence review.
- Avoid `gpt-5.4-nano` for this workflow unless the user explicitly asks for the cheapest smoke review.

## Orchestration

Preferred role contracts:

- Orchestrator: `base-instructions/optimize-plan-orchestrator.md`
- Review subagent: `base-instructions/plan-review-subagent.md`

Use those base instructions when the active Codex runtime/profile setup supports applying them. If it does not, continue with this skill's inline instructions; do not stop just because role-specific base instructions are not automatically applied.

Prefer fresh-context subagents, one per pass, when the active runtime supports them and the user has authorized subagent use.

If the runtime requires explicit authorization for subagents and the user has not provided it, ask for permission before starting the pass loop. If subagents are unavailable or the user declines, offer a single-agent fallback and label it clearly as lower isolation than the original fresh-context workflow.

Reviewer edit gating:

- Default to reviewer-side pre-edit gating. Before editing the plan, the reviewer must classify each proposed change as `BLOCKING`, `MATERIAL`, `MINOR`, or `OPTIONAL`.
- Reviewers may apply only `BLOCKING` and `MATERIAL` changes.
- Reviewers may apply a `MINOR` change only when it is directly adjacent to an applied `BLOCKING` or `MATERIAL` change and necessary to make that fix coherent.
- Reviewers must skip `OPTIONAL` changes.
- If the active runtime clearly supports keeping the same reviewer thread alive for a two-phase pass, the orchestrator may ask the reviewer to report proposed changes first, adjudicate them, then send the same reviewer back to apply only approved `BLOCKING` and `MATERIAL` changes. Do not require this path unless the runtime can actually support it.
- If the runtime does not clearly support that two-phase continuation, keep the reviewer-side pre-edit gate as the robust default.

## Pass Log

Maintain a pass log in working memory:

```text
Review A: ORCHESTRATOR_CLASSIFICATION: [...] | MATERIAL_ISSUES: [...] | MINOR_NOTES: [...] | CHANGES_MADE: [...] | SKIPPED_CHANGES: [...]
Review B: ORCHESTRATOR_CLASSIFICATION: [...] | MATERIAL_ISSUES: [...] | MINOR_NOTES: [...] | CHANGES_MADE: [...] | SKIPPED_CHANGES: [...]
```

Maintain a per-run permission registry:

```text
APPROVED_PERMISSION_PATTERNS:
- <runtime family>: <command prefix or exact pattern approved by the user>

PERMISSION_EVENTS:
- Review requested: <command or action> | runtime: <windows-powershell | posix-shell | other> | user response: approved/denied | reusable pattern: <pattern or none>
```

The registry is advisory context for later reviewers. It does not bypass Codex runtime approvals. If the runtime still asks for approval, comply with the runtime prompt and continue the same pass after approval.

Keep permission patterns ambidextrous:

- Detect the active runtime family from paths and shell behavior: Windows PowerShell, POSIX shell, or other.
- Use native read-only commands for that runtime instead of forcing Windows commands on Linux/macOS or POSIX commands on Windows.
- Keep equivalent command shapes stable within a run so saved approval prefix rules can match later passes.
- Track approved patterns by runtime family. Windows and WSL/Linux usually have separate Codex homes and separate rules files.
- Prefer built-in file/search tools when available. If shell commands are needed, prefer stable read-only forms.
- Never request broad arbitrary-shell approval and never preapprove destructive commands.

## Baseline State

Before spawning the first reviewer, record the target plan file state:

- target path
- whether the file exists
- best available VCS status for the target file, if the working directory is under version control
- best available diff stat or content fingerprint for the target file

Use native read-only commands for the active runtime family. If VCS status is unavailable, record `unknown` and continue.

Carry this baseline in the pass log. In the final report, compare the final target state to the baseline:

- If the final state differs from the baseline and one or more reviewers reported `CHANGES_MADE`, report the file changed during optimization.
- If the final state differs from the baseline but every reviewer reported `CHANGES_MADE: none`, flag a reporting mismatch instead of guessing that the diff is pre-existing.
- If the file was already modified at baseline and remains modified at the end with no reviewer changes, report that the file had pre-existing modifications.

## Each Reviewer Pass

Run exactly one thorough gated review-and-fix pass. For subagent mode, give the reviewer only neutral workflow context: file paths, mode instructions, target baseline, prior reviewer outcomes, and permission patterns. Do not tell the reviewer its ordinal pass number or whether it is early or late in the loop.

```text
You are performing an independent review in a plan optimization loop. Do one thorough gated review-and-fix pass on the plan document, then report back with a structured summary.

Model preference:
Use the reviewer model and reasoning effort selected by the orchestrator for this review. If no reviewer model was selected, prefer gpt-5.4-mini with high reasoning when available. If the runtime cannot set a subagent model or reasoning effort, continue with the runtime default and state that in the review summary.

Role contract:
Act as a fresh-context production plan reviewer. If a plan-review-subagent base instruction is available in this runtime, follow it. Otherwise, follow the review threshold and structured output rules in this prompt.

Mode: FILE_LOOP or PLAN_MODE

FILE_LOOP:
Read <file>, audit it, and edit it in place only for proposed changes that pass the pre-edit gate.

PLAN_MODE:
Read <input_file>. Think holistically before making edits. When you have a complete picture of all proposed changes and their classifications, write the full revised plan to <output_file> with only changes that pass the pre-edit gate. Rewrite the complete document rather than patching isolated fragments.

Prior reviewer outcomes:
<paste relevant prior reviewer outcomes without ordinal pass labels, or "No prior reviewer outcomes." if none>

Target file baseline:
<paste baseline target file status, diff stat or fingerprint, and whether it was already modified before the first reviewer>

Preapproved permission patterns for this run:
<paste APPROVED_PERMISSION_PATTERNS, or "none">

Command stability:
Use the active runtime's native read-only command family consistently. On Windows PowerShell, prefer stable forms such as `Get-Content -Raw`, `Select-String -Path ... -Pattern ... -Context ...`, and `Get-ChildItem ...`. On Linux/macOS POSIX shells, prefer stable forms such as `sed -n ...`, `rg -n ...`, and `find ... -maxdepth ... -type f -print`. Do not mix equivalent command shapes without a material reason.

Review instructions:
1. Read the plan file thoroughly before forming a conclusion.
2. Audit every concrete claim against the actual codebase. Use file reads, rg/search, shell commands, and web browsing when needed to verify file paths, function signatures, type names, package APIs, dependency versions, and external SDK behavior referenced in the plan.
3. Identify genuine material issues only. A material issue is one that would affect implementation correctness, production behavior, verification quality, maintainability of the planned change, or executable dependency order.
4. Do not invent problems, create make-work, or turn polish into issues. Do not report wording polish, stylistic preferences, optional enhancements, alternative designs, or tiny clarifications as material issues.
5. Pay particular attention to dependency ordering. Verify that every step only relies on outputs from earlier steps. Flag any case where a step depends on something produced later.
6. Before flagging an issue, check the prior reviewer outcomes. Do not re-raise anything already listed in prior `CHANGES_MADE` unless the change introduced a new concrete material problem.
7. Before editing, create a pre-edit classification list. Classify each proposed change as `BLOCKING`, `MATERIAL`, `MINOR`, or `OPTIONAL`.
8. Apply only `BLOCKING` and `MATERIAL` changes according to the mode instructions above.
9. Apply a `MINOR` wording, formatting, documentation, or clarity change only when it is directly adjacent to an applied `BLOCKING` or `MATERIAL` fix and necessary to make that fix coherent.
10. Skip `OPTIONAL` changes. Do not edit just to polish, restyle, add preference-driven alternatives, or expand scope.
11. If all proposed changes are `MINOR` or `OPTIONAL`, do not edit the plan. Report them under `SKIPPED_CHANGES` and set `MATERIAL_ISSUES: none`.
12. You are authorized to read and edit the target plan file for this review. Do not ask semantic permission to inspect or edit that target file; only runtime approval prompts may require user interaction.
13. Prefer built-in file/search tools for target-file inspection. Use shell commands for repo verification when they materially help.
14. When using shell commands, use native stable read-only command shapes for the active runtime family. Keep equivalent command forms consistent with the preapproved patterns where possible.
15. Do not run probe commands solely to test whether approvals or shell execution work. Only request commands that materially help inspect the target plan or verify a concrete repo/doc/API claim.
16. If a command/action requires user approval, record it under `PERMISSIONS_REQUESTED`. If approved, include the runtime family and a reusable exact command or safe prefix pattern when one is obvious. Do not stop the pass merely because approval was required.
17. Return only the structured report below.

PRE_EDIT_CLASSIFICATION:
- BLOCKING: <proposed change and reason>
- MATERIAL: <proposed change and reason>
- MINOR: <proposed change and reason>
- OPTIONAL: <proposed change and reason>
(or "none")

MATERIAL_ISSUES:
- BLOCKING: <issue that would prevent implementation or cause incorrect production behavior>
- MATERIAL: <issue that materially weakens correctness, verification, maintainability, or execution order>
(or "none")

BLOCKS_IMPLEMENTATION:
yes | no

CONTINUE_REASON:
- <specific unfixed risk that justifies another pass>
(or "none")

MINOR_NOTES:
- <minor wording, formatting, or optional clarity note that does not affect implementation>
(or "none")

CHANGES_MADE:
- <change 1>
- <change 2>
(or "none")

SKIPPED_CHANGES:
- <minor or optional proposed change intentionally not applied, with classification>
(or "none")

PERMISSIONS_REQUESTED:
- command/action: <command or action>
  runtime: windows-powershell | posix-shell | other
  reason: <why it was needed>
  user_response: approved | denied | not requested
  reusable_pattern: <exact command or safe prefix pattern, or "none">
(or "none")

REMAINING_CONCERNS:
- <anything you could not verify or were uncertain about>
(or "none")
```

In single-agent fallback mode, apply the same pass instructions yourself. Keep the pass log visible in your reasoning and final summary.

## Orchestrator Adjudication

After each subagent returns, first forward the raw subagent report to the user unchanged. Put it under a short label such as `Raw reviewer report`.

Then make a separate orchestrator classification:

```text
ORCHESTRATOR_CLASSIFICATION:
BLOCKING | MATERIAL | MINOR_ONLY | CLEAN | FLIP_FLOP

ORCHESTRATOR_REASON:
<one sentence grounded in the raw report and prior reviewer outcomes>

CLEAN_STREAK:
<current count>/3

DECISION:
continue | stop
```

The orchestrator owns this classification. Do not continue because edits were made. Do not let a reviewer reset convergence merely by making edits.

Close the completed reviewer subagent after forwarding its raw report and recording the orchestrator classification. Do not keep completed reviewer subagents open until the end of the loop.

Classification rules:

- `BLOCKING`: the plan would likely fail implementation, compile/test verification, runtime behavior, data safety, security, or dependency ordering.
- `MATERIAL`: the plan would materially weaken correctness, verification quality, maintainability of the planned change, or executable dependency order.
- `MINOR_ONLY`: the pass made only wording, formatting, optional hardening, documentation, ordering, output-format polish, non-blocking test expansion, or clarity edits.
- `CLEAN`: no material issue and `CHANGES_MADE` is `none`. `MINOR_NOTES` and `SKIPPED_CHANGES` may be non-empty, but the plan file was not changed.
- `FLIP_FLOP`: this review reopens or reverses a prior review's change without a new concrete material reason.

Three consecutive `CLEAN` classifications stop the loop. `MINOR_ONLY` classifications do not count toward the no-change streak.

## Convergence

After forwarding the raw report and making the orchestrator classification, check these conditions in order:

1. Flip-flop: orchestrator classification is `FLIP_FLOP`. Stop.
2. Permission registry: add approved `PERMISSIONS_REQUESTED` entries to `APPROVED_PERMISSION_PATTERNS` when they include a reusable pattern. Pass the updated registry to the next reviewer.
3. Append the pass result to the pass log.
4. Converged: three consecutive `CLEAN` classifications reached. Stop.
5. Safety limit: reviewer-pass count has reached 20. Stop.
6. Otherwise: continue to the next pass.

Continuation rules:

- The clean streak increments only on `CLEAN`.
- The clean streak resets to 0 after `BLOCKING`, `MATERIAL`, `MINOR_ONLY`, or `FLIP_FLOP`.
- `MINOR_ONLY` never counts as a no-change pass.

No-change target:

- Three consecutive `CLEAN` orchestrator classifications.

Only three consecutive no-change passes justify convergence. `MINOR_NOTES`, skipped optional enhancements, and non-blocking hardening proposals do not count as changes unless the reviewer actually edits the plan.

## Final Report

When stopping, report:

- Mode used: file loop or plan mode.
- Execution style: fresh-context subagents or single-agent fallback.
- Number of passes run.
- Stop reason: converged, flip-flop detected, safety limit reached, or user stopped.
- If flip-flop: quote the oscillating item.
- Output file path, or `edited in place: <file>`.
- Baseline target file state and final target file state.
- Whether the target file changed during optimization, was already modified before optimization, or has a reporting mismatch that needs investigation.
- Remaining concerns from the final pass, if any.
