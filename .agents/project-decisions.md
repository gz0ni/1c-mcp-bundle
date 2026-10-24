# Project Decisions

Recorded decisions and the reasoning behind them, so future work does not silently contradict them. Add an entry whenever a non-trivial choice is made. Latest first.

## Format

```
### <Decision title> (YYYY-MM-DD)

- Decision: <what was chosen, one sentence>
- Reason: <why this and not the alternatives>
- Alternatives considered: <what was rejected and why>
- Consequences: <what this decision affects>
```

---

<!-- New entries go here, newest first. -->

### Methodichka is the baseline; variant-driven edits are allowed (2026-10-24)

- Decision: start from the code printed in the teacher's PDF; permit only changes the variant requires (renames of procedures/functions/variables, literals, small fixes to make it run).
- Reason: the teacher penalizes unauthorized additions, but the variant itself requires adaptation. Pure copying would fail the assignment.
- Alternatives considered: exact transcription only (rejected — breaks variant requirements), free-form idiomatic 1C (rejected — penalized).
- Consequences: every implementation task starts by reading the relevant PDF page as an image; deviations are justified by the variant, never by taste. Missing steps are escalated to the user.

### Repository is a toolchain bundle, not a coursework archive (2026-10-24)

- Decision: `lab1`–`lab8` and `test` are gitignored entirely; only `.agents/`, `.opencode/skills`, configs, and docs are versioned.
- Reason: the user asked not to commit the infobases; they are large binary files that change constantly.
- Alternatives considered: committing `lab*/src` XML dumps (rejected by the user), committing `.dt` archives (large, opaque).
- Consequences: methodichka PDFs, which live in `labN/src/`, are not in the repository either. They stay local.

### Machine-specific configs are shipped as examples (2026-10-24)

- Decision: commit `*.example.*` files; keep `.v8-project.json`, `.opencode/opencode.json`, and `.opencode/1c-testpilot.profiles.yaml` out of git.
- Reason: they contain absolute paths and credentials, and the OpenCode config carries the `macos` MCP server, which must not be published.
- Alternatives considered: committing them with placeholders (leaks credentials on every edit), environment variables only (OpenCode does not merge partial configs).
- Consequences: a fresh clone needs three files copied from the examples and filled in.

### Credentials live in the registry and the profiles file (2026-10-24)

- Decision: store `user` / `password` per database in both `.v8-project.json` and `.opencode/1c-testpilot.profiles.yaml`.
- Reason: every batch operation and every client launch needs them; duplicating two fields is cheaper than remembering them per call.
- Alternatives considered: passing `/N` on every command (error-prone, caused hours of hanging), environment variables (Testpilot supports `password_env`, but the bases use empty passwords).
- Consequences: both files are secrets-bearing. Do not commit them if this ever becomes a git repo; add them to `.gitignore`.

### All three MCP servers enabled at once (2026-10-24)

- Decision: `1c-testpilot`, `1c-toolkit`, and `macos` are all declared in `.opencode/opencode.json`.
- Reason: the user asked for all three; a failing remote server does not block the others (`opencode mcp list` shows it as failed and continues).
- Alternatives considered: enabling `1c-toolkit` later (adds a manual step every session).
- Consequences: `opencode mcp list` always shows one red line until the EPF runs. That is expected.

### Project-scoped OpenCode config instead of editing the global one (2026-10-24)

- Decision: create `.opencode/opencode.json` using the V2 `mcp.servers` schema; leave `~/.config/opencode/opencode.json` untouched.
- Reason: the toolchain belongs to this workspace; global config is shared with other projects and uses the older V1 shape.
- Alternatives considered: appending to the global file (risks breaking unrelated projects), per-server CLI flags (not persistent).
- Consequences: the servers load only when OpenCode runs from this directory. Verified that global V1 servers still load alongside.

### Testpilot installed with pipx, macos-mcp run through uvx (2026-10-24)

- Decision: `pipx install --python /opt/homebrew/bin/python3.13 git+https://github.com/ROCTUP/1c-testpilot.git` and `uvx macos-mcp serve`.
- Reason: both isolate their dependencies; pipx gives a stable `~/.local/bin/1c-testpilot`, uvx needs no install step.
- Alternatives considered: plain `pip install --break-system-packages` (pollutes the global interpreter), cloning and running from source (no version pinning).
- Consequences: Testpilot upgrades need `pipx upgrade`; uvx resolves the MCP server on every start.

### Python port branch of cc-1c-skills (2026-10-24)

- Decision: use branch `port-opencode-py`, not `port-opencode`.
- Reason: there is no `pwsh` on macOS, so every PowerShell script is dead weight; the Python branch is the only runnable one.
- Alternatives considered: `port-opencode` (PowerShell, unusable), upstream Windows branch (same problem).
- Consequences: `.ps1` files still ship in each skill directory; ignore them.

### Symlinks to satisfy Testpilot's filename validation (2026-10-24)

- Decision: `~/.local/1cv8/8.3.27.1606/1cv8` → `/opt/1cv8t/8.3.27.1606/1cv8t`, `1cv8c` → `1cv8ct`.
- Reason: Testpilot rejects any executable whose basename is not exactly `1cv8` or `1cv8c`; the training build uses `1cv8t` / `1cv8ct`.
- Alternatives considered: copying binaries (breaks updates and signatures), patching Testpilot (maintenance burden).
- Consequences: all configs point at the symlinks, not at `/opt`. If the platform is reinstalled, recreate the links.

### Registry default is lab3 (2026-10-24)

- Decision: `.v8-project.json` has `"default": "lab3"`.
- Reason: lab3 is the active coursework base and the one with the fullest XML source dump.
- Alternatives considered: lab1 (no `src/`, no users), no default (agent must ask every time).
- Consequences: skills that fall back to the default will target lab3.

### Do not modify the global OpenCode config (2026-10-24)

- Decision: keep `~/.config/opencode/opencode.json` and `~/.config/opencode/AGENTS.md` as they are.
- Reason: shared across all projects; the user explicitly asked not to touch them.
- Alternatives considered: none.
- Consequences: workspace-specific rules must live in this repo's `AGENTS.md` / `.agents/`.
