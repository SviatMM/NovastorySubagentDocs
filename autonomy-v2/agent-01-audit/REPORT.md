# Autonomy V2: незалежний архітектурний аудит

## 1. Вердикт та межі перевірки

**Вердикт: `revise`.** Звичайний technical failure або review=FIX не повинен робити Owner диспетчером між агентами. Проте додаткові поля в Markdown/JSON і заборона повертатися не створюють виконавця, безпечного відновлення чи достовірного завершення. Рекомендовано невеликий контролер переходів, durable execution journal та реальне підключення action/return gates до підтримуваного host.

| Поле | Значення |
| --- | --- |
| Task | `autonomy-v2-agent-01-audit` |
| Дата | `2026-09-22` |
| Canonical source, READ ONLY | `SviatMM/ai-operating-system` |
| Перевірена гілка | `main` |
| **Exact audited source SHA** | **`713928f1425374bac6dfdf13ba842d582ce5c37b`** |
| Source root tree | `80ddc05d1556828fbd2f1ef0476f0ac6d4cff443` |
| Result repository | `SviatMM/NovastorySubagentDocs` |
| Result branch | `research/autonomy-v2-agent-01-audit` |
| Result base commit | `128a1b72f51f031de0fc4ec7f4d46e30fbd2675f` |
| Дозволена область результату | `autonomy-v2/agent-01-audit/` |

Це **candidate analysis, не реалізація V2, не її приймання та не зміна AI OS**. NovaStory repository не читався і не змінювався. Архітектура NovaStory Agent не проєктувалася. Публічний результат містить власні висновки, repository-relative paths та необхідні ідентифікатори контрактів, а не копії приватної документації чи коду.

## 2. Метод та сила доказів

Source main отримано через GitHub і повторно звірено перед публікацією; весь аудит прив'язаний до exact SHA вище. Прочитано root/scripts/extensions-tests trees, активні workflow/admission/adapter/skill, відповідний shared goal, product templates та основні validators. Обрізаний загальний tree response доповнено читанням потрібних піддерев. Це не аудит кожного архівного файла або всього PEF.

Два файли матеріалізовано в ізольованому середовищі; перед виконанням їхні байти перевірено за Git blob SHA:

| Файл | Git blob SHA | Перевірка |
| --- | --- | --- |
| `scripts/simulate_autonomous_block.py` | `12bfed693b9444d8e39fe7d7a128dfc9bdcc89e7` | exact match |
| `extensions/tests/test_simulate_autonomous_block.py` | `59dbdf4bb2121a6730f386050372941190fa4522` | exact match |

**Власноруч виконано:** на Python `3.13.5` штатні п'ять tests симулятора, **5/5 PASS**, та **14 синтетичних негативних/граничних проб** його `evaluate`. Вони не виконують реальних Git/network mutations. Наведені негативні результати відтворено; це не 14 acceptance PASS для V2.

**Не виконано:** повний 71-test extension suite, локальний Codex, справжній model/runtime live proof, OS consent, незалежний review реальної implementation, crash/restart інтеграція. Згадки джерела про 71 PASS та незалежний ACCEPT є історичними claims, а не моїм повторним тестом. Відсутність runtime enforcement нижче означає, що його не підтверджено в перевіреному repository execution path; невідомий зовнішній wrapper не оголошується неіснуючим.

## 3. Current-state map

Усі `[Sxx]` означають source files за pinned SHA розділу 1. Code line numbers також стосуються цього snapshot. Розрізняються чотири рівні: documented policy; executable validator при явному виклику; обов'язково підключений action/host gate; fault-tested end-to-end control.

