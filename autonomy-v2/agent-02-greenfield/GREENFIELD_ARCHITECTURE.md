# Greenfield: Authority-Bounded Durable Execution Kernel

**Task:** `autonomy-v2-agent-02-greenfield`  
**Design status:** proposed architecture, not an implemented runtime.  
**Language:** українська; machine identifiers and pseudocode are English.  
**Method:** цей документ сформовано й зафіксовано до першого читання canonical AI OS у цьому дослідженні. Вихідні дані — outcome та hard requirements із завдання, не запропонована Autonomy V2. SHA-256 початкового документа і source-reading provenance наведено в `COMPARISON.md` та `RESULT.json`.

## 1. First principles: що саме треба автоматизувати

Owner делегує досягнення результату, а не перекидання повідомлень між чатами. Сесія моделі є тимчасовим обчислювачем; її пам'ять, обіцянка або остання відповідь не є станом проєкту. Автономність — це здатність переходити між перевіреними станами, ремонтувати невдачі та відновлюватися без розширення делегованої влади. Вона не вимірюється тривалістю одного запуску.

Модель може помилитися в плані, оцінці готовності, трактуванні дозволу та описі виконаного. Тому модель пропонує дії, а мале детерміноване ядро допускає їх, виконує контрольовані переходи та перевіряє факти. Пропонована назва: **Authority-Bounded Durable Execution Kernel (ABDEK)**.

Основні об'єкти:

- **Outcome contract:** що має бути істинним після роботи, на яких поверхнях, із яким доказом і якою доставкою.
- **Authority grant:** що дозволено робити заради outcome; це окремий об'єкт, а не висновок із бажаного результату.
- **Predicate graph:** залежності між потрібними фактами, альтернативними способами їх отримати та діями.
- **Durable run:** журнал переходів, контрольні точки, receipts, докази, блокери й рішення, незалежні від моделі та вікна.
- **Execution kernel:** admission, scheduler, policy checks, effect gateway, evidence verifier та recovery coordinator.

### 1.1 Інваріанти

| ID | Інваріант |
|---|---|
| I1 | Кожна дія в момент допуску й перед незворотною відправкою належить чинному grant. Намір Owner не дорівнює універсальному дозволу. |
| I2 | Планувальник, worker, reviewer, repository text і зовнішня відповідь не можуть видавати чи розширювати grant. |
| I3 | Невідомий результат side effect не є невдачею, яку можна сліпо повторити. |
| I4 | `SATISFIED` для predicate потребує допустимого evidence, прив'язаного до потрібного artifact/environment revision. |
| I5 | Після втрати контексту система відновлює той самий run, а не створює нове виконання того самого ефекту. |
| I6 | Локально заблокований dependency не зупиняє незалежну авторизовану роботу. |
| I7 | Reviewer не може прийняти власний implementation; інтеграційні зміни перевіряються на кінцевому дереві. |
| I8 | Push не випливає з commit, deploy не випливає з push, новий remote не випливає з дозволу на старий remote. |
| I9 | Фінальний success є висновком verifier, а не текстом worker або фактом витраченого бюджету. |
| I10 | Вичерпання attempts/context/budget не надає нових повноважень і не перетворює partial на success. |
| I11 | Людині надсилають лише справді потрібне рішення/фізичну дію або один підсумок; технічні handoff проходять машинними каналами. |
| I12 | Відкликання влади забороняє нові дії; вже відправлені ефекти окремо звіряються, а не стираються з історії. |

### 1.2 Межі гарантій

Ця архітектура не обіцяє безумовного завершення кожного задуму. Liveness умовна: доступні достатні інструменти, ресурси, допустимий шлях і зовнішні відповіді. За недоступності середовища run може бути чесно припаркований. Це не вимога повернути Owner роль диспетчера.

Гарантія відсутності дубльованих бізнес-ефектів потребує або надійного provider idempotency contract, або атомарної транзакції/унікальності в системі призначення. Для довільного opaque API, де результат неможливо перевірити, немає одночасної безумовної гарантії «обов'язково завершити» і «ніколи не повторити ефект» після crash. Safe default: не допускати такі незворотні операції без додаткового протоколу; для вже неоднозначного результату — quarantine і звірка, не повтор. Позначення `at-most-one dispatch` не продається як `exactly-once business effect`.

## 2. Компоненти та trust boundary

```text
Owner / authenticated decision surface
              |
     Outcome + Authority registry
              |
       Durable execution kernel
       | admission / policy monitor
       | transactional run journal + scheduler
       | artifact/evidence verifier
       | effect gateway + reconciler
       | decision/permission inbox + outbox
       |
       +--> isolated planners/workers (proposals, patch artifacts)
       +--> independent reviewers (findings, attestations)
       +--> test/build/UI executors (measured evidence)
       +--> Git integration executor (exact repo/ref operations)
       +--> external adapters (capability-limited operations)
```

Registry та policy monitor — не редагований worker-ом Markdown. Захищені процесом/сховищем права, revision checks і аудит відокремлюють їх від робочого репозиторію. Repository instructions, issues, logs, web pages і tool output — недовірені дані. Вони можуть підказувати спосіб роботи, але не створюють credentials, budget чи approval. Вбудоване в README «Owner approved» не є authority event.

