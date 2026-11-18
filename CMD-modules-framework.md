# CitiMD - Module framework: development plan

> **Status:** planning (2026-11-17), revised with the product owner's answers. This document is **self-contained**: it was written only from the CitiMD phase 7 specification (`prompts-novo-app/en/16-phase7-module-system.md` and the master prompt) and the answers summarised below. API names here are CitiMD's own.
>
> **What I do not know:** I have not read CitiMD's current code. The starting point (section 2) is what phase 7 specifies; anything that may have changed in the implementation is marked **[verify]** and is closed in phase CM0.
>
> **Trust premise:** everything that runs inside the company and was already approved by the internal teams (network, GitHub, Copilot CLI…) is treated as **intranet**; what limits usage is the access the user and the machine already have. This plan therefore **does not restrict** what a module may reach: permissions and approval exist for **transparency and for things working well**, not to block.
>
> **Language:** the app is now **English-only**. There is no localization layer in this plan: module UI text is plain English, and nothing here asks for translated bundles, language packs or key parity.
>
> **Estimates:** hours of work, estimated, not measured. I assumed 15-20 h/week only to convert hours into weeks; adjust to your own pace.

## 1. What the answers decided

| Topic | Answer | Consequence in this plan |
| :--- | :--- | :--- |
| Stack | Desktop, Electron, same technology family as the prompts | No "discover the stack" phase; the framework extends the phase 7 module system |
| Documents | Text files (Markdown etc.), DOCX/PDF viewing; DOCX editing opens the default app | Module blocks inside a document only apply to Markdown, always with a readable fallback |
| Users and distribution | ≈ 15 users on your team; **you** install and update (portable zip); no shared network folders, only OneDrive folders users may share | Simple distribution: module package + zip + repository releases; no licensing service; **no network-share catalogue** |
| Network | Internal-only app; company-approved tools (GitHub, Copilot CLI) count as intranet | The host **does not block destinations**: reach is the machine's/user's. It handles proxy, TLS, limits and usage logging; no SSO for now |
| Identity | In future the user types only an ID (no password) | "Who" in logs is **self-declared**; not proof of identity |
| AI | Forbidden except authorised tools; GitHub Copilot CLI is approved. **No concrete use case today**; the idea is to use it later as the easiest way to bring AI into the app | The app **stays without native AI**. The Copilot CLI module is **parked** until there is a use case (section 6.6) |
| Module approval | Only you today; later other authors, with approval **to ensure things work well**, not to limit access | Approval is an **optional quality review** (checklist); no access gate |
| Logs / privacy | Local logs per instance; nothing on a central server | Debug log + local **audit log**, configurable retention, no content |
| First need | A read-only calendar module exists (GET JSON → calendar). Next ones: GET and POST; some with browser automation (Playwright/Selenium style) for systems without APIs | Reference modules = **migrate the calendar** + one module with POST + one with web automation |
| Repositories | CitiMD is on GitHub Enterprise; modules: one repository or several, undecided | Recommendation in section 6.12 |
| GET today | **Not yet confirmed** how the calendar module does its GET | Open item #1; how to check is in section 12 |
| CI | A runner probably exists on GitHub Enterprise but is unconfirmed and unused | Not on the critical path; local fallback script (section 7) |

## 2. Starting point and gaps

Phase 7 already delivers: a module `.zip` package with `manifest.json`, `main.cjs`, `renderer.js`, `workers`, `assets` and `locales`; a trust dialog on install; **a SHA-256 hash checked on every start**; failure isolation; clean removal through per-id tracking; `ModuleManager`; the renderer API (`commands`, `statusBar`, `settings.addSection`, `menus`, `views`, `sidebar`, `files.registerViewer`, `slots`, `diagnostics`, `profiles`, `i18n`, `invoke/onEvent`, `notify`, `workspace`, `fs`, `documents`, `search`); the main API (`ipc`, `prefs`, `process.fork`, core services); tooling (`module:build`, `module:pack`, `verify:core`, `hello-module`, a types package); the guide `docs/modules.md`.

**[verify] Since the app became English-only:** if the localization layer was removed, drop `i18n.*` and the `locales/` folder from the module package and from the `hello-module`; if it was only hidden, modules simply ship English strings and ignore the layer. Either way this plan treats module text as plain English.

