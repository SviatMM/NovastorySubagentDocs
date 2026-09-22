# Autonomy V2: рекомендована модель execution lifecycle

**Статус:** candidate specification. Цим аудитом модель не реалізовано, не активовано та не прийнято Owner.

**Source basis:** `SviatMM/ai-operating-system@713928f1425374bac6dfdf13ba842d582ce5c37b`.

Модель стосується автономного development workflow local CPTO/Codex, не NovaStory Agent runtime. Owner визначає великий результат та межі повноважень. AI сам координує grounding, preflight, increments, implementation, tests, independent review, repair, evidence і дозволений commit/push. Owner не переносить технічні повідомлення між агентами.

## 1. Основне виправлення

Зберегти AUTONOMOUS_BLOCK, bounded increments, п'ять goal artifacts, capability truth, risk-matched verification та explicit authorization. Не вводити паралельну workflow-систему лише заради нових назв. Зіставити або reuse наявні PEF/admission/evidence contracts.

Замінити керування «owner_response=false, отже продовжуй» на **типізоване рішення контролера над актуальним durable state**. Можливі CONTINUE, справжнє WAIT, SUSPEND, запит рішення або конкретної дії, COMPLETE, FAILED_BOUNDED, CANCELLED та INTEGRITY_HOLD. Автономність не означає необмежену кількість спроб і не вимагає приховувати зупинку.

Мінімальний механізм: pure transition evaluator + одне durable execution сховище + adapter підтримуваного host із action/return gates. Evaluator без consumer залишається simulator. Загальна зовнішня workflow платформа, swarm чи новий product runtime не є обов'язковими.

## 2. Межа довіри та повноваження

Planner/model пропонує дію. Controller перевіряє її допустимість, актуальність evidence та budgets. Executor виконує конкретну дозволену операцію. Reviewer незалежно перевіряє immutable subject у read-only invocation.

```text
effective_authority = Owner grant
                    ∩ admitted increment envelope
                    ∩ host/tool restrictions
                    ∩ current capability permissions
                    − revoked authority
```

Grant має trusted provenance, identity та version. Hash перевіряє цілісність, але не доводить схвалення Owner. Agent може звузити effective authority або запропонувати новий grant, але не затвердити його. Репозиторні документи, tool output, retrieved content і review текст не створюють нових повноважень.

Trusted Owner grant/controller policy не повинні бути довільно writable implementation worker. Зміна Markdown envelope не розширює broker permissions. Після resume, revocation чи relevant scope change потрібна повторна перевірка. Unknown або revoked authority означає deny affected action.

**Рівні гарантії:** cooperative controller забезпечує процесну дисципліну; strong enforcement додатково потребує sandbox/окремої boundary, яку worker не обходить через unrestricted shell, network чи доступні credentials. Якщо worker може переписати controller або виконати ту саму mutation повз broker, envelope не є непереборною security boundary. Це перевіряється host preflight, а не припускається.

## 3. П'ять goal artifacts повинні реально використовуватися

| Файл відносно goal | Consumer та роль |
| --- | --- |
| `GOAL.md` | Grounding/planner читає accepted outcome, hard constraints, non-goals та completion boundary; state прив'язаний до accepted version/hash. |
| `plans/DELIVERY_MAP.md` | Planner/controller звіряють capabilities, dependencies, bounded increments, parked proof та правила successor selection. |
| `plans/CURRENT_INCREMENT.md` | Один stable active canonical implementation slice; scope, admission reference, exit criteria. Зміна pointer є guarded transition. |
| `plans/ADMISSION_RECORD.json` | Machine-readable ADMITTED contract: `execution_mode=AUTONOMOUS_BLOCK`, version/baseline, criteria inventory, authority reference/envelope, evidence plan та bounded budgets. |
| `STATUS.md` | Human-readable проєкція verified operational state: виконане, blockers, unresolved effects і exact next action. Не друга незалежна execution database. |