| ID та місце | Що реально існує | Межа гарантії |
| --- | --- | --- |
| **S01** `core/ADAPTIVE_DELIVERY.md`, `core/AI_WORKFLOW.md` | AUTONOMOUS_BLOCK, bounded increments, completion predicates, repair/re-review, Owner/external/window semantics; правило двох матеріально однакових repairs | Markdown policy, не execution loop |
| **S02** `core/WORK_ADMISSION_PROTOCOL.md`, `profiles/cpto/PROFILE.md` | Admission, hard constraints, evidence hierarchy, bounded authorization та technical replan | Політика повноважень не означає tool-level enforcement |
| **S03** `adapters/codex/AGENTS.md`, `adapters/codex/.agents/skills/deliver-large-projects/SKILL.md` | Завантаження goal/admission, продовження, checkpoints, no-intermediate-response | Немає показаного коду final-response interception або dispatch наступного turn |
| **S04** `adapters/codex/.agents/skills/deliver-large-projects/references/quality-loops.md` | Risk-matched tests, separate review context, bounded repair; generic stop/escalation при відсутності нового evidence | Потрібен однозначний precedence із no-return правилом |
| **S05** `templates/product-repository/goals/_template/plans/ADMISSION_RECORD.json`, `plans/CURRENT_INCREMENT.md` того самого template goal | Execution mode та envelope; JSON default STANDARD/PROPOSED | Безпечний default, не дефект; template не активує існуючий product-local goal |
| **S06** `scripts/simulate_autonomous_block.py` | Виконуваний policy simulator: часткова структурна перевірка та обчислення owner_response/predicates | Не запускає implementation/review/Git; не зберігає state; не керує host |
| **S07** `scripts/validate_admission_traceability.py` | Required fields, hard requirement refs, prohibited substitutions, evidence baseline, superseded decisions, blockers при acceptance | Не перевіряє envelope як дозвіл на конкретну дію |
| **S08** `scripts/validate_context_freshness.py` | Receipt/source hashes та Git changes для selected derived context | Не reconciler operational STATUS, Git effects та всіх claims AI OS |
| **S09** `scripts/validate_pef_state_gates.py` | Read-only S3c evaluator: Owner provenance, goal/slice/scope/resolution matching, evidence statuses і transitions | Корисна executable основа, але не general autonomous runtime gate |
| **S10** `scripts/run_pef_activation.py` | Реальний вузький fingerprint-bound read-only Git-status runner; execute flag, timeout/termination, receipt | Не general writer, worker dispatcher чи consumer автономного execution record |
| **S11** `scripts/run-extension-validator.sh`, `scripts/validate_extensions.py` | Pinned imports/versions, repair isolated environment; schema/manifest/frontmatter validation | Вузький capability preflight, не готовність довільного model/runtime/API |
| **S12** `core/CAPABILITY_TRUTH_REGISTRY.md` | DOCUMENTED, LOCALLY_INVOCABLE, E2E_VERIFIED, UNAVAILABLE, STALE/UNKNOWN | Markdown registry; історичні receipts не доводять актуальну доступність на іншому host |
| **S13** `goals/2026-09-19-autonomous-block-execution/` | Реальні GOAL, STATUS, DELIVERY_MAP, CURRENT_INCREMENT, ADMISSION_RECORD, validation evidence | Документаційно-симуляційний shared increment, не proof product adoption |
| **S14** `extensions/tests/test_simulate_autonomous_block.py` | П'ять subprocess tests evaluator | Happy path вручну виставляє flags; resume test не відновлює живий executor |
| **S15** `scripts/init-product-ai-workspace.sh` | Копіює відсутні templates, зберігає існуючі файли | Не мігрує старі goals і не встановлює їм AUTONOMOUS_BLOCK |

**Висновок current state:** є змістовна policy та кілька справжніх validators, а також вузький read-only runner. Загальна end-to-end автономність не підтверджена. `valid=true`, exit code 0 чи наявність API-методу не дорівнюють дозволеному side effect або завершеному goal.

## 4. Відтворені граничні сценарії

Спільна база: AUTONOMOUS_BLOCK; дев'ять predicates без live_proof; resume з усіма required keys. «Усі true» означає, що ці дев'ять flags true, але `last_checks=[]`, без перевірених receipts. Кожна проба змінює вказану умову. P09/P10 використовують той самий record між двома викликами, без зовнішнього repair.

| ID | Вхід / зміна | Actual result S06 | Що доведено |
| --- | --- | --- | --- |
| P01 | `applicable_predicates=[]`, `predicates={}` | FINAL_COMPLETE | Порожній completion inventory проходить |
| P02 | Усі flags true, перевірок немає | FINAL_COMPLETE | Self-attestation достатня для симулятора |
| P03 | Усі true та active blocking finding FIX | FINAL_COMPLETE | Findings не блокують acceptance |
| P04 | Лише push=false; normal_push; generic preauthorization | FINAL_COMPLETE, push completed | Подія сама виставляє push=true без Git |
| P05 | Невалідний HEAD, resume lists суперечать flags | FINAL_COMPLETE | Немає semantic reconciliation |
| P06 | Required resume keys існують, кожне значення null | FINAL_COMPLETE | Перевіряється наявність ключів, не придатність resume |
| P07 | Невідомий event `review_fix`, усі true | FINAL_COMPLETE | Unknown event не відхиляється |
| P08 | Усі flags false | next: `complete predicate: commit` | Alphabetical order замість dependencies |
| P09 | True → technical_failure → continue без repair | Internal failure, потім FINAL_COMPLETE | Failure не інвалідує old acceptance |
| P10 | True → review_reject → continue без rereview | REVIEW_FINDING, потім FINAL_COMPLETE | Reject не є durable blocking transition |
| P11 | Вісім однакових technical_failure inputs | Вісім однакових repair outputs | Немає накопичуваного attempt/stall budget |
| P12 | Named-branch authorization phrase замість generic phrase | OWNER_BLOCKER | String matching не є typed admission integration |
| P13 | Top-level JSON null у evaluate | AttributeError | Немає structured invalid-state disposition |
| P14 | `applicable_predicates=null` | TypeError | Inventory type не захищений |

