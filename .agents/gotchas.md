# Gotchas

Traps and pitfalls the agent has stepped on in this project. Log a gotcha immediately whenever something does not build, run, or pass — right after the incident, while context is fresh, do not wait for session end. Check this file before repeating similar work. One entry per trap.

## Format

```
### <Short trap title> (YYYY-MM-DD)

- What happened / what was wrong
- Root cause
- How to avoid next time
```

---

<!-- New entries go here, newest first. -->

### python-docx reserializes untouched template parts (2026-11-11)

- Lab4 report package comparison detected changes in root relationships, footer, settings and styles after saving the reference-derived copy.
- Restore preserve-only ZIP members byte-for-byte from the retained reference after authoring; keep only intended document, new-image relationship and core-title changes. Recheck package fidelity before delivery.

### Computer Use GPU capture failed in the Configurator (2026-11-11)

- `sky.get_window_state` failed at Direct3D11 frame capture with `0x887A0005` (GPU device removed).
- Re-select the live window and retry once; if persistent, switch the capture/control provider instead of reusing stale screenshot coordinates.
- The GPU error persisted in both Sky providers. The global agent-desktop CLI can still save screenshots and list windows, although the old default state root fails private-file DACL checks. Its documented `AGENT_DESKTOP_HOME` relocation to a fresh task-local directory restores ref snapshots without changing old permissions. Explicit `focus-window` succeeded before headed keyboard input; `press --headed` alone had failed to acquire foreground focus.

### DOCX renderer requires unavailable LibreOffice on this Windows host (2026-11-11)

- `render_docx.py` failed because `soffice.exe` is not installed; sandbox temp cleanup also returned access denied.
- Microsoft Word is installed. Use a read-only native Word COM PDF export and bundled pypdfium2 to render page images; do not modify the reference DOCX during conversion.
- Word COM misinterprets paths with forward slashes and spaces as URLs. Normalize both paths to absolute Windows backslash paths before `Documents.Open` or PDF export.

### Testpilot command catalogue cannot inspect every section menu (2026-11-11)

- First catalogue search requires an unfiltered scan or `refresh=true`. On lab4 the scan then returned `menu_state_unavailable` for the Main section and no commands.
- Do not interpret an incomplete scan as a missing command. Inspect the window and navigate the desired section directly before verifying its visible commands.

### Testpilot no-active-window errors can indicate an exited client (2026-11-11)

- A launched lab4 client connected, then navigation and screenshot calls reported no active window. The launched PID no longer existed.
- Check process liveness before retrying UI commands. Clear the owned connection and do not assume a configuration failure from this session-state error.

### Nested Python quoting in PowerShell breaks inline verification (2026-11-11)

- A long `python -c` containing escaped XPath quotes failed with `SyntaxError: unterminated string literal`.
- Store nontrivial verification code in a workspace `.py` file and run it directly; avoid nesting shell and Python string escaping.

### Global agent-desktop refuses its private-file permissions (2026-11-11)

- `agent-desktop status` returned `PERM_DENIED: private file DACL grants an untrusted principal`; no local binary exists under this checkout's `node_modules`.
- Do not change permission checks or DACLs during coursework. Use Testpilot's native screenshot API for 1C verification.

### MXL DSL column widths are absolute, rowStyle extends across the sheet (2026-11-11)

- The lab4 query passed XML/module checks but rendered narrow columns and borders across the whole spreadsheet.
- `columnWidths` uses absolute platform units, not character counts; `rowStyle` sets formatting for the entire row, including unused columns.
- Use widths measured from the PDF screenshot and apply borders to the two explicit cells instead of the row. Always inspect the rendered form.

### Spreadsheet export can leave a format confirmation dialog (2026-11-11)

- A second Testpilot spreadsheet export returned `target_not_interactive` after the HTML export request.
- An accepted export request is asynchronous and can open another dialog. Inspect the active window and confirm the first export before sending another one.
- The actual lab4 dialog says the training license disables printing/saving spreadsheet documents. Do not attempt additional exports; use protocol cell reads and screenshots for evidence.

### Testpilot screenshot requires a restored 1C window (2026-11-11)

- `get_screenshot` returned `screenshot_window_minimized` after a successful lab4 query.
- The client was minimized; restore its window before requesting a screenshot. Protocol form actions still work while minimized.

### Sandboxed Designer cannot access its user profile (2026-11-11)

