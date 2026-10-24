# Gotchas

Traps and pitfalls the agent has stepped on in this project. Log a gotcha immediately whenever something does not build, run, or pass — right after the incident, while context is fresh, do not wait for session end. Check this file before repeating similar work. One entry per trap.

## Format

```
### <Short trap title> (YYYY-MM-DD)

- What happened / what was wrong
- Root cause
- How to avoid next time
```

---

<!-- New entries go here, newest first. -->

### Code in the methodichkas is images, not text (2026-10-24)

- `pdftotext` on `lab3/src/Лабораторная работа №3 (2022-2023).pdf` returns 89 KB of prose but zero `Процедура` / `Функция` matches, while `pdfimages -list` shows 16 embedded images.
- Root cause: the teacher exported listings as screenshots ("Microsoft: Print To PDF"), so the code exists only as pixels.
- Avoid: render the page with `pdftoppm -r 200 -png -f N -l N` and read the PNG with vision. Never reconstruct code from memory or from general 1C knowledge.

### Methodichka is the only allowed source of code (2026-10-24)

- Any code that is not literally in the teacher's PDF is treated as unauthorized initiative and penalized, even if correct.
- Root cause: course policy, not a technical constraint.
- Avoid: transcribe exactly, do not rename or add attributes, and stop to ask when a step is missing. See `.agents/rules.md`.

### `1Cv8Log` does not reveal base users (2026-10-24)

- `strings` on `1Cv8.1CD` and the event log found no usernames or passwords, so credentials could not be recovered from the files.
- Root cause: 1C stores password hashes and the log records authentication events without the login name in readable form.
- Avoid: do not dig through the base files for credentials. Open the base in the GUI and read the login dialog; the user list is shown in the dropdown.

### 1C hangs on login when the base has users (2026-10-24)

- `DESIGNER /F.../lab3 /DumpConfigToFiles` and `launch_client` hung for minutes with no output and no visible window. Process stayed alive at ~1% CPU, holding `1Cv8.1CD`.
- Root cause: the base requires authentication. 1C opens a login dialog and waits forever; batch mode does not fail, it blocks.
- Avoid: check credentials first. lab3 → user `admin` (Latin), lab2 → user `админ` (Cyrillic), both with an empty password. lab1 has no users at all.

### Empty password must be omitted, not passed as `/P ""` (2026-10-24)

- `DESIGNER /N admin /P "" /DumpConfigToFiles` hung on the login dialog, while `/N admin` alone worked.
- Root cause: an empty `/P` value is treated as "password not supplied", so the platform re-prompts. A wrong non-empty password also blocks on an error dialog instead of exiting.
- Avoid: never emit `/P` when the password is empty. Pass only `/N <user>`. The skill scripts already do this.

### 1C login dialog is invisible to the Accessibility tree (2026-10-24)

- `macos.Snapshot` listed no 1C window at all, even when the dialog was on screen and frontmost.
- Root cause: the app is launched from a bare binary, not a `.app` bundle, so LaunchServices does not register it (`lsappinfo info <pid>` shows `bundle path = [NULL]`, `Arch = !!none`). AX exposes only registered apps.
- Avoid: do not trust AX to find 1C dialogs. Take a screenshot (`screencapture -x`) and read it.

### Screen Recording permission is cached until the app restarts (2026-10-24)

- `screencapture` failed with `could not create image from display` even though Screen Recording was granted.
- Root cause: TCC attributes the request to the responsible process (`/Applications/OpenCode.app`, not `/usr/sbin/screencapture`) and caches the decision for the lifetime of that process. A grant made while OpenCode is running does not apply.
- Avoid: grant Screen Recording to `OpenCode.app` and fully restart OpenCode. The macOS MCP server is a separate process with its own grants.

### Testpilot refuses executables not named `1cv8` / `1cv8c` (2026-10-24)

- `launch_client(exe="/opt/1cv8t/8.3.27.1606/1cv8ct")` failed with `only 1cv8, 1cv8c may be launched`.
- Root cause: hard-coded filename validation in Testpilot.
- Avoid: use the symlinks `~/.local/1cv8/8.3.27.1606/1cv8` → `1cv8t` and `1cv8c` → `1cv8ct`.

### `/Applications/1cv8t.localized` contains stubs, not the platform (2026-10-24)

- Binaries under `/Applications/1cv8t.localized/8.3.27.1606/*.app/Contents/MacOS/` are ~52 KB and exit with code 255. Bundles are 948 KB total.
- Root cause: the training build keeps real binaries in `/opt/1cv8t/8.3.27.1606/`; the `.app` wrappers only locate them and fail outside their expected layout.
- Avoid: always launch `/opt/1cv8t/8.3.27.1606/1cv8t` or the `~/.local/1cv8/...` symlinks.

### `1c-toolkit` fails with ECONNREFUSED until the EPF is running (2026-10-24)

- `opencode mcp list` reports `1c-toolkit failed: ECONNREFUSED` on every start.
- Root cause: the remote MCP server at `http://127.0.0.1:6003/mcp` is served by `MCP_Toolkit.epf` inside a running 1C session. Nothing listens on 6003 by default.
- Avoid: treat this as expected until the user opens the EPF and starts the built-in server. Do not "fix" the URL.

### Skill auto-detection does not find this platform (2026-10-24)

- Skills auto-detect 1C via `glob('/opt/1cv8/*/1cv8')`; this machine has `/opt/1cv8t`.
- Root cause: the port assumes the standard Linux layout.
- Avoid: keep `v8path` in `.v8-project.json` and pass `-V8Path` when a script ignores the registry.

### Skill scripts invoke `python`, not `python3` (2026-10-24)

- Scripts failed with `command not found: python` on a clean macOS.
- Root cause: macOS ships no `python` alias.
- Avoid: the shim `~/.local/bin/python` → `/opt/homebrew/bin/python3.13` must stay in `PATH`.

### Testpilot macOS limitations (2026-10-24)

- `get_screenshot` and `desktop="isolated"` are unsupported on macOS; `tc_keyboard` works only on Windows/Linux.
- Root cause: platform support in Testpilot 1.9.1.
- Avoid: use `macos` MCP for input and screenshots, Testpilot only for the 1C protocol.

### `macos` MCP AppleScript mode has no Accessibility (2026-10-24)

- `Shell(mode="osascript")` fails with `-25211` ("Упрощенный доступ для osascript не разрешены").
- Root cause: the TCC grant belongs to the MCP server's uv-managed Python, not to `osascript`, and child processes do not inherit it.
- Avoid: use native tools (`Snapshot`, `Click`, `Type`, `Shortcut`); they work.

### Do not launch `1cv8st` / `1cv8ct` as clients (2026-10-24)

- `1cv8st` is the server, `1cv8ct` is the thin client; the designer is `1cv8t`. Running the wrong one produces exit 255 or a process that never connects.
- Root cause: confusing training-build suffixes (`t` = training, `c` = client, `s` = server).
- Avoid: `1cv8t` for designer/thick, `1cv8ct` for thin client, `1cv8st` only for a real server.

### `timeout` is not installed on macOS (2026-10-24)

- `timeout 120 python ...` → `command not found: timeout`.
- Root cause: GNU coreutils are not installed by default.
- Avoid: use `gtimeout` (coreutils) or a manual `for` loop with `kill -0`.