What phase 7 **does not have** and your answers require:

| Need | Situation in phase 7 | Gap |
| :--- | :--- | :--- |
| **GET/POST requests to internal systems** | The renderer CSP only allows the app itself; there is no network API for modules. **[verify]** how the calendar does its GET today (probably Node `fetch` in `main.cjs`, with no host control) | A host network API: proxy, corporate TLS, limits, typed JSON, usage log, kill switch (without blocking destinations) |
| **Declared permissions** | `contributes` is only a catalogue of extension points | No informative `permissions` (network, secrets, web automation, programs, folders) in the manifest and the trust dialog |
| **Secrets** (credentials for other systems) | Nothing | Local vault (OS encryption), never exposed to the renderer or logs |
| **Settings** | `settings.addSection` with a free component; `modulePrefs[id]` | No declarative schema, no defaults shipped with the app (internal system URLs differ per environment), no locked fields |
| **Data and periodic refresh** | Nothing | Per-module local storage, "last known data", jobs while the app is open |
| **Browser automation** | Nothing (and the app has "no terminal at runtime") | A controlled way to drive a browser for systems without an API |
| **External programs** | `process.fork` only runs a utility from the module's own package | Run declared programs, no shell, with timeout and logging (**parked**, section 6.6) |
| **Dialog / picker / panels** | `views`, `sidebar`, `slots`; no module dialog, no bottom or right panel | New surfaces |
| **Icon and position** | `sidebar.register` with icon and panel; fixed order | Icon as a file, user reorder/hide, badge |
| **Module logs and audit** | Single `log`; app log with rotation | Per-module log + local audit trail with retention |
| **Approval** | Trust dialog + hash | Optional list of reviewed modules (by hash) |
| **Author experience** | `module:build/pack`, `hello-module`, types | Project generator, testkit, static validator, dev mode, expanded guide |

## 3. Principles

1. **The core stays 100% offline.** No core feature uses the network. Anything that talks to company systems is a **module feature**.
2. **The intranet is one thing.** Internal systems and already-approved tools (GitHub, Copilot CLI…) are treated as internal. The host does not filter destinations; the network, the proxy and the user's account decide what is reachable.
3. **No telemetry and no central log.** Logs and audit stay on each instance's machine.
4. **Transparency instead of blocking.** A module declares what it uses (network, secrets, automation, programs, folders); this shows in the install dialog and the Modules screen, is part of the hash, and helps review. It is not a turnstile.
5. **Degrade, never break.** Offline shows the last known data; without the module, blocks and notes stay readable.
6. **Protect the user from mistakes, not from bad faith.** Actions that change data in another system ask for confirmation (kit default), credentials live in the vault and out of logs, and a module failure does not take the app down.
7. **Honesty:** the hash proves the package did not change after install; it does not prove authorship; modules run with the app's privileges. With a single author that is enough; isolation (6.11) is for **robustness** once there are more authors.

## 4. Archetypes and surfaces

| Archetype | Example | Surfaces |
| :--- | :--- | :--- |
| **Internal data view (read-only)** | Current calendar | tab (view), side mini-panel, status item, command, icon badge |
| **Operations panel (GET + POST)** | Create/update records in an internal system | tab with a form, **confirmation dialog**, audit, error handling and retry |
| **Web automation** | Read/fill a system without an API | command + progress panel, visible browser window when needed, notification |
| **External program** (parked) | GitHub Copilot CLI integration | panel with output, command on the selection |
| **File importer** | `.csv`/`.json` from a synced folder (for example OneDrive) into notes | settings section, scheduled job, generated `.md` notes |

| Surface | Use for | Do not use for |
| :--- | :--- | :--- |
| Tab (`views`) | long-running work (calendar, dashboard) | quick confirmations |
| Side panel | lookups next to the document | long forms |
| Bottom/right panel (new) | continuous output, details of the selected item | main navigation |
| **Dialog** (new) | a short decision or form, POST confirmation | displaying data meant to be consulted |
| Quick picker (new) | keyboard choice (system, environment, calendar) | forms |
| Own window (module route) | something floating outside the app | what fits in a tab |
| Status item | state in one sentence, one-click action | long text |
| Document header / title bar | an action on the open file | global actions |
| Notification (toast) | end of a background task | a question that needs an answer |

