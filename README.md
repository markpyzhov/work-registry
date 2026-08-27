# Work Registry

Work Registry is an operating discipline for teams that run AI agents across chats,
documents, meetings, and task systems. A scheduled runtime agent synchronizes that
evidence into plain Markdown held in Git, distills it into status-bearing Work State,
and generates a short Daily Brief. People correct what is wrong in flow, and each
correction becomes durable evidence that survives regeneration.

This repository is the reference package: the structure, [templates](templates/), an
end-to-end [example](example/), and [automation contracts](automation/).

## Why

Source systems — chats, documents, meeting transcripts, and task trackers — mostly
record events, not current state. Task trackers do carry per-ticket status, but it can
lag what was agreed elsewhere, and the context that explains or contradicts it lives in
the other systems. An agent retrieving over raw sources rebuilds the same context every
session and can report stale status with confidence.

The Registry reconciles evidence once, off the query path, so people and agents read one
small, current, source-traceable picture. It augments task systems, document stores, and
agent memory.

## Design principles

- **Local** — people and agents read the package directly as files.
- **Portable and shared** — plain Markdown in Git; no runtime owns the team's record.
- **Source-traceable** — every important claim links to a stable source ID.
- **Current by correction** — people correct generated state in place, and corrections
  remain traceable.
- **Action-oriented** — people receive the actionable items that matter most.

## Structure of a registry

```text
AGENTS.md                         the agent's operating contract
CLAUDE.md                         compatibility pointer to AGENTS.md
integrations/<source>.md          Source Registry: allowlist of tracked sources,
                                  with separate sync and review anchors
<project>/<source>/               synchronized evidence for one project
general/<source>/                 evidence spanning projects
work-states/WS-*.md               Work State items: status, tasks, evidence
work-states/index.md              one row per item, plus manual triage
work-states/comments/YYYY-MM.md   recorded corrections (CMT-*)
briefs/YYYY-MM-DD-daily-brief.md  dated generated briefs
```

Identifiers are stable and append-only: `SRC-*` sources, `WS-*` items, `T###` task
codes and `S#` source refs local to an item, `CMT-*` comments.

## Pipeline

The runtime agent — the persistent agent hosted in the team's agent runtime or
scheduler — runs three scheduled tasks in order:

1. **Source sync** — one task per integrated system (Slack, Google Docs, etc.) captures
   changed evidence and advances only its sync anchors.
2. **Work-state review** — reconciles changed registered Sources and `CMT-*` Comments
   into current, source-traceable `WS-*` items. Ambiguity goes to manual triage.
3. **Brief generation** — renders those items into a short, dated, source-traceable
   view.

Together, these tasks implement the article's Ingest, Distill, and Plan stages. People
and project agents complete the pipeline by using the package locally and correcting it
in place.

## Correction design contract

1. A person fixes a `WS-*` item or brief in place and pushes the commit on any branch
   (`cmt-*` by convention). No pull request, form, or ticket is required.
2. On the next work-state review, the agent processes unprocessed branches one at a
   time on top of current `main`, each on a temporary integration branch. For each
   branch it:
   1. records a `CMT-*` Comment (branch, commit SHA, author, time, diff, correction
      text) in `work-states/comments/YYYY-MM.md`;
   2. cherry-picks the commit, preserving the human author;
   3. restores the generated files and regenerates them from registered evidence plus
      all Comments; the latest applicable Comment decides status, earlier ones stay
      visible;
   4. advances `main` only after validation, so readers never see a half-applied edit.
      On conflict or unclear precedence it stops and leaves the item to manual triage.

## Example

[`example/`](example/) is a minimal registry for a fictional team and its project
`alpha`. It shows one run of the pipeline above, then one pass of the correction
design contract.

The story: customer onboarding is blocked on a data-access approval; the approval
is granted, and one human correction fixes the stale state.

1. **Source sync** registers four sources in
   [`integrations/`](example/integrations/) and captures their evidence: the
   [requirements draft](example/alpha/google-docs/2026-01-14-onboarding-requirements.md),
   [approval thread](example/alpha/slack/2026-01-15-access-approval-status.md), and
   [task](example/alpha/clickup/ONB-101.md) under the project; the
   [weekly sync](example/general/fireflies/2026-01-13-weekly-sync.md) under
   `general/` because it spans projects.
2. **Work-state review** reconciles them into
   [WS-001 Customer onboarding](example/work-states/WS-001-customer-onboarding.md) —
   tasks `T001–T004`, source refs `S1–S4`, status **blocked** on the approval — and
   lists it in the [index](example/work-states/index.md).
3. **Brief generation** renders the [2026-01-15 brief](example/briefs/2026-01-15-daily-brief.md),
   listing WS-001 under *Blocked / waiting on*.

