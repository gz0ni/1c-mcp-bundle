# Repo Skills

Repo-scoped skills live in this folder as `<skill-name>/SKILL.md`.

## Adding a skill

1. Create `<skill-name>/SKILL.md` with:
   - lowercase hyphenated name;
   - description starting with `Use when...` (trigger-focused);
   - lean instructions; link to `.agents/*.md` for large reference material.
2. List it in [.agents/skills.md](../skills.md).
3. Load it via the skill tool when a task matches its description.

## Current skills

- `agent-desktop` — desktop GUI control through the OS accessibility tree (installed from `vercel-labs/agent-desktop`).
- `agent-desktop-windows` — Windows platform guide for the same CLI.

Installed with `npx skills add`; do not edit in place, reinstall to update.

Candidate workflows worth capturing here later:

- `lab-base-triage` — open a base, identify its users, and decide whether batch mode will block.
- `dump-and-verify` — dump a base to XML and run `cf-info` as a smoke test.