- lab4 XML dump exited 1: access denied to `AppData/Roaming/1C/1cv8t/<uuid>`.
- 1C requires its normal profile outside the workspace. Run the authorized Designer command with escalation.
- `uv python list` also retries unwritable sandbox cache paths; avoid it or set a workspace cache directory.

### Windows Python and database registry differ from the old host (2026-11-11)

- The current `python` resolves to MSYS2 and has no `fitz`; `.v8-project.json` is absent.
- The reference files describe a different host. Use available PDF tools or a verified Python interpreter, and pass the lab4 path and platform explicitly.
- The bundled Python also lacks `fitz`, and bundled Poppler has no `pdftotext.exe`. Inspect installed packages/binaries before choosing the PDF extraction command.
- Use bundled Python with `-X utf8`: it has `pypdf`, `pdfplumber`, `pypdfium2`, `PIL`, and `lxml`. Bare MSYS2 Python lacks `lxml`; default bundled stdout encoding is CP1251 and fails on some PDF characters.

### `uv tool install` refuses to replace an existing `1c-testpilot.exe` (2026-11-05)

- `uv tool install "git+https://github.com/ROCTUP/1c-testpilot"` finished resolving and installing packages, then exited 1 with `error: Executable already exists: 1c-testpilot.exe (use --force to overwrite)`.
- Root cause: a stale trampoline `C:\Users\user\.local\bin\1c-testpilot.exe` survived the deleted uv tool environment (`uv tool list` said `No tools installed`), and the failing binary reported `uv trampoline failed to canonicalize script path`.
- Avoid: `uv tool install --force "git+https://github.com/ROCTUP/1c-testpilot"`. Verify with `uv tool list` (expect `1c-testpilot v1.9.2`) and a stdio `initialize`; `1c-testpilot --version` does **not** exist — the CLI only accepts `--help` and `--env-file`.

### Claude Code has no `.cloudcode` folder (2026-11-05)

- A request to "move MCP to `.cloudcode`" cannot be satisfied literally: Claude Code reads only `.mcp.json` in the project root, `~/.claude.json` (user scope) or a file passed with `--mcp-config`.
- Avoid: keep the project config in `.mcp.json`; if a custom path is mandatory, launch `claude --mcp-config <path>`, otherwise plain `claude` silently sees no project servers.

### Full `db-load-xml` into a base with users fails: `не осталось ни одного пользователя с административными правами` (2026-11-04)

- Loading `lab3/src` (a lab2 copy) into the `lab3` base failed after listing `Объект изменен: Пользователи информационной базы`.
- Root cause: base users keep roles by role UUID; the lab2 roles had other UUIDs, so `admin` would lose `Администратор`.
- Avoid: copy the role UUIDs of the target base into `Roles/*.xml` and `ConfigDumpInfo.xml` (dump the base first, read `<Role uuid=...>`), then reload.
- Also: `Remove-Item` on a Temp path was blocked by a guard on `D:\Rubbish`; write check output to a unique scratch file instead.
- Also: a new document's `Дата` defaults to today 00:00:00, so a `Цены` register with `Second` periodicity returns nothing for it when the price was posted later the same day; set the date after the price document in manual tests.

### `lab3` base already holds an old freehand config, not the lab2 copy (2026-11-04)

- A dump of the `lab3` base (`UchetKopiyKartin` with registers `Цены`/`УчетнаяПолитика`, enum `СтратегииУчета`, EPF-like `ЗаполнениеДемо`) differs from `lab2/src` in nearly every module. Its document modules carry a `ТипКопий` attribute and query-based code that the methodichka does not print.
- Root cause: an earlier attempt built lab3 without following the printed code; `lab3/src` did not exist.
- Avoid: lab3 starts from a copy of `lab2/src` (done in `lab3/src`), not from the base content. Back up the `lab3` base folder before the first `-UpdateDB` load that replaces it.
- Also: there is no `.v8-project.json` on Windows, so `db-dump-xml.py` fails with `1C executable not found`; pass `-V8Path <dir of 1cv8t.exe>` (`D:\Rubbish\...\8.3.27.1508\bin`, resolve it with `Get-ChildItem`, see the Cyrillic path gotcha).

### Reference fields read on the client fail: `Поле объекта не обнаружено (Родитель)` (2026-11-04)

