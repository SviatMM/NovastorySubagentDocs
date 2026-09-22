# Decision matrix — keep invariants, reduce machinery

The baseline and source IDs are defined in `SYNTHESIS.md`. Decisions are recommendations, not evidence of implementation. **KEEP NOW** means required in the minimum supported lane at the stated stage, not that every feature ships in Stage 1. **KEEP BUT SIMPLIFY** preserves the safety/liveness purpose with the specified smaller mechanism. **DEFER** requires a concrete trigger. **REJECT** rejects the stated variant, not a falsely attributed proposal.

## 1. All explicitly requested mechanisms

| # | Proposal | Decision | Minimum implementation / reason | Stage and source |
|---|---|---|---|---|
| 01 | Protected authority registry | KEEP BUT SIMPLIFY | Protected versioned grant rows in the same local journal; authenticated control input; worker cannot issue grants. No separate registry service or mandatory signing infrastructure. | 2; G sections 2–3, R-D03/D11 |
| 02 | Immutable obligation manifest | KEEP NOW | Seal each admitted revision, derive required predicates from product-local accepted requirements, reject duplicate IDs and omissions. Amend by authorized superseding revision, never mutate history. | 1; G sections 5–6/12, R-D02 |
| 03 | Predicate graph | KEEP BUT SIMPLIFY | Small acyclic prerequisite graph plus finite alternative-action records. No general AND/OR language, arbitrary graph engine or product knowledge duplication. | 1 schema, 2 scheduler; G sections 6–8, R-D02/D10 |
| 04 | Transactional journal | KEEP BUT SIMPLIFY | One local SQLite database with events and transactionally updated indexed state/outbox. No distributed event bus or competing Markdown authority. | 2; G section 4, R-D05 |
| 05 | Scheduler | KEEP BUT SIMPLIFY | One deterministic event loop selects an eligible action, dispatches sequentially, and persists timers. Fairness among ready predicates; bounded I/O waits. | 2; G sections 6/15, R-D10 |
| 06 | Response gate | KEEP NOW | Controller owns the user-facing final channel and recomputes completion/continuation. Model output is internal data; skipping an optional script cannot bypass it. | 2; G sections 12/15, R-D01 |
| 07 | Action gate | KEEP NOW | Check each canonical mutation/effect and each bounded sandbox-job grant before dispatch. No unrestricted worker write/network escape. JSON observation alone is insufficient. | 2 local, 3 external; G section 3, R-D01/D03/D11 |
| 08 | Capability freshness | KEEP NOW | Point-of-use generation/identity checks plus bounded probes and expiry. Pin executable/config where possible; TTL alone is not sufficient. Reuse current truth vocabulary. | 2, extended per adapter in 3; G section 14, R-D04 |
| 09 | Single-writer lock | KEEP NOW | Real process-held local lock for the managed resource, unique active claim and state-version checks. Duplicate resume is not a second writer. | 2; G section 9, R-D06 |
| 10 | Fencing epochs | KEEP BUT SIMPLIFY | Local controller generation rejects stale proposals/receipts. No timeout takeover; quiesce the old lane first. An integer does not fence an already-sent provider call. | 2; G sections 9/14, R-D06 |
| 11 | Effect gateway | KEEP BUT SIMPLIFY | A controller module owning only currently needed Git/service operations and credential handles. Not a service mesh, universal tool broker or public daemon. | 2 denies unsupported effects, 3 implements; G sections 2/13–14, R-D07/D11 |
| 12 | Idempotency/reconciliation | KEEP NOW | Durable logical operation identity, intent before dispatch, same-key/same-payload retry only under verified adapter semantics; explicit unknown outcomes. | 3 before any supported external write; G sections 13–14, R-D07/D08 |
| 13 | Evidence verifier | KEEP NOW | Trusted observations, requirement coverage, exact source/runtime binding, missing/stale evidence rejection, dependency invalidation and final receipt. | 1 strict contracts, 2 real producers, 3 remote verification; G section 12, R-D09 |
| 14 | Independent reviewer binding | KEEP NOW | Controller-assigned non-contributor review session over an immutable artifact, structured findings and rereview; same provider can be allowed by policy. No acceptance from prose alone. | 1 schema, 2 enforced assignment; G section 9, R-D09 |
| 15 | Content-addressed artifact storage | KEEP BUT SIMPLIFY | Reuse immutable Git objects; store small protected receipt/log files with verified digests and atomic publication. No new global CAS service or automatic cloud upload. | 2; G sections 4/12, R-D05/D09 |
| 16 | Distributed leases | DEFER | Single-host process lock suffices. Reconsider only for an approved multi-host requirement with real sink/gateway fencing and an explicit failure model. | Not required by Stage 4; G sections 4/9/14, R-D06 |
| 17 | Worker isolation | KEEP NOW | Baseline separation from canonical state, grants, trusted receipts and external credentials is required before strong gate claims. A writable scratch lane is not canonical authority. | 2; G sections 2/9, R-D11 |
| 18 | Budgets | KEEP NOW | Persist usage reservations, attempts, search, repair, review dispute and no-progress limits across turns/restarts. Reserve recovery capacity; unknown costs do not become zero. | 1 fields, 2 enforcement; G sections 3/8, R-D10 |
| 19 | Automatic multi-turn supervisor | KEEP NOW | Controller consumes a trusted turn-end event, records its next decision and launches another permitted turn without Owner input. Installed restart/wake capability is part of Stage 2. | 2; G sections 14–15, R-D01/D05/D10 plus this task's explicit requirement |

