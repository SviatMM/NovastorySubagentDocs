# Current Action Architecture

Audit source: `SviatMM/novastory@2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`

## 1. What actually executes today

NovaStory currently has two different execution planes that must not be conflated:

1. **Renderer-to-host actions**: React calls typed functions in `src/desktop-api/index.ts`; those call registered Tauri commands implemented in Rust. This plane can create/update NovaStory records, manage Chat lifecycle, connect/read bounded Project sources, manage local provider credentials, backup/restore, and query partial work-tool substrate.
2. **Model runtime**: a Chat run is sent to either `openai.responses.v1` or `codex.app-server.v1`. The model returns text/structured engine events. In the current Chat policy, the model has no NovaStory action tools.

The existence of plane 1 does not expose it to plane 2.

## 2. Current Chat execution chain

### Step A — user initiates a run

`src/app/App.tsx` submits only when the composer has non-empty text, the runtime is ready and there is no active run. It calls:

- `desktopApi.startChatRun(...)` for an existing Conversation; or
- `desktopApi.startDraftChatRun(...)` for first send.

The renderer chooses an explicit `RuntimeConfiguration` containing adapter, model, reasoning effort, service tier and permission profile.

### Step B — desktop API crosses the Tauri boundary

`src/desktop-api/index.ts` wraps `invoke(...)` with typed methods. `startChatRun` invokes `start_chat_run`; `startDraftChatRun` invokes `start_draft_chat_run`.

The Tauri handler list in `src-tauri/src/lib.rs` registers the Chat commands alongside many UI-facing operations such as Project/source/Agent/settings/output/work-tool commands.

### Step C — host validates selected runtime policy

`src-tauri/src/runtime_host.rs` resolves the exact adapter/model pair. No cross-model fallback is admitted.

`src-tauri/src/runtime.rs` validates `RuntimeConfiguration`. Only `PermissionProfileId::ChatOnly` currently resolves. Its host snapshot is:

- sandbox: read-only;
- approval policy: never;
- network access for tools: false;
- tools: false;
- commands: false;
- writes: false;
- environment access: false;
- workspace/path grants: empty.

`AskForApproval`, `ApproveForMe`, `FullAccess` and `Custom` are explicitly unavailable.

### Step D — durable state is created before execution

`StateStore::start_runtime_run_in_transaction` in `src-tauri/src/lib.rs` atomically creates:

- the user `Turn`;
- the streaming assistant `Turn` placeholder;
- one `RuntimeOperation` with adapter/model/configuration/policy/context snapshots.

It rejects a second active `RuntimeOperation` in the same Conversation.

For first send, draft Conversation materialization, attachment adoption and run creation occur in the same state transaction.

### Step E — run context is assembled

`StateStore::runtime_request` loads durable Conversation history.

For a Project Chat it prepends the current snapshotted Project instructions as a system message and adds selected Project text Resources as untrusted reference data. Chat text attachments are also added as untrusted reference data. The snapshot was persisted when the run was admitted, so later edits do not silently rewrite an already admitted run.

Limits are enforced rather than silently truncating context.

### Step F — worker starts the adapter

`start_chat_run` / `start_draft_chat_run` create a background worker through `spawn_runtime_worker`.

`RuntimeCoordinator::run_with_adapter`:

1. persists runtime-started state;
2. builds the `RuntimeRequest`;
3. calls the selected adapter;
4. persists each accepted engine event before UI notification;
5. settles exactly one terminal state;
6. removes the in-memory cancellation registration.

### Step G — Codex path is explicitly tool-less

`src-tauri/src/codex.rs` starts Codex App Server 0.153.3 over private stdio JSON-RPC.

The current configuration disables shell, unified exec, multi-agent, plugins, MCP-style integrations, remote control, memories and other authority-bearing features.

`thread_start_params` sends:

- `dynamicTools: []`;
- `runtimeWorkspaceRoots: []`;
- `selectedCapabilityRoots: []`;
- read-only sandbox;
- approvals `never`;
- base instruction stating tool access is unavailable.

`turn_start_params` again sends empty workspace roots and a read-only/no-network tool policy.

