---
name: codex-dev
description: Delegate bounded coding tasks to Codex (OpenAI). CC scopes the work, Codex writes the code, CC verifies the output. Cross-model review catches blindspots that same-model review misses. Use when user says "let codex write this", "codex dev", "delegate to codex", or when CC has scoped a self-contained implementation task with clear file boundaries.
---

# Codex Dev — CC orchestrate, Codex implement, CC verify

Delegate a bounded coding task to Codex (OpenAI). CC scopes the work, Codex writes the code, CC verifies the output. Cross-model review catches blindspots that same-model review misses.

## When to use

- User explicitly asks to delegate to Codex ("let codex write this", "codex dev")
- CC has already scoped a self-contained implementation task with clear file boundaries and clear success checks
- The task is implementation, not exploration or research

## When NOT to use

- Trivial changes (rename, typo, one-line fix) — do it directly
- Task is vague or unscoped — scope it first, then delegate
- Pure exploration/research — use Explore subagent
- Task requires heavy back-and-forth with user — discuss first

## Delegation criteria

Only delegate to Codex when ALL of these are true:
- Task is self-contained (clear start and end)
- You know which files to read and modify
- You can describe the expected outcome in one paragraph
- There is at least one verification command you can run after

If any of these are unclear, do more investigation yourself before delegating.

## Workflow

### Step 1: Scope

Read relevant files. Understand existing code. Then prepare:
- **Objective**: What to implement, one paragraph
- **Read files**: Files Codex should read for context
- **Write files**: Files Codex is allowed to modify or create
- **Interface contract**: If touching an API boundary — endpoint, method, request/response fields and types
- **Constraints**: Code style (Java: camelCase, Python: snake_case, Vue: `<script setup lang="ts">`), architecture rules from CLAUDE.md
- **Boundaries**: No unrelated refactoring. No new dependencies. No schema/config changes unless explicitly part of the task
- **Stop conditions**: If Codex needs files outside the write list, needs new dependencies, or needs to change an interface contract — stop and return, do not proceed

### Step 2: Delegate

```bash
codex exec -m $MODEL --sandbox workspace-write --full-auto --skip-git-repo-check "<prompt>"
```

Default model: gpt-5.4. User can override.

Do NOT suppress stderr. Errors are diagnostic information.

### Step 3: Verify

**Hard gates first (deterministic, scoped to changed files):**

For frontend changes:
- `npm run build` passes

For backend changes:
- `mvn compile` passes

For any changes:
- Run tests relevant to the changed files (not the full suite unless needed)
- No debug statements in changed files (console.log, System.out.println, print())
- No hardcoded credentials or secrets
- Changed files are within the allowed write list — if Codex wrote outside scope, flag it

Classify test failures:
- Pre-existing (failed before Codex) — note but do not block
- Introduced by Codex — must fix

If any hard gate fails, attempt one Codex fix via `codex exec resume --last`. If it fails again, stop and report to user.

**CC review second (LLM judgment on the diff):**
- Does the code do what was asked?
- Interface alignment: do frontend calls match backend definitions?
- Security: no injection, no unvalidated input at boundaries, no auth bypass
- Follows existing code patterns
- Codex weaknesses to watch: cross-file consistency, project-specific conventions, snake_case/camelCase at module boundaries

### Step 4: Report

```
## Codex-Dev Report
**Task**: [one-line summary]
**Model**: [model used]
**Files changed**: [list]
**Hard gates**: [all pass / which failed]
**Review**: [PASS or FAIL with issues]
**Next**: [ready to use / needs fix — Codex retry? manual?]
```

## Escalation rules

Codex must stop and return control if it:
- Needs to modify files outside the allowed write list
- Needs to add new dependencies
- Needs to change database schema or configuration
- Needs to alter an API contract that other code depends on
- Encounters ambiguity that requires a design decision

CC must stop and ask the user if:
- Hard gates fail after one retry
- Codex produced changes significantly different from what was scoped
- Security issue found in review

## Future: Parallel subtasks

Out of scope for V1. Revisit only when real usage shows single-Codex is insufficient for a recurring class of tasks, and only for frozen contracts with disjoint write surfaces and independently verifiable outputs.