Credentials не передаються моделі та worker filesystem. Effect gateway використовує короткоживучі scoped handles; raw secrets — у дозволеному secret store. Sandbox обмежує filesystem, network egress, command families, resource consumption і доступ до production. Тести чужого коду також запускаються в sandbox без production credentials. Одна текстова інструкція в системному prompt не є enforcement.

Мінімальна реалізація: один kernel-сервіс, transactional store із compare-and-swap/унікальними ключами, content-addressed artifact store, workers у disposable worktrees, Git та external adapters. SQLite із надійною локальною дисковою семантикою може бути варіантом для одного host; shared/HA виконання вимагає відповідного shared transactional store. Не пропонується зберігати конкурентний журнал як довільно перезаписуваний JSON на мережевій теці.

## 3. Authority model

### 3.1 Outcome окремо від влади

Owner затверджує outcome revision та grant revision через автентифікований канал. Kernel зберігає issuer identity, decision ID, issue/revoke events і непідробний зв'язок із run. Криптографічний підпис є можливим способом реалізації; контроль доступу й захищений журнал є обов'язковими незалежно від способу.

Grant містить:

```yaml
grant_id: grant-example
revision: 1
subject: run-example
issuer: authenticated-owner
outcome_revision: 1
repositories:
  - identity: approved-repository-id
    read_refs: [approved-base]
    write_paths: [approved-product-area]
    local_worktrees: disposable-only
operations:
  read: true
  edit: true
  tests_builds: sandbox-only
  local_commit: true
  push:
    enabled: true
    remote_identity: approved-remote-id
    ref: refs/heads/approved-work-branch
    mode: fast-forward-only
    create_ref: false
    required_checks: [artifact-verification, independent-review]
    permitted_downstream_effects: explicitly-enumerated
  deploy: false
boundaries:
  product_intent: approved-outcome
  architecture: approved-decision-set
  security: approved-security-policy
  data_placement: approved-locations-and-classifications
  scope: approved-deliverables
  destructive_production: deny
  irreversible_external: deny-unless-exactly-authorized
limits:
  spend: approved-amount-and-currency
  usage: approved-token-and-compute-budget
  concurrency: approved-worker-count
  repair_policy: approved-policy-id
  external_retries: adapter-specific
validity:
  expires_at: explicit-or-no-expiry
  revoked: false
delegation: subsets-only
```

Це приклад структури, не реальний дозвіл і не вимога дозволяти push у кожному run. До запуску неоднозначний дозвіл звужується до безпечного intersection, а не розширюється за припущенням.

Кожна дія має `resource`, `operation`, `expected_effect`, `data_classification`, `cost_reservation`, `artifact_revision`, `preconditions`. Policy monitor обчислює:

```text
allowed(action) = current_grant_contains(action)
               AND outcome_revision_matches
               AND resource_identity_matches
               AND all_safety_preconditions_true
               AND budget_reservation_committed
               AND lease_epoch_current
```

Фактична capability не є authority. Наявність shell, токена або кнопки Deploy не дозволяє їх використати. Дозвіл працювати із файлом не дозволяє обійти межу через symlink, alternate worktree, script side effect чи зміну CI. Gateway перевіряє canonical resource identity, real path, ref, executable policy і egress; path allowlist — лише частина захисту.

### 3.2 Хто вирішує

Kernel самостійно обирає implementation details, локальні виправлення, порядок незалежних задач, дозволені альтернативи, дозволені model/runtime substitutions. Owner зберігає рішення про продукт, security, architecture boundary, privacy/data placement, scope, spending, credentials, destructive production та irreversible effects.

«Потрібна інша бібліотека» не автоматично є Owner decision: сумісна заміна всередині затверджених boundary може бути локальним вибором. «Перенести персональні дані в інший cloud», «вимкнути обов'язкову перевірку» або «змінити модель довіри» — матеріальні зміни. Відоме рішення в decision registry перевикористовується лише за збігу умов; старий дозвіл на інший outcome не переноситься.

Grant amendment — нова Owner revision; ніколи не текст у worker status. Спочатку забороняються нові side effects під старою revision, потім визначається доля in-flight actions. Раніше зібраний evidence переоцінюється лише там, де зміна робить його непридатним. Автоматична rollback/compensation теж потребує окремого дозволу: «скасувати» не означає «можна виконати будь-яку зворотну дію».

## 4. Durable execution state

### 4.1 Durable record

Журнал зберігає append-only events та transactional materialized state. Snapshot прискорює restart, але може бути перебудований із журналу. `STATUS.md`, чат і dashboard — projections. Для різних фактів різні джерела істини: authority registry для дозволів; authenticated remote ref для факту push; object store для bytes; verifier records для приймання.

