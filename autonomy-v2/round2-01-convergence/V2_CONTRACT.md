# Autonomous Execution V2 — minimum contract

**Normative proposal, not an installed runtime.** MUST/MUST NOT describe the proposed V2 lane. Source IDs and baseline identities are in `SYNTHESIS.md`. Implementing this contract needs separate authorization.

## 1. Scope and deployment

Keep `AUTONOMOUS_BLOCK` as the execution mode. Add a versioned controller-backed engine, not a replacement goal taxonomy. The smallest supported topology is one trusted local controller, one local journal, one implementation job at a time, and separately assigned review. Admission, scheduling, gates, verifier and effect gateway are modules in that process.

A run binds `completion_scope` to the actual accepted request: an increment or a named goal/release slice. A large request MUST NOT silently become “first increment only.” One `CURRENT_INCREMENT` remains active for implementation. A later increment may be admitted automatically through existing checks when outcome-level delegation already covers it; otherwise the precise missing authority remains visible. Completing an increment does not satisfy the parent run's remaining obligations. There is no recursive orchestrator hierarchy.

The proposed first adapter drives a pinned `codex exec --json` invocation, captures its structured lifecycle stream, and resumes only the explicit session ID recorded for this run. Official documentation describes `turn.completed`/`turn.failed` events and explicit session resume. These interfaces do not themselves implement this contract. An App Server adapter is a later alternative, not a second required runtime; its `turn/completed` also carries failed/interrupted statuses. [W1–W2]

The controller MUST own the actual user-facing result channel. Direct interactive Codex sessions outside this launcher remain policy-assisted, not silently upgraded to V2. A running CLI child, progress log, or optional completion hook is not a durable supervisor.

## 2. Trust and action boundary

The trusted boundary consists of the installed controller/adapter, host isolation mechanism, grant administration path and protected journal/evidence store. Model messages, repository instructions, patches, tool output and worker-written reports are untrusted inputs. Trusted code is pinned outside worker write authority; a worker cannot update the controller used to judge its own patch.

Use an isolated scratch checkout for implementation. Worker edits there are proposals under a bounded `SANDBOX_WORK` grant, not canonical acceptance. Only the controller integrates an exact patch into its owned canonical worktree, runs acceptance jobs, changes authoritative state, or publishes. Gate **each canonical action/effect** and each sandbox job's resource/operation envelope. In-job containment enforces that envelope; post-hoc JSON observation does not replace containment.

Before enabling Stage 2, prove that worker code cannot write the journal, authority rows, controller code, trusted receipt files, canonical checkout or sibling worktrees; cannot inherit external-write credentials; and cannot use unmediated network/tool routes to reach those resources. Runtime model transport must be limited to approved destinations and data policy; repository programs do not inherit its authentication environment. Tests and build hooks execute untrusted code and require the same protection.

An OS-enforced sandbox or privilege-separated worker account may implement this boundary. A worktree, prompt, file mode under unrestricted identical privileges, or matching before/after Git status cannot. If the chosen host/runtime cannot enforce the boundary, admission into the **mechanically enforced** lane fails. Retain a clearly weaker policy-assisted mode; do not call it Stage 2 complete. [R-D11, R-A50–A51]

An action proposal contains: `action_id`, `run_id`, `expected_state_version`, `controller_generation`, `admission_revision`, `grant_revision`, `operation_class`, canonical resource identity, exact target, payload/artifact digest, prerequisites, capability generation and cost reservation request. Unknown operations or fields with invalid types are rejected.

The action gate permits dispatch only when current authority contains the exact action, relevant capabilities are fresh, ownership and expected artifact basis match, prerequisites hold, budget is reserved, and the adapter's recovery class is admissible. Worker-authored “approved” text is never a grant.

The local linearization point is the controller's serialized authorization-and-handoff to its executor. A revocation committed before that point denies dispatch. A later revocation stops subsequent dispatches and triggers reconciliation of already handed-off effects; it does not promise their reversal. Dispatch and Owner control events share this ordering. No database transaction spans a remote response wait. Compensation is a new, separately authorized action.

