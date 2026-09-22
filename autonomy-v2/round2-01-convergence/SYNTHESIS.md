# Autonomous Execution V2 — convergence

**Task:** `autonomy-v2-round2-01-convergence`  
**Status:** proposed implementation contract; no AI OS implementation or deployment performed.  
**Date:** 2026-09-22.

## 1. Decision

Retain AI OS. Add **one local execution controller**, not a renamed operating system or a distributed orchestration platform. Its minimum deployment is one trusted process, one protected local SQLite journal, one active isolated implementation lane, a small dependency scheduler, controller-owned action/response gates, and evidence verification. Add a narrowly scoped Git/effect adapter only after local continuation is enforced.

The decisive change is ownership of the execution boundary:

`Owner outcome -> existing product-local admission -> durable run -> controlled model turns and verification -> controller-authorized result`

A model turn is an attempt inside the run. It is not the run. When the model finishes a turn while eligible work remains, the controller schedules another turn or a deterministic action. It does not send the Owner an unfinished handoff asking for “continue.” A real host limit needs a host-produced receipt, not model prose. See `V2_CONTRACT.md` sections 5–7.

The minimum is achieved after Stages 1–2 for local work. Required publication becomes supported after Stage 3. Stage 4 is optional. Fixing the simulator alone must never be reported as fixing autonomous execution.

## 2. Baselines and evidence discipline

| Source | Exact commit | Use |
|---|---|---|
| Canonical AI OS | `713928f1425374bac6dfdf13ba842d582ce5c37b` | Pinned implementation/policy baseline; tree `80ddc05d1556828fbd2f1ef0476f0ac6d4cff443`. |
| Agent 02 Greenfield | `4562b8e453e3e37cc3d96ee09bf6f468322710f3` | Architecture, comparison, scenario inventory and result under `autonomy-v2/agent-02-greenfield/`. |
| Agent 03 Red Team | `1c53a1b09a802b1fb0ee3045d0b3c0568eecafe3` | Attack report, failure matrix, minimum defenses and result under `autonomy-v2/agent-03-redteam/`. |
| Agent 01 | No published result supplied | Excluded. No inferred findings or votes. |

These are immutable research inputs, not floating branch heads. Canonical source was read through the connected GitHub interface. Findings here are bounded to the inspected execution surfaces; they do not establish absence of an unobserved local supervisor on the Owner's computer.

Agent 02 reports design walkthroughs, not a running kernel. Agent 03 reports 22 reproduced adverse component probes, eight positive controls, and five existing simulator tests; its 52 scenarios are not 52 independent live exploits. Those executions are **inherited research evidence**, not tests newly run by this convergence task. This task performs source inspection and design synthesis. The acceptance tests in this package are specifications to implement, not claimed V2 test passes. [G-RESULT; R-REPORT sections 1, 4; R-RESULT]

## 3. What the current system gets right

Keep `AUTONOMOUS_BLOCK`, local CPTO accountability, product outcome/capability/increment/task separation, product-local requirements and admission, authorization envelopes, capability truth vocabulary, bounded repair, independent review, and the manual-only specialist boundary. Keep `GOAL.md`, `DELIVERY_MAP.md`, `CURRENT_INCREMENT.md`, and readable `STATUS.md`; change only the latter's execution-authority role when a run migrates. Ordinary implementation choices must not become Owner decisions. [A01–A06, A08]

AI OS is not simply “all Markdown”: the inspected simulator and bounded PEF runner are executable. Conversely, neither their existence nor a valid result record proves a general action/response interception loop. The simulator explicitly describes policy evaluation rather than orchestration. [A07, A10]

## 4. What must change

The inspected simulator accepts caller-selected applicability and direct events that mark tests, review, or push complete. Resume validation checks field presence without establishing usable provenance. The traceability validator builds a lookup before rejecting duplicate requirement IDs and can skip evidence-item checks for a truthy non-array plan. These are concrete reasons for strict input and obligation work, not a reason to rewrite the whole knowledge model. [A07, A09; R-REPORT sections 3–4]

The response rule currently lives in the execution instructions. An optional evaluator cannot prevent an agent from omitting it. V2 must own the actual user-facing completion channel and the launch of subsequent model turns. It must also mediate canonical writes and external effects: observing an execution event after a command ran is not an action gate. [A01, A06–A07; R-D01, R-D11]

`STATUS.md` currently owns verified present state in adaptive-delivery policy. V2 changes that explicitly: **journal = execution authority; STATUS = dated human-readable projection**. Product documents still define accepted product meaning, and remote observations still establish remote facts. The controller records those facts; it does not make Git or a database an oracle of semantic correctness. [A01, A04; G sections 4, 12; R-D05]

## 5. Convergence choices

Keep the Greenfield invariants but remove its deployment breadth. A predicate graph becomes an acyclic prerequisite table plus finite candidate records, not a general workflow language. Registry, scheduler, gates, verifier and effect gateway are modules of the same controller, not separately deployed services. Protected artifacts use existing immutable Git objects plus small digest-checked receipt files, not a new storage platform.

Keep the Red Team's three real boundaries: action, state/evidence, response. Do not defer basic isolation until unattended multi-worker rollout. A worker able to rewrite the controller, its journal, its grant, or its trusted receipts can defeat every proposed gate. Stage 2 therefore includes a tested process/filesystem/credential boundary. It does not require virtual machines for every reasoning turn, but it does require an actually enforced boundary. [G sections 2–4, 9, 15–16; R-D01–R-D11]

