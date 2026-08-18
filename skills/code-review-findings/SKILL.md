---
name: code-review-findings
description: Produce severity-ranked code review findings for Vitemis repositories. Use when asked to review a diff, working tree, branch, implementation, bug fix, refactor, test change, or generated report, especially for Claude, Gemini, Cursor, or Codex review passes that must avoid unauthorized writes.
---

# Code Review Findings

## Workflow

1. Read `/Users/vita/Vitemis/AGENTS.md`.
2. Confirm role and write boundary. Claude, Gemini, and Cursor are review copilots: no source edits, no formatting, no codegen, no build outputs, and reports only in their own report directory.
3. Confirm repository boundary:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

4. Establish review scope from the user request. If no explicit scope is given, inspect the working tree and changed files with read-only commands such as:

```sh
git diff --stat
git diff --name-status
git diff
```

5. Review for behavioral defects first:
   - correctness regressions
   - data loss or persistence bugs
   - concurrency, lifecycle, async, or state bugs
   - security and privacy risks
   - compatibility, migration, and dependency risks
   - missing tests for changed behavior
6. Avoid low-value style findings unless they hide a real bug.
7. Validate each finding by tracing the affected path. Do not report speculative issues without a concrete failure mode.

## Findings Format

Lead with findings, ordered by severity:

```text
[P0] Title
File: /absolute/path/to/file:line
Risk: What can fail for a user or maintainer.
Evidence: The code path or diff detail that supports the finding.
Fix direction: The smallest practical correction.
```

Severity guide:

```text
P0: data loss, security exposure, crash-on-launch, or unusable core workflow
P1: high-confidence user-visible regression or broken important path
P2: contained bug, missing validation, or meaningful test gap
P3: maintainability risk that is likely to cause future defects
```

If no findings are found, say so explicitly and list any remaining test gaps or unreviewed scope.

## Reports

When writing a report, use the permitted directory for the current role and filename format `MM_DD_YY-HH_MM-xxxx.md`. The report must include `MODEL_CHECK_RESULT`, `PATH_CHECK_RESULT`, `FILES_WRITTEN`, `SUMMARY`, `VALIDATION_RESULT`, `UNCERTAINTIES`, and `NEXT_RECOMMENDED_ACTION`.
