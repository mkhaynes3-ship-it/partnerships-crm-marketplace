---
name: crm-log
description: >
  This skill should be used when the user wants to add or change a single deal by
  hand — "add a deal", "log this brand", "add Nike to my pipeline", "move
  Gymshark to negotiating", "mark Vuori as closed won", "we lost the Alo deal",
  "update the value on", "push this to nurture", "I just got off a call with",
  or when forwarding an email to log. It records one deal, keeps the Gmail stage
  label in sync, and persists the change to the pipeline dashboard.
metadata:
  version: "1.1.0"
---

# Log a Deal

Fast single-deal capture and updates. Optimized for speed — she is usually
between other things when she uses this.

Read `${CLAUDE_PLUGIN_ROOT}/shared/pipeline-model.md` and
`${CLAUDE_PLUGIN_ROOT}/shared/data-model.md` first.

This skill is also how she edits deal data — value, close date, notes — since
the dashboard can't write back to itself. Always finish by calling
`crm-dashboard`, which is what persists the change.

## Adding a deal

Extract everything possible from what she said. Fill gaps from Gmail: search for
the brand or contact, and pull the contact's name, title, and email from the
thread and its signature block.

Required before writing: brand and stage. Everything else is best-effort.

Defaults when unstated:

- **Stage** — `Prospect` if no outreach exists in Gmail; `Contacted` if she has
  sent something; `Engaged` if they've replied.
- **Value** — `defaultDealValue` from config, flagged as an estimate.
- **Close date** — 45 days out from `Pitched` onward; blank before that.
- **Next action** — derive from the stage SLA.

Key the store entry by Gmail thread ID. If no thread exists yet (a pure
prospect), key it `prospect:<slug>` and re-key it to the real thread ID on the
first sync after she emails them.

Apply the Gmail stage label, write the store entry, then call `crm-dashboard`.

Confirm in one line: `Nike · Contacted · $15,000 · next: send Q4 deck by Thu`.

## Updating a deal

Match the deal by brand name, contact, or thread. On an ambiguous match, list
the candidates with their stage and value and ask which — never guess between
two live deals.

**On a stage change:**

- Swap the Gmail stage label — remove the old one, apply the new one.
- Reset the touch count and set a new next action from the new stage's SLA.
- Add a dated note recording the change and the reason.
- If the label write fails, record the intent in `manualStage` and say so.

**Moving to `Closed Won`:** require an amount and a `closedReason`. Ask for the
reason if unstated — won-reasons compound into the best prospecting list she
has. Congratulate briefly and move on.

**Moving to `Closed Lost`:** require a `closedReason`. Push gently for a
specific one — `Price` / `Timing` / `Went with competitor` / `No budget` /
`No response` / `Not a fit`. Offer to set it to `Nurture` with a revisit date
instead when the loss is about timing rather than fit, since timing losses are
the most winnable pipeline she has.

**Moving to `Nurture`:** require a `revisitDate`. Convert vague language to a
real date and confirm it.

## Logging a call or meeting

When she says she spoke to someone, capture what actually matters:

- What they committed to, and by when
- What she committed to, and by when
- Budget or timing signals
- New stakeholders named
- Objections raised

Append a dated note, set the next action from her commitment, and evaluate
whether the conversation justifies a stage change.

If she committed to sending something, set the next action to that date and
offer to draft it now.

## Bulk entry

When she pastes a list of brands or contacts, parse them all, present the parsed
rows as one table for approval, and write the approved set in a single pass.
Never ask a follow-up question per row.

## Guardrails

- Notes are append-only. Never rewrite or delete an existing note.
- Never remove a deal from the store.
- Never send an email from this skill.
- Always finish by calling `crm-dashboard`. A change that isn't persisted is a
  change that didn't happen.
