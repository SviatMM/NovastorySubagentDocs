# External system examples

Цей файл показує реальні patterns, а не «референси, які треба скопіювати». Кожен приклад має конкретний урок і concrete limit.

## 1. OpenAI Agents SDK

**Підтверджений факт.** SDK документує кілька orchestration patterns:

- manager agent, що викликає спеціалістів як tools;
- handoffs, де specialist перебирає conversation/control;
- code-orchestrated chaining;
- parallel independent agent calls;
- evaluator loops;
- guardrails і tracing.

**Що корисно для задуму:** NovaStory Agent природно відповідає manager-like role, коли він лишається власником user goal та інтегрує outputs. Для deterministic orchestration частину logic краще тримати в code/runtime, а не просити LLM щоразу винаходити protocol.

**Що не переносити автоматично:** SDK pattern не вирішує сам по собі product permissions, storage, GitHub handoff або instruction hierarchy NovaStory.

## 2. Anthropic multi-agent research system

**Приклад іншої системи.** Lead research agent декомпозує broad query, запускає parallel subagents для різних напрямів і синтезує результати. Engineering article підкреслює:

- benefit на breadth-first independent research;
- значний token/cost overhead;
- importance of delegation quality;
- необхідність bounded parallelism та observability;
- гіршу придатність для задач з великою shared context/dependency.

**Урок:** multi-agent — це спеціалізація під структуру задачі, а не universal upgrade.

## 3. SWE-agent

**Приклад іншої системи.** SWE-agent досліджує Agent-Computer Interface (ACI): які саме команди, feedback і interaction primitives бачить coding agent.

**Урок:** якість виконавця залежить не лише від prompt/model, а й від того, наскільки чітко спроєктовані інструменти, state feedback і bounded actions. Для Smart Subagent tool contract має бути простим і observable.

## 4. GitHub Apps + webhooks + Checks

**Підтверджений факт.** GitHub Apps підтримують scoped permissions/installation tokens, webhooks; Checks API прив'язує check runs до конкретних commits.

**Урок:** external handoff природно будувати навколо immutable commit identity, short-lived/scoped service auth і commit-pinned validation.

**Не робити висновок:** GitHub App не обов’язково потрібен у найпершому owner-only prototype; protocol можна відділити від credential implementation.

## 5. Git sparse checkout + partial clone

**Підтверджений факт.** Git підтримує sparse checkout для working tree subset і partial clone для on-demand object fetching.

**Урок:** локальний worker може працювати лише з потрібною task folder без materializing у working tree весь result repo.

**Обмеження:** це performance/working-set optimization, не authorization boundary.

## 6. OpenAI versioned prompt references

**Підтверджений факт.** OpenAI Responses interfaces підтримують prompt reference з `id`, variables та version.

**Урок:** pinning exact instruction version на run є нормальним operational pattern. NovaStory не зобов’язана повторювати той самий API.

## 7. OpenAI Model Spec

**Підтверджений факт.** Model Spec описує authority levels, conflicts, same-level later-instruction behavior, untrusted data та non-sycophantic/clear interaction expectations.

**Урок:** base/owner/task/data не можна змішувати в один «великий prompt без походження». Instruction authority і data provenance мають бути explicit.

## 8. Elastic hybrid search / reranking

**Підтверджений факт.** Elastic документує full-text + vector hybrid search і rank fusion; reranking застосовується після candidate retrieval.

**Урок:** hybrid/rerank — це escalation для measured retrieval problems, а не обов’язковий starting point.

## 9. Microsoft GraphRAG

**Приклад іншої системи.** GraphRAG витягає entities/relationships, будує graph/community structure та використовує її для global questions/aggregation над corpus.

**Урок:** graph retrieval може вирішити class проблем, який simple chunk RAG вирішує погано: глобальні зв’язки й corpus-level questions.

**Обмеження:** indexing складніший і дорожчий. Для простої першої knowledge system це, швидше за все, передчасно.

## 10. Reflexion

**Приклад наукової роботи.** Reflexion використовує feedback/reflection memory для покращення наступних attempts без зміни model weights.

**Урок:** короткий post-failure reflection може бути корисним input у retry, якщо він прив’язаний до concrete failure.

**Обмеження:** це не доказ, що нескінченні self-reflection loops завжди допомагають.

## 11. Self-Refine

**Приклад наукової роботи.** Self-Refine демонструє iterative generate → feedback → refine loop на різних задачах.

**Урок:** bounded verifier feedback → correction є обґрунтованим pattern.

**Обмеження:** improvement залежить від task/model; потрібні stop/budget rules.

## 12. Multi-agent debate

**Приклад наукової роботи.** Multiple model instances, що обмінюються відповідями/критикою, показали покращення на частині reasoning/factuality evaluations.

**Урок:** незалежні perspectives іноді ловлять blind spots.

**Обмеження:** consensus не гарантує істину; agents можуть мати correlated biases/errors.

## 13. LLM judge self-preference research

**Приклад наукової роботи.** Recent research показує self-preference bias в LLM judges.

**Урок:** Agent не повинен приймати власний output лише тому, що інший run того самого model family поставив високий score. Потрібні raw evidence, deterministic checks і, де важливо, human/independent review.

## Pattern comparison

| Pattern | Сильна сторона | Слабка сторона | Коли корисний |
|---|---|---|---|
| Manager + specialists | centralized goal ownership | manager bottleneck/context | large goal with bounded subtasks |
| Handoff | specialist autonomy | control/context transition complexity | distinct specialist conversation |
| Parallel workers | breadth/latency | cost/coordination | independent lanes |
| Critic/verifier | catches blind spots | extra cost, correlated bias | high-risk/uncertain output |
| Git commit handoff | immutable artifact identity | external system complexity | asynchronous result transfer |
| Hybrid retrieval | exact + semantic recall | infra/tuning | measured lexical/semantic misses |
| Graph retrieval | global relationships | expensive ingestion | recurring corpus-global questions |

## Підтверджений факт

OpenAI/GitHub/Git офіційні docs підтверджують конкретні orchestration, webhook/auth/commit та sparse/partial-clone mechanisms.

## Приклад іншої системи

Anthropic Research, SWE-agent, GraphRAG, Reflexion, Self-Refine та debate є зовнішніми examples/research, а не NovaStory implementation claims.

## Рекомендація для задуму

Запозичувати вузькі patterns: exact-SHA handoff, bounded delegation, evaluator loop, version pinning, staged retrieval complexity. Не копіювати чужу повну architecture.

## Припущення

Три ролі задуму можуть використовувати ці patterns без прямого architectural equivalence з зовнішніми systems.

## Ризик

Architecture-by-analogy: скопіювати те, що вирішує проблему іншої системи, якої тут немає.

## Відкрите рішення

Які саме patterns переходять з research у product design, вирішує власник після pilot evidence.
