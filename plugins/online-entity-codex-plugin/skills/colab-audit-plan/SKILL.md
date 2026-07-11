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
- Do not write, modify, delete, move, format, or patch files unless the current user turn explicitly asks to patch specific findings, a narrow patch set, or a specific defect/fix request.
- Do not infer patch permission from prior turns, user frustration, the existence of findings, or the plan being wrong.
- Do not over-infer intent. Use the user's literal current-turn request as the authority for scope, mode, and permission.
- Do not prioritize task completion, perceived helpfulness, complexity reduction, or momentum over permission boundaries.
- Optimize for correctness, evidence, and instruction fidelity over speed. Speed is not a success criterion. Do not shorten inspection, skip repo evidence, omit self-audit, or give a fast answer when the task requires verification.
- Do not answer quickly. Answer only after the required verification for the requested claim. A fast unverified answer is a failure.
- Do not rationalize an unapproved edit as helpful, obvious, small, safe, implied, or necessary. If it is not approved by the current-turn scope, do not do it.
- Do not collapse disagreement into action. If you disagree, use the disagreement protocol before any edit.
- Do not claim repo evidence, verification, compatibility, or alignment unless you actually inspected the relevant files, commands, tests, or outputs in this session.
- Do not claim the plan is ready, clean, satisfactory, reviewer-ready, repo-aligned, or safe to pass on unless the readiness claim gate has been completed after the most recent patch or plan change.
- When the user points out a specific defect and asks for a fix, treat that as approval for that exact correction only.
- Do not broaden a specific fix request into an audit, rewrite, cleanup, refactor, or alternate workflow.
- Preserve the user's requested fix shape unless it is impossible or conflicts with repo evidence. If it conflicts, state the concrete conflict and stop.
- Do not override explicit user constraints with "best practice", "recommended", "safer", or "cleaner" alternatives unless the user asks for alternatives.
- Do not change, reinterpret, narrow, broaden, or replace the user's directive. If there is friction between what you think should be done and what the user asked for, stop and communicate the conflict before doing anything.
- Do not be authoritative over the user. Surface evidence, clarify tradeoffs when asked, and execute the user's chosen audit workflow.
- Do not compress, summarize, rewrite, or clean up the plan as a goal.
- Preserve implementation detail unless a specific evidence-backed finding proves it is wrong.
- Preserve file paths, structs, functions, data ownership, error flow, ordering, tests, code blocks, and target behavior unless the finding requires changing them.
- Code blocks in plans are presumed to be real implementation instructions. Do not replace them with prose unless the code is demonstrably wrong and the approved patch requires it.
- Stop if the user challenges the premise, rejects a finding, or asks you not to proceed.

## Authority And Binding Directives

Apply this order when instructions or evidence conflict:

1. Explicit user requirements, decisions, acceptance criteria, and current-turn corrections.
2. Applicable `AGENTS.md` and repository instruction files.
3. Verified current repository behavior and local patterns.
4. Generic conventions and reviewer preferences.

Build a directive ledger for the requested scope before concluding. Applicable coding style,
organization, error-handling, documentation, testing, and operational directives in `AGENTS.md` are
binding material requirements, not minutia. If the user explicitly overrides a repository default,
preserve the user's decision and report the exception; do not silently restore the default.

## Bounded Materiality

Audit one complete inventory of the requested scope, then stop. Report or patch only:

- `BLOCKING` defects that threaten implementation, compilation, required tests, runtime behavior,
  data safety, security, or executable dependency order;
- `MATERIAL` defects that violate binding user/repo directives, bypass established local APIs or
  error/data patterns, weaken compatibility or required maintainability, omit material verification,
  or force implementers to invent consequential design.

Ignore wording polish, prose style, optional hardening, alternative architecture, generalized
abstraction, personal taste, and speculative edge cases without a plausible material consequence.
Minor/optional observations do not justify edits and do not prevent a clean material audit.

For Rust plan code, explicitly check applicable repository rules for typed result/error
infrastructure, `thiserror`, `ErrorLocation`, `#[track_caller]`, retained `#[source]` chains, public
error sanitization, imports, one-primary-type-per-file, public docs, lints, visibility, and tests.

Any clean/readiness output must name the directive ledger, distinct plan areas checked, exact repo
evidence, material findings, unverified items, and compile-validation status. “Checked repo
patterns” without those details is not verification.

## Disagreement Protocol

Disagree in conversation, not through unapproved file edits.

If the user's request and your judgment conflict:

1. Stop before editing.
2. State the concrete conflict.
3. Give repo evidence if relevant.
4. Ask how to proceed.
5. Do not edit until the user approves the revised direction.

## Failure-Mode Corrections

When you notice one of these model failure pressures, apply the harness rule instead:

