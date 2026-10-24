# Bugs

Known bugs found in this project and how they were fixed, so they do not reappear. Logged at the end of every session. Status: `open` / `fixed` / `won't fix`.

## Format

```
### <Short bug title> (YYYY-MM-DD) — status

- Symptom
- Root cause
- Fix applied
- How to prevent recurrence
```

---

<!-- New entries go here, newest first. -->

### lab2/lab3 hang in designer and test client (2026-10-24) — fixed

- Symptom: `DESIGNER /F.../lab2` and `launch_client(profile="lab3")` never returned; no window, ~1% CPU, `1Cv8.1CD` held open. A copy of lab3 in a temp directory behaved the same, so it was not a path or lock issue.
- Root cause: both bases require authentication, and 1C was sitting on an invisible login dialog. Credentials differ per base: lab3 `admin` (Latin), lab2 `админ` (Cyrillic), both empty passwords.
- Fix: pass `/N admin` / `/N админ` with no `/P`; added `user` / `password` to `.v8-project.json` and the Testpilot profiles.
- Prevent: check credentials before blaming the platform. See `.agents/gotchas.md` entries 1–3.

### `screencapture` returned "could not create image from display" (2026-10-24) — fixed

- Symptom: screen capture failed despite Screen Recording being granted to OpenCode.
- Root cause: TCC caches the permission decision for the lifetime of the responsible process; the grant happened after OpenCode had started.
- Fix: restarted OpenCode; capture now returns a valid 2940×1912 PNG.
- Prevent: after changing TCC permissions, fully quit and relaunch the app.

### Testpilot rejected the platform executable (2026-10-24) — fixed

- Symptom: `only 1cv8, 1cv8c may be launched`.
- Root cause: Testpilot validates the executable basename; the training build ships `1cv8t` / `1cv8ct`.
- Fix: symlinks in `~/.local/1cv8/8.3.27.1606/` with the required names.
- Prevent: always point configs at the symlinks.

### `1c-toolkit` MCP never connects (2026-10-24) — open

- Symptom: `opencode mcp list` → `1c-toolkit failed: ECONNREFUSED`.
- Root cause: the HTTP MCP server on port 6003 is provided by `MCP_Toolkit.epf`, which is not running.
- Fix applied: none; this is a manual step.
- Prevent: document the requirement; do not change the URL.

### `macos` MCP AppleScript mode fails with -25211 (2026-10-24) — won't fix

- Symptom: `Shell(mode="osascript")` → "Упрощенный доступ для osascript не разрешены (-25211)".
- Root cause: Accessibility is granted to the MCP server's uv-managed Python; `osascript` is a separate binary without its own grant.
- Fix applied: use native MCP tools (`Snapshot`, `Click`, `Type`, `Shortcut`) instead.
- Prevent: do not rely on AppleScript through the MCP server.

### `db-dump-xml.py` does not resolve the base from the registry (2026-10-24) — won't fix

- Symptom: running the script with only `-ConfigDir` fails with `specify -InfoBasePath or -InfoBaseServer + -InfoBaseRef`.
- Root cause: by design the script only reads `v8path` / `v8args` from `.v8-project.json`; base selection is the agent's job per `SKILL.md`.
- Fix applied: none; pass `-InfoBasePath` and `-UserName` explicitly.
- Prevent: read `.v8-project.json` first and build the arguments from it.
