# Architecture

Map of what lives where. This is a workspace, not an application: there is no build, only tooling around three 1C file infobases.

## Layout

```
/Users/gz0ni/Documents/Work/Languages/1c/
├── AGENTS.md                     # root entry point -> .agents/
├── .agents/                      # agent journals and reference docs (this folder)
├── .opencode/
│   ├── opencode.json             # project-scoped MCP servers (V2 schema: mcp.servers)
│   ├── 1c-testpilot.profiles.yaml
│   └── skills/                   # 81 cc-1c-skills + .cc-1c-skills.json + requirements.txt
├── .v8-project.json              # database registry read by every db-* skill
├── lab1/                         # file infobase, no users, no src/
├── lab2/                         # file infobase, user "админ", src/ present
└── lab3/                         # file infobase, user "admin", src/ present, default
```

Each `labN/` is a 1C file infobase: `1Cv8.1CD` (data + config), `1Cv8Log/` (event log), lock files `.cfl`, and `src/` when the configuration was dumped to XML.

## Entry Points

- Agent entry: root `AGENTS.md` → `.agents/AGENTS.md` → journals.
- 1C entry: `~/.local/1cv8/8.3.27.1606/1cv8` (designer/thick) or `1cv8c` (thin client).
- Skills entry: `.opencode/skills/<skill>/SKILL.md`, executed via `python .opencode/skills/<skill>/scripts/<skill>.py`.

## Toolchain Layer

### Skills (`.opencode/skills`)

81 skills from `cc-1c-skills`, branch `port-opencode-py`. Manifest `.cc-1c-skills.json` records `platform: opencode`, `runtime: python`, mode `copy`.

Groups:

- `cf-*` — configuration as a whole (info, init, edit, validate).
- `cfe-*` — extensions (borrow, patch, diff, init, validate).
- `db-*` — infobase operations (create, dump/load cf/dt/xml, update, repo, run, list, cfe-admin).
- `epf-*`, `erf-*` — external data processors and reports.
- `form-*` — managed forms.
- `meta-*` — metadata objects.
- `role-*`, `subsystem-*`, `interface-*` — rights, subsystems, command interface.
- `skd-*` — data composition schemas.
- `mxl-*` — spreadsheet templates.
- `xdto-*` — XDTO packages.
- `web-*` — Apache publication and web-client testing.
- Misc: `help-add`, `img-grid`, `support-edit`, `template-add/remove`, `v8-xsd-fetch`.

Each skill has a Python port; `.ps1` files are present but unusable (no PowerShell on macOS).

### MCP servers (`.opencode/opencode.json`)

- `1c-testpilot` — local stdio, drives a live `/TESTCLIENT`.
- `1c-toolkit` — remote HTTP `http://127.0.0.1:6003/mcp`, served by `MCP_Toolkit.epf` inside 1C. Not running by default.
- `macos` — local `uvx macos-mcp serve`, Accessibility tree and input.

### Registry (`.v8-project.json`)

Root `v8path` points at the `1cv8` symlink. Each database entry carries `path`, `configSrc`, `user`, `password`, `aliases`. `default` is `lab3`.

Skills read `v8path` from the registry but expect the agent to resolve the base and credentials; see `.agents/commands.md`.

## Data Flow

1. Agent reads `.v8-project.json` → picks base, path, user.
2. Agent runs a skill script with `python` → script calls the 1C binary in batch mode.
3. For live UI work, Testpilot launches `1cv8c /TESTCLIENT -TPort <port>` and connects over TCP.
4. Configuration edits go to `lab*/src` (XML), then `db-load-xml` + `db-update` push them into `1Cv8.1CD`.

## Known Gaps

- `1c-toolkit` is configured but never connected.
- `lab1` has no `src/` dump — only lab2 and lab3 do.
- No git, no CI, no tests.
- `macos` MCP cannot use AppleScript (TCC).

## Roadmap

1. Get `1c-toolkit` running (open `MCP_Toolkit.epf`, start server on 6003).
2. Dump `lab1` sources if it is going to be edited.
3. Keep `.agents/gotchas.md` current — it is the main time saver.