```yaml
schema_version: 1
run_id: run-example
sequence: 37
phase: ACTIVE
outcome: {id: outcome-example, revision: 1, digest: sha256-of-contract}
authority: {grant_id: grant-example, revision: 1, digest: sha256-of-grant}
controller: {epoch: 4, lease_owner: kernel-instance, lease_until: timestamp}
source:
  repository_id: approved-repository-id
  base_commit: full-commit-sha
  observed_remote_ref: full-ref
execution_lock:
  runtime_version: exact-version
  runtime_digest: sha256-of-runtime
  adapter_versions: {git: exact-version, external: exact-version}
  model_policy: approved-model-policy
  active_model_identity: provider-model-version-or-unresolved-alias
  evidence_schema: exact-version
  policy_digest: sha256-of-policy
plan:
  revision: 3
  graph_digest: sha256-of-predicate-graph
  predicates: [predicate-record-references]
  candidates: [action-record-references]
workspaces:
  - id: worktree-example
    branch: approved-work-branch
    base_commit: full-commit-sha
    fence_epoch: 4
    patch_digest: sha256-of-patch
budgets: {reserved: ledger-reference, consumed: ledger-reference}
actions: [action-record-references]
effects: [effect-journal-references]
reviews: [review-record-references]
blockers: [blocker-record-references]
decisions: [decision-record-references]
permission_requests: [permission-record-references]
evidence: [content-addressed-manifest-references]
continuation: {cursor: durable-cursor, trigger: durable-trigger, next_probe: timestamp}
publication: {required: true, receipt: null}
final_receipt: null
```

Кожен event: `event_id`, `run_id`, `sequence`, `type`, `actor`, `controller_epoch`, `observed_at`, `causation_id`, `payload_digest`, `schema_version`. Унікальність `event_id` та compare-and-swap на sequence запобігають подвійному застосуванню. Час не використовується як єдиний порядок подій. Hash chain допомагає виявляти пошкодження; вона не заміняє ACL, backups і захист від привілейованого переписування.

### 4.2 Транзакційні межі

В одній локальній транзакції: перевірка актуального epoch/grant, резерв бюджету, запис action/effect intent і outbox event. Зовнішній network call не стає частиною цієї транзакції магічно. Response/receipt потрапляє в inbox і журнал через deduplicated commit. Втрата відповіді залишає `UNCERTAIN`, а не повертає action у `READY`.

Checkpoint містить graph revision, незавершені effect IDs, dirty patch bytes/digest, worktree identities, test/review bindings, scheduled triggers і unresolved requests. Durable означає переживає заявлену failure domain: restart процесу, restart host або втрату host — це різні гарантії. Розташування та backup evidence затверджуються відповідно до privacy policy; відсутність дозволеного durable storage блокує admission відповідної гарантії, а не породжує прихований cloud upload.

## 5. Work admission і repository grounding

Admission — перевірка можливості безпечного старту, а не гарантія, що жодної нової проблеми не буде.

1. Записати outcome: observable result, in/out of scope, required artifacts, acceptance predicates, required external proof, delivery/ref requirements. «Зроблено локально» та «опубліковано» — різні вимоги.
2. Перевірити authenticated grant, resource identities, budgets, revocation/expiry та межі даних. Не просити Owner повторити вже наданий достатній дозвіл.
3. Ground repository на exact commit: структура, активні instructions, build/test entrypoints, dependency lockfiles, ownership, стан робочого дерева, remotes, CI/downstream side effects. Інструкції читаються як input; конфлікт із grant не виграє.
4. Зафіксувати capability observations: доступні runtime, модель, інструменти, sandbox, мережа, credentials handles, reviewer та persistent store. Observation має identity, version, probe, timestamp і validity conditions.
5. Побудувати candidate predicate graph та verification plan. Виявити owner decisions і фізичні prerequisites, але допустити незалежні зрізи роботи.
6. Записати admission record: contract/grant/source/lock digests, permitted actions, known unknowns, readiness per predicate, recovery class для кожного external adapter, фінальні критерії.

Неповний capability inventory не означає заборону всієї локальної роботи. Наприклад, відсутність login до deployment service не блокує sandbox implementation, якщо це незалежно й дозволено. Але план не може оголосити required live proof необов'язковим, аби пройти admission.

Existing dirty worktree не присвоюється агенту автоматично. Admission або використовує чистий isolated worktree, або явно відокремлює Owner changes із provenance. Production credentials, локальні персональні шляхи та приватні логи не переносяться в public artifacts.

## 6. Planning model

План — versioned AND/OR graph. AND описує необхідні prerequisites; OR — допустимі implementation strategies для того самого predicate. Task/action — спосіб змінити світ, predicate — факт, який треба довести. Наприклад, «компонент поводиться правильно» може мати дві реалізації, але не дві довільні різні дефініції правильності.

```text
outcome
  AND implementation accepted at tree T
        AND requirements verified
        AND build/tests pass
        AND independent review accepted
  AND delivery satisfied
        AND local commit identifies verified tree T
        AND authorized remote ref confirms commit C   # only if required
  AND required external predicates verified           # only as contracted
  AND no unresolved safety-critical uncertainty
```

Planner додає гіпотези, action candidates, dependency edges, estimated cost/risk і alternative rationale. Kernel версіонує graph і забороняє послаблювати contract, grant або verification policy через replan. Зміни acceptance належать contract revision, а не graph cleanup.

