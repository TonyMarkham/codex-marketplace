---
name: guided-implement
description: Guide a human through implementing an approved plan one small manual coding step at a time. Use when the user wants Codex to teach, explain, and present code without editing files.
---

# Guided Implement

Turn an approved implementation plan into one small human-executed coding step at a time.

This skill is for teaching and manual implementation. Codex must not edit files while using this skill.

Invoke it as:

```text
Use $guided-implement on <plan-file>.
```

Optional forms:

```text
Use $guided-implement on <plan-file>, starting at step <n>.
Use $guided-implement on <plan-file>. Verify my last edit, then give me the next step.
Use $guided-implement on <plan-file>. Focus on the parser section first.
```

## Hard Rules

- Do not write, modify, delete, move, or format files.
- Do not use `apply_patch`.
- Do not run formatters, fixers, generators, migrations, or test commands that modify files.
- The user writes all code manually.
- Present exactly one implementation step at a time unless the user explicitly asks for a larger batch.
- Stop after presenting the step. Wait for the user to apply it, ask for clarification, request verification, or ask for the next step.
- Before proposing edits to any existing file, read that file from disk.
- When presenting code, include the target path relative to the repository root.
- Prefer a `Find` and `Replace` strategy for edits to existing files.
- For insertions, provide exact unmodified `Insert After`, `Insert This`, and `Insert Before` landmarks.
- For new files, provide the full file contents.

## Workflow

1. Parse the user's request and identify:
   - the plan file
   - requested starting point, if any
   - whether the user wants verification of a previous manual edit
   - any scope limit such as "parser only" or "next DB step"
2. Read the plan file.
3. Identify the next smallest coherent implementation step.
4. Read every existing target file needed for that step.
5. Explain the step's purpose and why it comes next.
6. Present the manual edit instructions.
7. Stop.

If the plan is ambiguous or the target file contents do not match the plan, ask one focused question or describe the mismatch. Do not fabricate landmarks.

## Step Size

Prefer steps that are small, reviewable, and preserve cognitive focus:

- one new file
- one type or data model
- one function signature plus its direct call-site fallout
- one migration file
- one parser behavior
- one focused test case or test group

Do not combine broad cross-cutting work into one step unless splitting it would leave the user with incoherent code or misleading guidance.

## Existing File Edits

For replacements, use this format:

````text
Target file: path/from/repo/root.ext

Intent:
<one or two sentences>

Find:
```language
<exact existing text>
```

Replace With:
```language
<replacement text>
```

Why:
<short explanation>
````

For insertions, use this format:

````text
Target file: path/from/repo/root.ext

Intent:
<one or two sentences>

Insert After:
```language
<exact existing text immediately before insertion>
```

Insert This:
```language
<new code>
```

Insert Before:
```language
<exact existing text immediately after insertion>
```

Why:
<short explanation>
````

For deletions, use this format:

````text
Target file: path/from/repo/root.ext

Intent:
<one or two sentences>

Delete This:
```language
<exact existing text>
```

Why:
<short explanation>
````

Only use deletion steps when the plan requires removal or when the user asks for cleanup.

## New Files

For new files, use this format:

````text
Target file: path/from/repo/root.ext

Intent:
<one or two sentences>

Create File With:
```language
<full file contents>
```

Why:
<short explanation>
````

## Verification Mode

When the user says they applied a step and asks for verification:

1. Read the relevant file or files.
2. Compare them against the previous instruction and the plan.
3. Report whether the edit landed as intended.
4. If the edit is wrong, explain the smallest correction using the same manual edit format.
5. If the edit is correct, offer the next step only if the user asks for it or the prior instruction explicitly asked you to continue after verification.

Do not silently fix the file.

## Command Use

Use read-only commands when they help understand current context:

- file reads
- `rg` searches
- `git status --short`
- `git diff -- <path>` for verification
- language-aware read-only inspection commands

Avoid commands that write, format, build generated files, run migrations, or alter caches unless the user explicitly asks for that kind of verification and the command is known to be non-destructive in the current repo.

## Teaching Requirements

Each step should include:

- the target file path
- the intent of the step
- the exact manual edit
- a short explanation of important invariants or consequences
- where the user should stop

Keep the explanation scoped to the current step. Do not re-explain the entire plan unless the user asks.

## Final Behavior

Never say the implementation is complete unless the plan has no remaining steps or the user asks for a status summary and the inspected files support that conclusion.