Intake/resume звіряє IDs, versions, accepted criteria, current pointer та admission status. Наявність файлів без consumer не є integration. Markdown не перетворюється на неявну мову дозволів: executable поля задаються typed admission з посиланням на Owner-approved specification. Existing STANDARD/PROPOSED default зберігається; AUTONOMOUS_BLOCK вмикається лише для реально admitted scope.

## 4. Єдине durable execution сховище

Мінімально можливий `execution/STATE.json`: versioned atomic snapshot із action ledger усередині та immutable receipts під `evidence/`. Альтернатива transactional store допустима, але тоді JSON/STATUS є проєкціями, а не паралельними істинами. Вибір конкретного сховища потребує host/filesystem preflight.

State включає schema/controller version; goal/increment/admission IDs; generation; policy/grant/spec hashes; repository та immutable subject identity; phase/disposition; predicate dependencies; capability receipts; attempts/budgets; blockers; action ledger; worker ownership; exact next action або реальний resume trigger; notification/outcome identity.

Для file-based single-host реалізації потрібні writer lock, expected-generation check, temporary write, flush/fsync, atomic replacement та перевірена durability конкретного filesystem. Evidence спочатку записується durable за content hash, потім state посилається на нього. Orphan receipt не дає acceptance. Неперевірені semantics синхронізованого/network filesystem не оголошуються crash-safe.

Compaction не викидає unresolved intents, authority lineage, aggregate budgets, dependencies та потрібні receipts. Checkpoints робляться після переходів і перед effects, не лише наприкінці сесії, який може бути раптовим.

## 5. Predicates та різні рівні завершення

Predicate є записом: stable ID, admission-bound applicability, stage, dependencies, status, immutable subject/evidence refs та invalidation rule. Статуси:

```text
PENDING | RUNNING | PASS | FAIL | BLOCKED | INVALIDATED | NOT_APPLICABLE
```

NOT_APPLICABLE потребує accepted reason/authority. DEFERRED є scheduling attribute, не PASS. Відсутній required predicate, unknown status або порожній delivery inventory означають invalid state.

Окремо обчислюються `implementation_accepted`, `increment_complete`, `commit_eligible`, `publish_eligible`, `goal_complete`. Вони не взаємозамінні. Якщо Owner замовив goal, завершений increment не завершує весь goal.

Relevant mutation або новий blocking finding інвалідує affected acceptance і залежні predicates. Старий valid receipt зберігається як історія, але не переноситься автоматично на новий subject. Виправлення implementation потребує повторних applicable tests/review. Не всі metadata зміни скасовують усі checks; зміна acceptance/security/policy документа є relevant, навіть без runtime diff.

## 6. Lifecycle та transitions

```text
INTAKE → GROUND → CAPABILITY_PREFLIGHT → PLAN_AND_ADMIT
       → SELECT_RUNNABLE → EXECUTE → VERIFY → INDEPENDENT_REVIEW
                                               │
                         FIX / REJECT / FAIL ───┘
                                  ↓
                      RECORD_FINDING → REPAIR / REPLAN
                                  ↓
                      VERIFY → INDEPENDENT_REVIEW
                                  ↓ ACCEPT on exact subject
                      ACCEPT_IMPLEMENTATION → EVIDENCE
                                  ↓ when eligible and authorized
                      COMMIT → PUBLISH → RECONCILE
                                  ↓
                      CLOSE_INCREMENT → SELECT_SUCCESSOR
                                  ↓ only when goal criteria pass
                              FINAL_COMPLETE
```

Preflight до implementation не надає broad authority. Read-only discovery/probes працюють у початковому grant; install, paid call та mutations потребують власного scoped дозволу. Під час execution capability evidence перевіряється знову перед dependent action.

