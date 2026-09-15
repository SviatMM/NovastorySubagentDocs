# Implemented vs Planned

Audit source: `SviatMM/novastory@2ca6ad9a38ee400b3b449dc1994e9ea66a5ca8b1`

## Implemented

- typed renderer-to-host command boundary;
- durable Chat/Project/runtime state;
- Responses and Codex Chat adapters;
- exact model/runtime discovery and selection;
- Chat-only host policy;
- durable runtime receipts, cancel and interruption handling;
- revisioned Project instructions and run snapshots;
- bounded text Resources and attachments;
- scoped Project Source reads;
- built-in AgentTemplate and ProjectAgent identity/add/list behavior;
- Chat output blocks;
- local backup/restore.

## Partial

- ProjectAgent identity exists but Agent execution does not;
- terminal/change-review data contracts exist while terminal execution is unavailable;
- change review is present without a current Chat producer;
- workspace audit is narrower than a general action audit system.

## UI without backend execution

- Automation builder interactions;
- unavailable permission choices;
- image/voice placeholders.

## Planned or documented, not implemented

- general model-callable NovaStory actions;
- Agent instruction revision lifecycle;
- Agent memory/documentation lifecycle;
- Mission/Job/checkpoint execution;
- persistent result destinations/references;
- generic action approval lifecycle;
- Smart Subagent result handoff;
- Knowledge engine;
- Automation engine;
- broader external connectors;
- account/cloud identity and sync;
- remote executor model.