- lab2 `Производство` form: picking a `Копия` raised the error at `ТекСтрока.Группа = ТекСтрока.Копия.Родитель;` in the `&НаКлиенте` handler. `Оплата` and `УстановкаЦен` had the same line. The demo-data EPF filled `Группа` on the server, so the bug only showed up in manual input.
- Root cause: a managed-form client holds only the reference value, not the object's attributes.
- Avoid: read reference attributes through a `&НаСервереБезКонтекста` function (`ПолучитьГруппуКопии(Копия)` returns `Копия.Родитель`). Test `ПриИзменении` handlers by hand in the client, not only via server-side scripts.

### `db-load-xml` fails while the user's Configurator is open (2026-11-04)

- `Ошибка блокировки информационной базы для конфигурирования. Возможно, информационная база уже открыта Конфигуратором.`, exit code 1.
- Root cause: batch `/LoadConfigFromFiles` needs the configuration lock that an interactive Configurator holds.
- Avoid: check `Get-Process 1cv8*` first and ask the user to close the Configurator; do not kill it (unsaved edits).
- Also on this host: bare `python` is 3.10 without `lxml`; run skill scripts with `py -3.13`.

### Subsystem section is blank when its only commands are hidden or a common form has no command (2026-11-04)

- lab2: clicking «Индивидуальный предприниматель» showed an empty page.
- Root cause: `CommonForms/ФормаКонстант.xml` had `<UseStandardCommands>false</UseStandardCommands>`, so the form produced no section command, and the subsystem `CommandInterface.xml` hid both `Constant.*.StandardCommand.Open`. Nothing visible was left.
- Avoid: set `UseStandardCommands` to `true` on a common form that must appear in a section, and check `CommandsVisibility` before hiding the last commands. Fixed by doing both; the section now shows «Название организации», «Форма констант», «Юридический адрес» under «Сервис».
- Also: in the section panel, y≈98 is «Главное», not «Заказы» (y≈165) in a 1840×952 window. A click on the wrong row looks like a blank section.

### Testpilot on Windows needs `1cv8c.exe`, the training build only ships `1cv8ct.exe` (2026-11-04)

- `1c-testpilot` validates the launcher name: `('1cv8.exe', '1cv8c.exe')` on Windows (`_runtime.py`). The portable 8.3.27.1508 training build has only `1cv8t.exe` / `1cv8ct.exe`.
- Root cause: same hard-coded check as on macOS; the `t` suffix of the training build is rejected.
- Avoid: hard links next to the originals (a symlink elsewhere would break DLL lookup): `bin\1cv8c.exe` → `1cv8ct.exe`, `bin\1cv8.exe` → `1cv8t.exe`. Created 2026-11-04 in `D:\Rubbish\никому не нужные проги\1с\8.3.27.1508\bin`.

### Project `.mcp.json` servers stay "Pending approval" in Claude Code (2026-11-04)

- `claude mcp list` showed `1c-testpilot ... ⏸ Pending approval`; the agent is not allowed to write `.claude/settings.local.json` itself (auto-mode classifier: self-modification).
- Root cause: Claude Code requires the user to trust servers from a project `.mcp.json`.
- Avoid: the user approves once, in an interactive `claude` started in `D:\Work\1C` or via the app's MCP prompt. Do not try to bypass it.
- Fix (2026-11-05): `D:\Work\1C\.claude\settings.json` with `{"enabledMcpjsonServers": ["1c-testpilot", "1c-mcp"]}` removes the prompt — written to the *project* settings file, not `settings.local.json`. After that `claude mcp list` reports `1c-testpilot ✔ Connected` without an interactive session.

### Document attributes changed in `ОбработкаПроведения` are not saved (2026-11-03)

- Snapshot code (`ЭтотОбъект.НазваниеОрганизации = Константы...Получить()`) placed in `ОбработкаПроведения` left the attributes empty after `Записать(РежимЗаписиДокумента.Проведение)`.
- Root cause: by the time `ОбработкаПроведения` runs, the document data is already written; changing the object there is discarded.
- Avoid: fill snapshot attributes in `ПередЗаписью` guarded by `Если РежимЗаписи = РежимЗаписиДокумента.Проведение Тогда`.

### `Константы.СоздатьНаборКонстант()` does not exist on this platform (2026-11-03)

