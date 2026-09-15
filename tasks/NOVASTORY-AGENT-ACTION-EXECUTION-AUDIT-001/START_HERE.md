# NovaStory Agent Action Execution Audit

Task: `NOVASTORY-AGENT-ACTION-EXECUTION-AUDIT-001`

Source repository: `SviatMM/novastory`

Audited branch: `main`

Audited commit: `2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`

Result repository: `SviatMM/NovastorySubagentDocs`

Result branch: `audit/NOVASTORY-AGENT-ACTION-EXECUTION-AUDIT-001`

## Bottom line

NovaStory currently has a real typed desktop backend boundary and a real Chat runtime, but it does **not** yet have a general model-to-NovaStory action/tool loop.

The current physical Chat path is:

```text
user
  -> React composer
  -> desktopApi.startChatRun/startDraftChatRun
  -> Tauri command
  -> host runtime preflight
  -> persist user Turn + assistant placeholder + RuntimeOperation
  -> RuntimeCoordinator worker
  -> OpenAI Responses or Codex App Server adapter
  -> model response/events
  -> persist normalized engine receipts/output/terminal state
  -> Tauri runtime event + durable reconciliation
  -> renderer shows the result
```

The crucial boundary is that the current Codex Chat adapter starts the engine with no dynamic tools, no workspace roots, read-only sandboxing, approvals denied and shell/delegation/plugin/MCP features disabled. The model therefore cannot request a NovaStory Tauri command such as `read_project_file`, `update_project_settings`, `add_project_agent`, terminal execution, Git operations or Automation execution. Those commands may exist for the UI, but that does not make them model-callable.

Current ProjectAgent support is identity/configuration substrate only. Built-in `AgentTemplate` rows and Project-scoped `ProjectAgent` creation/list/reuse/conflict logic are live, but `ProjectAgent.execution_available` is explicitly false and Mission/Job/checkpoint/subagent/publication execution remains unbuilt.

Current instruction support is also narrower than the requested future design. Project instructions are live, revisioned and snapshotted into each Project Chat run. There is no revisioned AgentInstructionSet lifecycle, no Agent-specific instruction editing operation, no standalone Chat pair of persistent instruction files, and no model-mediated instruction-change action.

Smart Subagent via GitHub is not implemented in NovaStory. The current operation catalog keeps GitHub connector operations as future/documented intent. A safe future Git flow is specified in `SMART_SUBAGENT_GIT_FLOW.md` without claiming implementation.

## Status vocabulary used in this audit

- **WORKS**: executable current path exists and has code/current-state evidence.
- **PARTIAL**: useful substrate exists, but the requested end-to-end capability does not.
- **UI ONLY**: renderer/control exists without the required backend execution path.
- **DESCRIBED**: accepted/proposed documentation exists but no complete implementation was found.
- **NOT FOUND**: no current implementation contract or code path was found in the audited boundary.
- **OPEN DECISION**: Owner/architecture choice is explicitly unresolved.

## Read next

1. `CURRENT_ACTION_ARCHITECTURE.md` — what physically executes today.
2. `CAPABILITY_ACTION_MAP.md` — capability-by-capability status and evidence.
3. `MODEL_TO_BACKEND_FLOW.md` — current Chat flow versus the missing model action loop.
4. `INSTRUCTION_EDITING_FLOW.md` — current instructions and future two-file contract.
5. `SMART_SUBAGENT_GIT_FLOW.md` — future GitHub handoff/review contract.
6. `IMPLEMENTED_VS_PLANNED.md` — compact truth table.
7. `CONTRADICTIONS_AND_GAPS.md` — conflicts, misleading surfaces and missing contracts.
8. `OWNER_DECISIONS.md` — decisions that should remain Owner-controlled.
9. `SOURCES.md` — audit boundary and exact evidence files/functions.

## Audit honesty boundary

This audit read the remote repository through GitHub at the exact commit above. It did not modify `SviatMM/novastory`, did not run the native desktop application, did not rerun Rust/renderer tests locally, and could not observe whether an Owner-local checkout had uncommitted changes. Historical/current-state test results are treated as repository evidence, not as tests rerun by this audit.