Scheduler працює по готовому frontier: обирає allowed actions із максимальним корисним прогресом за поточними limits, з fairness для незалежних напрямів. Блокування node поширюється лише на transitive dependants, не на весь run. Старі докази інвалідуються за dependency impact; невідомий impact означає ширшу перевірку, не необґрунтоване повторне використання.

Корисний прогрес вимірюється переходами predicates, усуненням підтверджених findings, зменшенням невизначеності та verified integration — не кількістю повідомлень, комітів або годин. Метрики автономності: частка внутрішніх failures, розв'язаних без Owner; частка recovery без дубля; false-success count; unauthorized-action count; частка несправжніх Owner escalations. Для двох останніх безпекових класів ціль — нуль, не «краще середнє».

## 7. State machine

Run phase, node state і wait reason відокремлені. Інакше один `BLOCKED` приховує можливу роботу.

```text
Run phases:
  CREATED -> ADMITTING -> ACTIVE -> VERIFYING -> DELIVERING -> FINALIZING
                           ^           |            |            |
                           +-----------+------------+------------+  repair/replan
  any nonterminal -> PARKED -> RECONCILING -> ACTIVE/VERIFYING/DELIVERING
  any nonterminal after restart/drift -> RECONCILING
  any nonterminal -> CANCELLING -> CANCELLED
  ADMITTING -> REJECTED                    # no admissible contract/grant
  FINALIZING -> SUCCEEDED                  # all completion predicates true
  nonterminal -> CLOSED_INCOMPLETE         # explicit authorized closure only

Predicate states:
  UNPROVEN | SATISFIED | REFUTED | STALE | WAITING | UNCERTAIN

Action states:
  PROPOSED -> ADMITTED -> READY -> LEASED -> INTENT_RECORDED
    -> RUNNING/DISPATCHED -> OBSERVED -> VERIFIED
  failure -> FAILED -> new action/retry revision
  lost result -> UNCERTAIN -> RECONCILING -> VERIFIED/FAILED/QUARANTINED
  superseded before dispatch -> CANCELLED

Wait reasons (sets, per node):
  RETRY_TIMER | CAPABILITY | EXTERNAL_PERMISSION | OWNER_DECISION
  REMOTE_RECONCILIATION | RESOURCE_BUDGET | DEPENDENCY | SAFETY_QUARANTINE
```

`PARKED` не є фінальним результатом чи проханням перенести prompt. Це durable run із wake condition, probe policy та continuation cursor. Kernel може припаркувати affected node, лишивши run `ACTIVE`. Global `PARKED` допустимий лише коли нема ready allowed action і нема корисного локального reconciliation. Якщо бюджету бракує для навіть checkpoint/recovery, це admission defect: резерв на безпечне завершення транзакцій і відновлення виділяється наперед.

`SUCCEEDED`, `CANCELLED`, `REJECTED`, `CLOSED_INCOMPLETE` — різні terminal outcomes. Лише перший означає виконаний contract. Cancellation не гарантує відсутності in-flight effects: їх unresolved IDs залишаються видимими, а обов'язковий reconciliation може продовжуватися як safety task під явно виділеною read-only authority.

## 8. Blocker, retry та replan model

Blocker — typed record, а не текст «не можу продовжувати»:

```yaml
blocker_id: blocker-example
affected_predicates: [predicate-id]
blocked_actions: [action-id]
cause: CAPABILITY
observations: [evidence-id]
first_seen_sequence: 31
retryability: conditional
next_probe: timestamp-or-event
alternatives:
  considered: [strategy-a, strategy-b, strategy-c]
  admissible_now: [strategy-b, strategy-c]
  rejected: [{strategy: strategy-a, reason: observed-failure}]
required_resolution: machine-capability-or-authenticated-decision
request_id: null
owner_materiality: null
expiry_or_revalidation: explicit-condition
```

Немає абсолютного поля «всі можливі способи вичерпано» без обмеження множини. Record вказує, які реалістичні кандидати розглянуто, чому інші недопустимі, які assumptions ще неперевірені та чи залишився budget на пошук. Неможливо довести відсутність усіх невідомих алгоритмів простим лічильником.

### 8.1 Policy ремонту

До кожного failure додаються fingerprint (`predicate`, artifact, environment, normalized finding), observed evidence, causal hypothesis, repair delta та очікуваний discriminating test. Retry і replan відрізняються: retry повторює ту саму семантичну дію за дозволеної transient причини; replan змінює спосіб досягнення незмінного predicate.

Один failed test: відтворити, класифікувати, виправити або виконати обґрунтований transient rerun. Зелений rerun не стирає red run: flaky-result policy визначає, чи достатньо цього evidence.

Дві невдалі однакові repair hypotheses: заборонити третє сліпе повторення того самого delta; перейти до root-cause investigation, іншого strategy або diagnostic worker. Це рекомендоване значення policy, не закон природи. Лічильник ремонту не дозволяє повертати роботу Owner автоматично.

