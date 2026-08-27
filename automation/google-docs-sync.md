---
name: google-docs-sync
description: Synchronize changed allowlisted Google Docs, Sheets, and PDFs.
schedule: "0 9 * * *"
timezone: "<team timezone>"
---

# Task: Google Docs Sync

Read `AGENTS.md`, preflight with `git status --short`, then read
`integrations/google-docs.md` once. This task writes only registered source files and
sync anchors. It never writes Work State or changes Google Drive.

## Steps

1. Parse every registered entry and bare Google Drive URL. Keep file ID, project,
   destination, `Remote modified`, and `Last source sync` in memory.
2. Fetch metadata in batches of at most 25. A registered file changed when its remote
   `modifiedTime` is newer than `Remote modified`; missing anchors force one fetch.
3. Download changed files only:
   - Doc or PDF → read content and write `.md`.
   - Sheet → export `text/csv` and write `.csv`.
   - Unsupported type → skip and report an error.
4. Use the registered destination unchanged. If a connector saves a large result to a
   temporary file, move that result to the destination; do not fetch it again.
5. For each bare URL, fetch metadata, assign
   `SRC-GDOC-<PROJECT>-<YYYY-MM-DD>-<TITLE>`, route to `<project>/google-docs/` or
   `general/google-docs/`, write the file, and replace the URL with a full entry.
   Unclear project routes to `general/` and is reported.
6. After each successful file, set `Last source sync` to current UTC and
   `Remote modified` to the returned time. Never change `Last reviewed revision`.
   After every metadata check succeeds, update `Last checked`.

Registered destinations are canonical. A failed download keeps old anchors. No newer
files and no bare URLs means no source write; an advanced `Last checked` is still a
change.

Required connector operations: `get_file_metadata`, `read_file_content`, and
`download_file_content` (or equivalent configured names).

## Output

Report `Processed / Skipped / Updated / Errors / Notes`.

## Commit

Never set `user.name` or `user.email` with `git config` or `git config --global`.
Use only the one-shot `git -c` identity below.

Stage only downloaded files and edited lines in `integrations/google-docs.md`:

```bash
git add -- <touched source files> integrations/google-docs.md
git diff --cached --quiet && exit 0
git -c user.name=work-registry-bot -c user.email=bot@work-registry.local \
  commit -m "gdocs-sync: <summary>" -m "Automation: work-registry-bot"
```