- The common form for constants failed to open with `Метод объекта не обнаружен (СоздатьНаборКонстант)`.
- Root cause: the method is absent in 8.5.1 учебная (and in 8.3.x compatibility mode), and the `ConstantsSet` main form attribute is initialized by the platform anyway.
- Avoid: leave `ПриСозданииНаСервере` of a constants form empty; write constants with `Константы.<Имя>.Установить(...)`.

### New catalogs truncate names at 25 chars, silently creating duplicates (2026-11-03)

- An idempotent data loader kept creating copies of paintings whose names exceeded 25 chars: `НайтиПоНаименованию("Вечернее умиротворение с перфоратором у соседа")` returned an empty ref every run.
- Root cause: the default `<DescriptionLength>25</DescriptionLength>` in the catalog XML truncated the stored name, so the exact-match lookup never found the record.
- Avoid: raise `DescriptionLength` (e.g. 100) when demo data has long names, and verify catalog contents by dumping them to a file instead of trusting the count.

### 1C security dialogs have no child windows; click by coordinates (2026-11-03)

- `EnumChildWindows` on the 643x278 «Предупреждение безопасности» window returns nothing, and `SetCursorPos` + `mouse_event` from a background process often has no effect.
- Root cause: 1C draws its dialogs itself (`V8Window0.*` class), and the foreground lock blocks injected mouse input.
- Avoid: `SendMessage(hwnd, WM_LBUTTONDOWN/UP, 1/0, MAKELPARAM(x, y))` with client coordinates; «Да» sits at `(width - 132, height - 33)`. Expect 1–3 dialogs per freshly rebuilt EPF, and a repeat launch with no dialogs afterwards.

### PowerShell 5.1 reads BOM-less .ps1 as ANSI, mangling Cyrillic literals (2026-11-03)

- A script containing `'/N','Администратор'` failed to parse with `Нельзя найти элемент...` / broken tokens.
- Root cause: `-File` scripts without a UTF-8 BOM are decoded with the ANSI code page.
- Avoid: write `.ps1` files with UTF-8 BOM (`encoding='utf-8-sig'`), or build Cyrillic strings from `[char]` codes.

### `ОткрытьФорму` needs the full `Тип.Объект.Форма.ИмяФормы` path (2026-11-03)

- `ОткрытьФорму("Документ.Производство.ФормаДокумента")` raised `Неизвестное имя формы`.
- Root cause: the form-name path includes the `Форма` segment.
- Avoid: use `"Документ.Производство.Форма.ФормаДокумента"` (and `"Справочник.X.Форма.ФормаЭлемента"` for catalog forms).

### Server-side form opening is unavailable under `/Execute` (2026-11-03)

- `Документы.Производство.ПолучитьФорму(...).Открыть()` inside `ПриСозданииНаСервере` failed with `Интерактивные операции недоступны`.
- Root cause: the platform forbids interactive operations from a server call in that launch mode.
- Avoid: open forms on the client (`ПриОткрытии`) via `ОткрытьФорму(...)`, passing the ref through a form attribute filled on the server.

### A file infobase with users rejects DESIGNER without `/N` (2026-11-03)

- After users were created, `epf-build`/`DESIGNER` failed with `Пользователь ИБ не идентифицирован`.
- Root cause: authentication becomes mandatory once the user list is non-empty.
- Avoid: pass `-UserName Администратор` (empty password omitted) to every skill script that touches `lab2`.

### `agentmemory` MCP times out when its daemon is not running (2026-11-02)

- `opencode mcp list` showed `agentmemory failed: Request timed out`, while the `npx @agentmemory/mcp` process was alive (two PIDs under `~/.npm/_npx`).
- Root cause: the MCP server is a thin client for the agentmemory daemon (global config, `~/.config/opencode/opencode.json`, local `npx -y @agentmemory/mcp`); with no daemon listening, every call hangs until the timeout.
- Avoid: start the agentmemory daemon before relying on memory tools; the process being present in `pgrep` does not mean the MCP is usable. This is a global-config tool, not a repo one — do not add it to `.opencode/opencode.json`.

### `MCP_Toolkit` built-in server cannot start on macOS: the transport is a Windows DLL (2026-11-02)

