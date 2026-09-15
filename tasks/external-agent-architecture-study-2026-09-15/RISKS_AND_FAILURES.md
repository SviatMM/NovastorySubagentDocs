# Risks and failures

## Проста суть

Найбільша небезпека тут не в тому, що модель один раз помилиться. Небезпека, що система оформить помилку як перевірений результат: прийме не той commit, дасть файлу authority інструкції, запустить зайвих агентів, загубить version provenance або скаже «готово» за self-report виконавця.

## 1. GitHub transport / integrity

### R1. Forged webhook

**Failure:** сторонній caller підробляє event.

**Наслідок:** система може почати validation чужого SHA або виконати downstream actions.

**Контроль:** HMAC `X-Hub-Signature-256`, constant-time compare, repo/event/ref validation, no trust before signature pass.

### R2. Replay / duplicate delivery

**Failure:** той самий webhook доставлено повторно або replayed.

**Контроль:** `X-GitHub-Delivery` dedupe + candidate idempotency за repository/task/run/SHA.

### R3. Lost webhook

**Failure:** result commit є, event не дійшов.

**Контроль:** periodic reconciliation/polling unseen SHAs через той самий validator.

### R4. Mutable branch / branch race

**Failure:** branch head змінюється між notification, fetch і review.

**Контроль:** pin exact candidate SHA, never silently switch to current head.

### R5. Force-push / rewritten history

**Failure:** expected branch пересунуто на іншу історію.

**Контроль:** accepted SHA identity immutable; new observation = new candidate/policy event; no normal force-push workflow.

### R6. Wrong repository/ref/task

**Failure:** valid GitHub event, але не з expected context.

**Контроль:** match stable repository identity, expected ref, task/run manifest і allowed path.

### R7. Incomplete result

**Failure:** commit існує, але missing required artifact або invalid RESULT.json.

**Контроль:** deterministic completeness schema before model review.

### R8. Malicious/foreign file

**Failure:** executable, symlink, path traversal, huge archive, prompt-injection Markdown, unexpected file outside task scope.

**Контроль:** path/type/size allowlists, archive policy, symlink/path escape rejection, content treated as untrusted data.

### R9. Secret committed

**Failure:** credential потрапив у Git history.

**Контроль:** pre-accept secret scan; quarantine; no secret echo into model/log; rotate/revoke in real incident. Не використовувати справжні secrets в evals.

### R10. Overprivileged credential

**Failure:** Smart Subagent або orchestration backend отримує зайві repositories/actions.

**Контроль:** least privilege, repo-scoped permissions, short-lived GitHub App installation tokens для stable service runtime; rotation/revocation.

## 2. Multi-agent orchestration

### R11. Agent explosion

**Failure:** children запускають children, «для надійності» росте дерево.

**Контроль:** concurrency/depth/child count/budget caps, spawn reason, child-spawn disabled by default v1.

### R12. Duplicate work

**Failure:** кілька agents читають одні джерела й повертають однакові outputs.

**Контроль:** distinct subtask contracts, dedupe, require diversity reason, telemetry on duplicated-work ratio.

### R13. Shared writer race

**Failure:** паралельні agents змінюють одну branch/workspace.

**Контроль:** one writer per isolated branch/worktree/run, single integrator.

### R14. Correlated error

**Failure:** незалежні на вигляд agents помиляються однаково через model/source framing.

**Контроль:** raw evidence, alternative sources, deterministic tests, targeted different framing/model where risk justifies it.

### R15. Majority vote fallacy

**Failure:** 3 agents погодились → result accepted.

**Контроль:** authority/evidence/test over vote count.

### R16. Endless critic loop

**Failure:** кожен reviewer знаходить ще дрібницю, робота не завершується.

**Контроль:** acceptance criteria, severity threshold, retry budget, diminishing-return stop condition.

### R17. Reviewer self-preference/bias

**Failure:** LLM grader favors familiar style/output.

**Контроль:** deterministic gates, blind pairwise, multi-criteria graders, human review for product/style/high-impact choices.

## 3. Instruction system

### R18. Base/owner conflict

**Failure:** два permanent files дають несумісні правила.

**Контроль:** explicit override zones, pre-activation conflict lint, evals.

### R19. Silent instruction mutation

**Failure:** Chat «трохи виправив» owner rules і active behavior змінилась без provenance.

**Контроль:** proposed diff, immutable new version, explicit activation, run receipt.

### R20. Rollback by regeneration

