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

All tracked sources are recorded in [`source/snapshot.md`](source/snapshot.md) with:

- **Source URL** — the canonical Microsoft Learn page
- **ms.date** — the `ms.date` metadata field at time of last capture
- **git_commit_id** — the `gitcommit` hash at time of last capture
- **word_count** — approximate word count for quick diff detection

---

## How It Works

### On Every Agent Invocation (automatic)

Per the `agents.md` "Source Change Detection" section, agents **must**:

1. Fetch each source URL listed in `source/snapshot.md`
2. Extract the `ms.date`, `git_commit_id`, and `word_count` from the page metadata
3. Compare against the values in `source/snapshot.md`
4. Report any differences to the user

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

## File Structure

```
source/
  snapshot.md      ← Current state of all tracked source pages (metadata fingerprint)
healthcheck.md     ← This file (process documentation)
```