- Opened `tools/MCP_Toolkit.epf` in lab1, clicked "Запустить сервер"; the form log showed `Не удалось подключить внешнюю компоненту MCPHttp` and the status stayed `Отключено`, nothing listened on 6003.
- Root cause: the add-in behind the built-in server is `MCPToolkit/Templates/MCPHttpTransport/Ext/Template.bin`, `PE32+ executable (DLL) x86-64, for MS Windows`. There is no macOS binary in the EPF, so `ПодключитьВнешнююКомпоненту` always fails. `MCP_Toolkit_x86.epf` is the 32-bit Windows build, same story.
- Avoid: do not expect `1c-toolkit` to work on this Mac. `ECONNREFUSED` on 6003 is permanent here; the item moves to the Windows host. Verify the platform of an EPF add-in with `file .../Ext/Template.bin` before blaming the port or the URL.
- Dump used to confirm: `python .opencode/skills/epf-dump/scripts/epf-dump.py -V8Path "$HOME/.local/1cv8/8.3.27.1606" -InfoBasePath "$PWD/lab1" -InputFile "$PWD/tools/MCP_Toolkit.epf" -OutputDir /tmp/mcp_toolkit_src`.

### `agent-desktop` gets nothing usable out of 1C on macOS (2026-11-02)

- `npx -y agent-desktop status` reports `accessibility: granted`, `screen_recording: granted`. `list-apps` sees the 1C process (`1cv8c`, then `1cv8t`).
- Thin client launched from the symlink (`~/.local/1cv8/8.3.27.1606/1cv8c`): `snapshot --app 1cv8c -i --compact` returns `ref_count: 3` — only `Close`, `Minimize`, `Full Screen` in English. Every control inside the window, including the open EPF form, is invisible.
- Thick client launched from the bundle (`open -a "/Applications/1cv8t.localized/8.3.27.1606/1cv8t.app" --args ENTERPRISE /F...`): the window opens and works, but `snapshot --app 1cv8t` fails with `TIMEOUT` / `global_window_inventory_unstable` / `APP_UNRESPONSIVE`, and the 1C window is absent from `list-windows`.
- Root cause: 1C draws through wxWidgets and does not publish an accessibility tree on macOS; the same reason `macos.Snapshot` never lists a 1C entry.
- Avoid: this is the fail branch of the probe — desktop input on macOS stays with the `macos` MCP (coordinates + screenshots). Do not retry `agent-desktop` for 1C on macOS and do not install `agent-desktop-macos`.

### 1C tree nodes are readable in UIA but not actionable (2026-11-02)

- `snapshot` on the lab1 configurator returned 73 refs with readable Russian names, but on the configuration tree `click` failed with `POLICY_DENIED` (`target is not actionable`) and `expand` / `focus` failed with `ACTION_FAILED`, `COM HRESULT 0x80004001` (`E_NOTIMPL`).
- Root cause: 1C advertises `ExpandCollapse` and `SetFocus` in its UIA providers but does not implement them; tree nodes expose no `Invoke` pattern, so there is no semantic delivery path.
- Avoid: drive the 1C tree with the keyboard (`press --app 1cv8t.exe right --headed` expanded a node, `down` walks siblings) and use `--headed` mouse for anything else. Semantic clicks do work on window chrome (`Закрыть` closed the configurator with `InvokePattern.Invoke`).

### PowerShell decodes child-process UTF-8 as CP866, corrupting Cyrillic JSON (2026-11-02)

- `agent-desktop ... | Out-File -Encoding utf8` produced a file whose Cyrillic names were double-encoded (`╨Ъ╨╛╨╜╤Д╨╕╨│...`), so `JSON.parse` still worked but every name was garbage.
- Root cause: PowerShell decodes the child's stdout using the console code page (CP866 here), not UTF-8, before re-encoding on write.
- Avoid: redirect inside `cmd` so bytes pass through untouched, and set the console encoding for readable output:

  ```powershell
  [Console]::OutputEncoding=[Text.Encoding]::UTF8
  cmd /c ".\node_modules\.bin\agent-desktop.cmd snapshot --app 1cv8t.exe -i --compact > %TEMP%\snap.json"
  ```

### A Cyrillic path typed into PowerShell does not resolve (2026-11-02)

- `Start-Process 'D:\Rubbish\Установка на чужие программы\1с\...\1cv8t.exe'` failed with "не удается найти файл", while the file exists.
- Root cause: the command text is encoded differently from the file system name, so the literal never matches.
- Avoid: resolve the path through the provider instead of typing it: `$exe = (Get-ChildItem D:\Rubbish -Recurse -Depth 6 -Filter 1cv8t.exe | Select-Object -First 1).FullName`.

