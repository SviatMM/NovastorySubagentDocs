# Selective knowledge loading

## Проста суть

Не треба кожному Agent перед роботою видавати всю бібліотеку. Спочатку він бачить карту, потім відкриває потрібний район, потім конкретний документ і лише потрібний шматок. Складний vector/graph search потрібен лише тоді, коли проста карта реально починає промахуватись.

## 1. Рекомендована v1-ієрархія

```text
ROOT_INDEX
  → DOMAIN_INDEX
    → CAPABILITY_INDEX
      → CANONICAL_DOCUMENT
        → SECTION / CHUNK
```

### ROOT_INDEX

Дуже короткий: великі напрями, one-line descriptions, pointer на domain index, status/version.

### DOMAIN_INDEX

Наприклад `agents`, `projects`, `knowledge`, `automation`, `auth`, `files`, якщо такі напрями існуватимуть у canonical product map. Самі назви тут лише illustrative.

### CAPABILITY_INDEX

Дає конкретні capabilities, canonical source, dependencies та коротке «коли читати».

### Document/section

Agent отримує не весь документ автоматично, а потрібну секцію + enough surrounding context.

## 2. Metadata про джерело

Мінімально корисні поля:

- stable `source_id`;
- title/path/URI;
- domain/capability;
- one-sentence summary;
- authority class;
- status: `current | draft | historical | superseded`;
- version/commit/hash;
- updated_at;
- owner/source;
- tags/aliases;
- dependencies;
- supersedes/superseded_by;
- section anchors;
- retrieval notes: when relevant / when not relevant.

Це дозволяє відсіювати неправильний status ще до дорогого semantic search.

## 3. Звичайний покажчик

**Перевага:** найпрозоріший, cheap, deterministic routing. Добре працює, коли taxonomy невелика й назви capabilities стабільні.

**Недолік:** потребує догляду; user може сформулювати запит словами, яких у index немає.

**V1:** обов’язково.

## 4. Пошук за словами / exact search

Шукати спершу по:

- IDs;
- titles;
- aliases;
- tags;
- summaries;
- headings.

**Перевага:** точний для named entities, IDs і technical terms; зрозуміло, чому source знайдено.

**Недолік:** слабший на paraphrases/synonyms.

**V1:** так.

## 5. Semantic search

**Підтверджений факт.** Vector search системи знаходять semantically related chunks; OpenAI vector store search підтримує query, ranking options і metadata filters.

**Додавати коли:** evals показують recall misses через paraphrase/synonyms, які index/keyword не ловлять.

Не додавати лише тому, що «в усіх RAG є embeddings».

## 6. Hybrid search

Поєднання lexical + semantic корисне, коли corpus містить і точні назви/ID, і природні paraphrases.

**Підтверджений факт.** Elastic описує hybrid search як поєднання full-text та vector retrieval і рекомендує Reciprocal Rank Fusion як один зі способів fusion.

**Додавати коли:** lexical misses intent, semantic іноді пропускає exact identifiers, а evals показують measurable combined benefit.

## 7. Reranking

Перший retrieval збирає candidates дешево; reranker дорожче оцінює невеликий top-k.

**Додавати коли:** recall вже добрий, але top results шумні й agent регулярно читає зайві/менш авторитетні chunks.

Reranking не виправляє погану source authority metadata. Він лише краще сортує candidates.

## 8. Graph of relationships

Graph корисний для питань на кшталт:

- «які capabilities залежать від цієї policy?»;
- «що зміниться, якщо замінити цей component?»;
- global corpus relationship/summary questions.

**Приклад іншої системи.** Microsoft GraphRAG будує graph/community hierarchy, щоб краще відповідати на global questions по corpus, де vanilla chunk RAG може бути слабким. Сам проєкт попереджає, що indexing має вартість і це складніша система.

**Рекомендація:** не v1. Вводити лише якщо з'явиться recurring class глобальних relation questions, який index + hybrid + metadata стабільно провалює.

## 9. Active knowledge manifest

На кожен run/conversation підтримувати compact manifest:

