# Greenfield model vs current AI OS

## 1. Baseline, порядок роботи та межі висновків

**Canonical source:** `SviatMM/ai-operating-system`  
**Actual `main` at baseline read:** `713928f1425374bac6dfdf13ba842d582ce5c37b`  
**Source tree:** `80ddc05d1556828fbd2f1ef0476f0ac6d4cff443`  
**Baseline observation date:** `2026-09-22`  
**Source commit date:** `2026-09-19T05:59:34Z`  
**Result branch base:** `SviatMM/NovastorySubagentDocs@128a1b72f51f031de0fc4ec7f4d46e30fbd2675f`.

Спочатку створено повний `GREENFIELD_ARCHITECTURE.md`; лише потім виконано перший read source main і читання AI OS. Freeze time: `2026-09-22T12:04:49.667548+00:00`. SHA-256 frozen architecture bytes: `0961b1e637bf3731bf24a1e31d4e29aed992dc9c9ed02532c1cedf15f2207d4d`. Початковий документ збережено без зміни. Це зафіксований порядок цього дослідження, не криптографічне доведення відсутності будь-яких попередніх концептуальних впливів. Інші результати Autonomy V2 агентів і NovaStory не читалися.

Прочитано 15 файлів, перелічених у §9, на exact source SHA. Також переглянуто root та scripts tree. Повну recursive tree відповідь інструмент обрізав; вона не використовується для категоричного доведення відсутності компонентів у всьому repository. Аналіз scripts — **source inspection**, не запуск у локальному середовищі Owner. Локальний host, встановлені runtime/model versions, production і приватні зовнішні receipts не перевірялися. Вислів «не підтверджено» нижче означає відсутність відповідного механізму в перевіреному execution path, а не доказ, що він ніде не існує.

Публікується власний synthesis. Private source text/code, credentials, персональні локальні шляхи, author emails і signed download URLs не включені. Source locators — лише repository-relative paths, immutable commit/blob identities та назви релевантних розділів.

## 2. Висновок

**Current AI OS не є описаним у проблемі обов'язковим ланцюжком Owner → ChatGPT → Codex → Owner.** Його активна модель уже доручає локальному CPTO end-to-end delivery; remote transfer є опцією. У поточному `AUTONOMOUS_BLOCK` repair, review remediation, evidence, commit та дозволений normal push повинні залишатися всередині роботи. Звичайний reject не повинен повертати координацію Owner. [S01, S03, S04, S06, S07]

Тому рекомендація — не заміна всіх правил новим набором промптів. **Зберегти сильний policy/knowledge layer AI OS, а execution semantics перенести в перевірюване durable kernel.** Current AI OS краще відповідає на «як має поводитися CPTO». ABDEK краще специфікує «який стан переживе crash, хто має право виконати дію і який факт не дасть помилково завершити run». Це перевага запропонованої специфікації, а не вже доведена перевага реалізації. [S03–S09, S11–S15; greenfield §§2–4, 12–15]

Наявний autonomous-block simulator сам обмежує свій статус policy simulation. Є і конкретний executable read-only runner із fingerprint checks. Неправильно називати весь AI OS «лише Markdown», так само неправильно вважати ці два компоненти загальним crash-safe orchestrator. [S11, S13]

## 3. Де current AI OS кращий

### 3.1 Менша операційна ціна для звичайної роботи

AI OS має компактний load order, routing skills за задачею, bounded increment та risk-matched checks. Він не змушує кожне дрібне виправлення проходити важку інфраструктуру; admission має пояснюваний exemption для тривіальної зворотної роботи. Greenfield додає store, scheduler, policy gateway, version handling і artifact provenance — більше системи, яку теж потрібно безпечно підтримувати. Для простих локальних задач existing workflow дешевший і зрозуміліший. [S02, S05, S08, S14]

**Зберегти:** малий context, lazy loading, risk-matched verification та slim execution profile. Навіть slim profile не повинен обходити authority або зовнішню idempotency.

### 3.2 Краще розроблена модель product knowledge

Current AI OS розділяє canonical product knowledge, shared operating rules, accepted decisions, unapproved ideas, derived local context і remote packet. CPTO має обов'язок оновлювати knowledge після перевіреної роботи. Greenfield здебільшого описує execution records та evidence, а не повний product knowledge graph і його редакційний lifecycle. [S01, S03, S04, S14]

