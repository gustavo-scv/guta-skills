# Guta Skills

Personal, agent-agnostic repository for reusable AI agent skills.

This repo is organized like a small skills marketplace: every skill is a self-contained directory under `skills/`, and `registry.json` indexes the available packages so Claude Code, Cursor, Codex, Pi, Antigravity, or any other agent can pull one skill without cloning unrelated instructions.

## Quick Summary

Use this repo as a central source for personal agent skills. Each skill can be installed independently by copying its directory into the target agent's skills folder, or by reading `registry.json` and fetching the selected `skills[].path`.

The repository is intentionally portable:

- Paths in manifests use forward slashes.
- Skill packages keep their own references beside `SKILL.md`.
- `SKILL.md` remains the canonical human-readable instruction file.
- `skill.json` and `registry.json` provide machine-readable metadata for marketplace-style discovery.

## Skills At A Glance

- `remove-ai-signs`: Revise drafts to remove common AI-writing signs while preserving meaning, facts, citations, voice, and user intent.
- `playwright-cli`: Drive a real browser from the shell for navigation, UI checks, screenshots, traces, videos, network mocking, scraping, and Playwright test debugging.

## Repository Contract

```text
.
├── registry.json                 # Machine-readable catalog of all skills
├── schemas/
│   ├── registry.schema.json       # Registry JSON schema
│   └── skill.schema.json          # Per-skill manifest JSON schema
└── skills/
    └── <skill-name>/
        ├── SKILL.md              # Canonical agent instructions
        ├── skill.json            # Machine-readable package metadata
        └── references/           # Optional files used by SKILL.md
```

The skill directory is the install unit. Consumers should copy the whole directory so relative links from `SKILL.md` keep working.

## Installing One Skill

Use the target agent's personal skills directory when it has one:

| Agent | Suggested destination |
| --- | --- |
| Claude Code | `~/.claude/skills/<skill-name>/` |
| Cursor | `~/.cursor/skills/<skill-name>/` |
| Codex | `~/.codex/skills/<skill-name>/` |
| Pi | Use the agent's personal instruction or skill import location |
| Antigravity | Use the agent's project or personal skill import location |
| Generic agents | Import `SKILL.md` and preserve sibling files |

Example:

```bash
git clone <repo-url> guta-skills
mkdir -p ~/.claude/skills/playwright-cli
cp -R guta-skills/skills/playwright-cli/. ~/.claude/skills/playwright-cli/
```

PowerShell:

```powershell
git clone <repo-url> guta-skills
New-Item -ItemType Directory -Force "$HOME/.claude/skills/playwright-cli"
Copy-Item -Recurse -Force "guta-skills/skills/playwright-cli/*" "$HOME/.claude/skills/playwright-cli/"
```

For sparse checkouts or marketplace crawlers, read `registry.json`, select one `skills[].path`, then fetch that directory only.

## Adding A Skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter containing at least `name` and `description`.
2. Add `skills/<skill-name>/skill.json` using `schemas/skill.schema.json`.
3. Add the skill to `registry.json`.
4. Keep optional reference files one level below the skill directory when possible.
5. Treat `SKILL.md` as the human-readable source of truth and `skill.json` as the package manifest.

Skill names should use lowercase letters, numbers, and hyphens.
