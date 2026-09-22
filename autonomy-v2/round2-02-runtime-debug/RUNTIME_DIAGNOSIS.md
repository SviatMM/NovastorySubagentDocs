# Runtime diagnosis: чому Codex завершує короткі turns

**Дата дослідження:** 2026-09-22. **Статус:** source-inspected design; live runtime cause НЕ встановлена. Цей пакет не встановлює supervisor і не змінює AI OS.

## 1. Висновок і межа доказу

`root_cause_proven = false`. Надані повідомлення про нездатність закінчити роботу в одному turn або execution window — це текст моделі, а не подія таймера, завершення процесу чи відмова сервісу. Приблизна тривалість у хвилину сама по собі не доводить жодного з цих механізмів.

Потрібно розділити три речі: **закінчився provider turn**, **припинився runtime**, **виконано durable outcome**. Вони можуть статися незалежно. Звичайний завершений turn із незакінченою задачею — достатня причина для перевірки continuation gate, але недостатня для діагнозу добровільного рішення моделі: runtime міг завершити turn внутрішнім механізмом, який протокол не пояснює.

Практична рекомендація: спочатку інструментований baseline, потім один локальний власник continuation навколо Codex App Server. Він перевіряє durable state й сам запускає наступний turn; Owner не перекидає повідомлення Continue. Поточні офіційні документи також описують `/goal` і `Stop` hooks, тому твердження, що будь-який сучасний UI взагалі не здатний автопродовжувати, було б неправильним. Але їхня наявність у конкретній інсталяції й гарантія нашого response gate потребують окремого тесту [O1–O4].

## 2. Джерела й provenance

Canonical inspected: `SviatMM/ai-operating-system@713928f1425374bac6dfdf13ba842d582ce5c37b`. На момент читання main відповідав цьому SHA. Репозиторій приватний; нижче тільки locators і власний аналіз, без перенесення його текстів, коду, локальних шляхів або credentials. Аудит обмежений переліченим execution path, не всією історією і не невідомою локальною інсталяцією.

| ID | Прочитаний файл | Git blob SHA |
|---|---|---|
| A01 | `README.md` | `d408a0cd39d5b827ce29f0fe64344b9c4cab363d` |
| A02 | `core/PRINCIPLES.md` | `8e356c75ee7effb0cd55223ece7fdee1a65c55ad` |
| A03 | `core/AI_WORKFLOW.md` | `2af3dd099c309b2870f21976b22e66ae0dda496e` |
| A04 | `profiles/cpto/PROFILE.md` | `3f3e4bbd40f3e9677493cda3b211e64cab0a6b19` |
| A05 | `core/ADAPTIVE_DELIVERY.md` | `0db03722d599ca144c0cdb7cec959f17960f95d8` |
| A06 | `adapters/codex/AGENTS.md` | `3a281a31a0fa9e49e04f672245bdf520fad26f1f` |
| A07 | `adapters/codex/.agents/skills/deliver-large-projects/SKILL.md` | `9246734eef29648f91b36d87288999fa7387e127` |
| A08 | `templates/product-repository/goals/_template/plans/CURRENT_INCREMENT.md` | `d6af34395398a7b20698942b761842f098bf82be` |
| A09 | `templates/product-repository/goals/_template/STATUS.md` | `4cc6ab05398bf5f35a022097f9a17c72b0e25e27` |
| A10 | `scripts/simulate_autonomous_block.py` | `12bfed693b9444d8e39fe7d7a128dfc9bdcc89e7` |

Research inputs: Agent02 `4562b8e453e3e37cc3d96ee09bf6f468322710f3` (RESULT та початок GREENFIELD_ARCHITECTURE); Agent03 `1c53a1b09a802b1fb0ee3045d0b3c0568eecafe3` (RESULT та MINIMUM_DEFENSES, розділи 1–8). Використані вимоги: реальний response boundary, журнал, один writer, bounded retries, заборона blind replay. Їхні архітектурні висновки й компонентні проби не є вимірюванням поточних хвилинних Codex turns [R2,R3].

## 3. Кандидати причин і критерії розрізнення

