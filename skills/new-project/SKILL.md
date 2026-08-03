---
name: new-project
description: Initialize a new project or upgrade an existing Claude Code, Codex, or OpenCode project to the semiformal SDD workflow. Use when asked to create or reconcile AGENTS.md and CLAUDE.md, install or update docs/workflow.md, or enable the contract-audit templates.
---

# Initialize or Upgrade a Project

Set up one shared project-rules entity for Claude Code, Codex, and OpenCode, and synchronize the project to the currently installed workflow template.

## Preconditions

1. Work at the project root. Use the Git root when one exists; otherwise use the current empty or new-project directory.
2. Require these installed templates:
   - `~/templates/AGENTS-template.md`
   - `~/templates/workflow.md`
3. Inspect `AGENTS.md` and `CLAUDE.md` with symlink-aware checks. Check `-L` before `-e` so broken links are not mistaken for absent files.
4. Never delete, replace, or relink an existing regular file or symlink without explicit user approval.

## Reconcile Project Rules

Choose exactly one rules entity and make the other filename a relative symlink to it.

| Existing state | Required action |
|---|---|
| Neither path exists | Copy `~/templates/AGENTS-template.md` to `AGENTS.md`; create `CLAUDE.md -> AGENTS.md`. |
| Only regular `CLAUDE.md` exists | Preserve it unchanged; create `AGENTS.md -> CLAUDE.md`. |
| Only regular `AGENTS.md` exists | Preserve it unchanged; create `CLAUDE.md -> AGENTS.md`. |
| One path is already a valid relative symlink to the other regular file | Keep it unchanged. |
| Both paths are regular files | Show whether their contents are identical and ask which file remains the entity. If they differ, require the user to choose or merge the content before replacing either path with a symlink. |
| Either path is broken, cyclic, points outside the project, or has another unsupported type | Stop and report the exact state. Repair only after explicit approval. |

After reconciliation:

1. Verify both names resolve successfully.
2. Verify both resolve to the same filesystem entity.
3. Verify the symlink target is relative and is exactly the other project-rules filename.
4. Inspect the entity for a clear instruction to read `docs/workflow.md`. If absent, propose the minimal instruction and add it only with user approval.

## Synchronize the Latest Workflow

Treat the currently installed `~/templates/workflow.md` as the latest available canonical workflow. Do not fetch a different version at runtime.

1. Create `docs/` if it does not exist.
2. If `docs/workflow.md` is absent, copy the canonical workflow into place.
3. If it is byte-for-byte identical to the canonical workflow, leave it unchanged.
4. If it differs, show a unified diff before changing it:
   - If the project copy has no intentional customization, replace it with the canonical workflow after user approval.
   - If it has intentional customization, start from the canonical workflow and reapply only the approved project-specific changes. Do not patch new upstream sections into the old file as the base.
5. Do not declare success while the project still uses an older workflow. The final file must equal the canonical workflow or equal that workflow plus explicitly approved project-specific changes.

## Optional Contract Audit Templates

Ask whether the project expects new subplans with contract documents.

- If yes and `templates/contract-audit/` is absent, require `~/templates/contract-audit/` and copy it into the project.
- If the directory already exists, do not overwrite it silently; compare it with the installed templates and report differences.
- If no or uncertain, skip installation and state that it can be enabled later.

## Fill or Preserve Project Metadata

- For a new project, infer the project name, description, structure, technology stack, commands, and conventions from existing files when possible. Ask only for values that cannot be inferred, then replace the corresponding placeholders in the rules entity.
- For an upgraded project, preserve existing project-specific instructions. Do not replace the entity with the generic template.

## Completion Report

Report:

1. Which file is the rules entity and the exact symlink direction.
2. Whether `docs/workflow.md` was created, already current, replaced, or merged with approved customizations.
3. Whether contract-audit templates were installed, retained, or skipped.
4. Every changed path and any unresolved placeholders or conflicts.

Do not report completion unless both rules filenames resolve to the same entity and the workflow synchronization requirement is satisfied.
