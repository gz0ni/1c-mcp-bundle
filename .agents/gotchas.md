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

### Testpilot on Windows needs `1cv8c.exe`, the training build only ships `1cv8ct.exe` (2026-11-04)

- `1c-testpilot` validates the launcher name: `('1cv8.exe', '1cv8c.exe')` on Windows (`_runtime.py`). The portable 8.3.27.1508 training build has only `1cv8t.exe` / `1cv8ct.exe`.
- Root cause: same hard-coded check as on macOS; the `t` suffix of the training build is rejected.
- Avoid: hard links next to the originals (a symlink elsewhere would break DLL lookup): `bin\1cv8c.exe` → `1cv8ct.exe`, `bin\1cv8.exe` → `1cv8t.exe`. Created 2026-11-04 in `D:\Rubbish\никому не нужные проги\1с\8.3.27.1508\bin`.

### Project `.mcp.json` servers stay "Pending approval" in Claude Code (2026-11-04)

- `claude mcp list` showed `1c-testpilot ... ⏸ Pending approval`; the agent is not allowed to write `.claude/settings.local.json` itself (auto-mode classifier: self-modification).
- Root cause: Claude Code requires the user to trust servers from a project `.mcp.json`.
- Avoid: the user approves once, in an interactive `claude` started in `D:\Work\1C` or via the app's MCP prompt. Do not try to bypass it.

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