**Зберегти:** product-local knowledge й дві derived context surfaces. Kernel events мають бути input для knowledge maintenance, а не заміною предметної документації. Рішення Owner не зводяться до machine enums.

### 3.3 Уже є конкретні артефакти й вузькі validators

Admission traceability validator перевіряє зв'язки hard requirements, prohibited substitutions, baseline evidence, superseded decisions і blocking acceptance. Це реальний executable компонент із чітко обмеженим призначенням. PEF runner також має конкретну вузьку command surface та binding checks. Greenfield поки є дизайном, тож не може заявляти вищу implementation maturity. [S05, S09, S13]

**Зберегти:** validators як модулі admission та consistency checks. Потрібна композиція зі state/effect verifier, не видалення корисних перевірок і не роздування schema validator до удаваного універсального acceptance engine.

### 3.4 Сильні обмеження ролей і deployment claims

Current AI OS явно не дозволяє remote mode вигадувати локальні результати, а dormant specialist lane потребує прямого Owner selection. Capability registry розрізняє documented, executable, E2E, unavailable і stale; конкретні E2E твердження обмежені конкретним workflow. Це корисна дисципліна, яку greenfield має успадкувати, а не розмивати автоматичною заміною будь-якого worker. [S03, S04, S12, S14]

**Зберегти:** manual-only lanes як deny-by-default authority policy; capability truth як окремий вимір від permission; чесне визнання реальних session limits.

## 4. Порівняння всіх 15 design concerns

| Concern | Current AI OS: що підтверджено | Greenfield: додаткова/інша семантика | Рішення |
|---|---|---|---|
| Authority | Owner boundary, product-local admission та bounded preauthorization у правилах. [S03–S05, S10] | Authenticated versioned grant, subset delegation, per-effect recheck, revocation, resource binding, budget reservations. | Зберегти policy; додати enforcement. |
| Durable state | Goal/map/current increment/STATUS/evidence та resume fields. [S01, S06, S07, S15] | Transactional event journal, action/effect IDs, outbox/inbox, leases, checkpoints; Markdown є projection. | Змінити storage authority, не видаляти readable views. |
| Work admission | Product-local record, hard constraints, accepted decisions, baseline, evidence plan; validator. [S05, S09] | Grant/capability/artifact bindings, adapter recovery class, durable-storage readiness, per-predicate readiness. | Розширити existing admission. |
| Planning | Повний outcome, dependency map, один active increment, autonomous reshaping. [S06, S07] | Versioned AND/OR graph, runnable frontier, proven alternatives, evidence invalidation. | Зберегти outcome/increment semantics; машинно планувати всередині дозволеної області. |
| Blockers | Local failure, review, Owner, external permission, window end і final categories. [S06, S11] | Окремі run/node/wait states, affected closure, uncertain remote result, capability/resource waits. | Замінити overloaded classifier, залишити знайомі dashboard labels. |
| Retry/replan | Дві однакові невдачі вимагають іншого пояснення; review remediation internal. [S06–S08] | Durable hypothesis/fingerprint history, costs, transient retry policy, circuit breaker, strategy switch. | Зберегти правило; зробити вимірюваним. |
| Completion | Required predicate checklist; agent self-report не acceptance. [S03, S04, S10, S11] | Evidence-derived predicates, exact tree/contract binding, delivery receipts, atomic final transition. | Замінити self-asserted runtime truth verifier-ом. |
| External permissions | Login/2FA/OS/physical needs відділені від local repair. [S06, S07, S10] | Secure deduplicated request, tenant/scope probe, expiry, authenticated callback, no secret transport. | Додати протокол, не лише категорію. |
| Owner escalation | Матеріальні intent/security/data/architecture/cost/scope/authority зміни. [S04, S06, S07] | Decision IDs, proposed grant delta, stale-answer protection, machine continuation після відповіді. | Зберегти матеріальність; автоматизувати transport і resume. |
| Recovery | Новий chat читає goal/status/increment і перевіряє repository. [S07, S15] | Idempotent reconciliation by operation ID, fenced controller, Git/provider observation vs projection. | Додати crash semantics. |
| Evidence | Commands, basis, pass/fail, findings, unknowns; bounded E2E receipts. [S08, S12, S13] | Artifact manifest, provenance/roles, binding/freshness checks, immutable attempts, safe publication. | Зберегти evidence discipline; посилити структуру та перевірку. |
| Workers | Bounded nonoverlapping work, canonical integrator, conditional review context, opt-in specialists. [S04, S07, S08] | Isolated worktrees, leased/fenced capability subsets, patch CAS, integrated-tree re-review. | Зберегти ownership; додати concurrency enforcement. |
| Session/window | Не зменшувати outcome; physical end записується з continuation. [S06, S07, S14] | Supervisor outside model sessions, durable triggers, checkpoint/restart without Owner relay. | Зберегти чесність; окремо реалізувати scheduler deployment. |
| Version/capability drift | Capability evidence стає stale після релевантних змін. [S12] | Execution lock, compatibility policy, pinned restore, transactional schema migration, model identity uncertainty. | Додати operational transition protocol. |
| Commit/push | Local commit і named normal push мають бути дозволені; force/production не implicit. [S04–S07, S10] | Exact repo/ref/old/new SHA, hooks/downstream scope, durable intent, remote receipt, crash reconciliation. | Зберегти permission distinction; уточнити effect semantics. |

