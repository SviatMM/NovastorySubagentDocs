# Implementation plan: прибрати manual Continue без перебудови AI OS

**Статус:** запропонований порядок реалізації та acceptance contract. Цей research не встановлює controller, не запускає Codex на Mac, не змінює private canonical repositories і не доводить причину коротких turns. Source facts та їхні межі наведені в [RUNTIME_DIAGNOSIS](RUNTIME_DIAGNOSIS.md).

## 1. Ціль першого релізу

Один явно admitted local run у керованому workspace виконує finite safe task queue через кілька provider turns. Після normal turn completion controller перевіряє актуальний durable state, сам виконує bounded verification/repair planning і запускає наступну eligible дію. Owner не надсилає Continue. Один verified final або чесний park/справжній запит рішення завершує run.

Це не гарантія розв'язання будь-якого software завдання, безмежної пам'яті, нескінченного uptime чи безпечності довільних external effects. Scope першого релізу: ізольовані reversible local edits/tests із перевіреною sandbox boundary. Required push/deploy/live proof не можна вилучати з реального contract тільки тому, що MVP їх ще не виконує.

**Критична розвилка:** діагноз пояснює, чому закінчується turn; supervisor усуває залежність від людського повідомлення після дозволеного завершення. Можна довести ефективність continuation і водночас залишити причину native-client хвилинного cutoff невстановленою. Ці результати фіксуються окремо.

## 2. Послідовність реалізації

### S0. Зафіксувати фактичний runtime та ізолювати probe

Зібрати installed app/CLI build, executable digest, generated App Server schema, model/provider/effort, config та instruction manifest, native goal/hook state, effective sandbox, authentication mode і relevant timeouts. Зберегти приватні bytes тільки локально; у public report лише безпечні locators/metadata. Не брати версію зі старої розмови за поточну.

Створити disposable fixture і controller state area поза worker write boundary. Не використовувати AI OS або NovaStory як test sandbox. Перевірити, що worker не може змінити controller config, budgets, journal і protected acceptance checks. Заборонити production credentials, external writes та невідомі MCP surfaces.

**Exit evidence:** runtime/config/schema receipts; fixture baseline digest; sandbox negative probe; конкретні допустимі command/check adapters. Якщо ізоляція не підтверджена, integration не admit. Читання source або `codex --version` не є E2E capability proof.

### S1. Побудувати passive recorder та deterministic classifier

Реалізувати JSONL parser і concurrent stdout/stderr drains, RPC correlation, process-generation tracking, monotonic/wall timing, explicit watchdog events, structured error mapping та usage coverage. Event envelopes і normalized observations відповідають [TRACE_SCHEMA](TRACE_SCHEMA.md).

Recorder не повинен змінювати natural completion policy досліджуваного baseline. Tool failure, recovered retry, native turn terminal і client process exit окремі. Не додавати новий 60-second deadline під час дослідження невідомого cutoff. Synthetic watchdog tests мають інший cohort tag.

**Exit evidence:** executable replay fixtures із virtual clock для всіх terminal categories, duplicate/late/unknown events, lost ACK і incomplete trace. Непідтверджений model time-limit claim не може створити HOST_TURN_TIMEOUT або EXECUTION_WINDOW_LIMIT. Protocol parse failure повертає structured diagnostic outcome, не process success із порожнім результатом.

### S2. Виміряти natural termination

Виконати pre-registered 12-trial order A,B,C,D,E,F,F,E,D,C,B,A з [EXPERIMENT_PLAN](EXPERIMENT_PLAN.md). Базова серія має зберегти всі turns, rejected starts і unknown observations. За наявності доступу до обох surfaces зробити окремі matched native-client та App Server cohorts. Якщо native transport не спостережуваний, не заповнювати прогалини headless результатами.

**Exit evidence:** ledger із щонайменше 12 послідовними trials у completed cohort, exact normalized terminal mechanism/evidence або explicit UNKNOWN на кожному, immutable prompt/fixture/runtime basis і independently verified remaining frontier. Незавершений через quota batch має status INCOMPLETE і фактичний count, а не фальшивий PASS.

**Рішення після вимірювання:** налаштувати лише підтверджений timer/source; виправляти loaded prompt wording як окремий фактор; не збільшувати всі timeouts навмання. Typed usage або policy restrictions не обходяться автоматичним retry, model substitution чи account switching.

### S3. Foreground minimum supervisor: stop manual Continue

Додати до recorder один controller-owned journal, process-held exclusive lock, pinned contract/task manifest, незалежний verifier, gate та continuation outbox. Протокол і lifecycle — [SUPERVISOR_DESIGN](SUPERVISOR_DESIGN.md). Створити persistent thread, виконувати рівно один active turn і запускати наступний `turn/start` на тому самому thread тільки після terminal/reconciliation і durable gate decision.

Gate обов'язковий у реальному output path: worker final messages лишаються internal. Він не залежить від бажання моделі запустити validator. Human-readable STATUS генерується як projection. Task proposals та worker claims не можуть переписати required obligations, authority, budgets або accepted evidence.

