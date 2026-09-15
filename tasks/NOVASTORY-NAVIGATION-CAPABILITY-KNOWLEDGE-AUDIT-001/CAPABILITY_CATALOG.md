# NovaStory Capability Catalog

Source pin: `SviatMM/novastory@2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`.

`Chat` below means the current `chat_only` model principal, not what a human can click. `Agent` means executable ProjectAgent, not merely its stored identity.

| Напрям | Можливість | Пояснення користувачу | Chat | Agent | Серверна дія / дозвіл | Реальний стан | Головне джерело / доказ | Заборонене припущення |
|---|---|---|---|---|---|---|---|---|
| Shell | navigation/Search | Основні екрани й пошук по loaded Projects/Conversations працюють | explain | none | renderer only | UI/live UI | CORE_SHELL; App.tsx; frontend matrix | Search не є глобальним Knowledge/file/web search |
| Chats | list/open | Чати durable і відкриваються | context only | none | list/get_conversation | WORKS | workspace current state; desktop API | UI read не є model tool |
| Chats | first send | Перший send materializes Conversation і run | returns model result after host start | none | start_draft_chat_run + runtime preflight | WORKS | P1_P4_CURRENT_STATE | модель сама не створює Chat host-call |
| Chats | rename | revision-safe rename | explain | none | rename_conversation + CAS | WORKS | frontend matrix; operation catalog | text request ≠ mutation |
| Chats | move Project | association change; warning UX incomplete | explain | none | move_conversation + CAS | PARTIAL | workspace current state | history snapshots не переписуються |
| Chats | trash/restore | Delete зараз recoverable Trash | explain | none | trash/restore/list_trash | WORKS | workspace current state | не називати permanent purge |
| Chats | export | Markdown transcript export exists | explain | none | export_conversation | PARTIAL | frontend matrix | не обіцяти повний attachment/archive policy |
| Context | text attachments | bounded UTF-8 attachments work | receives admitted content | none | select/list/revoke/delete attachment | WORKS text only | workspace current state | не дає arbitrary filesystem access |
| Context | image/voice | pipeline відсутній | explain | none | missing adapter/privacy flow | BOUNDARY | frontend matrix/current state | не стверджувати capture/transcription |
| Runtime | models | live discovery is authority | explain supplied/discovered config | none | list_runtime_models | WORKS current adapters | current state; runtime code | historical table ≠ current availability |
| Runtime | Power/Fast | effort and service tier separate, exact snapshot, no fallback | runs already-selected config | none | capability validation | WORKS | workspace current state | Fast ≠ reasoning Power |
| Permission | chat_only | модель має лише bounded text Chat authority | explain/read admitted context | none | host policy | WORKS | P1_P4 security | prompt/UI label не grant |
| Permission | broader profiles | labels exist, mappings unavailable/open | explain unavailable | none | missing policy bindings | BOUNDARY | SECURITY_AND_RECOVERY U-08 | Full access label не визначає права |
| Projects | list/create/open | durable Project identity works | explain | none | list/create/get_project | WORKS | PROJECT_MODEL; desktop API | Chat не може сам викликати createProject |
| Projects | rename | revision-safe | explain | none | update_project_title | WORKS | frontend matrix | archive/delete lifecycle не доведений |
| Projects | instructions/defaults | Project settings are CAS/revisioned | consumes admitted revision | none | get/update_project_settings | WORKS | P1_P4 | це не AgentInstructionSet |
| Projects | local sources | connect/repair/primary/detach/read local source | no dynamic file tool | none | ProjectSource commands + device read grant | WORKS bounded read | RESOURCE_MODEL; project_workspace | no write/process/index/remote grant |
| Projects | text Resources | bounded text_fragment context works | may receive in Project run | none | resource commands | WORKS | P1_P4 | Resource ≠ ProjectSource |
| Agents | list templates | built-in versioned templates are real | explain | none | list_agent_templates | WORKS bounded/static | DATA_OWNERSHIP | not complete account library |
| Agents | Add ProjectAgent | Project/domain identity add/reuse/conflict works | explain | identity only | add_project_agent | PARTIAL | workspace current state | identity ≠ work execution |
| Agents | Agent instructions | full AgentInstructionSet lifecycle missing | explain | none | missing | NOT_BUILT | engine integration U-07 | Project settings ≠ Agent instructions |
| Agents | Mission/Job | durable agent work not implemented | no | no | missing Mission/Job/Attempt + policy | NOT_BUILT | architecture reconciliation | Agent row ≠ running agent |
| Agents | Smart Subagent | no audited feature hit; U-19 open | explain | no | none | NOT_BUILT / post-commit exploration | U-19; repo search | do not claim launch/connect/script success |
| Results | Chat output blocks | versioned Chat blocks display/store | may produce normalized block | none | output block projection | WORKS/PARTIAL | output_blocks.rs/current state | displayed block grants no action |
| Results | Agent ResultReference | future separate result/publication path | explain | no | missing | NOT_BUILT | DATA_OWNERSHIP | no global Agent result store |
| Files | inspector read | granted Project file can open in inspector | no model read tool | no | read_project_file + grant | WORKS UI | inspector/project_workspace | UI read ≠ AI filesystem authority |
| Files | repo changes/Undo | review substrate exists; current Chat writes nothing; inverse executor absent | explain | no | preview/list; undo denied | PARTIAL/BOUNDARY | operation catalog | no working repository write/Undo |
| Terminal | command execution | typed substrate exists, executor absent | no | no | terminal API denies without executor/grants | BOUNDARY | operation catalog/current state | terminal button/API ≠ shell execution |
| Knowledge | Second Brain | product concept only | explain | no | none | NOT_BUILT/OPEN | KNOWLEDGE_SECOND_BRAIN | Chats/Resources ≠ Knowledge |
| Planning | Tasks/Goals | intent only | explain | no | none | NOT_BUILT/OPEN | TASKS_GOALS_PLANNING | no current Project Tasks tab |
| Automation | draft builder | local scenario draft UI works | may propose text graph only | no | renderer state | UI_ONLY | AUTOMATIONS; frontend matrix | draft ≠ saved automation |
| Automation | save/run/schedule | engine/storage/scheduler absent | no | no | none | BOUNDARY/NOT_BUILT | AUTOMATIONS U-09 | do not report execution |
| Mail | connect/read/send | renderer Mail only | no | no | connector absent | UI_ONLY/BOUNDARY | frontend matrix | Compose ≠ sent mail |
| Calendar | external event | local views/draft only | no | no | connector absent | UI_ONLY/BOUNDARY | frontend matrix | draft ≠ saved external event |
| Provider | OpenAI credential | Keychain status/save/disconnect works; no readback | sees availability only | no | provider commands + Keychain | WORKS | provider.rs/P1_P4 | never expose secret |
| Backup | create | bounded portable state backup | no | no | create_backup | WORKS bounded | SECURITY_AND_RECOVERY Stage01 | not encrypted cloud/full-device backup |
| Recovery | restore | staged validation, rollback fail-closed, source grants revoked | no | no | validate/confirm restore + gates | WORKS bounded | Stage01-03/security | device grants/credentials not restored |
| Audit | general action log | revisions/runtime/workspace subset only | explain | no | partial records | PARTIAL | SECURITY_AND_RECOVERY | workspace_audit ≠ full effect ledger |
| Account | NovaStory Auth | real user account/Auth not built | explain | no | none | NOT_BUILT | cloud README; Stage04 | Account popover ≠ authenticated account |
| Cloud | Supabase/sync | selected future component only | explain | no | none | NOT_BUILT/docs-only | cloud README; COVERAGE_LEDGER | no live replication/failover |
| Devices | second Mac runner | Owner goal/proposal only | explain | no | none | NOT_BUILT/PROPOSED | EXECUTION_MODEL U-18 | no enrolled runner/remote Jobs |
| Admin | admin panel/rights | documentation concept only | explain | no | none | NOT_BUILT/docs-only | ADMIN_CONCEPT U-17 | Project owner ≠ service admin |
| Future | website/Island/full Glass/CRM/IDE | roadmap/draft horizon | explain | no | none | HORIZON/OPEN | PRODUCT_HORIZON | do not invent release/module semantics |

## Rule

A live UI or typed host command is not automatically an AI capability. Conversely a ProjectAgent identity is not executable until a Mission/Job path and authority exist.