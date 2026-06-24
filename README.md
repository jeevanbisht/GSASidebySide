# GSA Side-by-Side Coexistence Guides

Interactive, self-contained configuration guides for deploying **Microsoft Global Secure
Access (GSA)** alongside third-party security vendors (Zscaler, Cisco, Palo Alto, Netskope,
and more). Each guide walks an administrator through the supported coexistence scenarios,
visualizes the resulting traffic flow, and provides copy-ready configuration with
tenant-specific values.

🌐 **[Open the Landing Page](https://jeevanbisht.github.io/GSASidebySide/)** — central hub
linking every vendor guide.

---

## Table of Contents

- [What This Is](#what-this-is)
- [Available Guides](#available-guides)
- [Guide Features](#guide-features)
- [Repository Structure](#repository-structure)
- [Architecture](#architecture)
- [Anatomy of a Guide](#anatomy-of-a-guide)
- [Contributing a New Vendor Guide](#contributing-a-new-vendor-guide)
- [Source Healthcheck (Drift Detection)](#source-healthcheck-drift-detection)
- [Versioning](#versioning)
- [Deployment](#deployment)
- [Official Documentation](#official-documentation)

---

## What This Is

When an organization adopts Microsoft Global Secure Access, it rarely flips a switch and
removes its existing security vendor overnight. Instead, GSA and the incumbent vendor must
**coexist** — each owning a slice of the traffic (Internet, Microsoft 365, Private Access,
ZTNA, etc.) without conflicting.

Microsoft documents these coexistence patterns on [Microsoft Learn](https://learn.microsoft.com/en-us/entra/global-secure-access/),
but the official pages are dense reference text. This repository turns each one into an
**interactive single-page guide**:

- Pick your coexistence scenario from a card grid
- See an animated traffic-flow diagram for that scenario
- Follow accordion-style implementation steps
- Enter your **Tenant ID** / **Quick Access App ID** to auto-populate config blocks
- Copy ready-to-use bypass FQDNs, IP ranges, and PAC entries
- Track your progress with per-step verification checklists

Every guide is a **single `.html` file with zero external dependencies** — all CSS, JS, SVG
diagrams, and icons are inlined. Open it from disk, host it anywhere, or use the published
GitHub Pages site.

---

## Available Guides

| Vendor | Guide Source | Live Page |
|--------|--------------|-----------|
| Zscaler | [Source](Zscaler/gsa-zscaler-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/Zscaler/gsa-zscaler-coexistence.html) |
| Cisco Umbrella | [Source](CiscoUmbrella/gsa-cisco-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/CiscoUmbrella/gsa-cisco-coexistence.html) |
| Cisco Secure Access | [Source](CiscoSecureAccess/gsa-cisco-secure-access-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/CiscoSecureAccess/gsa-cisco-secure-access-coexistence.html) |
| Cisco VPN | [Source](CiscoVPN/gsa-cisco-vpn-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/CiscoVPN/gsa-cisco-vpn-coexistence.html) |
| Palo Alto Prisma Access | [Source](PaloAlto/gsa-paloalto-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/PaloAlto/gsa-paloalto-coexistence.html) |
| Netskope | [Source](Netskope/gsa-netskope-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/Netskope/gsa-netskope-coexistence.html) |

> **Preview:** An in-progress **iboss** guide lives under [`Preview/iboss/`](Preview/iboss/).
> It is not yet linked from the landing page and is excluded from the healthcheck registry
> until it is promoted to a top-level vendor folder.

---

## Guide Features

| Feature | Description |
|---------|-------------|
| **Scenario selection** | A 4-column card grid lets the user choose a coexistence pattern; each card lists what *GSA owns* vs. what the *vendor owns*. |
| **Traffic-flow diagrams** | Per-scenario animated SVG rendered inside a dark "HUD canvas" — color-coded paths (cyan = GSA, emerald/green = vendor), orthogonal connectors, floating particles. |
| **Step-by-step implementation** | Accordion steps with numbered sub-instructions and copy-to-clipboard config blocks. |
| **Tenant ID / Quick Access App ID input** | Enter your Entra Tenant ID and/or Quick Access App ID (GUID-validated) to replace `<tenantid>` and `<quickaccessappID>` placeholders across every config block. |
| **Copy gating** | Config blocks that need a GUID show a 🔒 lock and warning until the required ID is applied, preventing copy of placeholder text. |
| **Tenanted-endpoint badges** | Step headers display `Tenanted Endpoints: N` / `Quick Access App ID: N` counts so admins know which steps are tenant-specific. |
| **Progress tracking** | Per-step verification checklists gate a "Mark Complete" button; progress is tracked per scenario. |

---

## Repository Structure

```
GSASidebySide/
├── index.html                  ← Landing page (central hub + visual health table)
├── README.md                   ← This file
├── agents.md                   ← Build playbook / authoring spec for guides
├── healthcheck.md              ← Source drift-detection process documentation
├── source/
│   └── snapshot.md             ← Metadata fingerprint of every tracked Learn source
│
├── Zscaler/
│   └── gsa-zscaler-coexistence.html
├── CiscoUmbrella/
│   └── gsa-cisco-coexistence.html
├── CiscoSecureAccess/
│   └── gsa-cisco-secure-access-coexistence.html
├── CiscoVPN/
│   └── gsa-cisco-vpn-coexistence.html
├── PaloAlto/
│   └── gsa-paloalto-coexistence.html
├── Netskope/
│   └── gsa-netskope-coexistence.html
│
└── Preview/
    └── iboss/                  ← Work-in-progress guide (not yet published)
        ├── gsa-iboss-coexistence.html
        └── INTERNAL-SSE_Coexistence_iBoss_Solution_Guide_new_UI_ver2.docx
```

| File / Folder | Role |
|---------------|------|
| `index.html` | The GitHub Pages entry point. Shows vendor cards, reference-doc links, and a **Source Health Check** table reflecting guide freshness. |
| `agents.md` | The canonical **build playbook**. Defines page structure, the HUD diagram system, GUID/badge logic, color palette, spatial layout rules, and the new-vendor checklist. Read this before authoring or editing any guide. |
| `healthcheck.md` | Documents the automated process for detecting upstream Microsoft Learn changes and keeping guides current. |
| `source/snapshot.md` | The **authoritative registry** of tracked sources — one row per guide with source URL, `ms.date`, `git_commit_id`, and `word_count`. |
| `<Vendor>/*.html` | One self-contained guide per vendor. |
| `Preview/` | Guides in development, not linked from the landing page or tracked by the healthcheck. |

---

## Architecture

The repository is a **static site** with deliberately simple operational requirements:

- **No build step.** There is no bundler, transpiler, or package manager. Each guide is
  hand-authored HTML with inlined CSS/JS/SVG. What you commit is exactly what ships.
- **No runtime dependencies.** Guides do not call external CDNs or APIs. Vendor icons are
  embedded as inline SVG or base64. This keeps each file portable (open from disk, email it,
  host it behind a firewall) and fast to load.
- **File-size budget.** Guides target a small footprint (roughly 45–80 KB each) so they
  load instantly even offline.
- **GitHub Pages hosting.** `main` is published directly; pushing updates the live site.
- **Theming.** `index.html` and the guides use a shared CSS-variable palette (`--cp-*`) and
  respect light/dark preference via a `data-theme` attribute.

The **content pipeline** is documentation-driven rather than code-driven:

```
Microsoft Learn page  ──fetch──►  agents.md authoring rules  ──►  Vendor/*.html guide
        │                                                              │
        └──────────── source/snapshot.md (metadata) ◄── healthcheck ──┘
```

`agents.md` encodes *how* a guide should look and behave; `source/snapshot.md` records the
*state* of the upstream source each guide was built from; `healthcheck.md` describes how the
two are reconciled over time.

---

## Anatomy of a Guide

Every guide follows the four-section structure defined in `agents.md`:

1. **Tenant ID & Quick Access App ID input** — two GUID-validated fields above the scenario
   grid. Clicking **Apply** stores the values and re-renders all config blocks, replacing
   `<tenantid>` and `<quickaccessappID>` placeholders.
2. **Choose Your Coexistence Scenario** — a responsive card grid (4 → 2 → 1 columns). Each
   card declares the scenario, a short description, and the *GSA owns* / *Vendor owns* split.
3. **Traffic Flow Diagram** — a collapsible, animated SVG inside a HUD canvas showing
   `End-User Device → Proxy Boxes → Destinations`, with cyan paths for GSA and emerald/green
   paths for the vendor.
4. **Implementation Guide** — accordion steps, each with numbered instructions, copy-enabled
   config blocks, tenant badges, and a verification checklist gating "Mark Complete".

The underlying data for a guide lives in a single `DATA` array of scenario objects
(`id`, `scenario`, `title`, `desc`, `gsaOwns`, `vendorOwns`, `steps[]`). See
[`agents.md`](agents.md) → *Data Structure* for the full schema.

---

## Contributing a New Vendor Guide

The full, authoritative checklist is in [`agents.md`](agents.md) → *How to Adapt for a New
Vendor*. In summary:

1. **Read `agents.md` first** — it is the spec, not just background reading.
2. **Source from Microsoft Learn.** Use the canonical
   `how-to-{vendor}-coexistence` page as the single source of truth for scenarios, IPs,
   FQDNs, and steps.
3. **Pick a vendor color** distinct from GSA cyan (`#38bdf8`) and create a vendor icon
   (inline SVG circle + letter, or base64 avatar).
4. **Build the `DATA` array, scenario tiles, HUD diagrams, and config blocks**, following the
   spatial-layout and diagram-verification rules in `agents.md`.
5. **Keep placeholders.** Leave `<tenantid>` / `<quickaccessappID>` in config code (not just
   instruction text) so badges and copy-gating work.
6. **Run the Diagram Verification Checklist and the GUID/Badge Quality Check** in `agents.md`
   before shipping.
7. **Register the guide everywhere:**
   - Add a row to **`source/snapshot.md`** (source URL + current metadata).
   - Add the guide to the **Available Guides** table in this README and the **Official
     Documentation** list.
   - Add a vendor card and reference link to **`index.html`**.
   - Add the guide to the **healthcheck registry** in [`healthcheck.md`](healthcheck.md).
8. **Version it** (start at `Version 1.0` in the footer) and **publish** (commit + push;
   GitHub Pages auto-deploys).

> ⚠️ The healthcheck registry currently tracks **6 guides**. If you add a 7th, update the
> count guardrails in `healthcheck.md` and `source/snapshot.md` accordingly.

---

## Source Healthcheck (Drift Detection)

Because each guide mirrors a live Microsoft Learn page, upstream edits make guides stale.
[`healthcheck.md`](healthcheck.md) defines the process that catches this drift early:

1. Fetch **every** source URL in `source/snapshot.md` (all 6 — `source/snapshot.md` is the
   authoritative registry, not the shorter list in `agents.md`).
2. Extract `ms.date`, `git_commit_id`, and `word_count` from each page.
3. Compare against the recorded snapshot values.
4. Report differences in a results table covering **all 6 guides**.

Any one of these signals triggers a notification:

| Signal | Meaning |
|--------|---------|
| `ms.date` differs | The page was republished. |
| `git_commit_id` differs | The source markdown changed (typo fix → major rewrite). |
| `word_count` delta > 50 | Significant content added or removed. |

When a change is approved, the corresponding guide is updated, its version is bumped,
`source/snapshot.md` is refreshed, and `index.html`'s health table is updated to reflect
freshness (🟢 current / 🟠 behind).

**Run a healthcheck manually:**

```
copilot "Run a healthcheck on the GSA coexistence guides — check all sources in source/snapshot.md for updates"
```

---

## Versioning

Each guide carries its own version in the footer (`<p class="version-line">Version X.Y</p>`):

- **Major (X):** new scenario, new section/feature, or structural redesign.
- **Minor (Y):** wording fixes, config corrections, styling tweaks.

The version **must** be incremented on every publish. See `agents.md` → *Versioning* for the
auto-increment workflow.

---

## Deployment

The site is hosted on **GitHub Pages** from the `main` branch. Publishing is simply:

```powershell
cd C:\AIProjects\jb\Inspire\SxSZ
git add <changed files>
git commit -m "..."     # bump the guide version as part of the change
git push origin main
```

After the push, verify the live page loads, e.g.
`https://jeevanbisht.github.io/GSASidebySide/<Vendor>/gsa-<vendor>-coexistence.html`.

---

## Official Documentation

- [Microsoft: GSA + Zscaler Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-zscaler-coexistence)
- [Microsoft: GSA + Cisco Umbrella Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-cisco-coexistence)
- [Microsoft: GSA + Cisco Secure Access Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-cisco-secure-access-coexistence)
- [Microsoft: GSA + Cisco VPN Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-cisco-vpn-coexistence)
- [Microsoft: GSA + Palo Alto Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-palo-alto-coexistence)
- [Microsoft: GSA + Netskope Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-netskope-coexistence)
- [Global Secure Access documentation hub](https://learn.microsoft.com/en-us/entra/global-secure-access/)
