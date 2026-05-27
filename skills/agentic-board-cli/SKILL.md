---
name: agentic-board-cli
description: Use when user needs to inspect, update, diagram, template, export, or visually verify a local tldraw Agentic Board through the standalone `agentic-board` CLI and browser bridge. Trigger for Agentic Board workflow cards, SDK shape creation, board templates, bridge diagnostics, run_batch transactions, screenshots/captures, and live board regression checks.
---

# Agentic Board CLI

Use the standalone CLI path. Do not rely on the old plugin/MCP runtime for board operations.

## Repos

Prefer the installed `agentic-board` command. Do not assume machine-specific checkout paths. When repo paths are needed, read them from environment variables:

- App: `$env:AGENTIC_BOARD_APP_DIR`
- CLI: `$env:AGENTIC_BOARD_CLI_DIR`
- Integration tests: `$env:AGENTIC_BOARD_INTEGRATION_DIR`
- Project output root: `$env:AGENTIC_BOARD_PROJECT_DIR`

If `AGENTIC_BOARD_CLI_DIR` is unset, infer it only when the current working directory is inside a CLI checkout. If `AGENTIC_BOARD_APP_DIR` is unset, do not guess; ask the user for the tldraw app checkout or pass `agentic-board dev --app-dir <path>`.

Use shell-native environment syntax. PowerShell uses `$env:NAME = "value"`. Bash/zsh uses `export NAME="value"`.

## Startup

### Project-Local Board Isolation

Do not reuse a running default board for unrelated user projects. Each project should get its own local app URL and bridge port so agents do not write different project plans into the same board.

Before starting the board from a project workspace, confirm the app checkout and set a project-scoped port pair in the current shell. If the user already set `AGENTIC_BOARD_PORT` and `AGENTIC_BOARD_APP_URL`, keep those values.

```powershell
if (-not $env:AGENTIC_BOARD_APP_DIR) {
  throw "Set AGENTIC_BOARD_APP_DIR to the tldraw app checkout, or run agentic-board dev --app-dir <path>."
}

if (-not $env:AGENTIC_BOARD_PORT -or -not $env:AGENTIC_BOARD_APP_URL) {
  $sum = 0
  (Get-Location).Path.ToCharArray() | ForEach-Object { $sum += [int]$_ }
  $bridgePort = 5200 + ($sum % 300)
  $appPort = 5600 + ($sum % 300)
  $env:AGENTIC_BOARD_PORT = "$bridgePort"
  $env:AGENTIC_BOARD_APP_URL = "http://127.0.0.1:$appPort/"
}

if (-not $env:AGENTIC_BOARD_PROJECT_DIR) {
  $env:AGENTIC_BOARD_PROJECT_DIR = (Get-Location).Path
}
```

Bash/zsh equivalent:

```bash
test -n "$AGENTIC_BOARD_APP_DIR" || {
  echo "Set AGENTIC_BOARD_APP_DIR to the tldraw app checkout, or run agentic-board dev --app-dir <path>." >&2
  exit 1
}

if [ -z "$AGENTIC_BOARD_PORT" ] || [ -z "$AGENTIC_BOARD_APP_URL" ]; then
  sum=$(pwd | od -An -tu1 | awk '{ for (i = 1; i <= NF; i++) s += $i } END { print s + 0 }')
  bridge_port=$((5200 + (sum % 300)))
  app_port=$((5600 + (sum % 300)))
  export AGENTIC_BOARD_PORT="$bridge_port"
  export AGENTIC_BOARD_APP_URL="http://127.0.0.1:$app_port/"
fi

export AGENTIC_BOARD_PROJECT_DIR="${AGENTIC_BOARD_PROJECT_DIR:-$(pwd)}"
```

Then start with `agentic-board dev`. It passes the matching `VITE_BOARD_BRIDGE_URL` into the app automatically. After startup, run `agentic-board status` and verify the reported `bridgeUrl` and `boardAppUrl` match the current project shell. If not, stop and correct the environment before mutating the board.

Use the in-app `Stop` button or `agentic-board shutdown` when finished with a project board.

### Start Commands

Fast path:

```powershell
agentic-board dev
```

This starts the app from `AGENTIC_BOARD_APP_DIR`, starts the bridge, and opens the configured app URL. Use `agentic-board dev --app-dir <path>` when the app checkout is not in the environment. Use `dev --no-open` when another browser is already attached.

