# Sources

**Дата перевірки всіх веб-джерел:** 2026-09-15.

Пріоритет надано офіційній документації, першоджерелам, відкритим репозиторіям і науковим роботам. Кількість джерел не використовується як заміна якості.

## GitHub / Git

| # | Назва | Посилання | Що підтримує | Тип використання |
|---|---|---|---|---|
| G1 | Validating webhook deliveries | https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries | HMAC SHA-256 через `X-Hub-Signature-256`, secret, safe comparison | підтверджений факт |
| G2 | Webhook events and payloads | https://docs.github.com/en/webhooks/webhook-events-and-payloads | headers, push `ref`, `before`, `after`, commit payload | підтверджений факт |
| G3 | Best practices for using webhooks | https://docs.github.com/en/webhooks/using-webhooks/best-practices-for-using-webhooks | event/action checks, delivery IDs, duplicate/redelivery handling | підтверджений факт |
| G4 | About authentication with a GitHub App | https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/about-authentication-with-a-github-app | app/installation/user authentication model | підтверджений факт |
| G5 | REST API endpoints for GitHub Apps | https://docs.github.com/en/rest/apps/apps | installation access tokens, repository/permission scoping, token lifetime | підтверджений факт |
| G6 | Choosing permissions for a GitHub App | https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/choosing-permissions-for-a-github-app | least-privilege permission selection | підтверджений факт |
| G7 | Permissions required for fine-grained PATs | https://docs.github.com/en/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens | fine-grained token permissions per endpoint | підтверджений факт |
| G8 | About protected branches | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches | branch protection, signed commits/status constraints | підтверджений факт |
| G9 | Available rules for rulesets | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets | signed commits та інші repository rules | підтверджений факт |
| G10 | Troubleshooting required status checks | https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-required-status-checks | checks/status relation to current commit/merge gates | підтверджений факт |
| G11 | REST API endpoints for check runs | https://docs.github.com/en/enterprise-cloud@latest/rest/checks/runs | check run прив'язується до конкретного git reference/commit; write checks via GitHub Apps | підтверджений факт / приклад |
| G12 | REST API endpoints for commits | https://docs.github.com/en/rest/commits/commits | commit comparison/files and exact commit operations | підтверджений факт |
| G13 | git-sparse-checkout | https://git-scm.com/docs/sparse-checkout | working tree може містити subset paths | підтверджений факт |
| G14 | Partial clone design/documentation | https://git-scm.com/docs/partial-clone | object filtering/on-demand fetch | підтверджений факт |
| G15 | git-clone | https://git-scm.com/docs/git-clone | clone filtering/sparse options context | підтверджений факт |

## OpenAI

| # | Назва | Посилання | Що підтримує | Тип використання |
|---|---|---|---|---|
| O1 | OpenAI Model Spec, 2026-08-18 | https://model-spec.openai.com/2026-08-18.html | authority levels, conflicts, same-level later instructions, untrusted data, clear/non-sycophantic behavior | підтверджений факт |
| O2 | OpenAI Agents SDK — Multi-agent orchestration | https://openai.github.io/openai-agents-python/multi_agent/ | manager, handoffs, code orchestration, parallel independent agents, evaluator loops | підтверджений факт / приклад |
| O3 | OpenAI Agents SDK — Guardrails | https://openai.github.io/openai-agents-python/guardrails/ | input/output/tool guardrail concepts | підтверджений факт / приклад |
| O4 | OpenAI Agents SDK — Tracing | https://openai.github.io/openai-agents-python/tracing/ | tracing spans/observability for agent workflows | підтверджений факт / приклад |
| O5 | OpenAI Agents SDK — Testing | https://openai.github.io/openai-agents-python/testing/ | testing agent behavior/workflows | підтверджений факт / приклад |
| O6 | OpenAI API — Using the latest model | https://developers.openai.com/api/docs/guides/latest-model | outcome-oriented prompting, current model behavior guidance, delegation/verification/style guidance | підтверджений факт / основа рекомендації |
| O7 | OpenAI API — Create a response | https://developers.openai.com/api/reference/cli/resources/responses/methods/create | prompt reference can include `id`, variables, version | підтверджений факт / приклад version pinning |
| O8 | OpenAI API — Vector stores search | https://developers.openai.com/api/reference/python/resources/vector_stores/methods/search | semantic chunk search, filters, ranking options/query rewrite | підтверджений факт / приклад retrieval |
| O9 | OpenAI API — Evals create | https://developers.openai.com/api/reference/java/resources/evals/methods/create | evaluation definitions, data source/testing criteria | підтверджений факт / приклад eval infrastructure |
| O10 | OpenAI API — Graders | https://developers.openai.com/api/reference/ruby/resources/graders/subresources/grader_models | multiple grader types/scoring mechanisms | підтверджений факт / приклад eval infrastructure |

