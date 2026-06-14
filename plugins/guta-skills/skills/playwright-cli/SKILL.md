---
name: playwright-cli
description: Drive a real browser from the shell to navigate, click, fill, snapshot, screenshot, mock network, or run/debug Playwright tests. Use whenever the user mentions playwright, browser automation, end-to-end tests, scraping a live page, login flows, cookie/localStorage manipulation, taking page screenshots, recording video, tracing, or reviewing UI in a real browser — even when they don't name Playwright explicitly.
allowed-tools: Bash(playwright-cli:*) Bash(npx:*) Bash(npm:*)
---

# playwright-cli

Headless or headed browser automation via the `playwright-cli` command. Each command operates on the current session and returns a YAML snapshot of the page with stable `eN` refs (e1, e2, …) you reuse for follow-up actions. Always snapshot before acting, then act using refs.

## Install / detect

```bash
# Prefer global, fall back to npx
playwright-cli --version || npx --no-install playwright-cli --version
# If neither works
npm install -g @playwright/cli@latest
```

Substitute `npx playwright-cli` for `playwright-cli` everywhere below if only the local version resolves.

## Core loop

```bash
playwright-cli open https://example.com    # launches browser + navigates
playwright-cli snapshot                     # YAML tree with eN refs
playwright-cli click e3                     # act on a ref
playwright-cli fill e5 "user@example.com" --submit
playwright-cli snapshot                     # confirm new state
playwright-cli close
```

Target elements three ways: snapshot ref (`e3`), CSS (`"#main > button.submit"`), or Playwright locator (`"getByRole('button', { name: 'Submit' })"`, `"getByTestId('submit')"`).

## Commands

### Interaction
```bash
playwright-cli click e3
playwright-cli dblclick e7
playwright-cli fill e5 "value" [--submit]   # --submit presses Enter
playwright-cli type "free text"             # types into focused element
playwright-cli press Enter | ArrowDown | Tab
playwright-cli hover e4
playwright-cli select e9 "option-value"
playwright-cli check e12  |  uncheck e12
playwright-cli drag e2 e8
playwright-cli drop e4 --path=./file.png    # or --data="text/plain=hello"
playwright-cli upload ./document.pdf
playwright-cli dialog-accept ["text"]  |  dialog-dismiss
playwright-cli resize 1920 1080
```

### Navigation & tabs
```bash
playwright-cli goto https://...
playwright-cli go-back | go-forward | reload
playwright-cli tab-list | tab-new [url] | tab-select <i> | tab-close [i]
```

### Inspect
```bash
playwright-cli snapshot [selector] [--depth=4] [--boxes] [--filename=x.yaml]
playwright-cli eval "document.title"
playwright-cli eval "el => el.getAttribute('data-testid')" e5
playwright-cli screenshot [e5] [--filename=page.png]
playwright-cli pdf --filename=page.pdf
playwright-cli console [warning|error]
playwright-cli requests  |  request <index>
playwright-cli generate-locator e5 [--raw]
playwright-cli highlight e5 [--style="outline:3px dashed red"] [--hide]
```

### Storage
```bash
playwright-cli state-save [auth.json]  |  state-load auth.json
playwright-cli cookie-{list,get,set,delete,clear} [args]
playwright-cli localstorage-{list,get,set,delete,clear} [args]
playwright-cli sessionstorage-{list,get,set,delete,clear} [args]
```

### Network mocking
```bash
playwright-cli route "**/*.jpg" --status=404
playwright-cli route "https://api.example.com/**" --body='{"mock":true}'
playwright-cli route-list  |  unroute [pattern]
```

### Tracing / video / scripts
```bash
playwright-cli tracing-start  |  tracing-stop
playwright-cli video-start out.webm  |  video-stop
playwright-cli video-chapter "Title" --description="..." --duration=2000
playwright-cli run-code "async page => page.context().grantPermissions(['geolocation'])"
playwright-cli run-code --filename=script.js
```

## Sessions & launch options

