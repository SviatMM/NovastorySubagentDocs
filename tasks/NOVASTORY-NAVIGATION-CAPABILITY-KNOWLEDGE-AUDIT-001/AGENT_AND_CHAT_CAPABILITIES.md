# Agent and Chat Capabilities

Source pin: `SviatMM/novastory@2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`.

## Three different actors

1. **Human UI** can call many typed host operations: create/open/rename Project, rename/move/trash Chat, connect local ProjectSource, save Project settings, manage provider credential, backup/restore.
2. **Current Chat model** runs under `chat_only`. It receives bounded admitted text context and may return text/typed Chat output, but gets no generic NovaStory-command tool, shell, filesystem write, dynamic network tool, MCP/plugin, delegated agent or permission escalation.
3. **ProjectAgent** identity is live: templates, Project/domain identity, list/add/reuse/conflict. Durable Mission/Job execution, AgentMemory contents, revisioned AgentInstructionSet, subagents and publication/results are not built.

## Current Chat can

- use durable Conversation history assembled by the host;
- receive admitted Project instructions/text Resources and text attachments;
- execute the exact selected authenticated adapter/model with snapshotted reasoning/service-tier/permission configuration;
- stream/cancel and preserve partial/interrupted state;
- produce supported Chat output blocks;
- explain an unavailable action and propose a manual path.

## Current Chat cannot

- call product mutations merely because Tauri commands exist;
- run shell/terminal or write repositories;
- browse arbitrary local files;
- use web/MCP/plugins/delegation through the admitted Chat runtime;
- connect/send Mail or Calendar;
- save/run/schedule Automation;
- edit Knowledge/Tasks because those systems are not built;
- read Keychain secrets.

## Current ProjectAgent

**Live subset:** built-in versioned AgentTemplates; Project → Agents → Add Agent; durable Project/domain identity; template provenance; uniqueness/reuse/conflict.

**Missing:** openable Mission surface, Mission/Job/Attempt execution, memory contents, full AgentInstructionSet lifecycle, AI OS definition binding, ContextManifest, command/network/write/delegation policy, subagent staging, final documentation publication, ResultDestination/ResultReference, handover/archive/remove.

| Request | Human UI | Chat model | ProjectAgent |
|---|---|---|---|
| Create Project | live host path | explain only | no execution |
| Rename/move/trash Chat | live | explain only | no execution |
| Add ProjectAgent | partial live identity flow | explain only | identity target, no self-run |
| Change Project instructions | live | consumes next admitted revision, cannot save | no Mission execution |
| Change Agent-specific instructions | not complete | explain only | not built |
| Read Project file in inspector | bounded UI read | not a dynamic tool | not executable |
| Write Project file | no admitted current path | no | no |
| Terminal command | executor absent | no | no |
| Run Automation | engine absent | no | no |
| Backup/restore | human host operation | no | no |
| Run Smart Subagent | no audited implementation | no | no |

Future action tools must bind `principal + typed operation + resource + capability + scope + policy generation + approval/effect class + receipt`. Natural-language intent is never authority.