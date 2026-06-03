---
name: optimize-plan
description: Run one evidence-backed audit-and-edit pass on a Markdown implementation plan. Use when the user asks to optimize, harden, tighten, or make a plan more implementation-ready; use optimize-plan-orchestrator for repeated passes until stable.
---
# Optimize Plan

Run one audit-and-edit pass on a Markdown implementation plan.

Use `$optimize-plan` for a single pass. If the user asks to loop, stabilize, repeat until clean, or recreate the old multi-pass command workflow, hand off to `$optimize-plan-orchestrator`.

## Modes

Support exactly one mode per invocation:

- **In-place mode:** one Markdown plan path. Edit only that plan file.
- **Output-plan mode:** input Markdown path plus output Markdown path. Write the complete refined plan only to the output path.

If the target is missing, ambiguous, or not a Markdown plan, ask one direct question and do not edit.

## Rules

1. Read the input plan before changing anything.
2. Audit the plan against repository evidence before editing.
3. When invoked directly and subagents are explicitly authorized, launch an independent `audit-plan` custom agent or subagent first. When invoked by `$optimize-plan-orchestrator`, do not launch a nested `audit-plan` subagent; the `$optimize-plan` worker itself is the blind independent audit/edit pass. Otherwise, perform the audit yourself and label `AUDIT_USED` as `single-agent-fallback`.
4. Consider audit findings against verified repository evidence; do not treat speculative findings as required changes.
5. When invoked by `$optimize-plan-orchestrator`, act as a blind independent reviewer of the current plan file. Do not ask for, infer, or use prior pass summaries, prior findings, prior changes, clean streak, flip-flop state, or suppression instructions. Report `CONTEXT_CONTAMINATION` only when prior-review context appears in the prompt or parent-provided instructions outside the plan file. If the plan file itself contains status notes, audit notes, provenance, or prior-review wording, treat that as plan content: do not rely on it as authority, verify against current repo evidence, and report or patch it only if it is stale, misleading, or materially likely to bias implementation/review.
6. In in-place mode, edit only the named plan file.
7. In output-plan mode, write the complete refined plan only to the named output Markdown file.
8. Do not edit source code, tests, config, or unrelated documentation.
9. Make the smallest plan changes that resolve verified material issues.
10. Change only genuine implementation issues: incorrect APIs, wrong file paths, missing required steps, stale assumptions, internal contradictions, unsupported runtime assumptions, dependency-ordering defects, or other issues that would concretely cause implementation failure or incorrect behavior.
11. Pay particular attention to dependency ordering: every step must depend only on artifacts, decisions, credentials, or behavior created earlier in the plan or already present in the repository.
12. Preserve concrete implementation detail. Do not replace code blocks or file-specific instructions with broad prose unless the code is demonstrably wrong and the fix requires it.
13. Do not write patch provenance, pass notes, audit summaries, status notes, change logs, reviewer comments, or "this pass changed..." explanations into the plan file. The plan file should contain only the implementation plan. Put patch/change reporting only in `CHANGES_SUMMARY`, `CHANGES_MADE`, `CHAIN_SUMMARY`, and `REMAINING_CONCERNS`.
14. If the audit finds no concrete material issue, do not rewrite the plan for style alone.
15. Return `ZERO_CHANGES_REQUIRED: yes` only if this pass found no verified required plan changes and made no edits.
16. Return `ZERO_CHANGES_REQUIRED: no` if this pass made edits or found required changes.
17. Always return `CHANGES_SUMMARY` with the concrete plan edits made, or `none` if no edits were made.
18. Always return `CHAIN_SUMMARY` as a concise visible workflow summary: audit result, key evidence checked, decision made, and edit outcome. Do not include hidden reasoning or private chain-of-thought.

## Audit Threshold

Only treat a finding as required when it is evidence-backed and material to implementation. Required changes include:

- wrong file paths, type names, function signatures, commands, or package APIs
- missing implementation steps needed for later steps to work
- steps that depend on artifacts created later in the plan
- duplicate greenfield implementation of existing repo behavior without replacement justification
- invented helper APIs, abstractions, data models, or error flows that bypass established repo patterns
- verification steps that cannot exercise the material behavior being introduced

Do not make edits for wording polish, preferred style, speculative alternatives, or optional hardening.

## Output

```text
PLAN:
- <input path>

OUTPUT:
- <same as input for in-place mode, or output path>

MODE:
- in-place | output-plan

AUDIT_USED:
- independent-audit-agent | worker-self-audit | single-agent-fallback | no, with reason

ZERO_CHANGES_REQUIRED:
- yes | no

CHANGES_SUMMARY:
- <summary of concrete plan edits made, or "none">

CHAIN_SUMMARY:
- <concise visible workflow summary; no hidden reasoning>

CHANGES_MADE:
- <specific plan changes, or "none">

REMAINING_CONCERNS:
- <unresolved uncertainty, or "none">
```
