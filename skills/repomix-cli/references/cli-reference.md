# Repomix CLI — Complete Option Reference

Every option the `repomix` command accepts, grouped by purpose. Each entry says what it does and when to reach for it. Flags always override settings in `repomix.config.json`.

Invoke as `repomix [directories...] [options]`. With no arguments it packs the current directory to `repomix-output.xml`.

## Table of contents

1. [Positional input](#positional-input)
2. [Basic / meta](#basic--meta)
3. [Output options](#output-options)
4. [Output content shaping](#output-content-shaping)
5. [File selection & ignore](#file-selection--ignore)
6. [Remote repository](#remote-repository)
7. [Git integration](#git-integration)
8. [Configuration](#configuration)
9. [Security](#security)
10. [Token counting](#token-counting)
11. [Console / IO](#console--io)
12. [MCP server](#mcp-server)
13. [Agent skills generation](#agent-skills-generation)

---

## Positional input

| Argument | Description |
|----------|-------------|
| `directories...` | Space-separated directories to process, e.g. `repomix src tests docs`. Default `.` (current dir). Each is packed with include/ignore rules applied. |

Narrowing to specific directories is the cheapest way to shrink a pack — do it before reaching for compression.

## Basic / meta

| Option | Description |
|--------|-------------|
| `-v, --version` | Print the installed repomix version and exit. Use this to detect whether repomix is installed. |
| `-h, --help` | Show built-in help. Handy to confirm a flag exists in the user's installed version. |

## Output options

| Option | Description |
|--------|-------------|
| `-o, --output <file>` | Output file path. Default `repomix-output.xml`. The extension can drive the format (`.xml`, `.md`, `.txt`, `.json`). Use `"-"` as the value to write to stdout. |
| `--style <style>` | Output format: `xml` (default), `markdown`, `json`, or `plain`. **Keep XML for LLM input** — it is what major model providers recommend. |
| `--parsable-style` | Escape special characters so the output is always valid XML/Markdown even when source code contains markup that would otherwise break the structure. Reach for this if downstream parsing of the packed file fails. |
| `--stdout` | Write the packed output to stdout instead of a file, and suppress all logging. Use when piping into another command (`repomix --stdout \| llm "…"`). Mutually exclusive with `-o`. |
| `--split-output <size>` | Split output into multiple numbered files (`repomix-output.1.xml`, `.2.xml`, …). Size accepts `500kb`, `2mb`, `1.5mb`. Use when a single file is too large for an upload limit. |
| `--copy` | After packing, also copy the result to the system clipboard. Convenient when the user will paste into a chat UI. |

## Output content shaping

These control what goes *inside* the packed file. Most are token-reduction levers — apply them when a pack is too large (see the token gate in SKILL.md).

| Option | Description |
|--------|-------------|
| `--compress` | Extract essential code structure (classes, functions, interfaces, signatures) via Tree-sitter, dropping implementation bodies. **The biggest single token saver** when you need an overview rather than full source. |
| `--remove-comments` | Strip all code comments before packing. |
| `--remove-empty-lines` | Remove blank lines from every file. |
| `--truncate-base64` | Truncate long base64 data strings (embedded images, blobs) to cut noise and tokens. |
| `--output-show-line-numbers` | Prefix each line with its line number. Useful when you'll ask the model to cite exact lines. |
| `--no-file-summary` | Omit the file-summary header section from the output. |
| `--no-directory-structure` | Omit the directory-tree visualization. |
| `--no-files` | Generate metadata + structure only, with **no file contents**. Great for a cheap, high-level map of an unfamiliar or huge repo. |
| `--include-empty-directories` | Include folders that contain no files in the directory structure. |
| `--include-full-directory-structure` | Show the entire repo tree in the Directory Structure section even when `--include` patterns restrict which file *contents* are packed. Gives full context while keeping content focused. |
| `--header-text <text>` | Custom text inserted at the beginning of the output (e.g. task instructions for the model). |
| `--instruction-file-path <path>` | Path to a file whose contents are embedded as custom instructions in the output. Use for longer, reusable prompt preambles. |

## File selection & ignore

Repomix respects `.gitignore`, `.ignore`, `.repomixignore`, and a built-in default ignore set (`node_modules`, `.git`, build dirs) unless told otherwise.

| Option | Description |
|--------|-------------|
| `--include <patterns>` | Include **only** files matching these comma-separated [glob patterns](https://github.com/mrmlnc/fast-glob#pattern-syntax), e.g. `"src/**/*.js,*.md"`. The primary scoping lever. |
| `-i, --ignore <patterns>` | Additional comma-separated globs to exclude on top of the existing ignore rules, e.g. `"*.test.js,docs/**"`. |
| `--no-gitignore` | Do **not** apply `.gitignore` rules. Use when you deliberately want ignored files (e.g. build artifacts) included. |
| `--no-dot-ignore` | Do not apply `.ignore` file rules. |
| `--no-default-patterns` | Do not apply repomix's built-in ignore patterns. Combine carefully — without these you may accidentally pack `node_modules`. |

## Remote repository

Pack a GitHub repo without cloning it yourself.

| Option | Description |
|--------|-------------|
| `--remote <url>` | Clone and pack a remote repo. Accepts a full GitHub URL or `user/repo` shorthand. |
| `--remote-branch <name>` | Branch, tag, **or commit hash** to pack. Defaults to the repo's default branch. |
| `--remote-trust-config` | Trust and load config files that live in the remote repo. **Disabled by default for security** — only enable for repos you trust, since a remote config could change packing behavior. |

URL shorthands also work in place of `--remote-branch`: `…/repo/tree/<branch>` and `…/repo/commit/<hash>`.

## Git integration

| Option | Description |
|--------|-------------|
| `--include-diffs` | Append a section with git diffs of uncommitted (working-tree + staged) changes. Ideal for "review my current changes" tasks. |
| `--include-logs` | Append recent commit logs (default 50 commits). Gives the model history/context. |
| `--include-logs-count <n>` | Number of commits to include with `--include-logs`, e.g. `--include-logs-count 10`. |
| `--no-git-sort-by-changes` | By default repomix lists the most-frequently-changed files first (a useful signal of what matters). This flag disables that ordering. |

## Configuration

| Option | Description |
|--------|-------------|
| `-c, --config <path>` | Use a custom config file instead of `repomix.config.json`. |
| `--init` | Create a new `repomix.config.json` in the current directory, populated with defaults. |
| `--global` | With `--init`, write the config to the user's home/config dir so it applies machine-wide. Local config takes precedence over global. |

Global config locations: Windows `%LOCALAPPDATA%\Repomix\repomix.config.json`; macOS/Linux `$XDG_CONFIG_HOME/repomix/repomix.config.json` or `~/.config/repomix/repomix.config.json`. See `config.md` for all fields.

## Security

| Option | Description |
|--------|-------------|
| `--no-security-check` | Skip the scan for sensitive data (API keys, passwords, tokens). The check is **on by default** and flags likely secrets so you don't ship them to a model. Only disable when you're certain it's safe or the check produces false positives blocking a legitimate pack. |

## Token counting

These let you size a pack without reading it — the core of the token-gate workflow.

| Option | Description |
|--------|-------------|
| `--token-count-tree [threshold]` | Print the file tree annotated with per-file token counts. Optional threshold shows only files with ≥ N tokens, e.g. `--token-count-tree 100`. Use to see *where* the tokens are before trimming. |
| `--token-count-encoding <encoding>` | Tokenizer used for counting: `o200k_base` (GPT-4o, default), `cl100k_base` (GPT-3.5/4), etc. Switch to match the target model for accurate estimates. |
| `--token-budget <number>` | If the packed output exceeds N tokens, repomix exits with a **non-zero status** (the file is still written). The cleanest automated guard for CI and agent workflows — e.g. `--token-budget 200000`. |
| `--top-files-len <number>` | How many of the largest files to list in the summary. Default 5. Raise it to see more of the heavy hitters. |

## Console / IO

| Option | Description |
|--------|-------------|
| `--verbose` | Detailed debug logging: file processing, token counts, configuration. Use when diagnosing why a pack included/excluded something. |
| `--quiet` | Suppress all console output except errors. For scripting where you only care about the file. |
| `--stdin` | Read the list of file paths to pack from stdin, one per line. The listed files are added to the include set (normal ignore rules still apply; paths may be relative or absolute; duplicates are de-duped). Pair with `find`, `git ls-files`, `ripgrep`, etc. for fully custom selection. |

Example of `--stdin`:

```bash
git ls-files "*.ts" | repomix --stdin --stdout
```

## MCP server

| Option | Description |
|--------|-------------|
| `--mcp` | Run repomix as a Model Context Protocol server so MCP-aware AI tools can call it directly. Use when wiring repomix into an MCP client rather than invoking it per-command. |

## Agent skills generation

Generate a Claude Agent Skill that documents a codebase.

| Option | Description |
|--------|-------------|
| `--skill-generate [name]` | Generate Claude Agent Skills output under `.claude/skills/<name>/` (name auto-generated if omitted). Interactive: prompts whether to save to personal (`~/.claude/skills/`) or project (`.claude/skills/`) skills. |
| `--skill-output <path>` | Write the generated skill to this directory directly, skipping the interactive location prompt. Needed for non-interactive/CI runs. |
| `-f, --force` | Skip all confirmation prompts (including overwriting an existing skill directory). Combine with `--skill-output` for fully unattended generation. |

Generated structure:

```text
.claude/skills/<skill-name>/
├── SKILL.md                 # metadata & documentation
└── references/
    ├── summary.md           # purpose, format, statistics
    ├── project-structure.md # directory tree with line counts
    ├── files.md             # all file contents (grep-friendly)
    └── tech-stacks.md       # languages, frameworks, dependencies
```

Non-interactive example, including from a remote repo:

```bash
repomix --remote user/repo --skill-generate my-skill --skill-output ./output --force
```
