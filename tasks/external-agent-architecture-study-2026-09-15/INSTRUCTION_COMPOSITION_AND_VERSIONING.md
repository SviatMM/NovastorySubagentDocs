# Instruction composition and versioning

## Проста суть

Два постійні файли мають працювати як «конституція ролі + налаштування власника», а не як два випадкові тексти, що сваряться між собою. Їх треба версіонувати, перевіряти на конфлікти й фіксувати точну пару версій для кожного важливого run.

## 1. Ролі двох файлів

### Base role instruction

Стабільний шар, який описує:

- хто ця роль;
- основну відповідальність;
- загальний work method;
- правила чесності й evidence;
- як поводитись з невизначеністю;
- як використовувати tools/knowledge, не визначаючи самі permissions;
- invariant output/verification expectations, якщо вони справді універсальні.

Не слід класти сюди:

- енциклопедію продукту;
- довгу історію рішень;
- секрети;
- access tokens;
- allow/deny permissions, які runtime повинен enforcing сам;
- task-specific details.

### Owner instruction

Частіше змінюваний шар:

- пріоритети власника;
- бажаний стиль пояснення;
- правила прийняття некритичних припущень;
- product/workflow preferences;
- межі, які справді є preference/decision, а не security boundary;
- домовлені conventions.

## 2. Порядок завантаження

**Рекомендація для задуму:**

`base role → owner instruction → current task/user input → selectively loaded knowledge`.

**Підтверджений факт.** OpenAI Model Spec визначає instruction authority levels і зазначає, що пізніша інструкція того самого рівня може supersede попередню. Тому якщо обидва файли подаються на одному application/developer authority level, порядок має значення.

Практичне правило:

- base задає default;
- owner-файл може уточнювати або override лише явно дозволені preference/workflow areas;
- security, tool permissions, irreversible-action approvals і data-access boundaries не вирішуються через текстову суперечку двох prompt-файлів.

## 3. Як уникати суперечностей

До activation нової instruction version робити instruction lint:

1. знайти правила, що торкаються однієї поведінки;
2. класифікувати їх як compatible / explicit override / conflict / ambiguous;
3. вимагати явного override marker або рішення власника для conflict;
4. прогнати regression evals;
5. активувати immutable version.

Не варто покладатися на «модель якось зрозуміє». Вона часто зрозуміє, але саме слово «часто» робить погані runtime-контракти.

## 4. Версії

Кожна активована версія має бути immutable artifact з:

- `instruction_set_id`;
- semantic або monotonic version;
- content hash;
- created/activated timestamp;
- author/source;
- supersedes;
- change summary;
- eval result/reference;
- activation status.

Редагування створює нову версію, а не переписує стару.

## 5. Точна версія для конкретної відповіді

Run/response receipt для важливих задач має записувати:

```text
base_instruction: id + version + hash
owner_instruction: id + version + hash
model/config: exact runtime identifiers
knowledge_manifest: id + version/hash
tool_schema/runtime_policy: version
```

Це не означає, що receipt треба показувати користувачу в кожній побутовій відповіді. Це observability/provenance layer.

## 6. Редагування owner instruction через Chat

Рекомендований workflow:

1. користувач просить змінити правило;
2. Chat формує **proposed diff**, а не тихо редагує active instruction;
3. показує коротко: що зміниться, які старі правила зачіпаються, можливі конфлікти/наслідки;
4. programmatic validator перевіряє schema/limits/conflicts;
5. regression evals перевіряють поведінку;
6. owner явно приймає або відхиляє;
7. створюється нова immutable version;
8. active pointer переключається на неї;
9. receipt майбутніх runs фіксує нову version.

Для незначних текстових виправлень продукт може пізніше мати спрощений режим, але сам принцип preview → validate → activate варто зберегти.

## 7. Як показувати запропоновану зміну

Користувачу достатньо:

- короткої суті;
- diff або before/after тільки зміненого фрагмента;
- «що це змінить у поведінці»;
- conflicts/warnings;
- eval summary;
- кнопок/дій Accept / Edit / Reject.

Не треба показувати весь мегапромпт після кожної коми.

