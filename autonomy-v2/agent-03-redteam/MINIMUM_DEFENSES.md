# Minimum defenses after the attacks

## Status and design objective

This is a **proposal**, not an implemented change, accepted architecture, or authorization to modify AI OS. Source baseline: `713928f1425374bac6dfdf13ba842d582ce5c37b`. The attacks and their evidence are in `ATTACK_REPORT.md` and `FAILURE_MATRIX.md`.

The target is a small enforced execution boundary, not a new orchestration platform. Keep the existing goal/increment hierarchy, product-local admission, capability truth vocabulary, blocker classes, source/evidence distinctions, and useful validators. Do not turn NovaStory into a duplicate AI OS or move product requirements into the shared system.

The minimum safety claim should be:

> The agent may autonomously choose and execute an eligible in-scope next action. It may not grant itself new authority, erase required outcomes, certify its own unverifiable claims, or replay an unresolved effect. When safe progress is unavailable, it preserves state and stops honestly.

That is a proposed guarantee. It is not true of the current general `AUTONOMOUS_BLOCK` path merely because a simulator returns valid JSON.

## 1. Three boundaries to enforce

**Action boundary:** every consequential action must pass current identity, scope, authority, ownership, capability, and effect checks before dispatch.

**State/evidence boundary:** workers submit proposals and observations; only a trusted controller validates state transitions and writes authoritative receipts. A file hash proves identity, not truth or authorization.

**Response boundary:** a substantive completion claim must be derived from current obligations and verified observations. An optional script, prompt instruction, or process exit code cannot enforce a response channel it does not control.

One local controller can implement these boundaries. For a single-host setup, a transactional local journal and an actual process lock are a reasonable starting proposal. Distributed scheduling, a fleet database, message queues, or etcd are not prerequisites. If the actual host cannot mediate actions or completion responses, label the arrangement policy-assisted rather than mechanically enforced autonomy.

## 2. D01: Strict action and response gate

**Attacks:** A18, A30, A36–A37, A40, A46–A50, A52.

Require an explicit typed action proposal and a current run/state version. Reject unknown actions, invalid JSON shapes, duplicate IDs, contradictory resume values, missing evidence, and unknown gate results. Validation success is not permission, and permission is not completion.

Do not allow worker events such as repair-pass, rereview-accept, or normal-push to set completion predicates directly. They can request evaluation of a receipt. Only the evaluator changes predicate state after checking the observation and its basis.

A completion decision must be bound to the current run, intent/admission generation, obligation manifest, source/evidence basis, and state version. Intercept the real completion path; an agent should not be able to bypass the gate by omitting the validator call or printing a different answer. Unknown or stale verdicts fail closed for actions and success claims, not into endless silent retries.

A terminal response needs no unrelated implementation if the task is documentation-only. Conversely, a material implementation increment cannot self-declare an empty obligation set. Preserve the distinction between completion of a source-only audit, implementation acceptance, and live operation.

**Acceptance test:** attempt early completion through the actual integration boundary without a gate receipt; send a stale receipt, malformed record, valid-but-incomplete evaluator result, and empty predicate set. None may authorize an effect or success response.

**Owner boundary:** implementing the integration needs a separately authorized stage. A guard must not expand tool permissions just to make enforcement convenient.

## 3. D02: Preserve obligations through admission and replanning

**Attacks:** A04, A13–A14, A16–A17, A27, A31–A33, A37–A39, A52.

Use the existing product-local admission as the home of an immutable, versioned obligation manifest. Preserve stable requirement IDs and prohibited substitutions. Validate uniqueness before building lookup maps. Required types must be checked, not approximated by truthiness.

For every material outcome, record the acceptance predicate, required evidence class, dependencies, and applicability reason. An implementation task may move between approaches; its hard requirement cannot disappear because the old approach failed. Track `blocked_scope` together with the exact blocked IDs and dependency closure. A blocked task is not automatically a blocked goal, but its dependent acceptance predicate cannot be ignored.

