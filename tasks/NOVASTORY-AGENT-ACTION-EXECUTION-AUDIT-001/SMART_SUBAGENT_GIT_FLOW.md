# Smart Subagent Git Flow

## Current state

No implemented NovaStory Smart Subagent GitHub execution path was found at the audited commit. GitHub operations are represented in backend documentation as future connector/action vocabulary, not as a live NovaStory connector.

The flow below is therefore a future contract, not implementation evidence.

## Proposed local NovaStory flow

```text
NovaStory creates task record
-> pins result repository + allowed branch pattern + task path
-> sends task to Smart Subagent through a separately admitted adapter
-> stores external task/run identity
-> waits for a result notification or periodic check
-> receives candidate commit SHA
-> verifies repository, branch and parent expectations
-> inspects changed paths and commit contents
-> accepts only files under tasks/<TASK_ID>/
-> reads task result files
-> records review decision
-> continues NovaStory work using the accepted result reference
```

Local NovaStory may perform the GitHub network interaction through a connector while keeping local project/file access separate.

## Proposed cloud NovaStory flow

A future cloud control plane may receive GitHub notifications directly and persist task/commit receipts. It must not imply that cloud NovaStory can read local-only Project files. Local-only validation remains on an enrolled device when required.

## Notification and fallback polling

Primary path: repository notification identifies repository and candidate commit. Secondary path: bounded periodic check compares the last processed commit with the current allowed branch head.

Both paths converge on the same commit-verification function. Polling must not create a second processing semantics.

## Duplicate protection

Persist a processed key containing at least task ID, repository identity and commit SHA. Re-delivery of the same commit returns the prior decision rather than repeating review work.

## Branch and repository protection

For each Smart Subagent task, NovaStory should pin:

- exact result repository;
- allowed task branch or branch prefix;
- expected base/parent when applicable;
- task directory `tasks/<TASK_ID>/`;
- accepted commit author/source policy if later required.

A notification from another repository or an unexpected branch is rejected before reading its payload as a task result.

## Commit content verification

Before acceptance, verify:

- all changed paths are inside the task directory;
- no unrelated task output changed;
- required result files exist;
- result metadata names the expected task and source baseline;
- referenced NovaStory commit matches the task baseline or is explicitly marked stale;
- generated result contains no unexpected credential material or binary payloads according to the future repository policy;
- commit is reachable from the allowed branch.

## Main NovaStory repository boundary

Smart Subagent should not receive authority to write directly to the NovaStory main branch as part of this result channel. Its output belongs in the dedicated result repository. Any later NovaStory code/document change requires a separate review/admission path.

## Accept / reject

Acceptance stores the verified result commit SHA and task result reference. Rejection stores the reason and leaves NovaStory unchanged. A rejected result may be followed by a new task attempt; it must not mutate the meaning of the rejected commit.

## Gap list

Not currently implemented in NovaStory:

- Smart Subagent task/run records;
- GitHub result connector;
- notification receiver;
- polling worker;
- processed-commit ledger;
- task branch/path verifier;
- result acceptance/rejection records;
- automatic continuation of a ProjectAgent Mission from accepted result.
