# Scenario results — ABDEK

## 1. Метод і чесний статус перевірки

Пройдено всі **20 обов'язкових сценаріїв** як architecture-level state-transition walkthroughs frozen `GREENFIELD_ARCHITECTURE.md`. Це **не live tests, не fault-injection виконання production kernel і не запуск Codex/AI OS runtime**. Kernel у цьому результаті не реалізовано. Таблиця показує очікуваний safe outcome, а не вигаданий pass rate системи.

Для кожного сценарію визначено initial conditions, event/transition, продовження, durable evidence, Owner boundary, forbidden behavior та майбутній executable acceptance oracle. `SUCCEEDED` означає лише повний contracted outcome; `PARKED` чи node `WAITING` — чесна пауза, не success. Посилання `A§n` вказують на розділ greenfield architecture; `Gxx` — gap у `COMPARISON.md`.

Загальні assumptions: authenticated grant уже допускає названу локальну роботу; controller має чинний epoch; event store переживає заявлену failure domain; worker не може обійти gateway; deterministic verifier не є редагованим worker status. Scenario-specific exceptions наведено явно. Жоден сценарій не розширює permission для цього research run.

## 2. Матриця рішень

| ID | Mandatory scenario | Autonomous response | Human involvement | Safe resulting state |
|---|---|---|---|---|
| 01 | test fails once | Capture → diagnose → repair/retest → affected regressions | Немає, доки в межах grant | `ACTIVE → VERIFYING`; не success від одного rerun |
| 02 | same repair fails twice | Block identical hypothesis; diagnostic replan | Лише якщо новий path потребує матеріальної зміни | `ACTIVE`; за відсутності допустимого прогресу typed wait |
| 03 | independent review FIX | Findings → remediation → tests → independent re-review | Немає для bounded fix | `VERIFYING → ACTIVE → VERIFYING` |
| 04 | review FIX three times with different findings | Track findings and progress; continue under budget | Не через число rounds; лише реальна budget/scope/boundary delta | `ACTIVE/VERIFYING` або `WAITING` на конкретний decision |
| 05 | required model unavailable | Preserve requirement; approved equivalent only when policy permits | Лише для зміни required model/privacy/cost boundary | Affected node `WAITING(CAPABILITY)`; other frontier active |
| 06 | installed runtime version differs from pinned version | Quarantine affected execution; restore pin or approved compatibility path | Лише зміна boundary/approved pin policy | `RECONCILING`, then resume or wait |
| 07 | API changed | Quarantine adapter; sandbox contract diagnosis/repair | Лише semantic permission/data/cost change | Local work active; external node waiting |
| 08 | network unavailable | Persist backoff/circuit state; continue offline-independent work | Немає звичайного technical relay | `ACTIVE` or `PARKED(RETRY_TIMER)`; uncertain effects reconcile |
| 09 | login required | Deduplicated secure permission request; capability probe | Login in genuine service UI | Node `WAITING(EXTERNAL_PERMISSION)` |
| 10 | 2FA required | Challenge-bound request; no OTP in chat; verify capability | Physical 2FA action | Node waiting until machine-verifiable resolution |
| 11 | Owner architecture decision required | Persist bounded decision packet; continue independent work | Так: зміна architecture boundary | Node `WAITING(OWNER_DECISION)` |
| 12 | one implementation path blocked but two alternatives exist | Rank allowed alternatives and execute next viable strategy | Немає, якщо обидві в grant | `ACTIVE` via alternative path |
| 13 | current session ends halfway | Checkpoint/yield → new session → replay/reconcile | Немає business decision; executor needed if no supervisor | `RECONCILING → ACTIVE`, or explicit paused executor |
| 14 | process crashes after external side effect but before local receipt | Stable operation identity → external reconciliation | Лише unavoidable physical/provider resolution or new material decision | Verified receipt or `QUARANTINED`; never blind resend |
| 15 | commit succeeds, push fails | Retain commit; inspect remote; retry only permitted delivery | Немає для transient failure | `DELIVERING`; required push still unsatisfied |
| 16 | push succeeds, local process dies before STATUS update | Observe remote exact ref; reconstruct receipt; regenerate projection | Немає | `RECONCILING → FINALIZING` if other checks valid |
| 17 | stale STATUS contradicts Git | Resolve per-fact source of truth; rebuild STATUS | Немає for ordinary reconciliation | Recomputed state; uncertainty explicit |
| 18 | worker claims completion without evidence | Reject claim as proof; request actual verification | Немає | `UNPROVEN`, not `SUCCEEDED` |
| 19 | two workers modify overlapping files | Isolated patches; serialized integration; revalidate final tree | Немає unless actual product/security conflict | `ACTIVE → VERIFYING` |
| 20 | task can make progress in unrelated areas while one predicate is blocked | Schedule unaffected dependency frontier fairly | Only original legitimate request, once | Run stays `ACTIVE`; blocked node still waiting |