Manual path:

```powershell
$bridgeWs = "ws://127.0.0.1:$env:AGENTIC_BOARD_PORT/board-session"
$env:VITE_BOARD_BRIDGE_URL = $bridgeWs
npm --prefix $env:AGENTIC_BOARD_APP_DIR run dev -- --host 127.0.0.1 --port ([uri]$env:AGENTIC_BOARD_APP_URL).Port
```

Start or reuse the bridge:

```powershell
npm --prefix $env:AGENTIC_BOARD_CLI_DIR run build
agentic-board serve
```

Open the app configured by `AGENTIC_BOARD_APP_URL`:

```powershell
agentic-board open
```

The app connects to `ws://127.0.0.1:<AGENTIC_BOARD_PORT>/board-session`. The bridge also exposes `GET /capabilities`, `GET /session`, and `POST /shutdown` as HTTP shortcuts for connected boards.

Check readiness:

```powershell
agentic-board doctor
agentic-board list-capabilities
```

Proceed only when `doctor.result.ok` is true and the capability list includes `run_batch`, `create_shapes`, `create_arrows`, `render_template`, and `export_bounds`.

## Command Rules

- Use `--file` for substantial JSON payloads and `--json` for small one-off calls.
- Use `run_batch` for multi-step diagram creation so operations share one logical transaction.
- Prefer stable `shapeKey` values for SDK shapes and stable `itemId` values for workflow cards.
- Read current state before mutating when the task depends on existing board contents.
- Verify `agentic-board status` points at this project before every mutation if multiple projects are active.
- Keep long reasoning, logs, diffs, secrets, and raw stack traces out of board cards.
- Use browser/app exports for visual output; do not edit `.tldr` files offline in this workflow.
- Save board artifacts with `--out`; relative output paths are written under `AGENTIC_BOARD_PROJECT_DIR` when set, otherwise under the current working directory.
- If an example file path depends on `AGENTIC_BOARD_CLI_DIR` and that variable is unset, use inline `--json` or create a temporary JSON file in the current project instead of guessing a CLI path.

## Common Commands

Diagnostics:

```powershell
agentic-board status
agentic-board doctor
agentic-board get-session
agentic-board list-capabilities
agentic-board shutdown
```

Workflow cards:

```powershell
agentic-board read-index --json '{ "scope": "currentPage", "maxItems": 200 }'
agentic-board read-selection
agentic-board render-items --file "$env:AGENTIC_BOARD_CLI_DIR\examples\render-five.json"
agentic-board update-items --file "$env:AGENTIC_BOARD_CLI_DIR\examples\update-three.json"
agentic-board connect-items --file "$env:AGENTIC_BOARD_CLI_DIR\examples\connect-items.json"
```

SDK shapes and batches:

```powershell
agentic-board create-shapes --json '{ "shapes": [{ "type": "geo", "shapeKey": "demo:a", "x": 80, "y": 80, "text": "Demo" }] }'
agentic-board create-arrows --json '{ "arrows": [{ "shapeKey": "demo:arrow", "from": { "shapeKey": "demo:a" }, "end": { "x": 340, "y": 120 }, "label": "next" }] }'
agentic-board batch --file "$env:AGENTIC_BOARD_CLI_DIR\examples\sdk-batch.json"
agentic-board query-shapes --json '{ "text": "Demo", "maxItems": 50 }'
```

Templates and exports:

```powershell
agentic-board render-template --json '{ "templateId": "linear-flowchart", "instanceId": "demo:flow", "origin": { "x": 80, "y": 240 }, "replaceExisting": true }'
agentic-board render-scenario release-readiness --json '{ "replaceExisting": true }'
agentic-board export-bounds --json '{ "bounds": { "x": 40, "y": 40, "w": 800, "h": 400 }, "format": "png" }'
```

Save board artifacts:

```powershell
agentic-board save-snapshot --out .agentic-board/board-snapshot.json
agentic-board save-selection --out .agentic-board/selection.json
agentic-board export-page --out .agentic-board/page.png
agentic-board export-shapes --json '{ "shapes": [{ "shapeKey": "demo:a" }], "format": "png" }' --out .agentic-board/demo-a.png
agentic-board export-bounds --json '{ "bounds": { "x": 40, "y": 40, "w": 800, "h": 400 }, "format": "png" }' --out .agentic-board/region.png
agentic-board capture-region --json '{ "source": "viewport", "format": "png" }' --out .agentic-board/viewport.png
```

