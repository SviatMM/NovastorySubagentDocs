# Model to Backend Flow

## Current implemented Chat flow

1. React submits a message through `desktopApi.startChatRun` or `startDraftChatRun`.
2. Tauri resolves the exact runtime configuration and accepts only the current Chat-only permission profile.
3. NovaStory persists the user turn, assistant placeholder, runtime operation, configuration snapshot, policy snapshot and context snapshot before execution.
4. `RuntimeCoordinator` starts the selected Responses or Codex adapter.
5. The Codex adapter starts an ephemeral App Server thread with no dynamic tools or workspace roots and with read-only policy.
6. Engine events are normalized and persisted before renderer notification.
7. Terminal state is persisted and the renderer reconciles from durable state.

## Missing general action loop

No current implementation was found for:

```text
model receives NovaStory action schemas
-> model emits typed action request
-> NovaStory resolves actor and scope
-> NovaStory validates policy/revision
-> one host operation executes
-> durable action receipt is stored
-> sanitized action result returns to the same model turn
-> model continues
```

The current Tauri command surface belongs to the renderer/host boundary. It is not passed to the model as tools.

## Permission facts

Current `ChatOnly` resolves to read-only, no tool access, no commands, no writes, no environment access and no workspace roots. The other visible permission profiles are explicitly unavailable until additional host contracts exist.

## Reliability facts

Current Chat supports one active run per Conversation, durable run snapshots, receipt sequencing, duplicate detection, cancellation, interrupted settlement on restart and durable reconciliation after missed renderer events. These guarantees apply to Chat runtime events, not to a future general side-effect/action engine.
