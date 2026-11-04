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

- [ ] Start `MCP_Toolkit.epf` inside 1C and confirm `1c-toolkit` connects on port 6003. **Blocked on macOS (2026-11-02):** the EPF's transport add-in is a Windows `PE32+` DLL, so the built-in server cannot load here. Do this on the Windows machine with `tools/MCP_Toolkit.epf`.
- [ ] Add the Windows-MCP server when the Windows machine is set up (deliberately not in this repo yet). Same trip can close the `MCP_Toolkit` item.
- [ ] Dump `lab1` configuration to `lab1/src` if it is going to be edited (no `src/` yet).
- [ ] Confirm the `macos` MCP `Type` tool can enter text into the 1C login dialog (only `Shortcut`/`Click` were exercised).
- [ ] Fill `lab4`–`lab8` when the coursework reaches them; add their PDFs to `labN/src/`.
- [ ] Decide whether to keep `lab1` in the registry without credentials or add a user to the base.

## In Progress

- [ ] (none)

## Done

- [x] lab3 filled with lab2 data (2026-11-04): throwaway EPF `ПереносДанных` (scratchpad, not in repo) serialises constants, catalogs and documents of lab2 with `ЗаписатьXML` and reloads them into lab3 with the same refs, then reposts the 6 documents. Verified by re-export (byte-identical to the lab2 export) and by the `Цены` register: 21 records, no orphans. lab2 infobase users were not copied (lab3 keeps `admin`). Gotchas: `Структура`/`ФиксированнаяСтруктура` do not serialise to XML, subordinate registers cannot be cleared without a recorder filter, the EPF security dialog needs a click per fresh build.

- [x] lab3 variant 16 (2026-11-04): copied `lab2/src` to `lab3/src`, added registers `Цены` (Second, recorder-subordinate; `Копия`, `ТипКопий`, resource `Стоимость`) and `УчетнаяПолитика` (Year, independent, resource `СтратегияУчета`), `ОбработкаПроведения` in `УстановкаЦен`/`Производство`/`Оплата` (the last two with `Перечисления.ТипыКопий.ВанГог`), `ТипКопийПриИзменении`, `ПолучитьСтоимость` and the `Копия` handlers per methodichka figs 2.4–2.9. Role UUIDs aligned with the old `lab3` base so its user kept administrator rights; full load with `-UpdateDB`, `/CheckModules` clean, Testpilot: `УстановкаЦен` posted 100,00, `Производство` row picked up `Стоимость` 100,00 and `Сумма` 100,00. The old freehand `lab3` content was replaced (no backup, user's call). Not done: `.dt` for the report, demo data, `ТипКопийПриИзменении` not exercised in the client.

- [x] Move OpenCode MCP servers to Claude Code on Windows (2026-11-04): `context7`, `1c-mcp` in user scope (`gsheets`, `github`, `agentmemory`, `blockbench` dropped by the user); `1c-testpilot` in project `.mcp.json`. Verified with `claude mcp list` and a Testpilot launch/stop on lab1. `macos` MCP not moved — macOS-only.

- [x] lab2 variant 16: build `УчетКопийКартин` in `D:\Work\1C\lab2\src` (3 subsystems, 2 constants, 2 enums, 2 hierarchical catalogs, 3 documents, constants form, 2 roles, users), load with `-UpdateDB`, `cf-validate` + `form-validate` + `role-validate` + `/CheckModules` clean.
- [x] lab2 total not recalculated after deleting a row: `AfterDeleteRow` → `КопииПослеУдаления` (calls `ПересчитатьОбщуюСумму`) in `Производство` and `Оплата`; verified by Testpilot: 150,00 → 50,00 and 70,00 → 0,00 (2026-11-04).
- [x] lab2 fix `Поле объекта не обнаружено (Родитель)` in `Производство`/`Оплата`/`УстановкаЦен` forms (server function `ПолучитьГруппуКопии`), partial load with 8.3.27, verified by Testpilot: picking `GrrMonday` fills `Группа = Рембрант` in all three forms, no client error (2026-11-04).
- [x] lab2 section «Индивидуальный предприниматель» no longer blank (`ФормаКонстант` `UseStandardCommands=true`, constant commands unhidden), reloaded with `-UpdateDB`; sections «Заказы» and «Оплаты» checked by screenshot (2026-11-04).
- [x] lab2 demo documents via EPF `ДемоДанные`: `Контрагенты` (groups `Покупатели`/`Поставщики`, 4 items), 2 `УстановкаЦен`, 2 `Производство`, 2 `Оплата`, all posted with the constants snapshot — verified by `demo.txt` read-back.
- [x] lab2 demo data via EPF `ЗаполнениеДанных`: constants «ЗАО „Тлиз“» / «г. Севастополь, пр. Науки, 5», groups `ВанГог`/`Рембрант` + 11 paintings, users `Администратор`/`Головач Елена` with roles — verified by file read-back.
- [x] lab2 logic verified by EPF `ПроверкаЛогики`: document snapshot of constants on posting, `Группа` column, auto-total 350.00 on the form, catalog form “Группа” field and its hiding for groups, constants form open/save.
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
- [x] Install `agent-desktop` 1.0.2 project-local for the Windows host (`npm run setup` links the native binary) and its two skills into `.agents/skills/`. Verified: `status` reports accessibility granted, snapshot on Calculator returned 56 refs, `click` on `Семь` moved the display to `7`.
- [x] Probe `agent-desktop` on the macOS host. **2026-11-02 — fail, skipped.** Thin client from a bare binary: `ref_count` 3 (Close/Minimize/Full Screen, English names). Thick client from the bundle: `snapshot` → `TIMEOUT` / `global_window_inventory_unstable`, window absent from `list-windows`. 1C publishes no AX tree on macOS, so `macos` MCP stays the input path there.
- [x] Confirm the `agent-desktop` UIA tree for a live 1C client on Windows. 2026-11-02 on the lab1 configurator: 73-99 refs, readable Russian names, menus and the configuration tree are exposed, but tree nodes only advertise `SetFocus` and their `Expand`/`Click` return `E_NOTIMPL`. Semantic clicks work on window chrome; the tree has to be driven with `press` and `--headed` mouse.
