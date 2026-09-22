# Single-Mac supervisor: continuation і response gate

**Статус:** proposed implementation boundary, не запущений сервіс. Scope: усунути ручне Continue для admitted safe local work, не перебудовувати AI OS. Source IDs O1–O8/R2/R3 визначені в [RUNTIME_DIAGNOSIS](RUNTIME_DIAGNOSIS.md); journal contract — [TRACE_SCHEMA](TRACE_SCHEMA.md).

## 1. Найменша достатня топологія

```text
Owner: admit / status / cancel / real decision
                 |
       local controller process
       + process-held lock
       + SQLite run journal + budgets + outbox
       + independent state verifier / response gate
       + stdio JSONL App Server adapter
                 |
         owned codex app-server
                 |
       one isolated approved worktree
```

Один process на Mac, одна активна run ownership, одна App Server connection, один active provider turn. Python 3 з subprocess/asyncio, sqlite3 і process lock є достатнім implementation choice; distributed queues, cloud scheduler, Linear, Kubernetes чи fleet database не потрібні. VERSION/SHA і точну dependency версію вибрати й зафіксувати під час implementation preflight, не змінювати автоматично посеред run.

State directory задає оператор як `$CONTROLLER_STATE_DIR`, workspace — `$WORKSPACE`; це placeholders, не персональні шляхи. Controller state/config/verifier розміщені поза writable workspace. Sandbox має **реально** забороняти worker їх змінювати; chmod під тим самим OS user або назва прихованої папки не є такою гарантією. Перевірити це negative probe перед admission.

MVP допускає лише ізольовані локальні reversible edits/tests із sandboxed execution, без production credentials, платежів, external writes і deployment. Network та MCP surface звужені до явно потрібного. Не переходити у full access для обходу approvals або читання journal. Git push/інші ефекти, якщо вони required у реальному outcome, залишаються pending до окремого authorized executor; не позначати їх N/A для красивого final.

## 2. Реальний response boundary

Controller володіє **єдиним terminal output channel**, який обіцяє Owner final/blocker. Model message deltas, tool output та навіть model final зберігаються як internal observations; вони не пересилаються як завершена відповідь. Owner може читати explicit status view, але status view не завершує run і не просить Continue.

Gate викликається самим event loop після кожного terminal event, reconciliation, зміни authority або виконаного verifier action. Модель не повинна пам'ятати викликати optional script. Простий `owner_response=false` у файлі без interception цього каналу нічого не гарантує.

### Дані, які gate перевіряє

Immutable admitted requirement IDs і applicability reasons; поточні contract/grant revisions; фактичну owned workspace identity й relevant artifact hashes; nonempty required predicates; completion receipts від незалежних checks; pending findings; task dependency frontier; unresolved effects/children; persistent budgets. Markdown STATUS і worker claim — inputs/projections, а не proof.

Для кожного незакритого requirement має існувати eligible task, blocked task із реальною причиною або bounded planning/verification action. Unmapped requirement не перетворюється на empty successful queue. Пропозиція reviewer не додає повноважень; ACCEPT має бути прив'язаний до reviewed artifact і не підміняє required tests. Якщо повноцінного verifier для acceptance ще немає, run не отримує сильнішої гарантії через сам факт наявності SQLite.

### Gate ordering

```text
honor authenticated cancellation / revoked authority
-> validate ownership, durable state and current contract
-> reconcile outstanding dispatches, effects and live child operations
-> verify changed evidence; invalidate stale checks
-> compute all eligible admitted work, including repair and verification
-> enforce lifetime/attempt budgets
-> choose independent safe work if one dependency is blocked
-> reserve continuation or verification in a transaction
-> otherwise final only when all required predicates are verified
-> otherwise honest operational park or real decision/permission request
```

`turn.status=completed + eligible frontier + resources available` → `CONTINUE_INTERNAL`, ніколи progress-only final. Технічний REJECT → repair/retest/rereview в межах budget. Native failed turn може вести до safe retry або до park, залежно від cause та replay safety; не auto-retry все підряд.

