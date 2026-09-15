# Owner decisions

Тут лише рішення, які дослідження не повинно приймати замість власника. Для кожного є просте пояснення, варіанти, наслідки й recommendation.

## D1. Який GitHub credential для першого реального runtime?

**Суть:** чим основна система/Smart Subagent авторизується в result repository.

**Варіанти:**

- fine-grained PAT для owner-only prototype;
- GitHub App одразу;
- інший вузький credential для обмеженого локального сценарію.

**Наслідок:** PAT простіший стартово, але App дає кращу service identity, short-lived installation tokens, permissions і webhooks/checks для стабільного unattended runtime.

**Рекомендація:** prototype може почати з максимально вузького fine-grained PAT; production-like unattended handoff планувати на GitHub App. Protocol exact-SHA має не залежати від цього вибору.

**Статус:** відкрите рішення.

## D2. Direct commit чи PR для result handoff?

**Суть:** чи кожен результат повинен проходити pull request.

**Варіанти:**

- direct commit в isolated run branch;
- PR для кожного run;
- direct run branch → PR лише при promotion у canonical branch.

**Рекомендація:** третій варіант. V1 result transport не ускладнювати PR; canonical/high-risk promotion може використовувати PR/checks.

**Статус:** відкрите рішення.

## D3. Чи вимагати signed commits?

**Суть:** cryptographic provenance author commit.

**Наслідок:** корисно для protected/canonical branches, але додає key/signing operations і не замінює webhook authentication/permissions/content validation.

**Рекомендація:** optional на ephemeral run branches v1; evaluate required signing для canonical protected branch.

**Статус:** відкрите рішення.

## D4. Як активувати зміни owner instructions?

**Варіанти:**

- кожна зміна тільки після explicit Accept;
- minor edits автоматично, material changes explicit;
- staged rollout after evals.

**Рекомендація:** спочатку explicit Accept для всіх behavior-changing changes. Після накопичення telemetry можна окремо визначити safe minor class.

**Статус:** відкрите рішення.

## D5. Що owner instruction має право override?

**Суть:** де preference закінчується й починається invariant/runtime policy.

**Рекомендація:** owner може override style, priorities, product/workflow preferences у дозволених зонах; не може текстом видавати tool permissions, знімати security gates або змінювати evidence/completion integrity без окремої programmatic policy change.

**Статус:** потребує формального переліку override zones.

## D6. Який default max parallel Smart Subagents?

**Варіанти:** 2, 3 або вищий configurable cap.

**Рекомендація:** pilot default `3`, звичайний fan-out 1–3; 4+ потребує recorded reason. Після evals змінити за evidence.

**Наслідок:** нижчий cap зменшує cost/chaos, вищий може допомагати breadth-heavy research.

**Статус:** відкрите рішення.

## D7. Коли незалежний verifier обов’язковий?

**Варіанти:** кожна material task; risk-triggered; тільки owner-requested.

**Рекомендація:** risk-triggered. Mandatory для security/integrity, irreversible/high-impact changes та cases з низькою confidence/contradictory evidence. Для trivial tasks deterministic checks або self-check достатні.

**Статус:** власник має затвердити risk tiers.

## D8. Коли додавати semantic/hybrid retrieval?

**Варіанти:** одразу; після corpus size threshold; після measured retrieval misses.

**Рекомендація:** після measured misses. Corpus size сам по собі не доводить проблему.

**Статус:** відкрите до v1 knowledge eval baseline.

## D9. Чи потрібен GraphRAG/knowledge graph?

**Рекомендація:** не на v1. Розглядати, якщо recurring global relationship/corpus-summary questions стабільно провалюються на index/hybrid approach.

**Статус:** future decision.

## D10. Local-first чи cloud/API-first continuation результату?

**Варіанти:**

- завжди clone locally;
- API/cloud read by exact SHA;
- hybrid.

**Рекомендація:** hybrid: API/cloud для review/research/metadata; sparse local checkout лише коли потрібні build/test/filesystem workflows.

**Статус:** відкрите product/runtime decision.

## D11. Скільки зберігати run branches/results?

**Потрібно визначити:**

- retention window;
- чи видаляється branch після acceptance;
- чи task artifacts лишаються за immutable SHA/tag/reference;
- logs/evidence retention;
- privacy/data deletion requirements.

**Рекомендація:** не видаляти branch/commit одразу до завершення downstream acceptance/audit window. Exact policy залежить від storage/privacy needs.

**Статус:** відкрите рішення.

## D12. Межа між NovaStory Agent і NovaStory Chat

**Суть:** коли user goal переходить у long-running Agent workflow, а коли Chat лишається owner execution context.

**Рекомендація дослідження:** не фіксувати цю межу зовнішнім research без product evidence. Обидва можуть мати shared style/instruction infrastructure, але orchestration responsibility треба визначити за реальними сценаріями й UX.

**Статус:** відкрите продуктове рішення власника.

## D13. Який canonical формат instruction artifacts?

**Варіанти:** Markdown, YAML+Markdown, JSON schema + human text.

**Рекомендація:** спершу вибрати semantic contract і versioning, а serialization після prototype. Для human editing Markdown з невеликим structured header часто достатній; machine fields краще schema-validated.

**Статус:** відкрите рішення.

## D14. Який retry budget?

**Рекомендація:** одна автоматична correction для concrete verifier failure; друга лише при чітко іншому fix path; далі escalate. High-cost/destructive retries окремо.

**Статус:** затвердити після pilot failure distribution.

## D15. Які eval thresholds вважаються прийнятними?

**Рекомендація:** не вигадувати відсотки до baseline. Спершу виміряти current/simple baseline, потім затвердити thresholds окремо для integrity, quality, delegation, naturalness, cost/latency.

**Статус:** відкрите після baseline.

## Підтверджений факт

External docs дають технічні можливості й trade-offs, але не визначають product preferences NovaStory.

## Приклад іншої системи

GitHub Apps/Checks, OpenAI orchestration та Anthropic multi-agent research показують різні viable patterns, жоден не є обов’язковою повною architecture для цього задуму.

## Рекомендація для задуму

Спершу owner затверджує D1–D7, бо вони впливають на v1 protocol. D8–D15 можна закривати по мірі pilot evidence.

## Припущення

Після цього research буде окремий design/implementation stage. Цей пакет сам нічого не реалізує.

## Ризик

Якщо відкрите owner рішення тихо «вирішить» implementer, documentation почне описувати чужі припущення як product intent.

## Відкрите рішення

Усі D1–D15 вище.