Три review rounds із різними findings: оцінити resolved/open findings, регресії, стабільність artifact і витрати. Продовжити remediation за наявності прогресу й authority. При oscillation — diagnostic replan, альтернативний independent reviewer або environment check за policy. При вичерпанні budget — припаркувати й запросити лише матеріальне рішення про додатковий budget/зміну scope, а не «напиши новий prompt».

Для transient network/capability failure: bounded exponential backoff із jitter, circuit breaker, адаптерний retry budget, persisted next probe. Параметри беруться з admission policy. Repeated hidden calls не обходять spending cap: резервуються й actual usage, й невизначені charges. Відсутній провайдерний usage receipt враховується консервативно до звірки.

## 9. Independent review і worker model

Planner, implementer, test executor, reviewer та integrator мають окремі ролі й capability subsets. Reviewer отримує outcome/acceptance, immutable artifact/tree, релевантні dependency facts та evidence; не отримує авторитету від самозвіту implementer. Чиста review session з іншою identity/role separation — мінімум. Інша назва chat або та сама continuation implementer не є незалежністю. Інша модель знижує деякі correlated risks, але сама по собі теж не доводить незалежності; вимоги до model/provider diversity задає review policy.

Reviewer має read-only artifact доступ і власний execution sandbox. Якщо reviewer сам виправляє код, він стає contributor до нового artifact, і final acceptance має дати новий незалежний reviewer. Review attestation містить reviewer identity, role isolation, model/runtime identity, exact tree, contract/policy revisions, findings, evidence refs і verdict. Worker не може виставити `SUCCEEDED`.

Кожен worker має disposable worktree, base SHA, branch identity, patch artifact та lease з fencing epoch. Закінчення lease не робить старий worker безпечним саме по собі: gateway відхиляє його нові calls за epoch, а приймання patch перевіряє base/epoch. Уже відправлені зовнішні дії звіряються окремо.

Overlapping file writes не дозволяються в одному shared mutable worktree. Workers можуть паралельно створювати ізольовані patches, але один integrator серіалізує інтеграцію через compare-and-swap base. Conflict не передається Owner за замовчуванням: rebase/merge у допустимій області, вирішення, повторний build/test/review інтегрованого tree. Суміжні semantic dependencies перевіряються навіть без текстового conflict. Несподівані чужі changes не перезаписуються й не cherry-pick-аються без provenance.

## 10. External permission protocol

Login, 2FA, physical consent і device approval — typed human action, не автоматично нове product decision. Запит містить `request_id`, потрібний predicate, service identity, account/tenant scope, exact human action, причину, дозволені наслідки, expiry, secure continuation channel та machine-verifiable success probe. Secret bytes у request/response заборонені.

```text
NEEDED -> REQUESTED -> HUMAN_ACTION_PENDING
       -> CAPABILITY_OBSERVED -> REVALIDATED -> RESOLVED
                         or -> EXPIRED/CANCELLED/DENIED
```

Людина входить через автентичний UI потрібного сервісу, вводить 2FA там, а не в чат/репозиторій. Gateway отримує тільки дозволений handle і перевіряє tenant, scopes, audience, expiry та фактичну мінімальну capability. «Я ввійшов» без probe не відкриває predicate. OAuth grant із зайвими scopes не стає новим authority grant.

Запит дедуплікується за service/scope/predicate/request generation. Prompt для Owner не генерується на кожний poll або session restart. Таймаут не є consent; denial не обходиться через інший акаунт. Після успішного probe durable trigger продовжує той самий run. Незалежна робота триває до й після запиту. Оплата може одночасно потребувати spending decision і фізичного підтвердження; це два пов'язані records, а не універсальна кнопка «дозволяю все».

## 11. Owner escalation protocol

Owner decision request допустимий, коли конкретна необхідна зміна виходить за чинний intent/grant, або потрібна дія людини. Звичайний technical failure, недоступність конкретної implementation path, review FIX, session end чи mutable STATUS — не достатні причини.

Decision packet: current contract/grant revision, affected predicates, перевірені факти, точне рішення, варіанти, рекомендований варіант, trade-offs, потрібна authority delta, безпечна поведінка без відповіді, work already continuing, decision deadline лише за реальної зовнішньої умови. Не надсилати Owner сирий transcript як завдання самому синтезувати prompt.

Decision dedupe key: `run + boundary + decision_scope + precondition_digest`. Відповідь автентифікується, записується один раз та застосовується лише до matching request і revision. Якщо світ змінився, потрібна revalidation; запізнілий approval не «наздоганяє» довільно інший план. Відмова може виключити strategy, не обов'язково весь outcome. За відсутності відповіді заборонена гілка залишається blocked.

Runtime notification про PARKED може відображатися в dashboard без action request. Машинний supervisor сам відновлює worker/session. Несправність infrastructure не маскується під архітектурне питання Owner. Якщо supervisor відсутній, система прямо показує `PAUSED_NEEDS_EXECUTOR`, з durable continuation, і не стверджує, що продовжує працювати у фоні. Це deployment limitation, не business decision.

## 12. Evidence model та completion gate

Evidence record містить: predicate ID, producer identity/role, source tree або commit, exact command/probe specification, environment/runtime/model/adapter identities, input/output digests, exit code, started/finished observations, required checks, classification/redaction policy, artifact URI, verifier result та invalidation dependencies. Для UI — exact bundle/build identity, fixture/account/environment, screenshot/video/probe manifest без приватних даних; сам скриншот без artifact identity недостатній.