| Кандидат | Що має бути в trace | Чого недостатньо | Контроль |
|---|---|---|---|
| A: host-enforced timeout | Конкретний host/client watchdog, конфігурація, arm/fire, scope, ідентифікатор interrupt/kill, matching turn | Фраза моделі; тривалість 59–65 секунд; SIGTERM без provenance | Перевірити реальні таймери; окремо injected watchdog із відомим deadline |
| B: нормальне/добровільне закінчення turn | `turn/completed`, `turn.status=completed`, незакритий verified frontier, без зафіксованої примусової причини | Тільки фінальний текст; `completed` не розкриває внутрішньої мотивації | Повторити з незмінним runtime і контрольними prompt variants |
| C: usage/fair-use | Структурована релевантна відмова: quota/throttle/session budget; account snapshot, коли доступний | Будь-який HTTP 429; припущення про тариф; невідомий ліміт, записаний як zero | Зафіксувати auth mode, помилку і reset; не обходити ліміт іншим акаунтом |
| D: tool/runtime failure | Matching item failure, error chain, process identity/exit, terminal status і `willRetry` | Ненульовий exit тесту або короткий polling/yield | Відділити очікуваний failing test від fatal tool failure |
| E: client orchestration | Turn завершений, клієнт живий, наступний dispatch не створено попри eligible frontier | Відсутність подій у неповному UI export | Порівняти native UI і raw App Server на тому самому build/config |
| F: prompt/checkpoint effect | Залежність завершень від контрольованої зміни window/checkpoint language при однакових завданні й authority | Наявність слова window в інструкціях | Парне порівняння neutral/current AUTONOMOUS_BLOCK |
| G: інше | Context/session budget, policy stop, sleep/logout, hook result, approval wait, transport EOF, compaction або schema mismatch | Підгонка невідомої події під timeout | Зберегти raw discriminants; класифікувати UNKNOWN до доказу |

Один run може мати кілька причин: timeout інструмента → модель завершує turn → UI не запускає наступний. Trace повинен показати ланцюг, а не одну зручну етикетку. Документована механічна прогалина continuation не доводить, чому саме перший turn закінчився.

## 4. Карта таймерів: не змішувати рівні

| Рівень | Перевірений факт / proposed observation |
|---|---|
| App Server turn request | У перевіреному `TurnStartParams.ts` немає поля turn timeout. Це не доказ відсутності прихованого host/service limit [O6]. |
| MCP tool | Config reference задає `mcp_servers.<id>.tool_timeout_sec`, default 60 s; startup timeout окремий. Підозрілий кандидат лише коли саме цей tool запускався [O5]. |
| Provider stream | `model_providers.<id>.stream_idle_timeout_ms`, documented default 300000 ms, стосується idle stream, не загальної тривалості задачі [O5]. |
| Symphony | `read_timeout_ms` для startup/sync RPC; `turn_timeout_ms` у поточному SPEC §10.6 — silence interval, reset на app-server output; `stall_timeout_ms` — orchestrator inactivity. Не називати їх Codex hard runtime cap [O8]. |
| Native goal/hooks | Фіксувати goal state/token budget та hook invocation/output/timeout; це окремі джерела lifecycle, а не inferred model deadline [O3,O4]. |
| Наш controller | Явні hard turn deadline, inactivity watchdog і lifetime budgets з окремими timer IDs. Значення в SUPERVISOR_DESIGN — пропозиція, не defaults OpenAI. |
| OS/client/remote host | Sleep, процес-власник, транспорт, runtime wrapper, cancellation provenance. Без їхніх logs не доводимо, хто обірвав роботу. |

Універсальний підтверджений 60-секундний ліміт Codex у прочитаних джерелах не встановлений. Максимальна практична робота визначається ресурсами, доступністю Mac, моделлю/контекстом, permissions і правилами конкретного хоста, не довжиною одного prompt.

## 5. Аудит prompt path

A02–A08 уже вимагають продовжувати safe in-scope work і не перетворювати checkpoints на звіти Owner. Зберегти цю логіку. Проблема не доведена як відсутність відповідного побажання.

Знайдені прогалини:

1. У A03–A08 повторюється семантика фізично завершеного execution window, але немає обов'язкового trusted evidence record, що дозволяє присвоїти `EXECUTION_WINDOW_LIMIT`. Модель може помилково використати цей exit path для нормального завершення. Це **causal hypothesis**, не результат A/B тесту.
2. A07 описує міжсесійне відновлення; A09 містить зрозумілий людині наступний крок. Вони не створюють нового provider turn автоматично.
3. A10 — policy simulator. Він приймає назву execution-window події від caller, а не спостерігає host. Його `owner_response=false` не керує реальним response channel. Exit zero означає валідність запису, не доведене завершення роботи.
4. Предикати/applicability у симуляції не є незалежно виміряною чергою робіт. Їх не можна безпосередньо використати як authoritative durable gate.
5. A05 уже обмежує повторення однакових repairs; новий controller має зберігати лічильники між turns/restarts, а не просто повторити цю вимогу в prompt.

Приклад із запиту про save STATUS and stop розглядається як **патерн формулювання**, не як встановлена дослівна цитата з canonical source. Нічого з приватного тексту тут не відтворюється.

### Запропоноване нове формулювання — не встановлене правило