Replanning should produce a compact before/after diff: tasks changed, obligations retained, evidence invalidated, authority implications, and affected commitments. Ordinary equivalent task reshaping remains autonomous. Material changes to outcome, architecture, security, accepted risk, cost, or delivery commitments require the Owner.

Applicability is derived from the admitted outcome, not from whether evidence was convenient to obtain. Source-only work can legitimately have no live proof. A requested external outcome cannot become N/A because credentials expired, the service disappeared, or the execution window ended. Keep a pending/blocked external obligation visible even when implementation is accepted.

**Acceptance test:** remove a hard requirement, duplicate its ID with a weaker classification, drop a blocked predicate during replan, or mark required live proof N/A. The unchanged admission must reject each attempt. Verify that a genuinely source-only task is not forced to fabricate a live requirement.

## 4. D03: Effective authorization, not a copied envelope

**Attacks:** A03, A06–A07, A15–A16, A22, A31–A33, A42, A45, A50.

Bind authority to the exact repository/service identity, target ref/resource, action class, principal/tenant, allowed surfaces, bounds, accepted intent/admission version, and revocation generation. Include expiry or explicit freshness conditions where needed. Resolve effective authority at action time from a trusted source; the worker's statement that an envelope is active is not that source.

Workers cannot extend the envelope, accept their own exception, renew its expiry, or switch principals to escape a blocker. Availability, capability, and authority remain independent checks. A possible alternative is not yet an admissible alternative, and an admissible one may still contain material tradeoffs requiring the Owner.

An Owner intent change revokes old dispatch authority. Define the linearization point: actions already sent may still complete, so cancellation stops future dispatch and triggers reconciliation of in-flight effects. Do not promise that changing an intent flag reverses a remote operation already in progress.

Keep the current separation of decision and external assistance. Login/2FA/physical consent is an external-permission blocker. Approving spending or a new privilege is an Owner decision; a subsequent payment-confirmation step can separately require external assistance. The event label must follow the actual need, not collapse those stages into one ambiguous category.

**Acceptance test:** stale/revoked envelope, changed branch, changed principal, changed Owner intent, or an unapproved cloud fallback must be rejected before dispatch. A same-scope authorized action should not repeatedly ask for confirmation.

## 5. D04: Capability freshness at the point of use

**Attacks:** A01–A04, A06, A12, A15, A26, A32, A50–A51.

Reuse the existing capability truth vocabulary and context-freshness checks. Add a bounded capability observation for the action that will actually run: executable/build identity, plugin/configuration identity, API contract where available, principal/tenant, probe result, observation time, validity window, and generation.

A successful executable lookup is not authentication, model availability, or successful remote execution. A preflight is not permanent. Recheck on relevant runtime/configuration/API changes, renewed login, new principal, long suspension, or uncertain failure. Use a pinned process/artifact where possible to reduce check/use races. TTL limits age but does not detect a change that happens inside the TTL.

Invalidate affected evidence when the executable or behavior basis changes. Do not declare the entire repository stale for an unrelated file change, and do not treat a matching source hash as proof of an unchanged external API.

The existing S3d runner digest and S09 selected-source/Git checks are useful ingredients. They are not substitutes for live capability generations on unrelated tools.

**Acceptance test:** mutate the runtime/plugin identity between preflight and a synthetic action. The action must re-probe or refuse; no old green receipt may silently authorize new behavior.

## 6. D05: Durable checkpoints with failure-aware recovery

**Attacks:** A08, A17–A21, A23, A25, A34, A46, A49.

Choose one authoritative local execution journal. Existing `STATUS.md` and related goal documents should be human-readable projections or clearly dated checkpoints, not a competing mutable authority. Preserve their IDs and history. The journal records the current intent/admission/source basis, run/version, obligations, findings, evidence references, ownership, action intents, confirmed/unknown outcomes, budgets, and exact next safe action.