If the engine sends a server request requiring a capability, `Session::deny_request` rejects it and the run fails with a policy-denied classification. Therefore there is no current path:

```text
model -> NovaStory action request -> host command -> tool result -> model continues
```

### Step H — model/engine events are normalized and persisted

`runtime_host.rs::persist_engine_event` stores normalized receipts with sequence and duplicate checks. Exact source-event duplicates are ignored; conflicting duplicates fail. Engine text is projected into the assistant Turn in the same transaction as the receipt.

`src-tauri/src/output_blocks.rs` projects selected safe event fields into inert `OutputBlock` records. Unknown result kinds/versions are displayed without actions. An `approval_request` output block is a denied/inert receipt, not a live approval mechanism.

### Step I — terminal settlement and UI update

`StateStore::finish_runtime_run` persists the assistant Turn and `RuntimeOperation` terminal state and creates a terminal result/error block.

Tauri emits `novastory://runtime-event`. The renderer listens to these events and also refreshes durable state to reconcile missed events. UI delivery is therefore not the source of truth.

## 3. Identity and permission checks currently present

### Current principal

The local host uses the fixed principal `novastory.core-ui` for current folder grant logic. Current repository docs explicitly say account/Auth/Supabase are not built. Therefore there is no implemented signed-in Owner/Agent principal model for a general action gateway.

### Resource scope

Current file reads are host-mediated and scope checked:

- legacy folder reads require an active read grant for a binding;
- Project Source reads require Project/source/binding-generation match;
- root device/inode identity is rechecked;
- absolute paths, traversal and symlink escapes are denied;
- only regular bounded UTF-8 files are admitted for the relevant readers.

These are useful primitives for a future action gateway, but the Chat model cannot invoke them today.

### Revision scope

Project title/settings/resources, Chat lifecycle and Project Source mutations use expected revision/CAS checks in current bounded paths. Stale writes fail instead of overwriting newer state.

## 4. History, idempotency and interruption

Current Chat has meaningful reliability behavior:

- one active run per Conversation;
- persisted configuration/policy/context snapshots;
- durable engine receipt sequencing;
- duplicate source-event detection;
- output-after-terminal rejection;
- cooperative cancel plus durable terminal settlement;
- unfinished queued/streaming operations become `interrupted` on app reopen;
- no automatic Chat retry after interruption;
- missing local worker registration normalizes to a durable failure;
- renderer reconciles from durable state after event gaps.

This is **Chat runtime reliability**, not generic effect idempotency. Generic `EffectIntent`/`EffectReceipt`, approval leases, external connector idempotency and Agent Mission resume remain documentation/future work.

## 5. Current renderer-callable action families

The current registered/wrapped host surface includes, among others:

- Chat: list/get/start/cancel, rename, move, trash/restore, attachments, export;
- Projects: list/get/create/rename, settings and text Resources;
- Project Sources: list/connect/repair/primary/detach/read;
- models/runtime discovery;
- built-in AgentTemplates and ProjectAgent add/list;
- normalized output blocks;
- OpenAI credential status/save/disconnect;
- local backup/restore;
- legacy folder binding/read;
- work-tool capability/session/change-set APIs.

The work-tool APIs deliberately demonstrate why registration is not capability proof: terminal creation/input/resize/cancel fail closed because no contained executor exists, and Undo fails because no host-enforced write/inverse-patch executor exists.

## 6. Current answer to the requested physical chain

The requested universal chain is **not implemented**.

Today the closest real chain is:

```text
user writes message
  -> renderer selects runtime/configuration
  -> host validates Chat-only policy
  -> host persists run + immutable snapshots
  -> host sends context to model
  -> model generates response/events only
  -> host validates/persists normalized receipts
  -> host settles run
  -> UI displays durable result
```

Missing middle section:

```text
model discovers NovaStory actions
  -> model emits typed action request
  -> host authenticates acting principal
  -> host resolves resource/action scope
  -> host decides approval requirement
  -> host executes one typed operation/effect
  -> host persists idempotent receipt
  -> tool result is returned to the same model turn
```

That missing section is the central architecture gap for NovaStory Agent and action-capable NovaStory Chat.