## 5. Найбільші gaps та їхня перевірювана підстава

### G01 — Authority rules не дорівнюють reference monitor

Правила описують bounded authorization і нове Owner approval для чутливих змін. У перевіреному autonomy execution path немає демонстрації спільного gate, який отримує authenticated grant revision, canonical resource, reserved spend і fencing epoch перед кожною mutation. Simulator використовує named input actions, а не перевіряє identity зовнішнього account/ref чи issuer approval. [S04, S05, S11]

**Додати:** authority registry, scoped handles, denial precedence та gateway, який worker не може обійти shell/network. **Приймання:** stale approval, path escape, review request чи repository injection не створюють дозволений effect. Не оголошувати PEF fingerprint перевірки гарантією host-level sandbox для довільної програми. [S13; greenfield §§2–3]

### G02 — Немає доведеного протоколу uncertain side effect → reconcile

Resume record у templates містить branch/HEAD, predicates, checks і next action. Це корисно для відновлення контексту, але не містить transactional effect intent, stable business operation key, receipt state, lease epoch чи adapter-specific recovery contract. [S10, S15]

**Додати:** journal + outbox/inbox + typed `UNCERTAIN`, adapters із query/dedupe contract. **Приймання:** crash після external success, але до local receipt не призводить до повторного бізнес-ефекту. Для opaque API очікуваний safe результат — quarantine, не фальшиве гарантоване завершення. [Greenfield §§4, 14; W1, W2]

### G03 — Completion у simulator залежить від caller-provided assertions

Source inspection показує три конкретні межі policy simulator. Це **статичні контрприклади його використанню як runtime gate**, не результати запуску production:

1. Applicable predicate set надходить із input. Порожня множина за структурно наявного resume record не залишає incomplete predicate, тож може привести до final classification без виконаної роботи.
2. Preauthorized normal-push event змінює push predicate, але simulator не перевіряє Git remote або server receipt.
3. Resume fields перевіряються на наявність ключів; самі artifact bytes, authenticated grants, reviewer independence та відповідність active findings фінальному artifact не перевіряються як runtime facts. [S11]

Це не доказ, що активний Codex фактично приймає будь-який такий input. Це доказ, що pass simulator не доводить runtime completion. Його чесно заявлене simulator призначення потрібно зберегти.

**Змінити:** applicability бере immutable contract; status derives from verifier receipts. **Приймання:** empty required set для software-delivery contract, forged `push=true`, неіснуючий evidence і contradictory finding не завершують run. [Greenfield §12]

### G04 — STATUS має бути проєкцією, не конкурентним ledger

AI OS уже вимагає ground Git і не довіряти застарілому derived context. Тому проблема не в правилі «вірити STATUS більше за Git» — такого правила тут не приписується. Проблема — відсутність у inspected path детермінованого протоколу узгодження STATUS із Git/effects після torn update або кількох workers. [S03, S07, S15]