Content addressing доводить відповідність bytes, не правильність змісту. Перевіряється походження producer, trusted measurement path, schema, relevant coverage і binding. Компрометований executor потребує сильнішої isolation/attestation; hashes від нього не роблять брехню істинною. Missing/corrupt/stale evidence означає `UNPROVEN`/`STALE`, а не implied pass.

Зберігаються original failed attempts, repairs, review findings та superseded receipts. Public delivery — окремий sanitized manifest зі створеними автором висновками, дозволеними агрегатами й без секретів. Навіть hash приватного низькоентропійного значення може розкривати інформацію; не публікувати такі fingerprints. Storage ACL, encryption, retention, backups і data placement належать policy.

### 12.1 Completion predicates

Для contract revision `r` та final artifact `T`:

```text
SUCCESS(r,T) =
    outcome_requirements_verified(r,T)
  & required_builds_and_tests_valid(r,T)
  & independent_review_accepts(r,T,current_review_policy)
  & all_required_findings_resolved_or_explicitly_owner_accepted(r,T)
  & required_external_proof_valid(r,T,target_environment)
  & required_delivery_verified(r,T,exact_remote_and_ref)
  & no_unresolved_critical_or_outcome_relevant_effect_uncertainty
  & all_actions_within_valid_authority_at_their_dispatch
  & evidence_complete_durable_and_verifiable(r,T)
  & no_active_writer_or_unreconciled_artifact_mutation(T)
  & final_receipt_atomically_committed_once(run_id,r,T)
```

У реалізації final receipt записується після всіх інших перевірок в одному compare-and-swap переході `FINALIZING -> SUCCEEDED`; останній conjunct вище описує вже завершений стан, не циклічну передумову.

`required_*` походять тільки з contract/policy. Not applicable мусить бути обґрунтовано там, а не самостійно додано наприкінці. Owner risk acceptance не може скасувати незмінну platform/security boundary. Local implementation accepted може бути milestone, але при required live proof run не `SUCCEEDED`, поки proof нема. Contract може від початку бути implementation-only; це чесний інший outcome.

Зміна verified tree інвалідує відповідні tests/review. Evidence можна прив'язати до immutable tree до commit; після commit довести, що commit tree дорівнює verified tree, а policy для commit metadata теж виконана. Git success без tests/review не є completion. Deadline або exhaustion можуть призвести до `CLOSED_INCOMPLETE` лише за дозволеною closure policy; final report явно перелічує невиконані predicates.

## 13. Commit/push authorization semantics

Commit і push — різні action types. Grant окремо визначає local commit, tracked paths, branch/ref creation, remote identity, цільовий ref, fast-forward requirement, permitted downstream CI/effects та required checks. Push за наявності deployment webhook може мати зовнішні наслідки: admission не ігнорує їх і не трактує «не запускав deploy command» як доказ відсутності deploy.

Перед commit: зафіксувати exact index/tree digest, зміст staged files, parent SHA, allowed paths, attribution і hooks policy; заборонити unverified mutable hook changes. Kernel журналює commit operation key. Після crash він шукає вже створений commit у сховищі/ref за persisted intent і tree/parent/operation identity, а не створює дубль із новим timestamp. Durable record достатній для відновлення bytes; refs оновлюються атомарно за expected old SHA.

Перед push: перевірити current grant revision, remote repository identity, exact source commit, target ref, current authenticated remote SHA, ancestry/fast-forward, published artifact evidence та downstream effects. Не пушити `--all`, tags або інші worktrees. Force/force-with-lease, main update, PR, merge, release та deploy не є implicit fallback. Remote branch creation потребує явного дозволу.

Push intent зберігає `(repo_id, remote_id, ref, expected_old_sha, desired_new_sha, run_id, operation_id)`. Adapter має передати expected-old precondition у race-safe non-forced update, або завершити rejection; попередня read без guarded write недостатня. Зміна remote між read і write веде до reconciliation. Allowed rebase створює новий tree/commit і вимагає потрібної повторної перевірки; не перезаписувати невідомі remote changes.

Remote confirmation receipt містить authenticated ref observation і серверний operation evidence, якщо доступний. Exact equality із desired SHA достатня для contracted «branch points to C». Descendant із C у history достатній лише для іншого явно визначеного predicate «C опублікований у branch history» і не доводить, що поточний tip reviewed. Network failure залишає delivery pending/uncertain, але не скасовує вже валідний local commit.

Git object/ref idempotence не доводить exactly-once виконання довільного webhook. Downstream adapters повинні дедуплікувати business operation keys, або push не допускається для такого ефекту. Повтор API request за crash робиться тільки після reconciliation та за перевіреним retry contract.

## 14. Recovery, session/window та drift

### 14.1 Recovery protocol