Це не свідчення реального production exploit: S06 прямо є simulator. Проби показують, чому його не можна без змін підняти до trusted acceptance/action gate. P12 не доводить broken production adapter: сам adapter між admission і simulator не продемонстровано.

В authorized checkout pinned SHA штатний запуск: `python -m unittest extensions.tests.test_simulate_autonomous_block -v`. Для повторення проб достатньо викликати `evaluate` з описаним record та змінами. Повний private source не включено до public result.

## 5. Findings

### F01 | HIGH | No-return policy не є host enforcement

**Місце / доказ:** S03; S06 `evaluate`, `main`, module description; S10. Simulator читає один input, повертає JSON і завершується. Markdown не демонструє interception final response, виконання next action або restart host.

**Наслідок:** Codex може повернутися після FIX чи failure, хоча правило написано правильно. `owner_response=false` нічого не запускає.

**Виправлення:** supported host contract та controller, який реально споживає gate result. Без continuation/interception hooks гарантія advisory, не enforced. Додати host integration test; не створювати новий product agent runtime.

### F02 | HIGH | Самодеклароване завершення та некаузальні переходи

**Місце / доказ:** S06:41–59,72–89; P01–P03/P07/P09/P10. Caller контролює inventory і flags; receipts/findings не перевіряються; failure/reject не інвалідують predicates.

**Наслідок:** false completion, перенесення старого review на новий diff, вилучення неприємного required predicate або трактування невідомого review event як continue.

**Виправлення:** admission-bound непорожній inventory; typed events; dependency graph та invalidation closure; immutable reviewed/tested subject; перевірені receipts. Розділити implementation, increment і goal completion. Malformed/unknown input відхиляти, не продовжувати наосліп.

### F03 | HIGH | Authorization не зв'язана з action dispatch

**Місце / доказ:** S07 `REQUIRED`/`validate`; S06:69–73; S05 nested envelope проти S13 top-level action fields; P04/P12. S09 уже має scoped authorization matching, але він не підключений до S06.

**Наслідок:** eventual wrapper може прийняти broad self-reported permission або зайво запросити Owner. Technical replan може приховати зміну provider, privacy, cost чи architecture.

**Виправлення:** один versioned authorization contract: trusted Owner provenance, grant ID/hash, repository/branch/path/action/data/cost boundaries; agent може лише звужувати effective authority. Перевіряти перед side effect та після resume; material decision delta потребує нової authority. Hash не доводить схвалення сам собою. Strong enforcement потребує boundary, яку worker не обходить через unrestricted shell чи редагування controller/grant; інакше це cooperative discipline.

### F04 | HIGH | Durable notes не забезпечують crash-safe side effects

**Місце / доказ:** S06:47–59,66–67; resume поля S03/S05/S13; P05/P06. Немає autonomous action intent/receipt ledger, generation/CAS, idempotency identity чи writer fencing.

**Наслідок:** effect виконано, відповідь втрачено, resume повторює mutation; старий worker паралельно пише; checkpoint суперечить зовнішньому стану.

**Виправлення:** durable intent до dispatch, stable logical action identity, результат OUTCOME_UNKNOWN при неоднозначному timeout, reconciliation перед retry, single writer/fencing. Неспостережуваний non-idempotent effect не можна повторювати наосліп. Загальна exactly-once гарантія не випливає з checkpoint [EXT1].

### F05 | HIGH | No-return може зациклити execution або приховати зависання

**Місце / доказ:** S01/S04 repair policies; S06:57–59,74–88; P11/P13/P14. Текст має дві однакові repairs, але нема persisted total budget, no-progress deadline чи technical-exhaustion outcome. V2 вимагає continue при false навіть без runnable work.

**Наслідок:** нескінченне перейменування approaches, витрати без прогресу, silent stall або цикл на invalid record.

