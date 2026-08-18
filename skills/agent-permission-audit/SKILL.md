---
name: agent-permission-audit
description: Audit a repository or workspace for compliance with Vitemis Agent permission rules. Use when asked to check Agent roles, Codex/Claude/Gemini/Cursor write boundaries, report directories, report naming, Git restrictions, sensitive-file handling, or whether docs/AGENTS.md, docs/CLAUDE.md, and docs/GEMINI.md match the shared policy.
---

# Agent Permission Audit

## Workflow

1. Read `/Users/vita/Vitemis/AGENTS.md`.
2. Enter the target repository root and run:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Inspect only policy-relevant files unless the user explicitly asks for broader review:
   - `AGENTS.md`
   - `CLAUDE.md`
   - `GEMINI.md`
   - `docs/AGENTS.md`
   - `docs/CLAUDE.md`
   - `docs/GEMINI.md`
   - report directories
4. Check the required permissions:
   - Codex is the primary worker.
   - Claude, Gemini, and Cursor are read-only review copilots.
   - Claude writes only `claude-report/`.
   - Gemini writes only `gemini-report/`.
   - Cursor writes only `cursor-report/`.
   - Codex writes work reports only to `codex-report/`.
   - Git mutation commands are forbidden.
   - Sensitive files are not read or written.
5. Check reporting requirements:
   - `codex-report/`, `claude-report/`, and `gemini-report/` exist when required by the project.
   - Reports use `MM_DD_YY-HH_MM-xxxx.md`.
   - Report bodies start with `MODEL_CHECK_RESULT`.
6. Report findings first, ordered by severity, with file paths and line references when available.

## Output

Use this structure:

```text
MODEL_CHECK_RESULT:
PATH_CHECK_RESULT:
FINDINGS:
FILES_INSPECTED:
VALIDATION_RESULT:
UNCERTAINTIES:
NEXT_RECOMMENDED_ACTION:
```

Do not modify project files during an audit unless the user explicitly asks for fixes.
