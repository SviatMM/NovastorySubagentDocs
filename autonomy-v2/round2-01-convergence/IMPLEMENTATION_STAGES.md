# Implementation stages — smallest sensible rollout

**Status:** proposed increments, not implementation work authorized or performed by this research. All paths below are repository-relative AI OS surfaces. Existing paths are distinguished from proposed additions. Re-ground them against the implementation checkout before editing; the research baseline remains `713928f1425374bac6dfdf13ba842d582ce5c37b`.

## 1. Rollout decision

Keep four stages, with one important clarification: **basic isolation, capability freshness, budgets, actual multi-turn supervision and restart support belong in Stage 2**, not Stage 4. Without them a real response/action gate would be bypassable or would merely return an unfinished job in a more structured format.

Stage 1 fixes obligation/evaluation correctness. Stage 2 is the first minimum useful local controller. Stage 3 enables required Git/publication and specific external effects safely. Stage 4 adds parallelism only if measurements justify it. No stage requires Kubernetes, etcd, a distributed database, a message broker, or a new product agent runtime.

Within Stage 2, prove a thin vertical slice first: one pinned runtime child voluntarily ends, the controller records the event and starts the next authorized attempt without an Owner message. Do not build a generalized registry or scheduler framework before this integration seam is demonstrated. The final Stage 2 exit still requires all its safety tests.

All deterministic suites use disposable workspaces, a fake clock, fixed protocol fixtures, fault hooks and mock/queryable sinks. They MUST assert both permitted progress and denied effects through the real controller entry points. They do not require destructive production tests. A separate pinned-host conformance run is needed to establish that these entry points are actually installed for Codex; a fake-adapter suite alone is insufficient.

## 2. Stage 1 — obligation correctness and strict predicates

**Problem solved.** Reject empty/weak applicability, duplicate requirement IDs, malformed admission/checkpoints, stale evidence bindings and event-name completion laundering. Preserve accepted product requirements through replanning. Define structured validity, decision and completion as different outputs. This repairs the evaluator contract, not long-running autonomy. [A07, A09, A11; R-P01–P20]

**Likely existing surfaces.** `scripts/simulate_autonomous_block.py`, `scripts/validate_admission_traceability.py`, `extensions/tests/test_simulate_autonomous_block.py`; `core/WORK_ADMISSION_PROTOCOL.md`, `core/ADAPTIVE_DELIVERY.md`; product templates `goals/_template/plans/CURRENT_INCREMENT.md` and `goals/_template/STATUS.md` under `templates/product-repository/`.

**Proposed additions.** A small reusable obligation/schema/evaluation module under `scripts/autonomous_execution/` and focused negative fixtures/tests under `extensions/tests/`. Extend the product-local admission format with manifest version, accepted-source digests, unique predicate IDs, coverage and evidence classes; do not copy actual product requirements into shared AI OS.

**Still only policy.** Authenticity of production receipts, runtime interception, persistence, grants, worker containment, real tests/review execution, automatic continuation, external effects and recovery. Fixture evidence must be explicitly tagged test-only and must never satisfy a production run.

### Deterministic acceptance tests

| ID | Input / exercise | Required oracle |
|---|---|---|
| T01 | Null/list root, invalid enums, duplicate JSON keys/requirement IDs, truthy non-array evidence plan, invalid HEAD, contradictory resume sets | Structured fail-closed verdict; no successful completion or uncaught parser exception; hard constraints not shadowed. |
| T02 | All flags false with an empty list; all flags true with no receipts; removed required live proof | No final completion. Required predicates come from the sealed fixture manifest, not caller selection. |
| T03 | `repair_pass`, `rereview_accept`, `normal_push`, unknown dangerous event, or valid evaluation exit 0 | None creates proof or action authority. Consumers inspect structured decision/receipt fields. |
| T04 | Replan deletes/weakens a hard obligation; blocked live predicate becomes N/A; new increment omits the old predicate | Replan rejected or requires explicit authorized supersession; parent completion remains false. |
| T05 | Green receipt followed by current test failure, review rejection, changed test/tree/runtime basis, missing evidence | Affected predicates and dependants become non-satisfied; unrelated evidence is reused only with proven unchanged basis. |
| T06 | Tests pass but requirement coverage missing; active critical finding survives a claimed rereview | Completion denied until the coverage/finding obligation is genuinely resolved. |
| T07 | Well-formed source-only task with nonempty document/evidence obligations; required implementation-only task without live scope | Legitimate applicability accepted; no invented live or code requirements. |
| T08 | Superseded decision, unacknowledged prohibited substitution, unambiguous missing hard reference, mismatched baseline | Existing useful negative controls retained; admitted safe well-formed records still validate. |

