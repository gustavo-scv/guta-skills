# Repomix Workflows

End-to-end recipes. Each assumes repomix is available (`repomix` or `npx --yes repomix`). Substitute `npx --yes repomix` if only the local version resolves.

## 1. Token-safe ingestion (the default flow)

The goal: feed a codebase to the model without blowing up context. Size first, ingest second.

```bash
# 1. Preview where the tokens are, no full read needed
repomix --token-count-tree 1000

# 2. Pack with a hard guard. Non-zero exit means it's too big.
repomix --style xml --token-budget 200000
```

- If step 2 succeeds (exit 0): the pack is within budget — read `repomix-output.xml` and proceed with the task.
- If step 2 fails (non-zero exit): **do not read the file.** Tell the user the size, then narrow:

```bash
# Option A — restrict to the code that matters
repomix --include "src/**/*" --token-budget 200000

# Option B — compress to structure only (often a massive reduction)
repomix --compress --token-budget 200000

# Option C — drop tests, docs, generated dirs
repomix --ignore "tests/**,docs/**,**/*.min.js" --token-budget 200000
```

Re-pack and re-check until under budget or the user explicitly approves a larger pack.

## 2. Review my current changes

Fold uncommitted work and recent history into the pack so the model has full context for a review.

```bash
repomix --include-diffs --include-logs --include-logs-count 20 --style xml
```

Then read the output and review. For a focused review, also scope with `--include` to the changed area.

## 3. Analyze a remote GitHub repo by URL

No clone required. Always size remote repos first — you don't know how big they are.

```bash
# Preview size
repomix --remote owner/project --token-count-tree 1000

# Pack a specific branch/tag/commit under budget
repomix --remote owner/project --remote-branch main --token-budget 200000 -o project.xml
```

If it's too large, compress or scope just like a local repo:

```bash
repomix --remote owner/project --compress --include "src/**" --token-budget 200000
```

## 4. High-level map of an unfamiliar / huge repo

When you only need the shape of a codebase, skip file contents entirely — near-zero token cost.

```bash
repomix --no-files --include-full-directory-structure -o map.xml
```

Or a structure-only view that still shows signatures:

```bash
repomix --compress --remove-comments -o structure.xml
```

## 5. Pipe straight into another LLM CLI (no file written)

`--stdout` emits the pack with logging suppressed, so it pipes cleanly.

```bash
repomix --compress --stdout | llm "What does this codebase do and what are the main entry points?"
```

Mind the gate here too — piping a huge pack into another tool spends tokens there. Check size with `--token-count-tree` first if unsure.

## 6. Custom file selection via stdin

Drive selection with any tool that lists paths.

```bash
# Only TypeScript files tracked by git
git ls-files "*.ts" | repomix --stdin --stdout

# Only files matching a search
rg -l "TODO" | repomix --stdin -o todos.xml
```

Ignore rules still apply to stdin-listed files.

## 7. Reusable project config

Stop retyping flags for a repo you pack often.

```bash
repomix --init
# edit repomix.config.json — set style, include, ignore, compress, token budget defaults
repomix            # now uses the config; CLI flags still override per run
```

See `config.md` for every field.

## 8. CI guard for context size

Fail a pipeline if the packed repo would overflow a target model's window.

```bash
repomix --style xml --token-budget 200000 --quiet \
  || { echo "::error::repomix output exceeds 200k tokens"; exit 1; }
```

## 9. Generate a Claude Agent Skill from a repo

```bash
# Interactive (prompts for personal vs project skills location)
repomix --skill-generate my-project-skill

# Unattended (CI-friendly)
repomix --remote owner/project --skill-generate my-skill --skill-output ./output --force
```

Produces `.claude/skills/<name>/SKILL.md` plus `references/` (summary, structure, files, tech stacks).

## Choosing an output format quickly

- **Feeding an LLM** → `--style xml` (default). Best-supported structure for Claude/GPT/Gemini.
- **Human reading or docs** → `--style markdown`.
- **Scripted post-processing** → `--style json`, then `jq` (e.g. `jq -r '.directoryStructure' repomix-output.json`).
- **Minimal concatenation** → `--style plain`.
