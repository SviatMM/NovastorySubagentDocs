# Owner Decisions

The audit does not make these product decisions on the Owner's behalf.

## Action gateway

- Which NovaStory operations should ever be model-callable?
- Should NovaStory Chat remain conversation-only by default while ProjectAgent receives a separate action-capable runtime?
- What is the first safe action family to admit: read-only Project queries, NovaStory record updates, or another narrow slice?

## Identity and scope

- What principal represents the signed-in Owner, Chat runtime, ProjectAgent and child run once account identity exists?
- How should Project-level grants be represented and inherited?
- Which actions require explicit user confirmation versus a stored bounded policy?

## Instructions

- Physical authority for the two Agent instruction records and two Chat instruction records.
- Whether repository files are authoritative sources, exports, or imported materials.
- Global Chat instructions versus per-Conversation overrides.
- AgentTemplate defaults versus ProjectAgent overrides and propagation.
- Policy for instruction changes during active long work.

## Long work

- First ProjectAgent Mission slice and its minimum checkpoint/resume promise.
- ResultDestination technology and review/publication flow.
- Local-only versus Home Mac versus future cloud placement for the first long-running workload.

## Smart Subagent

- Confirm dedicated result repository as the only Smart Subagent publication target.
- Choose task branch naming/base rules and whether one task may have multiple result attempts.
- Decide the exact acceptance gate before NovaStory treats a result commit as trusted input.
- Decide whether local NovaStory, future cloud NovaStory, or both receive repository notifications.

## Recommendation for next design step

Define one narrow `model_action.request -> host operation -> model_action.result` contract for **read-only NovaStory/Project information first**. This proves action discovery, actor/scope validation, receipt persistence and model continuation without mixing the problem with file mutation, external systems, Agent Missions or cloud identity all at once.
