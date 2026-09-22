# Trace schema і terminal classifier

**Статус:** proposed contract v1, не зібрані live дані. Protocol sources O1/O6/O7 та private-source locators визначені в [RUNTIME_DIAGNOSIS](RUNTIME_DIAGNOSIS.md). Схема нижче належить controller, не OpenAI API.

## 1. Три незалежні записи

1. `event`: що реально отримано/виконано на boundary, ким і коли.
2. `turn_observation`: який provider turn закінчився, з якими фактами і наскільки повними даними.
3. `gate_decision`: що робити з durable run після independent verification.

Не перетворювати model final text на event типу host timeout. `observed_protocol_status`, `terminal_category`, `contributing_causes` і `run_disposition` — різні поля. Classification допускає уточнення із збереженням попередньої revision після late events; не переписує raw history.

## 2. Event envelope

Усі локальні IDs — UUID/ULID або equally collision-resistant opaque IDs. Provider IDs копіюються без нормалізації. UTC використовується для кореляції; duration вимірюється monotonic clock у визначеному clock domain.

```json
{
  "schema_version": 1,
  "event_id": "example-event-001",
  "run_id": "example-run",
  "task_id": "example-task",
  "attempt_id": "example-attempt",
  "controller_epoch": "example-epoch",
  "boot_id": "example-boot",
  "process_generation": "example-app-server-generation",
  "threadId": null,
  "turnId": null,
  "rpc_id": null,
  "direction": "local",
  "source": "controller",
  "kind": "dispatch_intent_persisted",
  "received_at_utc": "2026-09-22T12:00:00Z",
  "monotonic_ns": 1000000000,
  "clock_domain": "controller-clock-example-epoch",
  "state_revision": 1,
  "payload_redacted": {},
  "payload_digest": null,
  "redactions": [],
  "sequence": 1
}
```

Це синтетичний приклад, не вимірювання. `threadId/turnId=null` допустимі до provider acknowledgement; вони не вигадуються із номера attempt. `direction` = `in|out|local`; `source` = `app_server|controller|os_process_monitor|owner_channel|tool_observer|capability_probe`. Model output має source `app_server`, але його payload class `model_content`: transport provenance не робить зміст повідомлення runtime доказом.

Обов'язково журналювати: RPC intent/send/response/error; `turn/started`, `turn/completed`; `item/started`, `item/completed`; error notifications; timer arm/reset/fire/cancel; authenticated cancel; process spawn/exit; input requests/replies/resolution; usage snapshots; persistence failure; verification/gate/notification outbox records. Невідомі protocol notifications зберігати, а не відкидати мовчки.

## 3. Turn observation: обов'язкові поля

| Група | Поля й тип / правило |
|---|---|
| Identity | `run_id`, `task_id`, `attempt_id`, `dispatch_id`, `rpc_id`, `threadId`, `turnId`, `controller_epoch`, `process_generation` |
| Runtime basis | `client_kind`, `client_version`, `codex_version`, binary digest, schema digest, model/provider/effort, OS build, effective-config digest, prompt/instruction-manifest digest, auth mode; жодних tokens/credentials |
| Timing | `start_monotonic_ns` перед dispatch, `ack_monotonic_ns`, `first_event_monotonic_ns`, `first_action_monotonic_ns`, `end_monotonic_ns`, `start_utc`, `end_utc`, `wall_duration_ms`, `observed_monotonic_duration_ms`, `clock_domain`, `duration_quality` |
| Terminal | `terminal_protocol_event` = method/status/turn ID/event ID або null; `terminal_category`; `classification_revision`; `evidence_event_ids`; `cause_confidence`; `contributing_causes`; `coverage_gaps` |
| Subprocess | `subprocess_exit` = role/PID/generation/exit_code/signal/observed_time/expected_shutdown/initiator або null; tool exits окремо від app-server exits |
| Timeout | `timeout_source` = provider/host/controller/stall/tool/rpc/hook або null; timer ID, scope, configured limit, arm/reset/fire times, cancellation request, basis of attribution |
| Tools | `tool_call_count`, `completed_tool_call_count`, counts by type, `last_tool`, `last_action`, active item IDs, detached child ownership state |
| Errors | `last_error` з structured code, `codexErrorInfo`, upstream status, `willRetry`, item/turn scope, occurrence event; message тільки sanitized локально дозволений |
| Usage | `usage_limit_state` = available/exhausted/throttled/unsupported/unknown; observation time/age, limit ID, reset time, percentages where returned, auth mode, error reference, token accounting coverage |
| Durable frontier | `contract_revision`, `authorization_revision`, `workspace_basis`, `state_revision`, `frontier_at_turn_end`, `incomplete_required_predicates`, `verification_receipt_ids`, `unknown_effects`, `frontier_completeness` |
| Gate | `owner_response_gate_decision`, `gate_reason`, `next_action_id`, `next_trigger`, `budget_snapshot_id`, `owner_action_required`, durable outbox ID |