A small local transactional store is one option. A file-based design needs locking, validated versioning, atomic replacement, durability handling, and recovery of the prior valid record. Merely calling a text-write function is not a crash protocol. Local storage guarantees remain conditional on filesystem/storage behavior [W3]. No local journal provides atomic commit with an unrelated external API.

Before a consequential effect, persist its intent and obtain a successful durability acknowledgement. If storage is full, locked beyond budget, unreadable, or corrupt, stop dispatching new effects. Reserve enough practical capacity for small checkpoints where feasible, but never assume a free-space check guarantees future writes.

On resume, reconcile the journal with the actual owned Git worktree and relevant remote observations. A stale STATUS entry is neither authority to reset Git nor permission to disregard an external effect. Missing state must not reset action budgets or grant a fresh operation identity.

**Acceptance test:** inject failure before intent persistence, after intent persistence, after dispatch, after remote success, and during receipt/checkpoint storage. Each restart must either resume safely or stop with an explicit unresolved effect; it must not silently replay unknown work.

## 7. D06: One active writer, including after a lease expires

**Attacks:** A08, A16, A20–A21, A23–A24, A44.

For a single host, use an actual process-held lock and an atomic journal claim with run identity and state version. Do not add distributed leases when a single local writer suffices. Duplicate resume deliveries should join/read the existing run or refuse mutation, not create another writer.

For a multi-process or multi-host design that permits takeover, use an ownership epoch/fencing token and enforce it at every mutation boundary. A lease expiry alone is not enough: the old agent may be paused, lose contact, then continue. Heartbeat timestamps, a lock file name, or matching fingerprints are not fencing.

The small practical approach is one trusted effect gateway. Workers have no independent external-write credentials and cannot write shared state/worktrees outside the mediated lane. A losing or stale worker cannot dispatch merely by retaining an old record.

If a target service does not enforce fencing, the gateway must serialize and reconcile dispatch. Do not take over an unresolved operation while the previous process can still issue it. Some uncertainty requires quiescing the old process and using provider idempotency; it cannot be solved by incrementing a local integer. Lease-backed transactional ownership is a useful reference model, not an instruction to deploy etcd [W4].

**Acceptance test:** two contenders read the same state and race a barrier. Exactly one receives dispatch authority. Pause that winner, trigger the permitted takeover procedure, and attempt a stale write. The stale action must fail at the actual mutation boundary.

## 8. D07: Effect intent, deduplication, and reconciliation

**Attacks:** A04–A06, A08–A09, A19, A23–A24, A35, A45.

Every consequential operation needs a stable identity derived from the run/admission generation and intended logical effect, plus target identity and request hash. Retries reuse that identity only for the same request. A new attempt counter is not a new logical operation.

Persist intent before sending. Record dispatch and keep the remote outcome explicitly unknown until supported by a reliable acknowledgement or authoritative read-back. A timeout after dispatch is not evidence of failure. A client error after Git push is not evidence that the remote ref stayed unchanged.

When supported, use the provider's idempotency contract and check its retention window and parameter rules. For example, Stripe documents supported key-based request deduplication, not universal indefinite exactly-once execution [W1]. Do not issue a new key merely because the old request timed out.

Without remote idempotency, query by a stable correlation/resource identifier. Account for eventual consistency: a temporarily absent read is not necessarily proof that the operation never happened. Define the service-specific reconciliation horizon. If no reliable reconciliation path exists, preserve uncertainty and stop replay. At-most-once dispatch with a possible unknown outcome can be safer than an invented exactly-once guarantee.

Compensation is a separate effect with its own authorization, identity, and failure handling. Never automatically delete, refund, revert production, or resend communication just to make the journal look clean.

For Git, observe the exact remote ref and commit, not just a process exit code. Git's ref/publication semantics do not certify a deployment, database change, or other external outcome [W2].

