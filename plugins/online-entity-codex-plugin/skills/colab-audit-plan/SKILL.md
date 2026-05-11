---
name: colab-audit-plan
description: Audit implementation plans against the current repo without editing by default, then patch only explicitly approved findings. Use when the user wants collaborative plan review, repo-pattern checks, existing-behavior compatibility review, or controlled plan patches.
---

# Colab Audit Plan

Audit an implementation plan as a collaborator. Default to evidence-first review, not autonomous rewriting.

Invoke it as:

```text
Use $colab-audit-plan on <plan-file>. Do not edit.
Use $colab-audit-plan on <plan-file> and audit against repo patterns. Do not edit.
Use $colab-audit-plan to patch finding A1 only.
Use $colab-audit-plan to patch approved findings A1 and B2, then stop.
```

## Core Contract

- Default mode is audit-only.
- Do not write, modify, delete, move, format, or patch files unless the current user turn explicitly asks to patch specific findings.
- Do not infer patch permission from prior turns, user frustration, the existence of findings, or the plan being wrong.
- Do not compress, summarize, rewrite, or clean up the plan as a goal.
- Preserve implementation detail unless a specific evidence-backed finding proves it is wrong.
- Preserve file paths, structs, functions, data ownership, error flow, ordering, tests, code blocks, and target behavior unless the finding requires changing them.
- Code blocks in plans are presumed to be real implementation instructions. Do not replace them with prose unless the code is demonstrably wrong and the approved patch requires it.
- Stop if the user challenges the premise, rejects a finding, or asks you not to proceed.

## Modes

### Audit Mode

Use audit mode unless the current user turn explicitly asks for patching.

1. Read the plan file.
2. Inspect the current repo enough to verify plan claims and existing implementation surface.
3. Produce findings only. Do not edit files.
4. If no material findings exist, say that clearly and list any residual uncertainty.

### Patch Mode

Use patch mode only when the current user turn explicitly identifies approved findings or a narrow approved patch set.

1. Re-read the plan file before patching.
2. Re-check the repo evidence for each approved finding.
3. Patch only the approved finding or findings.
4. Use the smallest diff that preserves implementation detail.
5. Stop after the approved patch set.

If a requested patch would remove implementation detail, replace implementation detail with prose, invent a new abstraction, or affect behavior not named by the approved finding, stop and explain the blocker.

## Repo-Surface Audit

For each major planned feature or behavior, identify the existing repo surface before judging the plan:

- existing commands, modules, structs, functions, traits, tests, data flow, storage, public APIs, and user-facing behavior
- local patterns for module organization, visibility, parser/report shapes, error handling, naming, test layout, persistence, and output formatting
- whether the plan modifies, extends, replaces, or newly adds implementation

Treat these as material findings:

- the plan adds a parallel implementation for behavior that already exists
- the plan replaces existing behavior without explicitly saying so and justifying it with repo evidence
- the plan invents APIs, structs, modules, error types, helper functions, or abstractions where repo-local patterns already exist
- the plan classifies an intentional feature change as a repo-pattern violation
- the plan omits required compatibility work for existing behavior, tests, commands, storage, or user-facing output

If no existing implementation is found, state what searches or files support that conclusion.

## Finding Rules

Classify each finding by severity:

- `BLOCKING`: likely prevents implementation, compile/test verification, correct runtime behavior, data safety, or safe dependency order.
- `MATERIAL`: materially weakens correctness, compatibility, maintainability, verification quality, or repo-pattern alignment.
- `MINOR`: wording, local clarity, or small completeness issue that does not change implementation correctness.
- `OPTIONAL`: preference, alternative design, or enhancement that is not required.

Classify each finding by type:

- `repo pattern violation`
- `existing behavior compatibility risk`
- `actual plan defect`
- `intentional new behavior`
- `unverifiable assumption`

Do not list ordinary planned feature changes as repo-pattern violations. A new behavior is not a defect unless it conflicts with the repo, hides compatibility risk, lacks required verification, or contradicts the stated plan goal.

## Audit Output

Use this structure:

```text
PLAN:
- <path>

AUDIT_MODE:
- audit-only | patch-approved-findings

EXISTING_REPO_SURFACE:
- <files/functions/types/tests/data flow found, or "none found after searching ...">

PLAN_ALIGNMENT:
- <modifies existing implementation | extends existing implementation | replaces existing implementation | adds new implementation because none exists>: <evidence>

FINDINGS:

FINDING A1
- severity: BLOCKING | MATERIAL | MINOR | OPTIONAL
- type: repo pattern violation | existing behavior compatibility risk | actual plan defect | intentional new behavior | unverifiable assumption
- plan location: <section/header/line if available>
- repo evidence: <file/function/type/test/output evidence>
- why it matters: <implementation consequence>
- required correction: <smallest correction>
- patchable: yes | no

APPROVAL NEEDED:
- <finding IDs that should be patched, or "none">

RESIDUAL UNCERTAINTY:
- <anything not verified, or "none">
```

## Patch Output

After patching approved findings, report:

```text
PATCHED_FINDINGS:
- <finding id>: <what changed>

FILES_CHANGED:
- <path>

LIMITS:
- <anything intentionally not patched, or "none">
```

Do not continue into additional unapproved findings.