**Migration/compatibility risk.** Legacy records and tests assume worker booleans and an alphabetically chosen next predicate. Preserve their useful policy intent, not unsafe expected results. In particular, the old happy-path direct-pass events and “commit first” assertion need replacement. Use explicit schema versions; legacy input may be parsed as unverified claims but never upgraded to verified evidence. Keep legacy simulator mode clearly labelled rather than silently breaking consumers or pretending its exit code always meant delivery success.

**Owner decision required?** A separate authorization to implement this AI OS increment is needed; this research is not that authorization. Once authorized, enforcing already accepted constraints does not require new decisions per rejected record. No new runtime service, external credentials or production permission is required for this stage.

**Exit label:** `OBLIGATION_CONTRACT_VERIFIED`, not “Autonomous Execution V2 complete.”

## 3. Stage 2 — real local control plane and automatic continuation

**Problem solved.** Eliminate routine early handoffs inside the installed lane. Make authority and completion enforcement real, preserve state across model/process boundaries, bound unproductive loops, prevent concurrent canonical writers, bind review/evidence to actual observations, and recover local execution without an Owner acting as a message bus.

**Likely existing surfaces.** `core/AI_WORKFLOW.md`, `core/ADAPTIVE_DELIVERY.md`, `core/WORK_ADMISSION_PROTOCOL.md`, `core/CAPABILITY_TRUTH_REGISTRY.md`; `adapters/codex/AGENTS.md`; `adapters/codex/.agents/skills/deliver-large-projects/SKILL.md` and its `references/quality-loops.md`; goal/admission/STATUS templates. AI_WORKFLOW, AGENTS and quality-loops are likely integration surfaces identified by the pinned research source registers; inspect them directly when preparing the implementation patch.

**Proposed additions.** `scripts/run_autonomous_block.py` as launcher and a small `scripts/autonomous_execution/` package for controller, journal, gates, runtime adapter, verifier and projection. Add host installation/configuration documentation, a versioned local schema, and `extensions/tests/test_autonomous_execution_controller.py` with subprocess/fault fixtures. Use existing Python structure where appropriate; no daemon ecosystem or new service platform.

Implement one controlled scratch implementation lane and one controller-owned canonical integrator. Require real isolation before calling gates enforced. Controller-generated test/review receipts, grant administration and the journal remain outside worker write access. Include a pinned CLI JSON adapter first, explicit session mapping, `TURN_END_RECEIPT`, `WINDOW_LIMIT_RECEIPT`, `CONTINUATION_DECISION`, local outbox, process-held lock and controller generation. Install the approved local host restart/wake mechanism and document exactly what failure domain it supports.

**Still only policy / unsupported.** Universal semantic correctness is not mechanically provable. Arbitrary external mutations, paid/production actions, unsupported providers and broad worker fleets remain disabled. Stage 2 cannot truthfully satisfy a required push/live predicate with no Stage 3 adapter; those obligations stay pending. Direct sessions outside the V2 launcher remain policy-assisted. Do not reuse PEF's static read-only receipt declarations as proof of the new sandbox.

### Deterministic acceptance tests