## 8. Rollback

Rollback = переключити active pointer на попередню вже відому immutable version.

Не треба:

- просити модель «відновити, як було»;
- переписувати history;
- вгадувати старий текст з chat history.

Після rollback нові runs pin-ять стару version; historical receipts лишаються незмінними.

## 9. Перше повідомлення лише з файлом

Постійні base/owner instructions повинні бути зібрані **до** обробки user content, незалежно від того, є текст чи тільки attachment.

Якщо перше user message містить лише файл і немає явного UI action/task intent:

- не вигадувати «проаналізуй файл»;
- не запускати destructive/expensive work;
- можна прочитати мінімальну metadata, якщо це потрібно для безпечної відповіді;
- попросити коротко уточнити, що зробити з файлом.

Якщо intent однозначно задається UI flow, наприклад користувач натиснув конкретну дію «Summarize», це вже task signal і зайве уточнення не потрібне.

## 10. Інструкції ≠ permissions

Текст може сказати «не видаляй файл», але реальний safety boundary повинен бути в tool/runtime policy:

- scoped credentials;
- allowed tools/actions;
- path allowlists;
- approval gates;
- rate/budget limits;
- irreversible-action checks;
- sandboxing.

**Підтверджений факт.** Model Spec відрізняє instruction authority та untrusted data; зовнішній content сам по собі не отримує authority. Це хороший принцип для розділення policy від content.

## 11. Як не перетворити файли на енциклопедію

Критерій для permanent instruction:

> Чи це правило потрібне майже в кожній розмові цієї ролі, щоб модель поводилась правильно?

Якщо ні, воно ймовірно належить до knowledge, task context, tool contract, memory або окремого policy layer.

Великі приклади теж варто винести в eval corpus / examples library, а в prompt лишити коротке правило + один representative example максимум.

## 12. Якість instruction-файлів

Перевіряти не «чи красиво написано», а поведінку:

- instruction following;
- conflict handling;
- task completion;
- false completion rate;
- unnecessary clarification rate;
- overdelegation;
- unsupported certainty;
- natural Ukrainian style;
- file-only first message;
- regression on normal short tasks;
- knowledge loading discipline.

**Підтверджений факт.** OpenAI має Evals API/graders і рекомендує оцінювати prompts на реальних задачах. Поточні model guides також радять outcome-oriented prompting і підвищувати reasoning/complexity лише коли evals показують користь.

## 13. Minimum viable schemas

### Base role

```yaml
id: novastory-agent-base
version: 1
status: active
purpose: ...
responsibilities: ...
work_method: ...
truth_and_evidence: ...
uncertainty: ...
knowledge_usage: ...
completion_rules: ...
```

### Owner instruction

```yaml
id: owner-instructions
version: 12
status: active
preferences: ...
communication: ...
workflow_choices: ...
explicit_overrides: ...
open_decisions: ...
```

Формат YAML тут лише приклад. Не потрібно вибирати його як product decision без окремого рішення.

## Підтверджений факт

OpenAI Model Spec дає authority model і same-level recency behavior; OpenAI API підтримує versioned prompt objects у відповідних interfaces, що демонструє практичність version pinning як загального підходу.

## Приклад іншої системи

OpenAI prompt resources дозволяють посилатися на prompt `id` і version. Це не означає, що NovaStory повинна копіювати цей API, але підтверджує корисність immutable/version-pinned instruction artifacts.

## Рекомендація для задуму

Base first, owner second; explicit override zones; immutable versions; proposed diff → validation/evals → owner activation; exact versions у run receipt; permissions поза prompt.

## Припущення

Обидва постійні файли можуть бути assembled програмою на одному instruction authority layer. Якщо runtime дасть їм різні authority levels, precedence треба буде переглянути.

## Ризик

Silent instruction edits, hidden conflicts, oversized permanent prompts і security rules, що існують лише у тексті, швидко роблять систему непередбачуваною.

## Відкрите рішення

Owner має вирішити, які саме типи змін owner instruction потребують explicit confirmation, чи потрібен staged rollout, і які fields/format є canonical.
