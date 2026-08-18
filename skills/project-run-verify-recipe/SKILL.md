---
name: project-run-verify-recipe
description: Discover and document the correct run, build, lint, and test commands for a Vitemis repository. Use when onboarding a repo, preparing an agent work environment, deciding how to verify changes, or turning scattered project instructions into a concise verification recipe.
---

# Project Run Verify Recipe

## Workflow

1. Read `/Users/vita/Vitemis/AGENTS.md`.
2. Confirm repository boundary:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Read project entrypoints and manifests before inventing commands. Useful targets include:
   - `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, and `docs/`
   - `README*`, `CONTRIBUTING*`, `Makefile`, `Justfile`
   - `package.json`, lockfiles, workspace files
   - `pyproject.toml`, `requirements*.txt`, `uv.lock`, `poetry.lock`
   - `Package.swift`, `*.xcodeproj`, `*.xcworkspace`
   - `settings.gradle*`, `build.gradle*`, `gradlew`
   - `Cargo.toml`, `go.mod`, `.csproj`, `.sln`, `pubspec.yaml`
4. Prefer declared scripts and local wrappers over global commands.
5. Mark unsafe or state-changing commands clearly. Do not run deploy, publish, migration, install, cleanup, codegen, formatter, or package-update commands unless the user explicitly asks.
6. If command execution is appropriate, record exact command, cwd, result, and any files it created or modified.

## Recipe Format

Produce a concise recipe:

```text
MODEL_CHECK_RESULT:
PATH_CHECK_RESULT:
ENVIRONMENT_ASSUMPTIONS:
RUN_COMMANDS:
BUILD_COMMANDS:
TEST_COMMANDS:
LINT_OR_STATIC_CHECKS:
FORMAT_CHECKS:
UNSAFE_COMMANDS_TO_AVOID:
VALIDATION_RESULT:
UNCERTAINTIES:
NEXT_RECOMMENDED_ACTION:
```

When persisting the recipe as a report, write it only to the current role's permitted report directory and use `MM_DD_YY-HH_MM-xxxx.md`.
