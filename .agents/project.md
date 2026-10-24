# Project Context

Workspace for three 1C:Enterprise training databases (`lab1`–`lab3`) plus the local agent toolchain that works with them: 81 `cc-1c-skills` skills, three MCP servers, and a database registry consumed by the skills.

The point of the toolchain is to let an agent read and modify 1C configurations from XML sources and drive a live client, without a Windows machine.

## Platforms

- Host: macOS (darwin), Apple Silicon (arm64).
- 1C:Enterprise 8.3.27.1606 **training build**, x86_64 binaries running under Rosetta 2.
- No Windows and no `pwsh`: the PowerShell ports of the skills are unusable here, only the Python ports work.
- Client UI is macOS-native (wxWidgets 3.0).

## Toolchain

- 1C platform binaries: `/opt/1cv8t/8.3.27.1606/` — `1cv8t` (thick client + designer), `1cv8ct` (thin client), `1cv8st` (server).
- Stable launcher paths: `~/.local/1cv8/8.3.27.1606/1cv8` → `1cv8t`, `1cv8c` → `1cv8ct` (symlinks; required because Testpilot only accepts executables literally named `1cv8` / `1cv8c`).
- Python 3.13.16 (Homebrew). Shim `~/.local/bin/python` → `/opt/homebrew/bin/python3.13`, because every skill invokes `python` from `PATH`.
- `pipx` 1.18.4 — installed `1c-testpilot` 1.9.1 from git (`ROCTUP/1c-testpilot`).
- `uv`/`uvx` 0.13.5 — runs `macos-mcp`.
- Python packages installed globally with `--break-system-packages`: `lxml` 6.1.3, `Pillow` 13.0.0, `psutil` 7.2.2.
- OpenCode desktop 2.1.4 with `opencode-cli` 2.1.4.

## Package / Repository

- `name:` workspace folder `1c`, not a package.
- `version:` n/a.
- `org:` personal coursework.
- Git repository: `git@github.com:gz0ni/1c-mcp-bundle.git`, branch `main`. The repository holds the toolchain only; `lab1`–`lab8` and `test` are gitignored.

## Dependencies

| Dependency | Why |
|---|---|
| 1C 8.3.27.1606 training build | Runs the bases; no `ibcmd`, designer batch mode only |
| Python 3.13 + lxml/Pillow/psutil | Skill scripts parse XML, images, processes |
| 1c-testpilot | MCP server driving a live `/TESTCLIENT` |
| macos-mcp | Accessibility tree, clicks, typing, screenshots on macOS |
| 1c-mcp-toolkit | Remote MCP at `http://127.0.0.1:6003/mcp`, served by `MCP_Toolkit.epf` inside 1C |

## Design Basis

No UI/UX design work here. The "design" is the 1C configuration itself; XML dumps in `lab*/src` are the source of truth for configuration objects.

The teacher's methodical instructions live next to them: `labN/src/*.pdf`. They are the only authorized source of code and step order. Listings in those PDFs are images, not text — render pages with `pdftoppm` and read them with vision (see `.agents/commands.md`). The PDFs are not versioned because `lab*/` is gitignored.

## Status

Done:

- Skills installed and verified (`cf-info` on `lab3/src`, `db-dump-xml` on `lab3`).
- Testpilot connects to all three bases and can launch/stop a thin client.
- Batch designer mode verified for all three bases.
- Registry `.v8-project.json` filled with platform path, bases, and credentials.

Not done:

- `1c-toolkit` never connected — needs `MCP_Toolkit.epf` running inside 1C on port 6003.
- `macos` MCP AppleScript mode is blocked by TCC (`osascript` has no Accessibility).
