# Makenna's Plugins

A plugin marketplace for [Claude](https://claude.ai/download) — Cowork and Claude Code.

## Partnerships CRM

A CRM that lives in your Gmail instead of another tab you have to remember to open.

- Reads your inbox and works out which threads are live brand conversations
- Files each one by stage using Gmail labels — your pipeline is visible **in Gmail, on your phone**
- Emails you every weekday morning with who's waiting on a reply, what's going cold, and follow-ups **already written and sitting in your drafts**
- Renders a dashboard with your forecast, what's slipping, and what's closing soon

**It drafts emails. It never sends them.** Every follow-up waits in your drafts. The only mail it sends is your own morning digest, to you.

---

## Install

**In the Claude desktop app** (Cowork or Desktop):

1. Open the **Cowork** tab, then **Customize** in the left sidebar
2. Go to the **Plugins** tab
3. Under **Personal plugins**, click **+** → **Add marketplace**
4. Choose **Add from a repository** and paste this repo's URL
5. Find **Partnerships CRM** in the list and click **Install**

**In Claude Code:**

```
/plugin marketplace add YOUR-GITHUB-USERNAME/partnerships-crm-marketplace
/plugin install partnerships-crm@makenna-plugins
```

---

## Then set it up

Connect the **Gmail** connector, then start a task and say:

> **Set up my partnerships CRM**

Full walkthrough, recommended permission settings, and troubleshooting:
**[SETUP.md](plugins/partnerships-crm/SETUP.md)**

The one setting worth getting right: in **Settings → Connectors → Gmail → Permissions**, set **Send email** to **Ask every time**. The plugin is built never to email a contact, but a permission prompt is a guarantee rather than a promise.

---

## Requirements

- Claude **desktop app** — the dashboard renders in the desktop sidebar only
- **Gmail** connector — the only one required
- **Google Calendar** — optional, only for deal deadlines on your calendar

No spreadsheet, no Google Drive, no external CRM account.

## Updating

When this repo changes, re-sync the marketplace from the same Plugins screen and your installed version updates. Your deal data is unaffected — it lives in your Gmail labels and your own dashboard, not in the plugin.
