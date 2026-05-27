# Skills

Each child directory is an independently installable skill package.

## Package Rules

- `SKILL.md` is required and is the canonical instruction file.
- `skill.json` is required and describes the package for agents, scripts, and marketplaces.
- Copy the whole skill directory when installing a skill.
- Keep files referenced by `SKILL.md` inside the same skill directory.
- Use forward-slash paths in documentation and manifests.
- Agent-specific frontmatter is allowed when it is harmless for other agents to ignore.

## Current Skills

- `agentic-board-cli`: Operate and visually verify local tldraw Agentic Boards through the standalone `agentic-board` CLI.
- `playwright-cli`: Drive a real browser from the shell with Playwright for navigation, interaction, testing, screenshots, tracing, and network mocking.