1. Прочитати supported schema, перевірити journal integrity, отримати controller lease через atomic CAS і збільшити epoch. Старий процес fence-иться в gateway; його unfinished effects не забуваються.
2. Перевірити grant expiry/revocation, outcome revision та execution lock. Не поновлювати credentials або capabilities за пам'яттю моделі.
3. Відновити plan/budgets/requests/effects із журналу. Порівняти worktree bytes, local Git objects/refs і authenticated remote refs із recorded intents. Невідома розбіжність стає explicit reconciliation task.
4. Для кожного `INTENT_RECORDED`, `DISPATCHED`, `UNCERTAIN` перевірити adapter status за стабільним operation key. Confirmed effect записати як receipt; authoritative no-effect дозволяє retry лише за чинним contract; ambiguous лишити quarantined.
5. Відновити predicates із валідного evidence; інвалідувати stale bindings. Відбудувати STATUS/dashboard як projection, зберігши факт попередньої суперечності.
6. Розпочати ready allowed frontier або відновити durable wait. Повторний recovery застосовує ті самі dedupe keys і не породжує нові side effects, requests чи final receipts.

Суперечність `STATUS=done`, remote lacks C: перевірити історію/receipt і contract; не довіряти STATUS. Суперечність `STATUS=pending`, remote equals C: записати підтвердження push і продовжити final verification, не пушити повторно. Git не заміняє authority/evidence: наявність commit сама по собі не підтверджує tests або дозвіл.

### 14.2 External crash window

```text
LOCAL TX: persist operation_id + payload_digest + grant_rev + retry_contract
LOCAL TX: reserve dispatch generation / outbox
GATEWAY: recheck authority and fence; send with stable idempotency key
EXTERNAL: applies effect (response may be lost)
LOCAL TX: persist provider receipt + verified outcome
```

Crash між останніми двома рядками — `UNCERTAIN`. Adapter classes:

| Class | Recovery |
|---|---|
| Transactional internal effect | Atomic commit/unique operation key; replay reads existing result. |
| Provider-deduplicated effect | Same key + same semantic payload within documented retention; query/receipt verification; never switch key after timeout. |
| Queryable effect with authoritative identity | Reconcile exact business operation. A stale negative read is not proof of no effect; require documented consistency/uniqueness before retry. |
| Opaque non-idempotent effect | Strict default: inadmissible for required duplicate-free irreversible execution. Already in doubt: no blind resend, manual/provider reconciliation. |

Key retention expiry, payload mismatch, weak read consistency або consumer без dedupe позбавляють adapter його strong recovery class. Fencing у локальній базі не скасовує запит, що вже полетів у зовнішній світ. Повторно використана бізнес-ідентичність зберігається й між session/model/runtime changes. Compensation — нова authorized effect, не «видалення» початкової історії.

### 14.3 Session/window limit

Session є worker lease, не run. До limit worker припиняє брати нові неподільні дії, flush-ить journal/evidence/patch, здає lease й записує continuation capsule: run ID, durable cursor, artifact IDs, unresolved intents, next predicates та policy lock. Kernel запускає нову дозволену session, завантажує мінімальний релевантний context із authoritative records і продовжує. Model summary — допоміжний index, не заміна bytes/receipts.

Hard kill проходить звичайний recovery; work-in-progress зберігається checkpoint-ами в межах promised failure domain. Некомічені зміни не потрібно пушити для передачі контексту. Ліміти account, rate і spending не обходяться паралельними акаунтами чи новими сесіями. Без зовнішнього supervisor restart не автоматичний; deployment має чесно позначити цю відсутню capability.

### 14.4 Runtime/version/capability drift

Execution lock включає exact runtime/adapter/policy/schema identities, dependency locks, model requirements, prompt/review contract version і capability observations. На admission, restart, boundary before effect та після drift signal робиться revalidation. Dynamic capability probe має expiry; старий successful login не достатній назавжди.

Installed runtime differs from pinned: не тихо прийняти installed. Запустити pinned у sandbox або виконати лише наперед дозволений compatibility path із machine-tested contracts, authority equivalence та новим lock revision. Schema migration — versioned, tested, transactional, із rollback/backups; unsupported newer journal schema відкривається read-only або parked, не destructive downgrade. Новий runtime не отримує владу змінити grant parser на свою користь.

Required model unavailable: substitute лише якщо approval уже описує acceptable identity/class, privacy/data location, capabilities, budget і reviewer independence. Конкретно обов'язкову модель не можна оголосити необов'язковою. Залежні predicates чекають; незалежні працюють. Якщо єдиний шлях — змінити Owner-approved model/privacy/spending boundary, надсилається вузький decision request.

Provider alias без immutable version marker записується як unresolved alias із observation time/capability tests, не вигаданий pinned identity. Critical operations/reviews із вимогою відтворюваності залишаються blocked, якщо identity неможливо довести; policy може дозволити обмежений режим із чесно вказаною невизначеністю.

API changed: contract probes відокремлюють transport outage від semantic/schema change. Adapter quarantine зупиняє відповідні effects. Локальна сумісна repair, sandbox test та незалежна verification дозволені в grant; зміна scopes, data placement, money, irreversible semantics чи acceptance потребує Owner delta. Старий unknown request не перевідправляється під новою API semantics без reconciliation.

