# Guide: Add a Sync Task for a New System

This guide is written for an AI agent. To install a new integration, enable the
system's MCP or connector in the agent runtime, start an interactive agent session
in the registry root, and say:

> Follow `integration-sync-guide.md` to add `<source>`.

The agent does the work below and asks you only the questions it cannot answer
itself. This guide is run once per new system, interactively; it is not a
scheduled task and must not be placed in `automation/`.

## Agent instructions

You are adding `automation/<source>-sync.md`, a runnable scheduled task, and
`integrations/<source>.md`, its allowlist. Work in order. When a step says to ask
the user, ask and wait — never guess a policy answer.

### 1. Preflight

Read `AGENTS.md`. Run `git status --short`; report anything unexpected. List the
`<source>` connector's tools. If the connector is missing, stop and ask the user
to configure it in the runtime first.

### 2. Copy the closest existing task

Copy the task in `automation/` closest to the new system to
`automation/<source>-sync.md`:

- [`slack-sync.md`](automation/slack-sync.md) for chat;
- [`fireflies-sync.md`](automation/fireflies-sync.md) for meetings;
- [`google-docs-sync.md`](automation/google-docs-sync.md) for documents;
- [`clickup-sync.md`](automation/clickup-sync.md) for task trackers.

Keep its structure, retry defaults, and the invariants listed at the end of this
guide.

### 3. Discover the connector contract

Probe the connector and write the findings into the task text:

- The operations that read metadata, list changes, and read content — from the
  tool list and schemas, each confirmed with one test call.
- The durable native ID used for deduplication — from the ID fields in
  responses, confirmed stable by fetching the same item twice.
- The change marker: modified time, event cursor, or revision — from the same
  list and metadata responses.
- Pagination, batch size, and rate limits — from schema parameters and probing.
- A proposed list of supported content types with a local file format for each —
  from what the API returns.

### 4. Ask the user for the policy decisions

Ask these one by one and write the answers into the task text. Do not infer them
from the connector:

- **Project mapping.** List the native locations you found (channels, folders,
  spaces) and ask which project each belongs to, or `general/`.
- **Discovery scope.** Whether private or direct-message content may be
  captured. The connector serving it is not permission to capture it.
- **Upstream access.** Read-only, or the exact allowed mutations. Seeing
  mutating tools does not mean they are permitted.
- **Sensitive content.** How to handle deleted, retracted, unsupported,
  oversized, secret, or personal content.
- **Confirmation of your proposals.** The supported content types from step 3,
  plus the schedule cron and timezone for the task frontmatter.

Destination conflicts, unclear routing, no changes, partial failure, and clean
runs need no questions: the invariants below already fix their behavior.

### 5. Create the integration file

Create `integrations/<source>.md` following an existing one such as
[`example/integrations/slack.md`](example/integrations/slack.md). Ask the user
for the URLs of the resources to track and list them; the sync task adds `SRC-*`
IDs, destinations, and anchors on its first run.

### 6. Validate and hand over

Re-read `automation/<source>-sync.md`. It must contain no placeholder, open
question, or unverified operation name; a task with any of these is not runnable
and must not be scheduled. Then ask the user to run the new task once manually
and check the results (README, "Test manually") before scheduling it with the
other automation.

## Invariants every sync task keeps

Preserve these when adapting the copied task:

- Start by reading `AGENTS.md`, preflighting with `git status --short`, and
  reading `integrations/<source>.md` once to build the allowlist.
- Process only allowlisted sources; registered destinations stay fixed.
- Deduplicate by the native ID: an existing ID updates its registered
  destination.
- Write provenance with each file: source URL, native ID, captured or event
  time, and participants or author when relevant. Preserve verbatim evidence.
- New sources get a stable `SRC-<SYSTEM>-<PROJECT>-<STABLE-SUFFIX>` ID and a
  full registry entry with `Last reviewed revision: not reviewed`.
- Advance the sync anchor only after every page, file write, and registry edit
  for that source succeeds. Never change the review anchor.
- Unclear project routing goes to `general/<source>/` and is reported.
- Report `Processed / Skipped / Updated / Errors / Notes`, stage only the task's
  own files, and commit with the one-shot `git -c` bot identity used by the
  other tasks. A clean run commits nothing.