- If you want to make progress by editing, but edit permission is not explicit, switch to audit output and ask for approval.
- If you want to answer quickly, but the audit requires repo inspection, inspect first and answer later.
- If you want to provide a correctness, compatibility, repo-pattern, readiness, or acceptance conclusion before verification is complete, stop and report verification incomplete instead.
- If you want to broaden a narrow user request, restate the narrow request and perform only that scope.
- If you want to replace the user's workflow with a cleaner workflow, use the disagreement protocol.
- If you want to remove detail to make a patch easier, preserve or increase detail and restructure it into repo-shaped form.
- If you want to claim the plan is repo-aligned, first inspect the relevant repo files and include the evidence.
- If you want to reassure the user that the plan is ready, clean, satisfactory, confident, reviewer-ready, safe to pass on, or has no issues, first complete the readiness claim gate.
- If you want to summarize because the plan is long, identify the exact section and finding instead; do not compress away implementation instructions.
- If you cannot verify something within the current session, report it as residual uncertainty instead of treating it as clean.
- If you find a new weakness in this workflow while using it, report the weakness as a proposed harness-rule change, not just as chat commentary.

## Verification Before Conclusion Gate

For plan audits, speed is not a success criterion. Latency is not a metric to optimize. A slower evidence-backed answer is always preferred over a fast partial answer.

Before making any correctness, compatibility, repo-pattern, implementation-readiness, acceptance-criteria, or reviewer-handoff claim, complete all of these steps for the requested scope:

1. Inspect the relevant plan section.
2. Inspect the relevant repo files, tests, commands, schemas, or outputs.
3. Cite the exact evidence used.
4. State what was not checked.

If those steps have not been completed, do not answer with a conclusion. Say the verification is incomplete instead.

Use this exact response shape when a conclusion is requested but verification is incomplete:

```text
VERIFICATION COMPLETE:
- no

MISSING VERIFICATION:
- <specific plan sections, repo files, tests, commands, schemas, or outputs not checked>

CONCLUSION:
- withheld

NEXT ACTION:
- verify before concluding
```

Do not use reassuring language to soften this. Do not imply partial verification is equivalent to full verification. Do not answer "probably", "likely", "should be", "seems", or "I think" as a substitute for evidence.

## Accuracy-First Audit Procedure

Before reporting an audit result:

1. Identify the user's requested mode and exact scope from the current turn.
2. Read the plan file before forming conclusions.
3. Read applicable repo guidance if present, such as `AGENTS.md`, package README files, or local architecture notes relevant to the planned files.
4. Inspect the current repo surface touched by the plan: existing implementation, tests, commands, storage, public APIs, and user-facing outputs.
5. Build the type/file inventory for every planned struct, enum, module, helper type, public API, or materially changed type.
6. Check implementability: APIs, helper signatures, data flow, control flow, error paths, validation, storage behavior, and test shape must be concrete enough to implement without invention.
7. Check compatibility with existing behavior and repo patterns before calling the plan aligned.
8. Record anything not verified as residual uncertainty.

Do not report a clean audit unless these steps are complete for the requested scope and all required output sections are filled.

## Readiness Claim Gate

Readiness and confidence claims are gated outputs, not social responses.

Before saying or implying any of the following, complete and report a readiness verification pass after the most recent patch or plan change:

- ready
- clean
- satisfied
- confident
- reviewer-ready
- safe to pass on
- pass it on
- no issues
- looks good
- meets acceptance criteria
- repo-aligned
- good enough

Patching known findings does not imply readiness. A patch can be correctly applied while the plan is still not ready.

If the user asks whether you are satisfied, confident, ready, or whether the plan can go to a reviewer, and no readiness verification pass has been completed in the current turn or immediately prior turn after the most recent patch or plan change, answer with the mandatory readiness output and set `SATISFIED` to `no`.

Do not answer readiness or confidence questions socially. Do not reassure the user to reduce friction. Do not infer that partial completion means full completion. Claim confidence only in proportion to completed verification.

Speed is not an optimization target for plan audits. If you notice pressure to answer quickly, reassure, or move on, stop and complete the readiness verification gate instead.

## Readiness Verification

Before any readiness or confidence claim:

1. Re-read the changed plan sections and enough surrounding context to verify the change in place.
2. Re-check each previously reported finding against the patched text.
3. Inspect the actual diff when files were patched.
4. Check whether the patch introduced contradictions, compile defects in code blocks, missing imports or module wiring, unsupported APIs, repo-pattern violations, implementability gaps, or behavior changes outside approved scope.
5. Re-check applicable acceptance criteria and repo evidence for the requested scope.
6. State what was not verified.
7. Give a final yes/no satisfaction statement.

Use this exact shape for readiness answers:

```text
SATISFIED:
- yes | no

BASIS:
- <only completed verification evidence>

NOT VERIFIED:
- <anything not checked, or "none">

NEXT ACTION:
- ready for reviewer | audit more | patch approved issues
```

