---
name: crm-dashboard
description: >
  This skill should be used when the user asks to "show my pipeline", "open my
  dashboard", "how's my pipeline looking", "what's my forecast", "pipeline
  overview", "show me my deals", "where do I stand this quarter", or whenever
  another CRM skill needs to save deal data. It renders the pipeline dashboard
  artifact, which is both the visual view and the persistent store for deal
  values, close dates, and notes.
metadata:
  version: "1.1.0"
---

# Pipeline Dashboard

This skill does two jobs at once: it draws the dashboard, and **it is how deal
data persists.** Any skill that changes a value, close date, or note finishes by
calling this one. Skipping it means the change is lost.

Read `${CLAUDE_PLUGIN_ROOT}/shared/data-model.md` and
`${CLAUDE_PLUGIN_ROOT}/shared/pipeline-model.md` first.

## Step 1 — Fresh data

If Gmail hasn't been read this session, run `crm-sync` first. A dashboard built
on stale labels produces confident wrong decisions.

## Step 2 — Assemble

Combine the Gmail-derived deals with the store (see the merge order in
`data-model.md`) into this shape. Everything under `store` is persisted;
everything else is recomputed each render.

```js
{
  generated: "2026-08-17T08:00:00-05:00",
  owner: "Jordan Ellis",
  currency: "USD",
  quota: 250000,            // null if not configured
  quotaPeriod: "Quarter",
  stages: [                 // exactly these five, in order, always present
    {name:"Contacted",   count:9, value:74000},
    {name:"Engaged",     count:6, value:98000},
    {name:"Pitched",     count:5, value:142000},
    {name:"Negotiating", count:3, value:96000},
    {name:"Verbal",      count:2, value:58000}
  ],
  prospects:  {count:11, value:40000},
  nurture:    {count:7,  value:85000},
  closedWon:  {count:4,  value:112000},
  closedLost: {count:6,  value:0},
  deals: [{
    brand:"Nike", contact:"Dana Reyes", stage:"Negotiating",
    value:45000, weighted:29250, daysSilent:6,
    health:"Cold",             // exact string from the health table
    priority:9,                // value × stage probability × (daysSilent ÷ SLA)
    why:"Silent 6 days after you countered on deliverables.",
    nextAction:"Send revised scope", nextActionDate:"Aug 17",
    threadLink:"https://mail.google.com/mail/u/0/#all/THREADID",
    estimated:false            // true when value came from defaultDealValue
  }],
  store: { /* the full store object from data-model.md */ }
}
```

Rules:

- `stages` always contains all five funnel stages, even at zero. `Prospect`,
  `Nurture`, and closed deals are summarized separately — the funnel starts at
  first contact, because an uncontacted prospect isn't pipeline.
- `deals` is open deals only.
- `health` must match the health table exactly.
- `why` is required for anything not `Healthy` and must contain a specific
  number. "Needs follow-up" is not acceptable; "19 days since the deck landed,
  no reply after pricing" is.
- `estimated: true` on any deal falling back to `defaultDealValue`, so the
  dashboard can mark the forecast as partly guessed.

## Step 3 — Render and persist

Read `${CLAUDE_PLUGIN_ROOT}/assets/dashboard-template.html`. Make exactly two
substitutions:

1. Replace the contents of the `<script id="crm-store">` tag with the `store`
   object.
2. Replace the default object after `const DATA = /*__CRM_DATA__*/ ` up to its
   matching `};` with the assembled object above.

Change nothing else. The palette, type scale, and dark mode are validated — do
not restyle, add libraries, or swap colors.

Then:

1. Write the file.
2. `SendUserFile` to get a `file_uuid`.
3. `create_artifact` with id `partnerships-pipeline` (first time) or
   `update_artifact` with that same id (every time after). **Never create a
   second artifact** — a duplicate splits the store and the CRM starts
   disagreeing with itself. Call `list_artifacts` first if unsure.

If the desktop app is unavailable, `SendUserFile` alone is the fallback. Say
plainly that the data wasn't saved this run and what will happen next time —
this is the one failure worth surfacing, because it's the one that loses work.

## Step 4 — Read it back to her

The dashboard shows the numbers; the response supplies the judgment. Three or
four sentences, no headers:

> $468k open, $109k weighted — 1.9× coverage against the $250k quarter target,
> under the 3× you'd want with six weeks left. The real problem is
> concentration: Nike and Celsius are $76k of the $96k in Negotiating and both
> have gone quiet. Alo Yoga has been waiting two days on a rate card. I'd answer
> Alo today and put a phased option in front of Lululemon before that $18k rolls
> over.

Say the uncomfortable thing. Name the concentration risk, the stalled deal
nobody wants to reclassify, and the honest read on whether the number gets made.
A dashboard that only reports totals is a scoreboard.

If more than a third of open deals are `estimated`, say the forecast is soft and
offer to fill in the real values.

## How she edits data

The artifact can't write back to itself, so all edits happen by telling Claude:
*"Nike is actually $60k"*, *"add a note on Gymshark"*, *"close date on Celsius
is Oct 1"*. Those go through `crm-log`, which finishes by calling this skill.
Mention this once during setup, then never again.

## Variations

- **"How's this quarter looking"** — lead with coverage ratio and slip risk.
- **"What did I close"** — set the period to the requested range; lead with won
  deals, average deal size, and win rate by source.
- **"Show me just the cold ones"** — filter `deals` before rendering, keep the
  full funnel for context.