Use the same `agentic-board ... --out path` commands in Bash/zsh. Keep paths relative unless the user explicitly wants an absolute path.

Escape hatch:

```powershell
agentic-board call ping_editor
agentic-board call get_shapes --json '{ "scope": "currentPage", "maxItems": 100 }'
```

## SDK Command Surface

Run these before using uncommon SDK operations:

```powershell
agentic-board smoke
agentic-board list-capabilities
```

The CLI has typed wrappers for common commands, and every SDK-backed command is available through:

```powershell
agentic-board call <command_name> --json '{ }'
agentic-board batch --json '{ "atomic": true, "historyMark": "diagram", "operations": [{ "command": "<command_name>", "input": {} }] }'
```

Use command names exactly as reported by `agentic-board smoke` / `list-capabilities`.

Diagnostics and session:
`list_capabilities`, `get_session`, `get_pages`, `get_current_page`, `set_current_page`, `get_camera`, `set_camera`, `zoom_to_bounds`, `zoom_to_shapes`, `get_viewport`, `ping_editor`.

Read model:
`get_shapes`, `get_shape`, `get_shapes_by_key`, `get_bindings`, `get_assets`, `get_selection`, `get_current_tool`, `get_document_snapshot`, `get_store_snapshot`, `query_shapes`.

Workflow cards:
`read_board_index`, `read_selection`, `render_board_items`, `update_board_items`, `connect_board_items`, `capture_region`.

Shape creation and mutation:
`create_shapes`, `update_shapes`, `delete_shapes`, `duplicate_shapes`, `set_shape_meta`, `set_shape_props`.

Layout and geometry:
`move_shapes`, `resize_shapes`, `rotate_shapes`, `set_shape_bounds`, `align_shapes`, `distribute_shapes`, `stack_shapes`, `grid_layout`, `tree_layout`, `radial_layout`, `swimlane_layout`, `fit_frame_to_content`.

Ordering, grouping, and parenting:
`bring_forward`, `send_backward`, `bring_to_front`, `send_to_back`, `group_shapes`, `ungroup_shapes`, `reparent_shapes`, `create_frame_for_shapes`.

Arrows and bindings:
`create_arrows`, `update_arrows`, `bind_arrow`, `unbind_arrow`, `reconnect_arrow`, `validate_bindings`.

Text and rich text:
`create_text`, `update_text`, `update_rich_text`, `measure_text_shape`, `extract_text`.

Assets and media:
`create_asset`, `create_image_from_url`, `create_image_from_data_url`, `create_video_asset`, `place_asset`, `delete_unused_assets`.

Export, capture, and persistence:
`export_shapes`, `export_page`, `export_bounds`, `save_snapshot`, `load_snapshot`, `serialize_selection`.

Convenience save wrappers:
`save-snapshot`, `save-selection`, `export-page`, `export-shapes`, `export-bounds --out`, `capture-region --out`.

History and editor actions:
`undo`, `redo`, `mark_history_stop`, `clear_selection`, `select_shapes`, `set_current_tool`, `run_editor_action`.

Templates and transactions:
`render_template`, `run_batch`.

## Visual Verification

Use Playwright via the Playwright skill (/playwright or /skill:playwright) for live board checks. Verify that:

- The app page title is `tldraw agent`.
- The bridge reports `boardConnected: true`.
- Exports and captures return nonblank image data.
- Re-rendering stable IDs updates existing shapes instead of duplicating them.
- Arrows remain bound to intended endpoints after shape movement.
- Arrows don't overlap text directly. Prefer using elbow arrows if such issues arise.
- Shapes have adequate spacing and are positioned well within parent frames.

Run the full local gate when changing the bridge, CLI, templates, or skill instructions:

```powershell
npm --prefix $env:AGENTIC_BOARD_APP_DIR run build
npm --prefix $env:AGENTIC_BOARD_CLI_DIR run pack:check
npm --prefix $env:AGENTIC_BOARD_CLI_DIR test
.\scripts\stress-test.ps1
```

Run `.\scripts\stress-test.ps1` from `$env:AGENTIC_BOARD_INTEGRATION_DIR`.