```bash
# Named sessions (run several browsers in parallel)
playwright-cli -s=work open https://...
playwright-cli -s=work click e1
playwright-cli -s=work close
playwright-cli list  |  close-all  |  kill-all

# Browser / profile / connection
playwright-cli open --browser=chrome|firefox|webkit|msedge
playwright-cli open --persistent             # keeps profile across runs
playwright-cli open --profile=/path/to/dir   # explicit profile dir
playwright-cli attach --cdp=chrome|msedge|http://localhost:9222
playwright-cli attach --extension=chrome
playwright-cli -s=work detach                # leaves external browser running
playwright-cli delete-data                   # wipe persistent profile
```

## Output modes

```bash
playwright-cli --raw eval "JSON.stringify(performance.timing)" | jq .
playwright-cli --raw snapshot > before.yml
playwright-cli --raw cookie-get session_id
playwright-cli list --json
```

`--raw` strips status/snapshot framing so the value is pipeable. `--json` wraps the whole reply as JSON.

## Playwright tests

```bash
# Run tests (delegates to local @playwright/test)
npx playwright test
npx playwright test path/to/file.spec.ts --headed
npx playwright test -g "login"               # filter by title
npx playwright show-report                   # last HTML report
npx playwright codegen https://example.com   # record interactions into test code
```

When a test fails, ask for the traces directory in the report (`test-results/<name>/trace.zip`) and open it with `npx playwright show-trace <path>`.

## Recipes

**Login + save auth, reuse later**
```bash
playwright-cli open https://app.example.com/login
playwright-cli snapshot
playwright-cli fill e2 "user@example.com"
playwright-cli fill e3 "secret" --submit
playwright-cli state-save auth.json
playwright-cli close

playwright-cli open https://app.example.com --persistent
playwright-cli state-load auth.json
```

**Diff page state across an action**
```bash
playwright-cli --raw snapshot > before.yml
playwright-cli click e5
playwright-cli --raw snapshot > after.yml
diff before.yml after.yml
```

**Mock an API while exercising the UI**
```bash
playwright-cli open https://app.example.com
playwright-cli route "https://api.example.com/users/me" --body='{"id":1,"name":"Test"}'
playwright-cli reload
playwright-cli snapshot
```

## Conventions

- Snapshot before interacting — refs are only valid against the latest snapshot.
- Prefer refs over CSS; fall back to `getByRole` / `getByTestId` locators when refs are unstable (popups, virtualized lists).
- Close or `close-all` when finished to free the browser process; `kill-all` if a session is wedged.
- For attributes not visible in the snapshot (`id`, `data-*`, computed style) use `eval "el => el.attr" eN`.
- For UI/design feedback from the user, `playwright-cli show --annotate` opens an annotation overlay and returns their boxes + notes.

## Discovery

`playwright-cli --help` and `playwright-cli <command> --help` list every flag and command; rely on them when something here is missing rather than guessing.

## Deep dives

Read the matching file only when the user's task lands in that area — do not preload them.

- `references/playwright-tests.md` — running, filtering, and debugging Playwright tests; reading traces and reports.
- `references/test-generation.md` — generating new tests from interactions (codegen) and from snapshots.
- `references/spec-driven-testing.md` — plan / generate / heal loop for spec-driven test authoring.
- `references/request-mocking.md` — `route`/`unroute` patterns, response bodies, status codes, dynamic mocks.
- `references/running-code.md` — `eval`, `run-code`, and inline Playwright scripts against the live session.
- `references/session-management.md` — named sessions, persistent profiles, CDP attach, detach, cleanup.
- `references/storage-state.md` — cookies, localStorage, sessionStorage, `state-save`/`state-load` reuse.
- `references/tracing.md` — `tracing-start/stop`, viewing traces, capturing artifacts on failure.
- `references/video-recording.md` — `video-start/stop`, chapters, integration with traces.
- `references/element-attributes.md` — extracting `id`, `data-*`, computed style, and other attrs via `eval`.
