# AGENTS.md

Agent entry point for this workspace. Detailed context lives in `.agents/`.

Communication with the user: Russian. Code, commits, and documentation: English.

## Start Here

Read [.agents/AGENTS.md](.agents/AGENTS.md) before any work. It links to:

- [.agents/project.md](.agents/project.md) — what this workspace is, platform, toolchain.
- [.agents/commands.md](.agents/commands.md) — verified commands (designer, batch dump, skills, Testpilot).
- [.agents/gotchas.md](.agents/gotchas.md) — traps already stepped on; read before repeating work.
- [.agents/bugs.md](.agents/bugs.md) — known bugs and fixes.
- [.agents/project-decisions.md](.agents/project-decisions.md) — why things are set up this way.
- [.agents/architecture.md](.agents/architecture.md) — layout of `.opencode/`, skills, MCP, registry.
- [.agents/todo.md](.agents/todo.md) — current task checklist.

## Non-Negotiable Facts

- **Code comes only from the methodichkas.** The teacher's PDFs in `labN/src/` are the single source of truth for code, object names, UI steps, and order of operations. Code there is embedded as images, so render the page and read it with vision — never reproduce it from memory. Do not add anything beyond the methodichka. If a step is missing or unreadable, stop and ask.
- 1C platform: `/opt/1cv8t/8.3.27.1606/`; use the symlinks `~/.local/1cv8/8.3.27.1606/1cv8` and `1cv8c`, never the `/Applications` stubs.
- Base credentials: lab1 — no users; lab2 — `админ` (Cyrillic); lab3 — `admin` (Latin). Passwords are empty, and an empty password must be omitted, never passed as `/P ""`.
- Skills invoke `python`; the shim `~/.local/bin/python` must exist.
- Do not modify `~/.config/opencode/opencode.json`.

## Maintenance

Keep `.agents/gotchas.md` and `.agents/todo.md` current. Log a gotcha the moment something fails, not at session end.