Then the correction: security grants the approval, and a person fixes WS-001 on
branch `cmt-access-approved`. The next work-state review records it as
[`CMT-2026-01-001`](example/work-states/comments/2026-01.md) and regenerates:
WS-001 becomes **active**, `T001` closes, the changelog cites the Comment, and the
[2026-01-16 brief](example/briefs/2026-01-16-daily-brief.md) shows the item under
*Active*.

## Set up a registry

The pipeline is a Markdown task chain, configured in the team's runtime or scheduler.

The environment must provide a runtime agent that:

- can run the tasks on schedule (for example, scheduled tasks in Hermes, Claude Code,
  Codex, or a CI runner invoking an agent CLI);
- can integrate with the MCP services of the source systems;
- has Git access to commit and push the registry repository and read correction
  branches.

### 1. Create the repository

1. Create an empty Git repository.
2. Copy the contents of [`templates/`](templates/) into the new registry root.
3. Copy this repository's [`automation/`](automation/) directory beside them.
4. Set the team timezone and the bot name and email used by one-shot commit commands
   in the files.

### 2. Configure projects and sources

1. Choose stable project slugs such as `alpha`; reserve `general/` for material that
   spans projects.
2. Keep only the integration files and sync tasks the team uses.
3. In each `integrations/<source>.md`, list the URLs of the resources to include.
   Source sync then adds the rest of the metadata: stable `SRC-*` IDs, destinations,
   project mappings, and separate sync and review anchors.

### 3. Configure MCPs or connectors

Configure source access in the agent runtime or scheduler that executes the tasks.

1. In the runtime's MCP or connector settings, enable and authenticate each source
   system used by the Registry.
2. Grant the runner read access only to locations listed in the matching integration
   file. Sync tasks are read-only upstream unless a task explicitly says otherwise.
3. Map the operation names in each task to the runtime's actual tools:
   - [Slack](automation/slack-sync.md) — channel history, thread reads, scoped
     search, file reads;
   - [Fireflies](automation/fireflies-sync.md) — full transcript reads;
   - [Google Docs](automation/google-docs-sync.md) — metadata, content reads, and
     Sheet export;
   - [ClickUp](automation/clickup-sync.md) — task summaries, details, and subtasks.
4. Some tasks discover new documents in another system and register them for that
   system's task (for example, Slack sync registers linked Google Docs). If the team
   does not use that system, remove that part of the task or replace the service with
   one the team uses.

For another system, configure its MCP first, then ask an interactive agent to
follow the [guide for adding a sync task](integration-sync-guide.md): the agent
probes the connector, drafts the task, and asks for the policy decisions it cannot
discover.

### 4. Test manually

Run each configured sync task from the registry root. Confirm that it:

- reads only allowed locations;
- writes evidence under `<project>/<source>/` or `general/<source>/`;
- records provenance and advances sync anchors only after full success;
- commits only its own files;
- adds metadata (IDs, destinations, and so on) to the index.

Then run [work-state review](automation/work-state-review.md) and
[brief generation](automation/brief.md).

Then, validate the result:

1. The index files should contain current metadata and links to the downloaded
   resources.
2. The Work State files should reflect the current status of the work, with every
   important claim traced to its sources.
3. The Brief should surface only live work on one page, with each entry linked to its
   Work State item.
4. If you see an error in a Work State, update it inline.

For an initial source set of more than 20 documents, consider processing the Work State
in calendar-based chunks. This reduces the risk of losing information when the source
set is large.

### 5. Schedule automation

Create scheduled agent tasks from the Markdown files in `automation/`. The runner must:

- start in the registry root on current `main`;
- provide the configured connector tools and expose correction branch refs;
- run sync tasks in a fixed order, then review, then brief;
- publish `main` only after a task's commit and checks succeed.

Use the example cron times or replace them. Do not schedule unused integrations.

### 6. Attach it to a project

Everything in this section is a recommendation, not a requirement.

Make the registry visible inside each application repository using a symlink, Git
submodule, mount, or workspace link. A sibling-repository symlink can be:

```bash
ln -s ../work-registry work-registry
```

Add a short bootstrap section to the application's `AGENTS.md`:

```markdown
## Work Registry

When a request lacks context, read `work-registry/work-states/index.md`, then the
relevant `WS-*` items for this project. Open registered sources only to verify cited
evidence.
```

If the runtime uses the pointer convention, point the application's `CLAUDE.md` to its
`AGENTS.md`. This gives every project agent the same local Work State without copying
it into application code. When a project has a primary `WS-*` item, list that item and
its `<project>/` source folder as direct entry points in the project rules.

## Release and citation

Version `1.0.0` is identified by the annotated Git tag `v1.0.0`. Cite this package
using [`CITATION.cff`](CITATION.cff). It is released under the [MIT License](LICENSE).
