# Contradictions and Stale Sources

| Trap | Older/conflicting claim | Newer/current truth | Rule |
|---|---|---|---|
| ProjectAgent | AI_RUNTIME_AGENTS old baseline says static Agents/no ProjectAgent records | workspace current-state/DATA_OWNERSHIP: template + ProjectAgent identity/add/reuse/conflict live | old doc for product semantics, newer current-state for implementation |
| Codex effort | CHAT_SESSION_MODEL says effort/tier dropped/hardcoded low | current workspace state: exact effort/tier validated/snapshotted/sent | prefer successor current-state/code |
| Integrations | systems/INTEGRATIONS verified list says folder + Responses only | coverage/current-state also has Codex App Server and ProjectSources | current-state/coverage wins |
| Glass | older text implies active Glass option | current guide/matrix: Light/Dark; legacy glass normalizes to Light; full Glass post-1.0 | latest promoted overlay wins |
| Voice | accepted spec describes desired microphone flow | current state: explanation only, no capture/transcription | target ≠ implementation |
| Project creation | spec describes broader source setup | matrix: identity/settings live, optional one native source; broader flow partial | live matrix/current-state wins |
| Permission labels | Ask/Approve/Full names visible | only `chat_only` admitted; broader U-08 open | label never grants authority |
| Search | spec language can sound broad | matrix: commands + loaded Projects/Conversations | no Knowledge/files/web claim |
| Terminal | typed API/UI exists | no PTY executor; API-only/denial | API ≠ execution |
| Undo | RepoChangeSet/undo types exist | no current Chat producer, inverse writes absent | review substrate ≠ mutation |
| `status: current` | dated implementation prose inside current docs | newer reconciliation may supersede only that paragraph | metadata alone not freshness |
| Local worktree | historical docs mention dirty local checkpoints | GitHub audit sees remote pin only | local dirty/clean is unobservable |
| Supabase primary/recovery | planning language sounds operational | Stage04/cloud route says docs-only, NOT_STARTED, no replication/failover | always attach NOT_BUILT |
| Account Auth | audited U-03 leaves first method open | later Owner message wants Google/GitHub/Apple | post-commit Owner intent, not code |
| Smart Subagent | U-19 open/no repo hit | later Owner explores prototype | new intent ≠ implementation |
| Model table | dated Codex discovery table | runtime discovery is current authority | query live capability |
| Admin | detailed ADM responsibilities | frontmatter says docs-only/not-started, U-17 open | concept ≠ screen/API |
| Knowledge/Tasks | detailed product outcomes | schema/UI/authority open | design prose ≠ module |
| Stage04 DONE | goal Phase DONE | boundary says C2/Auth/account/sync NOT_STARTED | DONE applies only to docs/research scope |

High-risk inference traps: UI control → server action; server API → AI tool; entity row → running Agent; accepted target → implementation; portable state → device grant; connected Resource → ownership/context; rendered result → authorized effect.