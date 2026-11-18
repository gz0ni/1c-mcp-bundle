# Commands

All commands below were run on this machine and produced the stated result. Do not invent alternatives before checking these.

## Environment checks

```bash
# Platform launchers (symlinks, must exist)
ls -l ~/.local/1cv8/8.3.27.1606/     # 1cv8 -> /opt/1cv8t/.../1cv8t, 1cv8c -> /opt/1cv8t/.../1cv8ct

# Python shim used by all skill scripts
command -v python && python -V        # /Users/gz0ni/.local/bin/python, 3.13.16

# MCP servers
cd /Users/gz0ni/Documents/Work/Languages/1c && opencode mcp list
```

Expected `opencode mcp list`: `1c-testpilot`, `macos`, `context7`, `github`, `gsheets`, `opencode_console` connected; `agentmemory` connected only while its daemon runs (see `.agents/gotchas.md`); `1c-toolkit` failed with `ECONNREFUSED`. On this Mac that failure is permanent: `MCP_Toolkit.epf` serves the server through a Windows-only native add-in (see `.agents/gotchas.md`). `1c-toolkit` becomes usable only on the Windows host.

## MCP servers in Claude Code (Windows host)

Moved from OpenCode on 2026-11-04. Of the global OpenCode servers only `context7` and `1c-mcp`
went to the Claude Code user scope (`~/.claude.json`); `gsheets`, `github`, `agentmemory` and
`blockbench` were dropped at the user's request. The project server is in
`D:\Work\1C\.mcp.json` (gitignored; template
`.mcp.example.json`). `1c-testpilot` is installed with `uv tool install
"git+https://github.com/ROCTUP/1c-testpilot"` → `C:\Users\user\.local\bin\1c-testpilot.exe`;
profiles live in `.opencode/1c-testpilot.profiles.yaml` (lab1–lab8, test).

Each client keeps its own config since 2026-11-05 (all in `D:\Work\1C`, all gitignored):

| Client | Config | Schema |
|---|---|---|
| Claude Code | `.mcp.json` + `.claude/settings.json` (`enabledMcpjsonServers`) | `mcpServers`, stdio `command`/`env` |
| OpenCode | `.opencode/opencode.json` | `mcp.servers`, `type: "local"`, `command` array, `environment` |
| Codex | `.codex/config.toml` | `[mcp_servers.<name>]`, `command`/`args`/`env` |

Claude Code has no `.cloudcode` notion — it only reads `.mcp.json`, `~/.claude.json`, or a
`--mcp-config` path.

```bash
cd /d/Work/1C && claude mcp list
cd /d/Work/1C && opencode mcp list
```

Expected: `1c-testpilot` connected in both; `1c-mcp` (port 6003) `ECONNREFUSED` until
`MCP_Toolkit.epf` is running its server. Verified 2026-11-05: both clients report
`1c-testpilot connected`, the stdio handshake returns 11 `tc_*` tools (`tc_session`, `tc_app`,
`tc_window`, `tc_field`, `tc_doc`, `tc_table`, `tc_calendar`, `tc_keyboard`, `tc_form`,
`tc_scenario`, `tc_find`). Codex CLI is not installed on this host, so its config is unverified.

## Open the MCP Toolkit and start its server (Windows only)

```bash
cd /Users/gz0ni/Documents/Work/Languages/1c
~/.local/1cv8/8.3.27.1606/1cv8c ENTERPRISE /F$PWD/lab1 /Execute "$PWD/tools/MCP_Toolkit.epf" -DisableStartupDialogs
```

Then click "Запустить сервер" in the form (`Порт встроенного сервера: 6003`, `Запуск...`). Check with
`lsof -nP -iTCP:6003 -sTCP:LISTEN` and `curl -m 3 http://127.0.0.1:6003/mcp`.

macOS result (2026-11-02): the form opens, both security dialogs have to be accepted with Да, and the
button then logs `Не удалось подключить внешнюю компоненту MCPHttp` — the add-in is a Windows PE DLL.
Do not debug the port on macOS.

## Open a base in the designer (GUI)

```bash
/Users/gz0ni/.local/1cv8/8.3.27.1606/1cv8 DESIGNER /F/Users/gz0ni/Documents/Work/Languages/1c/lab3
```

- lab1: no `/N` (no users in the base).
- lab2: `/N админ` (Cyrillic).
- lab3: `/N admin` (Latin).
- lab4 (Windows, 2026-11-11): `/N admin` (Latin), empty password omitted. Use explicit `-InfoBasePath D:/Work/1C/lab4` and `-V8Path` resolved from the installed 8.3.27.1508 platform. The XML dump is `lab4/src`; verified final export is `lab4/lab4-variant16.dt`.

