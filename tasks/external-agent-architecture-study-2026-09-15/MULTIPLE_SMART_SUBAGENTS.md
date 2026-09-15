# Multiple Smart Subagents

## Проста суть

Кілька виконавців корисні не тому, що «більше AI = більше розуму». Вони корисні, коли роботу реально можна рознести по незалежних напрямках або коли друга незалежна думка ловить помилку, яку перший виконавець не бачить. Якщо підзадачі тісно пов’язані, паралельність часто лише множить координацію.

## 1. Коли запускати додаткового виконавця

Додатковий Smart Subagent виправданий, якщо присутній хоча б один сильний trigger і очікувана користь перевищує координаційний cost.

### Trigger A: незалежні напрями

Наприклад, один виконавець досліджує GitHub security, другий instruction versioning, третій retrieval. Їхні outputs можна перевірити окремо й об’єднати після завершення.

### Trigger B: альтернативні гіпотези

Є кілька правдоподібних рішень і важливо не закохатися в перше. Виконавці отримують різні hypotheses/constraints, а parent Agent порівнює evidence.

### Trigger C: adversarial critique

Потрібен окремий пошук слабких місць, security failures, counterexamples або missing acceptance criteria.

### Trigger D: незалежна verification

High-risk або expensive result варто перевірити окремим run, який отримує neutral criteria та evidence.

### Trigger E: breadth/context pressure

Один контекст стає надто широким: багато джерел, доменів або локально незалежних artifacts. Розбиття зменшує context pollution.

### Trigger F: ізоляція writer-а

Кілька implementation lanes можуть бути корисні, якщо кожна працює в окремій sandbox/branch і є один integrator.

## 2. Коли НЕ запускати

Не рекомендується fan-out, якщо:

- задача коротка й детермінована;
- підзадача B майже повністю залежить від незавершеної A;
- всі виконавці повинні постійно змінювати той самий shared state;
- deterministic test дешевший і надійніший за ще одного LLM;
- кожному агенту доведеться повторно завантажити той самий великий контекст;
- потрібне одне вузьке джерело істини, а не breadth;
- координація та інтеграція дорожчі за саму роботу;
- «друга думка» фактично отримує той самий prompt, sources і framing та не створює незалежності.

## 3. Рекомендований fan-out для v1

**Рекомендація для задуму:**

- default: 1 active Smart Subagent;
- типовий parallel fan-out: 2–3;
- 4+ лише коли Agent записав concrete breadth/parallelism reason;
- hard cap має бути runtime-configurable й окремим owner decision.

Число 3 не є науковою константою. Це conservative v1 default, щоб система спершу навчилася правильно делегувати, а не героїчно оплачувати хаос.

## 4. Вартість multi-agent

**Приклад іншої системи.** Anthropic у матеріалі про multi-agent research повідомляє, що agentic search значно дорожчий за звичайний chat, а multi-agent research ще дорожчий; водночас паралельні subagents допомагають на breadth-heavy research.

Наслідок: decision про fan-out треба оцінювати не лише через quality, а й через:

- total tokens;
- wall-clock latency;
- duplicated context;
- tool/API cost;
- integration time;
- false conflict rate;
- number of unnecessary workers.

## 5. Як давати різні напрями

Погано:

```text
Agent 1: досліди найкращу архітектуру
Agent 2: досліди найкращу архітектуру
Agent 3: досліди найкращу архітектуру
```

Краще:

```text
A: знайди підтверджені можливості та офіційні обмеження
B: шукай failure/security modes і контраргументи
C: порівняй простіший мінімальний варіант з масштабованим варіантом
```

Або для independent verification:

```text
Worker: створи artifact за AC-1..AC-5
Verifier: не дивись на авторський verdict; перевір AC-1..AC-5 за raw evidence
```

## 6. Structured result contract

Кожен subagent повертає компактний результат:

- subtask id;
- scope;
- claims/findings;
- evidence/sources;
- assumptions;
- risks;
- unresolved questions;
- artifacts/commit SHA;
- self-reported completion state.

Self-reported state не є final acceptance. Parent/validator перевіряє його.

## 7. Паралельність і dependencies

Agent будує маленький dependency graph:

```text
A research ─┐
B critique ─┼→ integrate → verify
C alternative ┘
```

Тільки nodes без незакритих hard dependencies запускаються паралельно. Якщо B потребує exact output A, B не треба фальшиво «паралелити» з A.

## 8. Writer isolation

Для tasks, що змінюють artifacts:

- один writer на branch/worktree/run area;
- інші reviewer/researcher не пишуть туди напряму;
- integration робить один визначений actor;
- input/output commits pin-яться exact SHA;
- conflict resolution записується явно.

## 9. Як порівнювати суперечливі результати

Parent Agent не рахує голоси. Він оцінює:

1. source authority;
2. directness evidence;
3. freshness;
4. відповідність task constraints;
5. reproducible test;
6. hidden assumptions;
7. risk if wrong.

Якщо обидва outputs мають сильні аргументи, запускається targeted resolver/verifier тільки по disputed claim, а не ще три повних research agents.

## 10. Захист від agent explosion

Runtime safeguards:

- global concurrency cap;
- per-parent child cap;
- depth cap;
- token/time/tool-call budget;
- no child spawning unless parent explicitly grants it;
- dedupe схожих subtasks;
- stop on repeated no-progress;
- observability: reason, cost, latency, result quality per delegation.

**Рекомендація.** Smart Subagent v1 не повинен за замовчуванням мати право запускати власних Smart Subagent. Інакше дерево дуже швидко починає займатися власним вирощуванням.

## 11. Decision scorecard

Перед spawn Agent може швидко оцінити 0/1:

- `independent`: subtask реально незалежний;
- `parallel_gain`: є wall-clock/coverage benefit;
- `diversity_gain`: інший напрям/критика зменшує blind spot;
- `verification_gain`: незалежний run materially знижує risk;
- `bounded_context`: subtask можна дати компактно;
- `coordination_cost_low`: integration проста.

При 0–1 сильному сигналі працює сам. При 2–3 — один додатковий worker може бути доречний. При 4+ — parallel plan зазвичай виправданий. Це heuristic для eval, не жорсткий theorem.

## 12. Що перевіряти в evals

- precision delegation: частка spawn, які справді дали measurable benefit;
- missed delegation: випадки, де один Agent провалив breadth/verification, а parallel baseline виграв;
- duplicated-work ratio;
- total tokens / successful task;
- latency;
- contradiction resolution accuracy;
- shared-state conflicts;
- acceptance quality;
- unnecessary fan-out rate.

## Підтверджений факт

OpenAI Agents SDK дозволяє code-orchestrated parallel agents. Anthropic описує multi-agent research з lead agent і parallel subagents та прямо обговорює cost/coordination trade-offs.

## Приклад іншої системи

Anthropic використовує parallel research directions для breadth-heavy задач; OpenAI SDK показує паралельний запуск незалежних agents через orchestration code.

## Рекомендація для задуму

Default one; fan-out 2–3 лише за triggers; bounded child/depth/budget; distinct subtask contracts; один integrator; targeted resolver для конфліктів.

## Припущення

Майбутній runtime зможе рахувати cost/latency та давати child runs окремі contexts/workspaces.

## Ризик

Agent explosion, duplicated context, shared writer conflicts, correlated errors і «consensus by repetition».

## Відкрите рішення

Точний concurrency/depth cap, бюджет і право child-agent spawning має визначити власник після pilot evals.
