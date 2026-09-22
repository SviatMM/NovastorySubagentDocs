# Experiment plan: чому turns закінчуються приблизно через хвилину

**Статус:** reproducible plan, НЕ виконаний live experiment. У цьому research немає доступу до фактичного Codex process на Mac, його protocol trace або exact current client/config. Немає вигаданих 10 successful runs, runtime durations чи pass rate.

Мета: розрізнити terminal mechanism, вплив prompt і поведінку client після terminal. План мінімум має 12 послідовних контрольних trials; окрема серія перевіряє 10 корисних automatic continuations. Definitions — [TRACE_SCHEMA](TRACE_SCHEMA.md), operational bounds — [SUPERVISOR_DESIGN](SUPERVISOR_DESIGN.md).

## 1. Що вважається спостереженням

Trial — конкретний task на зафіксованій fixture/config; provider turn — одиниця протоколу. Один trial може мати кілька turns. Зберігати **кожен** turn і кожен rejected start; не залишати тільки успішні й не зливати кілька turns в одну хвилину UI.

Первинні outcomes: normalized terminal category + evidence IDs, true turn duration, safe runnable frontier на кінці, automatic next dispatch або його відсутність. Secondary: token observations, useful tool count, last action/error, client UI claim, time-to-next-dispatch, unmet criteria, Owner gate decision.

Завершення простого task за 20 секунд — нормальний результат. Не вимагати minimum runtime. Не використовувати `sleep`, empty polling, безмежні цикли, штучно великі генерації чи brute force для заповнення хвилини. Вимірювати корисну роботу з кінцевим observable acceptance.

## 2. Preflight і experimental controls

Зняти exact app/CLI build, executable і generated schema digest, macOS build, model/provider/effort, auth mode, sandbox/approval config, active MCP servers і їхні таймери, goal state/token budget, hooks/timeout/driver, effective instruction loading manifest, relevant environment override **names**, not secret values. Capture user submission, dispatch, first action, last action, terminal і process lifetime.

Зберегти local-only prompt bytes/digests і resolved instruction precedence для кожного trial. Не публікувати private instructions/paths. Параметри hidden system prompt позначити unobservable. Не міняти runtime/model або auto-update під час batch; виявлене оновлення розбиває batch на окремі версійні cohorts.

Запустити на sandboxed synthetic fixture, не NovaStory/AI OS. Одна baseline revision, fixed public test expectations, dependency versions, machine settings, один writer. Кожний comparative trial отримує свій disposable checkout і **новий persistent thread** для усунення carryover; E2E continuation phase нижче навмисно використовує один thread. Окремий config root потрібний, щоб neutral prompt не підвантажив AUTONOMOUS_BLOCK через shared AGENTS/skills; copied credentials/config secrets у fixture заборонені.

Заздалегідь оголосити finite whole-batch budget; ceilings у SUPERVISOR_DESIGN — pilot proposal, не автоматичний spend authorization. Не паралелити trials і не запускати unrelated heavy workloads. Record unavoidable background-load/power/network anomalies. Відключення checkpoint language не означає видалення security чи Owner boundaries.

## 3. Reproducible fixture

Створити small Python-stdlib project `fixture`, offline після preparation. Пакет обробляє локальні JSON records, має pure normalization/validation, deterministic serialization, file reader та CLI; обов'язкові malformed/duplicate/empty/unicode cases. Test runner — `python3 -m unittest discover -s tests -v`; input corpus фіксований, versioned, без реальних персональних даних. Без package installation під час trials.

Harness підготовки створює tracked task manifest Q01–Q12 і **захищений expected-acceptance manifest поза writable fixture**. Worker може правити реалізацію і власні tests, але controller запускає незалежні checks/fixtures. Штатний task result — source diff, tests і конкретна acceptance evidence; не лише оновлена галочка.

Q01–Q12: normalize text, validate required fields, reject duplicate IDs, stable sort, deterministic JSON serialization, read input safely, useful CLI errors, CLI success path, unicode cases, malformed input cases, integration regression, concise usage documentation. Кожен має один або кілька fixed tests/checks. Це кінцева корисна робота, а не timer benchmark. Для довшого condition C додати finite cross-case matrix над цими behaviours, а не збільшувати без потреби обсяг CPU work.

## 4. Шість conditions

