# Agent Skills

Two kinds of skills are in play:

- **Installed project skills** — 81 skills under `.opencode/skills/`, from `cc-1c-skills` (branch `port-opencode-py`). They are copied artifacts; do not edit them in place.
- **Repo-scoped skills** — `.agents/skills/*/SKILL.md`, for workflows specific to this workspace (`agent-desktop` pair, installed from upstream).

## Installed Skills (`.opencode/skills`)

Manifest: `.opencode/skills/.cc-1c-skills.json` (`platform: opencode`, `runtime: python`, mode `copy`). Dependencies: `.opencode/skills/requirements.txt`.

Invocation pattern:

```bash
python .opencode/skills/<skill>/scripts/<skill>.py --help
```

| Group | Skills | Use when |
|---|---|---|
| Configuration | `cf-info`, `cf-init`, `cf-edit`, `cf-validate` | Inspect or change the configuration as a whole |
| Extensions | `cfe-init`, `cfe-borrow`, `cfe-patch-method`, `cfe-diff`, `cfe-validate` | Work with a `.cfe` extension |
| Infobase | `db-list`, `db-create`, `db-run`, `db-update`, `db-dump-cf`, `db-load-cf`, `db-dump-dt`, `db-load-dt`, `db-dump-xml`, `db-load-xml`, `db-load-git`, `db-repo`, `db-cfe-admin` | Operate on an infobase |
| External processors | `epf-init`, `epf-dump`, `epf-build`, `epf-validate`, `epf-bsp-init`, `epf-bsp-add-command` | Build or inspect `.epf` |
| External reports | `erf-init`, `erf-dump`, `erf-build`, `erf-validate` | Build or inspect `.erf` |
| Managed forms | `form-add`, `form-compile`, `form-decompile`, `form-edit`, `form-info`, `form-remove`, `form-validate`, `form-patterns` | Create or modify a form |
| Metadata | `meta-compile`, `meta-decompile`, `meta-edit`, `meta-info`, `meta-remove`, `meta-validate` | Add or change configuration objects |
| Roles | `role-compile`, `role-edit`, `role-info`, `role-validate` | Rights and RLS |
| Subsystems | `subsystem-compile`, `subsystem-edit`, `subsystem-info`, `subsystem-validate` | Sections and command interface |
| Command interface | `interface-edit`, `interface-validate` | Show/hide commands |
| Data composition | `skd-compile`, `skd-decompile`, `skd-edit`, `skd-info`, `skd-validate` | Reports (DCS) |
| Spreadsheet templates | `mxl-compile`, `mxl-decompile`, `mxl-info`, `mxl-validate` | Print forms |
| XDTO | `xdto-compile`, `xdto-decompile`, `xdto-edit`, `xdto-info`, `xdto-validate` | XML schemas and exchange |
| Web | `web-publish`, `web-unpublish`, `web-info`, `web-stop`, `web-test` | Apache publication and web client |
| Misc | `help-add`, `img-grid`, `support-edit`, `template-add`, `template-remove`, `v8-xsd-fetch` | Built-in help, image grids, support flags, templates, XSD |

## Important Constraints

- All `.ps1` scripts are unusable: no PowerShell on this machine. Use the `.py` counterparts.
- Scripts call `python` from `PATH`; the shim `~/.local/bin/python` must exist.
- `db-*` scripts read `v8path` from `.v8-project.json` but expect the agent to pass the infobase path and credentials explicitly.
- Auto-detection of the platform looks at `/opt/1cv8/*/1cv8`, which does not match this machine (`/opt/1cv8t`). Always rely on the registry `v8path`.

## Repo Skills

Installed from upstream with `npx skills add`, copied into this folder, do not edit in place.
Reinstall after an `agent-desktop` upgrade to keep them version-matched.

| Skill | Use when | Source |
|---|---|---|
| `agent-desktop` | Any desktop GUI task: snapshot, click, type, windows, clipboard, notifications | `vercel-labs/agent-desktop` |
| `agent-desktop-windows` | Windows specifics: UIA, PowerShell ref quoting, `--headed`-only input, elevation | `vercel-labs/agent-desktop` |

- The binary also embeds the same docs: `agent-desktop skills get desktop`, `agent-desktop skills get platform`.
- `agent-desktop-macos` / `agent-desktop-linux` / `agent-desktop-ffi` / `jev-desktop` are not installed; add them only if the target host changes.
- macOS install is deliberately deferred: probe the 1C AX tree first, see `.agents/todo.md` and `.agents/project-decisions.md`. There, `macos` MCP remains the desktop input path.

## Authoring Notes

- Add new repeatable workflows as `.agents/skills/<skill-name>/SKILL.md`.
- Use lowercase hyphenated skill names.
- Start skill descriptions with `Use when...`.
- Keep `SKILL.md` lean; link to `.agents/*.md` for large reference material.
