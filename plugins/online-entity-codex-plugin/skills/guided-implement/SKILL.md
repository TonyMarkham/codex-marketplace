---
name: guided-implement
description: Guide a human through an approved implementation plan one small manual step at a time without editing files. Use when the user wants Codex to teach, prescribe commands or edits, and verify progress while the user performs the changes.
---
# Guided Implement

Guide the user through an approved plan one small manual step at a time. The user applies every change. Codex must not edit files while using this skill.

## When To Use

Use this skill when the user asks for manual implementation guidance from an approved plan, for example:

```text
Use $guided-implement on <plan-file>.
Use $guided-implement on <plan-file>, starting at step <n>.
Use $guided-implement on <plan-file>. Verify my last edit, then give me the next step.
```

Do not use this skill for autonomous implementation. If the user wants Codex to edit files directly, use a normal implementation workflow instead.

## Hard Stop Rules

- Do not write, modify, delete, move, format, patch, or stage files.
- Do not call `apply_patch` or any file-modifying tool.
- Do not run commands that create, delete, move, rewrite, format, scaffold, migrate, install, restore, or otherwise change files.
- Do not run package managers, generators, formatters, migrations, or `dotnet new`/`dotnet add`-style commands yourself.
- The user writes all code and runs all mutating commands manually.
- Present exactly one implementation step at a time.
- Stop after that step and wait for the user.
- If the user says no, stop, pause, wait, or challenges the workflow, stop immediately.

## Required Context Check

Before giving any implementation step:

1. Read the approved plan. If a plan file is named, read it from disk.
2. Read relevant repository instructions such as `AGENTS.md` when the step depends on repository conventions.
3. Determine the first uncompleted step in the approved plan. Do not skip earlier steps unless the user explicitly says they are complete.
4. Classify the next step as one of:
   - Command step
   - New file step
   - Existing file edit step
   - Verification step
   - Blocked or mismatched step
5. Use the output format for that step type.

Preserve the approved plan's implementation method and order. If the next plan step is a CLI step, give the user exact commands to run manually. Do not convert CLI steps into direct file edits. If the next plan step is a file edit, give exact manual editing instructions. Do not convert manual edits into CLI steps unless the plan or repository instructions require it.

## Reading Requirements

- Before proposing edits to an existing file, read that file from disk.
- Before proposing a CLI command that depends on a path, verify the relevant parent path or repository context when practical using read-only tools.
- If file contents do not match the plan or expected landmarks, stop and report the mismatch instead of inventing an edit location.

## Output Formats

Return only one of these formats. Do not add prefaces, future-step previews, or extra commentary.

### Command Step

````text
Target: <repository root or exact working directory>

Intent:
<one or two sentences>

Run Manually:
```bash
<exact command 1>
<exact command 2 if part of the same atomic plan step>
```

Why:
<short explanation tied to the approved plan and repository rules>

Stop after running this step.
````

### New File Step

````text
Target file: <path relative to repository root>

Intent:
<one or two sentences>

Create File With:
```<language>
<full file contents>
```

Why:
<short explanation tied to the approved plan and repository rules>

Stop after applying this step.
````

### Existing File Edit Step

Use only after reading the existing file from disk.

````text
Target file: <path relative to repository root>

Intent:
<one or two sentences>

Find:
```<language>
<exact existing content or landmark>
```

Replace With:
```<language>
<exact replacement content>
```

Why:
<short explanation tied to the approved plan and repository rules>

Stop after applying this step.
````

For insertions, use exact unmodified landmarks:

````text
Target file: <path relative to repository root>

Intent:
<one or two sentences>

Insert After:
```<language>
<exact existing content before insertion>
```

Insert This:
```<language>
<new content>
```

Insert Before:
```<language>
<exact existing content after insertion>
```

Why:
<short explanation tied to the approved plan and repository rules>

Stop after applying this step.
````

### Verification Step

Use when the next approved step is a read-only verification command or manual check for the user to run.

````text
Target: <repository root or exact working directory>

Intent:
<one or two sentences>

Run Manually:
```bash
<exact verification command>
```

Why:
<short explanation tied to the approved plan and repository rules>

Stop after running this step.
````

### Blocked Step

```text
Blocked: <short reason>

Expected:
<what the approved plan or repository rule requires>

Found:
<what was actually found>

Why:
<why continuing would risk an incorrect implementation step>

Stop here.
```

## Verification Mode

When the user says they applied a step and asks for verification:

1. Read the relevant file or run only read-only inspection commands.
2. Compare the result against the previous instruction and the plan.
3. Report whether the edit or command result landed as intended.
4. If it is wrong, present one manual correction and stop.
5. If it is correct, offer the next step only if the user asks for it.

Never silently fix the file.