## 5. Manifest 0.2 (additive to phase 7)

All new fields are optional; no `permissions` means "no new capability".

```jsonc
{
  "id": "citi.calendar",
  "name": "Calendar",
  "version": "0.2.0",
  "apiVersion": "0.2",
  "icon": "assets/icon.svg",                       // file in the package; single colour, currentColor
  "requires": ["ui.dialog", "net"],                // host features; without them the module is disabled with a clear reason
  "activationEvents": ["onView:citi.calendar.month", "onCommand:citi.calendar.today", "onSchedule"],

  "permissions": {
    "net": {
      "hosts": ["system-x.internal:8080"],         // informative: shown in the dialog and in the audit
      "methods": ["GET"],                          // informative: what the module does (add POST etc. when used)
      "purpose": "Read appointments from System X" // plain English, shown to the user
    },
    "vault": false,                                // store credentials
    "jobs": true,                                  // scheduled tasks while the app is open
    "web": null,                                   // browser automation: { "hosts": [...] }
    "exec": null,                                  // external programs: { "programs": [...] }
    "fs": { "write": ["workspace:Calendar/**"] }
  },

  "contributes": {
    "commands":   [{ "id": "citi.calendar.today", "title": "Calendar: Go to today", "shortcut": "Ctrl+K A" }],
    "views":      [{ "id": "citi.calendar.month", "title": "Calendar" }],
    "sidebar":    [{ "id": "citi.calendar.mini", "title": "Agenda", "icon": "assets/icon.svg", "defaultOrder": 40 }],
    "statusItems":[{ "id": "citi.calendar.next", "side": "right" }],
    "settings": {
      "title": "Calendar",
      "properties": {
        "baseUrl":     { "type": "url", "title": "System X address", "default": "http://system-x.internal:8080", "lockable": true },
        "syncMinutes": { "type": "integer", "title": "Refresh every (minutes)", "min": 5, "max": 240, "default": 15 },
        "weekStart":   { "type": "enum", "title": "Week starts on", "values": ["mon", "sun"], "default": "mon" }
      }
    }
  }
}
```