## 3. Детальні traces та acceptance oracles

### 01 — Test fails once

**Setup.** Artifact `T1`; required predicate `P-test` має `UNPROVEN`; baseline та relevant test command відомі. Grant допускає edit/tests. Failure не створює production side effect.

**Trace.** Executor записує command/environment/artifact/exit/output evidence `E1`; predicate стає `REFUTED`. Planner формує causal hypothesis `H1`. Справжній code defect виправляється в `T2`; controlled transient failure може отримати policy-approved rerun без code change. Targeted check і affected regressions дають нові receipts; changed artifact проходить required review. `E1` не видаляється, а пов'язується з repair.

**Durable result.** Збережено failed run, hypothesis, patch digest, new test receipts і прив'язку до `T2`. `ACTIVE → VERIFYING`; не всі outcome predicates обов'язково вже виконані.

**Owner / forbidden.** Owner не залучається. Заборонено прибрати assertion, приховати exception чи визнати initial failure неіснуючим тільки через один зелений rerun.

**Executable oracle.** Inject first failing test; перевірити, що repair/retest відбуваються без Owner request, обидва attempts лишаються в evidence, а false success неможливий до required regressions/review. Flaky policy може залишити predicate unproven. **References:** A§§8–9, 12; G03, G08.

### 02 — Same repair fails twice

**Setup.** Два attempts із тим самим normalized failure fingerprint та repair hypothesis `H1`, без нового discriminating evidence; budget ще є.

**Trace.** Kernel відхиляє третій blind repetition `H1`. Diagnostic task порівнює альтернативні причини, environment і test contract; planner версіонує strategy. Нове `H2` повинно мати перевірювану відмінність, не нову назву того самого patch. Independent ready work продовжується, поки triage працює.

**Durable result.** Attempt history, failure normalization, hypotheses, cost consumption та rationale strategy switch зберігаються між sessions. Можливий `ACTIVE` з новим path або typed wait після вичерпання допустимих diagnostic дій.

**Owner / forbidden.** Число два не є Owner escalation trigger. Owner потрібен лише для конкретної нової material delta, наприклад зміни architecture або збільшення вичерпаного approved budget. Заборонені counter reset після restart, blind retry loop і fake pass.

**Executable oracle.** Submit same semantic repair тричі в різних sessions; третя не dispatch-иться. Submit допустиме нове hypothesis — воно може бути admitted. Фальшива заміна тільки ID не обходить policy. **References:** A§8; G05, G06.

### 03 — Independent review FIX

**Setup.** Tests на `T1` зелені; незалежний reviewer має read-only artifact/context. Він знаходить bounded defect `F1` і повертає FIX.

**Trace.** `P-review` не satisfied. Finding зі severity, location, reproduction і contract mapping потрапляє в durable ledger. Implementer робить `T2`, виконує targeted/regression checks. Reviewer перевіряє `T2`; попередній ACCEPT/FIX не переноситься без binding. Якщо reviewer сам редагував `T2`, final reviewer мусить бути іншим contributor-independent actor.

**Durable result.** `VERIFYING → ACTIVE → VERIFYING`; `F1` закривається лише через новий verified evidence/attestation. Старий report лишається історією.

**Owner / forbidden.** Немає Owner relay для звичайного fix. Не дозволено вважати tests достатніми замість required review чи передати Owner саме review transcript як задачу створити новий prompt.

**Executable oracle.** Inject FIX, наступну mutation й forged old-tree ACCEPT; gate відхиляє старий attestation, приймає лише допустиме review нового artifact. **References:** A§§9, 12; G08.

### 04 — Review FIX three times with different findings

**Setup.** Rounds `R1/R2/R3` дають `F1/F2/F3` для `T1/T2/T3`; попередні valid findings поступово закриваються. Repair budget/concurrency limits задано до початку.

