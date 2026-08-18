---
name: readonly-review-report
description: Create a read-only review report for Claude, Gemini, Cursor, or another review copilot under Vitemis conventions. Use when asked to perform or guide a non-Codex review, write to claude-report/ or gemini-report/, enforce no source edits, name reports as MM_DD_YY-HH_MM-xxxx.md, and structure the report with MODEL_CHECK_RESULT, findings, validation, and uncertainties.
---

# Read-only Review Report

## Rules

1. Read `/Users/vita/Vitemis/AGENTS.md`.
2. Do not modify source, tests, configs, build scripts, project files, resources, docs, templates, or generated metadata.
3. Do not run commands that intentionally mutate the working tree.
4. Do not execute Git mutation commands.
5. Write only the review report, and only to the reviewer-specific report directory.

Reviewer directory:

```text
Claude -> claude-report/
Gemini -> gemini-report/
Cursor -> cursor-report/
```

## Report Name

Use:

```text
MM_DD_YY-HH_MM-xxxx.md
```

`xxxx` should be a short lowercase ASCII task slug, for example:

```text
06_30_26-21_45-readonly-review.md
```

## Report Body

Use this structure:

```text
# <review title>

## MODEL_CHECK_RESULT
<model name, or unknown>

## PATH_CHECK_RESULT
<pwd, git root, whether they match>

## FINDINGS
<bugs, risks, regressions, missing tests, policy issues; findings first>

## FILES_WRITTEN
<the report path only>

## VALIDATION_RESULT
<read-only commands run; explicitly state if build/test was not run>

## UNCERTAINTIES
<unknowns, incomplete evidence, assumptions>

## NEXT_RECOMMENDED_ACTION
<manual next step or suggested Codex follow-up>
```

If there are no findings, say so clearly and list remaining test gaps or residual risk.
