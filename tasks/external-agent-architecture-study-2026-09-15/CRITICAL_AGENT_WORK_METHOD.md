# Critical Agent work method

## Проста суть

Сильний Agent не відрізняється тим, що пише найдовший план. Він відрізняється тим, що до роботи визначає, **що вважатиметься готовим**, під час роботи не плутає припущення з фактами, а після роботи намагається власний результат зламати.

## 1. Нормальний цикл великої цілі

Рекомендована state machine:

`INTAKE → CONTRACT → PLAN → EXECUTE/DELEGATE → INTEGRATE → VERIFY → ACCEPT | REVISE | BLOCKED`.

Кожен перехід має concrete output, а не лише внутрішнє «я подумав».

## 2. Task contract до плану

Agent формує короткий contract:

- **Objective:** який результат потрібен;
- **In scope / out of scope:** межі;
- **Constraints:** технічні, продуктові, часові, безпекові;
- **Acceptance criteria:** спостережувані умови готовності;
- **Evidence required:** чим доводиться кожний критерій;
- **Unknowns:** що поки невідомо;
- **Risks:** де результат може бути хибним;
- **Irreversible actions:** що потребує окремого approval;
- **Budget / stop condition:** скільки паралельності, retry та research виправдано.

Якщо user request уже містить чіткі criteria, Agent не переписує їх «своїми красивими словами» так, щоб сенс з’їхав.

## 3. Розділення складної цілі

Корисний decomposition unit має одну або кілька властивостей:

- незалежний evidence source;
- окремий output, який можна перевірити;
- мінімальна shared mutable state;
- чітка dependency на інші outputs;
- bounded context;
- окремий risk dimension.

Погане розбиття: 8 агентів досліджують те саме без різних гіпотез, а потім parent Agent має прочитати 8 майже однакових есе.

## 4. Ролі як режими, не обов’язково як окремі агенти

### Researcher

Шукає першоджерела, alternative explanations і missing facts. Його KPI — coverage/reliability evidence, а не переконливість тексту.

### Worker

Створює конкретний artifact або вирішує bounded subtask.

### Critic

Не переписує роботу з нуля. Шукає:

- strongest counterexample;
- hidden assumption;
- missed acceptance criterion;
- unsupported claim;
- simpler/safer alternative;
- security/recovery failure.

### Verifier

Звіряє final artifact з task contract та evidence. Якщо можливо, отримує contract + artifact + raw evidence, а не лише авторський self-report.

**Рекомендація.** Ці ролі можуть бути окремими Smart Subagent лише коли незалежність виправдовує ціну. На малих задачах один Agent може послідовно перейти між режимами.

## 5. Незалежна перевірка

Незалежність має кілька рівнів:

1. **Prompt independence:** verifier отримує neutral criteria, а не «доведи, що автор правий».
2. **Evidence independence:** verifier може перечитати source/test output.
3. **Execution independence:** для high-risk задач verifier запускає власну перевірку, а не цитує лог worker-а.
4. **Model/context independence:** іноді корисний окремий model run з іншим context framing.

Не кожна задача потребує всіх чотирьох.

## 6. Як запускати кілька Smart Subagent

Parent Agent спершу записує `delegation reason`:

- parallel independent research;
- alternative hypothesis;
- adversarial critique;
- independent verification;
- isolated implementation lane;
- context/breadth overflow.

Потім кожному дає різний bounded contract. Якщо причина однакова й outputs очікуються однакові, це має бути свідомий redundancy test, а не випадкове дублювання.

Детальні критерії у `MULTIPLE_SMART_SUBAGENTS.md`.

## 7. Об’єднання результатів

Parent Agent не просто concatenates outputs. Він будує інтеграційну таблицю:

| Claim/decision | Source A | Source B | Conflict? | Authority/freshness | Resolution |
|---|---|---|---|---|---|

Для artifacts — аналогічно: input SHA/version, validation result, dependencies, unresolved items.

## 8. Виявлення суперечностей

Коли два сильні виконавці не погоджуються:

1. сформулювати точне disputed claim;
2. відокремити різні definitions/assumptions;
3. порівняти source authority, date/freshness, directness;
4. визначити test або observation, що може розв’язати спір;
5. якщо не розв’язується — зберегти uncertainty як open decision, а не голосувати більшістю агентів.

Три однаково помилкові агенти не перетворюють помилку на факт. Демократія тут, на жаль, не рятує.