## 2. Other major Greenfield proposals

| Proposal / concern | Decision | Adjudication |
|---|---|---|
| Existing CPTO ownership, product-local admission, capability truth and bounded increments | KEEP NOW | Preserve current names, accepted requirement locations and specialist restrictions. The controller is execution infrastructure, not a new product requirements repository. [G-COMP sections 2–4] |
| Authenticated grant revisions and dispatch-time revocation | KEEP NOW | Local authenticated control channel and serialized authorization check; already-sent effects remain visible. Do not require signatures for a local protected store. [G sections 3/14] |
| Durable inbox/outbox and human request deduplication | KEEP BUT SIMPLIFY | Tables in the same database; request IDs and UI dedupe. No broker or new permission application. Login/2FA stays in the genuine provider UI. [G sections 4/10–11] |
| Outcome/admission execution lock and drift repair | KEEP BUT SIMPLIFY | Record policy/runtime/schema/adapter identities; re-probe affected capabilities and migrate only supported schema versions. No universal dependency solver or automatic host-wide upgrade. [G section 14] |
| Parallel isolated workers, patch integration and reviewer diversity | DEFER | Keep one implementation job at a time and a separate review assignment. Add parallel patch producers only for a measured bottleneck; preserve one canonical integrator. Mandatory provider diversity is not a default. [G section 9] |
| Automatic admission of later increments | KEEP BUT SIMPLIFY | Allowed only under existing outcome-level delegation, through the same admission checks and unchanged parent obligations. Otherwise report the exact missing authority, not a fake goal success. [G sections 5–6/15; G-RESULT D02] |
| Hash-chained event history | DEFER | ACLs, atomic transactions, sequence checks and retained evidence first. Hash chains do not protect against an actor that controls both history and trusted root. [G section 4] |
| Dedicated dashboard, generalized adapters, cloud backups, HA | DEFER | Add only for a demonstrated operational requirement and approved data placement. A local projection and a tested restore procedure are enough initially. [G sections 2/4/16] |
| Renaming all AI OS concepts to ABDEK | REJECT | Use Greenfield as input, not as a mandatory rebrand or wholesale replacement. The source proposes retaining AI OS policy; this rejection concerns an unnecessary adoption variant. |
| Registry-first platform build before proving turn continuation | REJECT | A long infrastructure prelude would delay the observed defect. Stage 1 correctness is followed immediately by a thin real Stage 2 vertical slice. |

All Greenfield G01–G09 concerns have a home: G01 -> action/authority gate; G02 -> Stage 3 effect protocol; G03 -> obligations/verifier; G04 -> journal/projection; G05 -> finite frontier; G06 -> actual multi-turn supervisor; G07 -> capability/schema generations; G08 -> bound review; G09 -> local ownership and guarded integration. No concern requires a distributed deployment in this task.