Перед dispatch повторно перевірити current revision і cancel flag. Gate/outbox decision та budget reservation commit разом. Terminal notification має унікальний `(run_id,state_revision,kind)` і містить current acceptance/evidence basis. Власна terminal UI дедуплікує за outbox ID; для стороннього notification sink без idempotency можливий duplicate delivery після crash, але це не має створювати нове виконання.

**Межа native UI:** цей controller не перехоплює повідомлення в іншому вже відкритому Codex UI. Для гарантованого channel gate запускати роботу через controller-owned interface. Під'єднаний до того самого writable thread другий UI повинен бути read-only або від'єднаний. Runtime safety і response suppression не забезпечуються тим, що controller просто дивиться на чужий transcript.

## 3. Поточний App Server contract

Official O1 і pinned source O6/O7 перевірено 2026-09-22. Публічний main source не дорівнює installed release. Preflight зберігає `codex --version`, executable digest і генерує installed contract:

```sh
codex app-server generate-ts --out ./schemas
codex app-server generate-json-schema --out ./schemas
```

Ці команди виконуються в disposable controller build area, не в canonical AI OS. Adapter підтримує тільки протестований schema digest/build; unknown contract → `PARKED_RUNTIME`, не silent downgrade. App Server має version-sensitive/experimental surfaces; використати stdio, не experimental WebSocket transport для MVP [O1].

### Startup wire skeleton

Запустити `codex app-server` як owned child, stdout лише protocol, stderr окремий sanitized diagnostic stream. JSONL messages з id/method/params; на цьому wire `jsonrpc` поле не додається [O1]. Приклад structural messages, не готовий grant:

```json
{"id":1,"method":"initialize","params":{"clientInfo":{"name":"local_runtime_supervisor","version":"0.1.0"}}}
```

Дочекатися відповіді initialize, потім:

```json
{"method":"initialized","params":{}}
{"id":2,"method":"thread/start","params":{"cwd":"<approved-absolute-workspace>","sandbox":"workspace-write","approvalPolicy":"on-request","ephemeral":false}}
```

`cwd` placeholder обов'язково замінюється на перевірений absolute path. `sandbox` enum перевірено у pinned `SandboxMode.ts`; actual effective sandbox/network policy ще підлягає preflight. `on-request` не означає blanket approval: responder нижче дозволяє лише підмножину чинної authority. Прийняти `result.thread.id`, зафіксувати journal, тоді:

```json
{"id":3,"method":"turn/start","params":{"threadId":"<returned-thread-id>","input":[{"type":"text","text":"<admitted initial work packet>"}]}}
```

Прив'язати повернений turn ID та notifications до durable dispatch. Після `turn/completed` прочитати `params.turn.status`: `completed`, `failed` і `interrupted` відрізняються. У notification `turn` містить ID; не шукати вигадане root `turnId`. `error` notification має `willRetry`, тож сам error ще не обов'язково terminal [O6,O7].

Наступна authorized дія — новий `turn/start` на тому самому `threadId` з compact continuation packet: run/state revisions, збережений outcome/constraints reference, завершені verified факти, unmet predicates, selected safe action, budget remainder, latest error evidence. Не ресендити весь початковий prompt кожного разу і не використовувати голе Continue без state basis.

Важливо: поточний `TurnStartParams` містить поведінку steering already-active turn. Тому власна one-active-turn invariant обов'язкова; не покладатися на очікуваний server error при другому start. `turn/steer` — active-turn steering, не заміна continuation після completed [O6].

### Інші потрібні boundaries

| Механізм | Семантика / рішення adapter |
|---|---|
| `thread/resume` | Відновити persisted thread за ID після reconnect/restart; потім нові turns на ньому [O1]. |
| `thread/read` з `includeTurns:true` | Звірити stored state/history; саме читання не є resume/subscription [O1]. |
| `thread/fork` | Створює інший thread; не використовувати як непомітний reset budget [O1]. |
| `turn/interrupt` | Передати matching thread/turn IDs; RPC ACK не доводить завершення. Чекати terminal/exit, drain і reconciliation [O1]. |
| `thread/tokenUsage/updated` | Accounting evidence із thread/turn IDs; deduplicate counters, unknown coverage explicit [O6]. |
| `account/rateLimits/read` / `account/rateLimits/updated` | Optional observable account state; unsupported не означає unlimited [O1]. |
| `thread/compact/start` | Context maintenance із власним lifecycle; не одночасно з task turn, не reset run [O1]. |
| `thread/goal/*` | Optional native goal driver. У recommended controller MVP не активувати паралельно з external continuation [O1,O3]. |