```text
AUTONOMOUS_RUN_CONTINUATION
The admitted outcome spans provider turns. A completed provider turn is not
completion of the run and is not evidence of an execution-window limit.

Continue safe, authorized, executable work. Persist verified checkpoints after
meaningful actions; a checkpoint is not a reason to stop or ask Owner to continue.

Only the controller may assign EXECUTION_WINDOW_LIMIT. It requires a trusted
runtime evidence record matching this run/thread/turn and identifying the
external deadline or termination mechanism, its source, and the observed event.
A model-authored sentence, estimated time, context discomfort, or normal
turn/completed event does not satisfy this requirement.

If no such evidence is visible, do not claim a runtime time limit. State the
observed facts and remaining work in the internal result. The controller will
verify durable state and schedule the next eligible turn within persistent
budgets. Do not claim a supervisor exists unless it is actually running.

Honor genuine cancellation and authority boundaries. Technical exhaustion,
missing telemetry, and resource exhaustion are operational park states, not
fabricated OWNER_BLOCKER decisions. Never delete an obligation to become done.
```

При SIGKILL чи power loss модель не може гарантовано записати останній STATUS. Controller повинен мати раніше зафіксовані receipts і відновлювати ambiguous in-flight роботу. Prompt цього не виправить.

## 6. Що реально може native UI

За поточною документацією `/goal` підтримує довгу роботу у desktop, interactive CLI та IDE; `Stop` hook може повернути блокувальне рішення з reason для continuation. Це реальні, але version-conditioned кандидати найменшого pilot [O3,O4].

**Не встановлено:** чи є вони в конкретній користувацькій версії, чи активні, чи goal має бюджет, чи hooks trusted/loaded, чи native client уже має власний автопродовжувач. Не підставляти версію зі старих розмов замість preflight поточної інсталяції.

**Межа гарантії:** звичайний prompt не володіє response transport. Hook може блокувати stop, але documented `suppressOutput` ще не реалізований; не обіцяти, що він приховає вже показаний текст. Нативні можливості не доводять наші правила evidence freshness, lifetime budgets чи відновлення після смерті controller. Для жорсткої вимоги «незакінчена безпечна робота не доходить до Owner як progress-only final» використовувати власний output boundary у supervisor. Не запускати `/goal`, auto-Stop і external `turn/start` як трьох конкурентних drivers.

## 7. Що потрібно для доведеного діагнозу

Потрібні 12 послідовних контрольних trials за EXPERIMENT_PLAN: protocol capture + process monitor + timer provenance + independently verified frontier. Зібрати trace на **тому самому client/build**, де спостерігається проблема. Якщо UI не дає protocol events, позначити обмеження і повторити через інструментований App Server; headless результат не приписувати native UI без matched configuration.

Доведена normalized category для одного turn не дорівнює доведеній універсальній root cause. Висновок має містити IDs виміряних trials, покриття missing events, контрприклади, і що саме виключено. Цього пакета недостатньо, щоб назвати причину поточних хвилинних turns.

## 8. Посилання

Усі changing product facts перевірялися 2026-09-22; generated schemas installed binary залишаються остаточним protocol contract для реалізації.

- **O1:** [Official App Server documentation](https://developers.openai.com/codex/app-server/), на момент перевірки redirect до [ChatGPT Learn App Server](https://learn.chatgpt.com/docs/app-server).
- **O2:** [Official CLI reference](https://developers.openai.com/codex/cli/reference/) — перевірити installed help разом із generated schema перед activation; цей locator не є твердженням про конкретну локальну версію.
- **O3:** [Official long-running work](https://learn.chatgpt.com/docs/long-running-work).
- **O4:** [Official hooks](https://learn.chatgpt.com/docs/hooks).
- **O5:** [Official configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference).
- **O6:** [OpenAI Codex source at inspected SHA](https://github.com/openai/codex/tree/c6c09fe29fb0b926ced5146e6ed47b6ff756c1a4/codex-rs/app-server-protocol/schema/typescript/v2): `TurnStartParams.ts`, `ThreadStartParams.ts`, `TurnCompletedNotification.ts`, `ThreadTokenUsageUpdatedNotification.ts`.
- **O7:** Same pinned source: `CodexErrorInfo.ts`, `ErrorNotification.ts`. Wire enum casing comes from these files, not prose labels in documentation.
- **O8:** [OpenAI Symphony SPEC at inspected SHA](https://github.com/openai/symphony/blob/be10a1b79df723d6d7612b5651c8522704dafb2e/SPEC.md), especially §§8.5, 10.1–10.7. Orchestration inspiration, not a replacement protocol schema.
- **R2:** [Agent02 pinned RESULT](https://github.com/SviatMM/NovastorySubagentDocs/blob/4562b8e453e3e37cc3d96ee09bf6f468322710f3/autonomy-v2/agent-02-greenfield/RESULT.json).
- **R3:** [Agent03 pinned minimum defenses](https://github.com/SviatMM/NovastorySubagentDocs/blob/1c53a1b09a802b1fb0ee3045d0b3c0568eecafe3/autonomy-v2/agent-03-redteam/MINIMUM_DEFENSES.md).

Next: [TRACE_SCHEMA](TRACE_SCHEMA.md), [SUPERVISOR_DESIGN](SUPERVISOR_DESIGN.md), [EXPERIMENT_PLAN](EXPERIMENT_PLAN.md), [IMPLEMENTATION_PLAN](IMPLEMENTATION_PLAN.md).
