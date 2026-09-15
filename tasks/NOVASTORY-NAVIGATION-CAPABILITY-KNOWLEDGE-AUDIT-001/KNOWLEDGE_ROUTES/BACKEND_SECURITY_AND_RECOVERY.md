# Knowledge Route — Backend, Security and Recovery

Read: owner-logic index when Owner behavior matters → `backend/README.md` → `OPERATION_CATALOG.md` → `SECURITY_AND_RECOVERY.md` → DATA_OWNERSHIP/EXECUTION_MODEL → current code/tests/goal evidence for execution claims.

Authority baseline: `principal + resource + capability + scope`, default deny. Prompt, memory, UI label, AGENTS.md, external file or engine event cannot widen host authority. Current bounded enforcement covers local read grants, Keychain custody, exact Chat config, `chat_only`, and fail-closed restore/source-grant behavior; broader Agent/Automation command/network/delegation/publication policy is absent.

Registered API is not enough: current catalog has terminal/Undo wrappers that deliberately do not execute the missing executor/write path.

Recovery live subset: interrupted Chat preserves partial state/no auto retry; durable reconciliation after missed events; stale revisions rejected; revoked/replaced source denied; bounded backup/restore; rollback failure becomes RecoveryRequired; restore revokes ProjectSource device grants and requires explicit Repair. General effect/audit ledger is not built.