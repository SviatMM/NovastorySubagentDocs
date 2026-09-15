# Sources

Audit pin: `SviatMM/novastory` branch `main`, commit `2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1` (`docs: prepare account and cloud contract decisions`, 2026-09-13). Audit date 2026-09-15. Local uncommitted worktree is not observable via GitHub.

## Canonical routing/product
`docs/HOME.md`, `PROJECT.md`, `INDEX.md`, `firmament/PROJECT-MOC.md`, `firmament/DELIVERY-MOC.md`, `product/CORE_SHELL.md`, `PROJECT_MODEL.md`, `CHAT_PROJECT_WORKSPACE_SPEC.md`, `CHAT_SESSION_MODEL.md`, `RESOURCE_MODEL.md`, `CAPABILITY_MAP.md`, `KNOWLEDGE_SECOND_BRAIN.md`, `TASKS_GOALS_PLANNING.md`, `BUILT_IN_APPS.md`, `PRODUCT_HORIZON.md`, `UNRESOLVED_DECISIONS.md`, `ADMIN_CONCEPT.md`, `architecture/PROJECT_AGENT_ENGINE_INTEGRATION.md`, `operations/PERMISSIONS_BOUNDARY.md`, `automations/AUTOMATIONS.md`, `systems/INTEGRATIONS.md`, `backend/owner-logic/README.md`.

## Current implementation/reconciliation
`backend/README.md`, `P1_P4_CURRENT_STATE.md`, `CHAT_PROJECT_WORKSPACE_CURRENT_STATE.md`, `OPERATION_CATALOG.md`, `DATA_OWNERSHIP.md`, `EXECUTION_MODEL.md`, `SECURITY_AND_RECOVERY.md`, `COVERAGE_LEDGER.md`, `design/FRONTEND_GUIDE.md`, `design/FRONTEND_CONTROL_MATRIX_2026_09_08.md`, `product/FRONTEND_AUDIT_2026_09_10.md`, Stage04 STATUS.

## Cloud/account planning
`backend/cloud/README.md`, `backend/cloud/AUTH_DEVICES_SECRETS.md`. Used only as documentation-only planning, never live cloud proof.

## Direct code/wiring
`src/app/App.tsx` and active app components/tests; `src/desktop-api/index.ts`/types; Rust host `lib.rs`, projects/project_workspace/chat_workspace/runtime/codex/provider/inspector/output_blocks modules. Operation/coverage ledgers guard against treating API-only terminal/Undo wrappers as execution.

Exact repository search for `Smart Subagent` returned no result at audited default branch; U-19 supplies broader concept context.

Post-commit project conversation (2026-09-14) was used only to record newer Owner login and Smart Subagent intent, never as implementation evidence.

Excluded as authority: old archives/ZIPs, derived `docs/context/*`, superseded frontend maps, completed prompts/handoffs, screenshots as runtime proof, public web as replacement for private repository truth.