## 3. All Red Team minimum defenses

| Defense | Decision | Implementation decision and attack coverage |
|---|---|---|
| D01: actual action/response boundaries | KEEP NOW | Stage 1 malformed/claim rejection; Stage 2 real hooks and turn controller. A18/A30/A36–A37/A40/A46–A50/A52. |
| D02: preserved obligations | KEEP NOW | Sealed product-local manifest, applicability and replan checks. A04/A13–A14/A16–A17/A27/A31–A33/A37–A39/A52. |
| D03: effective scoped authorization | KEEP NOW | Protected local grants, exact targets and dispatch generation. A03/A06–A07/A15–A16/A22/A31–A33/A42/A45/A50. |
| D04: capability at point of use | KEEP NOW | Identity/generation/pinning plus adapter probes. A01–A04/A06/A12/A15/A26/A32/A50–A51. |
| D05: durable state and recovery | KEEP BUT SIMPLIFY | Single-host journal, retained artifacts, bounded storage failure and reconciliation. A08/A17–A21/A23/A25/A34/A46/A49. |
| D06: single active writer | KEEP BUT SIMPLIFY | Process lock, resource claim, local generation and quiescent restart; no distributed lease requirement. A08/A16/A20–A21/A23–A24/A44. |
| D07: safe effect identity and reconciliation | KEEP NOW | Required before the relevant write adapter is enabled; unsupported opaque replay denied. A04–A06/A08–A09/A19/A23–A24/A35/A45. |
| D08: owned Git and exact publication | KEEP NOW | Isolated integration/index, owned hunks, verified commit tree and remote observation. A09/A21–A24/A31/A34/A42–A44. |
| D09: evidence/review/acceptance binding | KEEP NOW | Trusted production of receipts, full obligation coverage, immutable review basis and invalidation. A03/A10–A13/A17/A22/A25/A28/A33–A36/A38/A40–A41/A47. |
| D10: budgets and honest stops | KEEP NOW | Finite candidate/search budget and explicit pause semantics; no universal alternatives proof. A01–A02/A05/A07/A10–A11/A14/A18/A20/A26–A29/A39/A49. |
| D11: real read-only/effect mediation | KEEP NOW | Basic isolation in Stage 2, not deferred with extra workers. Validate actual enforcement rather than static no-network claims. A40–A41/A50–A51. |

The attack coverage above covers A01–A52. It is traceability of proposed defenses, not a claim every attack is eliminated by a schema validator. Stage-specific tests distinguish structural, integrated-runtime and provider behavior.

## 4. Explicitly rejected failure modes

| Variant | Decision | Why |
|---|---|---|
| Caller-chosen empty applicability, worker pass booleans, review prose or process-exit success as completion authority | REJECT | They bypass obligations and observed evidence. |
| A blocked predicate becomes N/A or disappears during replan | REJECT | Availability cannot redefine an accepted outcome. |
| Model prose establishes `EXECUTION_WINDOW_LIMIT` | REJECT | Only a trusted host/runtime limit receipt can establish that class. |
| Optional response gate or “please continue” after every normal model turn | REJECT | The controller, not Owner, must own internal continuation. |
| No-return even when no safe budgeted work exists | REJECT | It produces endless loops, silent failures or dishonest blocker labels. |
| `STATUS.md` and database both writable authorities | REJECT | Contradiction resolution becomes ambiguous and replay unsafe. |
| Lease expiry or local epoch proves remote cancellation/exactly-once | REJECT | Old dispatched calls and provider uncertainty remain. |
| Kubernetes, etcd or distributed database mandatory for one Mac | REJECT | No evidence in the supplied research establishes that necessity. Neither agent is represented as requiring that stack. |
| Unrestricted same-privilege worker plus “protected” Markdown/DB | REJECT | It can bypass the proposed trust boundary. |
| New NovaStory Agent runtime, product requirements duplicated in AI OS, force-push fallback or implicit deployment | REJECT | Outside task, existing authority, or both. |