```text
source_id
version/hash
section(s)
reason_loaded
authority/status
loaded_at_turn/topic
```

Це дає:

- dedupe;
- provenance;
- можливість prune;
- recovery після compaction;
- розуміння, чому конкретне knowledge було в контексті.

## 10. Усунення повторів

Dedup key: `source_id + version/hash + section_id`.

Якщо та сама секція вже активна, не додавати її вдруге лише тому, що другий query теж її знайшов. Якщо нова версія supersedes стару, old content прибирається з active set, але provenance history зберігається.

## 11. Зміна теми

Коли topic materially змінюється:

1. оновити current topic/goal;
2. зберегти cross-topic facts/decisions, які реально потрібні;
3. prune bulky chunks старої теми;
4. root/domain route для нової теми;
5. load only new relevant sections;
6. зберегти manifest history, щоб можна було повернутись.

Не треба видаляти всю conversation memory. Треба відділяти compact state від bulky source content.

## 12. Відновлення після context compaction

Перед/під час compaction зберігати compact recovery state:

- current goal/task contract;
- accepted decisions;
- open questions;
- important artifact/result SHAs;
- source IDs + exact versions + sections;
- active constraints;
- next action.

Після відновлення system rehydrates sources за IDs/versions, а не покладається на summary, що приблизно переказує source.

## 13. Що робити, якщо нічого не знайдено

Agent не вигадує knowledge.

Flow:

1. перевірити exact aliases/tags;
2. broaden lexical query;
3. якщо доступний semantic layer — semantic fallback;
4. перевірити dependencies/related domain;
5. якщо source немає — позначити knowledge gap і працювати з external research/owner clarification відповідно до дозволів.

## 14. Якщо знайдено забагато

Фільтри перед model context:

- status/authority;
- domain/capability;
- exact dependencies;
- version/freshness;
- query relevance;
- section-level retrieval;
- top-k budget.

Після цього, за потреби, reranking.

## 15. Якщо джерела суперечать

Не обирати просто higher semantic score.

Порядок:

1. authority;
2. explicit status `current/superseded`;
3. version/freshness;
4. scope applicability;
5. direct evidence;
6. якщо конфлікт лишається — surface it as unresolved.

## 16. Пропонований escalation ladder

### Stage 0
Manual/index routing + direct source reads.

### Stage 1
Exact/keyword search over structured metadata and headings.

### Stage 2
Semantic search with metadata filters when synonym/paraphrase misses measured.

### Stage 3
Hybrid lexical+semantic when both exact identifiers and intent search matter.

### Stage 4
Reranking when candidate quality/noise becomes bottleneck.

### Stage 5
Graph/community layer only for recurring global relationship questions.

Кожен перехід має мати failed eval cases, а не лише бажання «зробити RAG серйознішим».

## 17. Метрики

- retrieval recall on known-answer cases;
- authority correctness;
- stale/superseded source rate;
- irrelevant chunk ratio;
- duplicated chunk ratio;
- knowledge tokens per successful task;
- topic-shift prune effectiveness;
- recovery accuracy after compaction;
- latency/cost;
- answer quality delta from added retrieval stage.

## Підтверджений факт

OpenAI vector store search підтримує semantic chunk retrieval, metadata filters і ranking options. Elastic документує hybrid retrieval/ranking. GraphRAG адресує global corpus questions через graph/community representation.

## Приклад іншої системи

GraphRAG є прикладом складнішої retrieval architecture; Elastic hybrid/RRF — прикладом простішого lexical+vector fusion.

## Рекомендація для задуму

V1 = indexes + metadata + exact/keyword + section reads + active knowledge manifest. Semantic/hybrid/rerank/graph тільки через measured failure trigger.

## Припущення

Майбутній knowledge layer матиме stable source IDs/versions і section-addressable content.

## Ризик

Over-retrieval, stale sources, semantic false positives, duplicate context і складний graph ingestion можуть з'їсти виграш від «розумнішого» пошуку.

## Відкрите рішення

Owner пізніше має визначити canonical metadata schema, storage/index engine і допустимі latency/cost budgets після v1 evals.
