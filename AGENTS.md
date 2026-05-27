# Agent Instructions

This repository is an agent-agnostic personal skill registry.

When an agent needs a skill from this repo:

1. Read `registry.json` to discover available skills.
2. Select the matching `skills[].id` or `skills[].name`.
3. Read that skill's `skill.json` for package metadata.
4. Load the selected skill's `SKILL.md`.
5. If `SKILL.md` links to sibling files, read those files from the same skill directory.

Do not assume a specific vendor runtime. Unknown frontmatter keys in `SKILL.md` should be ignored unless the current agent supports them.

When adding or editing skills, keep each skill independently installable and preserve relative paths inside its directory.