## 4. Event loop — execution sketch

Це алгоритм для реалізації, не код уже працюючого supervisor:

```text
acquire process-held exclusive lock; open/validate journal
recover ownership, persisted authority, budgets and ambiguous dispatches
start or reuse only our verified app-server; initialize once per connection
resume stored non-ephemeral thread, or create initial thread once

while run is nonterminal:
    multiplex stdout protocol, stderr, child-exit, timers, Owner commands
    persist observations and match outstanding RPCs/server requests
    update provisional state; never publish raw model final

    on matching terminal or reconciled process loss:
        drain/account remaining known operations within grace budget
        independently verify worktree and receipts
        classify observed terminal cause
        gate = evaluate(current durable run)
        transactionally record gate and reserve its next action

        if gate continues:
            recheck cancellation/revisions/ownership
            send turn/start on same idle thread
        elif gate requires controller verification:
            run bounded verifier; persist measured receipt; re-evaluate
        elif gate waits:
            register durable next trigger; keep actual supervisor alive
        else:
            publish one honest final/park/decision notice from outbox
```

Protocol reader має працювати під час очікування відповіді на власний RPC: App Server може сам надіслати request. Concurrent stdout/stderr drains запобігають pipe backpressure deadlock. Обмежити розмір line/buffer; oversized/invalid message → structured protocol failure і bounded recovery, не зависання і не truncation JSON з вигаданою відповіддю.

Verifier не запускає довільний рядок із worker STATUS на довіреному host. Preapproved check adapters отримують scoped inputs і самі sandboxed; неперевірені repository scripts можуть мати side effects. Protect controller evidence/check definitions і bind results до artifact hashes. Після edits старі залежні test/review receipts invalidated.

## 5. Timers, stalls і resource envelope

У inspected `turn/start` немає timeout field [O6]. Наведені нижче значення — **proposed conservative pilot policy**, не обіцянка або limits Codex. Всі budgets persistent на run і obligation, не на нову сесію. Достатня верхня межа ресурсів потребує реального interrupt/kill enforcement і grace accounting; токенний ліміт за delayed observations є stop threshold, не гарантований точний billing cap.

| Budget | Proposed pilot ceiling | При вичерпанні / exact resume condition |
|---|---|---|
| Hard turn deadline | 20 min від dispatch, плюс 10 s interruption grace | HOST_TURN_TIMEOUT, reconcile; повтор лише safe, у retry budget |
| Event silence | warning 90 s; interrupt після 5 min без релевантної provider activity, якщо немає bounded known tool/input wait | STALL_TIMEOUT; перевірити process/tool state, bounded restart |
| Whole run | 4 h elapsed lifetime від admission, включно з очікуванням; restart не поновлює | PARKED_BUDGET; explicit resource-envelope extension для того самого run |
| Provider turns | 40 issued attempts; count unknown-sent і maintenance; native loop off | PARKED_BUDGET; нова authorized ceiling або revised scope decision, не reset ID |
| Observed tokens | 300000 run-level tokens, з per-dispatch reserve і обліком coverage | Stop threshold; точний spending cap тільки з підтриманим billing control; без достатніх даних strict cap run не admit |
| Same failure signature | 2 failed attempts тим самим підходом | Змінити explanation і bounded diagnostic plan, не третій identical repair |
| Repair attempts | 6 per stable obligation, 12 total per run | PARKED_BUDGET; конкретне нове evidence/approved budget revision |
| No verified progress | 2 turns → replan; максимум 2 such replans per run | PARKED_BUDGET; зміна діагнозу/обґрунтований revised plan, не cosmetic edit |
| Reviewer dispute | 2 repair/review rounds + 1 independent adjudication | PARKED_BUDGET або справжнє material decision; не forced ACCEPT |
| Alternative search | 3 authority-equivalent alternatives per obligation, 2 probes кожна; max 12 probes per run | PARKED_BUDGET із explored set; не вимога довести всі можливі альтернативи світу |
| Transport retry | 3 retries на logical request, delays 1/5/15 s, тільки після replay-safety determination | PARKED_RUNTIME; restored transport + reconciled operation |
| App Server restart | 2 per run | PARKED_RUNTIME; healthy capability probe й підтверджена ownership |
| RPC response wait | 30 s proposed; timeout не доводить, що request не прийнято | sent_unknown → reconciliation перед retry |
| Pending genuine input | 30 s на local policy resolution; далі notify once, bounded wait до run deadline | Validated answer/permission event matching request і authority |
| Known usage reset | Не retry раніше reset; 3 refresh attempts, bounded by whole-run budget | WAIT_USAGE; fresh allowed state, same principal, remaining resources |
| SQLite busy | 5 s; write/fsync failure негайно зупиняє нові dispatches | PARKED_RUNTIME; storage health + integrity/reconciliation check |

