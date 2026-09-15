# NovaStory Navigation Map

Pinned to `SviatMM/novastory@2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`. Implementation claims below use the current
frontend control matrix, current-state docs, active `src/app/App.tsx`, desktop
API wiring and bounded verification records.

## Global shell

Current active `Surface` values in `src/app/App.tsx` are:
`home`, `mail`, `calendar`, `automation`, `scenario`, `chats`, `projects`,
`project`, `chat`, `settings`, `help`.

The expanded sidebar/header route is:

```text
NovaStory
├─ Search
├─ AI
│  ├─ Chats
│  └─ Projects
├─ Home
├─ Calendar
├─ Automation
└─ Settings
```

Mail is reached from the right-side header control rather than the main sidebar.
Notifications and Account are right-side popovers. Account can route to Settings
and Help. There is no permanent global Chat item and no global Agents item.

The fixed left titlebar cluster is Sidebar / Back / Forward. Sidebar collapse and
hover-peek are renderer behavior. Search and AI are shown in the expanded
sidebar. Search/`⌘K` currently searches commands plus loaded Projects and
Conversations; it is **not** evidence for Knowledge, filesystem, web or external
service search.

## Surface-by-surface map

| Part | User path | Visible actions | What opens | Current truth |
|---|---|---|---|---|
| Home | launch app → Home | Edit, disabled Add widget | local Home editing state | `UI_ONLY`; real widget/query/layout contracts are not built |
| Chats library | AI → Chats | search/filter, New Chat, open history, trash/history actions | standalone Chat draft or durable Conversation | list/open/new-first-send are `WORKS` |
| Chat | Chats → open/new → first send | `+`, Permissions, Model, microphone, Send/Stop; rename/move/export/delete; inspector/panes | durable Conversation, transcript, typed results | text Chat/runtime path `WORKS`; images/voice/terminal/write effects are boundaries |
| Projects library | AI → Projects | browse/search, create, open | Project workspace | identity/list/create/open `WORKS` |
| Project Overview | Project → Overview | recent Conversations, Project identity controls | current Project | `PARTIAL`; Overview is intentionally shallow |
| Project Conversations | Project → Conversations | open/create/use Project Chats and lifecycle actions | Project-linked Conversations | `WORKS` |
| Project Agents | Project → Agents | Add Agent, choose template/domain, list agent rows | ProjectAgent identity/list | add/reuse/conflict `PARTIAL`; no executable Mission/detail work surface |
| Project Repositories | Project → Repositories | connect/repair/make primary/detach local source; preserved text Resources | native folder picker, scoped source state | bounded local ProjectSource `WORKS`; remote/write/index/watch absent |
| Project Instructions | Project → Instructions | edit/save Project instructions/default runtime settings; saved comparison disclosure | revisioned Project settings | `WORKS` for Project settings; not AgentInstructionSet editing |
| Model picker | New/Open Chat → Model | discovered model, reasoning effort, service tier/Fast | runtime capability submenu | control/path `WORKS`; enabled models come only from live discovery |
| Permissions picker | Chat composer → Permissions | host-advertised profiles | selected policy snapshot | only `chat_only` execution is currently available; broader labels are boundaries |
| Attachments | Chat composer → `+` | choose/revoke/delete UTF-8 text file | native picker + attachment list | bounded text attachments `WORKS`; images unavailable |
| Right inspector | Chat → Inspector toggle / supported link | Conversation projection, public HTTPS, granted Project file, diff/text | right pane | bounded file/web/current-Chat reads `WORKS`; does not grant AI tools |
| Bottom terminal | Chat/Project → terminal toggle | availability/reason | bottom pane | typed substrate exists, but executor is absent: `BOUNDARY` |
| Structured results | Chat transcript | copy/view table, code/source, document, diff, result/reference | inert/versioned result blocks | Chat subset `WORKS/PARTIAL`; visual block grants no action |
| Mail | header Mail | local folders/search/compose; Connections route | Mail renderer surface | `UI_ONLY/BOUNDARY`; Send/external mailbox connection not implemented |
| Calendar | sidebar Calendar | Month/Week/Agenda, local event draft; connection route | Calendar surface | `UI_ONLY/BOUNDARY`; Save/external calendar is disabled |
| Automation library | sidebar Automation | tabs/search/create | focused Scenario Builder | library/editor shell `UI_ONLY` |
| Scenario Builder | Automation → create/open | name, Diagram/History, add Trigger/Action/Search/Transform/AI/Router, undo/zoom/fit, AI drawer | focused full-window builder | draft editing `UI_ONLY`; Save/Run/Schedule/config/AI-send are `BOUNDARY` |
| Settings: General | Settings → General | Light/Dark and ordinary local UI | settings content | theme persistence `UI_ONLY/LIVE UI state` |
| Settings: AI & Runtime | Settings → AI & Runtime | runtime refresh; OpenAI status/credential connect/disconnect | runtime/provider settings | discovery and Keychain-backed OpenAI credential operations `WORKS` |
| Settings: Connections & Devices | Settings → Connections & Devices | legacy folder connect/revoke; disabled Mail/Calendar and Mac runner controls | connection settings | local folder broker `WORKS`; Mail/Calendar/runner `BOUNDARY` |
| Settings: Security & Permissions | Settings → Security | displays current boundaries/default-deny concepts | settings content | mostly projection; no general policy editor |
| Settings: Data & Backup | Settings → Data & Backup | create backup, select/validate restore, confirm restore | native file flow/status | bounded local backup/restore `WORKS` with Stage 01–03 hardening |
| Settings: Keyboard Shortcuts | Settings → Shortcuts | record workspace search/new chat/sidebar shortcuts | renderer session bindings | `UI_ONLY`; not a durable host shortcut system |
| Account | top-right Account | session avatar; Companion boundary; Settings/Help | popover/routes | renderer state only; real NovaStory account/Auth not built |
| Notifications | top-right notifications | dismiss/open empty popover | popover | `UI_ONLY`; no durable notification system established |
| Help | Account → Help | static explanations/shortcut route | Help surface | `UI_ONLY` |
| Knowledge / Second Brain | no current user route | none | no live surface | `NOT_BUILT/OPEN`; concept only |
| Tasks & Goals | no current dedicated route; removed/deferred from Project tabs | none | no live planning module | `NOT_BUILT/OPEN` |
| General Results library | no current global result route | Chat transcript/output blocks only | Conversation-local results | Agent Result Store/ResultIndex `NOT_BUILT` |
| NovaStory account/Auth | Account popover exists, but no real account login route | avatar/session-like local UI, Settings/Help | renderer popover | account/Auth `NOT_BUILT`; Stage04 docs only |
| Cloud/Supabase | no current product route | none | no live cloud surface | `NOT_BUILT/documentation-only` |
| Second computer/Home Mac | Settings → Connections & Devices shows disabled boundary | unavailable runner control/reason | settings row | `NOT_BUILT/PROPOSED` |
| Admin panel | no desktop user route | none | separate future service-operator concept | `NOT_BUILT/documentation-only` |
| General audit log | no full user-facing audit route | bounded runtime/workspace/history projections only | no general ledger surface | `PARTIAL`; general effect audit `NOT_BUILT` |
| Future CRM/IDE/Files/Island/full Glass | no guaranteed current route | none unless a renderer placeholder separately exists | future area | `HORIZON/OPEN` |

## Removed/replaced navigation

The current accepted navigation supersedes older shapes:

- global AI launcher is **Chats + Projects only**;
- top-level/global **Agents** was removed; AgentTemplates are reached via
  `Project → Agents → Add Agent`;
- Chat is not a permanent sidebar item;
- Project tabs are exactly **Overview / Conversations / Agents / Repositories /
  Instructions**;
- Project **Tasks & Goals**, **Knowledge** and generic **Activity** were removed
  or deferred from Project navigation; this is not evidence that underlying
  future concepts are rejected;
- generic user-facing `Resources` in the Project workspace was replaced by the
  `Repositories` source surface, while preserved `text_fragment`
  `ProjectResource` records remain live internally and in a disclosure;
- wide global search was replaced by the sidebar magnifier/palette;
- current main appearance selector is Light/Dark; legacy stored `glass`
  normalizes to Light. Full-program Glass is a separate post-1.0 goal.

## Navigation does not equal capability

A reachable surface may be only renderer state. Conversely a registered host
command may be API-only and unreachable in the active UI. For an execution
claim, pair this map with `CAPABILITY_CATALOG.md` and
`AGENT_AND_CHAT_CAPABILITIES.md`.