| ID | Input / exercise | Required oracle |
|---|---|---|
| T09 | Real controller launcher receives a normal completed lifecycle event while tests/review remain ready | One continuation decision and one next action/turn are scheduled without any Owner input. No final response or window-limit classification. |
| T10 | Same event includes model prose claiming the execution window ended; nested tool output spoofs a host event | Prose/nested output rejected as limit authority. T09 behavior continues under available budget. |
| T11 | Trusted host deadline/allocation fixture actually ends the window; arbitrary SIGKILL and plain interruption tested separately | Only the supported host cause produces `EXECUTION_WINDOW_LIMIT`; others recover as crash/cancel/technical state. Durable later wake resumes the same run when permitted. |
| T12 | Omitted completion script, arbitrary model “done,” stale final receipt, incomplete evaluation exit 0 | None escapes the actual controller-owned success channel. Controller may emit a truthful non-success pause. |
| T13 | Duplicate turn-end/wake receipt; concurrent resume contenders at a barrier; crash between schedule and launch | At most one active mutation job and one claim. Duplicate events do not reset budgets, enqueue duplicate turns or create new run identities. |
| T14 | Stop old worker mid-job, attempt generation takeover/stale patch, replace worktree at same path | No live-lock theft. No new mutation before verified quiescence. Old generation or changed identity cannot integrate. Unknown foreign edits are preserved. |
| T15 | Worker attempts direct canonical/journal/grant/receipt edit, network write, credential read, helper execution or sibling-worktree mutation | Actual host boundary denies escape; no forged trusted receipt. A failed isolation probe denies lane admission. |
| T16 | Runtime/config changes after preflight but inside TTL; login switches principal; envelope revoked between scheduling and dispatch | Relevant capability/evidence invalidated; current action denied before handoff. No unauthorized fallback or implied permission renewal. |
| T17 | Same failed repair across new sessions, repeated review oscillation, exhausted finite candidate/search budget | Persistent counters and bounded pause; no fake Owner blocker and no endless unbudgeted turns. Useful distinct progress can continue within its remaining cap. |
| T18 | One required predicate waits for login; another is ready; no-ready timer; later authenticated permission probe | Independent work continues. One deduplicated permission request. Same run resumes after verified identity/capability; no secret transported through text. |
| T19 | Crash at each local transition/file-reference boundary; full disk, bounded DB busy failure, corrupted/unsupported schema | No dispatch after failed durability acknowledgement, no blank-state reset. Previous committed history or explicit recovery pause; missing evidence is not green. |
| T20 | Implementer claims review acceptance; reviewer has wrong tree/is a contributor; fixes lack retest/rereview | Claims cannot satisfy review/remediation. Correct independent current-basis receipts can; existing findings persist through disputes. |
| T21 | Edit STATUS to say complete; restart; require clean delivered checkout | Journal prevails, projection is rebuilt/dated, remote/source facts reconciled. No self-referential endless status commit cycle. |
| T22 | Complete first increment of a larger admitted run, with/without existing next-increment authority | No parent false completion. Automatically admit eligible next increment under existing delegation; otherwise show exact material authority gap, not a routine “continue” request. |

**Pinned-host conformance exit, additional to deterministic tests.** Run the actual installed Codex binary through the same launcher on a disposable task that needs at least two turns and independent verification. Capture real native turn-end events, host process identities, scheduling receipts and controller-rendered output. Verify no Owner message was supplied between turns. Restart the host-managed controller and demonstrate journal-based wake. Attempt a real harmless sandbox escape marker. Record actual versions/limitations; mocked receipts cannot satisfy this deployment exit.

**Migration/compatibility risk.** New trusted process and credential boundary; native runtime protocol drift; legacy tools bypassing the launcher; incompatible journal schema; STATUS readers expecting mutable authority. Migrate one opt-in run first. Import old evidence as unverified, re-ground, then record a cutover. Do not dual-write state or fall back to Markdown execution on DB failure. Rollback disables new dispatch, preserves the journal/effect identities and leaves readable projections; it does not delete history or restart a blank task.

**Owner decision required?** Approve any new security/process boundary, local private storage/retention and restart installation, operational-pause policy, finite resource limits and allowed model/review substitutions not already covered. Approve the bounded implementation itself. No repeated approval is required for later turns within that envelope.

**Exit label:** `LOCAL_AUTONOMOUS_BLOCK_E2E_VERIFIED` with exact supported scope. Not external delivery or unrestricted unattended execution.

## 4. Stage 3 — safe Git and narrowly required external effects

**Problem solved.** Make approved commit/push and specific external actions recoverable without blind replay, wrong-target publication, foreign staging, or false completion from process output.

**Likely existing surfaces.** Existing authorization/admission, delivery and evidence contracts; Codex delivery/review instructions and capability registry. Inspect `scripts/run_pef_activation.py` before reusing its bounded invocation ideas, but do not silently convert that existing read-only lane into an external-write runner. The new controller's protected adapter owns publication.

**Proposed additions.** Narrow Git/effect adapter modules under `scripts/autonomous_execution/`, versioned effect-intent/receipt records, remote observation routines and `extensions/tests/test_autonomous_execution_effects.py`. Start with owned local commit plus one named normal-push target. Add a service adapter only when an actual admitted task needs it and its retry/reconciliation semantics are documented.

**Still only policy / unsupported.** Arbitrary opaque non-idempotent replay remains denied. Git success cannot establish deployment, database migration or external behavior. No exactly-once webhook/business-effect guarantee without a verified provider contract. No default production, payment, account creation, PR, merge, force push or deployment authority.

### Deterministic acceptance tests

