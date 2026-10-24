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

Expected `opencode mcp list`: `1c-testpilot`, `macos`, `agentmemory`, `context7`, `github`, `gsheets`, `opencode_console` connected; `1c-toolkit` failed with `ECONNREFUSED` unless `MCP_Toolkit.epf` is running.

## Open a base in the designer (GUI)

```bash
/Users/gz0ni/.local/1cv8/8.3.27.1606/1cv8 DESIGNER /F/Users/gz0ni/Documents/Work/Languages/1c/lab3
```

- lab1: no `/N` (no users in the base).
- lab2: `/N админ` (Cyrillic).
- lab3: `/N admin` (Latin).

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