| Code | Condition | Конкретний task і відмінність |
|---|---|---|
| A | Trivial | Реалізувати одну pure normalization behaviour та 2 supplied assertions; завершити, коли verified. Немає прихованого TODO. |
| B | Multi-step repository | Validation + serialization + CLI integration + tests + docs в одному task. Формулювання про спостережуваний outcome, не кількість turns. |
| C | Explicit long-running safe | Виконати весь finite Q01–Q12 backlog і declared regression matrix. Явно дозволені bounded local repair/review; не просити працювати N хвилин. |
| D | Remaining TODO queue | Той самий queue-family baseline Q01–Q12; durable manifest і checkpoint-after-verified-work language, без ствердження про platform limit. |
| E | No checkpoint language | Як D: ті самі requirements/authority/queue/acceptance, але прибрати session/window/checkpoint/stop wording з controllable instructions. Harness сам snapshots state; це не модельний checkpoint prompt. |
| F | Current AUTONOMOUS_BLOCK | Як D/E, але додати локально exact applicable current AUTONOMOUS_BLOCK instructions із canonical SHA `713928f1425374bac6dfdf13ba842d582ce5c37b`; private bytes не публікувати. |

D/E/F — paired prompt controls, а не різні продуктові завдання. Для C/D/E/F треба запитувати завершити весь admitted backlog, **не інструктувати “виконай один крок і зупинись”**. Якщо все завершено в одному turn, зафіксувати це як результат, не вигадувати remaining TODO для примусу continuation.

F manifest включає applicable bootstrap/role/workflow/adaptive/skill/increment layers та hashes. Task-specific instructions не повинні знищити досліджуваний stop-language effect сильнішим artificial system message. Коли isolated F не може faithfully відтворити current installation, назвати його controlled reconstruction і додати окремий native-config cohort.

## 5. Мінімальний batch: 12 consecutive trials

Pre-register порядок без post-hoc відбору:

| Trial | Condition | Repetition |
|---|---|---|
| 01 | A | 1 |
| 02 | B | 1 |
| 03 | C | 1 |
| 04 | D | 1 |
| 05 | E | 1 |
| 06 | F | 1 |
| 07 | F | 2 |
| 08 | E | 2 |
| 09 | D | 2 |
| 10 | C | 2 |
| 11 | B | 2 |
| 12 | A | 2 |

Це мінімальний descriptive diagnostic sample, не statistical proof із двома repetitions. Незрозумілий результат веде до bounded додаткових matched repetitions, не до висновку “винен prompt”. Ledger не видаляє rejected starts/timeout/невдалі trials. Якщо batch перервав account limit, чесно вказати completed count <12 і exact resume condition; не фальсифікувати досягнення мінімуму.

### Phase P0 — перевірка вимірювача, до витрачання provider usage

Fake App Server/recorded synthetic JSONL і virtual clock перевіряють classifier: completed+TODO, real cancel, known host deadline, silence timeout, recovered/fatal tool failure, quota/throttle/session cap, process kill, EOF-only, duplicate/late events, invalid frame, lost ACK, unrelated turn events, storage error. Fault fixtures чітко tagged `synthetic=true`. Вони доводять інструментування, не production root cause.

### Phase P1 — natural termination capture

Виконати 12 trials на current failing native client, з passive capture, без нового external auto-continue, без нових Stop hooks і без Goal mode, **якщо вони не були частиною реальної failing configuration**. Фактичну configuration спочатку виміряти; якщо goal/hook уже були активні, залишити для native baseline і додати окремий off cohort.

Між окремими trials harness/operator задає наступний зафіксований task; це experimental scheduling, не remedy Continue. Усередині trial після natural terminal спочатку зберегти state й незалежно перевірити frontier. Немає ручного “підштовхування” до завершення до snapshot.

Якщо UI не дозволяє passive protocol capture, зберегти доступні logs/build/UI timestamps, а unavailable fields = null/UNKNOWN. Screenshot/final sentence не підміняє terminal event. Не декларувати HOST_TURN_TIMEOUT у native cohort, коли host event не видимий.

### Phase P2 — matched raw App Server

Повторити той самий 12-trial order через інструментований App Server. Зіставити binary/schema, model/config/auth із P1. Це ще 12 trials, а не змішування різних клієнтів в один sample. Якщо builds/policies відрізняються, порівняння позначити confounded. Спостережуваний headless normal completion не виключає native-host timeout в іншій конфігурації.

Natural-stop mode controller P2 **записує** hypothetical gate і frontier, але не auto-start усередині trial. Так можна відділити механізм завершення від ефективності remedy.

### Phase P3 — actual automatic continuation

На окремому fresh workspace/thread увімкнути лише `continuation_driver=external_controller`. Виконати один admitted finite run із щонайменше 10 заздалегідь визначеними корисними task units (Q01–Q10 або більше), які controller admission scheduler передає послідовно. Така task-unit серія тестує controller lifecycle, **не використовується як доказ natural one-step stopping**.