**Виправлення:** typed dispositions CONTINUE, WAIT, SUSPEND, REQUEST_DECISION, REQUEST_ACTION, COMPLETE, FAILED_BOUNDED, CANCELLED, INTEGRITY_HOLD. Attempts/time/spend/no-progress budgets переживають sessions/replans/splits. WAIT потребує реального wake mechanism і deadline; без нього SUSPEND, не «ще працює». Технічне вичерпання не потребує вигаданого Owner decision [EXT2].

### F06 | HIGH | У AI OS є конкретний stale operational state про себе

**Місце / доказ:** S13 `STATUS.md`, `plans/CURRENT_INCREMENT.md`, `evidence/VALIDATION.md`. Commit/push залишаються pending, хоча remote `refs/heads/codex/autonomous-block-profile-2026-09-17` уже дорівнює audited SHA, як і main. Current increment має review checkboxes true, але completed list їх не містить. Validation evidence одночасно заявляє review PASS/ACCEPT і зберігає текст review pending. S08 не reconciles ці claims.

**Наслідок:** resume повторює виконану роботу або повідомляє недостовірний next action. Старий baseline сам по собі не дефект; дефект у суперечливих operational claims.

**Виправлення:** startup/resume/report reconciliation між authoritative journal, actual Git refs та immutable evidence subject. STATUS зробити проєкцією. Tested subject/tree, resulting implementation commit і checkpoint generation розділити: commit не може містити власний exact SHA. Суто metadata commit не повинен автоматично інвалідувати всі тести.

### F07 | HIGH | Owner-blocker rule змішує негайну заборону дії з exhaustion і повідомленням

**Місце / доказ:** V2 пункти 3–6 у порівнянні з S02/S09. Блокування meaningful increment та exhaustion не можуть бути передумовою припинення неавторизованої або небезпечної дії.

**Наслідок:** агент пробує unsafe alternatives або відкладає security decision, бо ще є дрібна робота; інший край — global stop через login для одного predicate.

**Виправлення:** негайно freeze affected action/scope; продовжувати лише незалежну admitted роботу. Окремо рахувати impact та notification. Urgent security/authority incident або cancellation може вимагати повідомлення до exhaustion. «Зробити все можливе» не включає обхід 2FA/consent чи новий data egress.

### F08 | HIGH | Розділення external proof та local ACCEPT може послабити acceptance після failure

**Місце / доказ:** V2 пункт 9; S01/S05 applicability; S06:41–46 та P01. Inventory не прив'язаний до immutable admission.

**Наслідок:** hard-required proof переноситься в майбутнє, а goal оголошується done; exact runtime замінюється іншим. Протилежна помилка — global live gate блокує дозволений non-deploying checkpoint commit.

**Виправлення:** stage-specific predicates/dependencies задаються acceptance contract. Implementation ACCEPT, commit eligibility, publish eligibility та goal completion різні. Required proof не видаляється і не стає NOT_APPLICABLE без authority. Named-branch push перевіряється на downstream CI/deployment effects. Blocked live proof залишається явним незавершеним критерієм.

### F09 | HIGH | Тести доводять simulation, не автономний workflow

**Місце / доказ:** S14; S13 historical validation; P01–P14. Happy path вручну змінює flags; resume test не завершує worker і не відновлює journal. P08 закріплює alphabetical commit next action.

**Наслідок:** кількість PASS створює false confidence щодо preflight, repair, independent review, resume та side effects.

**Виправлення:** unit/negative/property tests плюс crash injection, fake runtime з observable effect counter, temporary bare Git remote, реальний host return-gate integration і окремий authorized live smoke. Future matrix T01–T26 наведена в RECOMMENDED_MODEL.md; fixture не дорівнює external live proof.

### F10 | MEDIUM | Capability preflight не повинен бути глобальною одноразовою перепусткою

**Місце / доказ:** V2 пункт 1; S11/S12. Version/import probe та capability truth вже є, але універсальний operation-level contract/probe не продемонстровано.

**Наслідок:** remote login блокує безпечну local роботу; або stale PASS переживає model/protocol drift. Навіть probe може коштувати грошей, передавати дані чи встановлювати залежності.

**Виправлення:** per-capability exact operation/version/protocol/model/auth-scope evidence, TTL і dependency fingerprint; recheck на drift/resume/before use. Спочатку read-only/no-secret probes. Install/paid call/OS consent мають власний authority gate. Version або model inventory не є E2E proof.

### F11 | MEDIUM | blocked_scope та alternatives_exhausted недовизначені

**Місце / доказ:** V2 пункти 2/4/5. Approach є способом, predicate критерієм, task/increment/goal одиницями роботи: це не одна ієрархія.

