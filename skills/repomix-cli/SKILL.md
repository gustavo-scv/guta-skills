---
name: repomix-cli
description: Pack a whole codebase (local folder or remote GitHub repo) into one AI-ready file with the repomix CLI, defaulting to XML for LLM consumption. Use whenever the user wants to bundle, flatten, pack, or "repomix" a repository for an LLM; feed a codebase to Claude/ChatGPT/Gemini; get a single-file snapshot of a repo; estimate or count a repo's token size; extract repo structure, git history, or a compressed code-structure summary; or analyze a GitHub repo by URL — even when they don't name repomix explicitly. Always estimate tokens before reading packed output into context, and confirm with the user before any pack exceeding ~200k tokens.
allowed-tools: Bash(repomix:*) Bash(npx:*) Bash(npm:*)
---

# repomix-cli

[Repomix](https://github.com/yamadashy/repomix) packs an entire repository — local directory or remote GitHub repo — into a single, structured file that an LLM can read in one shot. The default and recommended format is **XML**, because Anthropic, Google, and OpenAI all recommend XML-style tags for structuring model input.

This skill is for *producing and managing those packed files via the CLI*. The golden rule: **packing is cheap and local; reading the result into your context is the expensive part.** Always size the output before you ingest it.

## Workflow at a glance

1. **Ensure repomix is available** (detect → offer global or local install if missing).
2. **Decide scope and format** — which files, which output style (XML by default).
3. **Pack, then check the token count it reports** — do *not* read the whole file blindly.
4. **Token gate**: if the pack is **> ~200k tokens**, stop and confirm with the user before ingesting or sending it anywhere. Offer to narrow scope.
5. **Consume** — read the file (if within budget), pipe it to another tool, or hand the path to the user.

## Step 1 — Detect or install

Repomix is a Node CLI distributed on npm. The user may not have it installed. Check first:

```bash
repomix --version || npx --yes repomix --version
```

- If `repomix` resolves → use `repomix` directly everywhere below.
- If only `npx` works → prefix every command with `npx --yes` (e.g. `npx --yes repomix --style xml`). `npx` fetches it on demand — good for a one-off, slower because it re-resolves each run.
- If neither resolves → **ask the user how they want it installed** before continuing. Do not silently install global packages.

Present the choice plainly:

| Option | Command | When it fits |
|--------|---------|--------------|
| **Global** (persistent) | `npm install -g repomix` (or `yarn global add repomix`, `bun add -g repomix`, `brew install repomix`) | They'll use it repeatedly; want a short `repomix` command. |
| **One-off / local** (no install) | `npx --yes repomix …` | They want to try it once or avoid touching global packages. |

Update a global install with `npm update -g repomix`. After installing, re-run `repomix --version` to confirm.

> Run repomix from the repository root (or pass directory paths) so `.gitignore` and project structure resolve correctly.

## Step 2 — Choose scope and format

**Format (`--style`):** XML is the default and the right choice for LLM consumption — keep it unless the user has a specific reason otherwise. The four formats:

| Style | Flag | Use it for |
|-------|------|-----------|
| **XML** (default) | `--style xml` | Feeding code to any LLM. **Primary choice.** |
| Markdown | `--style markdown` | Human reading, GitHub/docs embedding, lighter-weight model input. |
| JSON | `--style json` | Programmatic post-processing with `jq` or scripts. |
| Plain | `--style plain` | Simplest possible concatenation; minimal markup. |

**Scope:** the more you can narrow up front, the smaller and more useful the pack. Common levers (full list in `references/cli-reference.md`):

- Specific directories: `repomix src tests`
- Glob include: `repomix --include "src/**/*.ts,**/*.md"`
- Extra ignores: `repomix --ignore "**/*.test.ts,dist/**"`
- Compress to structure only (signatures, no bodies — big token savings): `--compress`
- Strip comments / blank lines: `--remove-comments --remove-empty-lines`

## Step 3 — Pack and read the reported token count

Run the pack. Repomix writes the file (default `repomix-output.xml`) **and prints a summary that includes the total token count** (using the `o200k_base` / GPT-4o tokenizer by default). Read that number from the command output — you do **not** need to open the packed file to know its size.

```bash
repomix --style xml
# … prints "Total Tokens: N tokens" in the summary
```

To preview size **before** committing to a full read, prefer one of these cheap probes:

- `repomix --token-count-tree` — packs and prints a file tree annotated with per-file token counts, so you see where the weight is. Add a threshold to focus: `repomix --token-count-tree 1000` (only files ≥ 1000 tokens).
- `repomix --token-budget 200000` — repomix exits with a **non-zero status** if the output exceeds the budget (the file is still written). This is the cleanest automated guard — see the token gate below.

## Step 4 — Token gate (REQUIRED for large repos)

Reading a packed file into your context costs that many tokens, every turn it stays there. A large repo can be enormous. So before ingesting a pack or sending it onward:

**If the reported total exceeds ~200,000 tokens, STOP. Do not read the file into context or pipe it to a model. Tell the user the size and ask how they want to proceed.** A good prompt to the user looks like:

> "Packing this repo produces ~`<N>` tokens (`repomix-output.xml`, `<size>`). That's a large context investment. Want me to (a) proceed and read it anyway, (b) narrow scope — e.g. `--include` only `src/`, drop tests/docs — or (c) use `--compress` to keep just code structure? Compression typically cuts tokens dramatically."

Why this matters: the user pays for that context and it crowds out room for the actual task. Narrowing or compressing usually gives a better result *and* a smaller bill. Only proceed past 200k on explicit confirmation.

To make the gate automatic in scripts or CI, let repomix enforce it:

```bash
repomix --style xml --token-budget 200000 || echo "Pack exceeds 200k tokens — narrow scope before ingesting."
```

Levers to get back under budget (apply, re-pack, re-check): `--include` a subset · `--compress` · `--remove-comments` · `--ignore` heavy dirs · `--no-files` (metadata/structure only).

## Step 5 — Consume the output

- **Hand the file to an LLM via you:** read `repomix-output.xml` (only once it's within budget) and proceed with the user's actual task.
- **Pipe directly to another CLI without writing a file:** `repomix --stdout | llm "Explain this codebase"`. `--stdout` suppresses all logging so the pipe is clean.
- **Clipboard:** add `--copy` to also copy the result to the system clipboard.
- **Just give the user the path:** many users want the file to upload to a chat UI themselves — pack it and tell them where it is.

## Remote GitHub repositories

No clone needed — repomix fetches it for you:

```bash
repomix --remote yamadashy/repomix                       # user/repo shorthand
repomix --remote https://github.com/user/repo            # full URL
repomix --remote user/repo --remote-branch dev           # branch, tag, or commit hash
repomix --remote https://github.com/user/repo/tree/main  # branch URL form
```

The same token gate applies — you often don't know a remote repo's size in advance, so pack with `--token-count-tree` or `--token-budget` first.

## Including git context

When the user is debugging or reviewing recent work, fold git history into the pack:

```bash
repomix --include-diffs                          # uncommitted working-tree + staged diffs
repomix --include-logs                           # last 50 commit messages (default)
repomix --include-logs --include-logs-count 10   # last 10 commits
repomix --include-diffs --include-logs           # both
```

## Reusable configuration

For a repo packed repeatedly, create a config file so the team doesn't retype flags:

```bash
repomix --init            # writes repomix.config.json with defaults
repomix --init --global   # writes a machine-wide default config
```

CLI flags always override the config file. Full field reference in `references/config.md`.

## Generating a Claude Agent Skill from a repo

Repomix can emit a ready-to-use Claude Skill describing a codebase:

```bash
repomix --skill-generate my-skill --skill-output ./output --force
```

This writes `.claude/skills/<name>/` with `SKILL.md` plus reference files (summary, structure, files, tech stack). Details in `references/cli-reference.md`.

## Reference files

Read these when you need more than the common path above:

- **`references/cli-reference.md`** — every CLI option, grouped (input, output, filtering, remote, git, config, security, token counting, MCP, agent skills), each with what it does and when to reach for it. Consult this whenever a user asks for a flag not covered here, or you need the exact behavior of one.
- **`references/config.md`** — the full `repomix.config.json` schema with every field, defaults, and a complete annotated example.
- **`references/workflows.md`** — end-to-end recipes: token-safe ingestion, code review with diffs, remote-repo analysis, compressed structure maps, piping into other LLM CLIs, and CI guards.

## Quick reference — most-used commands

```bash
repomix                                  # pack cwd → repomix-output.xml (XML)
repomix src --include "**/*.ts"          # only TS under src/
repomix --compress                       # code structure only (token-lean)
repomix --token-count-tree 500           # preview token weight by file
repomix --token-budget 200000            # non-zero exit if over budget
repomix --remote user/repo               # pack a GitHub repo
repomix --stdout | llm "Review this"     # pipe straight to another tool
repomix --style markdown -o repo.md      # Markdown instead of XML
repomix --init                           # create repomix.config.json
```