**Змінити:** verified source per fact, event sequence, CAS, `STATUS.md` як rebuildable projection. **Приймання:** stale `pending` після успішного push відновлюється з remote evidence без нового push; stale `done` не обходить contract. [Greenfield §§4, 13–14]

### G05 — Категорії блокування не виражають independent frontier

Current AI OS має dependency mapping, технічний replan і single active increment. Проте status template описує один blocker class та одну next action; simulator також видає одну категорію/наступну дію. Це не machine representation blocked dependency closure та кількох одночасно runnable alternatives. [S06, S07, S11, S15]

**Додати:** per-predicate waits, AND/OR alternatives, scheduling fairness і explicit rejected-candidate reasons. **Не робити:** перетворювати довільне перемикання на інший increment на implicit scope authority. Cross-increment admission мусить бути наперед дозволене outcome-level grant або схвалене окремо. [Greenfield §§3, 5–8]

### G06 — Session resume instructions не створюють supervisor

Current AI OS правильно визнає physical window limit і залишає resumable record. У skill continuation описується як подальше завантаження записів новою session; це не доказ автоматичного запуску нової session. [S06, S07, S14]

**Додати:** реальний supervisor, durable timer/event queue, controller leases і host lifecycle integration. **Приймання:** supervisor відновлює logical run без перенесення prompt Owner. Без supervisor показувати paused executor capability, а не «працюю у фоні». [Greenfield §§7, 11, 14–15]

### G07 — Drift awareness є, crash-compatible upgrade protocol не доведений

Registry відрізняє stale capability та наказує перевіряти його повторно. В inspected autonomy records немає повного execution lock із model/adapter/schema/policy identities чи state migration protocol. [S10, S12, S15]

**Додати:** pin/observation locks, compatibility decision matrix, alias uncertainty, restart-time version validation і transactional migrations. **Приймання:** installed ≠ pinned не запускає небезпечну дію; unavailable required model не замінюється без дозволу на privacy/cost/capability changes. [Greenfield §14]

### G08 — Review discipline потребує artifact/identity enforcement

Risk-matched quality loop рекомендує незалежний контекст за релевантності, а autonomous increment містить independent review predicate. В inspected path немає enforceable зв'язку reviewer identity ↔ exact integrated tree ↔ contract revision, що забороняє reuse old ACCEPT після нової mutation. [S08, S10, S11]

**Додати:** read-only reviewer capabilities, immutable artifact input, attestation, invalidation та новий reviewer після reviewer-authored patch. **Приймання:** три різні review FIX — три remediation tasks за прогресу; не три автоматичні Owner escalations. [Greenfield §§8–9, 12]

### G09 — Один writer як правило не є fencing

Наявні worker правила зменшують collisions і зберігають canonical integration. Вони не демонструють, як уже запущений старий worker перестає мати effect authority після lease loss, або як atomic base comparison захищає інтеграцію. [S04, S07]

**Додати:** isolated worktrees, gateway fence, per-resource concurrency, serialized integrator, dependency-impact revalidation. **Приймання:** overlapping patches не перезаписують чужі changes; merged tree отримує власні checks/review. [Greenfield §9]

## 6. Що залишити, прибрати, змінити й додати

### Залишити

Залишити CPTO як єдину відповідальну роль Owner-facing delivery; optional remote transfer; product-local admission і source grounding; повний outcome, delivery map та bounded increments; hard constraints/prohibited substitutions; local repair/review loops; якісні evidence вимоги; capability truth distinctions; manual-only specialist lanes; accepted-vs-proposed knowledge; existing validators і вузький read-only runner. [S01–S15]

### Прибрати з ролі authoritative execution mechanism

Не видаляти документацію. Прибрати можливість трактувати mutable checklist, worker prose, caller-selected predicates і simulator final classification як достатній runtime proof. Прибрати single blocker field як єдине scheduling representation. Прибрати ручний message relay як необхідну continuation mechanism. Не вважати retry counter, довжину session або прогресний текст ознакою завершення. [S07, S10, S11, S15; greenfield §§7–8, 12, 15]

### Змінити

