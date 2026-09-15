# Contradictions and Gaps

## 1. Host commands exist, but model actions do not

The repository has a broad typed Tauri command surface. The current Codex Chat adapter nevertheless starts with no dynamic tools and a read-only Chat-only policy. Any statement that “the model can execute NovaStory commands because the commands exist” would be false.

## 2. ProjectAgent exists, but Agent execution does not

`project_agents` rows, built-in templates and Project/domain uniqueness are implemented. The runtime-facing ProjectAgent object explicitly reports execution unavailable. Mission/Job/checkpoint/subagent/publication records are absent.

## 3. Permission UI is broader than live host authority

Several permission labels exist in the interface, but only Chat-only resolves successfully. The current-state documentation correctly treats the others as unavailable. UI vocabulary must not be mistaken for permission implementation.

## 4. Structured approval blocks are not approvals

Output blocks can represent an approval-request-shaped result, but the current Chat host denies unsupported engine capability requests. There is no general approval settlement system.

## 5. Terminal API is not terminal execution

Typed terminal session commands and storage exist. The host explicitly reports no contained executor, so creation/input/resize/cancel execution paths do not establish a usable terminal capability.

## 6. Change review is not file mutation

Recorded change-set review/preview infrastructure exists. Current Chat does not create such changes, and Undo cannot perform inverse writes. This is evidence/review substrate only.

## 7. Project instructions are not the requested Agent/Chat instruction architecture

Project instructions are revisioned and used by Project Chat. There is no separate two-record Chat instruction system and no revisioned AgentInstructionSet implementation.

## 8. Attachment-only first message gap

Backend attachment snapshots can exist for drafts, but renderer submission requires non-empty text. Therefore “first message consists only of a file/photo” is not currently supported end to end. Images are also unavailable in the current attachment path.

## 9. Knowledge and search terminology exceeds implementation

Knowledge operations appear in the backend catalog as future/documented operations. No current knowledge engine, index or model-callable retrieval path was found.

## 10. Automation has meaningful UI but no engine

The renderer can manipulate scenario UI state, while Save/Run/Scheduling and durable execution remain unavailable because the backend engine is not implemented.

## 11. GitHub/Smart Subagent is future work

GitHub appears in integration/operation planning, but no current NovaStory GitHub connector or Smart Subagent task/result watcher was found.

## 12. Account identity is not current action identity

Current local resource grants use a fixed application principal. Account/Auth/cloud identity is explicitly not built in Stage 04. A future action gateway must not pretend the current fixed principal is the final Owner/Agent identity model.

## 13. Current reliability does not cover general effects

Chat event deduplication and terminal settlement are live. Generic action idempotency, external-effect reconciliation and durable approval settlement remain future contracts.