| Подія | Перехід і поведінка |
| --- | --- |
| Technical/test failure | EXECUTE/VERIFY → REPAIR або REPLAN; зберегти failure, інвалідувати affected acceptance, списати attempt budget. |
| Review FIX/REJECT | REVIEW → REPAIR → VERIFY → REVIEW; finding IDs лишаються blocking до valid remediation/rereview. |
| Один approach unavailable | SELECT_RUNNABLE/REPLAN; інший admitted approach або dependency-safe independent task. |
| Capability drift | Scoped PREFLIGHT; dependent dispatch не проходить на старому PASS. |
| Нове Owner decision | NEEDS_DECISION для affected scope; unsafe/unauthorized action зупиняється негайно. |
| Login/2FA/consent | NEEDS_OWNER_ACTION для affected scope; безпечний checkpoint та одна конкретна human action. |
| Очікування реальної job/provider із resumer | WAITING із durable trigger, deadline та budget. |
| Window/session end без живого resumer | SUSPENDED; зберегти resume capsule, не показувати «ще працює». |
| Recovery limit вичерпано | FAILED_BOUNDED або конкретний REQUEST_DECISION щодо зміни approved budget/constraint. |
| Invalid state/authority conflict/невідомий unsafe effect | INTEGRITY_HOLD; нові affected effects заборонено, bounded read-only reconciliation. |
| Owner cancellation | CANCELLING → CANCELLED; не dispatch нові effects, reconcile in-flight outcomes. |

EXECUTION_WINDOW_LIMIT є причиною suspension, не Owner decision. FINAL_COMPLETE є outcome, не blocker. Phase, cause, impact та notification не зливаються в один enum.

## 7. Capability preflight без global overblocking

Receipt capability фіксує exact required operation, provider/tool identity, executable/version, protocol/schema, exact model за потреби, auth scope/expiry, OS/network requirements, timestamp/TTL, relevant dependency fingerprint, probe result і evidence references. Credentials у receipt не зберігаються.

Послідовність: local discovery → version/import/contract check → minimal authorized non-destructive authenticated invocation → exact live proof, якщо цього вимагає predicate. Доступний CLI, `--version`, model inventory чи credential не доводять виконання конкретної операції.

Unavailable/stale capability блокує її consumers, не весь goal автоматично. Local implementation продовжується, коли admission не ставить remote proof її prerequisite. Scoped recheck потрібен після drift/resume/expiry і перед risk-sensitive use. Preflight не усуває race між probe та дією: реальна failure все одно проходить recovery policy.

Substitute model/runtime є технічним варіантом лише коли admission допускає еквівалентність і не змінюються semantics, privacy, cost та acceptance. Exact-runtime requirement не задовольняється іншою моделлю. Global install/upgrade, новий data egress або відключення security check не є автоматично дозволеним repair.

## 8. Blocker model та alternatives

`task → approach → predicate → increment → goal` не є коректною ієрархією. Approach є способом, predicate критерієм, а failure може одночасно зачепити кілька nodes. Authoritative blocker record містить origin attempt, множину affected_refs, dependency-derived impact, evidence та потрібну human dependency.

Власний приклад контракту:

```json
{
  "id": "B-17",
  "cause": "CAPABILITY_UNAVAILABLE",
  "origin_attempt_id": "A-3",
  "affected_refs": ["predicate:external-live-proof"],
  "impact": {
    "blocks_current_increment_exit": false,
    "blocks_goal_exit": true
  },
  "alternatives": {
    "status": "AVAILABLE",
    "admission_version": 4,
    "candidate_refs": ["approach:local-contract-check"],
    "evidence_refs": ["evidence/probe-3.json"]
  },
  "required_owner_decision": null,
  "required_owner_action": null,
  "next_reassessment": "on-capability-change"
}
```

Local contract check у прикладі просуває implementation, **не закриває exact external proof**. Impact визначається dependencies та accepted exit criteria, не впевненістю моделі. `blocked_scope` можна лишити derived compatibility summary, не джерелом authority.