**Failure:** система просить модель відтворити стару інструкцію з пам’яті.

**Контроль:** immutable version store + active pointer.

### R21. Permissions hidden in prompt

**Failure:** текст каже «не роби X», але tool фактично дозволяє X.

**Контроль:** runtime permission enforcement, scoped credentials, approval gates.

### R22. Instruction bloat

**Failure:** base/owner стають енциклопедією, витісняють task context і множать conflicts.

**Контроль:** permanent-only test; knowledge/examples/history зберігати окремо; prompt token budget.

## 4. Natural Chat

### R23. Mechanical simplicity template

**Failure:** відповідь завжди має «суть/причина/наслідок/приклад», навіть коли це безглуздо.

**Контроль:** rules як defaults with proportionality, eval simple prompts.

### R24. Oversimplification

**Failure:** модель прибирає critical technical term заради «простоти».

**Контроль:** preserve precise names, explain on first use.

### R25. Decorative criticism

**Failure:** Agent шукає вигаданий мінус лише щоб виглядати незалежним.

**Контроль:** critique only material weakness; evidence/consequence required.

### R26. Sycophancy

**Failure:** model automatically agrees with owner idea.

**Контроль:** premise check, strongest material weakness, safer/simpler alternative if real.

## 5. Knowledge system

### R27. Over-retrieval

**Failure:** десятки chunks «про всяк випадок».

**Контроль:** index routing, top-k/token budget, active manifest, prune on topic shift.

### R28. Stale/superseded source

**Failure:** semantic score високий, але document historical.

**Контроль:** authority/status/version filters before relevance ranking.

### R29. Duplicate context

**Failure:** та сама section завантажена кілька разів через різні queries.

**Контроль:** source_id+version+section dedupe.

### R30. Semantic false positive

**Failure:** vector search знаходить схожий, але не applicable document.

**Контроль:** metadata filters, lexical identifiers, rerank/authority, agent citation/provenance.

### R31. Graph complexity without problem

**Failure:** expensive entity extraction/community indexing для corpus, який добре працював з index/keyword.

**Контроль:** introduce GraphRAG only after recurring global-question failures are measured.

### R32. Context compaction loses provenance

**Failure:** summary пам’ятає conclusion, але не source version.

**Контроль:** compact recovery state з source IDs/versions/sections і accepted artifact SHAs.

## 6. Completion / evidence

### R33. Commit exists therefore done

**Failure:** transport success плутається з task success.

**Контроль:** commit only triggers validation; acceptance criteria determine done.

### R34. Subagent self-report accepted

**Failure:** `RESULT.json: success` сприймається як proof.

**Контроль:** result state is claim; parent/validator checks evidence.

### R35. Plan completion mistaken for implementation

**Failure:** опис/документ названо реалізованою функцією.

**Контроль:** separate statuses: proposed/approved/planned/implemented/verified/accepted.

### R36. Verification report mistaken for test

**Failure:** один agent каже «я перевірив», другий цитує це як independent evidence.

**Контроль:** verifier accesses raw evidence or re-executes material checks.

## 7. Operational severity model

Рекомендована класифікація:

- **P0 integrity/security:** wrong artifact accepted, secret leak, permission bypass;
- **P1 correctness:** false completion, missing required result, stale authority source;
- **P2 efficiency:** overdelegation, over-retrieval, excessive cost;
- **P3 style:** awkward language, unnecessary headings.

Higher severity blocks acceptance even if average quality score high.

## Підтверджений факт

GitHub docs підтверджують webhook signature/delivery/ref/SHA mechanics та scoped app permissions; Model Spec відділяє instruction authority від untrusted content; external multi-agent reports описують coordination/token costs.

## Приклад іншої системи

Anthropic multi-agent research показує реальний coordination/cost trade-off; GraphRAG показує складність richer retrieval; research on LLM judges показує bias risk.

## Рекомендація для задуму

Детерміновано закрити integrity risks; multi-agent/retrieval/style risks контролювати budgets, metadata, evals і observability.

## Припущення

Майбутній runtime матиме state store для run candidates, instruction versions та knowledge manifests.

## Ризик

Найгірша комбінація: overprivileged tool + untrusted file interpreted as instruction + acceptance by self-report. Ці три boundaries треба розділити архітектурно.

## Відкрите рішення

Owner має визначити severity/approval tiers, retention policy, authentication strategy і tolerance for cost/latency trade-offs.
