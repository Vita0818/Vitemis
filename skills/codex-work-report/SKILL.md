---
name: codex-work-report
description: Write a Codex work, implementation, documentation, or audit report under Vitemis conventions. Use when Codex has completed a task and needs to record model, path, files written, summary, validation, uncertainties, and next actions in codex-report/ with filename format MM_DD_YY-HH_MM-xxxx.md.
---

# Codex Work Report

## Workflow

1. Read `/Users/vita/Vitemis/docs/VITEMIS_AGENT_POLICY.md`.
2. Confirm the target repository root:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Write reports only to `codex-report/`.
4. Use report filename format:

```text
MM_DD_YY-HH_MM-xxxx.md
```

Example:

```text
06_30_26-21_45-docs-shim-update.md
```

5. Do not run Git mutation commands. If version-control action is needed, list manual commands for the user.

## Report Body

Use this structure:

```text
# <task title>

## MODEL_CHECK_RESULT
<current model name; unknown if unavailable. Unless the project entrypoint has a stricter model gate, do not stop because a version string differs.>

## PATH_CHECK_RESULT
<pwd, Git root, and whether they match the expected repository>

## FILES_WRITTEN
<new or modified files; include report path if writing a report>

## SUMMARY
<what changed or what was audited>

## VALIDATION_RESULT
<commands run and results; explicitly say if build/test was not run>

## UNCERTAINTIES
<unknowns, source/doc conflicts, missing evidence, unrun checks>

## NEXT_RECOMMENDED_ACTION
<suggested next step; do not auto-commit or push>
```

For implementation work, distinguish actual edits from existing dirty worktree state. For documentation-only work, state that build/test was not run unless it actually was.
