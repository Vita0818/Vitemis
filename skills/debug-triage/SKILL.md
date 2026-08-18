---
name: debug-triage
description: Triage build, test, runtime, integration, or user-reported failures in Vitemis repositories before editing code. Use when diagnosing errors, flaky tests, crashes, regressions, CI failures, command failures, logs, or uncertain root causes.
---

# Debug Triage

## Workflow

1. Read `/Users/vita/Vitemis/AGENTS.md`.
2. Confirm path and Git state:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Capture the symptom before proposing a fix:
   - exact command or user action
   - full error class and first relevant stack frame
   - affected platform, runtime, target, or environment
   - recent relevant edits
   - whether the failure is deterministic or flaky
4. Reproduce only with commands that are appropriate for the current role. Review copilots must stay read-only unless the user grants a one-time exception.
5. Narrow the failing layer before editing:
   - configuration or dependency resolution
   - compile or type-check
   - unit, integration, UI, or end-to-end test
   - runtime state, persistence, network, or platform lifecycle
   - data contract, migration, or compatibility
6. Identify the smallest safe fix path. Codex may implement when the user asked for a fix; review copilots must report diagnosis and suggested patches only.
7. Validate with the same failing command when feasible, then add the narrowest related check that proves the root cause is covered.

## Output

Use this structure:

```text
MODEL_CHECK_RESULT:
PATH_CHECK_RESULT:
SYMPTOM:
REPRODUCTION_RESULT:
ROOT_CAUSE:
FIX_PLAN_OR_PATCH_DIRECTION:
VALIDATION_RESULT:
UNCERTAINTIES:
NEXT_RECOMMENDED_ACTION:
```

When a durable report is requested, write it only to the current role's permitted report directory and use `MM_DD_YY-HH_MM-xxxx.md`.
