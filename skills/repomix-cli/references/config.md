# Repomix Configuration File

Create with `repomix --init` (local) or `repomix --init --global` (machine-wide). The file is `repomix.config.json` by default; point at another with `-c <path>`. It supports JSON, JSONC/JSON5 (comments and trailing commas allowed), and a `$schema` for editor autocompletion.

**Precedence:** CLI flags > local config > global config > built-in defaults.

## Complete annotated example

```json
{
  "$schema": "https://repomix.com/schemas/latest/schema.json",
  "input": {
    "maxFileSize": 50000000
  },
  "output": {
    "filePath": "repomix-output.xml",
    "style": "xml",
    "parsableStyle": false,
    "compress": false,
    "headerText": "Custom header information for the packed file.",
    "fileSummary": true,
    "directoryStructure": true,
    "files": true,
    "removeComments": false,
    "removeEmptyLines": false,
    "topFilesLength": 5,
    "showLineNumbers": false,
    "truncateBase64": false,
    "copyToClipboard": false,
    "includeEmptyDirectories": false,
    "git": {
      "sortByChanges": true,
      "sortByChangesMaxCommits": 100,
      "includeDiffs": false,
      "includeLogs": false,
      "includeLogsCount": 50
    }
  },
  "include": ["**/*"],
  "ignore": {
    "useGitignore": true,
    "useDotIgnore": true,
    "useDefaultPatterns": true,
    "customPatterns": ["additional-folder", "**/*.log"]
  },
  "security": {
    "enableSecurityCheck": true
  },
  "tokenCount": {
    "encoding": "o200k_base"
  }
}
```

## Field reference

### `input`
| Field | Default | Meaning |
|-------|---------|---------|
| `maxFileSize` | `50000000` | Skip files larger than this many bytes (avoids packing huge binaries/data). |

### `output`
| Field | Default | Meaning |
|-------|---------|---------|
| `filePath` | `"repomix-output.xml"` | Output file path (extension can drive format). |
| `style` | `"xml"` | `xml` \| `markdown` \| `json` \| `plain`. Keep `xml` for LLM input. |
| `parsableStyle` | `false` | Escape special chars for always-valid XML/Markdown. |
| `compress` | `false` | Tree-sitter structure-only extraction (big token saver). |
| `headerText` | — | Custom text prepended to the output. |
| `fileSummary` | `true` | Include the summary section. |
| `directoryStructure` | `true` | Include the directory tree. |
| `files` | `true` | Include file contents (set `false` for a metadata-only map). |
| `removeComments` | `false` | Strip comments. |
| `removeEmptyLines` | `false` | Strip blank lines. |
| `topFilesLength` | `5` | How many largest files to list in the summary. |
| `showLineNumbers` | `false` | Prefix lines with line numbers. |
| `truncateBase64` | `false` | Truncate long base64 strings. |
| `copyToClipboard` | `false` | Also copy output to clipboard. |
| `includeEmptyDirectories` | `false` | Show empty folders in the tree. |

### `output.git`
| Field | Default | Meaning |
|-------|---------|---------|
| `sortByChanges` | `true` | List most-frequently-changed files first. |
| `sortByChangesMaxCommits` | `100` | How many commits to scan for the change-frequency ranking. |
| `includeDiffs` | `false` | Append uncommitted git diffs. |
| `includeLogs` | `false` | Append commit logs. |
| `includeLogsCount` | `50` | Number of commits when `includeLogs` is on. |

### `include`
Array of glob patterns for files to include. `["**/*"]` packs everything not ignored. Equivalent to `--include`.

### `ignore`
| Field | Default | Meaning |
|-------|---------|---------|
| `useGitignore` | `true` | Honor `.gitignore`. |
| `useDotIgnore` | `true` | Honor `.ignore`. |
| `useDefaultPatterns` | `true` | Apply built-in ignores (`node_modules`, `.git`, build dirs). |
| `customPatterns` | `[]` | Extra globs to ignore. Can also live in a `.repomixignore` file. |

### `security`
| Field | Default | Meaning |
|-------|---------|---------|
| `enableSecurityCheck` | `true` | Scan for and flag likely secrets (API keys, passwords) before packing. Leave on unless it produces false positives. |

### `tokenCount`
| Field | Default | Meaning |
|-------|---------|---------|
| `encoding` | `"o200k_base"` | Tokenizer for counts: `o200k_base` (GPT-4o), `cl100k_base` (GPT-3.5/4), etc. Match it to the target model for accurate sizing. |