| Alternatives status | Значення |
| --- | --- |
| UNASSESSED | Потрібне bounded дослідження. |
| AVAILABLE | Є executable safe admitted варіант. |
| EXHAUSTED_ADMITTED_SET | Немає viable alternative у зафіксованому reasonable candidate set за цієї admission/baseline. |
| BUDGET_LIMITED | Оцінка/спроби досягли ресурсної межі; це не доказ технічного вичерпання. |

`alternatives_exhausted=true` лише похідне від EXHAUSTED_ADMITTED_SET. Для кожного assessed approach потрібні hypothesis, semantic-equivalence argument, authority check, actual evidence та rejection reason. Candidate set обмежений задачею й budget; не треба доводити вичерпання всіх теоретичних рішень.

Матеріальне exhaustion за потреби проходить bounded competing-explanation review, якщо це admitted і reviewer доступний. Заборонені/unsafe approaches відсікаються без execution. Нове capability/baseline evidence може відкрити reassessment, але не обнуляє вже витрачені resources.

## 9. Bounded repair та meaningful progress

Дві матеріально однакові failed repairs зупиняють **цей approach**, не весь goal. Semantic attempt identity прив'язана до failure signature, hypothesis, subject і зміни, не до нового текстового label.

Admission задає per-approach та total attempt caps, wall-clock/compute/spend limits, no-progress threshold, review cycles і wait deadlines. Конкретні величини risk-matched, bounded та approved; універсальна цифра не підміняє рішення про budget.

Progress означає новий valid predicate result, усунений finding, перевірену competing hypothesis або зменшення named uncertainty. Більше тексту, перейменований approach чи heartbeat не є meaningful progress. Counters переживають session/context compaction, replans, review rejects та increment splits.

Transient retry використовує bounded backoff; hard protocol mismatch не лікується нескінченним retry. Після total limit виконується інша admitted незалежна робота, bounded wait/suspension або FAILED_BOUNDED. Owner потрібен лише для конкретної зміни constraint/budget/authority, а не тому, що агент мусить вигадати причину завершити loop.

## 10. Technical replan проти прихованого рішення Owner

Для material replan фіксується delta за accepted behavior, scope, hard constraints, data flow/retention, trust boundary, provider, interface, architecture, cost/quota, deployment, rollback та authority. Technical replan дозволений лише без material decision delta і в effective envelope.

Допустимий приклад: виправити command invocation або внутрішній алгоритм, зберігши contracts/resources. Недопустимий без authority: замінити exact model, надіслати private source новому provider, додати paid dependency, послабити auth або прибрати правильний failing test. Alternative availability не створює дозволу.

Worker не змінює controller, policy чи acceptance, щоб пройти gate. Proposal зберігається окремо; до належного approval діє стара boundary. Host restriction жорсткіша за admission означає effective deny.

## 11. No-return gate та поведінка щодо Owner

Routine technical failures і review FIX усуваються всередині блоку. Affected action freeze і notification Owner є різними рішеннями: security/authority stop негайний, а інша independent allowed робота може тривати. Не можна вимагати exhaustion перед припиненням unsafe operation.

Перед routine delivery response controller читає latest durable state, перевіряє integrity, reconciles unresolved effects і permission freshness, потім обчислює disposition:

```text
if state/authority unsafe: INTEGRITY_HOLD
elif cancelled: CANCELLED after bounded reconciliation
elif requested completion scope accepted: COMPLETE
elif urgent human safety/authority decision: REQUEST_DECISION
elif runnable admitted work and budget: CONTINUE
elif concrete human action is the useful next dependency: REQUEST_ACTION
elif concrete scope/cost/architecture decision is needed: REQUEST_DECISION
elif real wake trigger and deadline exist: WAIT
elif resumable but no live resumer exists: SUSPEND
else: FAILED_BOUNDED
```

CONTINUE містить concrete action ID. WAIT містить справжній task handle/trigger/deadline, а не обіцянку LLM. Invalid record із legacy `owner_response=false` не запускає blind loop. SUSPEND не називається Owner blocker.