**Acceptance test:** simulate success with a lost response, failed-before-send, delayed visibility, idempotency-key expiry, repeated resume, and a conflicting request using the same key. No case may create an unapproved duplicate or a false success claim.

## 9. D08: Git and worktree guard

**Attacks:** A09, A21–A24, A31, A34, A42–A44.

Record and revalidate resolved repository identity, worktree registration, branch, expected HEAD/source-tree basis, remote identity, target ref, and ownership. Treat a path reappearing after deletion as a new identity until verified.

Prefer an isolated owned worktree and staging area. Capture pre-existing dirty changes and avoid broad staging. Inspect the exact staged patch, including renames/deletions, generated files, submodules, symlink effects, and secrets. File-path allowlisting alone does not authorize another person's hunks in the same file.

Before publication, verify the frozen tree and intended commit, scoped normal-push authorization, destination ref, and current remote basis. Use an explicit ref destination; ambient Git configuration can select a different push target [W2]. Never force-push or rewrite others' work to resolve ordinary concurrency.

When upstream advances, do not blindly accept or overwrite it. Preserve both histories, perform only authorized integration, and revalidate the resulting snapshot. Checks may be reused only when their complete relevant basis is demonstrably unchanged.

After an uncertain push, read the exact destination ref. An exact match is straightforward evidence. A descendant requires ancestry analysis; an unexpected commit or target is not permission to repair history unilaterally.

**Acceptance test:** wrong branch, wrong remote, removed/replaced worktree, stale HEAD, upstream advance, unrelated staged hunks, and a dirty sibling tree. The guard must preserve unrelated work and reject unintended publication.

## 10. D09: Evidence, review, and external acceptance

**Attacks:** A03, A10–A13, A17, A22, A25, A28, A33–A36, A38, A40–A41, A47.

A minimum receipt identifies the producer/controller, run/action, source-tree and relevant environment basis, command or observation contract, start/end, exit/result, retained evidence digest/reference, requirement coverage, and remaining limitations. Collect process output through the controller rather than asking the worker to write a plausible log afterwards.

Keep authoritative receipts outside worker write authority. Hashes prevent unnoticed byte drift only when the trusted reference is protected; they do not authenticate a claim made by the same untrusted writer. A second model or a different agent ID is not automatically independent review.

Bind review to a frozen source set. Each finding needs a requirement or risk, evidence, severity, and disposition. Closing a finding requires a verified fix or an evidence-backed rejection of the finding, current tests, and the required rereview. Review disagreement invokes bounded technical adjudication, not endless argument or automatic Owner escalation.

Failures and new findings invalidate affected predicates. Invalidation propagates through dependencies. New code, runtime changes, revoked applicability, and failed remote postconditions cannot coexist with reusable old green flags unless the relevant basis is explicitly proven unaffected.

Acceptance must check behavioral coverage as well as execution. All tests passing does not establish that the tests cover the Owner's requirements. Preserve negative cases, prohibited substitutions, skipped tests, mocks, and gaps. A controller can prove that a command ran; semantic correctness still needs risk-matched tests and review.

Retain separate dimensions for implementation, local verification, independent review, publication, external/live proof, and Owner acceptance when required. Do not collapse them into one optimistic boolean. Missing live proof remains missing; a Git commit cannot replace it. Evidence referenced by final acceptance must still resolve or have a retained verified equivalent.

**Acceptance test:** fabricated worker report, missing receipt, deleted evidence, stale tested tree, wrong reviewer basis, unresolved critical finding, missed requirement with passing tests, and failure-after-green. Each must prevent the affected success claim.

## 11. D10: Bounded progress and honest blocker classification

**Attacks:** A01–A02, A05, A07, A10–A11, A14, A18, A20, A26–A29, A39, A49.

Use one persistent budget for the attempt, not a fresh budget per chat. Bound retries, repeated repairs, reviewer disputes, candidate search, and total resource use. Store failure and plan signatures so equivalent rewrites do not look like new progress. Choose actions by dependency readiness and safety, not alphabetic predicate order.