## Реальні agent/retrieval systems

| # | Назва | Посилання | Що підтримує | Тип використання |
|---|---|---|---|---|
| E1 | Anthropic — How we built our multi-agent research system | https://www.anthropic.com/engineering/multi-agent-research-system | lead + parallel subagents, breadth benefit, coordination/token costs, delegation heuristics | приклад іншої системи |
| E2 | Anthropic — Building effective agents | https://www.anthropic.com/engineering/building-effective-agents | workflows vs agents, parallelization/multiple perspectives, simplicity trade-offs | приклад іншої системи / основа рекомендації |
| E3 | Anthropic — Demystifying evals for AI agents | https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents | outcome-oriented agent evaluation and eval design | приклад іншої системи |
| E4 | SWE-agent — Background / Agent-Computer Interface | https://github.com/princeton-nlp/SWE-agent/blob/main/docs/background/index.md | ACI/tool interface design for software agents | відкритий репозиторій / приклад |
| E5 | Microsoft GraphRAG documentation | https://microsoft.github.io/graphrag/ | graph/community retrieval for global corpus questions | приклад іншої системи |
| E6 | Microsoft GraphRAG repository | https://github.com/microsoft/graphrag | open implementation; indexing/cost complexity context | відкритий репозиторій / приклад |
| E7 | Elastic — Hybrid search | https://www.elastic.co/docs/solutions/search/hybrid-search | lexical + vector retrieval; rank fusion | підтверджений факт / приклад |
| E8 | Elastic — Ranking and reranking | https://www.elastic.co/docs/solutions/search/ranking | first-stage candidates and reranking concepts | підтверджений факт / приклад |

## Наукові роботи

| # | Назва | Посилання | Що підтримує | Тип використання |
|---|---|---|---|---|
| P1 | Reflexion: Language Agents with Verbal Reinforcement Learning | https://arxiv.org/abs/2303.11366 | feedback/reflection memory can improve subsequent attempts on evaluated tasks | науковий приклад |
| P2 | Self-Refine: Iterative Refinement with Self-Feedback | https://arxiv.org/abs/2303.17651 | generate-feedback-refine loop improves results in reported tasks | науковий приклад |
| P3 | Improving Factuality and Reasoning in Language Models through Multiagent Debate | https://arxiv.org/abs/2305.14325 | multiple model instances/debate can improve selected reasoning/factuality evaluations | науковий приклад |
| P4 | Self-Preference Bias in LLM-as-a-Judge | https://arxiv.org/abs/2604.22891 | risk of self-preference bias in LLM judges | науковий факт / risk evidence |

## Які рекомендації спираються на які джерела

### Exact-SHA GitHub handoff

G1–G3, G11–G12 підтверджують webhook authentication/delivery та commit-specific primitives. **Рекомендація цього дослідження:** candidate identity = exact commit SHA; branch лише routing pointer.

### GitHub App як довгостроковий service auth

G4–G7 підтримують scoped app/token models. **Рекомендація:** GitHub App для stable unattended runtime; narrow fine-grained PAT допустимий як prototype choice.

### Signed commits / protected canonical branch

G8–G10. **Рекомендація:** сильніше застосовувати на canonical/promotion boundary, не підміняти ними result validation.

### Local bounded checkout

G13–G15. **Рекомендація:** partial/sparse local continuation за exact accepted SHA.

### Два instruction-файли і precedence

O1 підтримує instruction authority/conflict principles. O7 є прикладом prompt version pinning. **Рекомендація:** base first, owner second, immutable versions, explicit override zones, receipt pinning.

### Agent orchestration і selective multi-agent

O2–O5, E1–E3, P1–P3. **Рекомендація:** manager-owned goal, bounded parallelism, critic/verifier only where value > coordination cost, explicit stop rules.

### Natural Chat

O1, O6. **Рекомендація:** observable style behaviors та evals замість лозунгу «говори природно».

### Selective knowledge

O8, E5–E8. **Рекомендація:** start with indexes/lexical metadata; semantic → hybrid → rerank → graph only after measured failures.

### Evaluation

O5, O9–O10, E3, P4. **Рекомендація:** deterministic integrity + task tests + model graders + blind human/owner review; judge model не є єдиним truth source.

## Важливе обмеження джерел

Жодне джерело в цьому файлі не використовувалося як доказ фактичної реалізації NovaStory. Репозиторій NovaStory навмисно не досліджувався відповідно до межі завдання.