Routine progress не є terminal response. Для cancellation, bounded technical failure, suspension без resumer чи integrity incident допустимий один правдивий operational status/outcome, окремий від REQUEST_DECISION і FINAL_COMPLETE. Інакше success-only/no-return policy робить невиконання невидимим. Passive UI status не перетворює Owner на message bus.

Для login/2FA/consent спочатку виконати дозволену підготовку та незалежну корисну роботу, потім попросити **одну конкретну дію у довіреному interface**. Не просити пароль/token/2FA code у чаті. Нові витрати потребують рішення; підтвердження вже схваленої payment action може бути permission dependency. Обхід consent не належить до alternatives.

Notification ID та acknowledgement зберігаються durable; resume не повторює той самий запит без нових обставин. Exactly-once notification залежить від dedup механізму host. Неоднозначна доставка не означає, що Owner повідомлення прочитав.

## 12. Local acceptance, external proof, commit та publish

Admission задає prerequisites окремо для implementation acceptance, commit, publish та goal completion. Split proof не дозволяє після failure знизити hard requirement до optional або NOT_APPLICABLE.

```text
I-implementation: local checks + immutable independent review = ACCEPT
P-exact-live-runtime: BLOCKED_EXTERNAL, still required for goal
commit_eligible: true only under admitted non-deploying checkpoint policy
publish_eligible: separately checked for named branch and downstream effects
goal_complete: false
```

Якщо live proof від початку hard prerequisite для commit/deployment, перенесення потребує належної authority. Blocked proof залишається видимим у delivery map. Increment може закритися лише за його погодженими exit criteria; закриття цього increment не робить goal complete.

Перед commit/push перевіряються staged path allowlist, secrets/publication safety, exact repository/branch/remote, tested subject, actual HEAD/ref та Git-hook/CI/deployment effects. Local commit, publication, merge і release мають різні permissions. Named normal push не означає дозволений deployment. Force push не є repair для ref conflict.

## 13. Crash-safe effects та resume

Перед кожною mutation durable записати action_id, logical intent, normalized target/arguments digest, admission/grant version, preconditions, stable idempotency key за підтримки target, replay policy, expected outcome та INTENT_RECORDED. Лише після цього dispatch; після відповіді durable receipt і state transition.

```text
INTENT_RECORDED → DISPATCHED → SUCCEEDED | FAILED | OUTCOME_UNKNOWN
OUTCOME_UNKNOWN → RECONCILE → SUCCEEDED | SAFE_TO_RETRY | HOLD
```

Один logical retry має той самий idempotency identity. Інша навмисна операція отримує інший intent ID навіть за однакових arguments. Змінені arguments під тим самим key означають conflict. Перед retry перевіряється retention/validity target idempotency contract.

Crash після effect і до receipt потребує authoritative query target state. Для commit звірити recorded subject/parent/intent; для push exact named remote ref та expected commit. Уже досягнутий результат записати без повторної mutation. Відхилений normal push вимагає reconciliation/replan, не force. Неспостережуваний uncertain non-idempotent effect переходить HOLD, а не blind retry. Загальна exactly-once гарантія для довільного external target не декларується.

Startup/resume: schema/controller compatibility → writer ownership → generation/grant revocation check → reconcile unresolved effects → actual Git/worktree/capability changes → dependency-based invalidation → counters recovery → STATUS regeneration → SELECT_RUNNABLE. Dirty чужі changes не стираються; оцінюється collision/ownership.

Один canonical writer на overlapping surfaces. Old worker epoch не dispatch нові effects; прострочений heartbeat сам собою не доводить смерть worker. Takeover потребує fencing/підтвердженої зупинки або надійного ownership механізму. Уже in-flight external operation може завершитися після timeout, її outcome все одно reconciles.

