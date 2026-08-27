---
name: clickup-sync
description: Synchronize changed allowlisted ClickUp epics and tasks.
schedule: "30 9 * * *"
timezone: "<team timezone>"
---

# Task: ClickUp Sync

Read `AGENTS.md`, preflight with `git status --short`, then read
`integrations/clickup.md` once. Each registered entry is one epic or standalone task.
This task is read-only upstream and never writes Work State.

Required connector operation: `clickup_get_task` with summary, detailed, and subtasks
support, or its configured equivalent.

## Epic workflow

For each registered epic:

1. Parse its native task ID, destination, `Last source sync`, and `Remote modified`.
   Convert timestamps to the connector's comparison unit, usually Unix milliseconds.
   A bare or incomplete entry is unmapped: report and skip it.
2. Request the epic summary with `subtasks=true`. This single call must return the
   epic and every subtask's `date_updated` and `orderindex`.
3. Build the changed set:
   - missing `Remote modified` → epic and all subtasks;
   - epic `date_updated` newer than the anchor → epic;
   - subtask `date_updated` newer than the anchor → that subtask;
   - subtask absent from the local file or locally marked removed → that subtask;
   - local subtask with a native ID absent from the complete remote list and not
     already marked removed → removed.
4. If the set is non-empty, fetch detailed data for its members in one bounded
   parallel batch of at most 25. Retry one timeout once; then skip that member and
   keep old anchors.
5. Patch the local file:
   - epic fields: status, priority, assignee, due date, description, updated date;
   - subtask fields: the same fields plus native ID and `orderindex`;
   - new subtasks: insert at their `orderindex` position;
   - removed subtasks: retain and mark `removed upstream` with the sync date;
   - unchanged fields and local notes: preserve exactly.
6. Only after every required fetch and write succeeds, set `Remote modified` to the
   greatest returned `date_updated` and `Last source sync` to current UTC.

A registered standalone task follows the same flow without subtasks.

## New snapshot shape

When a registered destination does not exist, create it only after a full fetch. Store
the epic or task ID and title, status, priority, assignee, due date, updated date,
description, then a subtask list ordered by `orderindex`. Each subtask stores the same
fields and its native ID. Do not add a generated narrative summary.

## Rules

- ClickUp wins when captured source fields conflict with the local snapshot.
- Never rewrite an unchanged task or reorder subtasks without `orderindex` evidence.
- Never create, edit, assign, close, or comment on an upstream task.
- Keep local-only notes and tasks, but mark and report them as `local-only`; they are
  not ClickUp evidence and must not be cited or uploaded.
- Preserve the registered destination and stable `SRC-CU-*` ID.
- Partial data never advances anchors.
- A successful no-change check advances `Last source sync`, but not `Remote modified`.

## Output

Report each entry as `<id>: N updated, M local-only`, then
`Processed / Skipped / Updated / Errors / Notes`.

## Commit

Never set `user.name` or `user.email` with `git config` or `git config --global`.
Use only the one-shot `git -c` identity below.

Stage only touched task files and edited lines in `integrations/clickup.md`:

```bash
git add -- <touched task files> integrations/clickup.md
git diff --cached --quiet && exit 0
git -c user.name=work-registry-bot -c user.email=bot@work-registry.local \
  commit -m "clickup-sync: <summary>" -m "Automation: work-registry-bot"
```
