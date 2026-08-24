# Follow-up Template Library

The plugin's built-in defaults. If `templateOverrides` in the dashboard store
holds a version of a template, that version wins — she can reword any of these
by saying so, without reinstalling the plugin.

## Variables

`{{FirstName}}` `{{ContactName}}` `{{Brand}}` `{{Company}}` `{{OwnerName}}`
`{{Value}}` `{{LastTouchSummary}}` `{{DaysSilent}}` `{{NextAction}}`

`{{LastTouchSummary}}` must be generated from the actual thread — one clause
naming the specific thing discussed ("the Q4 activation deck", "your March
launch timing"). Never fill it with a generic phrase.

---

## Universal rules for every draft

1. **Thread, don't restart.** Follow-ups reply in the existing thread. A new
   subject line orphans the context and reads as a mass send. Only start a new
   thread on a genuine new angle (touch 3) or after 30+ days of silence.
2. **One ask per email.** Two questions halves the reply rate. Pick the single
   next commitment you actually want.
3. **Under 90 words** for any follow-up. If the pitch needs more, it needs a
   call, not a longer email.
4. **Never open with an apology.** No "sorry to bother", "just checking in",
   "circling back", "bumping this to the top of your inbox", "wanted to touch
   base", or "per my last email". These signal that the email carries no new
   information — because it usually doesn't. Every follow-up must add something.
5. **Make the no easy.** A fast no is worth more than a slow maybe. Deals that
   die quickly free up the calendar; deals that linger cost real money.
6. **Match their register.** If they write in three-word replies, do the same.
7. **Specific > enthusiastic.** One concrete number beats three exclamation
   points.
8. **No fake deadlines.** Manufactured urgency is transparent and expensive.
   Real constraints ("the Q4 slate locks Oct 15") are fine and effective.

---

## `Contacted` — the touch ladder

### `contacted-t2` · Day +3 · Bump with new value

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> One thing I left out: {{NewProofPoint}}.
>
> Worth 15 minutes to see if there's a fit for {{Brand}}?
>
> {{OwnerName}}

`{{NewProofPoint}}` is a real, specific fact — a result, an audience number, a
comparable partnership. If there isn't one, don't send touch 2; go straight to
touch 3.

### `contacted-t3` · Day +7 · New angle

> **Subject:** {{Brand}} × {{Company}} — {{SpecificIdea}}
>
> Hi {{FirstName}} —
>
> Different thought than my last note. {{SpecificIdea}} — {{OneLineWhy}}.
>
> If this isn't your lane, happy to be pointed to whoever owns it.
>
> {{OwnerName}}

New thread is allowed here. The "point me to the right person" close is the
highest-yield line in cold outreach; it gives a reply that costs them nothing.

### `contacted-t4` · Day +14 · Social proof

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> We just wrapped something similar with {{ComparableBrand}} — {{Result}}.
>
> Same play could work for {{Brand}}. Want me to send the one-pager?
>
> {{OwnerName}}

Only use a real comparable brand and a real result. If neither exists, skip to
touch 5.

### `contacted-t5` · Day +21 · Breakup

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> I'll stop here so I'm not cluttering your inbox. If {{Brand}} revisits
> partnerships later this year, I'm easy to find.
>
> {{OwnerName}}

The breakup email has the highest reply rate in the entire ladder. Keep it
gracious and genuinely final — no "unless…" hook at the end. If they reply,
move to `Engaged`.

---

## `Engaged` — `engaged-nudge` · SLA 4 days

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> Following up on {{LastTouchSummary}}. Is {{NextAction}} still the right next
> step, or has the timing shifted?
>
> {{OwnerName}}

Naming a possible timing shift gives them a graceful, honest out — which
produces a real answer instead of silence.

---

## `Pitched` — `pitched-nudge` · SLA 3 days

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> Any reactions to the proposal? Happy to adjust scope or phasing if the number
> is the sticking point.
>
> If it's easier, I can walk through it in 10 minutes this week.
>
> {{OwnerName}}

Naming price as a possible objection up front is what converts silence into a
counter. Most stalls at `Pitched` are budget, not interest.

### `pitched-nudge-2` · Day +7

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> Checking whether this is still live on your end. Totally fine either way — a
> no just helps me plan.
>
> {{OwnerName}}

---

## `Negotiating` — `negotiating-nudge` · SLA 2 days

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> Where did {{OpenPoint}} land on your side?
>
> If it helps move things, I can {{SpecificConcession}}.
>
> {{OwnerName}}

Never concede on price without taking something back — shorten the term, trim a
deliverable, move to net-30, add an option year. A concession given for free
teaches the other side that the next one is free too.

---

## `Verbal` — `verbal-nudge` · SLA 3 days

> **Subject:** (reply in thread)
>
> Hi {{FirstName}} —
>
> Contract's with {{TheirSide}} — anything you need from me to get it over the
> line? Happy to loop in whoever handles the paperwork directly.
>
> {{OwnerName}}

At `Verbal`, the risk is administrative, not commercial. After two unanswered
nudges, ask to be connected to their ops or legal contact — chasing the
champion for a signature they don't control wastes both people's time.

---

## `Nurture` — `nurture-revisit` · On revisit date

> **Subject:** {{Brand}} — picking this back up
>
> Hi {{FirstName}} —
>
> You mentioned {{RevisitReason}} — that window's about here.
>
> {{WhatsNew}}. Worth a fresh look?
>
> {{OwnerName}}

`{{WhatsNew}}` is mandatory. A revisit with no new information is just a
re-send, and it burns a warm contact.

---

## `Needs Reply` — inbound waiting on her

No template. Draft a real, specific answer to what they actually asked. If a
complete answer needs information she doesn't have, draft a holding reply that
commits to a time:

> Hi {{FirstName}} — got it. Let me check on {{OpenItem}} and come back to you
> by {{Day}}.

A holding reply within the same business day preserves far more deals than a
perfect reply three days later.

---

## Never draft

- Anything to a contact who has said no, unless the deal is in `Nurture` and the
  revisit date has arrived.
- A second follow-up within 48 hours of the previous one.
- Any message with a guilt hook ("I've reached out three times…").
- Anything to a `Closed Lost` contact without an explicit new reason.
- Replies to out-of-office auto-responses.