## 3. Sealed obligations, not mutable pass flags

### 3.1 Admission authority

Extend the existing product-local admission with a sealed obligation manifest. Store a protected copy/digest in the journal; keep product meaning in product-local canonical documents. AI OS stores reusable schemas/rules, not duplicate product requirements.

Each immutable revision records:

| Field group | Required content |
|---|---|
| Identity | Run, goal/slice, active increment, repository identity, source baseline, manifest revision and digest. |
| Accepted basis | Requirement-index version/digest; accepted decision references; authenticated intent/delegation reference; hard constraints and prohibited-substitution references. |
| Obligations | Unique obligation/predicate IDs; requirement references; required evidence class; verification recipe and coverage; prerequisites; applicability and its admitted reason. |
| Delivery | Exact required artifact/commit/ref/external outcome, or explicit initially non-required dimensions. |
| Control | Effective grant reference; policy/review versions; finite limits; allowed technical substitutions; known blocked dependencies. |

Reject duplicate JSON keys, duplicate IDs before lookup construction, unknown classifications, null required values, malformed arrays, dangling dependencies, cycles, contradictory checkpoint partitions and unreadable accepted sources. An unsealed model proposal is not an admitted manifest.

The controller derives the required set from accepted admission and required operation/delivery rules. `applicable_predicates` supplied by a worker is at most a comparison input; it cannot select obligations. Every admitted material outcome needs nonempty coverage. Documentation-only work has document/evidence obligations, not fabricated implementation or live tests. A complete set of passing tests without coverage for an accepted requirement is insufficient.

Semantic extraction from free prose is not magically deterministic. Preserve the original accepted references and require an independent admission/coverage check where interpretation is needed. Missing or ambiguous product meaning is resolved from canonical sources or a material Owner decision; it is not silently omitted to obtain a valid schema.

### 3.2 Replanning and applicability

A replan may alter tasks, order, hypotheses and eligible approaches. It MUST retain the run's admitted obligation IDs and required delivery/live dimensions. Record a before/after obligation diff, affected evidence and authority consequences. Reslicing retains deferred obligations in the parent run. It cannot turn a blocked original outcome into a completed one.

`NOT_APPLICABLE` is an admitted classification with a reason derived from the outcome, not a runtime state available to the worker. Missing login, failed tests, unavailable reviewer, budget exhaustion and host limits never establish N/A. A real acceptance change requires an authorized superseding admission; history records which former commitment was changed, not that it was fulfilled.

Predicate state is `UNPROVEN`, `SATISFIED`, `REFUTED`, `STALE`, or `UNCERTAIN`. Wait/blocker records are orthogonal. Only the verifier may set `SATISFIED`. `repair_pass`, `rereview_accept`, `normal_push`, worker booleans and prose are requests/claims to evaluate, not truth-producing events. [A07, A09; R-D01–D02]

## 4. Minimal durable records and ownership

Use one journal for execution decisions. Suggested logical tables are `runs`, `events`, `grants`, `admissions`, `predicates`, `actions`, `receipts`, `findings`, `effects`, `budget_entries`, and `outbox`. These are tables/modules, not separately deployed services. Derived indexes/state are updated with events in the same transaction and can be rebuilt; they are not competing authorities.

Each event has unique ID, run ID, increasing sequence, producer identity, controller generation, causation ID, schema version and payload digest. Each state transition checks expected version. Uniqueness constraints cover received event IDs, active attempt, logical effect identity and final receipt identity. An outbox entry and its causative transition commit together.

Use local transactional storage with verified durability settings; SQLite WAL with `synchronous=FULL` is the proposed default. Do not put the active database on a shared/synchronized/network filesystem. Check every commit acknowledgement; a free-space precheck or WAL alone is not proof that later writes will succeed. Local restart durability is conditional on the storage stack, not a promise against disk or host loss. [W3]

