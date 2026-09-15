# Main recommendations

## На хлопський розум

Найсильніша перша версія тут не та, де найбільше агентів, баз даних і «магічної пам’яті». Найсильніша та, де кожен крок можна точно перевірити: хто отримав задачу, який commit повернув, що саме було перевірено, які інструкції діяли і чому Agent вирішив, що робота готова.

## Рекомендований v1

### 1. GitHub як перевірений handoff, а не просто сховище файлів

**Рекомендація для задуму.** Smart Subagent працює у власній гілці й task/run-папці. Після push основна система отримує webhook, бере з payload точний `after` commit SHA і перевіряє саме цей commit. Назва гілки використовується як routing metadata, але не як identity результату.

Мінімальний маршрут:

`task created → run branch → result commit → signed webhook → exact SHA → validator → ACCEPT / REJECT / RETRY`.

Періодичне опитування GitHub зберігається як fallback для пропущеного webhook і викликає той самий validator, а не окрему логіку.

### 2. Окремий writer на одну ізольовану робочу область

**Рекомендація для задуму.** Кожен одночасний Smart Subagent отримує окрему гілку або іншу ізольовану write-area. Десять агентів, що «акуратно» пишуть в одну гілку, є дуже дорогим способом винайти гонки стану.

### 3. Приймати commit, не branch head

**Підтверджений факт.** GitHub push webhook містить `before`, `after` і `ref`; `after` є SHA останнього commit після push.

**Рекомендація для задуму.** Після фіксації candidate SHA подальше пересування гілки не повинно змінювати те, що саме перевіряється. Якщо потрібне виправлення, воно створює новий candidate SHA / attempt.

### 4. Два постійні instruction-файли, але жодних permissions у них

**Рекомендація для задуму.** Завжди збирати контекст у порядку:

`base role instruction → owner instruction → task/user context → selectively loaded knowledge`.

Base задає стабільну роль і спосіб роботи. Owner-файл уточнює стиль, пріоритети й робочі правила. Tool permissions, доступ до секретів, allow/deny operation gates і irreversible-action checks повинні забезпечуватися програмою.

### 5. Версія інструкцій є частиною receipt відповіді

**Рекомендація для задуму.** Для кожного серйозного run зберігати принаймні `base_instruction_version/hash`, `owner_instruction_version/hash`, `model/config`, `tool schema version`, `knowledge manifest id/version`. Тоді відповідь можна реально відтворити й розбирати, а не сперечатися з привидом «десь модель минулого разу поводилась інакше».

### 6. Agent працює через task contract та evidence gates

**Рекомендація для задуму.** До делегування Agent формує:

- objective;
- in/out of scope;
- constraints;
- acceptance criteria;
- required evidence;
- unknowns і ризики;
- budget/stop conditions.

Потім кожен результат звіряється з цим контрактом. «Сильний виконавець сказав, що готово» не є acceptance criterion.

### 7. Додаткові Smart Subagent запускаються за критеріями, а не за ритуалом

**Приклад іншої системи.** Anthropic описує multi-agent research як особливо корисний для breadth-first досліджень з незалежними напрямами, але значно дорожчий за single-agent підхід.

**Рекомендація для задуму.** Додатковий виконавець виправданий, коли є хоча б одна сильна причина: незалежні напрямки можна досліджувати паралельно; потрібна незалежна критика; один контекст занадто широкий; альтернативні гіпотези мають реальну цінність; або ізоляція зменшує ризик. Default v1: 1 виконавець; звичайний fan-out 2–3; більше лише з явною причиною.

### 8. Natural Chat описується через спостережувану поведінку

**Рекомендація для задуму.** Не писати «пояснюй на хлопський розум» як магічне закляття. Записати поведінку, яку можна перевірити:

- перші 1–3 речення дають суть;
- далі причина й наслідок;
- конкретний приклад лише коли допомагає;
- знайомі слова замість канцеляриту;
- точний технічний термін не викидається, а коротко пояснюється;
- сильний контраргумент користувачу не приховується;
- заголовки та списки застосовуються лише коли структура справді цього потребує.

### 9. Knowledge v1: indexes + metadata + exact/lexical search

**Рекомендація для задуму.** Почати з коротких покажчиків і metadata про authority/status/version/dependencies. Додавати semantic search, hybrid retrieval і reranking лише після конкретних eval failures. Graph retrieval варто розглядати ще пізніше, якщо з’являються повторювані глобальні питання про зв’язки великого корпусу, які базовий retrieval стабільно не вирішує.

### 10. Evals є частиною архітектури

**Рекомендація для задуму.** Перед activation нової instruction version або retrieval-стратегії проганяти стабільний набір звичайних, складних і навмисно проблемних сценаріїв. Integrity-критерії перевіряти детерміновано там, де це можливо; стиль і якість можна оцінювати моделлю + людиною, але не одним самозакоханим LLM-суддею.

## Мінімум зараз → розширення потім

| Етап | Мінімум | Додавати лише коли є причина |
|---|---|---|
| Handoff | direct commit в isolated branch, exact SHA, webhook, polling fallback | GitHub App checks, PR gates, protected canonical branches |
| Auth | вузько scoped credential для контрольованого прототипу | GitHub App installation tokens для стабільного unattended runtime |
| Instructions | 2 versioned small files + run receipt | richer editor, staged rollout, automated prompt regression suite |
| Agent | task contract + one verifier gate for risky work | richer critic/researcher swarms |
| Knowledge | indexes + exact/keyword retrieval | semantic → hybrid → rerank → graph only by evidence |
| Evals | deterministic integrity + curated behavior cases | continuous eval telemetry and statistical regression tracking |

## Що не варто робити у v1

- не вимагати PR для кожного тимчасового result branch, якщо нічого не потрібно merge у canonical branch;
- не робити signed commits єдиним механізмом довіри до ephemeral result branches;
- не запускати 5–10 subagents «для якості» без незалежних задач;
- не давати owner instruction можливість текстом обходити programmatic security gates;
- не завантажувати весь knowledge corpus «про всяк випадок»;
- не впроваджувати GraphRAG лише тому, що слово `graph` звучить солідно.

## Підтверджений факт

GitHub підтримує webhook signatures і delivery identifiers; push payload містить exact commit SHA. OpenAI Agents SDK документує manager/handoff/code-orchestration patterns і паралельний запуск незалежних агентів. Поточний OpenAI Model Spec визначає authority levels та правило, що пізніша інструкція того самого рівня може замінювати попередню.

## Приклад іншої системи

Anthropic multi-agent research використовує lead agent та паралельні subagents для breadth-first research; SWE-agent підкреслює цінність добре спроєктованого agent-computer interface; GraphRAG додає graph/community structure для глобальних питань по корпусу, але має суттєво складніший ingestion.

## Припущення

У майбутній системі результат Smart Subagent можна представити як immutable candidate commit і основна система матиме backend-компонент, здатний валідувати webhook/polling events. Це target assumption, а не факт про NovaStory.

## Ризик

Найбільші ризики: mutable branch refs, duplicate/replayed webhook, overprivileged credentials, prompt injection з result-файлів, secret leakage, overdelegation, correlated agent mistakes, stale knowledge та instruction drift.

## Відкрите рішення

Authentication strategy, обов’язковість PR/signed commits, точний concurrency cap, activation policy для змін owner instructions та межа між Agent і Chat залишаються owner decisions. Див. `OWNER_DECISIONS.md`.