`alternatives_exhausted` must be relative to a named, finite eligible candidate set, capability/intent generation, and exploration budget. Record each candidate's feasibility, requirement equivalence, authority eligibility, and rejection evidence. Do not require proof that no imaginable solution exists. Do not mark exhaustion because reasoning is difficult.

When a candidate exists, evaluate whether it is admissible before whether it is attractive. Autonomous selection is reasonable among approved equivalent mechanics. Choices changing privacy, cost, security, architecture, reliability commitments, or product meaning remain material Owner decisions.

Use the current blocker vocabulary. A technical failure stays `TECHNICAL_LOCAL_FAILURE`; review work stays `REVIEW_FINDING`; host limits stay `EXECUTION_WINDOW_LIMIT`; external assistance stays `EXTERNAL_PERMISSION_BLOCKER`. Require an explicit decision-impact record before `OWNER_BLOCKER`: what must be decided, why existing authority does not resolve it, available choices, and changed consequences.

### Necessary clarification to the proposed no-return rule

The current response policy permits Owner-facing responses only at completion or Owner/external-permission blockers. Taken literally with unbounded repair and no real resumer, this can prevent any honest response when safe progress is impossible.

**Proposed revision:** suppress routine internal checkpoints, but permit a truthful non-success operational stop when a host/resource limit is reached or the bounded eligible actions are exhausted. Keep the existing blocker class and attach checkpoint, stop reason, unresolved predicates/effects, and exact resume condition. Do not label this completion or an Owner decision. This is a proposed response-policy change, not a newly installed parallel goal status system.

An actual scheduler may later resume the run if available and authorized. Otherwise say that a new invocation is needed; do not promise background work. A no-return gate must protect against premature success, not suppress cancellation, safety stops, or platform reality.

**Owner decision needed before implementation:** accept this operational-stop exception and the material budget/fallback defaults. Routine future technical pauses should not each require Owner adjudication.

**Acceptance test:** unavailable model, repeated identical failure, permanent dependency loss, locked store, reviewer deadlock, and execution-window end. Verify bounded termination, durable state, no fake Owner blocker, no fabricated success, and no unsupported promise of resumption.

## 12. D11: Narrow effect mediation and a real read-only boundary

**Attacks:** A40–A41, A50–A51.

The trusted controller, its policy basis, and authoritative evidence store must not be editable by the same untrusted worker whose actions they constrain. Route external writes and final acceptance through the narrow controller. Unrestricted shell access, writable guard code, and reusable external-write credentials let the worker bypass a policy file or forge local receipts.

For read-only tooling, pin the executable and sanitize effective environment/configuration. Disable unapproved hooks/helpers, avoid inherited credentials and writable control paths, and use OS/process restrictions where a strong no-network/no-write guarantee is claimed. Git supports executable fsmonitor configuration, so the word status does not make a process harmless [W5]. A narrow sanitized command is risk reduction, not a universal sandbox proof.

A receipt must report what was actually enforced and observed. Do not hard-code no-network/no-writes and infer it from unchanged Git status. Optional goal-local receipt writing is itself a bounded write exception and should be described honestly.

Recompute the relevant gate decisions from authoritative current state or verify a trusted producer's fresh decision. Matching hashes of caller-supplied failed gates establish consistency, not permission. Do not trust repository text, worker output, or a copied envelope as a new authorization source.

**Acceptance test:** failed gates with matching fingerprints, ambient executable replacement, effectful Git helper, attempted evidence-store modification, and direct worker write bypass. The actual integrated boundary must deny or accurately constrain them.

**Owner boundary:** new sandbox/security architecture and credential separation require explicit approval. They are not silently authorized by this audit. Without them, state the weaker trust assumption clearly.

## 13. Minimal rollout, not a platform rewrite