`STATUS` та current-increment exit view генерувати з durable ledger і verified facts. Текстову authorization envelope переводити в normalized grant, але зберігати читабельне представлення. Admission validator лишити traceability validator-ом, а completion/effect checks винести в окремі компоненти. Existing autonomous mode зберегти як UX/policy label, не як твердження про реалізований scheduler. Review applicability визначати на admission, а не convenience-based наприкінці. [S05–S11, S15]

### Додати

Додати transactional run store; authenticated approval/permission events; effect gateway із revoke/fence checks; idempotency/reconciliation adapters; evidence manifests і verifier; durable scheduler/supervisor; per-node waits/alternatives; cost ledger; drift locks/migrations; sandbox/credential separation; fault injection і concurrency tests. Це proposal, не чинна авторизація змінювати AI OS, NovaStory чи production. [Greenfield §§2–16]

## 7. Рекомендована інтеграція без wholesale rewrite

**Етап A — observational control plane.** Підключити journal і artifact manifests поруч із наявним workflow; STATUS є derived view; жодної нової external authority. Порівнювати declared та observed predicate state. Entry: approved local storage/privacy. Exit: restart відновлює всі observations, а contradictions видимі.

**Етап B — gate локальних дій.** Sandbox workers, scoped local filesystem, single integrator, evidence/review bindings, authenticated grant. Existing validators працюють як checks admission. Exit: forbidden local writes, fake completion і stale review відхиляються.

**Етап C — контрольовані Git effects.** Окремі commit/push intents, exact refs, remote reconciliation, no-force enforcement, downstream-effect inventory. Exit: scenarios 15–17 і ref races відтворено executable fault tests.

**Етап D — supervisor і external adapters.** Session restarts, drift locks, durable waits, secure consent callback, adapter recovery classification. Exit: scenarios 5–14 та 19–20 відтворено під заявленими failure-domain assumptions.

**Етап E — ширші outcomes.** Auto-admission наступних increments тільки в попередньо затвердженому outcome/grant; budget і scope не ростуть непомітно. Exit: не потрібно Owner переносити повідомлення, а material decisions залишаються Owner. Greenfield не потребує одночасного ввімкнення всіх етапів.

Це послідовність архітектурного впровадження, не часовий графік і не оцінка автономності в годинах. Ніякий етап не реалізовано в source repository цим research commit.

## 8. Матеріальні Owner decisions перед впровадженням

**Для завершення цього дослідження додаткових рішень Owner не потрібно:** дозволи на чотири synthesized файли та named branch уже надані. Нижче — майбутні deployment choices, які цей документ не схвалює замість Owner.

| Decision | Чому матеріальне | Безпечний default до рішення |
|---|---|---|
| Control plane та evidence placement | Privacy, секрети, backups, failure domain, доступ інших machines/providers. | Локальне дозволене сховище; не обіцяти host-loss recovery без backup. |
| Scope delegated auto-admission | Чи run може переходити між increments без нового запиту; product priorities та boundaries. | Виконувати лише явно дозволену область. |
| Spend/usage/concurrency та search budget | Вартість workers/review/retries не повинна розширюватися мовчки. | Finite conservative caps і reserve для checkpoint/reconciliation. |
| Approved model/runtime substitutions | Capability, reproducibility, data location та independent-review policy. | Pin; не робити невизначену substitution. |
| Kernel trust/security boundary | Хто керує grants, gateway, credentials handles та updates; чому worker не може їх обійти. | Read-only shadow mode до прийнятої security architecture. |
| Allowed refs і downstream effects | Push може запускати workflow; publication не дорівнює дозволу на production. | Named non-force research/development refs; deploy/irreversible effects заборонені без конкретного approval. |

## 9. Source evidence index

Усі `Sxx` стосуються `SviatMM/ai-operating-system@713928f1425374bac6dfdf13ba842d582ce5c37b`. Це locators для власника джерела, не public копія його вмісту. Наведені blob SHA — ідентичності прочитаних файлів. Source висновки прив'язані до цих точок, а не до mutable `main` у майбутньому.

