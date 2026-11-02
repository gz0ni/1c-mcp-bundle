# 1c-mcp-bundle

Agent toolchain for 1C:Enterprise coursework on macOS: the `cc-1c-skills` skill set, MCP servers for a live client and desktop automation, and a knowledge base of traps already stepped on.

The repository is a **toolchain bundle**, not a coursework archive. Infobases (`lab1`–`lab8`, `test`) stay local and are gitignored.

## What is inside

| Path | Purpose |
|---|---|
| `AGENTS.md` | Root entry point for coding agents |
| `.agents/` | Project knowledge base: gotchas, decisions, bugs, commands, architecture |
| `.agents/skills/` | Upstream skills copied in: `agent-desktop`, `agent-desktop-windows` |
| `.opencode/skills/` | 81 skills from `cc-1c-skills` (Python ports, branch `port-opencode-py`) |
| `.opencode/opencode.example.json` | MCP servers: `1c-testpilot`, `1c-toolkit` |
| `.opencode/1c-testpilot.profiles.example.yaml` | Launch profiles for the test client |
| `.v8-project.example.json` | Database registry consumed by the `db-*` skills |
| `package.json` | Project-local npm tooling (`agent-desktop`) |

## Rule: code comes from the methodichkas

The teacher's PDFs (`labN/src/*.pdf`) are the baseline for code, object names, and step order. Listings in those PDFs are **images**, not text, so pages must be rendered and read with vision:

```bash
pdftoppm -r 200 -png -f 12 -l 12 "lab3/src/Лабораторная работа №3 (2022-2023).pdf" /tmp/page
```

Allowed: renames and literal changes required by the variant, plus small fixes to make the printed code run.

Forbidden: adding functionality, objects, attributes, or algorithms that the methodichka does not describe, and restructuring a working solution. If a step is missing or unreadable, ask instead of guessing. See `.agents/rules.md`.

## Prerequisites

- macOS (Apple Silicon), 1C:Enterprise 8.3.27.1606 training build in `/opt/1cv8t`.
- Python 3.13 with `lxml`, `Pillow`, `psutil`.
- `python` must resolve to Python 3 — the skill scripts call it by that name.
- `pipx` (Testpilot) and `uv`/`uvx` (macos-mcp).
- Poppler (`pdftoppm`, `pdfinfo`, `pdfimages`) for reading methodichkas.
- Node.js 20+ and npm, for the Windows desktop-automation CLI.

## Setup

```bash
# 1. Skills are committed; dependencies are not
python3 -m pip install --break-system-packages -r .opencode/skills/requirements.txt

# 2. Stable launcher names (Testpilot only accepts 1cv8 / 1cv8c)
mkdir -p ~/.local/1cv8/8.3.27.1606
ln -sf /opt/1cv8t/8.3.27.1606/1cv8t  ~/.local/1cv8/8.3.27.1606/1cv8
ln -sf /opt/1cv8t/8.3.27.1606/1cv8ct ~/.local/1cv8/8.3.27.1606/1cv8c

# 3. Testpilot
pipx install --python /opt/homebrew/bin/python3.13 git+https://github.com/ROCTUP/1c-testpilot.git

# 4. Local configs from examples
cp .v8-project.example.json .v8-project.json
cp .opencode/opencode.example.json .opencode/opencode.json
cp .opencode/1c-testpilot.profiles.example.yaml .opencode/1c-testpilot.profiles.yaml
# then replace /absolute/path/to/repo and /Users/<you>

# 5. Check
opencode mcp list

# 6. Desktop automation CLI (Windows host; on macOS the `macos` MCP covers it)
npm install
npm run setup   # links the native binary; npm blocks the package's own postinstall
.\node_modules\.bin\agent-desktop.cmd status
```

`1c-toolkit` will report `ECONNREFUSED` until `MCP_Toolkit.epf` is opened in 1C and its built-in server starts on port 6003. That is expected.

## Local configuration (not in git)

- `.v8-project.json` — absolute paths and base credentials.
- `.opencode/opencode.json` — machine-specific paths and the `macos` MCP server.
- `.opencode/1c-testpilot.profiles.yaml` — base credentials.

The `macos` MCP server is intentionally **not** part of this repository; it lives only in the local `opencode.json`. On a Windows host, desktop input comes from the project-local `agent-desktop` npm package instead (`node_modules/` is gitignored, `package.json` and the lockfiles are versioned).

## Known traps

Read [`.agents/gotchas.md`](.agents/gotchas.md) before working. The most expensive ones:

- Bases require authentication: lab1 has no users, lab2 uses `админ` (Cyrillic), lab3 uses `admin` (Latin); passwords are empty.
- An empty password must be omitted, never passed as `/P ""` — the platform hangs on a re-prompt.
- The 1C login dialog is invisible to the macOS Accessibility tree; use a screenshot.
- `/Applications/1cv8t.localized/*.app` are stubs that exit with code 255.
