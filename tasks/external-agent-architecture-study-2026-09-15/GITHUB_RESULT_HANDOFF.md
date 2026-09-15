# GitHub result handoff

## Проста суть

GitHub можна використати як надійний «поштовий ящик» між Smart Subagent і основною системою, але довіряти треба не назві гілки й не повідомленню «я все зробив». Надійна одиниця передачі — **конкретний commit SHA**, який пройшов перевірку.

## 1. Рекомендований протокол v1

### Створення run

Основна система створює унікальні:

- `task_id` — логічна задача;
- `run_id` — конкретна спроба;
- expected repository;
- expected branch;
- allowed result path;
- task manifest з acceptance criteria.

Приклад target-структури:

```text
tasks/<task_id>/
  TASK.json
  runs/<run_id>/
    RESULT.json
    artifacts/...
```

Це **рекомендація**, а не твердження про існуючу схему.

### Виконання

Smart Subagent отримує лише потрібний task context і працює у власній гілці, наприклад `run/<task_id>/<run_id>`. Він не ділить одну writable branch з іншими виконавцями.

### Публікація

Subagent записує результат, створює commit і push. Основна система не приймає відповідь «branch готова» як доказ.

### Сповіщення

Основний канал: GitHub `push` webhook.

**Підтверджений факт.** GitHub webhook delivery містить headers `X-GitHub-Delivery`, `X-GitHub-Event`, а при налаштованому secret — `X-Hub-Signature-256`. Push payload містить `ref`, `before` і `after`, де `after` є commit SHA після push.

### Перевірка

Backend повинен:

1. перевірити HMAC signature в constant-time comparison;
2. перевірити event type/action;
3. перевірити repository identity;
4. перевірити expected ref;
5. зафіксувати `after` SHA як candidate;
6. отримати tree/commit за цим exact SHA;
7. перевірити task/run manifest;
8. перевірити allowed paths, completeness і типи файлів;
9. запустити content/security validation;
10. повернути `ACCEPT`, `REJECT`, `RETRYABLE_FAILURE` або `BLOCKED`.

### Продовження

Після acceptance downstream-робота повинна посилатися на accepted SHA. Якщо branch потім пересунулась, accepted artifact identity не змінюється.

## 2. Webhook проти polling

| Механізм | Перевага | Слабке місце | Рекомендація |
|---|---|---|---|
| Push webhook | майже одразу, event-driven, точний SHA в payload | delivery може бути пропущено; endpoint треба захищати | основний канал |
| Periodic polling | простий recovery, не залежить від доставки одного event | затримка, API cost, race якщо читати лише branch head | fallback/reconciliation |

**Рекомендація для задуму.** Обидва механізми повинні викликати один і той самий `candidate commit validator`. Poller не має «приймати branch», він лише знаходить нові SHA й подає їх у той самий pipeline.

### Ідемпотентність

Webhook delivery ID корисний для replay/deduplication конкретної доставки. Але сутність результату краще ідентифікувати ключем на кшталт:

`repository_id + task_id + run_id + commit_sha + validator_version`.

Тоді повторний webhook і fallback polling не запускають повторне acceptance одного й того самого candidate без причини.

## 3. GitHub App, fine-grained PAT, deploy key

### GitHub App

**Підтверджений факт.** GitHub Apps мають permission model; installation access tokens можна обмежити репозиторіями й permissions, а токени мають короткий строк життя.

**Рекомендація.** Для стабільного unattended/cloud runtime це кращий довгостроковий варіант: окремий service identity, короткоживучі credentials, мінімальні права, webhooks і Checks API в одному механізмі.

### Fine-grained PAT

**Підтверджений факт.** Fine-grained PAT дозволяє обмежувати repository access і permissions.

**Рекомендація.** Прийнятний для контрольованого owner-only prototype, якщо права мінімальні, token не потрапляє в prompts/files і є зрозуміла rotation/revocation процедура. Не варто робити його довгостроковим універсальним service credential, якщо система росте.

### Deploy key

Може бути вузьким credential для одного repo, але гірше моделює application identity, permission lifecycle та багаторепозиторну оркестрацію. Для цього задуму GitHub App зазвичай чистіший масштабований напрям.

## 4. Окремі гілки, окремі папки, PR чи direct commit

### Окрема гілка

**Рекомендація.** Один active run = одна write branch. Це мінімізує writer collisions і дає просту provenance boundary.

### Окрема папка

Навіть з branch isolation корисна task/run folder. Вона дозволяє перевірити, що виконавець не торкнувся сторонніх результатів.

### Direct commit

Для result-only repository v1 direct commit у run branch простіший і дешевший, якщо основна система сама виконує acceptance.

### Pull request

PR корисний, коли результат має бути людсько reviewed або merged у canonical branch, коли потрібні required checks/rulesets, або коли GitHub UI є частиною approval workflow.

**Рекомендація.** Не робити PR обов’язковим для кожного ephemeral result branch на першій версії. Додавати його на межі «candidate → canonical» або для high-risk/human approval work.

