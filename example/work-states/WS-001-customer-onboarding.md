# WS-001 Customer onboarding

Status: active
Priority: P1
Owner: Onboarding team
Projects: alpha

## Sources

- S1 — `SRC-GDOC-ALPHA-2026-01-14-ONBOARDING-REQUIREMENTS` — [requirements draft](../alpha/google-docs/2026-01-14-onboarding-requirements.md)
- S2 — `SRC-FIREFLIES-GENERAL-2026-01-13-WEEKLY-SYNC` — [weekly sync](../general/fireflies/2026-01-13-weekly-sync.md)
- S3 — `SRC-SLACK-ALPHA-2026-01-15-ACCESS-APPROVAL-STATUS` — [approval thread](../alpha/slack/2026-01-15-access-approval-status.md)
- S4 — `SRC-CU-ALPHA-ONB-101` — [onboarding task](../alpha/clickup/ONB-101.md)

## Request / Context

Enterprise customer signed 2026-01-10; onboarding must finish before go-live (S2).
Everything after the data-access approval is ready (S1). The approval was granted on
2026-01-16 (`CMT-2026-01-001`), so the remaining steps are unblocked.

## Task List

- [ ] `T002` — Environment setup (S1, S4)
- [ ] `T003` — Sample import (S1)
- [ ] `T004` — Validation run (S1)

## Done / Evidence

- `T001` — Data-access request submitted 2026-01-12 (S1); approved 2026-01-16
  (`CMT-2026-01-001`)

## Next Update Needed

status_call

## Changelog

- 2026-01-16 — `CMT-2026-01-001`: status blocked → active; approval granted, `T001`
  closed, dropped the stale "may slip to next quarter" assumption.
- 2026-01-15 — created from S1–S4 as **blocked** on the data-access approval (S1, S3).
