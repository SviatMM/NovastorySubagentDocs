# Sources

## Audit baseline

Repository: `SviatMM/novastory`

Branch: `main`

Commit: `2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`

The baseline was re-read from the remote `main` branch at the start of this task.

## Canonical/current documentation read

- `docs/HOME.md`
- `docs/PROJECT.md`
- `docs/backend/owner-logic/README.md`
- `docs/backend/OWNER_LOGIC_WORKBOOK.md`
- `docs/backend/README.md`
- `docs/backend/COVERAGE_LEDGER.md`
- `docs/backend/P1_P4_CURRENT_STATE.md`
- `docs/backend/CHAT_PROJECT_WORKSPACE_CURRENT_STATE.md`
- `docs/backend/OPERATION_CATALOG.md`
- `docs/backend/DATA_OWNERSHIP.md`
- `docs/backend/EXECUTION_MODEL.md`
- `docs/backend/SECURITY_AND_RECOVERY.md`
- `docs/architecture/PROJECT_AGENT_ENGINE_INTEGRATION.md`
- `docs/automations/AUTOMATIONS.md`
- `docs/systems/INTEGRATIONS.md`
- `goals/2026-09-13-backend-stage04-account-cloud-contract-preparation/STATUS.md`

## Current code inspected

- `src-tauri/src/lib.rs`
  - Tauri command registration
  - `StateStore::start_runtime_run`
  - `StateStore::start_runtime_run_in_transaction`
  - `StateStore::runtime_request`
  - `StateStore::finish_runtime_run`
  - `RuntimeCoordinator` start/run/cancel paths
  - provider/folder/backup/restore host commands
- `src-tauri/src/runtime.rs`
  - `PermissionProfileId`
  - `resolve_permission_profile`
  - `RuntimeConfiguration::validate_shape`
  - `RuntimeAdapter`
- `src-tauri/src/runtime_host.rs`
  - runtime resolution/model discovery
  - Project settings validation
  - engine receipt persistence
- `src-tauri/src/codex.rs`
  - App Server 0.153.3 boundary
  - disabled feature set
  - policy verification
  - `thread_start_params`
  - `turn_start_params`
  - engine request denial
- `src-tauri/src/projects.rs`
  - Project settings/resources/context snapshot
- `src-tauri/src/project_workspace.rs`
  - Project Sources
  - AgentTemplates / ProjectAgents
  - scoped Project file read
- `src-tauri/src/chat_workspace.rs`
  - Chat lifecycle and text attachments
- `src-tauri/src/work_tools.rs`
  - terminal/change-set substrate and explicit unavailable execution
- `src-tauri/src/output_blocks.rs`
  - normalized inert output projection
- `src-tauri/migrations/state/005_project_workspace.sql`
  - AgentTemplate/ProjectAgent/audit storage
- `src/desktop-api/index.ts`
  - renderer command wrappers and runtime event listener
- `src/app/App.tsx`
  - Chat submit requirements, model/permission UI, Project instructions, Agents and sources

## Verification boundary

This audit used GitHub repository content and current-state evidence. It did not execute the desktop application or rerun the repository test suites. Therefore historical test results in current-state docs are cited as repository evidence only, not as fresh tests performed by this audit.

Because the audit read the remote GitHub tree rather than an Owner-local checkout, pre-existing uncommitted local working-directory changes cannot be observed from this audit environment.