| ID | Repository-relative path | Relevant sections | Blob SHA |
|---|---|---|---|
| S01 | `README.md` | Role flow; Durable information model; Context loading | `d408a0cd39d5b827ce29f0fe64344b9c4cab363d` |
| S02 | `core/PRINCIPLES.md` | Shared defaults | `8e356c75ee7effb0cd55223ece7fdee1a65c55ad` |
| S03 | `core/AI_WORKFLOW.md` | Operating model; Goal and delivery lifecycle; Context maintenance | `2af3dd099c309b2870f21976b22e66ae0dda496e` |
| S04 | `profiles/cpto/PROFILE.md` | Decision and evidence discipline; Delivery model; Other workers | `3f3e4bbd40f3e9677493cda3b211e64cab0a6b19` |
| S05 | `core/WORK_ADMISSION_PROTOCOL.md` | Admission record; Capability-level traceability; Local CPTO execution control | `3e92c62eec5a7747cb94dcff8d261da67e7b5925` |
| S06 | `core/ADAPTIVE_DELIVERY.md` | Authority layers; AUTONOMOUS_BLOCK; Autonomous quality | `0db03722d599ca144c0cdb7cec959f17960f95d8` |
| S07 | `adapters/codex/.agents/skills/deliver-large-projects/SKILL.md` | AUTONOMOUS_BLOCK; Orchestrate workers; Continue across sessions | `9246734eef29648f91b36d87288999fa7387e127` |
| S08 | `adapters/codex/.agents/skills/deliver-large-projects/references/quality-loops.md` | Independent review; Repair; Stop/escalation; Evidence | `bbfcac461b8b127985ca19bd41491dcca773c5de` |
| S09 | `scripts/validate_admission_traceability.py` | `REQUIRED`, `validate`, CLI result | `996f0d865a5b22672a5805747fd0977d67dd8d6d` |
| S10 | `templates/product-repository/goals/_template/plans/CURRENT_INCREMENT.md` | Authorization; Completion gate; Exit record | `d6af34395398a7b20698942b761842f098bf82be` |
| S11 | `scripts/simulate_autonomous_block.py` | `evaluate`, `result`, predicate/event classification | `12bfed693b9444d8e39fe7d7a128dfc9bdcc89e7` |
| S12 | `core/CAPABILITY_TRUTH_REGISTRY.md` | State semantics; Bounded capability claims | `d5af04fe6fb94a9935672524af86535db3317683` |
| S13 | `scripts/run_pef_activation.py` | `validate`, `status_command`, `receipt`, `main` | `a11781ddf7d564c18738ce46f769054f8f88442e` |
| S14 | `adapters/codex/AGENTS.md` | Load order; Autonomous continuation | `3a281a31a0fa9e49e04f672245bdf520fad26f1f` |
| S15 | `templates/product-repository/goals/_template/STATUS.md` | Durable execution state; Blockers and contradictions | `4cc6ab05398bf5f35a022097f9a17c72b0e25e27` |

## 10. Після-design перевірка зовнішніх assumptions

Ці primary sources переглянуто **після** freeze greenfield та читання ключових AI OS файлів. Вони підтверджують окремі engineering constraints; з них не виводиться твердження про реалізацію AI OS або NovaStory.

**W1 — Temporal, Activity Definition / Idempotency, перевірено 2026-09-22.** Documentation розрізняє запис завершення activity та фактичні спроби виконання: lost completion report може викликати retry; business idempotency має забезпечувати система призначення. Звідси design consequence: durable workflow engine сам по собі не гарантує duplicate-free зовнішні ефекти. Reference: `https://docs.temporal.io/activity-definition`.

**W2 — Stripe API, Idempotent requests, перевірено 2026-09-22.** Однаковий key повторює збережений result; параметри перевіряються, а після pruning key може бути використаний як новий запит. Документована можливість pruning після щонайменше 24 годин означає, що наш adapter не може вважати будь-який давній key вічною dedupe гарантією. Це приклад constraint, не рекомендація виконувати платежі в цьому run. Reference: `https://docs.stripe.com/api/idempotent_requests`.

**W3 — Git, git-push manual, перевірено 2026-09-22.** Push оновлює refs і переносить objects; fast-forward constraints та refspec визначають, що змінюється. Atomic multi-ref update, якщо підтримано, не є загальною business transaction для довільних downstream effects. Design consequence: bind exact ref, verify remote state, не трактувати можливість push як дозволений deploy. Reference: `https://git-scm.com/docs/git-push`.