## 15. Pseudocode: autonomous execution loop

Це специфікація, не твердження про наявний код. Функції з `tx_` атомарні; ефекти виконуються лише gateway; dispatcher/supervisor живе поза model session.

```python
async def execute(run_id):
    run = durable.load(run_id)
    epoch = tx_acquire_controller_and_fence_prior(run_id)
    await reconcile_run(run, epoch)       # replay, never reset to blank task

    while not durable.is_final(run_id):
        run = durable.load_current(run_id)
        assert_controller_epoch(run_id, epoch)
        await drain_deduplicated_inbox(run_id)
        await reconcile_dispatched_or_uncertain_effects(run_id, epoch)

        if run.cancel_requested or authority.revoked(run.grant):
            tx_stop_new_actions(run_id)
            await settle_or_record_inflight_effects_read_only(run_id)
            tx_close_as_cancelled_with_explicit_uncertainties(run_id)
            break

        if execution_lock_or_capabilities_drifted(run):
            tx_pause_affected_nodes(run_id)
            await restore_pin_or_apply_preapproved_compatibility(run_id)
            await invalidate_impacted_evidence(run_id)
            # unresolved drift remains a typed wait, not implicit permission

        await admit_new_candidates_and_revalidate_existing(run_id)
        await classify_failures_and_findings(run_id)
        await repair_or_replan_within_contract_and_limits(run_id)
        # same failed hypothesis is not blindly replayed; findings persist

        for need in durable.unresolved_human_dependencies(run_id):
            if need.is_external_permission:
                tx_outbox_request_once(secure_permission_packet(need))
            elif is_material_owner_decision(need, current_grant(run_id)):
                tx_outbox_request_once(bounded_decision_packet(need))
            else:
                tx_route_to_machine_diagnostic_or_wait(need)
        # A human request blocks only its dependency closure.

        if all_non_delivery_completion_checks_pass(run_id):
            await ensure_authorized_required_delivery(run_id, epoch)
            # inspect existing commit/ref/receipt before any new effect

        if completion_verifier.all_required_predicates_hold(run_id):
            tx_freeze_artifact_and_record_single_final_receipt(run_id)
            tx_outbox_final_summary_once(run_id)
            break

        frontier = scheduler.ready_allowed_actions(run_id)
        if frontier:
            for action in scheduler.select_within_reserved_limits(frontier):
                lease = tx_lease_and_reserve(action, epoch)
                intent = tx_record_intent_and_outbox(action, lease)
                await gateway.execute_or_reconcile(intent)
                # timeout => UNCERTAIN; verified failure => typed repair input
            continue

        if can_make_new_information_gain_within_policy(run_id):
            await diagnostic_worker.propose_alternative_or_probe(run_id)
            continue

        waits = derive_wait_conditions_from_current_records(run_id)
        assert no_ready_authorized_independent_action(run_id)
        tx_checkpoint_and_park(run_id, waits)
        await supervisor.wait_for_durable_trigger(run_id, waits)
        # event/timer/permission/decision/capability; no chat message relay
        epoch = tx_reacquire_if_needed_and_reconcile(run_id, epoch)

    return durable.final_receipt_or_explicit_terminal_status(run_id)
```

`gateway.execute_or_reconcile` перевіряє adapter recovery class, чинний grant/epoch, semantic payload і budget до dispatch. Воно ніколи не використовує новий key просто через timeout. Await зовнішнього виклику не повинен блокувати scheduler незалежних workstreams: production implementation використовує асинхронні bounded tasks/outbox workers, а не один network call у global database lock.

Session yield не є виходом із `while not final` для логічного run: supervisor зберігає ownership continuation. Коли deployment не має supervisor, фактична функція повертає `PAUSED_NEEDS_EXECUTOR` із cursor, а не final success. Обіцянка майбутнього виконання без такого механізму заборонена.

## 16. Мінімальна реалізація та acceptance самої системи

Почати з одного kernel, одного integrator та bounded workers, не з десятків агентів. Порядок: захищений authority registry → journal/outbox/inbox та artifact store → admission/graph/scheduler → sandbox/gateway/Git adapter → evidence/review gates → permission/decision UI → restart/drift/fault harness. Dashboard додається як projection, а не як control-plane база даних.

Перед використанням для production потрібні executable tests: crash у кожній межі intent/dispatch/receipt; repeated restart; concurrent controller fencing; replayed approval; grant revocation during dispatch; symlink/path escape; stale artifact review; secrets in logs; multiple worktree conflict; remote ref race; duplicate notification delivery; idempotency-key expiry; spend reservation race; unsupported schema; model alias drift. Сценарії в `SCENARIO_RESULTS.md` — design traces, не заміна цих тестів.

Контрольоване приймання системи: нуль unauthorized effects у harness; нуль повторених business operations для adapters із перевіреним strong contract; opaque uncertain operations не повторюються; worker claim без evidence не фіналізує run; restart не губить findings/requests/receipts; independent frontier продовжується; final predicate неможливо обійти текстовим STATUS. Якщо adapter або storage не відповідає припущенням, клас гарантій явно знижується або відповідна дія не допускається — ніколи не приховано.