У цій серії кожна unit має реальний check і власний provider turn, а наступну unit controller dispatches без Owner message після verified completion. На final units також перевірити regression/docs. Додатково використати один whole-backlog prompt із natural early completions: gate повинен продовжувати його тільки за реального remaining frontier. Не просити модель марнувати час, щоб набрати 10 turns.

Критерії: ≥10 послідовних accepted turns/task units на одному thread; zero Owner Continue messages; кожний completed+eligible результат дає durable continuation; не більше одного active turn; exact terminal category для кожного; один verified final; attempts/budgets не reset. Якщо модель або ресурси не дають ≥10 valid turns, записати незавершену серію, а не додавати synthetic turns до live count.

Окремі bounded recovery subtests: kill App Server після acknowledged action; kill controller після persisted dispatch intent; simulate lost ACK; authenticated cancel; duplicate controller start; stale verifier evidence; бюджет вичерпано. Не вбивати сторонні процеси, не використовувати destructive/external production effects. Перевірити повернення того самого run, budget і thread lineage, не replay unknown mutation.

### Phase P4 — optional native remedy comparison

Окремо, не одночасно, випробувати `/goal` та synchronous Stop hook на тому самому queue task. Зберегти native goal budgets і `stop_hook_active`, hook reason/exit/timeout. Compare human interruptions, durable-state freshness, final visibility, recovery. Непідтриманий feature = unsupported у цьому build, не глобально відсутній продукт [O3,O4].

## 6. Harness interface contract

Нижче **пропонований CLI майбутнього harness**, не існуюча команда Codex:

```sh
runtime-probe prepare --fixture records-v1 --manifest experiment.json
runtime-probe batch --cohort app-server-natural --order A,B,C,D,E,F,F,E,D,C,B,A
runtime-probe verify-frontiers --batch <batch-id>
runtime-probe continuation --units Q01:Q10 --driver external-controller
runtime-probe export --batch <batch-id> --public-metadata-only
```

Harness `experiment.json` фіксує baseline SHA/digest, fixture generator version, seed, per-condition instruction hashes, allowed actions/checks, runtime fingerprint, max resource envelope і sequence. Preparation не змінює існуючі source repos. Same manifest дозволяє повторити generation; private F prompt отримується локально з authorized checkout, не завантажується з public artifacts.

## 7. Result ledger

Один normalized JSON/CSV row на observed turn/start attempt, плюс raw sanitized events і durable gate receipts:

```text
batch_id,trial_id,condition,rep,client_kind,client_version,codex_version,
model,config_hash,prompt_hash,run_id,attempt_id,threadId,turnId,
start_monotonic_ns,end_monotonic_ns,wall_duration_ms,duration_quality,
terminal_protocol_event,protocol_status,terminal_category,cause_confidence,
evidence_event_ids,subprocess_exit,timeout_source,tool_call_count,
last_action,last_error,usage_limit_state,usage_coverage,
frontier_count,unmet_predicates,frontier_completeness,
owner_response_gate_decision,next_dispatch_id,owner_continue_count,coverage_gaps
```

Full frontier і evidence refs зберігаються JSON, не втрачаються через scalar CSV export. Public exports замінюють IDs на synthetic stable aliases і не містять actual private prompts/paths/tool text. `not_run`, `unsupported`, `unknown` різні значення. Немає fabricated zero resource usage.

## 8. Як робити висновок

- Повторюваний trusted deadline fire для того самого turn доводить відповідний **host/controller timeout у виміряній конфігурації**. Модельна фраза не потрібна.
- Normal completed events з verified TODO доводять early turn completion, не його внутрішню мотивацію. Відсутність next dispatch у fully captured client доводить, що саме цей path не продовжив run.
- Fatal typed quota/throttle error дозволяє відповідну classification; server overload, tool timeout і session budget не змішуються.
- Вплив F проти D/E оцінювати лише при однаковій базі й відсутності confounders. Дві repetitions дають сигнал для наступного bounded тесту, не остаточне causal attribution.
- Кластер біля 60 секунд аналізувати від dispatch, first tool і last stream activity окремо. Порівняти з actual MCP/hook/client timer settings. Ongoing useful events протягом хвилини суперечать **silence** timeout саме цього інтервалу, але не виключають hard wall deadline.
- У synthetic positive control можна зсунути власний timer threshold і перевірити moving cutoff. Не видавати injected timeout за reproduction невідомого production host limit.

Підсумковий verdict містить observed counts, точні evidence IDs, unknown fraction, причинний chain і межу узагальнення. Root-cause flag можна змінити тільки після реальних receipts із конкретної failing environment. Документи й mock tests самі цього не доводять.