Уникнути self-hash циклу: reviewed implementation subject, resulting commit і checkpoint generation є різними identities. STATUS може бути derived post-commit receipt. Commit не повинен містити власний exact SHA.

## 14. Independent review, increments та long jobs

Review виконується в окремому read-only invocation/context щодо immutable subject. Receipt містить reviewer identity/type, subject hash, admission version, verdict, stable finding IDs та evidence. Інший provider/model не є самодостатньою гарантією незалежності.

FIX/REJECT відкриває blocking findings та інвалідує affected acceptance. Repair дає новий subject; targeted/regression checks і re-review прив'язуються до нього. Недоступного reviewer можна замінити тільки admitted substitute. Self-review не видається за independent; за відсутності допустимого reviewer predicate BLOCKED.

Один current increment означає canonical інтегратора/write boundary, не один величезний task чи сесію. Bounded non-overlapping tasks і parked verification nodes допустимі. Successor selection/admission автономне лише всередині approved goal envelope. Split зберігає hard criteria, unresolved proof, aggregate attempts/spend та parent outcome. Сторонній cleanup не є виправданням нескінченного відкладення справжнього blocker.

Long job потребує durable job ID, bounded timeout, heartbeat/lease, meaningful progress marker, cancellation path, authoritative status query і result receipt. Heartbeat без progress не подовжує виконання безмежно. Deadline/lease/budget failure веде до reconciliation/cancellation/failed state; output старої generation не інтегрується.

Після execution-window end продовжує **реальний перевірений supervisor/resumer**, якщо він налаштований. Без нього стан SUSPENDED із точним host/manual resume trigger, не фіктивна background activity. Наступний запуск читає durable goal/journal, не потребує перенесення переписки Owner.

## 15. Обов'язкова тестова матриця

Це **вимоги до майбутньої реалізації**, не tests, пройдені цим аудитом. Integration fixtures реально запускають subprocesses, виконують mutations у disposable environment та проходять той самий action/return path. Fake provider потрібен для fault injection, але не рахується external live proof.

| ID | Сценарій | Спостережуваний критерій |
| --- | --- | --- |
| T01 | Окремий read-only reviewer process повертає FIX для дефекту | Repair → tests → rereview на matching subject; без normal terminal response між ними. |
| T02 | Test subprocess реально падає; bounded patch виправляє fixture | Captured fail/pass receipts, invalidation та regression rerun. |
| T03 | Model-unavailable provider fixture | Scoped BLOCKED; allowed alternative/independent work без silent exact-model substitution. |
| T04 | Version drift після preflight | Dependent dispatch denied до scoped recheck/replan. |
| T05 | Protocol mismatch/malformed output | Structured failure, no fake success, bounded recovery. |
| T06 | Один approach blocked, інший admitted працює | Другий dispatch без Owner request; hard criteria збережено. |
| T07 | Login/2FA fixture і окремий authorized human live pilot | Одна конкретна action request, без secrets у logs, resume після consent; fixture не підміняє pilot. |
| T08 | Replan додає provider/security/architecture boundary | Affected action не виконується; scoped decision request, independent allowed work триває. |
| T09 | Kill після intent, після effect, перед/після receipt | Counter mutation не дублюється; uncertain outcome reconciled або HOLD. |
| T10 | Session/window/context restart між repair та review | Findings, budgets, authority і next action збережено; old ACCEPT не відновлюється. |
| T11 | Stale STATUS/head/phase | Journal/Git reconciliation регенерує projection, prose не стає truth. |
| T12 | Commit і push у temporary bare remote виконано, STATUS каже pending | Exact commit/ref знайдено; повторний implementation commit не створено. |
| T13 | Local ACCEPT та required external proof blocked | Stage-authorized checkpoint допустимий; goal_complete=false. |
| T14 | Live proof був hard prerequisite для commit, planner прибирає його | Acceptance weakening denied без new authority. |
| T15 | Repeated repairs змінюють labels, replan/split/resume | Semantic attempts та total/no-progress budgets збережено; loop bounded. |
| T16 | Job має heartbeat без progress | Progress deadline спрацьовує, bounded cancel/reconcile, не вічний RUNNING. |
| T17 | Два controllers і stale worker | CAS/ownership/fencing допускають лише чинного writer; stale receipt quarantined. |
| T18 | Envelope розширено або grant revoked перед dispatch | Effective deny; у strong mode немає shell/broker bypass. |
| T19 | Empty inventory, unknown event/status, null або broken journal | Structured invalid/INTEGRITY_HOLD, ніколи FINAL_COMPLETE чи spinning continue. |
| T20 | ACCEPT іншого subject; reviewer unavailable | Receipt rejected; review BLOCKED без fake independent self-review. |
| T21 | Owner cancels long operation | Нові effects не dispatch; in-flight reconciled; truthful CANCELLED. |
| T22 | Named push запускає unauthorized deployment | Publish denied; commit/publish/release permissions відокремлені. |
| T23 | Host хоче normal final response при pending required predicate | Реальний gate intercept і наступний action або valid wait/suspend. |
| T24 | Host закрито із resumer/без resumer | Реальний restart у першому випадку, SUSPENDED у другому. |
| T25 | Notification delivered, crash до acknowledgement | Dedup/uncertain-delivery policy без повторного Owner spam. |
| T26 | Exact external runtime/model authorized live smoke | Справжній operation/version/model receipt; unavailable означає BLOCKED, не fixture PASS. |

