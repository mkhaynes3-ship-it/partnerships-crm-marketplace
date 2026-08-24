# Data Model

**Gmail is the database. The dashboard is the view.**

There is no spreadsheet. Everything that can be derived from email is derived
live; only the handful of facts email cannot know are stored. This is what makes
the CRM survive a lost artifact, a new device, or a six-month gap.

---

## Where each fact lives

| Fact | Source | Why |
|------|--------|-----|
| Deal stage | Gmail label `CRM/…` | Writable, durable, visible in Gmail on her phone |
| Needs reply | Gmail label `CRM/Needs Reply` | Same |
| Brand, contact name, email, title | The thread + signature block | Always current; never goes stale |
| Last contact date & direction | Newest message in the thread | Cannot drift out of sync |
| Touch count | Count of her outbound messages since the stage label was applied | Self-correcting |
| Thread link | Gmail thread ID | Derived |
| **Deal value** | Dashboard artifact JSON | Email doesn't know it |
| **Close date** | Dashboard artifact JSON | Email doesn't know it |
| **Notes / objections** | Dashboard artifact JSON | Her judgment, not a message |
| **Config** (name, currency, quota, digest time) | Dashboard artifact JSON | Set once at setup |

Nothing is stored that email already knows. That rule is what prevents the
tracker-drift problem every spreadsheet CRM eventually has.

---

## Gmail label taxonomy

Setup creates these. A thread carries **exactly one** stage label.

```
CRM/
├── 1 Contacted
├── 2 Engaged
├── 3 Pitched
├── 4 Negotiating
├── 5 Verbal
├── Won
├── Lost
├── Nurture
└── Needs Reply      (applied and removed independently of stage)
```

Suggested colors via `colorPreset` so the pipeline reads at a glance in Gmail:

| Label | Preset |
|-------|--------|
| `1 Contacted` | `LABEL_COLOR_PRESET_LIGHT_GRAY` |
| `2 Engaged` | `LABEL_COLOR_PRESET_BLUE` |
| `3 Pitched` | `LABEL_COLOR_PRESET_TEAL` |
| `4 Negotiating` | `LABEL_COLOR_PRESET_PURPLE` |
| `5 Verbal` | `LABEL_COLOR_PRESET_DARK_BLUE` |
| `Won` | `LABEL_COLOR_PRESET_GREEN` |
| `Lost` | `LABEL_COLOR_PRESET_GRAY` |
| `Nurture` | `LABEL_COLOR_PRESET_YELLOW` |
| `Needs Reply` | `LABEL_COLOR_PRESET_RED` |

Rules:

- On a stage change, remove the old stage label and apply the new one — never
  leave two.
- Create labels with `autoCreateParentLabels: true` so the `CRM` parent appears.
- Never create, rename, or delete a label outside the `CRM/` prefix.
- Label display names are what she sees; label **IDs** are what Gmail search
  accepts. Resolve names to IDs with `list_labels` before querying.

---

## The artifact store

The dashboard artifact (`partnerships-pipeline`) carries its own data inside the
HTML, in exactly this block:

```html
<script id="crm-store" type="application/json">
{ …the store… }
</script>
```

Read it back with `device_stage_files` (`artifact_ids: ["partnerships-pipeline"]`),
parse the JSON out of that script tag, merge, and write the full document back
with `update_artifact`.

### Store shape

```json
{
  "version": 1,
  "updated": "2026-08-17T08:00:00-05:00",
  "config": {
    "ownerName": "Jordan Ellis",
    "ownerEmail": "jordan@example.com",
    "company": "",
    "currency": "USD",
    "defaultDealValue": 5000,
    "quota": 250000,
    "quotaPeriod": "Quarter",
    "digestTime": "08:00",
    "digestDays": "Mon-Fri",
    "maxDraftsPerDigest": 5,
    "syncLookbackDays": 30,
    "excludedDomains": ["lovb.com"]
  },
  "deals": {
    "THREAD_ID_abc123": {
      "brand": "Nike",
      "value": 45000,
      "closeDate": "2026-09-30",
      "notes": [
        {"date": "2026-08-14", "text": "Dana confirmed Q4 budget exists; needs sign-off from Marcus."}
      ],
      "closedReason": null,
      "revisitDate": null,
      "manualStage": null
    }
  },
  "closedThisPeriod": {"won": [], "lost": []},
  "templateOverrides": {}
}
```

Rules for the store:

- **Keyed by Gmail thread ID.** That is the only identifier that never changes.
- Store *only* the fields above. Anything derivable from Gmail is deliberately
  absent — if it's in both places, they will disagree.
- `manualStage` is an escape hatch: if she overrides a stage verbally and the
  label write fails, record it here and reconcile on the next sync.
- `notes` is append-only, newest last. Never rewrite an existing note.
- `templateOverrides` maps a template ID to her edited version and always wins
  over the plugin defaults.
- One deal per thread. If a brand runs two separate deals, they are two threads.

### Merge order on every sync

1. Read Gmail — this establishes which deals exist, their stage, and all dates.
2. Read the artifact store — this supplies value, close date, and notes.
3. A thread present in Gmail but missing from the store gets a new store entry
   with `value: null`.
4. A store entry whose thread no longer appears in Gmail is **kept**, not
   deleted — it may simply have fallen outside the lookback window.
5. Gmail always wins on stage and dates. The store always wins on value, close
   date, and notes.

---

## Recovery

The store is the only thing that isn't inherently durable, so it is backed up
automatically and continuously.

**Every digest email ends with a hidden recovery block:**

```html
<!-- CRM-RECOVERY-V1
{"THREAD_ID_abc123":{"brand":"Nike","value":45000,"closeDate":"2026-09-30"}}
-->
```

Invisible when reading the email; trivially recoverable when needed.

If the artifact cannot be read — deleted, evicted to cloud-only storage, or the
desktop app isn't connected — recover in this order:

1. Search Gmail for the most recent digest (`subject:Pipeline from:me`), parse
   the `CRM-RECOVERY-V1` block, and restore the store.
2. Rebuild the deal list from `CRM/` labels, which never left.
3. Only then ask her to re-enter anything still missing.

**Never block on a missing store.** Deal values affect the forecast; they do not
affect who needs a follow-up today. Run the digest, note that values are
temporarily unknown, and keep going.

---

## Degraded modes

| Situation | Behavior |
|-----------|----------|
| Desktop app not connected | Labels and drafts still work. Skip the artifact update; say the dashboard will refresh next time. |
| Artifact is a cloud placeholder | Recover from the last digest email. Suggest she open the artifact once to pull it local. |
| Gmail unreachable | Stop. Send nothing, change nothing, say why. |
| A label write fails | Record the intended stage in `manualStage` and retry next sync. Never silently drop it. |

The follow-up engine must keep working in every row above except the last one.
