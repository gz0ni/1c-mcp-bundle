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

### Preserve the user's lab3 DOCX layout for the lab4 report (2026-11-11)

- Decision: use the supplied lab3 report as the layout authority; replace only its lab number, subject and body content.
- Reason: the user explicitly asked to retain the student, teacher and report formatting. Generic thesis numbering/layout rules do not override that request.
- Consequences: title text boxes, margins, styles and footer convention remain unchanged. New actual screenshots are cropped with native Word picture formatting for legibility; source screenshot bytes are retained. Native Word PDF export provides visual QA because LibreOffice is unavailable on this host.

### Expose the existing lab4 processor in Orders (2026-11-11)

- Decision: add `DataProcessor.Запрос` to the existing `Заказы` subsystem.
- Reason: the user explicitly requires opening and running the processor through visible buttons instead of entering navigation links.
- Consequences: the verified path is Orders > Service > Query > Execute. This supersedes the initial decision to leave section membership unchanged; no new objects or query logic were introduced.

### Lab4 continues the supplied lab3 base (2026-11-11)

- Decision: preserve all existing lab4 objects and data; add only the query processor from PDF pp. 19–24 and variant 16 on p. 31.
- Reason: p. 26 explicitly requires the previous labs' result, so restoring lab2 is unnecessary.
- Consequences: the variant's price uses the existing `Стоимость` attribute, output alias `Цена`; aggregation is minimum per document, sorted descending. Earlier lab code and pre-existing test data are outside this change.

### Desktop automation on macOS stays with the `macos` MCP (2026-11-02)

- Decision: skip `agent-desktop` on the macOS host; 1C desktop input continues through `macos` MCP (Snapshot/Click/Type/Shortcut) plus screenshots.
- Reason: the probe failed on both launch paths — 3 refs for the thin client (window buttons only), a `TIMEOUT` for the bundle-launched thick client. 1C publishes no AX tree on macOS, so agent-desktop has nothing to read.
- Alternatives considered: installing it anyway (adds a permission grant and a second GUI path that sees nothing), retrying with different `--app` values (the app was found; the tree is empty, so the target is not the problem).
- Consequences: `agent-desktop` stays a Windows-host tool; `.agents/skills/agent-desktop` documents Windows behaviour only in practice. Numbers and commands are in `.agents/gotchas.md` and `.agents/commands.md`.

### `MCP_Toolkit` / `1c-toolkit` is a Windows-host task (2026-11-02)

- Decision: `tools/MCP_Toolkit.epf` is only useful on Windows; do not expect port 6003 on the Mac.
- Reason: the built-in server is a native add-in shipped as `PE32+ DLL x86-64` inside the EPF, so 1C on macOS cannot load it, and the helper component chain (`MCPHttp`, `ToonConverter`, `SyntaxHelpReader`, `ScreenCapture`) is Windows-only too.
- Alternatives considered: proxy mode (needs an external MCP server to point at, nothing to run here), rebuilding the add-in for macOS (no sources for the native part).
- Consequences: `1c-toolkit` in `.opencode/opencode.json` keeps failing with `ECONNREFUSED` on this machine and that is expected; the todo item waits for the Windows machine.

### agent-desktop on macOS is deferred until the AX tree is proven (2026-11-02)

- Decision: do not install `agent-desktop` on the macOS host yet; probe the live 1C accessibility tree first and install only if it is usable.
- Reason: on macOS it reads the same Accessibility API as the `macos` MCP, so it adds no new capability, only refs and targeted `find`/`is`/`get` instead of a full snapshot. That is worth a TCC Accessibility grant and a second GUI path only if 1C exposes a real tree. The recorded "no AX" finding came from launching the bare binary, and 1C is used as a `.app` bundle, so the premise needs re-checking.
- Alternatives considered: installing anyway (adds a grant, a duplicate path, and a false expectation), skipping outright (loses a possible 10x token saving on verification steps).
- Consequences: probe commands and the pass/fail rule live in `.agents/todo.md`; `agent-desktop-macos` is not installed; on macOS the desktop input path stays `macos` MCP until the probe fails or passes.
- Outcome (2026-11-02): the probe failed on both launch paths (3 refs for the thin client, `TIMEOUT` for the bundle-launched thick client). Kept as history; the decision below records the final verdict.

### agent-desktop installed project-local for the Windows host (2026-11-02)

- Decision: `npm install agent-desktop` in the repo root (`node_modules/`, gitignored) plus its two skills from `npx skills add` into `.agents/skills/`.
- Reason: the Windows host had no desktop input tool at all — no `macos` MCP (darwin-only), and `1c-testpilot` drives the 1C protocol, not the GUI. agent-desktop gives UIA snapshots with refs, headless clicks, window and clipboard control, and needs no permission grant on Windows.
- Alternatives considered: raw `Add-Type` + Win32 P/Invoke (write it ourselves, no snapshot semantics), Windows-MCP server (extra MCP process, unverified), coordinate clicking off screenshots (kept only as the fallback when the UIA tree is flat).
- Consequences: repo root is now an npm package (`package.json`, `package-lock.json`, `skills-lock.json` versioned; `node_modules/` ignored). `npm run setup` is required after `npm install` because the postinstall hook is blocked. The macOS toolchain is untouched: `macos` MCP stays the input path there.

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

### Lab5 follows the printed register and posting algorithm (2026-11-11)

- Decision: use existing lab4 in lab5; add only `ОстаткиПроизводств` and `Оплаты`, receipt movements, grouped stock check and turnover movements from PDF figures 2.1–2.9 and variant16 on p.21.
- Reason: screenshots place quantity/amount under Resources, despite the prose calling them attributes. No expense movements or FIFO/LIFO algorithm is printed in this lab, so no such functionality was invented.
- Preserved all existing forms, role IDs and inherited code, including the user-added copy-type column. Reposted the two existing production/payment documents to populate new registers. No lab3 import.
- Verification and limitations: `lab5/evidence/verification.md`; final DT `lab5/lab5-variant16.dt`.

### Preserve the supplied lab4 report for lab5 (2026-11-11)

- Decision: derive the lab5 DOCX from the user-supplied lab4 DOCX, preserving student, teacher, cover layout, direct paragraph formatting, styles, section geometry and footer.
- Reason: the user explicitly requested the same example, student and teacher. Two-line lab5 subject replaces the previous subject plus its blank line, preserving cover height.
- Evidence: native Word render, 13 inspected pages, 8 genuine screenshots, exact source module listings and package/pixel fidelity checks in `lab5/report-work/`. Only final DOCX copied to the requested Homework/5 folder.
