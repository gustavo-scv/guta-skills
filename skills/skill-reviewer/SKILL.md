---
name: skill-reviewer
description: Review agent skills for trigger quality, portability, metadata correctness, installability, and maintainability. Use after creating or editing SKILL.md files, skill manifests, or marketplace-ready skill packages.
---

# Skill Reviewer

Review agent skills as reusable packages, not just markdown documents. Focus on whether another agent can discover, install, load, and apply the skill correctly.

## Review Order

1. Check `SKILL.md` frontmatter.
2. Check the body for clear, actionable instructions.
3. Check references, scripts, and sibling files.
4. Check `skill.json` and registry entries when present.
5. Check portability across Claude Code, Cursor, Codex, Pi, Antigravity, and generic agents.

## What To Inspect

- `name` is lowercase, hyphenated, stable, and matches the directory.
- `description` says what the skill does and when to use it.
- Trigger terms are specific enough for discovery but not so broad that the skill fires constantly.
- Instructions are concise, direct, and worth loading into context.
- Long details live in one-level reference files linked from `SKILL.md`.
- Commands are portable or clearly split by shell/platform.
- Relative paths use forward slashes.
- Agent-specific metadata is harmless for agents that ignore unknown keys.
- Manifests list every file needed for installation.
- Registry entries point to existing `path`, `manifest`, and `entrypoint` files.

## Quality Bar

Flag issues when:

- The skill is mostly generic advice an agent already knows.
- The description is vague, first-person, or missing trigger scenarios.
- The skill depends on hidden local state without explaining required env vars, files, or tools.
- The package cannot be copied independently because references point outside its directory.
- The skill includes stale, platform-specific commands without alternatives.
- The manifest and `SKILL.md` disagree about the name, description, or required files.
- The skill is bloated enough that a reference file would make it easier to load selectively.

## Output Format

Lead with findings, ordered by severity. For each finding, include:

- The affected file.
- The concrete problem.
- The change needed to make the skill easier to discover, install, or use.

If there are no blockers, say that clearly and mention any remaining portability or test gaps.
