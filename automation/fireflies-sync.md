---
name: fireflies-sync
description: Import allowlisted Fireflies transcripts.
schedule: "30 8 * * *"
timezone: "<team timezone>"
---

# Task: Fireflies Sync

Read `AGENTS.md`, preflight with `git status --short`, then read
`integrations/fireflies.md` once. This task resolves transcript URLs already in the
allowlist; it does not discover meetings, change Fireflies, or write Work State.

## Steps

1. Find inbox URLs without a `SRC-FIREFLIES-*` entry. No URLs means stop clean.
2. Extract each transcript ID and fetch the full transcript, not only its summary.
3. Use the surrounding project mapping. Route unclear or cross-project meetings to
   `general/fireflies/` and report the choice.
4. Write `YYYY-MM-DD-<title>-<transcript-id>.md` with source URL, date, duration,
   participants, and the verbatim speaker transcript.
5. Replace the inbox URL with a stable source entry and destination. Set
   `Last source sync` only after the file is complete; set
   `Last reviewed revision: not reviewed`.

Reuse an existing file for the same transcript ID. A failed or partial fetch stays in
the inbox and gets no sync anchor.

Required connector operation: `fireflies_get_transcript` or its configured equivalent.

## Output

Report `Processed / Skipped / Updated / Errors / Notes`.

## Commit

Never set `user.name` or `user.email` with `git config` or `git config --global`.
Use only the one-shot `git -c` identity below.

Stage only new transcripts and edited lines in `integrations/fireflies.md`:

```bash
git add -- <touched transcript files> integrations/fireflies.md
git diff --cached --quiet && exit 0
git -c user.name=work-registry-bot -c user.email=bot@work-registry.local \
  commit -m "fireflies-sync: <summary>" -m "Automation: work-registry-bot"
```
