---
name: dirty-worktree-summary
description: Summarize uncommitted changes in a Vitemis repository without mutating Git state. Use before implementation, review, cleanup, handoff, reporting, or whenever the user asks what is dirty, what changed, or how to separate existing user edits from current agent work.
---

# Dirty Worktree Summary

## Workflow

1. Read `/Users/vita/Vitemis/docs/VITEMIS_AGENT_POLICY.md`.
2. Confirm the target repository root before interpreting changes:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Use only read-only Git inspection commands. Good defaults:

```sh
git diff --stat
git diff --name-status
git diff --cached --stat
git diff --cached --name-status
```

4. Do not run Git mutation commands, including add, commit, push, pull, fetch, merge, rebase, reset, restore, checkout, clean, tag, branch, or stash.
5. Classify dirty state into:
   - staged changes
   - modified tracked files
   - deleted or renamed files
   - untracked files
   - generated files, caches, or build outputs
   - likely pre-existing user edits versus current-task edits, only when evidence supports the distinction
6. For sensitive paths such as `.env`, credentials, keys, certificates, or private user files, list only the path and status. Do not print, summarize, or quote contents.

## Output

Lead with the operational summary:

```text
MODEL_CHECK_RESULT:
PATH_CHECK_RESULT:
DIRTY_SUMMARY:
LIKELY_USER_CHANGES:
LIKELY_AGENT_CHANGES:
UNTRACKED_FILES:
SENSITIVE_PATHS_NOT_INSPECTED:
VALIDATION_RESULT:
UNCERTAINTIES:
NEXT_RECOMMENDED_ACTION:
```

If a durable report is requested, write it only to the current role's permitted report directory and use `MM_DD_YY-HH_MM-xxxx.md`.