Додати persistent finite ceilings з SUPERVISOR_DESIGN, authenticated cancel, bounded input handling, usage waits і resource parks. Після вичерпання budget залишити той самий run/requirement ID та exact resume condition, а не почати новий run зі свіжими counters. Lost ACK спочатку reconcile; raw retry `turn/start` не є безпечним default.

**Exit evidence:** live useful queue із мінімум 10 accepted послідовних turns/task units, одна thread lineage, нуль Owner Continue, жодних двох active turns, актуальний gate receipt перед кожним continuation, один verified final. Додатково whole-backlog case із natural early completion повинен продовжитись за реального eligible frontier. Artificial one-unit scheduling доказує scheduler lifecycle, не причину natural stopping.

**Межа релізу:** foreground process забезпечує continuation тільки поки він справді працює. Цього достатньо, щоб прибрати Continue у живому run, але не для заяви про crash-surviving background service.

### S4. Crash-surviving single-Mac operation

Додати recovery entrypoint, перевірку DB/schema, old-child quiescence, process identity/ownership, durable session-store continuity, `thread/resume`, unknown-dispatch reconciliation, evidence invalidation і retry budgets. Після cancel/revocation автоматичне відновлення не дозволене. Невідомий irreversible effect паркується, а не повторюється.

Після окремого дозволу встановити локальний LaunchAgent або інший реальний process supervisor. Startup/restart викликає ту саму recovery path і отримує той самий lock. Не покладатися на mere lock filename чи PID без start-generation identity. Невідомий orphan writer має бути quiesced або takeover відхилено.

Перевірити local filesystem durability policy, включно з потрібною Mac-specific sync behaviour для заявленого failure domain. SQLite transactions не є атомарними з provider RPC. Disk-full, busy timeout, corruption і partial journal failure мають зупиняти dispatch, а не обходити persistence. Power-loss guarantee не випливає лише з назви WAL.

**Exit evidence:** контрольовані App Server/controller crashes у різних dispatch windows; budget continuity; cancel persistence; duplicate-start exclusion; sleep/wake/reconnect і restart tests; документовані умови, за яких agent реально стартує. Mac без живого supervisor, живлення або доступного середовища не виконує обіцяну роботу.

### S5. Окремі consequential-effect adapters, тільки за потреби

Після достатніх локальних доказів розширювати рівно потрібний ефект: наприклад, narrow Git publication на точний repo/ref із scoped staging, current authority, normal non-force update, remote read-back та uncertain-outcome reconciliation. Кожен новий effect має intent/receipt/operation identity і negative tests.

General effect gateway, multiple workers, revocation/fencing across processes, cloud queues, payments і deployments не є prerequisite S3. Не переносити весь Agent02 greenfield kernel, щоб вирішити turn continuation. Але й не продавати S3 sandbox як доведену безпечність production mutation. Повноваження на implementation або consequential effects не надані самим цим research.

## 3. Optional native pilot, не другий controller

Перед або поряд із плануванням S3 можна окремо виміряти supported `/goal` або synchronous Stop hook. Поточні official possibilities описані в RUNTIME_DIAGNOSIS O3/O4; підтримку конкретного binary перевіряє S0. У кожному run рівно один `continuation_driver`: native_goal, stop_hook або external_controller.

Native pilot може прибрати manual Continue з меншою кількістю коду. Приймати його замість S3 можна тільки якщо його **фактична** response boundary, durable-state check, finite budgets та stop/recovery behaviour задовольняють заявлені вимоги. Відсутність реалізованого `suppressOutput`, hook coverage або failure behaviour не компенсується prompt-обіцянкою. Pilot без цього статусу позначається policy-assisted, а не enforced run gate.

## 4. Мінімальна структура implementation package

Це майбутня структура окремого local package, не файли, створені цим research:

```text
controller/
  cli.py                 admit, run, status, cancel, resume
  app_server.py          framing, RPCs, server requests, event correlation
  recorder.py            timers, process observations, redaction
  store.py               transactional journal, budgets, dispatch/outbox
  gate.py                current-state decisions and terminal output boundary
  verifier.py            protected acceptance/check adapters
  recovery.py            quiescence, thread/session/worktree reconciliation
  policy.py              finite bounds and existing authority checks
  schemas/               generated contract for pinned installed runtime
  tests/                 synthetic replay, fixtures and local integration tests
```

Модулі можуть бути об'єднані; вимога до boundaries, не до кількості файлів. Python stdlib достатня для skeleton, але schema validation, logging і OS process identity реалізувати явно. Не давати raw model tool arguments виконуватися як trusted control-plane commands.

Run manifest задає точний allowed workspace, stable requirement/task IDs, dependencies, protected check definitions, authority revision, selected model/runtime, all budget ceilings, stop/resume policy і continuation driver. Для першого fixture manifest створює оператор/harness; для real products існуючий admission може бути input через narrow validated adapter. Зміна AI OS files для цього research не потрібна.

