# NovaStory Main Map

This file is intentionally short. It answers **where to go**, not the whole
question.

Source pin: `SviatMM/novastory@2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`.

| User intent / entity | Open this audit route | First canonical NovaStory source |
|---|---|---|
| What is NovaStory / current delivery state | `KNOWLEDGE_ROUTES/CURRENT_STATE_AND_DELIVERY.md` | `docs/HOME.md` → `docs/PROJECT.md` → `docs/firmament/DELIVERY-MOC.md` |
| Home, global shell, navigation, settings | `KNOWLEDGE_ROUTES/DESIGN_AND_FRONTEND.md` | `docs/product/CORE_SHELL.md` + `docs/design/FRONTEND_CONTROL_MATRIX_2026_09_08.md` |
| Projects and Project lifecycle | `KNOWLEDGE_ROUTES/PRODUCT_AND_PROJECTS.md` | `docs/product/PROJECT_MODEL.md` |
| Chats / Conversations / composer / results | `KNOWLEDGE_ROUTES/CHAT_AND_CONVERSATIONS.md` | `docs/backend/CHAT_PROJECT_WORKSPACE_CURRENT_STATE.md` |
| Agents, AgentTemplates, ProjectAgents, models | `KNOWLEDGE_ROUTES/AGENTS_AND_MODELS.md` | `docs/architecture/PROJECT_AGENT_ENGINE_INTEGRATION.md` + current-state docs |
| Files, ProjectSources, Resources, knowledge | `KNOWLEDGE_ROUTES/FILES_RESOURCES_AND_KNOWLEDGE.md` | `docs/product/RESOURCE_MODEL.md` + `docs/backend/DATA_OWNERSHIP.md` |
| Permissions, host operations, backup/recovery, audit | `KNOWLEDGE_ROUTES/BACKEND_SECURITY_AND_RECOVERY.md` | `docs/backend/README.md` → `SECURITY_AND_RECOVERY.md` |
| Automation, Mail, Calendar, connectors | `KNOWLEDGE_ROUTES/AUTOMATION_AND_INTEGRATIONS.md` | `docs/automations/AUTOMATIONS.md` + current frontend matrix |
| Account, Auth, cloud, devices, second computer | `KNOWLEDGE_ROUTES/CLOUD_ACCOUNT_AND_DEVICES.md` | `docs/backend/cloud/README.md` + `docs/product/UNRESOLVED_DECISIONS.md` |
| Can Chat/Agent actually do the requested action? | `AGENT_AND_CHAT_CAPABILITIES.md` | current host/runtime evidence |
| Exact user navigation path | `NAVIGATION_MAP.md` | frontend control matrix + active `src/app/App.tsx` |
| Cross-product capability status | `CAPABILITY_CATALOG.md` | capability map + current-state + operation catalog |
| Conflicting/stale statement | `CONTRADICTIONS_AND_STALE_SOURCES.md` | use newest authoritative evidence, not filename status alone |
| Lost context / changed topic / search rule | `RETRIEVAL_RULES.md` | this audit routing layer |
| Owner choice still needed | `OWNER_DECISIONS.md` | `docs/product/UNRESOLVED_DECISIONS.md` + Owner Logic |

## Dependency expansion

Do **not** automatically expand into account/Auth, cloud, permissions or
integration docs.

Examples:

- “Build an automation from local NovaStory state” starts in Automation. Open
  Integrations only if a step uses an external service. Open Account/Auth only
  if that connector requires account identity.
- “What models are available?” starts in Agents & Models and live runtime
  discovery. Cloud docs are irrelevant.
- “Read a file in this Project” starts in Files/Resources. Security becomes a
  dependency because a concrete ProjectSource grant is required.
- “Send email” starts in Automation & Integrations; the current result is a
  boundary before any Auth design matters because no mail connector exists.