**Trace.** Kernel зберігає finding identity, artifact origin, resolved/open/regressed status та витрати. Нові contract-relevant findings породжують remediation незалежно від номера round. Нові вимоги поза contract не стають acceptance автоматично: scope ambiguity отримує proper decision/adjudication. За progress — далі repair; за oscillation — diagnostic replan/дозволений alternate reviewer; за budget exhaustion — typed budget wait.

**Durable result.** Progress оцінюється за закритими valid findings та стабільністю artifact, не «три рази review». `ACTIVE/VERIFYING` триває за достатньої влади й ресурсів.

**Owner / forbidden.** Owner не отримує третій FIX як автоматичний blocker. Вузьке питання допустиме, коли справді треба підняти cap або змінити approved scope/boundary. Заборонено нескінченний review loop без budget, review-shopping задля бажаного verdict і замовчування нових findings.

**Executable oracle.** Three distinct findings не створюють Owner request за прогресу; repeated regression активує diagnostic policy; exhausted cap не допускає четвертий billable round без дозволу. **References:** A§§3, 8–9; G08.

### 05 — Required model unavailable

**Setup.** Verification policy вимагає модель `M1` для конкретного reviewer або implementation step. Provider повертає unavailable. Наявність іншої моделі ще не означає дозволену equivalence.

**Trace.** Capability observation `M1` стає unavailable. Kernel перевіряє наперед затверджену substitution policy: capability, immutable identity або чесний alias статус, cost, privacy/data placement, independence. Якщо equivalent дозволений policy — rebind lock і потрібні checks. Якщо `M1` є exact non-substitutable requirement — node чекає. Детерміновані tests/builds та інша не залежна від `M1` робота продовжуються.

**Durable result.** Missing-model evidence, blocked dependencies, alternatives і next probe збережено. `WAITING(CAPABILITY)` не стирає required predicate.

**Owner / forbidden.** Рішення потрібне лише для зміни обов'язкової вимоги чи boundary; без зміни — machine wait. Заборонено тихо обрати дешевшу/слабшу модель, іншого data processor або скасувати review.

**Executable oracle.** Unapproved replacement відхиляється, approved equivalent допускається з revalidation, independent ready task виконується. **References:** A§14.4; G07.

### 06 — Installed runtime version differs from pinned version

**Setup.** Checkpoint очікує lock `L1/runtime V1`, але нова session запускається на `V2`. Journal schema може також відрізнятися.

**Trace.** До unsafe execution — `RECONCILING`. Kernel перевіряє identity/digests і підтримку schema. Пріоритет: cached pinned runtime або дозволене його отримання; далі лише preapproved compatibility path із contract tests, no-authority-expansion proof і новим lock revision. Migration store відбувається transactional, із rollback/backup. Unsupported newer schema не відкривається writer-ом старого runtime.

**Durable result.** Observed mismatch, compatibility evidence, migration ID та invalidated proofs. Після допустимого відновлення — той самий run та operation IDs; інакше affected nodes waiting.

**Owner / forbidden.** Owner потрібен лише для нової architecture/security/spending policy чи зміни pin requirement. Заборонено «версія майже та сама», destructive downgrade і запуск old uncertain operation з новою semantics.

**Executable oracle.** Start `V2` із `V1` lock: no mutation до перевірки; crash у migration не створює напівсхему; repeated recovery не розширює grants. **References:** A§§4, 14; G07.

### 07 — API changed

**Setup.** Adapter мав contract `C1`, нова відповідь порушує schema/semantics. Може існувати prior request із lost response.

**Trace.** Contract probe відрізняє known transient status від incompatible API. Залежні external effects quarantine; prior uncertain request reconcile за старою business identity. Worker досліджує дозволену API документацію/fixtures і змінює власний adapter у sandbox. Нові scopes, data flows, charging чи irreversible semantics не допускаються як «технічний fix». Після verification і lock revision відновлюється допустима частина frontier.

**Durable result.** Old/new contract identities, failed probe, patch/review/test evidence, affected predicates і unresolved operation IDs.

**Owner / forbidden.** Сумісний in-scope fix — автономно. Зміна authority/data/cost boundary — narrow Owner decision. Заборонено підганяти response parser так, щоб ігнорувати небезпечні невідомі поля/результати, або повторити opaque old effect під новим key.