Нульова кількість tool calls є valid observation, але невідомий usage не є zero usage. Відсутність error не дорівнює доведеній відсутності error, якщо trace неповний.

### Timing details

Фіксувати і тривалість від UI submission, і від фактичного `turn/start`: різниця може бути queue/initialization. Wall-clock duration і monotonic duration окремі; зміна годинника не повинна створювати negative duration. Preflight перевіряє семантику chosen Mac clock щодо sleep; журналює sleep/wake або detection gap. Не віднімати monotonic stamps різних boot/clock domains.

Після crash без terminal stamp `end_monotonic_ns=null`, `duration_quality=right_censored`. Остання persisted activity дає нижню межу, restart observation — верхню лише для wall interval. Не вигадувати точний час смерті. Lifetime resource accounting після restart консервативне; зміна UTC назад або невідомий sleep не відновлює витрачений budget.

### Tool counting

Рахувати distinct execution items за `(threadId,turnId,item.id)`, не output deltas. Призначення item type визначає pinned adapter: shell/MCP/file changes тощо. Plan update, reasoning і текст assistant не є tool execution. Poll/wait рахувати окремо; persisted counters не збільшуються від duplicate events. Exit 1 тесту може бути expected negative test і не класифікується автоматично як terminal TOOL_FAILURE.

### Usage

O7 має wire codes `usageLimitExceeded`, `rateLimitExceeded`, `sessionBudgetExceeded`, `contextWindowExceeded`, `serverOverloaded` та інші. Невідомий код зберігати. `error.willRetry=true` означає, що runtime повідомив про retry, а не terminal failure. Account `rateLimits`/`rateLimitsByLimitId` підтримка залежить від auth/runtime: unsupported залишається unsupported. Snapshot біля 100% без causal rejection не доводить причину завершення.

Для `thread/tokenUsage/updated` зберігати raw counters і epoch. Cumulative snapshot не додавати до попереднього cumulative snapshot. Adapter має довести delta semantics для installed schema; compaction/new thread не обнуляють run-level spent. Unknown або regression counter → conservative reservation і coverage gap, не negative spending. Tokens не є точним ChatGPT fair-use meter або грошовим рахунком.

## 4. Нормалізовані категорії

| Category | Достатня спостережувана підстава | Що категорія НЕ означає |
|---|---|---|
| `MODEL_TURN_COMPLETED` | Matching `turn/completed` з `turn.status=completed`, без stronger causal forced termination | Не доводить мотивації моделі або завершення outcome |
| `HOST_TURN_TIMEOUT` | Trusted outer host hard-deadline record, пов'язаний з цим turn і фактичним interrupt/kill, або документована typed host timeout response | Не будь-яка хвилинна пауза; не MCP timeout |
| `PROCESS_EXIT` | Unexpected app-server або owning host process exit із wait/OS evidence, коли раніше немає пояснювального terminal event | Не stdout EOF без exit observation; не shell test exit |
| `STALL_TIMEOUT` | Наш watchdog fire за записаною inactivity policy та subsequent stop action | Доводить watchdog decision, не що reasoning реально зависло |
| `USAGE_LIMIT` | Matching terminal/rejected-start usage or rate-limit error; subtype quota або throttle | Не довільний 429; не unsupported account read; не infinite retry permission |
| `TOOL_FAILURE` | Causal terminal chain до конкретної fatal tool/runtime operation | Не будь-який recoverable tool error перед успішним turn |
| `USER_CANCEL` | Authenticated user cancellation ID пов'язаний з interrupted turn/termination | Не кожен `interrupted`; controller watchdog не user |
| `UNKNOWN` | Немає достатніх даних; unexplained interruption, unobserved host, EOF-only, conflicting trace | Не OWNER_BLOCKER і не execution-window evidence |
| `RESOURCE_BUDGET` | Controller lifetime budget або typed `sessionBudgetExceeded`; subtype/source обов'язкові | Не host wall timeout або акаунтний fair-use |
| `CONTEXT_LIMIT` | Typed `contextWindowExceeded`/verified compaction failure chain | Не привід почати новий run зі свіжими authority/budgets |
| `PROVIDER_FAILURE` | Terminal structured transport/server/sandbox/policy failure з subtype | Не привід автоматично повторювати policy-restricted дію |

