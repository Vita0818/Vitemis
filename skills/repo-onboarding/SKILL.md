---
name: repo-onboarding
description: Set up or verify a repository's Vitemis Agent entrypoints and report directories. Use when adding a new project, preparing Agent docs, checking docs/AGENTS.md, docs/CLAUDE.md, docs/GEMINI.md, creating codex-report/claude-report/gemini-report, or aligning a repository with the shared Vitemis Agent policy.
---

# Repo Onboarding

## Workflow

1. Read `/Users/vita/Vitemis/docs/VITEMIS_AGENT_POLICY.md`.
2. Enter the target repository root and run:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Preserve existing files:
   - If `docs/AGENTS.md`, `docs/CLAUDE.md`, or `docs/GEMINI.md` already exists, do not rewrite it unless the user explicitly asks.
   - If a report directory already exists, leave its contents untouched.
4. Ensure these docs-level entrypoints exist:
   - `docs/AGENTS.md`
   - `docs/CLAUDE.md`
   - `docs/GEMINI.md`
5. Ensure these report directories exist:
   - `codex-report/`
   - `claude-report/`
   - `gemini-report/`
6. Keep docs-level entrypoints as thin shims:
   - Read the shared policy.
   - Read the project root entrypoint if present.
   - State the Agent's role and write directory.
   - State report naming as `MM_DD_YY-HH_MM-xxxx.md`.
   - State the report body starts with `MODEL_CHECK_RESULT`.
7. Do not add project-specific architecture, status, task lists, or risk lists to shared Vitemis-level files.

## Completion Check

Before finishing, verify:

```sh
find docs -maxdepth 1 -type f \( -name AGENTS.md -o -name CLAUDE.md -o -name GEMINI.md \) | sort
find . -maxdepth 1 -type d \( -name codex-report -o -name claude-report -o -name gemini-report \) | sort
```

Report created files and any existing files left untouched. Do not run Git mutation commands.