**Executable oracle.** Introduce schema drift і changed charging semantics: перше допускає sandbox repair, друге не dispatch-иться без approval. **References:** A§§5, 14.4; G02, G07.

### 08 — Network unavailable

**Setup.** Network-dependent node не може дістатися service; інша частина graph має достатньо локальних inputs.

**Trace.** Якщо dispatch ще не відбувся — persisted bounded retry/backoff із jitter/circuit breaker. Якщо timeout стався після send — effect `UNCERTAIN`, спершу reconciliation. Offline-local implementation/tests продовжуються за dependencies. Якщо ready frontier порожній — `PARKED(RETRY_TIMER/CAPABILITY)` із durable wake condition.

**Durable result.** Network observations, attempt budget, next probe, possibly uncertain operation ledger; restart не скидає backoff/лічильники й не множить requests.

**Owner / forbidden.** Ordinary network outage не є Owner decision. Заборонено міняти approved data destination, вимикати security checks, відкривати інший account чи обходити мережеві обмеження для «автономності». Якщо відновлення реально потребує фізичного втручання — окремий evidence-backed human-action request.

**Executable oracle.** Blackhole network, restart worker і перевірити bounded rate; independent local task завершується; timeout-after-send не викликає blind new-key retry. **References:** A§§7–8, 14; G02, G05, G06.

### 09 — Login required

**Setup.** Operation дозволена, але approved service/account не має чинної authenticated capability. Це ще не дозвіл створити account або передати credentials у нове місце.

**Trace.** Kernel створює один permission request із service identity, tenant, minimal scopes, dependency і secure continuation. Owner входить у справжньому UI сервісу. Gateway отримує scoped handle; мінімальний probe перевіряє потрібну capability та account. Predicate розблоковується тільки після успішної machine observation.

**Durable result.** Request ID, expiry/status, permitted handle reference, probe receipt; секретні values відсутні. Інша локальна робота триває.

**Owner / forbidden.** Людина виконує лише login, не переносить prompt. Заборонено збирати пароль у chat/Markdown, публікувати token або зарахувати «я ввійшов» без validation як доступ.

**Executable oracle.** Duplicate session/replayed callback не створюють новий request; wrong tenant не відкриває node; correct scoped probe відновлює той самий run. **References:** A§10; G01, G06.

### 10 — 2FA required

**Setup.** Login stage потребує другого фактору/challenge. Challenge має account binding, expiry і continuation identity.

**Trace.** Permission request уточнює фізичний крок і dependency. OTP/push approval виконується у service-controlled surface. Kernel не зберігає factor secret або screenshot із кодом; отримує тільки verified auth result через approved adapter. Expired challenge не перетворюється на consent і потребує нового challenge generation без розширення authority.

**Durable result.** Challenge-generation metadata, request dedupe, expiry та minimal post-auth probe evidence. Affected node waiting; independent work active.

**Owner / forbidden.** Так, фізична 2FA дія. Немає product decision, якщо scopes не змінилися. Заборонено обійти 2FA, переслати код worker-у або визнати timeout підтвердженням.

**Executable oracle.** Expired/replayed/wrong-account approval не resume-ить node; valid challenge + probe resume-ить один раз. **References:** A§10; G01, G06.

### 11 — Owner architecture decision required

**Setup.** Єдина наразі перевірена стратегія змінює accepted architecture boundary, наприклад trust model або data placement. Grant цього не містить.

**Trace.** Kernel не допускає boundary-crossing action. Decision packet описує факти, alternatives, trade-offs, recommendation, точну grant/contract delta і безпечний стан без відповіді. Одночасно виконується незалежна авторизована робота. Owner answer автентифікується та version-bound; approval створює нову revision, denial виключає strategy або лишає dependency blocked.

**Durable result.** Decision ID, question scope, precondition digest, answer provenance, superseded/current revisions; impacted evidence переоцінюється.

**Owner / forbidden.** Це справжній матеріальний Owner decision. Заборонено позначити його «локальна оптимізація», прийняти silence як approval або застосувати запізніле рішення до зміненого outcome.

**Executable oracle.** Worker proposal не змінює grant; signed/authenticated matching decision робить лише запитану delta; replay або mismatched revision відхиляється. **References:** A§§3, 11; G01.

### 12 — One implementation path blocked, two alternatives exist