Use a process-held single-writer lock and local controller generation. Do not steal ownership on a heartbeat timeout. Restart first establishes that the former mutation lane is quiescent; stale proposals then fail generation checks. This is not distributed fencing of an external provider. Unknown effects still require reconciliation. Distributed leases are deferred, not an MVP dependency.

Use finite persisted usage, repair, search and no-progress budgets. A truthful `SAFE_OPERATIONAL_PAUSE` is allowed without inventing an Owner decision. It is neither success nor an excuse for routine early returns. No-return means “do not hand routine internal continuation to the Owner,” not “never report a safety or infrastructure stop.”

The complete adjudication, including all 19 explicitly requested mechanisms and all eleven Red Team defenses, is in `DECISION_MATRIX.md`.

## 6. Boundaries of the guarantee

The proposed guarantee is conditional and testable: inside the installed V2 lane, the model cannot alone weaken sealed obligations, broaden authority, certify its own claims, emit final success, or replay an unresolved effect. It can finish a turn; the controller then decides continuation from durable state.

This does not guarantee perfect requirements interpretation, flawless review, eventual service availability, survival of host/disk loss, or exactly-once business effects at arbitrary providers. A compromised trusted host/controller is outside the boundary. An unrestricted worker is not outside it: allowing that worker while claiming enforcement is a failed deployment.

The user's local runtime has not been inspected. Public runtime documentation establishes a candidate integration surface, not an installed capability. Stage 2 must pass the real pinned-runtime conformance and bypass tests before changing its capability record to `E2E_VERIFIED`. A missing supervisor must be reported as a deployment gap, not hidden behind “resume later.”

## 7. Owner decisions, without making Owner the scheduler

This research authorizes publication of this package only. A subsequent AI OS implementation increment needs separate authorization. Before enabling the controlled lane, resolve any genuinely missing approval for the host trust boundary, private local storage/retention and restart service, finite usage limits, operational-pause policy, and permitted runtime/reviewer substitutions. Reuse existing approvals where sufficient; SQLite selection or a routine repair is not inherently a new Owner decision.

Before external operations, resolve only missing exact repository/ref/service/principal and downstream-effect permissions. Broader parallel workers require a later decision only when they change cost, security, or other material commitments. No decision is needed merely to start another already authorized turn, rerun a safe test, investigate a bounded finding, or regenerate STATUS.

**First implementation increment:** seal product-local obligations and make fabricated, empty, stale, or weakened completion records fail closed, with the Stage 1 negative suite. Its exit must explicitly say “record correctness only”; the next increment proves actual automatic turn continuation and action/response mediation.

## 8. Source register

All A-references below are repository-relative to canonical AI OS at the pinned baseline. They are source locators, not copied private source. All G/R references are relative to the pinned research directories above.

| ID | Locator / inspected concern |
|---|---|
| A01 | `core/ADAPTIVE_DELIVERY.md`: authority layers, autonomous block, blocker and repair policy. |
| A02 | `core/WORK_ADMISSION_PROTOCOL.md`: admission, traceability, envelope and product-local boundary. |
| A03 | `templates/product-repository/goals/_template/plans/CURRENT_INCREMENT.md`: completion and verification contract. |
| A04 | `templates/product-repository/goals/_template/STATUS.md`: authored durable-state fields. |
| A05 | `adapters/codex/.agents/skills/deliver-large-projects/SKILL.md`: hierarchy and execution loop. |
| A06 | Same skill: worker and cross-session continuation sections. |
| A07 | `scripts/simulate_autonomous_block.py`: `evaluate`, `result`, `main`; blob `12bfed693b9444d8e39fe7d7a128dfc9bdcc89e7`. |
| A08 | `core/CAPABILITY_TRUTH_REGISTRY.md`: capability states and bounded claims. |
| A09 | `scripts/validate_admission_traceability.py`: `validate`; blob `996f0d865a5b22672a5805747fd0977d67dd8d6d`. |
| A10 | `scripts/run_pef_activation.py`: `validate`, `status_command`, receipt production; blob `a11781ddf7d564c18738ce46f769054f8f88442e`. |
| A11 | `extensions/tests/test_simulate_autonomous_block.py`: five legacy tests; blob `59dbdf4bb2121a6730f386050372941190fa4522`. |
| G | `GREENFIELD_ARCHITECTURE.md`, numbered sections. |
| G-COMP | `COMPARISON.md`, especially concerns table and G01–G09. |
| G-SC | `SCENARIO_RESULTS.md`, scenarios 01–20; design traces. |
| G-RESULT | `RESULT.json`: scope and declared limitations. |
| R-REPORT | `ATTACK_REPORT.md`: source register, P01–P22 and C01–C08. |
| R-Pxx | `ATTACK_REPORT.md`, recorded component probes P01–P22; inherited execution evidence. |
| R-Axx | `FAILURE_MATRIX.md`, attack A01–A52. |
| R-Dxx | `MINIMUM_DEFENSES.md`, defense D01–D11. |
| R-RESULT | `RESULT.json`: scenario inventory and execution limits. |

Official public implementation references checked on 2026-09-22: W1 `https://developers.openai.com/codex/noninteractive/`; W2 `https://developers.openai.com/codex/app-server/`; W3 `https://www.sqlite.org/wal.html`; W4 `https://git-scm.com/docs/git-push`; W5 `https://git-scm.com/docs/git-ls-remote.html`. Their documented interfaces inform adapter choices, not claims about the Owner's installed versions.