## 5. Acceptance matrix

| Test | Очікувана перевірка |
|---|---|
| Completed + eligible TODO | Gate зберігає CONTINUE_INTERNAL і сам dispatches next turn, без progress-only terminal response. |
| Completed + усі verified predicates | Один FINAL_COMPLETE із evidence basis, no extra turn. |
| Empty або worker-edited applicability | Не приймати success без immutable admitted obligations. |
| Stale test/review after edit | Invalidate relevant receipt; створити verification/repair work. |
| One blocked dependency + independent task | Виконати незалежну authorized задачу, не park весь goal автоматично. |
| Model каже execution-window limit | Класифікація за protocol/host evidence; речення не створює timeout receipt. |
| Timer fire/cancel/completed race | Retain event order і ambiguity; no fabricated USER_CANCEL або timeout attribution. |
| `error.willRetry=true` | Не починати competing turn; спостерігати actual terminal/recovery. |
| Lost `turn/start` ACK | Persist sent_unknown, reconcile до будь-якого повтору. |
| Controller crash залишив child | Quiesce/verify ownership; no second writer або unverified takeover. |
| Authenticated Owner cancel | Persist cancellation, interrupt і reconcile; no continuation після restart. |
| Real decision/login request | Matching bounded wait/notice; no invented answer or permission. |
| Signature/repair/review/alternative budget exhausted | Honest park із counters, evidence та exact resume condition; no false OWNER_BLOCKER. |
| Usage exhausted/unsupported | Correct typed wait/unknown; no quota bypass, fabricated zero або infinite polling. |
| Storage busy/full/corrupt | Stop new dispatch, preserve diagnostics where possible; no persistence bypass. |
| Uncertain external effect | PARKED_UNKNOWN_EFFECT, authoritative reconciliation перед replay. |
| New schema/build | Capability revalidation або PARKED_RUNTIME; no silent field guessing. |
| Ten useful automatic turns | Same run/thread lineage, zero Owner Continue, one active turn, persistent accounting і single final. |

Повний controller ще не перевіряється цією таблицею: це acceptance specification. Component replay, local fixture E2E і робота в реальному product environment повинні мати різні evidence labels.

## 6. Owner decisions, що справді потрібні для activation

Жодне рішення нижче не блокує завершення цього source-only research. Перед реальною installation потрібні лише невідомі material boundaries:

- **Deployment/authority:** схвалити controller-owned execution/output surface, початкові allowed workspace/action scope та заборонені effects. Існуючий grant можна перевикористати, якщо він уже точно охоплює це; не питати заново про відомі approved details.
- **Resources:** finite wall/usage/retry envelope, handling of unobservable usage, дозволені runtime/model substitutions або їх заборона. Числа pilot policy не є автоматичним дозволом на витрати.
- **Privacy/uptime:** local journal location/access/retention і допустимий sensitive capture; окремо дозвіл на login/background startup installation. Credential або OS consent, якщо реально знадобляться, є external-permission step, не питання дизайну.

Операційні повідомлення мають називати справжню дію: наприклад, відновити доступність сховища і пройти integrity check; отримати matching answer для конкретного pending request; дочекатися observed rate reset; затвердити нову resource ceiling для того самого run. Порожнє “Owner, continue” не є resume condition.

## 7. Зв'язок із шістьма завданнями research

| Primary task | Основний artifact | Практичний доказ при реалізації |
|---|---|---|
| Diagnostic model | TRACE_SCHEMA + RUNTIME_DIAGNOSIS | Classifier replay та natural terminal ledger із provenance. |
| Response gate | SUPERVISOR_DESIGN §§2,4 | Early-final bypass і stale/empty-state negative tests на actual output channel. |
| Automatic continuation | SUPERVISOR_DESIGN §§3,6,7,9 | Same-thread turns, input/cancel, lost ACK, process recovery і встановлений supervisor для crash uptime. |
| Debug experiment | EXPERIMENT_PLAN | 12 controlled trials/cohort + окремий 10-turn continuation run без вигаданих даних. |
| Prompt audit | RUNTIME_DIAGNOSIS §5 | Loaded instruction manifest; matched D/E/F; external-evidence-only limit assignment. |
| Minimal controller | SUPERVISOR_DESIGN + цей план | S3 foreground MVP; S4 recovery; S5 separate effect expansion. |

## 8. Чесний release/reporting contract

Result цього research: diagnosis framework, protocol-grounded supervisor design, experiment та implementation plans. Live Codex turns у ньому не виконані. Root cause лишається unproven, доки не зібрано конкретні native-client runtime receipts.

Implementation звіт має окремо показати: що documented; що locally invocable; які synthetic checks виконані; які E2E trials виконані; скільки real continuations пройшло; які гарантії не перевірено; який exact resume condition діє для parked runs. Prompt change, green simulator output чи commit документів не дозволяють поставити E2E_VERIFIED.
