# Agent Configuration Model

Placement rules for agent surfaces in this workspace. Stable conventions live here, `AGENTS.md` links to it.

## Surfaces

- `AGENTS.md` (root): auto-loaded repository entry point. Kept small — language rule, pointer to `.agents/AGENTS.md`.
- `.agents/AGENTS.md`: detailed entry point, links to every journal and reference doc.
- `.agents/*.md`: human- and agent-readable reference docs.
- `.agents/skills/*/SKILL.md`: repo-scoped skills, loaded only when a task matches.
- `.opencode/skills/*/SKILL.md`: the 81 installed `cc-1c-skills`. These are **project-scoped skill installations**, not repo-authored skills; do not edit them unless updating the upstream port.
- `~/.config/opencode/AGENTS.md`: global user-level rules. Do not modify without an explicit request.

## Placement Rules

- Put stable conventions in `AGENTS.md` only when every task must see them.
- Put detailed explanations in `.agents/*.md` and link them from `AGENTS.md`.
- Put reusable task workflows in `.agents/skills/<skill-name>/SKILL.md`.
- Put mechanical enforcement in scripts or checks — do not rely on prose when tooling can enforce the rule.
- Keep user-specific preferences out of the repository.

## MCP Servers

Configured in `.opencode/opencode.json` using the V2 schema (`mcp.servers`), which is scoped to this workspace only. The global `~/.config/opencode/opencode.json` uses the older V1 shape and is intentionally untouched; both sets of servers load together.

| Server | Type | Purpose | Requirement |
|---|---|---|---|
| `1c-testpilot` | local stdio | Launch and drive a live 1C `/TESTCLIENT` | `~/.local/bin/1c-testpilot`, profiles file |
| `1c-toolkit` | remote HTTP | 1C MCP toolkit (`http://127.0.0.1:6003/mcp`) | `MCP_Toolkit.epf` running inside 1C |
| `macos` | local stdio | Accessibility tree, clicks, typing, screenshots | Accessibility permission for the uv-managed Python |

### Testpilot environment

```json
{
  "TC1C_TRANSPORT": "stdio",
  "TC_PLATFORM_VERSION": "8.3.27.1606",
  "TC1C_PLATFORM_EXE": "/Users/gz0ni/.local/1cv8/8.3.27.1606/1cv8c",
  "TC1C_PROFILES_FILE": "/Users/gz0ni/Documents/Work/Languages/1c/.opencode/1c-testpilot.profiles.yaml"
}
```

`TC1C_PLATFORM_EXE` must point at a file literally named `1cv8c` — Testpilot rejects anything else with `only 1cv8, 1cv8c may be launched`.

### Profiles

`.opencode/1c-testpilot.profiles.yaml` defines `lab1`, `lab2`, `lab3` with `base`, `exe`, `version`, `wait`, and credentials (`user`, `password`). Testpilot re-reads the file on each call, so edits apply without restarting the MCP server.

## Skill Authoring Rules

- Lowercase hyphenated names.
- Descriptions start with `Use when...` and describe trigger conditions.
- Keep `SKILL.md` lean; link to `.agents/*.md` for reference material.
