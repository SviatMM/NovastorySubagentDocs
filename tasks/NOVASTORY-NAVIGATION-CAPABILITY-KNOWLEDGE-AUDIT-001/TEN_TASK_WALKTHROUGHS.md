# Ten Task Walkthroughs

## 1 “Що зараз працює в чаті?”
Main map → Chat route → workspace current-state → frontend matrix → code if needed. Do not read cloud/automation/admin/knowledge. Agent can explain live text Chat, lifecycle, text attachments, model config, typed outputs. It cannot call image/voice/terminal/write/broader permissions/Mission complete.

## 2 “Створи проєкт”
Main map → Product/Projects → PROJECT_MODEL → frontend matrix → createProject/projects.rs. Do not read Auth/Knowledge/Automation. Human UI has live host path; Chat/Agent can explain but cannot truthfully say “I created it”.

## 3 “Додай агента до проєкту”
Main map → Agents/Models → engine integration → workspace current-state → project_workspace proof. Human UI can add/reuse ProjectAgent identity; no Mission/memory/subagent/publication can be claimed.

## 4 “Зміни інструкцію агента”
Agents route → engine integration User instructions → DATA_OWNERSHIP. If **Project instructions**, update_project_settings is live for UI. If **ProjectAgent AgentInstructionSet**, full lifecycle not built. If **template/account defaults**, U-07 open. Do not substitute Project setting mutation for Agent-specific instruction.

## 5 “Поясни, які моделі доступні”
Agents route → live list_runtime_models → current-state F2. Historical model table is not current availability. Runtime/provider authentication may become dependency.

## 6 “Налаштуй автоматизацію”
Automation route → AUTOMATIONS → frontend matrix. Local draft builder exists. Integrations/security only if an external action appears. Cannot claim saved/run/scheduled/background Automation.

## 7 “Підключи пошту”
Automation/Integrations route → frontend matrix Mail/Connections → integrations future contract. No mail connector exists; do not jump to generic cloud Auth and pretend connection is possible. No read/send.

## 8 “Знайди результат попередньої роботи”
If Chat result: Chat route → list/open Conversation → output blocks/transcript. Search palette helps loaded Conversations only. If Agent result: Agents → Files/Resources → future ResultReference/ResultDestination, currently NOT_BUILT. Do not invent global result index.

## 9 “Запусти Smart Subagent”
Agents route → U-19 → engine integration research/subagent boundary. Exact repo search found no audited feature. Can explain future boundary; cannot launch/connect/script UI or report execution. Later Owner prototype discussion updates intent only.

## 10 “Віднови роботу після переривання”
Chat: current-state/P1-P4 → preserve committed partial output, mark interrupted, explicit continuation, no automatic retry/same ephemeral engine-thread claim. Shell draft: bounded localStorage best effort only. ProjectAgent Mission: checkpoint design accepted but NOT_BUILT. Database restore is a separate recovery route with explicit restore/source-grant rules.