| Stage | Bounded implementation proposal | Exit evidence |
|---|---|---|
| 1. Record and predicate correctness | Strict schema/unique IDs; immutable obligations; current-source evidence; invalidation; structured failure paths. Preserve existing validators and add targeted negative tests. | Existing positive controls still pass; P01–P20 regressions are blocked at the appropriate layer. |
| 2. Actual integration boundary | One local controller mediates action proposals, completion responses, authority, and ownership with a protected local journal. | Bypass, early-return, stale envelope, concurrent resume, and crash tests exercise the real entry points. |
| 3. Narrow external effects | Add only the currently needed operation adapters, explicit Git publication, deduplication/reconciliation, and capability refresh. | Lost-response, wrong-target, replay, and external-proof failure tests pass in disposable fixtures. |
| 4. Broader unattended use | Expand only after guard limits and trusted process/credential boundaries are approved. | Independent review of the integrated lane, known limitations, operational-stop behavior, and rollback/recovery evidence. |

These are proposed implementation increments, not commands to modify AI OS now. No source runtime, dependencies, service accounts, migrations, secrets, or deployment are changed by this research package.

## 14. Acceptance suite at the actual boundary

| Test family | Required negative cases | What must be observed |
|---|---|---|
| Completion | Missing/empty predicates, valid-but-incomplete exit 0, omitted gate call | No success claim escapes the integrated response path. |
| Admission | Duplicate IDs, non-array plan, blocked safety/readiness, removed hard outcome | No material action admitted by malformed or weakened records. |
| Authority | Revocation, expiry, changed intent, branch/principal mismatch | Dispatch denied before new effect; in-flight effects reconciled. |
| Evidence/review | Fake logs, worker self-report, wrong source basis, missing artifact, review rejection | No fabricated or stale predicate transition; affected checks reopen. |
| Capability | Runtime update, API drift, unavailable model, renewed login | Relevant capabilities/evidence refresh; no unauthorized fallback. |
| Ownership | Duplicate delivery, simultaneous resume, paused old writer after takeover | At most one valid dispatcher; stale epoch cannot mutate. |
| Effect recovery | Success with lost response, delayed visibility, expired dedup window | No unsafe duplicate; unknown outcome remains explicit. |
| Storage | Crash at every journal/effect boundary, disk full, DB contention | Previous valid state survives or an explicit safe stop occurs. |
| Git | Wrong ref, upstream advance, replaced worktree, foreign dirty hunks | Unrelated work preserved; exact intended ref observed after publication. |
| Liveness | Repeated failure, unavailable alternatives, review deadlock, host limit | Bounded honest non-success stop without false Owner decision. |
| Read-only lane | Failed gate with matching hashes, helper execution, evidence-store tampering | Real enforced boundary or accurately limited claim, not a decorative receipt. |

Run effect/crash/concurrency tests against disposable repositories and mock or sandbox services. Do not test duplicate payments, production data loss, or owner-worktree destruction in a live environment. A simulator suite is necessary regression evidence but not proof that the real action/response hooks are installed.

## 15. What remains outside the guarantee

No small guard can guarantee the semantic correctness of every requirement, universal exploration of all possible approaches, eventual availability of an external service, uninterrupted execution beyond host limits, or global exactly-once effects without supporting remote semantics. A compromised trusted controller/OS is outside this proposed threat boundary; unrestricted untrusted worker access is not acceptable while claiming strong enforcement.

The desired practical guarantee is narrower and testable: the agent cannot unilaterally weaken its obligations, broaden its effective authority, conceal unknown effects, or claim a result that its current evidence does not establish.

**Recommended next step:** authorize a bounded implementation increment for Stage 1 plus the real completion/action integration proof. Do not label the entire autonomy goal complete after repairing only simulator tests.

Public reference IDs W1–W5 and their official URLs are recorded in `ATTACK_REPORT.md`. They support idempotency, Git, local durability, lease/ownership, and helper-execution facts; the guard layout here is an original proposal.
