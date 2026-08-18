---
name: agent-entrypoint-maintenance
description: Create, audit, or maintain Vitemis agent entrypoint files and report-directory rules. Use when asked to ensure AGENTS.md, CLAUDE.md, GEMINI.md, docs-level shims, codex-report, claude-report, gemini-report, cursor-report, report naming, model recording, or role permissions are present and consistent.
---

# Agent Entrypoint Maintenance

## Workflow

1. Read `/Users/vita/Vitemis/AGENTS.md`.
2. Confirm target repository boundary:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Inventory agent entrypoints and report directories:
   - `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`
   - `docs/AGENTS.md`, `docs/CLAUDE.md`, `docs/GEMINI.md`
   - `codex-report/`, `claude-report/`, `gemini-report/`, `cursor-report/` when applicable
4. Preserve existing files by default. Create missing files or directories only when the user requested maintenance. Patch existing files only after reading them and only for the requested policy gap.
5. Keep Vitemis-level documents generic. Do not add project-specific paths, stacks, requirements, risks, or project lists to shared Vitemis-level docs.
6. Project-level entrypoints may contain project-specific rules, but must still reference the shared Vitemis policy and preserve local stricter requirements.

## Required Rules

Agent entrypoints must make these rules visible:

- Codex is the primary worker.
- Claude, Gemini, and Cursor are read-only review copilots.
- Codex writes reports only to `codex-report/`.
- Claude writes only to `claude-report/`.
- Gemini writes only to `gemini-report/`.
- Cursor writes only to `cursor-report/` if that directory is used by the project.
- Review copilots must not run Git mutation commands. Codex may run non-destructive Git mutation only when the user explicitly requests that exact Git operation in the current task. Editing, organizing, fixing, testing, or preparing work is not a commit request.
- If the user requests a commit, the commit is limited to the current Git root. Do not recurse into, stage, commit, tag, push, or otherwise mutate nested Git repositories, submodules, vendored checkouts, or generated dependency checkouts unless the user explicitly names each repository and asks for separate repository-local Git actions.
- Completed persistent changes must be written back to the relevant project docs promptly. If no docs update is needed, the final report must say why.
- `docs/NEXT_TARGET.md` is the temporary next-objective record. Read it when present, keep at most one active target, and delete it when the target is complete or no longer current.
- Reports use `MM_DD_YY-HH_MM-xxxx.md`.
- Report bodies begin with `MODEL_CHECK_RESULT` and include path, files written, summary, validation, uncertainties, and next action.
- Model identity is recorded for traceability. Unless the project entrypoint declares a stricter model gate, do not stop work solely because a model version string differs.

## Output

When reporting maintenance results, use:

```text
MODEL_CHECK_RESULT:
PATH_CHECK_RESULT:
FILES_WRITTEN:
SUMMARY:
VALIDATION_RESULT:
UNCERTAINTIES:
NEXT_RECOMMENDED_ACTION:
```

If a durable report is requested, write it only to the current role's permitted report directory with `MM_DD_YY-HH_MM-xxxx.md`.