### `npm install` without a `package.json` writes to `%USERPROFILE%` (2026-11-02)

- `npm install agent-desktop` in the repo root reported success, but `node_modules/` never appeared in the project and `npm ls` printed `user@ C:\Users\user` with the new dependency.
- Root cause: no `package.json` in the repo root, so npm walked up and installed into the home directory's own `package.json`. `C:\Users\user\package.json` is not part of this project.
- Avoid: the repo root now has a private `package.json`. Never run `npm install <pkg>` in a directory without one. If it happens again, `npm uninstall <pkg>` from the home directory to clean up.

### npm blocks `agent-desktop` postinstall, so the native binary is never linked (2026-11-02)

- `npm install agent-desktop` printed `npm warn install-scripts 1 package had install scripts blocked`, and `node_modules/agent-desktop/bin/agent-desktop-win32-x64.exe.sha256` existed while `.bin/agent-desktop.cmd` still pointed at the Node wrapper.
- Root cause: this npm enforces an `allowScripts` policy; third-party `postinstall` hooks are skipped unless approved.
- Avoid: run `npm run setup` (wraps `node node_modules/agent-desktop/scripts/postinstall.js`). It copies the bundled binary, writes the digest sidecar, and links the `.cmd` shim. Verify with `.\node_modules\.bin\agent-desktop.cmd version`.

### `npx skills add` fails with `schannel: failed to receive handshake` (2026-11-02)

- `npx skills add https://github.com/vercel-labs/agent-desktop` died at `Cloning repository` with an SSL/TLS handshake error, while `npx skills use` on the same URL worked (it downloads over HTTPS without git).
- Root cause: git on this host defaults to the `schannel` TLS backend, which fails against GitHub from this network.
- Avoid: run the install with git's OpenSSL backend in the environment, no global config change needed:

  ```powershell
  $env:GIT_CONFIG_COUNT='1'; $env:GIT_CONFIG_KEY_0='http.sslBackend'; $env:GIT_CONFIG_VALUE_0='openssl'
  npx -y skills add https://github.com/vercel-labs/agent-desktop --skill agent-desktop agent-desktop-windows --agent opencode -y
  ```

### `agent-desktop snapshot --app` wants the process name, not the window title (2026-11-02)

- `--app calc` and `--app ApplicationFrameHost.exe` both returned `APP_NOT_FOUND` / `WINDOW_NOT_FOUND` while Calculator was on screen.
- Root cause: `launch calc` starts the UWP host, but the window belongs to `CalculatorApp.exe`; the tool matches exact process identity from `list-apps` / `list-windows`.
- Avoid: read the name from `list-windows` (`app_name`) and pass that. UWP apps surface as their own `*App.exe`, not as `ApplicationFrameHost.exe`.

### Code in the methodichkas is images, not text (2026-10-24)

- `pdftotext` on `lab3/src/Лабораторная работа №3 (2022-2023).pdf` returns 89 KB of prose but zero `Процедура` / `Функция` matches, while `pdfimages -list` shows 16 embedded images.
- Root cause: the teacher exported listings as screenshots ("Microsoft: Print To PDF"), so the code exists only as pixels.
- Avoid: render the page with `pdftoppm -r 200 -png -f N -l N` and read the PNG with vision. Never reconstruct code from memory or from general 1C knowledge.

### No invented functionality beyond the methodichka (2026-10-24)

- Unauthorized additions are penalized even when they are correct 1C. Variant-driven edits (renaming a procedure, function, or variable, changing literals, small fixes to make the printed code run) are allowed.
- Root cause: course policy, not a technical constraint.
- Avoid: start from the printed code, change only what the variant requires, do not add objects/attributes/algorithms the methodichka does not describe, and stop to ask when a step is missing. See `.agents/rules.md`.

### `1Cv8Log` does not reveal base users (2026-10-24)

- `strings` on `1Cv8.1CD` and the event log found no usernames or passwords, so credentials could not be recovered from the files.
- Root cause: 1C stores password hashes and the log records authentication events without the login name in readable form.
- Avoid: do not dig through the base files for credentials. Open the base in the GUI and read the login dialog; the user list is shown in the dropdown.

### 1C hangs on login when the base has users (2026-10-24)

