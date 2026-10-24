# TODO

Task checklist. The agent maintains this file: creates items, marks them done, keeps it honest. Work from here — do not lose tasks between sessions.

## Format

- `[ ]` pending task — add due date or note if needed
- `[x]` done task — keep for history, do not delete immediately
- `[~]` in progress — who/what is working on it

## Rules

- Add a task the moment it is discovered, even if vague.
- Mark `[x]` only when actually verified done (checks passed), not by intent.
- Review this file at the end of every session: close finished items, add follow-ups.

---

## Open

- [ ] Start `MCP_Toolkit.epf` inside 1C and confirm `1c-toolkit` connects on port 6003.
- [ ] Add the Windows-MCP server when the Windows machine is set up (deliberately not in this repo yet).
- [ ] Dump `lab1` configuration to `lab1/src` if it is going to be edited (no `src/` yet).
- [ ] Confirm the `macos` MCP `Type` tool can enter text into the 1C login dialog (only `Shortcut`/`Click` were exercised).
- [ ] Fill `lab4`–`lab8` when the coursework reaches them; add their PDFs to `labN/src/`.
- [ ] Decide whether to keep `lab1` in the registry without credentials or add a user to the base.

## In Progress

- [ ] (none)

## Done

- [x] Install 81 `cc-1c-skills` into `.opencode/skills` with manifest and `requirements.txt`.
- [x] Install `pipx`, `uv`, and Python deps `lxml` / `Pillow` / `psutil`.
- [x] Create the `python` shim at `~/.local/bin/python`.
- [x] Install `1c-testpilot` 1.9.1 and configure `macos-mcp`.
- [x] Write `.opencode/opencode.json` (V2 `mcp.servers`) without touching the global config.
- [x] Write `.opencode/1c-testpilot.profiles.yaml` for lab1–lab3.
- [x] Write `.v8-project.json` with `v8path`, three bases, default `lab3`.
- [x] Create platform symlinks `~/.local/1cv8/8.3.27.1606/1cv8` and `1cv8c`.
- [x] Verify `cf-info` on `lab3/src`.
- [x] Verify batch `DESIGNER /DumpConfigToFiles` for lab1, lab2, lab3.
- [x] Diagnose and fix the lab2/lab3 hang (credentials, empty password handling).
- [x] Verify Testpilot `launch_client` and `stop_client` for lab2 and lab3.
- [x] Verify the `db-dump-xml` skill end-to-end on lab3.
- [x] Fix Screen Recording permission for `screencapture`.
- [x] Create `.agents/` journals and root `AGENTS.md`.
- [x] Initialize the git repository and push to `git@github.com:gz0ni/1c-mcp-bundle.git` (`main`).
- [x] Add the methodichka rule (code only from the teacher's PDFs, listings are images).
- [x] Create `lab4`–`lab8` and keep all lab folders out of git.