Цифри мають бути окремо прийняті для installation, а не вважатися дозволом витратити ресурси під час цього research. Lower ceilings можна обрати без розширення повноважень.

Розрізняти `last_byte`, `last_provider_event`, `last_tool_progress`, `last_verified_work_progress`. Порожній heartbeat не є useful work; reasoning silence не є доведеним deadlock. Known long tool отримує окремий finite declared tool deadline; input wait має власний timeout. Ці винятки не обходять hard turn і lifetime wall budgets. OS sleep/wake переноситься в trace, не переписується як хвилинний host timeout.

## 6. User input й approvals

Мінімальний dispatcher підтримує actual request IDs для `item/commandExecution/requestApproval`, `item/fileChange/requestApproval`, `item/permissions/requestApproval`, `item/tool/requestUserInput`, `mcpServer/elicitation/request`, а також version-supported authentication requests. Shapes генеруються зі встановленого schema, а не з пам'яті [O1].

Local policy responder дозволяє тільки точну preauthorized action/subset із matching workspace/grant. Не використовувати acceptForSession для невизначеного набору майбутніх дій. Unknown request/authority → decline/cancel, або permission wait без нових ефектів. `autoResolutionMs` не є дозволом винайти відповідь. `serverRequest/resolved` може означати cleanup; не трактувати як consent.

Якщо модель просить факт, уже підтверджений admitted record, controller може відповісти з evidence reference. Product/security/spending change → справжнє OWNER_DECISION_REQUIRED. Login/2FA/OS consent → EXTERNAL_PERMISSION_REQUIRED. Складність алгоритму або вичерпання repair budget — не Owner decision. Коли input блокує активний turn, безпечне припинення/збереження wait дозволяє пізніше виконати незалежні задачі, але тільки після quiescence; не стартувати конкурентний turn поверх нерозв'язаного активного.

Не логувати credential values і не обіцяти auto-login. Якщо доступна тільки небезпечна approval escalation, park, а не full access. User cancel персистується до interrupt і ніколи не викликає automatic continuation, включно з restart.

## 7. Process death, restart і context

**Що має вижити після App Server restart:** run/contract/grant revisions, canceled/parked state, thread ID, всі dispatch intents і unknown states, budget counters/reservations, accepted evidence/failure signatures, worktree identity/baseline і local artifacts, configured non-ephemeral Codex session storage, binary/schema/config identity, pending input IDs та їхній lifecycle, timer/resource policy, notification outbox. Auth зберігається підтриманим Codex механізмом, не копією токена в SQLite.

Відновлення thread дає conversation continuity, але не відроджує загиблий shell process і не обіцяє replay всіх transient notifications. `thread/read`/`thread/resume` та фактичний workspace reconciliation доповнюють journal; transcript не є transaction log [O1].

Recovery sequence:

