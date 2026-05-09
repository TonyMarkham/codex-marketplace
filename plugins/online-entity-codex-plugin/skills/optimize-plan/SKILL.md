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

## Pass Log

Maintain a pass log in working memory:

```text
Pass 1: MATERIAL_ISSUES: [...] | MINOR_NOTES: [...] | CHANGES_MADE: [...]
Pass 2: MATERIAL_ISSUES: [...] | MINOR_NOTES: [...] | CHANGES_MADE: [...]
```

## Each Pass

Run exactly one thorough review-and-fix pass. For subagent mode, give the pass agent this task with the pass number, file paths, mode instructions, and pass log filled in:

```text
You are performing pass N of a plan optimization loop. Do one thorough review-and-fix pass on the plan document, then report back with a structured summary.

Model preference:
Use the reviewer model and reasoning effort selected by the orchestrator for this pass. If no reviewer model was selected, prefer gpt-5.4-mini with high reasoning when available. If the runtime cannot set a subagent model or reasoning effort, continue with the runtime default and state that in the pass summary.

Role contract:
Act as a fresh-context production plan reviewer. If a plan-review-subagent base instruction is available in this runtime, follow it. Otherwise, follow the review threshold and structured output rules in this prompt.

Mode: FILE_LOOP or PLAN_MODE

FILE_LOOP:
Read <file>, audit it, and edit it in place to resolve all genuine issues found.

PLAN_MODE:
Read <input_file>. Think holistically before making edits. When you have a complete picture of all issues and their resolutions, write the full revised plan to <output_file>. Rewrite the complete document rather than patching isolated fragments.

Previous pass history:
<paste the full pass log here, or "This is pass 1. No prior history." if first pass>

Review instructions:
1. Read the plan file thoroughly before forming a conclusion.
2. Audit every concrete claim against the actual codebase. Use file reads, rg/search, shell commands, and web browsing when needed to verify file paths, function signatures, type names, package APIs, dependency versions, and external SDK behavior referenced in the plan.
3. Identify genuine material issues only. A material issue is one that would affect implementation correctness, production behavior, verification quality, maintainability of the planned change, or executable dependency order.
4. Do not invent problems, create make-work, or turn polish into issues. Do not report wording polish, stylistic preferences, optional enhancements, alternative designs, or tiny clarifications as material issues.
5. Pay particular attention to dependency ordering. Verify that every step only relies on outputs from earlier steps. Flag any case where a step depends on something produced later.
6. Before flagging an issue, check the pass history. Do not re-raise anything already listed in a previous pass's CHANGES_MADE unless the change introduced a new concrete material problem.
7. Fix blocking and material issues according to the mode instructions above. Apply minor wording or formatting edits only when they are adjacent to material fixes; do not edit just to polish.
8. Return only the structured report below.

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

REMAINING_CONCERNS:
- <anything you could not verify or were uncertain about>
(or "none")
```

In single-agent fallback mode, apply the same pass instructions yourself. Keep the pass log visible in your reasoning and final summary.

## Convergence

After each pass report, check these conditions in order:

1. Converged: clean-pass target reached. Stop.
2. Flip-flop: any item in this pass's `MATERIAL_ISSUES` closely matches an item in any previous pass's `CHANGES_MADE`. Stop.
3. Safety limit: pass count has reached 8. Stop.
4. Otherwise: append to the pass log and apply the continuation gate below.

Continuation gate:

- Passes 1-4: continue only when `MATERIAL_ISSUES` is not `none` and `CONTINUE_REASON` names a concrete unfixed risk.
- Passes 5-8: continue only when `BLOCKS_IMPLEMENTATION` is `yes`.
- Stop when `BLOCKS_IMPLEMENTATION` is `no` after pass 4, even if the pass made edits.
- Stop when changes are hardening, documentation, ordering, corruption validation, output formatting, test expansion, or clarity improvements unless the reviewer explains a concrete compile failure, test failure, data loss/corruption, incorrect runtime behavior, security exposure, or dependency-order blocker those edits prevented.

Clean-pass target:

- Before pass 5: three consecutive passes with `MATERIAL_ISSUES: none`.
- From pass 5 onward: one pass with `BLOCKS_IMPLEMENTATION: no`.

Only concrete implementation-blocking risks justify late-loop continuation. `MINOR_NOTES`, wording polish, optional enhancements, alternative designs, and non-blocking hardening do not reset convergence.

## Final Report

When stopping, report:

- Mode used: file loop or plan mode.
- Execution style: fresh-context subagents or single-agent fallback.
- Number of passes run.
- Stop reason: converged, flip-flop detected, safety limit reached, or user stopped.
- If flip-flop: quote the oscillating item.
- Output file path, or `edited in place: <file>`.
- Remaining concerns from the final pass, if any.