Категорії описують **terminal observation**, а `root_cause_candidates` описує гіпотези. Не додавати прихований universal timeout із model text. `EXECUTION_WINDOW_LIMIT` — сумісний legacy projection тільки для externally evidenced фізичного execution deadline, не synonym усіх park states.

### Causal classifier algorithm

```text
1. Persist all evidence; match run/thread/turn/process generation.
2. Build ordered causal chain: request -> event -> stop action -> terminal/exit.
3. If terminal completed was already observed before an intentional shutdown,
   classify the turn as MODEL_TURN_COMPLETED; record shutdown separately.
4. If a trusted cancellation/deadline/budget action caused interruption/exit,
   classify by that initiator; retain actual provider status and exit details.
5. Otherwise interpret typed terminal/rejected-start errors via pinned adapter.
6. Otherwise unexpected process exit -> PROCESS_EXIT; EOF-only -> UNKNOWN.
7. Otherwise matching completed status -> MODEL_TURN_COMPLETED.
8. Otherwise UNKNOWN; do not fill missing causality using assistant prose.
9. Reconcile late events with a new classification revision, never erase history.
```

Race приклад: deadline fires одночасно з completed. За неможливості встановити causal ordering зберегти `UNKNOWN` або `cause_confidence=ambiguous` і обидві події; не застосовувати примітивний priority list, що завжди звинувачує timeout. Valid completed turn після recovered tool error має contributing tool failure, але primary MODEL_TURN_COMPLETED.

## 5. Durable frontier і gate receipt

Мінімальний task record: stable ID, admitted requirement IDs, dependencies, action class, scope, status, attempt budget key, evidence requirements, blocking reason і equivalent alternatives. Worker може запропонувати task/result; controller перевіряє й записує authoritative state.

```json
{
  "schema_version": 1,
  "run_id": "example-run",
  "state_revision": 14,
  "contract_revision": 1,
  "authorization_revision": 1,
  "frontier_completeness": "verified",
  "incomplete_required_predicates": ["regression-tests"],
  "frontier_at_turn_end": [
    {
      "task_id": "run-regressions",
      "dependencies_satisfied": true,
      "authorized": true,
      "evidence_fresh": true,
      "retry_safe": true,
      "budget_available": true,
      "eligible": true
    }
  ],
  "unknown_effects": [],
  "owner_response_gate_decision": "CONTINUE_INTERNAL",
  "gate_reason": "verified eligible work remains",
  "owner_action_required": false,
  "next_action_id": "run-regressions",
  "next_trigger": "controller_dispatch_outbox",
  "terminal_notification_allowed": false
}
```

Приклад не є receipt від виконаного тесту. Eligibility — conjunction актуальних dependency/authority/ownership/evidence/resource checks. Невідомий список задач не є порожнім verified frontier. Якщо unmet predicate не має task, gate створює bounded internal planning action або parks із `STATE_INCOMPLETE`, але не final success.

Gate decisions: `CONTINUE_INTERNAL`, `VERIFY_INTERNAL`, `REPLAN_INTERNAL`, `WAIT_RUNTIME`, `WAIT_USAGE`, `WAIT_INPUT`, `FINAL_COMPLETE`, `PARKED_BUDGET`, `PARKED_RUNTIME`, `PARKED_UNKNOWN_EFFECT`, `PAUSED_USER`, `OWNER_DECISION_REQUIRED`, `EXTERNAL_PERMISSION_REQUIRED`. `WAIT_INPUT` не автоматично human gate: спочатку перевірити наявні authorized answers. Одна blocked dependency не приховує незалежні eligible tasks.

## 6. Мінімальний durable storage contract

Proposed SQLite на локальному диску, один controller writer, append-only events і transactional materialized state. `STATUS.md` — projection, не другий mutable authority. WAL + `synchronous=FULL` + bounded busy timeout є proposed settings; filesystem failure все одно потребує recovery. Дані controller поза write sandbox worker.

