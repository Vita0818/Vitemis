# Vitemis Agent Policy

This file is the Vitemis workspace-level Agent policy and Codex auto-discovery entrypoint.

OpenAI Codex automatically reads `AGENTS.md` when `/Users/vita/Vitemis` is the active project root or current directory. Keep this file reusable: do not add project lists, project status, technology stacks, business risks, or repository-specific instructions here.

If Codex is running inside a nested Git repository, that repository's own `AGENTS.md` is the automatic entrypoint. To inherit the Vitemis policy in a nested project, the project-level `AGENTS.md` must explicitly reference `/Users/vita/Vitemis/AGENTS.md`.

## 1. Role Model

Codex is the primary worker.

- Codex may read and modify files in the current target repository when the user explicitly asks for implementation, fixes, documentation, tests, maintenance, or version-control setup.
- Codex must follow the current repository `AGENTS.md`, repository documentation, the user task boundary, and this policy.
- Codex must not modify across repository boundaries unless the user explicitly asks for a cross-repository task.
- Codex may write work reports or audit reports under `codex-report/`.

Claude, Gemini, and Cursor are read-only review copilots.

- They may only perform read-only review, evaluation, risk analysis, test suggestions, and code comments.
- They must not modify source, tests, configuration, build scripts, project files, resources, documentation bodies, templates, or generated project metadata.
- Their only write locations are their own report directories:
  - Claude: `claude-report/`
  - Gemini: `gemini-report/`
  - Cursor: `cursor-report/`
- If the directory does not exist, a review copilot may create only its own report directory.
- Report directories may contain only Markdown or text reports, not scripts, patches, executables, configuration files, or project resources.
- Unless the user grants a one-time exception, review copilots must not run commands that intentionally mutate the worktree, including build, format, package, codegen, install, cleanup, or migration commands.

## 2. Git Permissions

Git version control is manual by default. Without an explicit user request for version-control work in the current task, all Agents may only inspect Git state. A request to edit files, organize docs, run tests, or prepare work is not a request to commit.

Allowed read-only Git commands:

```sh
git status --short
git diff
git diff --check
git log
git show
git rev-parse --show-toplevel
```

Codex may run non-destructive Git mutation commands such as `git init`, `git remote`, `git add`, `git commit`, `git fetch`, `git branch`, `git tag`, or `git push` only when the user explicitly asks for that exact Git operation in the current task and the environment permission policy allows it.

Commit scope is repository-local:

- If the user asks for a commit, stage and commit only files that belong to the current Git root.
- Do not recurse into, stage, commit, tag, push, or otherwise mutate nested Git repositories, submodules, vendored checkouts, or generated dependency checkouts.
- Do not mix a parent repository commit with child repository commits in the same task unless the user explicitly names each repository and asks for those separate commits.
- If a requested commit could affect nested repositories, stop and report the repository boundaries before running Git mutation.

All Agents are forbidden from destructive Git operations unless the user explicitly requests the exact destructive operation and confirms the target:

```sh
git reset --hard
git clean
git checkout -- <path>
git restore --source
git rebase
force push
history rewrite
```

Additional rules:

- Do not stage unrelated files.
- Do not stage nested repository contents or submodule pointer changes unless the user explicitly requests that exact parent-repository pointer update.
- Do not stash unless the user explicitly asks.
- Do not create PRs unless the user explicitly asks.
- Do not delete, revert, or clean user changes.
- If a needed Git operation is outside the current task, list the command for the user instead of running it.

## 3. Repository Entry Protocol

Before an Agent works in a repository, it must confirm the current path and repository boundary.

Run from the intended repository root:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

Requirements:

- `pwd` and `git rev-parse --show-toplevel` must point to the same repository root.
- If the current directory is not the target repository root, stop modifying and report the path mismatch.
- After reading `git status --short`, distinguish pre-existing user changes from planned current-task changes.
- Do not overwrite, revert, format, clean, or opportunistically fix unrelated existing changes.

## 4. Sensitive Information Boundaries

All Agents must not read, print, summarize, copy, transmit, or write:

- `.env`, `.env.*`
- API keys, tokens, passwords, cookies, sessions
- private keys, certificates, p12 files, provisioning profiles
- SSH keys
- Keychain contents
- account credentials
- unrelated private user files

If a task appears to require sensitive information, stop and ask the user for a safe alternative.

## 5. Write Directory Conventions

Codex:

- May write business files according to the user task and repository rules.
- May write `codex-report/`.
- Must not write Claude/Gemini/Cursor report directories unless the user explicitly asks to organize reports.
- Must promptly write completed persistent changes back to the relevant project documentation. If no documentation update is needed, explain why in the final report.

Claude:

- May write only `claude-report/`.

Gemini:

- May write only `gemini-report/`.

Cursor:

- May write only `cursor-report/`.

Recommended report directories live at the target repository root:

```text
<repo>/codex-report/
<repo>/claude-report/
<repo>/gemini-report/
<repo>/cursor-report/
```

## 6. Report Naming And Body

Reports written to report directories must use:

```text
MM_DD_YY-HH_MM-xxxx.md
```

Rules:

- `MM_DD_YY-HH_MM` uses local time in 24-hour format.
- `xxxx` is a short task slug using lowercase ASCII letters, digits, and hyphens.
- Example: `06_30_26-21_45-permission-audit.md`.
- Do not overwrite an existing report filename.

Recommended report body:

```text
# <short report title>

## MODEL_CHECK_RESULT
<current model name; use unknown if unavailable. Unless a project entrypoint has a stricter model gate, record the model without stopping solely because a version string differs.>

## PATH_CHECK_RESULT
<pwd, Git root, and whether they match the expected repository.>

## FILES_WRITTEN
<new or modified files; read-only review reports write none.>

## SUMMARY
<completed work or review conclusion.>

## VALIDATION_RESULT
<commands run and results; explicitly say when build/test was not run.>

## UNCERTAINTIES
<unknowns, evidence gaps, conflicts, or items needing human confirmation.>

## NEXT_RECOMMENDED_ACTION
<suggested next step; do not auto-run unrelated follow-up work.>
```

Review copilot reports should lead with risks, findings, evidence, and uncertainties. Codex work reports should clearly separate actual edits, validation, and unverified boundaries.

## 7. Project-Level Entrypoints

Each project should keep project-specific facts in its own `AGENTS.md`.

Project-level `AGENTS.md` files should:

- Reference `/Users/vita/Vitemis/AGENTS.md` when the project should inherit this policy.
- State the project root and working-directory checks.
- State modification boundaries, source directories, test entrypoints, and build entrypoints.
- State project-specific forbidden areas.
- State completion criteria and final report format.
- Require completed persistent changes to be written back to relevant project documentation promptly; if no documentation update is needed, require the final report to explain why.
- Use `docs/NEXT_TARGET.md` as the temporary next-objective record when a concrete next target exists. Project agents must read it when present and delete it when the target is completed or no longer current.

This Vitemis-level policy must not maintain project lists, project-specific technology stacks, or project-specific risks. When a project is added, migrated, archived, or renamed, update only that project's own `AGENTS.md`.

## 8. Dependency-First And No-Fallback Policy

This rule applies to every first-party project, platform target, tool, test surface, and future implementation in the Vitemis workspace. The canonical detailed contract is `docs/DEPENDENCY_POLICY.md`.

- When a user-selected, already-adopted, or license/provenance/security/platform-approved external dependency provides the required capability, Codex must integrate that dependency's official capability directly.
- Codex must not reimplement an equivalent capability or add a substitute adapter, shim, compatibility layer, wrapper, proxy, facade, parallel backend, preview backend, shadow implementation, or "temporary fallback" around it.
- Local code is limited to the thinnest lifecycle, type, permission, configuration, and bundle wiring strictly required by the dependency's official API. It must not reinterpret, duplicate, or replace the dependency's core behavior.
- If the exact dependency cannot currently be integrated because of version, build, signing, licensing, platform, security, or official-API limitations, implementation of that capability must stop. Codex must report the blocker and request a user decision; it must not silently degrade, switch technology, select another provider/backend, or continue with an incomplete substitute.
- Existing fallback or duplicate implementations do not establish precedent. They must not be expanded and should be recorded as removal candidates for a separately authorized task.
- Security fail-closed behavior and explicitly required backward decoding/data migration are not functional fallbacks. They must remain narrowly scoped to their security or compatibility contract and must never become alternate product implementations.
- Only a new explicit user decision naming the exact dependency, exact scope, and exit condition may override this rule.

Every project-level `AGENTS.md`, `docs/ARCHITECTURE.md`, `docs/DO_NOT_BREAK.md`, and `docs/TESTING.md` must carry this rule or an equally strict project-specific version. Verification must prove that dependency failure is explicit and that no secondary implementation or provider is invoked.
