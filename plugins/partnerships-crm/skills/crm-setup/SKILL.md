---
name: crm-setup
description: >
  This skill should be used the first time someone uses the partnerships CRM, or
  when they say "set up my CRM", "set up the partnerships tracker", "get my deal
  tracker going", "start my pipeline", "install the CRM", "reset my CRM", or ask
  where their pipeline lives. It creates the Gmail label set, backfills existing
  deals from the inbox, builds the pipeline dashboard, and schedules the daily
  digest.
metadata:
  version: "1.1.0"
---

# CRM Setup

Run once. Idempotent — safe to re-run; detect what already exists and only
create what's missing.

Read `${CLAUDE_PLUGIN_ROOT}/shared/data-model.md` and
`${CLAUDE_PLUGIN_ROOT}/shared/pipeline-model.md` before starting.

## Step 0 — Check what already exists

Call `list_labels` and look for the `CRM/` tree. Call `list_artifacts` and look
for `partnerships-pipeline`.

- **Both present**: this is a re-run. Report what exists and offer to repair
  rather than rebuild. Never wipe an existing store.
- **Neither present**: fresh setup, continue.

Confirm Gmail is reachable before doing anything else. If it isn't, stop and say
so — every other step depends on it.

## Step 1 — Collect settings

One round of AskUserQuestion, four questions max:

1. **Name and company** for email signatures — offer her Gmail account name as
   the default.
2. **Typical deal size** — ranges (`Under $2k`, `$2k–10k`, `$10k–50k`, `$50k+`).
   The midpoint becomes `defaultDealValue`.
3. **Target for the period** — amount, and month or quarter. Skippable; it only
   enables the coverage ratio.
4. **Digest time** — default 8:00 AM local, weekdays.

Also ask (or infer from her email domain) which domains are **internal** so
colleagues never get logged as deals. This one matters — without it the backfill
fills up with coworkers.

If she skips anything, take the default and state what was assumed.

## Step 2 — Create the Gmail labels

Create the `CRM/` tree from `data-model.md`, with the listed `colorPreset`
values and `autoCreateParentLabels: true`. Skip any that already exist. Never
touch a label outside the `CRM/` prefix.

## Step 3 — Backfill from the inbox

This is what makes setup worth doing — do not skip it.

Search the last 90 days across sent and received. Run several narrow queries
rather than one broad one:

- `newer_than:90d (partnership OR sponsorship OR collab OR collaboration)`
- `newer_than:90d ("media kit" OR "rate card" OR "one pager" OR proposal)`
- `newer_than:90d ("work together" OR activation OR "brand deal" OR sponsorship)`
- `newer_than:90d in:sent -in:draft` filtered to external domains with a reply

Exclude: internal domains, `no-reply`/`noreply` senders, newsletters,
`category:promotions`, `category:social`, and calendar notifications.

For each surviving thread, read enough of it to infer `brand`, contact name and
email, and **stage** using the signal table in `pipeline-model.md`.

**Present everything as one table and get approval before writing.** Say plainly
that stage inference from old email is a guess worth skimming. Let her strike
rows, correct stages, and add deal values inline. Then:

- Apply the stage label to each approved thread.
- Build the store entry for each, keyed by thread ID.

Cap the review table at 40 rows. If more qualify, take the 40 most recent and
say how many were held back.

## Step 4 — Build the dashboard

Run the `crm-dashboard` skill. It creates the `partnerships-pipeline` artifact
and writes the store into it — that call is what makes the CRM persistent, so it
is not optional.

If the desktop app isn't connected, say so plainly: the labels and follow-ups
work now, and the dashboard will appear the first time she runs this from the
desktop app.

## Step 5 — Schedule the daily digest

Use `create_trigger` — never local cron, which dies with the session.

- **Cron**: her digest time on her digest days, converted from local time to UTC.
- **Name**: `Partnerships CRM — Daily Digest`
- **Prompt** — must be complete and standalone, since each firing starts fresh:

  > Run the partnerships CRM daily digest using the crm-digest skill from the
  > partnerships-crm plugin. Deal stages live in Gmail `CRM/` labels; deal
  > values and notes live in the `partnerships-pipeline` dashboard artifact.
  > Sync Gmail first, then send the digest email. Create follow-ups as Gmail
  > drafts only — never send an email to a contact.

Confirm the schedule in plain language ("every weekday at 8am").

## Step 6 — Closing summary

Report in plain language:

- How many deals were found and labeled, broken down by stage
- That the pipeline is now visible in Gmail under `CRM/` on any device
- That the dashboard is in her sidebar
- The digest schedule, in words
- Four things worth remembering: *"sync my CRM"*, *"show my pipeline"*, *"who do
  I need to follow up with"*, *"add [brand] at $X"*

State clearly, once: **this system drafts emails but never sends them to
contacts.** The only email it sends is the digest, to her.

Then tell her the single highest-value next step: **add deal values to the
top 10 deals**, because the forecast and the priority ranking are both worthless
without them. Offer to walk the list right now.
