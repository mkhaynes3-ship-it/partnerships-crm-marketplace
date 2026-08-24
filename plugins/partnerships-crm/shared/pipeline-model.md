# Pipeline Model

The shared sales logic for this CRM. Every skill reads stage definitions, cadence
rules, health scoring, and forecast math from here. Do not invent alternate stage
names or cadences anywhere else in the plugin.

---

## Stages

Eight stages. A deal sits in exactly one. Stage is determined by what the
**counterparty** has done, never by what the seller hopes.

| # | Stage | Enters when | Exits when | Win prob |
|---|-------|-------------|------------|----------|
| 0 | `Prospect` | Brand identified, no outreach sent yet | First outreach email sent | 5% |
| 1 | `Contacted` | Outreach sent, no reply received | They reply (→ Engaged) or cadence exhausts (→ Nurture/Lost) | 10% |
| 2 | `Engaged` | They replied with anything other than a no | Proposal, rates, or deck sent (→ Pitched) | 25% |
| 3 | `Pitched` | Proposal / rate card / media kit / deck delivered | They respond to terms (→ Negotiating) or accept (→ Verbal) | 40% |
| 4 | `Negotiating` | Price, deliverables, timing, or exclusivity under active discussion | Terms agreed (→ Verbal) or walked (→ Lost) | 65% |
| 5 | `Verbal` | Agreed in principle; contract, IO, or PO pending signature | Signed (→ Closed Won) or died in legal (→ Lost) | 90% |
| 6 | `Closed Won` | Signed / countersigned / PO issued | Terminal | 100% |
| 7 | `Closed Lost` | Explicit no, went to a competitor, budget killed, or cadence fully exhausted | Terminal (may be revived into Nurture) | 0% |
| — | `Nurture` | Not now, but a real future window exists. Requires a `Revisit Date`. | Revisit date arrives → back to `Contacted` | 5% |

**Stage discipline rules:**

- Never advance a stage on the seller's action alone. Sending a proposal moves a
  deal to `Pitched`; a proposal being *opened* or *promised* does not.
- "Sounds great, send me something" is `Engaged`, not `Pitched`.
- "Let me run it by my team" after seeing pricing is `Negotiating`, not `Verbal`.
- A deal cannot skip backwards silently. If a deal regresses (e.g. `Negotiating`
  → `Engaged` because the champion left), record the reason as a dated note.
- `Closed Lost` is not a failure state to be avoided. A pipeline with no losses
  is a pipeline full of lies, and it destroys forecast accuracy.

---

## Follow-up cadence

Each stage has a **silence SLA** — the number of days without contact after which
a follow-up is owed. The clock starts on `Last Contact Date` regardless of who
sent the last message, with one exception: if **they** sent the last message and
it needs a reply, the SLA is always 1 day (see `Needs Reply` below).

| Stage | Silence SLA | Rationale |
|-------|-------------|-----------|
| `Prospect` | 2 days | A prospect sitting uncontacted is not a prospect, it's a note. |
| `Contacted` | See touch ladder below | Cold outreach needs a sequence, not a single nudge. |
| `Engaged` | 4 days | Warm but unqualified — stay present without crowding. |
| `Pitched` | 3 days | Proposals decay fastest. Attention peaks in the first 72 hours. |
| `Negotiating` | 2 days | Active negotiation loses momentum almost immediately. |
| `Verbal` | 3 days | Contract chase. Polite persistence, escalating to their ops/legal contact. |
| `Nurture` | Revisit Date | No nudges until the date arrives. |
| Closed | — | No cadence. |

### The `Contacted` touch ladder

Cold outreach that stops after one email leaves most of the pipeline on the
table. Run this ladder, then stop cleanly.

| Touch | Day (from first send) | Angle |
|-------|----------------------|-------|
| 1 | 0 | Initial outreach — specific, relevant, one clear ask |
| 2 | +3 | Bump with new value (a stat, a case study, a recent result) |
| 3 | +7 | Different angle entirely — new hook, do not restate touch 1 |
| 4 | +14 | Social proof / relevant win with a comparable brand |
| 5 | +21 | Breakup email — permission to close the file |

After touch 5 with no response: move to `Nurture` with a revisit date 90 days
out, or `Closed Lost` with reason `No response — cadence exhausted`.

**Never send more than one follow-up per 48 hours to the same contact**, at any
stage. Two nudges in one day reads as desperate and is the fastest way to get
filtered.

### `Needs Reply` overrides everything

If the most recent message in a thread is **inbound** and unanswered, that deal
is the top of the list regardless of stage or value. Response time is the single
highest-leverage variable in partnership sales. Target: same business day.

---

## Health scoring

`Health` is derived, never entered by hand. Compute it as
`days_silent` measured against the deal's stage SLA:

| Health | Condition | Meaning |
|--------|-----------|---------|
| `🔴 Needs Reply` | Last message is inbound and unanswered | Act today |
| `🟢 Healthy` | `days_silent` < SLA | On track, no action needed |
| `🟡 Due` | `days_silent` >= SLA and < 2× SLA | Follow-up owed now |
| `🟠 Cooling` | `days_silent` >= 2× SLA and < 3× SLA | Slipping — change the angle, don't just bump |
| `🔴 Cold` | `days_silent` >= 3× SLA | Needs a revival play or an honest reclassification |
| `⚪ Scheduled` | Stage is `Nurture` and revisit date is future | Intentionally dormant |
| `✅ Closed` | `Closed Won` or `Closed Lost` | Out of pipeline |

A `Cold` deal in `Negotiating` is a bigger emergency than a `Cold` deal in
`Contacted` — sort the digest by `stage_probability × value × staleness`, not by
staleness alone.

---

## Forecast math

- **Weighted value** = `Value × Probability` (probability from the stage table).
- **Committed** = sum of `Verbal` + `Closed Won` in the period.
- **Best case** = sum of all open weighted values.
- **Coverage ratio** = total open pipeline ÷ quota for the period. Healthy is
  3×. Below 2.5× means the top of the funnel needs work now, not next month.
- **Slip risk** = any deal with a `Close Date` in the current period whose health
  is `Cooling` or worse. Surface these separately; they are what turns a good
  forecast into a missed one.

If a deal has no `Value`, use the `Default Deal Value` from the Config tab so
forecast math stays intact. Flag it in the digest as an estimate.

---

## Stage-change triggers to detect from email

When reading a thread, map language to stage changes:

| Signal in their reply | Move to |
|-----------------------|---------|
| Any substantive reply from a cold contact | `Engaged` |
| "Send me your rates / a proposal / your deck / media kit" | stays `Engaged` until sent, then `Pitched` |
| "What would this cost" / "can you do X instead" / "our budget is" | `Negotiating` |
| "Let's do it" / "we're in" / "send the contract" | `Verbal` |
| "Signed" / "countersigned" / attached executed agreement / PO number | `Closed Won` |
| "We're going a different direction" / "no budget this year" / "we've signed with" | `Closed Lost` |
| "Not right now, check back in Q3" / "revisit after the season" | `Nurture` + revisit date |
| Out-of-office auto-reply | No stage change. Push next action to their return date. |

Out-of-office replies must never be counted as engagement, and must never reset
the touch counter.
