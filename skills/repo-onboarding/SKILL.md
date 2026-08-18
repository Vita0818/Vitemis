---
name: repo-onboarding
description: Set up or verify a repository's Vitemis Agent entrypoints and report directories. Use when adding a new project, preparing Agent docs, checking root AGENTS.md, CLAUDE.md, GEMINI.md, docs-level shims, creating codex-report/claude-report/gemini-report, or aligning a repository with the shared Vitemis Agent policy.
---

# Repo Onboarding

## Workflow

1. Read `/Users/vita/Vitemis/AGENTS.md`.
2. Enter the target repository root and run:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Preserve existing files:
   - If root `AGENTS.md`, `CLAUDE.md`, or `GEMINI.md` already exists, do not rewrite it unless the user explicitly asks.
   - If `docs/AGENTS.md`, `docs/CLAUDE.md`, or `docs/GEMINI.md` already exists, do not rewrite it unless the user explicitly asks.
   - If a report directory already exists, leave its contents untouched.
4. Ensure these root entrypoints exist:
   - `AGENTS.md`
   - `CLAUDE.md`
   - `GEMINI.md`
5. Ensure these docs-level entrypoints exist:
   - `docs/AGENTS.md`
   - `docs/CLAUDE.md`
   - `docs/GEMINI.md`
6. Ensure the temporary next-target document exists when setting up an active project:
   - `docs/NEXT_TARGET.md`
7. Ensure these report directories exist:
   - `codex-report/`
   - `claude-report/`
   - `gemini-report/`
8. Root `CLAUDE.md` and `GEMINI.md` must be real automatic entrypoints, not only docs shims:
   - Read `/Users/vita/Vitemis/AGENTS.md`.
   - Read project `AGENTS.md`.
   - State Claude/Gemini are read-only review copilots.
   - Restrict writes to `claude-report/` or `gemini-report/` respectively.
   - Forbid source edits, docs edits, build/config edits, Git mutation, and mutating commands.
   - State that even if the user asks for a commit, Claude/Gemini must not execute it and should remind the user to use Codex with current-Git-root-only scope.
   - Require completed persistent changes to be written back to relevant project docs promptly, or require the final report to explain why no docs update was needed.
9. Keep docs-level entrypoints as thin shims:
   - Read the shared policy.
   - Read the project root entrypoint if present.
   - State the Agent's role and write directory.
   - State report naming as `MM_DD_YY-HH_MM-xxxx.md`.
   - State the report body starts with `MODEL_CHECK_RESULT`.
10. Do not add project-specific architecture, status, task lists, or risk lists to shared Vitemis-level files.

## Completion Check

Before finishing, verify:

```sh
find . -maxdepth 1 -type f \( -name AGENTS.md -o -name CLAUDE.md -o -name GEMINI.md \) | sort
find docs -maxdepth 1 -type f \( -name AGENTS.md -o -name CLAUDE.md -o -name GEMINI.md \) | sort
test -f docs/NEXT_TARGET.md
find . -maxdepth 1 -type d \( -name codex-report -o -name claude-report -o -name gemini-report \) | sort
```

Report created files and any existing files left untouched. Do not run Git mutation commands unless the user explicitly requested the exact Git operation in the current task. A commit request is current-Git-root-only; do not stage, commit, tag, push, or otherwise mutate nested Git repositories, submodules, vendored checkouts, or generated dependency checkouts.