Логічні таблиці (реалізація може нормалізувати JSON payloads):

```sql
CREATE TABLE runs (
  run_id TEXT PRIMARY KEY,
  revision INTEGER NOT NULL,
  contract_revision INTEGER NOT NULL,
  authorization_revision INTEGER NOT NULL,
  status TEXT NOT NULL,
  state_json TEXT NOT NULL
);
CREATE TABLE events (
  run_id TEXT NOT NULL REFERENCES runs(run_id),
  sequence INTEGER NOT NULL,
  event_id TEXT NOT NULL UNIQUE,
  envelope_json TEXT NOT NULL,
  PRIMARY KEY (run_id, sequence)
);
CREATE TABLE dispatches (
  dispatch_id TEXT PRIMARY KEY,
  run_id TEXT NOT NULL REFERENCES runs(run_id),
  state_revision INTEGER NOT NULL,
  kind TEXT NOT NULL,
  state TEXT NOT NULL,
  request_json TEXT NOT NULL,
  thread_id TEXT,
  turn_id TEXT
);
CREATE UNIQUE INDEX one_unresolved_dispatch_per_run
ON dispatches(run_id)
WHERE state IN ('reserved','sent_unknown','acknowledged','running');
CREATE TABLE budgets (
  run_id TEXT NOT NULL REFERENCES runs(run_id),
  budget_key TEXT NOT NULL,
  used INTEGER NOT NULL CHECK (used >= 0),
  reserved INTEGER NOT NULL CHECK (reserved >= 0),
  ceiling INTEGER NOT NULL CHECK (ceiling >= 0),
  PRIMARY KEY (run_id, budget_key)
);
CREATE TABLE gate_outbox (
  outbox_id TEXT PRIMARY KEY,
  run_id TEXT NOT NULL REFERENCES runs(run_id),
  state_revision INTEGER NOT NULL,
  kind TEXT NOT NULL,
  body_json TEXT NOT NULL,
  delivery_state TEXT NOT NULL,
  UNIQUE (run_id, state_revision, kind)
);
```

Enable foreign keys explicitly. `BEGIN IMMEDIATE`: verify state revision/contract, append observation, update verified state, charge/reserve budgets, create one continuation OR terminal outbox record, commit. Dispatch тільки після commit. Next turn cancellation/intent revisions повторно перевіряються перед send. Ніколи не тримати SQL transaction відкритою під час network/RPC wait або model execution.

Важливо: local transaction не атомарна із `turn/start`. `sent_unknown` — реальний стан, не автоматичне право повторити RPC. Unique RPC ID або `clientUserMessageId` самі не дають documented exactly-once semantics. Recovery описаний у SUPERVISOR_DESIGN.

## 7. Failure signatures і privacy

Persistent failure key = normalized error category/code + action class + stable obligation ID + relevant tool/model version + failing check identifier. Відкинути timestamps і випадкові request IDs. Artifact hash зберігати як evidence, але не як єдиний failure key: косметична зміна файлу не повинна reset budget. Додаткові caps per obligation і per run обмежують signature churn/replan renaming.

Public export містить allowlisted metadata, агреговані counters, normalized terminal categories, synthetic IDs, durations, verifier/gate outcomes. Не публікувати raw prompts, assistant/tool text, command args, environment, auth events payloads, transcripts, absolute workspace paths, account IDs, URLs із query credentials чи private filenames. Redaction до стандартного логування; доступ до додаткового локального sensitive capture — окреме свідоме рішення, обмежені права й retention. Digest приватного тексту не замінює перевірку ризику його публікації.

Raw protocol coverage може бути достатньою без збереження чутливого content: method, status, error discriminant, IDs, item class, time й sanitized metadata. Missing fields позначати redacted/unsupported, не вигадувати. Public safety scan необхідний, але regex scan не є доказом відсутності всіх secrets.

## 8. Acceptance oracle для classifier

Replay fixtures з virtual clock: normal completion + TODO; genuine user interrupt; host deadline; stall deadline; recovered tool error; fatal tool failure; usage rejection; provider overload; process death; EOF без exit; completed-then-shutdown; lost ACK; unknown new enum; duplicate event; late terminal; canceled stale dispatch; missing usage; clock-domain change. Кожен має expected terminal category, separate run disposition і gate outcome. Це майбутні component tests, не вже виконаний runtime experiment.