Retain exact accepted patch/tree bytes, command observations and compact required logs. Reuse immutable Git objects and protected digest-checked files. Publish a receipt file by write/flush, immutable rename and directory durability acknowledgement before committing its database reference. A crash may leave an unreferenced file; a missing/corrupt referenced file invalidates evidence. Hashes prove identity, not truth or authorization. No separate content-addressed service is required.

One process-held OS lock protects the local controller/store; resource claims prevent another managed run from writing the same canonical worktree/ref. A lock-file name, PID or heartbeat timestamp is not ownership. Do not unlink or steal a live lock after a timeout. On a valid restart, increment the local generation and reject stale proposals, callbacks and patch integration requests.

Before restarting a worker or taking over mutation, establish that the old controlled process/job is quiescent. If that cannot be established, pause rather than create another writer. Local generation checks fence the mediated local interface, not remote calls already in flight. Read-only monitoring may coexist with the single writer. [G sections 4/9/14; R-D05–D06]

Canonical patch application is not atomic with SQLite. Persist the exact base and target bytes/digests before applying it in the owned checkout. After a crash, compare each affected path with its recorded preimage/postimage; finish only unambiguous authorized writes, or reconstruct a fresh owned checkout from the retained snapshot. Preserve any third-state/foreign bytes and pause for reconciliation. Never assume a partially applied patch completed because its journal intent exists.

## 5. Required runtime events and receipts

The following uppercase names are **new controller contract records**, not claims that Codex already emits them natively.

### 5.1 `TURN_END_RECEIPT`

Produced by the trusted runtime adapter from its private lifecycle transport and host process observation, never from assistant text or nested tool stdout. Required fields:

```text
receipt_id, run_id, attempt_id, session_id,
native_turn_id_or_controller_attempt_mapping,
controller_generation, state_version_at_launch,
admission_digest, grant_revision_at_launch,
runtime_adapter_fingerprint, process_identity_and_start_nonce,
window_id, host_boot_id, observed_start, observed_end,
native_event_type, native_status, raw_event_digest,
process_exit_code_or_signal, child_quiescence_observation,
usage_receipt_or_unknown_reservation
```

Mapping rules:

| Observation | Controller classification |
|---|---|
| Expected runtime completion event with normal completed status | `MODEL_COMPLETED_TURN`. It proves only that the model attempt ended. |
| Model says “cannot finish in this execution window,” followed by normal completion | `MODEL_COMPLETED_TURN` plus an untrusted explanation; not a window-limit receipt. |
| Failed/interrupted native turn | Classified from trusted cause: technical failure, explicit cancellation, or a separately proven host limit. Not automatically a window limit. |
| Process exits 0 but required lifecycle event is missing/corrupt | Protocol failure requiring bounded reconciliation, not run completion. |
| Process disappears or receives a signal without an attested limit cause | Crash/interruption recovery. Do not invent a time-limit cause. |
| Per-turn context/output cap | Turn-level interruption/continuation condition unless the host proves the entire current allocation ended. Account-wide restrictions must not be bypassed. |

A completed receipt is consumed once. If child quiescence is not established, no next mutating attempt is dispatched merely because a completion message arrived.

### 5.2 `WINDOW_LIMIT_RECEIPT`

`EXECUTION_WINDOW_LIMIT` requires a trusted host/runtime record that the current execution allocation can no longer execute another turn. Required fields are run/attempt/window/boot IDs, trusted producer, enforced limit kind, host policy/allocation ID, supporting runtime code or supervisor observation digest, observed termination, last durably committed sequence and known unresolved actions/effects.

For an enforced deadline, include the host-established deadline and comparable monotonic observations within the identified boot/window. For host allocation expiry/preemption, include the actual host allocation event. The limit must have existed independently of the model's attempted return; the controller cannot manufacture a tiny window after the fact to excuse early completion. A generic exception, elapsed time estimate, voluntary completion, ordinary review rejection or arbitrary watchdog failure is insufficient.