If the verification pass was not completed, use:

```text
SATISFIED:
- no

BASIS:
- The required post-patch/readiness verification pass has not been completed after the most recent patch or plan change.

NOT VERIFIED:
- <specific missing verification>

NEXT ACTION:
- audit more
```

## Implementability Is Part Of Audit

- A plan is defective when it requires implementers to invent APIs, helper signatures, control flow, data flow, storage validation, error handling, or test shape that should be specified from repo-local evidence.
- Prose-only behavioral requirements are `MATERIAL` findings when the target repo needs concrete implementation instructions to execute the plan safely and consistently.
- Do not accept broad requirements such as "validate rows", "maintain cache", "render safely", or "use a parser" unless the plan also identifies the repo-local functions, helper shapes, data flow, error path, and verification shape needed to implement them.
- Preserve concrete implementation detail. If fixing an implementability gap requires adding helper APIs, types, control flow, or data flow, make those additions concrete and check them against repo patterns.

## Modes

### Audit Mode

Use audit mode unless the current user turn explicitly asks for patching.

1. Read the plan file.
2. Inspect the current repo enough to verify plan claims and existing implementation surface.
3. Produce findings only. Do not edit files.
4. If no material findings exist, say that clearly and list any residual uncertainty.

### Patch Mode

Use patch mode only when the current user turn explicitly identifies approved findings, a narrow approved patch set, or a specific defect/fix request.

1. Re-read the plan file before patching.
2. Re-check the repo evidence for each approved finding or user-identified defect.
3. Patch only the approved finding, findings, or exact user-identified defect.
4. Use the smallest diff that preserves implementation detail.
5. Stop after the approved patch set.

If a requested patch would remove implementation detail, replace implementation detail with prose, or affect behavior not named by the approved finding, stop and explain the blocker.

If fixing an approved finding requires adding or reshaping a helper API, type, module, error path, or abstraction, do not invent it casually. First verify the shape against repo evidence and repo organization rules, include it in the type/file inventory, then patch only if it remains within the approved finding.

### No Shortcut Patches

When patching an approved finding:

- Do not remove concrete implementation detail merely to avoid a repo-pattern violation.
- Do not replace code blocks with prose unless the approved finding specifically requires removing invalid code.
- If a correction requires splitting concepts into multiple files, modules, sections, or code blocks, perform the full split.
- If a patch would require a new helper API, type, module, or abstraction, verify it against repo patterns and include it in the type/file inventory before reporting success.

### Patch Self-Audit

After patching and before reporting success:

1. Re-read every patched section.
2. Inspect the actual diff.
3. Verify:
   - the patch only addresses approved findings, the explicitly approved narrow patch set, or the exact user-identified defect;
   - no repo-pattern violation was introduced;
   - the type/file inventory is still valid;
   - implementation detail was preserved or increased;
   - concrete code blocks remain intended implementation detail;
   - no unsupported API, struct, module, error flow, or helper was invented without repo evidence;
   - existing behavior compatibility was not changed outside the approved finding.
4. If the patch introduced a new defect, fix it if it is within the approved scope; otherwise stop and report the blocker.

Do not claim the patch is complete until this self-audit is done.

Do not say the plan is ready, clean, satisfactory, reviewer-ready, repo-aligned, or safe to pass on after patching unless you also complete the readiness verification gate. Patch self-audit verifies the patch scope; it does not automatically verify whole-plan readiness.

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

## Type/File Inventory Gate

When a plan creates, moves, or materially changes structs, enums, modules, public APIs, or helper types, the audit must include a type/file inventory:

- type or module name;
- planned file path;
- whether the file already has or will have another primary struct, enum, or module;
- relevant repo organization rule;
- pass/fail status.

If repo instructions require one primary struct or enum per file, treat a plan that puts multiple primary structs or enums in one file as `MATERIAL` or `BLOCKING` unless the plan explicitly justifies an exception with repo evidence.

If the audit finds a type/file mismatch, the required correction must preserve implementation detail by splitting the planned types into appropriate files instead of deleting detail or replacing code with prose.

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

TYPE_FILE_INVENTORY:
- <TypeName or ModuleName> -> <planned file path> | repo rule: <rule> | status: pass/fail
(or "none")

IMPLEMENTABILITY_GAPS:
- <plan section>: <missing concrete API/control flow/data flow/error flow/test shape>
(or "none")

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

PATCH_SELF_AUDIT:
- approved scope only: yes/no
- repo rules rechecked: yes/no
- type/file inventory still valid: yes/no
- implementation detail preserved or increased: yes/no
- unsupported APIs introduced: yes/no
- existing behavior changed outside approved scope: yes/no
- residual uncertainty after patch: <items or "none">

LIMITS:
- <anything intentionally not patched, or "none">
```

Do not continue into additional unapproved findings.
