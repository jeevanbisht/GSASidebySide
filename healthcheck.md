# Healthcheck — Source Change Detection

This document defines how agents monitor the Microsoft Learn source pages for changes
and notify the user when upstream documentation has been updated.

---

## Purpose

Each coexistence guide in this repo is built from an official Microsoft Learn page.
When Microsoft updates those pages (new scenarios, changed IPs/FQDNs, revised steps),
our guides become stale. This healthcheck system detects drift early.

---

## Source Registry

> ⚠️ **There are SIX (6) tracked guides — check ALL of them, every time.**
> The authoritative registry is [`source/snapshot.md`](source/snapshot.md), **not** the
> `agents.md` "Source Change Detection" table (which only lists 4 vendors and is incomplete).
> Always iterate over every row in `source/snapshot.md`. If a count of guides is ever < 6,
> something was skipped — stop and recount.

**The 6 guides that MUST be checked on every healthcheck:**

| # | Vendor | Guide HTML | Source slug |
|---|--------|-----------|-------------|
| 1 | Zscaler | `Zscaler/gsa-zscaler-coexistence.html` | `how-to-zscaler-coexistence` |
| 2 | Cisco Umbrella | `CiscoUmbrella/gsa-cisco-coexistence.html` | `how-to-cisco-coexistence` |
| 3 | Cisco Secure Access | `CiscoSecureAccess/gsa-cisco-secure-access-coexistence.html` | `how-to-cisco-secure-access-coexistence` |
| 4 | Cisco VPN | `CiscoVPN/gsa-cisco-vpn-coexistence.html` | `how-to-cisco-vpn-coexistence` |
| 5 | Palo Alto | `PaloAlto/gsa-paloalto-coexistence.html` | `how-to-palo-alto-coexistence` |
| 6 | Netskope | `Netskope/gsa-netskope-coexistence.html` | `how-to-netskope-coexistence` |

All tracked sources are recorded in [`source/snapshot.md`](source/snapshot.md) with:

- **Source URL** — the canonical Microsoft Learn page
- **ms.date** — the `ms.date` metadata field at time of last capture
- **git_commit_id** — the `gitcommit` hash at time of last capture
- **word_count** — approximate word count for quick diff detection

> 🔁 **Self-check before reporting:** confirm your results table has exactly 6 rows,
> one per guide above. Discover the file list dynamically too —
> `Get-ChildItem -Recurse -Filter "*coexistence*.html"` should return 6 files.

---

## How It Works

### On Every Agent Invocation (automatic)

Per the `agents.md` "Source Change Detection" section, agents **must**:

1. Fetch **every** source URL listed in `source/snapshot.md` (all 6 — do not stop at 4)
2. Extract the `ms.date`, `git_commit_id`, and `word_count` from the page metadata
3. Compare against the values in `source/snapshot.md`
4. Report any differences to the user, with a results table containing **all 6 guides**

### Change Indicators (any one triggers a notification)

| Signal | What Changed | Likely Impact |
|--------|-------------|---------------|
| `ms.date` differs | Page was republished | Content updated |
| `git_commit_id` differs | Source markdown changed | Could be typo fix or major rewrite |
| `word_count` delta > 50 | Significant content added/removed | New scenarios or removed steps |

### Agent Notification Format

When changes are detected, report:

```
⚠ Source changes detected:

{Vendor} (source/snapshot.md last: {old_date})
  - ms.date changed: {old} → {new}
  - git_commit_id changed: {old_short} → {new_short}
  - word_count changed: {old} → {new} ({+/-delta})

{Vendor} — No changes detected. ✓

How would you like to proceed?
  1. View detailed diff for {vendor}
  2. Update the guide to match new source
  3. Skip — acknowledge but take no action
```

### After User Approves Update

1. Fetch the full source page content
2. Identify what changed (new scenarios, IPs, FQDNs, steps)
3. Apply changes to the corresponding HTML guide
4. Bump the version (minor for content fixes, major for new scenarios)
5. Update `source/snapshot.md` with the new metadata values
6. Commit and push

---

## Manual Healthcheck Command

To trigger a healthcheck manually:

```
copilot "Run a healthcheck on the GSA coexistence guides — check all sources in source/snapshot.md for updates"
```

---

## Adding a New Source

When a new vendor guide is added:

1. Add a row to `source/snapshot.md` with the source URL and current metadata
2. The healthcheck will automatically include it in future checks

---

## Index.html Health Tracking

The healthcheck also updates `index.html` to reflect source freshness visually. This ensures anyone visiting the landing page can see at a glance whether guides are current.

### What Gets Updated

1. **Guide card metadata** — each card shows:
   - `Guide v{X.Y} · Built {date}` — the guide's internal version and when it was last built/committed
   - `Source: {date}` — when the Microsoft Learn source was last updated (`updated_at` field)
   - A green dot (🟢) if the guide was built after the source update, orange (🟠) if stale

2. **Reference Documentation section** — each link displays `Updated {date}` from the source's `updated_at` metadata

3. **Source Health Check table** — a summary table at the bottom with columns:
   | Column | Source |
   |--------|--------|
   | Vendor | Guide name |
   | Guide Version | From `Version X.Y` in the guide's footer |
   | Guide Built | Last git commit date for that guide's HTML file |
   | Source Updated | `updated_at` field from the Microsoft Learn page |
   | Status | "Current" (green) if built > source date, "Behind" (orange) if source is newer |

### How to Update index.html During Healthcheck

After fetching source metadata and comparing:

1. **Get guide versions**: Parse each guide's `<p class="version-line">Version X.Y</p>`
2. **Get build dates**: Use `git log -1 --format="%ai" -- {path}` for each guide
3. **Get source dates**: Use `updated_at` from the fetched Microsoft Learn page metadata
4. **Determine status**: If guide build date > source updated_at → "Current", else → "Behind"
5. **Update index.html**:
   - Update `.card-versions` spans with current values
   - Update `.doc-meta` spans with source dates
   - Update the health table `<tbody>` rows
   - Update the "Last health check: {date}" line at the bottom

### CSS Classes for Status

```css
/* Card-level indicators */
.dot--fresh   → green (guide is current)
.dot--stale   → orange (guide is behind source)

/* Health table indicators */
.dot--current → green
.dot--behind  → orange
```

### Example: Marking a Guide as Stale

If a source was updated on Jun 20 but the guide was last built Jun 16:

```html
<!-- Card -->
<div class="card-versions">
  <span>Guide v1.0 · Built Jun 16, 2026</span>
  <span><span class="dot dot--stale"></span>Source: Jun 20, 2026</span>
</div>

<!-- Health table row -->
<td><span class="health-status"><span class="dot dot--behind"></span>Behind</span></td>
```

---

## File Structure

```
source/
  snapshot.md      ← Current state of all tracked source pages (metadata fingerprint)
healthcheck.md     ← This file (process documentation)
index.html         ← Landing page with visual health indicators
```