The host adapter may provide the receipt during recovery after a hard kill. In its absence, record an unknown crash/technical pause. Never invent a checkpoint acknowledgement: after failed persistence, report the last successful checkpoint and any unrecorded observations as uncommitted.

### 5.3 `CONTINUATION_DECISION`

After a turn ends or a wake event arrives, the controller records: causative event/receipt, current state version, admission/grant/capability generations, incomplete obligations, eligible frontier, relevant budget balances, chosen action and one of `DISPATCH_ACTION`, `START_NEXT_TURN`, `WAIT`, `SAFE_OPERATIONAL_PAUSE`, `REQUEST_OWNER_DECISION`, `REQUEST_EXTERNAL_PERMISSION`, `FINAL_COMPLETE`, or `CANCELLED`.

These are decisions, not new blocker classes. Every next-turn decision gets a durable `TURN_SCHEDULED` outbox entry with a unique attempt ID. Repeated receipt delivery cannot schedule two attempts. Dispatch rechecks the current state and authority; an older valid decision is not a reusable permit.

## 6. Controller loop and early-return semantics

After a normal model turn, do not ask the model whether the Owner should say continue. Evaluate the journal:

```text
recover/claim resource; reconcile prior attempts and effects
while run is not closed:
    consume authenticated control/runtime events once
    apply revocation, invalidation, receipts and budget observations
    verify all predicates against their current basis
    enqueue any genuinely necessary human request once
    if completion gate passes: commit final receipt and result outbox; stop
    if safety, authority or storage prevents dispatch: record honest pause/wait
    else select a ready, authorized, fresh, budgeted action
         if deterministic: dispatch through gate
         if model work: persist TURN_SCHEDULED; launch/continue explicit session
         if no ready action: schedule bounded probe/search or persist wait/pause
    await next trusted event; never require an Owner message for internal progress
```

One model's last message is buffered as an internal artifact. The controller renders the Owner-facing verdict and verified dimensions from its own receipt. Arbitrary worker “done” text is never forwarded as final success. Read-only progress projections may show facts without requesting intervention.

**Required trace:** turn 1 implements a slice and voluntarily completes; tests/review remain ready; controller consumes `MODEL_COMPLETED_TURN`, records the unchanged obligation set, schedules verification or turn 2, and continues. No `OWNER_BLOCKER`, no `EXECUTION_WINDOW_LIMIT`, no Owner “continue,” and no final receipt occur at that boundary.

The scheduler uses a prerequisite DAG and finite candidates, not alphabetical predicate order. A blocked node blocks only its dependency closure. It continues unrelated admitted work, subject to actual resource/safety constraints. Short transactions and bounded adapter timeouts prevent an unbounded remote wait from monopolizing the loop; a timer does not require a distributed queue.

The local controller stays alive across model children. An installed host restart/wake mechanism reopens durable pending runs after process/host restart when the host is available and policy permits. Its deployment record includes launcher identity/configuration, supported wake causes and a verified restart receipt. A planned callback is not an installed scheduler. Without that capability, expose `PAUSED_NEEDS_EXECUTOR` as a deployment limitation and fail Stage 2 acceptance, rather than promise autonomous resumption.

After a real window limit, the same run can resume automatically in a later permitted allocation using this host mechanism. Neither durable state nor a new session overrides quotas, a revoked grant or a spending cap.

## 7. Blockers, search and operational pauses

Keep existing blocker classes; separate them from run state. Run state is `ACTIVE`, `WAITING`, `PAUSED`, or `CLOSED`, with a separate closure result. `FINAL_COMPLETE` is success, not a reason to block.