**Наслідок:** scalar scope губить кілька affected predicates; boolean не показує оцінені alternatives, baseline, evidence або budget.

**Виправлення:** origin_attempt_id, множина affected_refs та dependency-derived impact. blocked_scope лишити derived summary для сумісності. Alternatives: UNASSESSED, AVAILABLE, EXHAUSTED_ADMITTED_SET, BUDGET_LIMITED; evidence-backed bounded candidate set. Budget limit не доказ technical impossibility. Не вимагати вичерпання всіх теоретичних рішень.

### F12 | MEDIUM | Unique increment може роздутися або зупиняти великий goal після кожного кроку

**Місце / доказ:** S01/S03/S05; V2 очікування end-to-end великого результату. Поточний mode формально прив'язаний до одного admitted increment.

**Наслідок:** моноліт на всі retries/live waits або ручний Owner routing кожного successor. Зміна current pointer сама не дає authority.

**Виправлення:** один canonical writer/current implementation slice для overlapping workspace, bounded tasks і parked verification nodes. Successor admit/select автономне лише всередині approved goal envelope. Split зберігає criteria, blockers, aggregate budgets і parent outcome. Independent work просуває accepted goal, не сторонній нескінченний cleanup.

### F13 | HIGH | Independent review потребує provenance та immutable subject

**Місце / доказ:** S04; frozen-source-set claim S13; S06:77–83; P03/P10. Policy має корисний frozen checkpoint, але evaluator не звіряє reviewer identity, exact subject або finding lifecycle.

**Наслідок:** self-review видається за independent; old ACCEPT переноситься на новий diff; недоступна review model замінюється implementation agent без дозволу.

**Виправлення:** окремий read-only invocation/context, reviewer identity, exact subject, verdict і stable finding IDs. Repair інвалідує affected acceptance; перевірки та re-review прив'язуються до нового subject. Substitute reviewer тільки за admission policy. Недоступність означає BLOCKED, не вигаданий ACCEPT. Інший model/provider сам по собі не гарантує незалежності.

**Разом: 0 critical, 10 high, 3 medium.** Critical exploit у працюючому general executor не продемонстровано; такий executor у цьому audited path не підтверджений. Severity оцінює ризик для заявлених гарантій, а не стверджує production incident.

## 6. Найсильніші заперечення до запропонованої V2

1. Boolean no-return не вирішує scheduling. Мовчання не означає живого виконавця, runnable action або deadline.
2. Exhaustion є обмеженим висновком щодо admitted candidate set за конкретного evidence/budget, не універсальним доказом неможливості.
3. Не всі stops є Owner decisions: cancellation, invalid state, unknown side effect, відсутній host та bounded technical failure потребують власних правдивих outcomes.
4. Partial acceptance корисна тільки без прихованого переписування hard criteria після невдачі.
5. Нові Markdown states без спільної authority/evidence моделі збільшують drift. Треба інтегрувати наявні PEF/admission/receipts, а не створювати паралельну систему.

## 7. Final architecture assessment

Зберегти bounded increments, grounding, capability truth, risk-matched verification, repair/re-review та explicit preauthorization. **Не приймати V2 як лише policy patch.** Мінімальна необхідна конструкція: typed transition evaluator + durable journal + supported host adapter/action gate, із scoped guarantees і fault tests.

Це не рекомендація впроваджувати важку workflow платформу. Existing PEF authorization/evidence semantics варто reuse або явно map. Advisory policy може бути корисним проміжним результатом, але її не можна назвати guaranteed autonomous execution.

Можна забезпечити перевірені переходи й відхилення недоказаного завершення в контрольованому host. Продовження після смерті host потребує реально налаштованого resumer [EXT3]. Невідомий outcome зовнішньої дії потребує reconciliation, а не впевненішого повторення. Критерій успіху: менше ручного message routing Owner без permission widening, silent stalls і false completion.

## 8. Зовнішні первинні джерела

Перевірено 2026-09-22. Це джерела загальних recovery принципів, не докази коду AI OS; впровадження AWS або Temporal не вимагається.

- **EXT1:** AWS Builders' Library, Making retries safe with idempotent APIs. Пояснює неоднозначність timeout, client request identity та semantic idempotency. https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/
- **EXT2:** Temporal, Detecting Activity failures. Розрізняє per-attempt timeout, overall retry duration і heartbeat. https://docs.temporal.io/encyclopedia/detecting-activity-failures
- **EXT3:** Temporal Workflow Execution overview. Durable execution спирається на persisted state/history та реальний execution mechanism. https://docs.temporal.io/workflow-execution