Test instrumentation: dispatch/message inventory, observable effect counter, subject/receipt hashes, temporary Git refs, state generations і aggregate budget ledger. Assertions перевіряють effects та transitions, не лише текст моделі.

## 16. Впровадження без overengineering

**Increment A: contract/reducer.** Один admission/schema mapping, predicate graph, dispositions, authority references, invalidation та negative/property tests. Це ще не end-to-end autonomy.

**Increment B: durable single-host execution.** Atomic journal, effect ledger, Git reconciliation, bounded budgets, action adapter, writer protection і crash/fault tests.

**Increment C: actual host integration.** Return gate, continuation, independent review receipts та preflight на конкретному supported host. Не переносити guarantees на інший host без перевірки.

**Increment D: bounded operational pilot.** Окремо authorized live T07/T26, restart/resumer та visibility checks. External-blocked proof зберігається; claims обмежені перевіреним scope.

Approved goal envelope може дозволяти автономні переходи між цими increments без Owner microtask routing. Нові security/data/cost/architecture boundaries однаково потребують authority. Passing unit suite не підміняє host acceptance; automatic push/resume вмикається лише після відповідних checks.

## 17. Відкриті питання перед реалізацією

1. Який конкретний host interface реально підтримує dispatch, final-response interception, cancellation та durable resumption?
2. Де trusted grants/controller policy захищені від worker writes, і потрібна cooperative чи strong-enforcement гарантія?
3. Які successors можна admit/select автономно та які stage-specific gates дозволяють commit/publish при blocked live proof?
4. Які approved risk-matched budgets, wait deadlines і notification rules?
5. Які reviewers/substitutes вважаються independent і які data-egress boundaries діють?
6. Які targets підтримують idempotency/query/fencing, а які uncertain outcomes потребують HOLD?
7. Яке durable storage/filesystem має перевірені lock/atomicity/crash semantics?
8. Як мігрувати existing goal/admission/PEF vocabularies без permission widening і retroactive acceptance?

Це admission inputs наступного implementation етапу, не підстава оголошувати цей аудит незавершеним.

## 18. Зв'язок із доказами

`REPORT.md` містить exact source paths, поточні findings, locally reproduced probes та межі перевірки. Ця модель є власною рекомендацією, не описом уже наявного коду. Generic idempotency/recovery/timeout принципи також зіставлено з первинними EXT1–EXT3 у REPORT; adoption конкретного framework не вимагається.