Rules:
- `contributes` and `permissions` are **checked against the code** by `module:validate` (registering something undeclared, or using the network outside the host APIs, fails the build). They exist so the manifest tells the truth about the module.
- The **trust dialog** (already in phase 7) now also shows: extension points, hosts, methods, programs, web automation and write folders.
- A manifest change in an update → a different hash → the install dialog appears again, showing what changed.
- `apiVersion` 0.1 keeps loading (today's calendar does not have to change to keep working): define the compatibility rule in CM0 (a "0.1 and 0.2" window) instead of requiring an identical minor.

## 6. New API areas

Names are CitiMD's own. An area is "done" only when it meets the definition of done in section 8.

### 6.1 Network (`host.net`)

Every module request goes through the **main process** (the renderer CSP does not change).

```ts
host.net.request({ connection?, url?, method, path?, query?, headers?, body?, as?: 'json' | 'text' | 'bytes', timeoutMs? })
  : Promise<{ status, headers, data }>
host.net.json<T>(path, { method?, body?, schema? }): Promise<T>       // validates the response when given a schema
host.net.connection(id): { baseUrl, status() }                        // declared, user-configurable connections
```

- **Reach = the machine's reach.** The host **does not block destinations**: if the user and the network can reach it, the module can. The manifest's `hosts` and `methods` are informative (dialog, audit, `module:validate`). If it ever proves useful, a preference `modules.enforceDeclared` could refuse anything undeclared (a safety net against misconfiguration); it stays off.
- **Proxy:** use the system proxy (honouring the Windows bypass list for internal addresses). **TLS:** verification always on, using the operating system certificate store (corporate CA); never "ignore certificate errors".
- **Limits:** 30 s timeout, response up to 25 MB, concurrent requests per module, exponential backoff on 5xx/429 (in scheduled jobs).
- **Per-environment configuration:** `baseUrl` is module configuration, not code; the default can come from a defaults file shipped with the app (6.7) and be **locked**.
- **Visibility and control:** the install dialog shows `purpose`; *Settings → Modules → Network* lists per module the hosts and last activity (time, method, host, status, bytes; **never** body, headers or query) and has "block this module" and a **global "no network" switch**.
- **Incoming connections** stay out: the app opens no port; it only makes outgoing requests or reads files.

### 6.2 Secrets and identity (`host.vault`, `host.user`)

- `host.vault.set/has/delete/use`: stores credentials for **other systems** (for calls and for web automation) using Windows per-user encryption. The renderer never receives the value (only `has`). `vault.use(name)` hands the value to `host.net`/`host.web` to **inject** it without the module code seeing it. It never goes into logs, audit, exports or preference backups; it is **not portable** between machines (document it: each user configures their own).
- `host.user.id()`: the ID the user entered (if the app has that feature), **read-only**, to stamp the audit and prefill fields. With no password it is identification, not authentication: nothing security-sensitive should rely on it.
- No SSO for now; it is out of scope and out of every permission.

### 6.3 Local data and "last known data" (`host.store`)

- Per-module collections (key/value and simple queries) in the module's data folder, with an **update timestamp**; the UI says "updated X ago" and stays useful offline.
- Optional text index so the content enters the app's **search** (the existing name and content index), always with the source identified.
- **Optional materialisation** as `.md` notes in the workspace (a `generated: true` header and the generated part between markers) for what the user wants to search, link and take elsewhere.
- Removing a module asks whether to delete its internal data too; **user files are never deleted**.
- **OneDrive-synced folders:** workspaces and importer folders may live in OneDrive. Files "online-only" can be slow to open and sync can lock or half-write files; use the phase 7 core services (`cloudProviderOf`, `withLockRetry`, atomic writes) rather than raw file calls.

### 6.4 Scheduled jobs (`host.jobs`)

`host.jobs.define({ id, everyMinutes, onlyWhenOnline, run(ctx) })` and `runNow(id)`. Runs **only while the app is open** (no background service); respects "no network"; does a catch-up refresh on start; requires `permissions.jobs`.

### 6.5 Browser automation (`host.web`)

For systems **without an API**. Two options, decided in spike S1 (section 9):

| Option | How | Pros | Cons |
| :--- | :--- | :--- | :--- |
| **A. The host drives a window of the app itself** (recommended for simple flows) | The host opens a browser window inside the app, with a per-module isolated session (optional persistent cookies), and offers: `open(url)`, `waitFor(selector)`, `click`, `fill`, `fillSecret(selector, vaultName)`, `text`, `table`, `download(target)`, `screenshot`, `close` | No new dependency or extra binary; works offline/internal; does not depend on an installed browser; credentials stay in the host | Fewer features than a test framework (multiple tabs, tracing, advanced waits) |
| **B. Playwright in the module** (complex cases) | The module bundles `playwright-core` and uses the **system Edge/Chrome** (`channel: 'msedge'`), no browser download, running in a module utility process | Complete, mature API; Apache-2.0 | Needs Edge on all 15 machines; weight; more surface; credentials pass through module code (benefits from isolation) |

Selenium is **not recommended**: it needs a browser driver distributed separately, version-locked to the browser, and more pieces to maintain.

Common rules: `web.hosts` declared (informative); **one flow at a time**; a **visible** mode for debugging and for the user to take over MFA/captcha ("take control" and "continue"); downloads only to a module temp folder; screenshots are not stored by default (they may contain sensitive data); timeout per step and per flow; errors return diagnostics **without** page content. Because selectors break when the target system changes, each automation module keeps **recorded test pages** (HTML) and tests its selectors against them.

### 6.6 External programs (`host.exec`) and the Copilot CLI — **parked**

There is no concrete Copilot use case today. The idea is to use it later as the easiest way to bring AI in. This section is a sketch so that decision stays cheap; **nothing here is scheduled** (phase CM7 and spike S2 are parked until a use case exists).

- `host.exec.run(program, args, { cwd?, timeoutMs, onOutput })`: runs programs declared in `permissions.exec.programs` (informative, shown in the dialog), **no shell** (argument vector, avoids quoting mistakes and accidental injection), environment inherited from the user, working folder inside the workspace by default, timeout, streamed output and guaranteed termination when the app closes. The audit records program, exit code and duration (not arguments or output).
- Company-approved programs (GitHub CLI, Copilot CLI…) count as internal: **no "data egress" warning and no extra confirmation** for using the vendor's cloud.
- **When a use case appears, spike S2 checks, on the company-approved version:** (1) how it authenticates (GitHub login inside the CLI; where the token is stored); (2) whether a **non-interactive** mode exists that a program can drive, or only an interactive mode (which would need a pseudo-terminal and conflicts with "no terminal at runtime"); (3) what it reads on its own in the folder where it runs; (4) which folder it should run in to see the intended context. Tentative design: a command such as "Ask Copilot about the selection/file", a panel with the answer, working folder = the workspace, audit without content. The app does not gain native AI; if more than one AI module appears, a shared "provider" feature can be discussed then.

### 6.7 Settings (schema, defaults and locks)

- Declarative `contributes.settings`: `string`, `url`, `integer`, `number`, `boolean`, `enum`, `multiselect`, `path`, `list`, `secret`, `group`, with `title`, `default`, limits, `when` and `description`, all plain English. The host draws the section, validates, applies defaults and includes the fields in the "Modified" view and in "Reset". A free-component section remains possible, including combined.
- **Defaults shipped with the app:** `data/module-defaults.json` (next to the portable executable) sets initial per-module values (for example each environment's `baseUrl`) and can mark fields `locked: true` (shown disabled, with the source). Since you update the zip, this is the way to give all 15 users the same configuration.
- Scope: app by default; per workspace only if the field asks for it (`scope: 'workspace'`).

### 6.8 Surfaces, icon, shortcuts and UI kit

- `host.ui.dialog.open({ id, title, size, dismissible, component, params })` → resolves with the result; focus trapped, `Esc` closes, focus returns, named `role="dialog"`; one at a time. `host.ui.confirm({ title, message, danger? })` for confirmations (every POST that changes data). `host.ui.pick(...)` for the quick picker.
- `host.panels.register({ id, area: 'bottom' | 'right', ... })`; `host.views.open(id, { params })`; view state restored on reopen (silently ignored if the module is not active).
- **Icon:** an SVG file in the package (single colour, sanitised on install, follows icon size and theme). The user reorders, hides and groups activity-bar items (keyboard included); the module declares only `defaultOrder`/`defaultVisible`; profiles and always-visible items keep precedence. `host.sidebar.setBadge(id, number|dot|null)`.
- **Shortcuts:** default in the manifest, reassignable on the shortcuts screen (with the source shown), two-key chords allowed, conflicts resolved in favour of the core with a notice.
- **UI kit** (`Button`, `Field`, `Select`, `Table`, `Tabs`, `EmptyState`, `Spinner`, `Badge`, `DateField`): native look, theme and accessibility; **no hard-coded colours** in modules (the validator flags them).

### 6.9 Logs and audit (privacy / LGPD)

Everything **local, per instance**; nothing goes to a server.

| Log | Content | Where | Retention |
| :--- | :--- | :--- | :--- |
| **Module debug log** | `host.log` messages (level, module, short text) | `userData/logs/modules/<id>.log`, size rotation | by size (like the app log) |
| **Audit** | events: install/review/disable, permission shown, network request (time, module, method, host, status, bytes), web automation (domain, flow, result), external program (name, exit code), write action (POST) | `userData/logs/audit.jsonl` (one line per event) | **configurable days** (suggested default: 180, to confirm with the responsible area) |

Rules: **no content** (request/response bodies, document text, credentials, query strings); the "user" field is the self-declared ID (6.2) and the documentation says so; an "Export logs" button on the Modules screen for support; the "Erase local data" command **warns** that it also erases the audit within retention and asks for confirmation. A privacy section in the README: what is recorded, where and for how long.

### 6.10 Commands and events

- Module commands and shortcuts already exist (phase 7). Add `commands.run(id, args)` so a module can call another's command, and **events** a module can subscribe to without being loaded all the time: `document.opened`, `document.saved`, `workspace.changed`, `connection.changed`, `job.finished` (via `activationEvents: ["onEvent:document.saved"]`).
- Commands that **change data in another system** (POST etc.) use `host.ui.confirm` by default (a kit recommendation, to prevent accidental clicks); the author may skip it when the flow already has its own confirmation.

### 6.11 Review, distribution and isolation

**Today (only you):** no gates. You install the package; phase 7's trust dialog and hash stay as they are. Distribution to the 15 users: the app zip with the modules folder, or the module package as a GitHub Enterprise release asset. **There are no shared network folders**, and OneDrive is not a good place for the module catalogue or for a list the app must trust (sync conflicts, placeholder files, partial writes), so keep catalogues in the repository or inside the zip.

**When there are other authors — approval for quality, not to limit access.** A review checklist before releasing a module to the team:
- `module:validate`, tests and `typecheck` green in CI (or locally, section 7); no hard-coded colours;
- uses `host.net`/`host.vault` instead of its own network and secret handling; errors handled and "offline" works;
- manifest permissions match the code; logs without content; clean removal.
- Optional: `approved-modules.json` (id, version, hash, reviewer, date), **stored in the repository and shipped inside the app zip** (`data/`), plus the preference `modules.requireApproved` for **whoever wants** to require review (off by default). It records *what was reviewed*; the future "access group" is the group of reviewers.
- Digital signatures stay out of scope.

**Isolation (CM10) for robustness:** the module's `main.cjs` runs in a **utility process**, so a hung, looping or leaking module **cannot take down or freeze the app** and the host can terminate it. It makes sense once there is more than one author or heavier automation modules; it is not a prerequisite for your use today.

### 6.12 Repositories

Recommendation: a **`citimd-modules` monorepo** on GitHub Enterprise while there is one author and the API is 0.x:

```
citimd-modules/
  packages/module-api/   # types (or publish to the Enterprise package registry later)
  packages/module-kit/   # typed REST client, dates/timezones, backoff, simple parsers
  packages/module-testkit/
  modules/calendar/  modules/<other>/ ...
  templates/             # module:new templates
  approved-modules.json  # optional, see 6.11
```
- Benefits now: an API change and the update of every module land in the same PR; one CI; `module-kit` reuse without publishing a package.
- Per-module releases with tags such as `calendar-v0.2.0` (the `.zip` artefact + hash in the release notes).
- **When to split a module into its own repository:** when it has another maintainer, a different lifecycle, or secrets/data that people who do not maintain it should not see. `CODEOWNERS` can restrict review per folder before that.
- The decision blocks nothing: a module is a folder with `manifest.json`; moving it to another repository is copying the folder.

## 7. Tooling and author experience

- `npm run module:new -- <id> --template calendar|post-panel|web-automation|external-program`: a ready folder with manifest, `main.ts`, `renderer.tsx`, tests, tsconfig and CI. Templates are small real modules.
- `module-kit`: a typed REST client over `host.net` (base URL, JSON, errors, pagination, backoff), dates and timezones, table helpers.
- `module-testkit`: an in-memory fake host; a fake HTTP server and fake clock; ready-made checks ("activate and deactivate leaves nothing behind", "manifest declares what it uses", "secrets never in log/audit/IPC", "module works offline with stale data").
- Dev mode: `{{ENV_PREFIX}}_MODULES_DEV=<dist folder>` loads without packaging (unpackaged app only) and reloads on change.
- `module:validate`: manifest, permissions vs real use, hard-coded colours, direct network/exec use outside the host.
- Guide (`docs/modules.md`, expanded): a 15-minute quickstart, concepts, surfaces, network and secrets, web automation, external programs, logs and audit, review, recipes, type reference, a pre-release review checklist.
- **CI on GitHub Enterprise [verify a runner exists]:** typecheck, tests, `module:validate`, build; package into a tagged release. **Fallback if there is no runner:** `npm run verify` runs the same steps locally and the release is built by hand. Nothing in the plan depends on CI.

## 8. Definition of done for an API area

1. Registered by module id, removed on deactivation; an id without the module prefix is refused.
2. A host method **and** a type in the `module-api` package (`typecheck` checks them against the real host).
3. A manifest entry (`contributes`/`permissions`), validated and, for permissions, shown in the trust dialog.
4. A "no modules" test (nothing visible) and a "register and deactivate leaves nothing" test.
5. Keyboard and accessibility; no colour outside the tokens.
6. An audit event defined when the area touches network, secrets, automation or external programs.
7. Guide updated and `verify:core` passing.

## 9. Roadmap

Prefix **CM**. Hours are estimated, not measured; spikes S1 and S2 reduce the uncertainty of phases 6 and 7.

| Phase | Deliverable | Hours | Depends on |
| :--- | :--- | ---: | :--- |
| **CM0** | Check the baseline in the code (**how the calendar does its GET**, state of `contributes`, user ID, logging, whether the i18n layer was removed); decisions in section 11; `apiVersion` compatibility rule; spec 0.2 | 10 | - |
| **S1** | **Web automation spike:** proof of concept of options A and B against a real internal system (login, read a table, fill and submit, download), using the team machines' Edge; decide | 12 | CM0 |
| **S2** | *Parked.* **Copilot CLI spike:** authentication, non-interactive mode, what it reads; decide feasibility and design | 10 | a concrete use case |
| **CM1** | Manifest 0.2 (informative `permissions`, `requires`, `activationEvents`, icon); trust dialog with permissions; Modules screen; basic `module:validate`; optional reviewed list | 20 | CM0 |
| **CM2** | `host.net`: proxy, corporate TLS, limits, typed JSON, configurable connections, Network panel and "no network" switch; **per-module debug log and audit** with retention | 35 | CM1 |
| **CM3** | Declarative settings, `module-defaults.json` with locked fields, `host.vault` (secrets) | 25 | CM1 |
| **CM4** | Dialog/confirm/picker, bottom and right panels, views with params and restore, icon file, order/hide/badge, reassignable shortcuts, UI kit | 40 | CM1 |
| **CM5** | `host.store` (data and last known data, optional index, materialisation) and `host.jobs` | 20 | CM2 |
| **CM6** | `host.web` (option chosen in S1), `web` permission, visible mode, single flow, test pages | 45 | CM2, CM3, S1 |
| **CM7** | *Parked.* `host.exec` and the **Copilot CLI module** | 35 | CM2, S2 |
| **CM8** | `module:new`, `module-kit`, `module-testkit`, dev mode, guide | 35 | starts after CM1, closes at the end |
| **CM9** | **Migrate the calendar** to API 0.2; a reference module with **POST** (form, confirmation, audit); verification script on a clean install | 30 | CM2-CM5, CM8 |
| **CM10** | **Module main isolation** (robustness): utility process, guaranteed termination, bridge to the host | 35 | CM2, CM6; **recommended before other authors** |
| | **Total** (with the parked phases) | **≈ 352** | |
| | **Total without the parked phases (S2, CM7)** | **≈ 307** | |

**Minimum slice** (calendar migrated + one safe GET/POST module, no web automation, no Copilot, no isolation): CM0 (10) + CM1 (20) + CM2 (35) + CM3 partial, settings and defaults only (15) + CM4 partial, dialog/confirm and icon file only (20) + CM8 partial, `module:new` and dev mode only (15) + CM9 (30) ≈ **145 h**, about 8-10 weeks. Order: CM0 → CM1 → CM2/CM3 → CM4 → CM9 (milestone), then S1/CM6 as real need appears, with CM10 before anyone else writes modules. Copilot work starts only when a use case exists.

**Acceptance criteria**

| Id | Criterion |
| :--- | :--- |
| AC-CM01 | App without modules: no module item appears; `verify:core` passes |
| AC-CM02 | The current calendar (apiVersion 0.1) keeps loading; once migrated, it behaves the same |
| AC-CM03 | Every module request goes through `host.net` (proxy, system TLS, limits) and shows in the audit; `module:validate` flags network use outside the host |
| AC-CM04 | The install dialog shows declared hosts and methods; an action that changes data asks for confirmation by default |
| AC-CM05 | "No network" cuts every module; the calendar shows the last data with "updated … ago" |
| AC-CM06 | A secret never appears in logs, audit, preferences, renderer IPC or exports (a test that scans) |
| AC-CM07 | With `requireApproved` on (optional), a package missing from the list or with a different hash does not activate; changing the manifest changes the hash |
| AC-CM08 | Audit records network, automation, programs and activations **without content**; respects retention; "Erase local data" warns |
| AC-CM09 | Reordering/hiding icons persists, works from the keyboard and respects profiles |
| AC-CM10 | Removing a module removes its icon, settings, shortcuts and jobs; materialised notes and blocks stay readable |
| AC-CM11 | (when CM7 is done) an external program runs with no shell, with a timeout, and is terminated when the app closes; the audit shows it without arguments or output |
| AC-CM12 | A new author, using `module:new`, gets a module with a view, a command and a setting working in 15 minutes |
| AC-CM13 | (after CM10) a hung or looping module does not freeze the app and can be stopped from the Modules screen |

## 10. Recipes

### 10.1 Migrate the Calendar (read-only)
Replace the current GET with `host.net.json` and a response `schema`; `baseUrl` becomes configuration (default in `module-defaults.json`, locked if you want); refresh with `host.jobs`, storing in `host.store` with a timestamp; show "offline, updated … ago" on failure; icon file with a "today's events" badge; command and shortcut `Calendar: Go to today`; permissions `net` (GET) and `jobs`. No POST.

### 10.2 Operations panel with POST
A form in a tab; the button calls `host.ui.confirm` with a summary of what will be sent; `host.net.json(..., { method: 'POST' })`; error handling with a manual retry (no hidden queue); audit of the action (without the body); `net` permission with `methods: ["GET","POST"]`; confirmation before sending.

### 10.3 Web automation on a system without an API
Command "Import from System Y": opens the flow in a visible window the first time (the user resolves MFA if any), `fillSecret` with the vault credential, reads the table, stores it in `host.store`, notifies. Selector tests against recorded HTML. `web.hosts` and `vault` permissions.

### 10.4 Copilot CLI (parked sketch)
Text selection → command "Ask Copilot" → `host.exec.run` with the Copilot CLI executable and the selection as input → answer in the bottom panel → audit without content. Built only when a use case exists.

## 11. Decisions and risks

| Id | Decision / risk | Recommendation |
| :--- | :--- | :--- |
| D1 | Web automation: host window (A) or Playwright (B) | Start with S1; default A for simple flows, B only if the proof shows the need |
| D2 | Is the Copilot CLI viable without an interactive terminal? | Parked; run S2 only when a use case exists; if only interactive mode exists, weigh a pseudo-terminal against "no terminal at runtime" |
| D3 | Audit retention | Suggest 180 days; confirm with the responsible area |
| D4 | Reviewed list (`approved-modules.json`) | Only when there are other authors; kept in the repository and shipped in the zip (no shared folder exists) |
| D5 | Monorepo vs one repository per module | Monorepo now (6.12) |
| D6 | `apiVersion` compatibility | A "0.1 and 0.2" window with the 0.1 surface frozen |
| R1 | **A faulty module can hang the app** (same process) | Code review and tests; CM10 (utility process) before other authors |
| R2 | **The hash does not prove authorship** | With one author and internal distribution it is enough; sign the reviewed list only if it ever matters |
| R3 | User ID without a password: self-declared audit | Document it; it is a record, not access control |
| R4 | Fragile web automation and credentials for other systems | Vault, visible flow, recorded-HTML tests, one flow at a time, a notice that the target may change |
| R5 | Scope (≈ 307 h without parked phases) | Minimum slice of ≈ 145 h; web automation only when a real need appears |
| R6 | No CI runner on GitHub Enterprise | Verify; otherwise the verification script runs locally (`npm run verify`) |
| R7 | OneDrive-synced files (online-only placeholders, partial syncs) | Use the core file services; test the importer against an online-only file |

## 12. Open questions (these close CM0)

1. **How does the calendar module do its GET today?** Not confirmed yet. To check in the CitiMD repository: (a) search the module's `main.cjs` and `renderer.js` for `fetch(`, `http.request`, `https.request`, `axios`, `net.request`; (b) look at the `connect-src` in the app's CSP (`index.html` or the window's session headers) for the endpoint's host; (c) check whether the request is made from the renderer (needs the CSP to allow it) or from the main process via an IPC `invoke`. The answer decides whether CM2 is a migration of existing behaviour or a new capability, and whether the CSP was loosened for it (it should be put back).
2. Does the "user ID" feature already exist in the app, or will it still be created?
3. Which systems without an API are the first automation targets, and do they have MFA/captcha? Is Edge installed on all 15 machines?
4. What log retention does the responsible area accept?
5. Does your GitHub Enterprise have an Actions runner? (Unconfirmed; not on the critical path.)
6. Do the internal IP ranges and domains the systems use go through the proxy or bypass it?
7. Was the localization layer (`i18n`, `locales/`) removed from the app and from the module API, or only hidden?
8. When a Copilot (or other AI) use case appears: what should it do first? That defines S2.