**Setup.** Predicate `P` має OR strategies `A/B/C`; `A` невиконуваний, `B/C` уже допустимі в grant і не послаблюють acceptance.

**Trace.** Evidence робить `A` failed/unavailable, не `P` неможливим. Planner зіставляє `B/C` за risk/cost/feasibility, обирає `B`, записує rationale й перевіряє актуальні capabilities. Якщо `B` падає, `C` лишається кандидатом за limits. Весь outcome не надсилається Owner як blocked лише через `A`.

**Durable result.** AND/OR graph revision, alternative identities, admissibility reasons, observations та new action lineage; final predicate і grant незмінні.

**Owner / forbidden.** Немає Owner involvement. Заборонено вигадати, що всі альтернативи вичерпані, або підмінити `P` легшим результатом. Якщо `B/C` виявляються authority-changing, їх не можна обрати потай.

**Executable oracle.** Disable `A`; scheduler dispatch-ить `B` чи `C`, not Owner request. Contract digest залишається незмінним. **References:** A§§6–8; G05.

### 13 — Current session ends halfway

**Setup.** Implementation незавершена, є dirty patch, findings та потенційно in-flight operations. Logical run має незалежний durable store.

**Trace.** Graceful end: припинити брати небезпечні неподільні actions, checkpoint patches/manifest/graph/requests/intents, release worker lease. Hard kill: новий controller fence-ить стару epoch, відновлює останній durable checkpoint і reconciles sent actions. Supervisor запускає дозволену нову session, яка читає records і exact repository basis, не transcript як source of truth.

**Durable result.** Той самий run ID, stable operation IDs, evidence/findings та continuation cursor. Перехід через `RECONCILING` до ready frontier; uncheckpointed reversible local computation може бути повторене, external effects — не сліпо.

**Owner / forbidden.** Немає business decision. Без фактичного supervisor результат чесно `PAUSED_NEEDS_EXECUTOR`; автоматичне майбутнє виконання не обіцяється. Заборонено обхід account/session spending limits або implicit push dirty state для context transfer.

**Executable oracle.** Kill/restart worker на edit/test/review/dispatch boundaries; granted operations і findings не губляться, stale worker fenced. Окремий negative test: без supervisor не заявляється auto-resume. **References:** A§§4, 14.1, 14.3; G02, G06.

### 14 — Crash after external side effect, before local receipt

**Setup.** Intent `O1`, semantic payload digest, grant revision і retry contract уже durable. Provider виконав effect, але процес помер до local receipt. Це ключовий неоднозначний failure window.

**Trace.** Restart бачить `DISPATCHED/UNCERTAIN`, не `READY`. Reconciler звертається до provider із тією самою business identity. За strong dedupe contract повторне звернення з тим самим key/payload у межах retention не створює другого effect; підтверджений result записується як receipt. За queryable provider потрібна authoritative correspondence, а stale negative read не дозволяє retry. За opaque non-idempotent provider дія мала бути inadmissible; якщо uncertainty вже існує — quarantine, без повторної відправки.

**Durable result.** Один business operation ID; збережено всі transport observations і outcome proof/uncertainty. Можливий verified recovery або `QUARANTINED`. **Універсальний автоматичний success для opaque API тут не заявляється.**

**Owner / forbidden.** Provider/physical reconciliation іноді потребує людини; якщо всі machine probes вичерпано, запит формулюється як точна необхідна дія/рішення. Owner не просять просто написати інший prompt. Заборонено новий idempotency key після timeout, повтор після key expiry без додаткового proof і трактування local lease як зовнішнього dedupe.

**Executable oracle.** Crash після provider commit до local receipt; strong adapter business counter залишається `1` при repeated recovery. Opaque adapter не dispatch-ить повтор, gate не виставляє success. Окремо перевірити retention expiry, payload mismatch, stale negative observation та split-brain workers. **References:** A§§1.2, 4.2, 14.1–14.2; G02; W1–W2 у comparison.

### 15 — Commit succeeds, push fails

**Setup.** Verified tree `T` створив local commit `C`; grant допускає exact normal push; push є required delivery predicate. Помилка могла статися до send або після remote effect.

**Trace.** Local commit receipt і `C` зберігаються. Delivery adapter спочатку читає approved remote/ref. Якщо remote already `C` — переходить до scenario 16. Якщо достовірно лишився approved old SHA, authority чинна і non-force preconditions збереглися — дозволений bounded retry того самого intent. Divergent ref запускає reconciliation, а не force. Потрібний rebase/patch integration інвалідує відповідне review/evidence.

