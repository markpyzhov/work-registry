# Agent Instructions — Example Work Registry

This is the canonical instruction file for this registry. `CLAUDE.md` points here.
Task runbooks are under `../automation/`.

## Ownership

- `integrations/` — strict source allowlists, destinations, sync anchors, and review
  anchors.
- `alpha/<source>/` and `general/<source>/` — synchronized evidence.
- `work-states/` and `briefs/` — generated state. Humans correct them on branches.
- `work-states/comments/` — durable `CMT-*` records created by work-state review;
  never edited by hand.

## Core rules

- Read the matching integration file before using or writing source evidence.
- Use registered destinations unchanged. A bare URL must become a full `SRC-*` entry
  before review.
- Never renumber `SRC-*`, `WS-*`, `T###`, `S#`, or `CMT-*` identifiers.
- `Last reviewed revision` belongs only to work-state review. Sync tasks use
  `Last source sync`, `Remote modified`, or `last_checked` and never move them back.
- Instruction order: explicit user request → integration data → this file → task
  details.

## Local routing

- Project: `alpha`.
- Project evidence: `alpha/google-docs/`, `alpha/slack/`, `alpha/fireflies/`, or
  `alpha/clickup/`.
- Cross-project or unclear evidence: `general/<source>/`; report unclear routing.

## Automation

Run [Slack](../automation/slack-sync.md) →
[Fireflies](../automation/fireflies-sync.md) →
[Google Docs](../automation/google-docs-sync.md) →
[ClickUp](../automation/clickup-sync.md) →
[work-state review](../automation/work-state-review.md) →
[brief](../automation/brief.md).

## Git

- The runner starts on current `main`; transport and backup are runner concerns.
- Synced evidence and generated state commit to `main`. Logic changes use review.
- Preflight with `git status --short`. Never absorb an existing change in a touched
  file. Stage only task-owned files; clean runs make no commit.
- Automation uses one-shot identity
  `work-registry-bot <bot@work-registry.local>` and commit body
  `Automation: work-registry-bot`.
- Never set bot `user.name` or `user.email` with `git config` or
  `git config --global`.
