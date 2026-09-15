# START HERE — NovaStory navigation/capability audit

## Audit identity

- Task ID: `NOVASTORY-NAVIGATION-CAPABILITY-KNOWLEDGE-AUDIT-001`.
- Source repository: `SviatMM/novastory`.
- Audited source branch: `main`.
- Audited source commit: `2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1` (`docs: prepare account and cloud contract decisions`, 2026-09-13).
- Result repository: `SviatMM/NovastorySubagentDocs`.
- Result branch: `audit/NOVASTORY-NAVIGATION-CAPABILITY-KNOWLEDGE-AUDIT-001`.
- Audit date: 2026-09-15.
- Mode: read-only audit of NovaStory; only this `tasks/NOVASTORY-NAVIGATION-CAPABILITY-KNOWLEDGE-AUDIT-001/` result tree is written.

The prompt supplied `<TASK_ID>` and `<RESULT_REPOSITORY>` placeholders rather
than concrete values. The audit therefore uses the explicit working assumption
above: this Task ID and the separately accessible `SviatMM/NovastorySubagentDocs`
repository. This assumption affects delivery naming only, not NovaStory product
truth.

GitHub can prove the remote `main` commit but cannot inspect the Owner's local
uncommitted working tree. Therefore this audit is exact for the remote tree at
`2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`. It does **not** claim that an unpushed local checkout is clean or
identical.

## What this package is

This is a retrieval and truth-classification layer over NovaStory's existing
knowledge graph. It deliberately does not replace `docs/HOME.md`, `docs/INDEX.md`,
Firmament MOCs, Owner Logic, backend catalogs, current-state documents, code, or
tests.

Use this hierarchy:

```text
START_HERE.md
  -> NOVASTORY_MAIN_MAP.md                 # short router only
    -> KNOWLEDGE_ROUTES/<direction>.md     # direction router
      -> canonical NovaStory document      # accepted/current meaning
        -> exact section
          -> current-state/code/test proof # only when implementation matters
```

Do not read every route “just in case”. That defeats the entire point and turns
retrieval into document soup.

## Files in this audit

| File | Purpose |
|---|---|
| `NOVASTORY_MAIN_MAP.md` | Minimal top-level intent router. |
| `NAVIGATION_MAP.md` | User-visible paths, screens, controls and actual state. |
| `CAPABILITY_CATALOG.md` | Cross-product capability/status/action matrix. |
| `AGENT_AND_CHAT_CAPABILITIES.md` | What the current Chat model and future/current Agent identity may actually do. |
| `KNOWLEDGE_ROUTES/*` | Direction-specific canonical reading routes. |
| `RETRIEVAL_RULES.md` | Search, source priority, deduplication, topic switch and context recovery rules. |
| `TEN_TASK_WALKTHROUGHS.md` | Concrete minimal-reading walkthroughs for ten requested tasks. |
| `CONTRADICTIONS_AND_STALE_SOURCES.md` | Known conflicts and traps. |
| `MISSING_DOCUMENTATION.md` | Gaps that prevent safe routing or capability claims. |
| `OWNER_DECISIONS.md` | Accepted Owner decisions vs open/newer Owner intent. |
| `SOURCES.md` | Audited source ledger and authority levels. |
| `RESULT.json` | Machine-readable audit summary. |

All eight requested `KNOWLEDGE_ROUTES` are retained because they have genuinely
different dependency boundaries. They are small indexes, not duplicate
specifications.

## Truth vocabulary

Use two independent axes.

**Product/truth status**

- `OWNER_STATED` — direct Owner requirement, not necessarily accepted design.
- `ACCEPTED` — Owner-approved product/architecture/security direction.
- `PROPOSED` — candidate mechanics, not canonical.
- `OPEN` — material unresolved choice.
- `HORIZON` — future area, not current scope.

**Implementation status**

- `WORKS` — active UI/host path with current implementation evidence.
- `PARTIAL` — bounded implementation exists but the named capability is incomplete.
- `UI_ONLY` — user can interact with renderer state, but no durable/server/external effect is claimed.
- `BOUNDARY` — intentionally unavailable; UI/API may expose the boundary/reason.
- `NOT_BUILT` — no required execution path.
- `UNKNOWN` — evidence is insufficient or conflicts without a newer resolution.

A visible control, TypeScript wrapper, planned API name, accepted target, or
completed documentation goal is never enough to promote a capability to
`WORKS`.

## Critical result in one paragraph

The current product has a substantial **human-operated** desktop shell and
bounded live host operations for Chats, Projects, local ProjectSources,
Project-level instructions/resources, runtime discovery/Chat execution,
OpenAI credential custody, backup/restore, inspectors and ProjectAgent
identity/add/reuse. But the current **AI principal** is intentionally narrower:
`chat_only` can reason over admitted Chat context and return text/typed Chat
outputs, while shell, filesystem writes, generic commands, dynamic tools,
network tools, MCP/plugins and delegation are denied. A `ProjectAgent` can be
created as durable identity but cannot yet execute Missions, maintain implemented
memory, publish its documentation, or launch subagents. Mail, Calendar external
effects, Automation execution, account/Auth/cloud/sync, second-computer runner,
general audit ledger, Knowledge and Tasks/Goals remain UI-only, boundary, open,
or not built depending on the item.

## Start rule

For any new request:
1. open `NOVASTORY_MAIN_MAP.md`;
2. choose exactly one primary route;
3. open a secondary route only when a named dependency becomes material;
4. use `RETRIEVAL_RULES.md` before asserting “works”, “can change”, or “ran”;
5. never call an action complete unless the active principal had a real admitted
   execution path and there is a success receipt/state change.