**Durable result.** `P-commit=SATISFIED`; `P-push` pending/uncertain до remote proof. Run `DELIVERING`, не `SUCCEEDED`. Не створюється новий commit тільки через failure transport.

**Owner / forbidden.** Немає Owner relay для звичайного network failure. Нові refs, credentials, cost, force чи deployment потребують конкретної влади. Заборонено повторне «commit того самого» і звіт «все завершено» за required failed push.

**Executable oracle.** Fail push перед/після remote update; commit identity стабільна, remote changes не губляться, only authorized exact ref touched. **References:** A§13; G02–G04.

### 16 — Push succeeds, process dies before STATUS update

**Setup.** Remote approved ref уже дорівнює `C`; journal має persisted push intent, а readable STATUS ще показує pending. Final receipt може бути відсутній.

**Trace.** Recovery перевіряє remote identity та exact ref/commit. Evidence reconciler записує observed-success receipt для того самого operation ID й відбудовує projection. Якщо всі інші acceptance predicates чинні для `C`/tree — `FINALIZING → SUCCEEDED`. Якщо remote вже descendant, перевіряється точний delivery contract: history inclusion не рівнозначне reviewed current tip.

**Durable result.** Один logical push receipt і один terminal receipt, STATUS відповідає journal. Нова відправка push не потрібна для already-confirmed exact outcome.

**Owner / forbidden.** Owner не залучається. Заборонено сліпо пушити ще раз через stale STATUS чи автоматично прийняти чужий newer tip як reviewed artifact.

**Executable oracle.** Kill після remote update, до local status write; restart reconstructs receipt без повторного ref mutation. Verify separate case remote descendant/foreign commit — gate не стирає uncertainty. **References:** A§§13–14; G02–G04.

### 17 — Stale STATUS contradicts Git

**Setup.** Варіант A: STATUS каже done, remote не підтверджує contracted commit. Варіант B: STATUS каже pending, remote уже підтверджує commit. Варіант C: wrong worktree/branch робить локальне порівняння хибним.

**Trace.** Kernel визначає repository/worktree/ref identities; reads Git objects/refs, journal intents/receipts, grant та evidence bindings. STATUS не є proof. A: delivery стає unproven/uncertain і досліджується drift/зовнішня зміна; B: reconstruct receipt; C: correct scope before observation. Сама наявність Git commit не доводить тестів, review чи дозволу.

**Durable result.** Conflict event зі старими assertions, authoritative observations і причиною resolution; відновлена projection. Жодного silent overwrite історії.

**Owner / forbidden.** Ordinary contradiction розв'язує система. Якщо факти виявили material unauthorized production change чи unresolved ownership conflict, ескалується саме це. Заборонено `reset --hard` чужих changes, force push чи cherry-pick unknown work заради красивого STATUS.

**Executable oracle.** Seed contradictory projections і wrong worktree identifiers; final state derives from authenticated per-fact truth, not last file edit timestamp. **References:** A§§4, 13–14; G04.

### 18 — Worker claims completion without evidence

**Setup.** Worker надсилає `done`/green checkboxes, але required command receipts, artifact bytes або review attestation відсутні, недоступні чи не відповідають поточному tree.

**Trace.** Claim записується як untrusted report; verifier не підвищує predicate. Scheduler запускає дозволені actual checks або просить bounded evidence production. Fake path, hash mismatch, wrong producer, unrelated screenshot і stale review відхиляються. За неповного доступу перевірка waiting, не pass.

**Durable result.** `UNPROVEN/STALE`, evidence rejection reason і next verification action. Worker не має endpoint, який сам виставляє terminal state.

**Owner / forbidden.** Owner не переносить запит між агентами. Заборонено довірити raw boolean, довільній applicable set або красивому звіту роль completion evidence. Security incident у executor розглядається окремо.

**Executable oracle.** Submit forged success із empty required predicates, nonexistent evidence URI і valid hash of irrelevant bytes; gate відхиляє всі як outcome proof. Authentic relevant receipts допускають перехід. **References:** A§§2, 9, 12; G01, G03, G08.

### 19 — Two workers modify overlapping files

