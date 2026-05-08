---
description: Optimize a plan document through iterative fresh-context Codex review/edit passes until stable.
argument-hint: "<file> | <input_file> <output_file> [--model <model>] [--reasoning <effort>]"
---

# /optimize-plan

Optimize a plan document through iterative fresh-context review/edit passes until stable.

## Arguments

`$ARGUMENTS` is one of:

- `<file>`: file loop mode. Review and edit `<file>` in place each pass.
- `<input_file> <output_file>`: plan mode. Read `<input_file>` and write the complete refined plan to `<output_file>` each pass.
- `--model <model>`: optional. Preferred model for fresh-context review agents, for example `gpt-5.5`, `gpt-5.4`, or `gpt-5.4-mini`.
- `--reasoning <effort>`: optional. Preferred reasoning effort for review agents, for example `medium`, `high`, or `xhigh`.

Parse the arguments before doing anything else. Strip `--model <model>` and `--reasoning <effort>` from the file arguments after recording them.

Default to the current Codex model and reasoning effort when no override is provided. Do not map Anthropic model names such as Sonnet or Opus; this command is for Codex and GPT models.

## Your Role

You are the orchestrator. Do not perform the review or edit pass yourself unless subagents are unavailable in the active Codex runtime.

Use fresh-context Codex subagents, one per pass, and evaluate their structured reports to decide whether to continue. If subagents are unavailable, stop and explain that this command requires Codex subagent support rather than silently doing a single-context review.

## Pass Log

Maintain a pass log in working memory throughout:

```text
Pass 1: ISSUES_FOUND: [...] | CHANGES_MADE: [...]
Pass 2: ISSUES_FOUND: [...] | CHANGES_MADE: [...]
```

## Each Pass

Spawn one fresh-context Codex subagent with the selected model and reasoning effort when those controls are available. Ask it to run exactly one thorough review-and-fix pass with this prompt, substituting the pass number, file paths, mode instructions, and current pass log:

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

## Convergence Check

After receiving each pass report, check these conditions in order:

1. Converged: `ISSUES_FOUND` is `none` for three consecutive passes. Stop.
2. Flip-flop: any item in this pass's `ISSUES_FOUND` closely matches an item in any previous pass's `CHANGES_MADE`. Stop.
3. Safety limit: pass count has reached 12. Stop.
4. Otherwise: append to the pass log and spawn the next pass.

## Final Report

When stopping, tell the user:

- Mode used: file loop or plan mode.
- Number of passes run.
- Stop reason: converged, flip-flop detected, or safety limit reached.
- If flip-flop: quote the oscillating item.
- Output file path, or `edited in place: <file>`.
- `REMAINING_CONCERNS` from the final pass, if any.
