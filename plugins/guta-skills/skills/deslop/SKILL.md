---
name: deslop
description: Remove AI-generated code slop from current branch changes while preserving behavior. Use when cleaning diffs, reviewing AI-authored edits, removing unnecessary comments, simplifying defensive code, or aligning changes with local style.
---

# Remove AI code slop

Check the diff against the branch's merge base and remove AI-generated slop introduced in the current branch. Default to `main` only when it is clearly the repository's base branch.

## Focus Areas

- Extra comments that are unnecessary or inconsistent with local style
- Defensive checks or try/catch blocks that are abnormal for trusted code paths
- Casts to `any` used only to bypass type issues
- Deeply nested code that should be simplified with early returns
- Other patterns inconsistent with the file and surrounding codebase

## Guardrails

- Keep behavior unchanged unless fixing a clear bug.
- Prefer minimal, focused edits over broad rewrites.
- Do not revert unrelated user changes or cleanup outside the reviewed diff.
- Keep the final summary concise (1-3 sentences).
