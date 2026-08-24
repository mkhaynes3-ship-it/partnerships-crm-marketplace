# Setup Guide

Everything you need connected and turned on, in order. About 15 minutes, most
of it Claude working while you skim a list.

---

## Before you start

**You need the Claude desktop app.** Not the website, not the phone app. The
dashboard lives in the desktop sidebar and won't appear anywhere else.

Download it at [claude.ai/download](https://claude.ai/download) if you don't
have it. Everything else in this guide happens inside that app.

---

## Step 1 — Install the plugin

Open the `.plugin` file that came with this guide. Claude shows you the contents
and an **Install** button. Click it.

That's the whole install. Nothing to configure yet.

---

## Step 2 — Connect Gmail

This is the only connector that's genuinely required.

**Settings → Connectors → Gmail → Connect.** Sign in with the Google account
your brand outreach actually lives in. If you run outreach from a work address
and personal mail from another, connect the work one.

When Google shows the permission screen, it will ask to **read, compose, send,
and permanently delete** mail. That wording is Google's, and it's the same
scope every email tool requests — it can't be narrowed per-app. What this plugin
actually does with it:

| Permission | What the plugin uses it for |
|---|---|
| Read mail | Find outreach threads, see who replied and when |
| Manage labels | Create and apply the `CRM/` labels — nothing outside that prefix |
| Compose drafts | Write follow-ups into your drafts folder |
| Send | **Only** the daily digest, to yourself |
| Delete | Never used |

The plugin never emails a contact. That rule is written into every skill and is
not a setting you can flip on by accident.

### Also turn on

- **Google Calendar** (optional) — only if you want deal deadlines on your
  calendar. Skip it otherwise; nothing breaks.
- **Google Drive** — not needed. Earlier versions used a spreadsheet; this one
  doesn't.

---

## Step 3 — Check your permission settings

**Settings → Connectors → Gmail → Permissions.**

Recommended settings for the first two weeks:

| Action | Setting | Why |
|---|---|---|
| Search / read mail | **Allow** | Runs constantly; approving each read is unusable |
| Create labels | **Allow** | Setup creates nine at once |
| Apply / remove labels | **Allow** | Every sync moves deals between stages |
| Create drafts | **Allow** | Drafts are harmless — nothing leaves your outbox |
| **Send email** | **Ask every time** | ← the important one |

Setting **send** to *ask every time* is your hard backstop. The plugin is built
never to email a contact, but a permission prompt is a guarantee rather than a
promise. The only thing that should ever trigger it is your own daily digest,
arriving from you, to you. **If you ever see a send prompt addressed to a brand
contact, say no and tell whoever set this up.**

After a couple of weeks, if the only prompts you've seen are your own digests,
you can switch send to *Allow* — or leave it, since one prompt a day is cheap.

---

## Step 4 — Run setup

Start a new Cowork task in the desktop app and say:

> **Set up my partnerships CRM**

Claude will ask four quick questions:

1. **Your name and company** — goes in email signatures
2. **Typical deal size** — a range is fine; it's used to estimate deals you
   haven't priced yet
3. **Your target for the quarter or month** — optional; it turns on the coverage
   ratio
4. **What time you want the digest** — default is 8am on weekdays

It will also ask which email domains are **internal** — your own company's
domain, and anyone else who shouldn't get logged as a deal. Answer this one
carefully. Without it, your first pipeline fills up with coworkers.

### Then it scans your inbox

Claude reads the last 90 days of mail looking for partnership conversations,
guesses the stage of each, and **shows you a table before writing anything.**

**Actually read this table.** Stage inference from old email is genuinely a
guess — it can read "sounds great, send me a proposal" as further along than it
is. Strike the rows that aren't real deals, correct the stages that are wrong,
and add deal values if you know them off the top of your head.

Twenty minutes here is what makes the next six months accurate. It is the single
highest-value part of setup.

---

## Step 5 — Confirm it worked

Three things should now be true:

1. **In Gmail**, a `CRM` label group in the left sidebar with nine labels under
   it, and your deals filed into them. Check on your phone too — this is how
   you'll see your pipeline when you're not at your desk.
2. **In the Claude desktop sidebar**, a **Partnerships Pipeline** dashboard
   showing your funnel and forecast.
3. **In Settings → Scheduled tasks**, a task called *Partnerships CRM — Daily
   Digest*.

If any is missing, say *"check my CRM setup"* and Claude will find the gap.

---

## Step 6 — Add deal values

The forecast and the daily priority ranking are both driven by deal value. Until
values are in, the CRM will tell you *who* to follow up with but not *which
matters most*.

Say: **"walk me through adding values to my top deals"** — you'll get a list and
can rattle off numbers. Five minutes, and it's the difference between a to-do
list and a pipeline.

---

## Daily use

| Say this | What happens |
|---|---|
| "Sync my CRM" | Reads recent email, moves stages, flags what changed |
| "Who do I need to follow up with?" | Ranked list + drafts waiting in Gmail |
| "Show my pipeline" | Opens the dashboard, refreshed |
| "Add Nike at $30k, they replied yesterday" | Logs the deal |
| "Move Gymshark to negotiating" | Updates stage and label |
| "We won Vuori — $28k" | Closes it and asks why you won |
| "Draft a follow-up to Lululemon" | Reads the thread, writes the draft |
| "Nike is actually $60k" | Corrects the value |

The morning digest arrives on its own. Most days you'll skim it, open two or
three drafts, and hit send.

---

## Where your data lives

Worth understanding, because it's why this is hard to break:

- **Deal stages live in your Gmail labels.** Not in a database somewhere — in
  your own Gmail, on every device, readable without Claude.
- **Everything else about the conversation is read live from the thread.** Who
  replied last, when, how many times you've followed up. Nothing to fall out of
  sync.
- **Deal values, close dates, and notes live in the dashboard**, since email
  doesn't know them.
- **Every digest email contains a hidden backup** of your deal values. If the
  dashboard is ever deleted, the CRM rebuilds itself from your labels and the
  last digest.

You can drag a thread to a different `CRM/` label in Gmail yourself, and the next
sync will respect it.

---

## Troubleshooting

**"The dashboard didn't appear"**
The desktop app wasn't connected, or you ran the task from the web. Re-run
*"show my pipeline"* from the desktop app.

**"It says it can't read the dashboard"**
The artifact got offloaded to cloud-only storage. Open it once in the sidebar to
pull it back down. Claude will recover values from your last digest in the
meantime — nothing is lost.

**"My first backfill is full of coworkers"**
The internal domains weren't set. Say *"exclude @yourcompany.com from my CRM"*
and re-run the sync.

**"The digest never arrived"**
Check Settings → Scheduled tasks that it exists and is enabled. Note the time is
stored in UTC — if you moved time zones, say *"move my digest to 8am"* and it
recalculates.

**"It moved a deal to the wrong stage"**
Just tell it: *"Nike is still at pitched, not negotiating."* It fixes the label
and the dashboard.

**"Claude asked to send an email to a brand contact"**
Say no. The plugin should never do this. Report it to whoever set this up.

---

## What it will never do

- Email a contact. Drafts only, permanently.
- Touch any Gmail label outside the `CRM/` prefix.
- Delete an email, ever.
- Delete a deal — deals close, they don't disappear.
- Make up a statistic, case study, or comparable brand for a follow-up. If a
  proof point isn't real, it asks you for one.
- Send two follow-ups to the same person within 48 hours.