| Class / stop kind | Required meaning and next behavior |
|---|---|
| `TECHNICAL_LOCAL_FAILURE` | Observed command/tool/protocol/availability failure or unresolved diagnosis. Repair, probe, retry or replan safely within limits; no automatic Owner escalation. |
| `REVIEW_FINDING` | A current review issue or disputed technical claim. Reproduce/adjudicate, remediate, retest and rereview under a finite budget. |
| `OWNER_BLOCKER` | A concrete material choice outside current intent/authority: product, scope, security, architecture, data, cost or accepted risk. Require decision-impact evidence and exact missing authority. |
| `EXTERNAL_PERMISSION_BLOCKER` | Login, 2FA, OS/device approval, approved payment confirmation or other human-only consent. Use the genuine service UI; never request secret material in the journal/chat. |
| `EXECUTION_WINDOW_LIMIT` | Physical allocation end supported by section 5.2. Preserve/recover the run and use an actual later wake mechanism. |
| `SAFE_OPERATIONAL_PAUSE` | A stop kind on `PAUSED`, not an additional Owner blocker: no safe budgeted progress, storage failure, unresolved effect, missing executor, or bounded technical exhaustion. Preserve the underlying blocker/cause and incomplete obligations. |

A decision request records affected IDs, current authority, verified facts, exact decision/options, changed consequences, recommended choice and safe behavior without a reply. “Reasoning is difficult” does not satisfy it. Existing sufficient permission is reused. Request IDs deduplicate notifications and stale answers cannot amend a different admission generation.

An approved spending decision and a subsequent physical payment confirmation are separate dependencies. Successful login requires a capability probe of the intended principal/tenant/scopes; it does not broaden authority. Continue unaffected work while a human dependency waits.

For each blocked predicate, record `blocked_scope` (`task`, `approach`, `predicate`, `increment`, `goal`) plus exact IDs and computed dependants. Record a finite candidate-set revision. Each candidate needs requirement equivalence, authority eligibility, capability/feasibility evidence, preconditions and rejection/attempt history.

`alternatives_exhausted` means all candidates in that named set are unavailable/ineligible/failed and the allowed discovery/search budget for that generation is exhausted. It does not quantify over all imaginable strategies. If eligible actions exist but the total execution budget is exhausted, record **budget exhaustion**, not false candidate exhaustion. A worker-submitted empty set alone proves neither condition.

Persist usage, attempt counts, elapsed execution allowance, per-effect retry limits, search allowances, repair/finding signatures and no-progress counters. Two materially identical failed repairs stop that blind approach and cause bounded diagnosis, not an automatic return to Owner. New labels or sessions do not reset counters. Different useful review findings may justify further work within the same limits; endless oscillation does not.

Progress counters advance only on verified predicate/finding changes or a recorded discriminating observation, not on a new turn, renamed plan or arbitrary commit. Total limits still bound a stream of novel but unproductive hypotheses. A hard monetary cap is claimed only when the adapter/provider can bound each dispatched job's charge; otherwise expose the enforced turn/time/usage bounds and uncertain cost honestly, reserve conservatively and pause before more billable dispatch.

A pause receipt states reason and evidence, last checkpoint, unresolved predicates/effects, next safe action or missing precondition, and the actual wake mechanism or its absence. Notify once per meaningful pause generation, without presenting routine internal checkpoints as handoffs. A budget increase is an Owner decision only when proposed; reaching a cap is already sufficient to stop safely.

## 8. Evidence, review and completion

An acceptance receipt is captured by a trusted producer and binds run/action, admission/requirement IDs, artifact tree or retained snapshot, verification recipe/test-suite digest, relevant environment/runtime/adapter identity, start/end, exit/result, discovered/passed/failed/skipped checks, retained output reference/digest, coverage and limitations. Worker-written reports are claims to verify.

A model may author a test in its patch, but cannot mark it passed. The controller runs the approved recipe against the frozen artifact and validates expected coverage, skip policy and negative cases. Changing tests to remove an obligation cannot repair the contract. Actual invocation proves execution, not universal semantic correctness; independent requirement/behavior review remains necessary.

The controller creates a distinct review assignment with a fresh read-only context, contributor exclusion, required review policy, immutable artifact basis and structured verdict/findings. A different display name or the implementer's continued session is not independent. Different providers are optional unless already required. If the reviewer edits the artifact, it becomes a contributor and cannot supply final independent acceptance for that revision.

