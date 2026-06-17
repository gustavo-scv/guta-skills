# Skills

Each child directory is an independently installable skill package.

Marketplace installs use the mirrored skill copies under `../plugins/guta-skills/skills/`.

## Package Rules

- `SKILL.md` is required and is the canonical instruction file.
- `skill.json` is required and describes the package for agents, scripts, and marketplaces.
- Copy the whole skill directory when installing a skill.
- Keep files referenced by `SKILL.md` inside the same skill directory.
- Use forward-slash paths in documentation and manifests.
- Agent-specific frontmatter is allowed when it is harmless for other agents to ignore.

## Current Skills

- `repomix-cli`: Pack local or remote repositories into AI-ready context files with token estimation and safety gates.
- `remove-ai-signs`: Revise drafts to remove common AI-writing signs while preserving meaning, facts, citations, voice, and user intent.
- `playwright-cli`: Drive a real browser from the shell with Playwright for navigation, interaction, testing, screenshots, tracing, and network mocking.
- `deslop`: Remove AI-generated code slop from branch changes without changing behavior.
- `thermo-nuclear-code-quality-review`: Run a strict maintainability and architecture audit focused on structural simplification.
- `skill-reviewer`: Review skills for trigger quality, metadata correctness, portability, installability, and marketplace readiness.