Never pass `/P ""` — see `.agents/gotchas.md`.

## Batch mode (no GUI)

```bash
cd /Users/gz0ni/Documents/Work/Languages/1c

# lab1
/Users/gz0ni/.local/1cv8/8.3.27.1606/1cv8 DESIGNER /F$PWD/lab1 \
  /DumpConfigToFiles /tmp/dump_lab1 -DisableStartupDialogs

# lab2
/Users/gz0ni/.local/1cv8/8.3.27.1606/1cv8 DESIGNER /F$PWD/lab2 /N админ \
  /DumpConfigToFiles /tmp/dump_lab2 -DisableStartupDialogs

# lab3
/Users/gz0ni/.local/1cv8/8.3.27.1606/1cv8 DESIGNER /F$PWD/lab3 /N admin \
  /DumpConfigToFiles /tmp/dump_lab3 -DisableStartupDialogs
```

All three exit `0` and write `Configuration.xml` + `ConfigDumpInfo.xml`.

## Skills (Python ports)

```bash
cd /Users/gz0ni/Documents/Work/Languages/1c

# Read configuration summary from XML sources
python .opencode/skills/cf-info/scripts/cf-info.py -ConfigPath lab3/src

# Dump configuration from a base to XML (skill resolves v8path from .v8-project.json)
python .opencode/skills/db-dump-xml/scripts/db-dump-xml.py \
  -InfoBasePath "$PWD/lab3" -UserName admin -ConfigDir /tmp/skill_dump_lab3
```

Note: `db-dump-xml.py` does **not** resolve the base from the registry by itself — the agent must read `.v8-project.json` and pass `-InfoBasePath` / `-UserName`. The script only auto-reads `v8path` and `v8args`.

## Testpilot (MCP tools)

```text
tc_session(action="list_profiles")
tc_session(action="launch_client", profile="lab3", wait=90, connect=True)
tc_session(action="stop_client", connection_id="<id>")
```

Profiles live in `.opencode/1c-testpilot.profiles.yaml`.

## Reading methodichkas (PDF)

Code in the teacher's PDFs is embedded as images — text extraction returns prose only. Render pages and read them with vision.

```bash
cd /Users/gz0ni/Documents/Work/Languages/1c

# page count and image inventory
pdfinfo "lab3/src/Лабораторная работа №3 (2022-2023).pdf"
pdfimages -list "lab3/src/Лабораторная работа №3 (2022-2023).pdf"

# confirm the listings are not text (expect 0)
pdftotext "lab3/src/Лабораторная работа №3 (2022-2023).pdf" - | grep -cE "Процедура|Функция"

# render a page to PNG for vision (200 dpi is enough to read code)
mkdir -p /tmp/methodichka
pdftoppm -r 200 -png -f 12 -l 12 \
  "lab3/src/Лабораторная работа №3 (2022-2023).pdf" /tmp/methodichka/lab3

# or extract the embedded screenshots directly
pdfimages -png "lab3/src/Лабораторная работа №3 (2022-2023).pdf" /tmp/methodichka/img
```

Then open the produced PNG with the `read` tool. Never retype code from memory.

## Screenshots and UI automation (macOS)

```bash
screencapture -x -t png /path/to/shot.png    # works only if OpenCode has Screen Recording
```

```text
macos.Snapshot(use_vision=False)   # AX tree + coordinates
macos.Click(loc=[x, y])
macos.Type(loc=[x, y], text="...", press_enter=True)
macos.Shortcut(shortcut="return")
```

Coordinates are logical points (screen is 1470×956 logical on this Mac).

## Desktop automation (Windows)

Use the built-in Codex computer-use tool for desktop input.

## Verification

Run before claiming work is done:

```bash
cd /Users/gz0ni/Documents/Work/Languages/1c
opencode mcp list
python .opencode/skills/cf-info/scripts/cf-info.py -ConfigPath lab3/src
```

## CI

None. The repository has no workflows.

## Tests

None. Skills are verified by running them against `lab1`–`lab3`; there is no test suite.

## Lab5 on Windows (2026-11-11)

- Base: `D:/Work/1C/lab5`, user `admin` (Latin), empty password omitted; platform 8.3.27.1508.
- Use the verified Python db-* skills with explicit `-InfoBasePath`, `-UserName`, `-V8Path`.
- Native `/CheckModules`: Start-Process -Wait -PassThru, separate quoted `/Out` argument; expect exit0 and `Синтаксических ошибок не обнаружено!`.
- Sources: `lab5/src`; backup: `lab5/evidence/before-lab5.dt`; final: `lab5/lab5-variant16.dt`.
- Scope check: bundled Python `-X utf8 lab5/evidence/verify-scope.py`. Behavior-test source and instructions: `lab5/evidence/verification.md`.
