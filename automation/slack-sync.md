---
name: slack-sync
description: Synchronize allowlisted Slack threads and discover linked files.
schedule: "0 8 * * *"
timezone: "<team timezone>"
---

# Task: Slack Sync

Read `AGENTS.md`, preflight with `git status --short`, then read
`integrations/slack.md` once. Scan only listed channels. New direct-message discovery
requires an explicit human request; existing registered DM threads may refresh. This
task never writes Work State.

## Messages

1. Build channel-ID → project and channel-ID → `last_checked` maps.
2. Read messages newer than each anchor in bounded parallel batches. Page until the
   boundary is complete.
3. Collect threads from new messages and saved thread files. Refresh saved threads
   checked in the last 14 days; on Monday refresh all saved threads. Deduplicate by
   `(channel_id, thread_ts)` and fetch replies newer than each thread's `Last checked`.
4. Match saved files by `Thread TS`. Create unmatched threads under
   `<project>/slack/` or `general/slack/`; append only new replies to matched files.
   Store channel, thread ID, last checked time, participants, and verbatim messages.
5. Register every new thread with a stable `SRC-SLACK-*` ID. Record the newest complete
   timestamp, but do not advance the channel anchor until file discovery finishes.

## File and URL discovery

1. On every run, inspect new messages and fetched replies for Google Drive URLs and
   Slack file IDs. Also run channel-scoped document, spreadsheet, and PDF searches
   from the channel's old `last_checked` boundary. Never search the whole workspace.
2. When search results omit attachment metadata, read the containing thread. Collect
   file ID, URL, title, type, size, channel, message timestamp, and project.
3. Deduplicate Google file IDs against `integrations/google-docs.md` and Slack file IDs
   against `integrations/slack.md`.
4. When Google Docs sync is configured, add unseen Google URLs to
   `integrations/google-docs.md` as bare allowlist entries; the following Google task
   resolves and downloads them the same day. Otherwise report the URLs without
   registering them.
5. Save unseen Slack uploads of at most 10 MB under `<project>/slack/` and add full
   `SRC-SLACK-FILE-*` entries. Larger files use the configured large-file store;
   without one, report and do not register them.
6. Unclear project routes to `general/<source>/` and is reported. Keep the Slack
   message and its linked file as separate registered origins.
7. Advance each thread and channel anchor to its newest fully processed timestamp only
   after messages, replies, URLs, and files all succeed. Never move an anchor backward.

No new messages or files means stop clean. Partial pagination keeps old anchors.
Secrets or personal data are not saved; report them out of band.

Required connector operations: channel history, thread reads, channel-scoped search,
and file reads; common names are `slack_read_channel`, `slack_read_thread`,
`slack_search_public_and_private`, and `slack_read_file`.

## Output

Report `Processed / Skipped / Updated / Errors / Notes`.

## Commit

Never set `user.name` or `user.email` with `git config` or `git config --global`.
Use only the one-shot `git -c` identity below.

Stage only touched Slack files and integration lines (add `integrations/google-docs.md`
only when this task changed it):

```bash
git add -- <touched Slack files> integrations/slack.md
git diff --cached --quiet && exit 0
git -c user.name=work-registry-bot -c user.email=bot@work-registry.local \
  commit -m "slack-sync: <summary>" -m "Automation: work-registry-bot"
```
