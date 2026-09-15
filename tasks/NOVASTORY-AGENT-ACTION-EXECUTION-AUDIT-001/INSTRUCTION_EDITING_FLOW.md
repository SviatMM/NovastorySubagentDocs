# Instruction Editing Flow

## Current state

Project instructions are implemented. They live in `project_settings`, use revision checks, and are copied into the immutable context snapshot of an admitted Project Chat run.

Built-in AgentTemplates also contain instruction text. Adding a template to a Project copies that text into the new ProjectAgent row. A revisioned AgentInstructionSet lifecycle and a ProjectAgent instruction-edit command were not found.

## Future two-record contract

For NovaStory Agent and NovaStory Chat, keep two logical records:

1. base role/configuration;
2. Owner instructions.

NovaStory should track stable record IDs, current revisions, immutable history and provenance. Each run should snapshot the exact revisions used before the first model response.

Attachment-only first messages must still load the instruction pair first. Current Chat submit requires non-empty text, so attachment-only first send is not implemented.

## Future edit flow

```text
model proposes an instruction change
-> user sees target and old/new difference
-> host checks target and expected revision
-> accepted edit creates a new immutable revision
-> UI reports success or conflict
```

Product enforcement rules remain separate from editable instruction text.

Rollback should select/create a new current revision based on an older revision rather than deleting history.

## Open decisions

- physical storage for Agent and Chat instruction records;
- global versus per-Chat overrides;
- AgentTemplate defaults versus ProjectAgent overrides;
- external file import/export versus authoritative binding;
- behavior when instructions change during active long-running work.