- `DESIGNER /F.../lab3 /DumpConfigToFiles` and `launch_client` hung for minutes with no output and no visible window. Process stayed alive at ~1% CPU, holding `1Cv8.1CD`.
- Root cause: the base requires authentication. 1C opens a login dialog and waits forever; batch mode does not fail, it blocks.
- Avoid: check credentials first. lab3 → user `admin` (Latin), lab2 → user `админ` (Cyrillic), both with an empty password. lab1 has no users at all.

### Empty password must be omitted, not passed as `/P ""` (2026-10-24)

- `DESIGNER /N admin /P "" /DumpConfigToFiles` hung on the login dialog, while `/N admin` alone worked.
- Root cause: an empty `/P` value is treated as "password not supplied", so the platform re-prompts. A wrong non-empty password also blocks on an error dialog instead of exiting.
- Avoid: never emit `/P` when the password is empty. Pass only `/N <user>`. The skill scripts already do this.

### 1C login dialog is invisible to the Accessibility tree (2026-10-24)

- `macos.Snapshot` listed no 1C window at all, even when the dialog was on screen and frontmost.
- Root cause: the app is launched from a bare binary, not a `.app` bundle, so LaunchServices does not register it (`lsappinfo info <pid>` shows `bundle path = [NULL]`, `Arch = !!none`). AX exposes only registered apps.
- Avoid: do not trust AX to find 1C dialogs. Take a screenshot (`screencapture -x`) and read it.
- Scope (2026-11-02): this was observed with a bare-binary launch. 1C is normally started from its `.app` bundle, which LaunchServices does register, so the AX tree may exist there. Unverified — the probe is in `.agents/todo.md`; do not treat this entry as proof that 1C is invisible to AX in general.

### Screen Recording permission is cached until the app restarts (2026-10-24)

- `screencapture` failed with `could not create image from display` even though Screen Recording was granted.
- Root cause: TCC attributes the request to the responsible process (`/Applications/OpenCode.app`, not `/usr/sbin/screencapture`) and caches the decision for the lifetime of that process. A grant made while OpenCode is running does not apply.
- Avoid: grant Screen Recording to `OpenCode.app` and fully restart OpenCode. The macOS MCP server is a separate process with its own grants.

### Testpilot refuses executables not named `1cv8` / `1cv8c` (2026-10-24)

- `launch_client(exe="/opt/1cv8t/8.3.27.1606/1cv8ct")` failed with `only 1cv8, 1cv8c may be launched`.
- Root cause: hard-coded filename validation in Testpilot.
- Avoid: use the symlinks `~/.local/1cv8/8.3.27.1606/1cv8` → `1cv8t` and `1cv8c` → `1cv8ct`.

### `/Applications/1cv8t.localized` contains stubs, not the platform (2026-10-24)

- Binaries under `/Applications/1cv8t.localized/8.3.27.1606/*.app/Contents/MacOS/` are ~52 KB and exit with code 255. Bundles are 948 KB total.
- Root cause: the training build keeps real binaries in `/opt/1cv8t/8.3.27.1606/`; the `.app` wrappers only locate them and fail outside their expected layout.
- Avoid: always launch `/opt/1cv8t/8.3.27.1606/1cv8t` or the `~/.local/1cv8/...` symlinks.
- Correction (2026-11-02): the bundle does work when LaunchServices starts it — `open -a "/Applications/1cv8t.localized/8.3.27.1606/1cv8t.app" --args ENTERPRISE /F<base>` brought up the thick client (window `Конфигурация`, both dialogs interactive). Only the direct invocation of `Contents/MacOS/<bin>` from a shell is the broken path, and it did not even return exit 255 in that test — it hung with no window. Use `open -a` when a bundle launch is explicitly needed; otherwise keep the `/opt/1cv8t` binary.

### `1c-toolkit` fails with ECONNREFUSED until the EPF is running (2026-10-24)

- `opencode mcp list` reports `1c-toolkit failed: ECONNREFUSED` on every start.
- Root cause: the remote MCP server at `http://127.0.0.1:6003/mcp` is served by `MCP_Toolkit.epf` inside a running 1C session. Nothing listens on 6003 by default.
- Avoid: treat this as expected until the user opens the EPF and starts the built-in server. Do not "fix" the URL.

### Skill auto-detection does not find this platform (2026-10-24)