| ID | Input / exercise | Required oracle |
|---|---|---|
| T23 | Crash before durable intent, after intent, after reservation, after sink success, during receipt persistence | Never send without acknowledged intent. Every potentially dispatched unresolved operation is reconciled before retry. |
| T24 | Success with lost response; duplicate wake; same key/different payload; expired dedupe; delayed negative reads | No new logical key for the unresolved effect; conflicting payload rejected; uncertainty retained where replay is not proven safe. |
| T25 | Commit object created, process dies before local ref/receipt update | Same tree/parent/metadata object is recovered; guarded ref transition; no fresh timestamp duplicate commit. |
| T26 | Wrong repository/ref/principal, implicit upstream, unauthorized branch creation, revoked grant or changed target | Denied before dispatch. No force/alternate-target fallback. |
| T27 | Push succeeds then client fails; remote equals C, is descendant, or conflicts | Equality creates verified publication receipt without repeat push. Descendant follows the exact admitted predicate; conflict preserves both histories. |
| T28 | Upstream advances between read/write; ordinary fast-forward differs from a strict expected-old requirement | Adapter refuses unsupported conditional semantics; no claim preflight read was CAS; no force-with-lease workaround. |
| T29 | Foreign dirty hunks in an allowed path, broad staging, symlink/rename escape, replaced worktree, effectful helper | Only owned verified patch can be committed; unknown work preserved; actual containment and sanitized invocation enforced. |
| T30 | Remote publication valid but live proof fails; unknown relevant external operation exists | Implementation/publication dimensions may pass, overall completion cannot. Compensation remains a separately gated operation. |

**Migration/compatibility risk.** Existing general “normal push” strings need exact target/operation bindings. Credentials move behind the gateway; old remote side effects may have no operation IDs and require manual/safe read-back classification, not synthetic historical success. Ref races, CI hooks and provider consistency can reduce the admissible guarantee. Rolling back an adapter disables new dispatch but retains unresolved intents and a reconciliation path.

**Owner decision required?** Only missing exact repository/ref/service/principal permissions, branch creation, downstream CI effects, data/cost/security consequences or irreversible operations. Already sufficient named normal-push authorization is reused. Safe read-back/reconciliation is included explicitly in the grant; a force push or destructive compensation is never inferred.

**Exit label:** verified exact adapter/repository/ref/recovery class. Local plus Stage 3 exits constitute the minimum for an admitted increment requiring publication.

## 5. Stage 4 — broader unattended workers, only if needed

**Problem solved.** A measured throughput bottleneck that persists after sequential V2 works. It is not a prerequisite for solving early returns, durable continuation or truthful completion.

**Likely existing surfaces.** Codex bounded worker contracts, delivery/review skill, capability registry and the Stage 2 controller. Preserve manual-only AntiGravity unless explicitly authorized. No NovaStory runtime changes.

**Proposed additions.** Only bounded parallel scratch/patch producers or extra read-only reviewers; one canonical integrator remains. Add worker identity/resource-limit fixtures. Do not introduce multi-host leases as an automatic consequence of local parallelism.

**Still only policy / unsupported.** Universal agent correctness, unattended irreversible providers and multi-host availability remain outside the guarantee. No recursive swarm, autonomous grant administrator or self-updating controller.

**Deterministic acceptance tests.** T31: overlapping worker patches, stale bases and semantic dependency changes cannot integrate without serialized checks and required integrated-tree rereview. T32: contributor reviewer, worker escape, duplicate worker result and aggregate cost/concurrency race cannot bypass identity, resource limits or the single canonical writer. Re-run T01–T30; expanded concurrency must not weaken them.

**Migration/compatibility risk.** More process races, evidence attribution and aggregate cost accounting. Add one capability at a time; disable the pool and preserve its outstanding patches/receipts to return to sequential operation. A future multi-host proposal needs a separate failure model and actual sink/gateway fencing, not merely lease expiry.

**Owner decision required?** A new decision only for changed delegation, cost, isolation, provider diversity or activation of a manual-only lane. Defer the whole stage absent an observed benefit and approved bounds.

## 6. Recommended first implementation increment and release claim

Name the first increment **“Seal obligations and reject fabricated completion.”** Limit its implementation scope to Stage 1 reusable validation/contracts and targeted tests, with no new external writes or general supervisor. Include a documented Stage 2 integration probe plan; do not bury the early-turn defect behind a months-long platform program.

The first increment's acceptance is T01–T08 plus retained useful compatibility controls, code review and a precise evidence report. Its final report must state that automatic continuation is not yet installed. The next increment is the real Stage 2 vertical boundary and restart loop. A V2 rollout is not accepted merely because a simulator returns valid JSON or these proposed tests are listed in documentation.