Findings retain IDs, severity, requirement/risk, reproduction/evidence and disposition history. The implementer cannot close a finding from prose. Closure requires valid current tests and independent rereview, or evidence-backed independent adjudication of a false positive. Genuine product ambiguity/residual-risk changes follow the Owner boundary; ordinary reviewer disagreement is bounded technical work. Do not shop for an agreeable reviewer while dropping prior findings.

Any relevant source, test, runtime, configuration or API change invalidates dependent green predicates; new failures/findings reopen them. Unknown dependency impact requires wider revalidation. Missing evidence does not remain green. A final check resolves all required evidence and checks current artifact/authority/safety state again. [G sections 9/12; R-D09]

For sealed manifest M and final artifact T, completion requires:

```text
required(M) is nonempty and covers all admitted obligations
AND every required predicate has valid verifier evidence for its required basis
AND required review is independently accepted on T
AND no required finding or relevant unknown effect remains unresolved
AND required delivery and live dimensions are independently established
AND no active mutation can change the artifact being accepted
AND action history contains no unresolved authority/safety violation
AND current admission/state versions still match the evaluated snapshot
```

Commit the unique final receipt and result outbox entry atomically after these checks. `valid=true`, process exit 0, all available tests passing, a local commit, or a provider HTTP 200 alone cannot establish this conjunction. The result distinguishes implementation acceptance, local checks, review/remediation, publication and required live proof. Success of one dimension never overwrites failure/absence of another.

Deduplicate final display by run/admission/final-receipt ID. This is one local final decision, not an unconditional exactly-once guarantee for arbitrary notification providers. Later contradictory evidence appends an invalidation/incident record; it does not erase historical receipts or quietly reuse an old success.

## 9. Capability and runtime drift

Reuse current capability states. Add observations tied to executable/build, adapter/configuration, API contract where observable, principal/tenant, probe result, timestamp/expiry and generation. Check relevant identity before use and after restart, long pause, auth change or drift signal. A TTL cannot detect an update within that TTL; pin the execution artifact/process where feasible.

On drift, block only affected dispatch, reconcile in-flight effects and invalidate dependent evidence. Restore the approved pin or use a pre-authorized compatible alternative with new probes and bindings. Do not silently change provider, data placement, credentials, security or required review quality. An unresolved model alias is recorded honestly, not represented as an immutable version.

Unsupported journal schema fails closed for writes. Migrations need supported version transitions, protected backup and interruption tests; no automatic destructive downgrade. A proposed update to guard code is reviewed under the already installed guard, not used to approve itself. [A08; G section 14; R-D04]

## 10. Git and external effects — required before Stage 3 dispatch

Persist logical `operation_id`, run/admission origin, exact target, canonical payload digest, intended postcondition, authority/capability binding, adapter recovery contract and budget reservation **before** dispatch. Replanning or restarting does not assign a fresh business identity to the same unresolved effect. Reusing an identity for a different payload is rejected.

Use `PREPARED -> DISPATCH_RESERVED -> CONFIRMED`, with `UNKNOWN`/`QUARANTINED` outcomes as needed. Persist the reservation before handing off. A crash anywhere after reservation is potentially dispatched, even when it might actually have died before send. Missing acknowledgement or timeout is not proof of failure.

| Recovery class | Permitted behavior |
|---|---|
| Local transactional/immutable operation | Read the committed result or reconstruct identical object bytes; apply guarded local ref change once. |
| Provider-deduplicated | Same logical key and payload only within the verified scope/retention/parameter contract; reconcile authoritative outcome. |
| Reliably queryable operation identity | Read by stable identity using documented consistency and reconciliation horizon. A transient absent read is not proof of no effect. |
| Opaque non-idempotent / expired dedupe / incompatible semantics | No automatic replay. Preserve uncertainty; require provider reconciliation or a material authorized resolution. |

