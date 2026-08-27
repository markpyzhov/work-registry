---
name: work-state-review
description: Turn corrections and changed sources into current Work State.
schedule: "30 10 * * *"
timezone: "<team timezone>"
trigger: daily; optionally on correction push, at most once per hour
---

# Task: Work-State Review

Read `AGENTS.md` and preflight with `git status --short`. The runner must expose current
`main` and correction branches. Use only integration files, their registered
destinations, `work-states/`, and briefs when inspecting a proposed correction. Never
scan for extra evidence.

## Reading protocol

Read every selected input without sampling:

1. Inventory integration files, selected source destinations, relevant `WS-*` files,
   and Comment files before extracting findings.
2. Read files up to 180 lines in full. For larger Markdown files, list all line ranges
   or headings, then read 100–180-line batches in order. For CSV or tables, use logical
   row ranges.
3. Keep a coverage ledger with file, completed ranges, next range, and findings with
   `SRC-*` plus line, row, or heading. Do not rely on a summary without completing its
   source batch.
4. Read the index and every selected Work State item completely. Chunk a large item,
   branch diff, integration file, or Comment file by the same rule.
5. Before creating an item, search `work-states/` for its `SRC-*`, task IDs, and likely
   title, then fully read every candidate. This avoids duplicating closed or inactive
   work without scanning outside the allowed directory.
6. After the last batch, compare the ledger with the file outline. Missing coverage
   makes that input partial; do not update its review anchor or reconcile unseen text.

## 1. Import correction branches

A correction branch changes only `work-states/` or `briefs/`. List available branch
refs, ignore branches with no changes there, and process the rest one at a time. Use
`git rev-list --reverse main..<branch>` for commit order. A source commit already
named in any `CMT-*` record is processed and skipped.

For each unprocessed commit, oldest first:

1. Record branch, full SHA, Git author name and email, author time, and the exact diff.
   A merge commit, unrelated file change, or unsafe conflict goes to manual triage
   without import.
2. Create `wr-integrate-<short-sha>` from the then-current `main` and run
   `git cherry-pick <full-sha>`. Cherry-pick preserves the human author. On conflict,
   run `git cherry-pick --abort`, switch to `main`, and stop without changing it.
3. Assign the next `CMT-YYYY-MM-###` ID. Add the source metadata and a `Comment:` line
   carrying the correction text to the monthly comment file.
4. Restore the proposed files with
   `git restore --source=main --staged --worktree -- work-states briefs`. Apply the
   Comment through reconciliation instead. Clear corrections update the affected
   `WS-*` item; unclear, unsupported, or conflicting corrections go to manual triage.
   The latest applicable Comment wins while older Comments remain.
5. Validate IDs, citations, index data, and the clean generated form. Commit the
   Comment and regeneration on the integration branch:

   ```bash
   git add -- <touched comments, Work State, index, and brief files>
   git -c user.name=work-registry-bot -c user.email=bot@work-registry.local \
     commit -m "wr-correction: <CMT-ID>" -m "Automation: work-registry-bot"
   git switch main
   git merge --ff-only wr-integrate-<short-sha>
   ```

Delete only the temporary integration branch. Leave the person's branch intact. The
runner publishes the updated `main` after the full task succeeds.

## 2. Review changed sources

1. Read all integration files once. Build the strict allowed set with `SRC-*` IDs,
   destinations, and review anchors. Read the index, relevant `WS-*` files, and current
   comments; during days 1–7, also read the previous month.
2. Apply every Comment that has no `Disposition` line. Clear corrections update Work
   State and gain a short disposition. Unclear Comments stay open in manual triage. A
   clear completion closes its `T###` even when a related task-system item is open,
   unless the Work State task explicitly tracks that exact item.
3. Select a source when its review anchor is missing, says `not reviewed`, or its local
   SHA-256 differs from the recorded hash.
4. Review every selected source with the reading protocol. A partial review keeps its
   old anchor and records completed and next ranges in triage.
5. Extract only requests, outcomes, status, priority, blockers, completion, follow-ups,
   task references, and evidence. Never mutate a source system. A suggested but
   missing task-system item goes to triage. Text marked `local-only` in a synchronized
   task file is not source-system evidence and must not be cited.
6. Reconcile into the matching item. A clear new outcome gets the next `WS-*` ID;
   ambiguity goes to triage. Keep one outcome per item. `S#` and `T###` values are
   append-only, and every factual claim cites an `S#` or `CMT-*`.
7. After full coverage, set
   `Last reviewed revision: YYYY-MM-DD / sha256:<first 12>`. Update the index.

No unprocessed correction, no unresolved Comment, and no changed source means clean.

## Output

Report `Processed / Skipped / Updated / Errors / Notes`, including Comments, items,
triage, and partial sources.

## Commit

Never set `user.name` or `user.email` with `git config` or `git config --global`.
All automation commits use one-shot `git -c` identity only.

After correction commits, commit any remaining source reconciliation separately:

```bash
git add -- <touched Work State, comment, and integration files>
git diff --cached --quiet && exit 0
git -c user.name=work-registry-bot -c user.email=bot@work-registry.local \
  commit -m "wr-review: <summary>" -m "Automation: work-registry-bot"
```
