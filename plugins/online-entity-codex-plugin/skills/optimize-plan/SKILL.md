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

Do not map Anthropic model names such as Sonnet or Opus. This workflow runs on Codex with GPT models.

## Orchestration

Prefer fresh-context subagents, one per pass, when the active runtime supports them and the user has authorized subagent use.

If the runtime requires explicit authorization for subagents and the user has not provided it, ask for permission before starting the pass loop. If subagents are unavailable or the user declines, offer a single-agent fallback and label it clearly as lower isolation than the original fresh-context workflow.

## Pass Log

Maintain a pass log in working memory:

```text
Pass 1: ISSUES_FOUND: [...] | CHANGES_MADE: [...]
Pass 2: ISSUES_FOUND: [...] | CHANGES_MADE: [...]
```

## Each Pass

Run exactly one thorough review-and-fix pass. For subagent mode, give the pass agent this task with the pass number, file paths, mode instructions, and pass log filled in:

```text
You are performing pass N of a plan optimization loop. Do one thorough review-and-fix pass on the plan document, then report back with a structured summary.

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
3. Identify genuine issues only. Do not invent problems or create make-work. Only flag something if it would cause failure, incorrect behavior, unclear implementation, or materially weaker verification.
4. Pay particular attention to dependency ordering. Verify that every step only relies on outputs from earlier steps. Flag any case where a step depends on something produced later.
5. Before flagging an issue, check the pass history. Do not re-raise anything already listed in a previous pass's CHANGES_MADE unless the change introduced a new concrete problem.
6. Fix every issue you identified according to the mode instructions above.
7. Return only the structured report below.

ISSUES_FOUND:
- <issue 1>
- <issue 2>
(or "none" if the plan is clean)

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

1. Converged: `ISSUES_FOUND` is `none` for three consecutive passes. Stop.
2. Flip-flop: any item in this pass's `ISSUES_FOUND` closely matches an item in any previous pass's `CHANGES_MADE`. Stop.
3. Safety limit: pass count has reached 12. Stop.
4. Otherwise: append to the pass log and run the next pass.

## Final Report

When stopping, report:

- Mode used: file loop or plan mode.
- Execution style: fresh-context subagents or single-agent fallback.
- Number of passes run.
- Stop reason: converged, flip-flop detected, safety limit reached, or user stopped.
- If flip-flop: quote the oscillating item.
- Output file path, or `edited in place: <file>`.
- Remaining concerns from the final pass, if any.