No global exactly-once claim is made. A compensation, delete, refund or resend is a separate authorized effect. If the journal cannot acknowledge persistence, dispatch stops. If receipt storage fails after remote success, the earlier intent survives and recovery queries the provider before considering another send.

For Git, use an owned worktree/index and explicit patch/hunk inventory, preserving foreign dirty work. Revalidate repository/common-directory/worktree identity, branch, source basis, paths including renames/deletions/symlinks, and allowed downstream effects. Sanitize executable/environment/configuration and unapproved hooks/helpers; a read-looking command can invoke auxiliary code. [R-D08/D11]

A local commit intent binds exact tree, parent, message and commit metadata; retain enough bytes to rediscover/reconstruct the same object after a crash. Check commit-tree equality with the accepted artifact. Guard local ref updates by expected old value. Do not manufacture duplicate commits with new timestamps merely to recover.

A push intent names repository identity, remote identity, source commit C, full destination ref, observed old ref, required publication predicate and branch-creation permission. Use normal non-force publication only. No implicit upstream, `--all`, tag push, force/force-with-lease, PR, merge, release or deployment fallback. Assess triggered CI/webhook effects separately.

Preflight remote reads are not atomic compare-and-swap. Ordinary Git fast-forward rules protect against non-fast-forward overwrite, not every application-level expected-old condition. Where the contract requires stricter expected-old semantics, the adapter must prove server-side non-force conditional-update support or refuse that operation; do not disguise force-with-lease as ordinary push. Concurrent advancement leads to authorized reconciliation and revalidation, not history overwrite. [W4]

After any attempted/uncertain push, independently read the **exact** remote ref through the trusted adapter. Equality with C satisfies a contracted “tip is C at verification time” predicate. A descendant is sufficient only for an explicitly admitted “C is in history” predicate, with ancestry proof; it does not prove current tip review. Record observation time. CLI output alone cannot set `push=SATISFIED`. Git publication does not prove deployment/live behavior or exactly-once webhook processing. [W4–W5]

## 11. Restart, STATUS and migration

Recovery order is fixed: acquire valid local ownership; validate schema/integrity; quiesce or quarantine prior jobs; reload intent/grants/limits; reconcile unfinished attempts/effects and actual worktree/remote identity; revalidate evidence/capabilities; rebuild projections; then schedule the current eligible frontier. Never resume by simply executing a prose `next_action` from STATUS.

A crash after `TURN_SCHEDULED` but before confirmed launch does not justify two children. Reconcile the attempt's recorded host identity/start nonce, establish old-job quiescence, then decide a new attempt under the same run. Accepted patches and state survive acknowledged checkpoints; unaccepted scratch work may be lost. Unknown usage remains reserved conservatively. Missing/corrupt state never creates a blank authorized run with reset budgets.

**STATUS decision:** transactional execution journal is authoritative for execution; `STATUS.md` is a dated projection carrying run ID, admission revision, journal sequence and basis. Product requirements/decisions remain canonical inputs, not rewritten by journal projection. Git/provider observations establish their own facts, not product acceptance.

Migration is per run. Import legacy STATUS/admission/checklist data once as unverified claims with provenance; inspect actual Git, receipts, obligations and authority; then record an explicit V2 cutover. Thereafter edits to STATUS cannot grant authority, close findings or change predicates. Projection tampering is overwritten/reported, never imported as a control command. Database failure allows read-only diagnosis, not fallback execution from Markdown.

Avoid a self-referential commit loop: freeze the deliverable tree, including a dated STATUS checkpoint if required, before its final tests/review/commit. The post-publication final receipt lives in the protected journal. A current live Markdown projection can be rendered outside the frozen checkout; the committed STATUS is explicitly “as of sequence N.” Do not rewrite and repush the verified tree merely to insert its own final SHA. If a contract requires additional committed status content, define a finite checkpoint boundary beforehand and verify that artifact normally.

Local storage recovery and rollback preserve unresolved effect identities and prior evidence. Never delete foreign work, roll back a shared database, or reset a remote ref merely to make a checkpoint look consistent.