1. Взяти lock; перевірити DB/schema, run status і generations. Cancelled run не оживає.
2. З'ясувати, чи old child App Server/tool process ще може писати. PID + process generation/start identity; без broad `pkill` і без довіри до PID після reuse.
3. Quiesce тільки owned процеси з bounded grace, verify stopped. Lock controller сам не зупиняє orphan child. Якщо ownership невизначена — park, не takeover.
4. Перевірити workspace identity, current diff і evidence. Ніколи автоматично reset/clean чужі changes.
5. Запустити pinned App Server з тим самим supported persisted session store; initialize; `thread/resume` того самого ID; read/reconcile turns.
6. Закрити confirmed dispatches. Unknown sent request без явної provider кореляції не resend blindly. `clientUserMessageId` може допомогти кореляції, але не задокументований тут як idempotency key.
7. Recompute frontier і budget; лише після цього наступний turn.

Якщо `turn/start` був прийнятий, а ACK втрачено, друга відправка може означати нову/steered роботу. Controller не гарантує exactly-once provider dispatch. У local reversible MVP можна після quiescence звірити artifacts і спланувати safe remaining work; повторювати patch із нуля небезпечно. Для uncertain irreversible/external effect — PARKED_UNKNOWN_EFFECT і authoritative read-back/receipt; поки повного adapter немає, не replay.

Context finite: same thread не означає безмежну пам'ять. Під час compaction зберігати durable outcome/authority/evidence, maintenance turn і витрати. Якщо persisted thread unrecoverable, bounded recovery може створити replacement thread з explicit lineage і verified context packet лише після quiescence; той самий run і всі budgets. Заборонено підмінювати model/provider для обходу quota/policy або приховувати втрату контексту.

## 8. Native `/goal` / Stop hook як менший pilot

Можна окремо випробувати Goal mode, або синхронний `Stop` hook, який читає controller-verifiable durable state. Official Stop continuation shape [O4]:

```json
{"decision":"block","reason":"Verified admitted work remains: run the recorded next safe action within the existing budget."}
```

Hook має працювати синхронно; background hook не керує continuation. `continue:false` означає stop, не продовження. `stop_hook_active` — telemetry, не правило безумовно дозволити другий stop. Повторно перевіряти state і persistent counters на кожному invocation. Transcript file format не stable API; не парсити його як єдиний authority. Hook виконує лише коротку перевірку, не повну test suite.

Це хороший **measured pilot**, але не автоматичний substitute для строгого own-channel gate: `suppressOutput` не реалізований, tool hooks мають coverage exceptions, hook timeout/failure behavior треба виміряти, а Interrupt hook не скасовує user interrupt [O4]. Не запускати hook, Goal mode і controller одночасно як continuation owners. Чітко записати `continuation_driver=native_goal|stop_hook|external_controller`.

## 9. Uptime і Symphony

Foreground MVP працює, поки живий controller і доступний Mac. Для continuation після його crash потрібен **дійсно встановлений supervisor**, наприклад локальний macOS LaunchAgent із restart policy та тим самим lock/recovery entrypoint. Це implementation stage, не можливість цього дослідницького чату. LaunchAgent не забезпечує роботу при вимкненому Mac, logout conditions, loss of storage чи network; фактичні умови startup/resume треба перевірити. Не обіцяти необмежену background роботу.

Symphony демонструє корисну схему same-live-thread continuation, event loop, timeout і retry/reconciliation [O8]. Його worker max-turn boundary не є lifetime budget усього outcome, а його stream-silence timeout не загальний turn cap. Ми використовуємо ідею малого runner, але не додаємо issue tracker, cloud queue чи всі workflow semantics Symphony. Перезапуск worker не повинен reset наші run budgets.

## 10. MVP проти майбутньої effect safety

**MVP, потрібне для відсутності manual Continue:** stable protocol adapter; one active lock/turn; durable admitted task state й independent verifier; terminal response gate; same-thread continuation; persistent finite budgets; input handling; lost-ACK/process-death reconciliation; narrow sandbox. Foreground uptime достатній лише для живого process.

**Для crash-surviving local autonomy:** встановлений LaunchAgent/еквівалент, orphan quiescence, persistence/restore tests і wake/reconnect behavior.

**Майбутнє, не prerequisite цього MVP:** non-bypassable general effect gateway; authenticated revocation/fencing across workers; idempotent publication/payment/deploy adapters; multiple hosts. Без них не заявляти safety для довільних shell effects чи production. Не імпортувати повний greenfield kernel, щоб прибрати одну кнопку Continue.