## 9. Критичність до власної ідеї

Перед ACCEPT Agent повинен пройти bounded self-check:

- що тут найімовірніше неправильно;
- яке припущення найбільше впливає на результат;
- який acceptance criterion ще не має evidence;
- чи є простіший варіант;
- чи створено новий ризик;
- що я називаю фактом без першоджерела;
- чи не плутаю plan/implementation/verification.

Відповіді не обов’язково показувати користувачу як довгий ritual. Важливий outcome.

## 10. Stop conditions

Agent завершує роботу коли:

- усі mandatory acceptance criteria мають evidence;
- unresolved items або non-blocking, або явно винесені як owner/open decisions;
- verification пройдена;
- немає known P0/P1 failure, який робить результат непридатним;
- подальша робота має нижчу очікувану цінність, ніж її cost/risk.

Agent **не** завершує роботу лише тому, що:

- закінчився початковий plan;
- subagent сказав «done»;
- є commit;
- є красивий report;
- більшість виконавців погодилась.

## 11. Retry policy

Рекомендований bounded default:

- перший verifier fail з concrete fix → одна автоматична correction attempt;
- друга спроба лише якщо failure class зрозумілий і нове виправлення відрізняється;
- повтор тієї самої помилки або невизначений blocker → зупинка/escalation;
- destructive/expensive retry потребує окремого approval/budget rule.

Це recommendation, не universal constant.

## 12. Verification ladder

Від дешевшого до дорожчого:

1. schema/static checks;
2. exact invariants/allowlists;
3. unit/task-specific tests;
4. integration checks;
5. independent model critique;
6. independent re-execution/research;
7. human/owner acceptance.

Agent піднімається по ladder відповідно до risk/uncertainty, а не запускає максимум завжди.

## 13. Що кажуть зовнішні підходи

### OpenAI Agents SDK

**Підтверджений факт.** Документація описує manager-as-tools, handoffs і code-orchestrated patterns; незалежних агентів можна запускати паралельно, а evaluator loops є окремим orchestration pattern.

### Anthropic multi-agent research

**Приклад іншої системи.** Lead agent планує broad research, делегує паралельним subagents і синтезує результати. Автори прямо відзначають високу token cost і те, що multi-agent підхід краще підходить для незалежних breadth-first напрямів, ніж для tight shared context.

### Reflexion / Self-Refine

**Приклад дослідження.** Наукові роботи показують, що feedback/reflection та iterative refinement можуть покращувати результат на низці задач. Це аргумент за bounded verify-revise loop, але не доказ, що нескінченна самокритика завжди корисна.

### Multi-agent debate

**Приклад дослідження.** Debate/multiple model instances може покращувати окремі reasoning/factuality benchmarks. Водночас це не скасовує correlated error та judge bias.

## 14. Мінімальна структура Agent decision record

```json
{
  "objective": "...",
  "acceptance_criteria": ["AC-1", "AC-2"],
  "plan": ["..."],
  "delegations": [
    {"reason": "independent verification", "subtask": "..."}
  ],
  "evidence": ["..."],
  "unresolved": ["..."],
  "verdict": "ACCEPT|REVISE|BLOCKED"
}
```

Це conceptual example, не запропонована обов’язкова product schema.

## Підтверджений факт

Зовнішні агентські frameworks підтримують manager/delegation, parallel tasks, evaluator loops і guardrails; академічні роботи дають evidence, що iterative feedback або multiple perspectives можуть допомагати на частині задач.

## Приклад іншої системи

Anthropic Research — lead + parallel subagents; OpenAI Agents SDK — manager/handoff/code orchestration; SWE-agent — tool/interface design для bounded code work.

## Рекомендація для задуму

Task contract → selective delegation → structured integration → risk-proportional verification → explicit verdict. Критик і verifier є функціями, а не обов’язковими окремими агентами для кожного кроку.

## Припущення

NovaStory Agent у майбутньому матиме змогу створювати окремі Smart Subagent runs та отримувати structured results. Це target assumption, не факт реалізації.

## Ризик

Overdelegation, shared-state races, reviewer loops, correlated hallucination і acceptance за self-report можуть створити враження «дуже серйозного процесу» без реального приросту якості.

## Відкрите рішення

Owner має визначити default risk tiers, максимальний fan-out/retry budget і які категорії задач обов’язково потребують незалежного verifier.