- Skills auto-detect 1C via `glob('/opt/1cv8/*/1cv8')`; this machine has `/opt/1cv8t`.
- Root cause: the port assumes the standard Linux layout.
- Avoid: keep `v8path` in `.v8-project.json` and pass `-V8Path` when a script ignores the registry.

### Skill scripts invoke `python`, not `python3` (2026-10-24)

- Scripts failed with `command not found: python` on a clean macOS.
- Root cause: macOS ships no `python` alias.
- Avoid: the shim `~/.local/bin/python` → `/opt/homebrew/bin/python3.13` must stay in `PATH`.

### Testpilot macOS limitations (2026-10-24)

- `get_screenshot` and `desktop="isolated"` are unsupported on macOS; `tc_keyboard` works only on Windows/Linux.
- Root cause: platform support in Testpilot 1.9.1.
- Avoid: use `macos` MCP for input and screenshots, Testpilot only for the 1C protocol.

### `macos` MCP AppleScript mode has no Accessibility (2026-10-24)

- `Shell(mode="osascript")` fails with `-25211` ("Упрощенный доступ для osascript не разрешены").
- Root cause: the TCC grant belongs to the MCP server's uv-managed Python, not to `osascript`, and child processes do not inherit it.
- Avoid: use native tools (`Snapshot`, `Click`, `Type`, `Shortcut`); they work.

### Do not launch `1cv8st` / `1cv8ct` as clients (2026-10-24)

- `1cv8st` is the server, `1cv8ct` is the thin client; the designer is `1cv8t`. Running the wrong one produces exit 255 or a process that never connects.
- Root cause: confusing training-build suffixes (`t` = training, `c` = client, `s` = server).
- Avoid: `1cv8t` for designer/thick, `1cv8ct` for thin client, `1cv8st` only for a real server.

### `timeout` is not installed on macOS (2026-10-24)

- `timeout 120 python ...` → `command not found: timeout`.
- Root cause: GNU coreutils are not installed by default.
- Avoid: use `gtimeout` (coreutils) or a manual `for` loop with `kill -0`.

### Workspace journal writes and process inventory need escalation (2026-11-11)

- Lab5 preflight: sandboxed Add-Content to .agents/todo.md returned access denied; Get-CimInstance could not access CIM.
- Use an authorized escalated call for journal maintenance and process inventory. PDF rendering within lab5 works without escalation.


### Lab5 test fixture assumed a document attribute (2026-11-11)

- The external test failed before posting: object field ОбщаяСумма was absent.
- Read the actual metadata attribute names before building fixtures. This was a test harness error, not an application failure; transaction rollback kept fixtures out of the base.


### Lab5 inherited price register rejects duplicate copy rows (2026-11-11)

- A successful-payment fixture with two rows for the same copy failed with duplicate keys in Цены (period, copy, copy type).
- The inherited lab3 price handler creates one register row per document row. Do not rewrite that handler outside the lab5 methodichka scope. Verify duplicate requested quantities through the stock-refusal path; use distinct copies for successful multi-row payment verification.
- The initial receipt-total assertion was later traced to the test query omitting the document dimension; see the resolved virtual-table entry below.


### Balance virtual table omits unused dimensions (2026-11-11)

- The first lab5 fixture counted balance rows without selecting ДокументВзаиморасчетов; the platform aggregated them and the expected two-document assertion failed.
- Select the document dimension explicitly when verifying party detail, then total the returned rows. Application receipt quantities were correct; this failure was in the test query.


### Large inline screenshot payloads fail shell launch (2026-11-11)

- Saving Testpilot PNG data as a large inline PowerShell command returned an MXC packaged-app launcher error; no image file was written.
- Write the base64 payload to a workspace file with apply_patch, then decode via a short shell command. A test-client Table also refuses field get_text; use tc_table read_rows.


### Direct GUI launch does not provide a reliable batch wait (2026-11-11)

- The direct PowerShell invocation of 1cv8t /CheckModules returned before an output log existed (subsequent Get-Content exit 1).
- Use Start-Process -Wait -PassThru with separate /Out argument and explicit quoting; verify process exit and the log before claiming module validation.


### meta-compile emits empty register record-set modules (2026-11-11)

- Lab5 scope verification expected only two new XML files, but the platform round-trip also included two generated RecordSetModule.bsl files.
- Inspect the modules, require them to be empty, and include them in the expected generated file set. These are not extra application handlers.
