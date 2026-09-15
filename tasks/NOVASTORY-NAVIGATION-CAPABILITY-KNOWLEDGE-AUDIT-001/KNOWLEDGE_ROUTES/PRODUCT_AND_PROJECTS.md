# Knowledge Route — Product and Projects

Read: `product/PROJECT_MODEL.md` → `product/CHAT_PROJECT_WORKSPACE_SPEC.md` for accepted target → `backend/CHAT_PROJECT_WORKSPACE_CURRENT_STATE.md` for current implementation → frontend control matrix → `desktop-api`/Rust Project code if execution proof needed → U-01/U-07 for expansion.

Current workspace: **Overview / Conversations / Agents / Repositories / Instructions**. Project identity/list/create/open/rename work. Project settings are revisioned. Bounded local ProjectSources and text Resources work. Overview is partial. Repositories are local bounded read, not remote auth/write/index/watch/process. Agents support template + ProjectAgent identity only.

Removed/deferred Project tabs: Tasks & Goals, Knowledge, generic Activity.

Keep entities distinct: Project; Conversation; ProjectSource (external folder/repo relation + device grant); ProjectResource (bounded text context); broader Resource abstraction; ProjectAgent. A connected source neither transfers content ownership nor gives ambient AI access.