# Partnerships CRM

A CRM that lives where the deals actually happen — your inbox. It keeps a
pipeline current from your Gmail, tells you who's about to go cold,
writes the follow-ups, and shows you a real forecast.

**It drafts. It never sends.** Every follow-up waits in your Gmail drafts for
you to read and send yourself.

---

## Getting started

**See `SETUP.md` for the full walkthrough**, including which permissions to set
and why. The short version:

1. Use the **Claude desktop app** (the dashboard doesn't render on web or mobile)
2. Connect **Gmail** — the only required connector
3. Set Gmail's **send** permission to *ask every time* as a backstop
4. Say **"Set up my partnerships CRM"**

Setup creates your `CRM/` Gmail labels, scans the last 90 days of mail to find
deals you're already working (you approve the list before anything is written),
builds your dashboard, and schedules the daily digest.

---

## What to say

| Say this | What happens |
|----------|--------------|
| "Sync my CRM" | Reads recent email, updates stages and dates, flags what changed |
| "Who do I need to follow up with?" | Ranked list, plus drafted follow-ups in your Gmail drafts |
| "Show my pipeline" | Interactive dashboard — forecast, funnel, what's slipping |
| "Add Nike to my pipeline" | Logs a new deal, pulling contact details from your email |
| "Move Gymshark to negotiating" | Updates the stage, the label, and the next action |
| "We won the Vuori deal — $28k" | Closes it won and logs the reason |
| "Draft a follow-up to Lululemon" | Reads the thread, picks the right play, writes the draft |
| "What's my forecast this quarter?" | Coverage ratio, weighted pipeline, and what's at risk |

---

## The daily digest

Every weekday morning you get one email:

- **Who's waiting on a reply from you** — the single highest-leverage thing in
  partnership sales
- **Follow-ups already drafted** and sitting in your drafts, with the opening
  line quoted so you can approve most of them from your phone
- **Deals going cold**, each with a specific recommended play — not "follow up"
  but "the deck landed and the number didn't; offer a phased version"
- **Closing soon**, and what's blocking each one

Change the time, the days, or turn it off by saying so.

---

## How the pipeline works

Eight stages: `Prospect` → `Contacted` → `Engaged` → `Pitched` → `Negotiating`
→ `Verbal` → `Closed Won` / `Closed Lost`, plus `Nurture` for the not-right-now
deals worth reviving.

Each stage has a **silence limit** — how many days without contact before a
follow-up is owed. `Negotiating` is 2 days; `Contacted` runs a five-touch
sequence over three weeks and then stops cleanly. Deals are ranked by value ×
stage probability × how overdue they are, so a stalled $40k negotiation always
outranks a cold $2k cold email.

Stages only advance on something *they* did. Sending a proposal moves a deal to
`Pitched`; a proposal being promised does not. That's what keeps the forecast
honest.

---

## Where your data lives

There's no spreadsheet to maintain, because most of a CRM is already in your
inbox:

- **Deal stages live in your Gmail labels** — visible on every device, readable
  without Claude, and draggable by hand. Move a thread to a different `CRM/`
  label yourself and the next sync respects it.
- **Contact, dates, who replied last, and how many times you've followed up**
  are read live from the thread. Nothing to fall out of sync.
- **Deal values, close dates, and notes** live in the dashboard, since email
  doesn't know them. Change them by saying so: *"Nike is actually $60k."*
- **Every digest email carries a hidden backup** of your values. Delete the
  dashboard and the CRM rebuilds from your labels and the last digest.

---

## Skills

| Skill | Purpose |
|-------|---------|
| `crm-setup` | One-time setup, including backfilling deals from your inbox |
| `crm-dashboard` | Renders the pipeline **and stores your deal values** |
| `crm-sync` | Reconcile Gmail against your stored deal data |
| `crm-digest` | Daily priorities, drafted follow-ups, digest email |
| `crm-log` | Add or update a single deal by hand |
| `crm-followup` | Draft a specific follow-up or cold outreach |

---

## What it will never do

- Send an email to a contact. Drafts only, permanently.
- Delete a deal. Deals close; they don't disappear.
- Delete an email, ever.
- Touch Gmail labels outside the `CRM/` prefix.
- Invent a statistic, case study, or comparable brand for a follow-up. If a
  proof point isn't real, it asks you for one.
- Send two follow-ups to the same person within 48 hours.

---

## Requirements

- Claude **desktop app** — the dashboard renders in the desktop sidebar only
- **Gmail** connector — required
- **Google Calendar** — optional, only for putting deal deadlines on your calendar

No Google Drive, no spreadsheet, no external CRM account.

Full setup instructions and recommended permission settings: **`SETUP.md`**.
