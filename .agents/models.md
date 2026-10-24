# Model Catalog and Routing

Optional, for projects that route LLM models. This workspace does not pin models; the table records what works well in practice. Active as of 2026-10-24.

## Routing

| Task class | Model to propose |
|---|---|
| Default coding | Whatever the session uses; no preference |
| Complex refactoring, planning | A reasoning-capable model |
| Long autonomous work | A model with a large context window |
| Vision / UI debugging | Any vision-capable model (screenshots of the 1C UI are the main input) |
| Code review | A second model, different from the author |
| Exploration / research | Fast/cheap model, ideally a subagent |
| Routine (tests, simple edits) | Fast/cheap model |

## Workflow

1. Classify the task against the table.
2. If the current session model does not fit, propose a switch in one line: `Switch to /models -> <model> for this task.`
3. Delegate exploration and review to subagents instead of the main model.
4. Never burn top-tier models on routine work.

## Notes for This Project

- Vision matters: the only reliable way to see 1C dialogs is a screenshot, because AX does not expose the app.
- Long context matters: skill files and XML dumps are large; prefer targeted reads over dumping whole directories.
