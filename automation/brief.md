---
name: daily-brief
description: Write a short, traceable view of today's current Work State.
schedule: "30 11 * * *"
timezone: "<team timezone>"
---

# Task: Daily Brief

Write `briefs/YYYY-MM-DD-daily-brief.md` after work-state review. Follow `AGENTS.md`
and preflight with `git status --short`. This task reports Work State; it never creates
or repairs it.

## Inputs

Read in this order:

1. `integrations/*.md` — build the strict source allowlist and note stale or
   unreviewed anchors. This validates citations; it is not planning state.
2. `work-states/index.md` — select live `open`, `active`, and `blocked` items and read
   manual triage. Closed or superseded items are read only when changed since the
   previous brief.
3. Selected `WS-*.md` files — these are the source of current status, priority,
   blockers, tasks, evidence, and next-update needs.
4. Every Comment file containing a `CMT-*` cited by a selected item. Also read the
   current month, plus the previous month during days 1–7, for unresolved corrections.
5. The previous brief — compare presentation only; never reuse it as evidence.

Open a registered source file only to verify an existing `S#` citation. Do not derive
new Work State from it.

Use dates written in Work State, its Changelog, and Comments. File modification time
is not evidence. Use the previous brief date as the change cutoff.

## Hard exclusions

- Anything not registered in `integrations/`.
- Bare URLs, placeholders, and entries without both a stable `SRC-*` ID and local
  destination.
- Exploratory notes, scratch files, generated summaries, extracts, PR or deployment
  notes, and application code.
- Briefs, the Work State index, and other Work State items as evidence for a factual
  claim; follow their `S#` or `CMT-*` lineage.
- Unreviewed source content as current state. Report its missing coverage under
  manual triage.
- Guesses based on an `active` status. “Worked today” requires dated evidence in the
  item or a Comment.

## Build the brief

Use only facts already present in selected Work State. Write these sections:

1. **Active** — work with evidence of activity today. This answers what is moving now.
2. **Priority order** — live items in explicit recorded order; otherwise sort by
   `P1`, `P2`, `P3`, then `WS-*` ID. Give one cited reason from the item. This answers
   what comes next without inventing a ranking.
3. **Blocked / waiting on** — blocker, owner or dependency, and age when known. This
   makes stalled work actionable.
4. **Recently changed** — status, priority, scope, or evidence changed since the prior
   brief. This prevents readers from rereading unchanged work.
5. **Follow-ups** — only items whose `Next Update Needed` requires a message, meeting,
   document, or decision.
6. **Needs manual triage** — ambiguous, weak, stale, partial, or unreviewed coverage.

For every section:

- Include only material entries; otherwise write `None found from allowed sources`.
- Name the `WS-*` item and relevant `T###`, `S#`, and `CMT-*` references.
- State uncertainty directly. Never infer a date, owner, status, or priority.
- Use short sentences and plain words. Remove background that does not change an
  action or decision.

Keep the brief to one page. If live work does not fit, summarize counts and aging; do
not silently omit items. Regenerate only today's file and never change older briefs.

## Commit

Never set `user.name` or `user.email` with `git config` or `git config --global`.
Use only the one-shot `git -c` identity below.

Stage only today's brief. A clean run makes no commit:

```bash
git add -- briefs/YYYY-MM-DD-daily-brief.md
git diff --cached --quiet && exit 0
git -c user.name=work-registry-bot -c user.email=bot@work-registry.local \
  commit -m "brief: <summary>" -m "Automation: work-registry-bot"
```