## 5. Exact commit SHA та захист від branch substitution

Назва гілки є mutable pointer. Тому перевірка має починатися з exact SHA з authenticated event/polling observation.

Після отримання candidate SHA:

- перевірити, що commit належить очікуваному repository;
- перевірити, що SHA reachable з expected ref у потрібний момент або відповідає protocol state;
- перевірити parent/attempt relationship, якщо воно потрібне;
- не «переключатися» автоматично на новіший branch head посеред review;
- наступний push створює новий candidate.

## 6. Signed commits і branch protection

**Підтверджений факт.** GitHub branch protections/rulesets можуть вимагати signed commits та status checks.

**Рекомендація.** Signed commits корисні як provenance signal, особливо для canonical/protected branches. Для ephemeral isolated result branches v1 вони не замінюють webhook authentication, exact-SHA validation і repository permissions. Тому не робити їх єдиним gate у мінімальній версії.

Для canonical acceptance branch пізніше доречні:

- no force-push;
- required status/checks;
- signed commits, якщо operational setup це підтримує;
- merge/approval policy відповідно до ризику.

## 7. Checks перед прийняттям

Мінімальні deterministic checks:

- `RESULT.json` валідний;
- task_id/run_id збігаються;
- лише allowed paths;
- required files присутні;
- заборонені symlink/path escape відхиляються;
- file size/type limits;
- secret scanning;
- archive/binary policy;
- no unexpected executable content, якщо воно не дозволене задачею;
- acceptance-specific tests.

Content файлів треба вважати untrusted input. Навіть якщо їх створив «наш» Subagent, вони можуть містити prompt injection або випадкову інструкцію, яку downstream Agent не повинен сприймати як system authority.

## 8. Retry та виправлення

Невдалий candidate не переписується заднім числом.

Рекомендована модель:

`run 1 / candidate A → reject → feedback → candidate B`.

Зберігати:

- failure code;
- validator version;
- bounded human-readable feedback;
- previous candidate SHA;
- attempt number.

Не використовувати force-push як нормальний retry-механізм. Історія виправлень є корисним evidence.

## 9. Кілька Smart Subagent одночасно

Кожен отримує:

- окремий `run_id`;
- окрему branch;
- окрему result folder;
- bounded permissions;
- власний expected task contract.

Parent Agent збирає accepted candidate SHAs і лише тоді порівнює результати. Shared writable state краще уникати; якщо integration потрібна, один визначений integrator створює новий commit з явними parent inputs.

## 10. Локальне отримання лише папки задачі

**Підтверджений факт.** Git має sparse checkout для роботи з підмножиною tree та partial clone для уникнення завантаження непотрібних objects з on-demand fetch.

**Рекомендація.** Для локального продовження великого result repo використовувати partial clone + sparse checkout на `tasks/<task_id>` або еквівалентний bounded path. Але security boundary це не створює: permissions усе одно повинні контролювати repo access.

## 11. Cloud continuation без локального clone

Cloud worker може читати конкретні файли/tree за exact commit SHA через GitHub API. Це корисно, якщо downstream-робота не потребує повного git working tree.

**Рекомендація.** API-first continuation для review/research/metadata; локальний checkout лише коли потрібні repo tools, build/test або filesystem-oriented workflow.

## 12. Failure matrix

| Failure | Реакція |
|---|---|
| invalid webhook signature | reject before parsing trusted state |
| duplicate delivery | dedupe, no duplicate acceptance |
| missed webhook | polling знаходить unseen SHA |
| branch moved after event | continue validating pinned SHA |
| force-push | treat new observed SHA as new candidate; do not rewrite accepted identity |
| wrong task/run manifest | reject |
| incomplete result | reject with missing-artifact code |
| foreign/unexpected file | quarantine/reject according to allowlist |
| secret detected | reject/quarantine; never echo secret in model prompt |
| validator temporarily unavailable | retry validation, not task execution |
| execution produced bad content | new task attempt/candidate |

## Підтверджений факт

GitHub документує HMAC webhook validation, delivery/event headers, exact push SHAs, GitHub App installation tokens, fine-grained permissions, protected branches/rulesets і required checks.

## Приклад іншої системи

GitHub Checks API дозволяє прив’язувати check run до конкретного commit. Це корисний приклад того, як result validation можна робити commit-pinned, а не branch-name-pinned.

## Рекомендація для задуму

V1: isolated run branch + task folder + direct commit + authenticated webhook + exact SHA + common validator + polling fallback. GitHub App, PR checks і protected canonical branch додавати після стабілізації protocol.

## Припущення

Основна система матиме backend, який зберігає run state й може верифікувати GitHub events. Це цільове припущення.

## Ризик

Найнебезпечніша помилка — змішати «сповіщення про branch» з «доведеним результатом». Branch може рухатись, webhook може повторитись, а content може бути шкідливим.

## Відкрите рішення

Owner має визначити довгострокову authentication strategy, чи потрібні PR на canonical acceptance, чи вимагати signed commits і як довго зберігати ephemeral run branches.
