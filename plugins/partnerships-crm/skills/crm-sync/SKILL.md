---
name: crm-sync
description: >
  This skill should be used when the user asks to "sync my CRM", "update my
  pipeline", "check my inbox for deals", "refresh the tracker", "did anything
  move", "catch up my deals", or when any other partnerships CRM skill needs
  fresh data before running. It reads recent Gmail activity, updates deal stages
  via Gmail labels, detects new inbound opportunities, and merges the result
  into the pipeline dashboard.
metadata:
  version: "1.1.0"
---

# CRM Sync

Reconcile Gmail against the stored deal data. Every other skill assumes sync ran
recently.

Read `${CLAUDE_PLUGIN_ROOT}/shared/data-model.md` and
`${CLAUDE_PLUGIN_ROOT}/shared/pipeline-model.md` first. Follow the merge order
in `data-model.md` exactly — Gmail wins on stage and dates, the store wins on
value, close date, and notes.

## Step 1 — Load both sides

**Gmail** — resolve `CRM/` label names to IDs with `list_labels`, then pull
every thread carrying a `CRM/` label, plus new candidates (Step 3).

**Store** — stage the `partnerships-pipeline` artifact with `device_stage_files`
and parse the `crm-store` script tag.

If the artifact can't be read, follow the recovery ladder in `data-model.md`:
last digest email → labels → ask. Never stop the sync over a missing store.

## Step 2 — Update labeled deals

For each thread with a `CRM/` label, read the newest messages and derive:

- Last contact date and direction
- Touch count since the stage label was applied
- Whether the newest message is inbound and unanswered

Then evaluate a stage change using the signal table in `pipeline-model.md`. Read
what they actually wrote — never infer from a subject line.

**On a stage change:** remove the old stage label, apply the new one, and note
the change for the report. Moving to `Closed Won`/`Closed Lost` needs a
`closedReason`; moving to `Nurture` needs a `revisitDate` — infer both from the
thread, and ask only if genuinely ambiguous.

**Apply `CRM/Needs Reply`** when the newest message is inbound and unanswered.
Remove it as soon as she replies.

**Ignore out-of-office auto-replies completely** — no date update, no touch
count, no stage change, no `Needs Reply`. Set the next action to their stated
return date instead.

**Add a note** only when the thread contains something genuinely new: a budget
figure, a timeline, a new stakeholder, a stated objection. Append; never rewrite.
Routine scheduling chatter is not a note.

## Step 3 — Detect new deals

Search recent mail (`syncLookbackDays`, default 30) using the same query shapes
as the setup backfill, excluding `excludedDomains`, no-reply senders,
newsletters, and promotions.

For anything unlabeled that looks like a real partnership conversation, propose
it: brand, contact, inferred stage, and a one-line summary.

**New deals need approval before being labeled.** Existing-deal updates do not —
those are mechanical. A misclassified thread pollutes the pipeline and the
forecast, so the confirmation is worth the friction.

When running unattended from the digest, auto-approve but mark each new entry's
first note with `AUTO:` so it's easy to audit.

## Step 4 — Write back

Merge into the store and update the artifact via `crm-dashboard`. If the desktop
app is unreachable, keep the label changes (those already landed) and say the
dashboard will catch up next run.

## Step 5 — Report

Lead with what changed. Keep it scannable:

```
Synced 34 deals · 12 threads with activity

Moved forward
  Nike → Negotiating (they asked about Q4 pricing)
  Gymshark → Verbal (confirmed, contract pending)

Needs your reply today
  Alo Yoga — Sarah asked for the rate card 2 days ago
  Celsius — Mark countered on deliverables yesterday

New deals found
  Vuori — inbound, asked about collab opportunities

Went cold
  Lululemon — 19 days silent at Pitched, $18k
```

If nothing changed, say so in one line. Never pad an empty sync.

## Guardrails

- Never send an email from this skill.
- Never remove a deal from the store — deals close, they don't disappear.
- Never move a deal backwards without noting why.
- Never touch labels outside the `CRM/` prefix.
- When a stage is genuinely ambiguous, leave it alone and flag it in the report
  rather than guessing.
- If a label write fails, record the intent in `manualStage` and retry next
  sync. Never drop it silently.