**Setup.** Workers `W1/W2` стартують із base `B` і змінюють той самий файл. Вони не мають shared mutable worktree та прямого remote-write credential. Integrator має один fenced lease.

**Trace.** Workers віддають immutable patches `D1/D2` зі scope/base/epoch. Integrator застосовує `D1`, змінює integration base через CAS; `D2` на stale base не може безперевірково перезаписати `D1`. Controlled merge/rebase у sandbox розв'язує text/semantic conflicts. Combined artifact `T` отримує нові targeted/dependency regression tests і independent review.

**Durable result.** Patch provenance, integration order, base comparisons, conflict-resolution delta, final tree й verified receipts. Expired `W1` не може пізніше застосувати прямий effect; stale patch можна тільки переоцінити як нову пропозицію.

**Owner / forbidden.** Не кожен Git conflict є Owner decision. Потрібен Owner тільки за справжньої super-scope product/security/architecture ambiguity. Заборонено last-writer-wins, одночасний shared index, чужий force push і успадкування isolated branch ACCEPT на merged tree.

**Executable oracle.** Race overlapping patches і stale controller dispatch; один integrator перемагає CAS, інший перебазовується/повертається на review; no lost update, no duplicate side effect. **References:** A§9; G09.

### 20 — Unrelated work can progress while one predicate is blocked

**Setup.** Outcome має independent branches `P1` і `P2`; `P1` чекає login/decision/API, але `P2` має ready inputs, authority і budget. Обидві лежать у дозволеній admission області.

**Trace.** Blocker прив'язаний до `P1`; scheduler обчислює лише його transitive dependency closure. `P2` виконується з fairness, tests/review/evidence оновлюються. Для `P1` існує один deduplicated request/probe. Global park допустимий лише коли немає ready allowed actions або корисної допустимої діагностики.

**Durable result.** Run лишається `ACTIVE` з partial predicate progress; `P1` waiting. Фінал не success, поки required `P1` не verified або Owner не змінив contract. Якщо незалежна робота насправді в іншому неадміченому increment, спершу потрібна допустима auto-admission, не scope bypass.

**Owner / forbidden.** Лише початкове матеріальне питання/фізична дія, якщо вони справді потрібні; технічні прогресні повідомлення не стають новими запитами. Заборонено global blocker через одну dependency, видалення заблокованого predicate з acceptance і несанкціоноване розширення scope.

**Executable oracle.** Block `P1`, assert `P2` dispatches і завершує свої checks; Owner request count не росте при polls/restarts; whole run не final до `P1`. **References:** A§§5–8, 11, 15; G05.

## 4. Cross-scenario safety assertions

У майбутньому executable harness кожен trace повторюється з restart перед/після кожної durable boundary та з duplicate event delivery. Обов'язкові наскрізні assertions:

```text
no dispatch outside current authenticated grant
no acceptance from untrusted worker assertion
no silent deletion of required completion predicates
no new business operation key merely because of timeout or session change
no blind retry of uncertain opaque effects
no reuse of review for a different artifact without valid re-verification
no unfenced overlapping canonical writes
no global park while an independent allowed action is ready
no repeated human request for the same unresolved dependency generation
no claimed automatic continuation without a running supervisor
no claimed success when required delivery/live proof is missing
no secret or personal-path propagation into public evidence
```

Dedupe гарантія notification outbox сама по собі не гарантує exactly-once доставки довільним email/chat provider. Для «один фінальний результат» потрібен idempotent update одного status/message resource або receiver-side dedupe. Якщо transport цього не забезпечує, logical final receipt залишається єдиним, а possible duplicate notification delivery — явно обмежений transport guarantee. Не плутати це з повторенням бізнес-ефекту.

## 5. Невирішені реалізаційні питання, які не приховуються

Архітектура задає поведінку для всіх 20 сценаріїв. До runtime acceptance ще потрібно реалізувати та перевірити storage durability, actual sandbox/fencing, semantic grant normalization, adapter idempotency/reconciliation, authentic measurement/review identities, scheduler fairness, secure callback та model/runtime migration. Для opaque external effects без query/dedupe автоматичний success не обіцяється; без supervisor автоматичне session continuation не обіцяється.

Отже, результат цієї перевірки — **20 специфікованих safe transition paths із чіткими conditional waits**, а не «20 production tests passed». Жодна кількість годин роботи, повідомлень або retries не використана як визначення автономності.
