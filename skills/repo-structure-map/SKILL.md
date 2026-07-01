---
name: repo-structure-map
description: Map a Vitemis repository's structure, entrypoints, tests, generated areas, documentation, and ownership cues before planning changes. Use for repo onboarding, large reviews, cross-project audits, agent handoffs, or whenever the agent needs a reliable high-level map.
---

# Repo Structure Map

## Workflow

1. Read `/Users/vita/Vitemis/docs/VITEMIS_AGENT_POLICY.md`.
2. Confirm repository boundary:

```sh
pwd
git rev-parse --show-toplevel
git status --short
```

3. Inventory files with fast read-only commands. Prefer `rg --files`; fall back to `find` when needed.
4. Exclude dependency caches, build outputs, generated artifacts, and large binary assets unless they are central to the request.
5. Read only the entrypoint files needed to understand structure:
   - agent files and docs
   - package or build manifests
   - top-level README or contributing files
   - source and test directory names
6. Identify:
   - application or library entrypoints
   - primary source roots
   - test roots and test framework clues
   - configuration and build files
   - generated, vendor, cache, or artifact directories
   - documentation and agent policy files
   - report directories
   - likely unsafe or sensitive paths that should not be inspected

## Output

Use this structure:

```text
MODEL_CHECK_RESULT:
PATH_CHECK_RESULT:
STRUCTURE_SUMMARY:
ENTRYPOINTS:
SOURCE_ROOTS:
TEST_ROOTS:
BUILD_AND_RUN_CLUES:
DOCS_AND_AGENT_FILES:
GENERATED_OR_IGNORED_AREAS:
SENSITIVE_PATHS_NOT_INSPECTED:
UNCERTAINTIES:
NEXT_RECOMMENDED_ACTION:
```

Do not modify files while mapping unless the user explicitly asks to update documentation. If persisting the map, write it only to the current role's permitted report directory with `MM_DD_YY-HH_MM-xxxx.md`.
