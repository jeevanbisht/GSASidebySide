# GSA Coexistence Guide – Build Playbook

Use this playbook to create professional coexistence guides for GSA + any vendor (Zscaler, Netskope, Palo Alto, etc.)

---

## Architecture

Single self-contained HTML file. No external dependencies. Everything inlined (CSS, JS, SVGs, icons as base64).

---

## Page Structure (4 Sections)

### Section 0: Tenant ID Input
- Appears above scenario grid
- GUID-format input field with validation (regex: `[0-9a-f]{8}-...-[0-9a-f]{12}`)
- White background, strong border (1.5px), inset shadow to stand out from page
- "Apply" button disabled until valid GUID entered
- On apply: replaces all `<tenantid>` placeholders in config blocks with actual value
- Shows green confirmation text: `✓ Tenant ID applied: {guid}`

### Section 1: Choose Your Coexistence Scenario
- 4-column card grid (`grid-template-columns: repeat(4, 1fr)`) with 16px gap
- Each tile has: **Scenario label**, **Title**, **Description**, **GSA Owns** list, **Vendor Owns** list
- Tile style: `border-radius: 8px`, no card-shadow, no transform on hover, no gradient on selected
- Active tile gets `border-color: var(--cp-accent)` + `box-shadow: 0 0 0 1px var(--cp-accent)`
- Responsive: 2 columns at ≤960px, 1 column at ≤600px
- Clicking a tile loads Sections 2 & 3 for that scenario

### Section 2: Traffic Flow Diagram (Collapsible)
- Inline SVG diagram per scenario wrapped in a **HUD canvas** (dark futuristic container)
- Collapsible via chevron toggle
- Shows: End-User Device → Proxy Boxes → Destination Boxes
- Color-coded paths: Cyan (#38bdf8) = GSA, Emerald (#10b981) = Vendor
- Dark sci-fi aesthetic with floating particles, corner brackets, glow effects

### Section 3: Implementation Guide
- Accordion-style steps (click to expand)
- Each step has: title, numbered sub-instructions, code/config blocks with copy button
- Steps with tenant-specific config show **"Tenanted Endpoints: X"** badge in the header row
- Verification checklist (checkboxes) at the end of each step
- "Mark Complete" button (disabled until all checkboxes checked)
- Progress tracked per scenario via `doneMap`

---

## Tenant ID Feature

### How it works
1. User enters their Microsoft Entra tenant ID (GUID) in the top input field
2. Clicking "Apply" stores the value and re-renders active scenario steps
3. All `<tenantid>` placeholders in config code blocks get replaced with the actual GUID
4. Steps containing tenant-specific FQDNs show a count badge in the step header

### Tenanted FQDNs pattern
```
<tenantid>.internet.client.globalsecureaccess.microsoft.com
<tenantid>.m365.client.globalsecureaccess.microsoft.com
<tenantid>.private.client.globalsecureaccess.microsoft.com
<tenantid>.auth.client.globalsecureaccess.microsoft.com
<tenantid>.private-backup.client.globalsecureaccess.microsoft.com
<tenantid>.internet-backup.client.globalsecureaccess.microsoft.com
<tenantid>.m365-backup.client.globalsecureaccess.microsoft.com
<tenantid>.auth-backup.client.globalsecureaccess.microsoft.com
```

### Step header badge logic
```javascript
// In renderSteps(), for each step with a cfg block:
const tCount = st.cfg.code.split('\n')
  .filter(l => l.includes('<tenantid>') && l.includes('.client.globalsecureaccess')).length;
if (tCount > 0) tenantBadge = `<span class="step-tenant-badge">Tenanted Endpoints: ${tCount}</span>`;
```

### Copy button gating
Config blocks with `<tenantid>` placeholders **block the Copy button** until a valid tenant ID is applied:

- Button shows "🔒 Copy" when tenant ID is missing
- Clicking it shows red warning text: *"⚠ Enter Tenant ID first"* (3s timeout)
- A yellow notice bar appears above the code: *"⚠ Enter your Tenant ID above to populate tenant-specific endpoints before copying."*
- Once tenant ID is applied, lock/notice disappear and Copy works normally

```javascript
function doCopy(id, e) {
  e.stopPropagation();
  const txt = document.getElementById(id).textContent;
  if (txt.includes('<tenantid>') && !tenantId) {
    // Show warning, block copy
    const btn = document.getElementById(`cb-${id}`);
    btn.textContent = '⚠ Enter Tenant ID first';
    btn.classList.add('warn');
    setTimeout(() => { btn.textContent = 'Copy'; btn.classList.remove('warn'); }, 3000);
    return;
  }
  navigator.clipboard.writeText(txt).then(() => { /* success feedback */ });
}
```

```javascript
// In configHTML(), show notice when tenantId not yet applied:
const hasTenanted = code.includes('<tenantid>');
const notice = (hasTenanted && !tenantId)
  ? `<div class="config-tenant-notice">⚠ Enter your Tenant ID above...</div>` : '';
const btnLabel = (hasTenanted && !tenantId) ? '🔒 Copy' : 'Copy';
```

### CSS for tenant input & copy gating
```css
.tenant-field {
  border: 1.5px solid var(--border-strong);
  background: #ffffff;
  box-shadow: inset 0 1px 3px rgba(0,0,0,0.08);
}
.tenant-field:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.12);
}
.step-tenant-badge {
  font-size: 0.625rem;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 3px;
  background: rgba(37, 99, 235, 0.08);
  color: #2563eb;
  margin-left: auto;
}
.btn-copy.warn { color: #dc2626; font-size: 0.6rem; }
.config-tenant-notice {
  padding: 6px 12px;
  font-size: 11px;
  color: #b45309;
  background: rgba(245, 158, 11, 0.08);
  border-bottom: 1px solid rgba(245, 158, 11, 0.2);
}
```

### Validation
```javascript
const GUID_RE = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
```

---

## HUD Diagram System

Traffic flow diagrams use a futuristic "HUD canvas" design — a dark sci-fi container with animated particles, glowing neon paths, and corner bracket decorations.

### HUD Canvas Structure

Each diagram is wrapped in a `hudWrap()` function that generates:

```html
<div class="hud-canvas">
  <!-- Floating particle field (18 particles) -->
  <div class="hud-particles">...</div>
  <!-- Corner bracket decorations -->
  <div class="hud-corner hud-corner--tl"></div>
  <div class="hud-corner hud-corner--tr"></div>
  <div class="hud-corner hud-corner--bl"></div>
  <div class="hud-corner hud-corner--br"></div>
  <!-- SVG diagram -->
  <svg class="hud-svg" viewBox="0 0 930 340">...</svg>
</div>
```

### HUD Canvas CSS

```css
.hud-canvas {
  position: relative;
  background: linear-gradient(135deg, #0a0e1a 0%, #111827 40%, #0c1222 100%);
  border-radius: 12px;
  border: 1px solid rgba(56, 189, 248, 0.15);
  padding: 32px 24px;
  overflow: hidden;
  width: 100%;
  box-shadow: 0 0 40px rgba(56, 189, 248, 0.05), inset 0 0 60px rgba(0,0,0,0.3);
}
```

Key rules:
- `width: 100%` — canvas fills the container (no max-width constraint)
- `.hud-svg` also `width: 100%` with no max-width — scales responsively
- `.diagram-container` uses `display: block` (not flex) to avoid centering constraints

### Particle Field

```javascript
function particles(count) {
  let p = '';
  for (let i = 0; i < count; i++) {
    const left = Math.random() * 100;
    const delay = Math.random() * 8;
    const size = 1.5 + Math.random() * 1.5;
    const opacity = 0.3 + Math.random() * 0.4;
    p += `<div class="hud-particle" style="left:${left}%;animation-delay:${delay}s;width:${size}px;height:${size}px;opacity:${opacity}"></div>`;
  }
  return p;
}
```

- 18 particles per diagram
- Each particle: 1.5-3px, cyan glow, floats upward over 8s with random delay
- Uses `@keyframes particleFloat` animation

### Corner Brackets

```css
.hud-corner { position: absolute; width: 16px; height: 16px; pointer-events: none; }
.hud-corner--tl { top: 12px; left: 12px; border-top: 2px solid rgba(56,189,248,0.4); border-left: 2px solid rgba(56,189,248,0.4); }
/* ... similar for --tr, --bl, --br */
```

### Box Dimensions

| Element | Width | Height | rx |
|---------|-------|--------|----|
| Client device | 155px | 70px | 6 |
| Proxy box (GSA/Vendor, 4-line) | 240px | 90-92px | 6 |
| Proxy box (GSA/Vendor, 3-line) | 240px | 82px | 6 |
| Destination box | 210-220px | 60-68px | 6 |

### Spatial Layout Rules

Diagrams use viewBox `0 0 930 340` (or `900 300` for Zscaler's simpler 2-3 destination layouts). Content must fill the viewBox width — **no wasted space on the right**.

```
Column layout across the viewBox (930px wide):

  Left box (client):    x=30,  width=155  → ends at 185
  Middle boxes (proxy): x=280, width=240  → ends at 520
  Right boxes (dest):   x=650, width=220  → ends at 870

  Gaps: 95px (left→mid), 130px (mid→right), 60px right margin

For 900px viewBox (Zscaler scenarios 1-3):

  Left box (client):    x=30,  width=145  → ends at 175
  Middle boxes (proxy): x=280, width=210  → ends at 490
  Right boxes (dest):   x=640, width=210  → ends at 850

  Gaps: 105px (left→mid), 150px (mid→right), 50px right margin
```

Key rules:
- **Right boxes at x=640-650** with width 210-220px (ending near 850-870) to avoid empty right space
- **Middle boxes at x=280** with width 210-240px — room for text breathing
- **Arrow paths**: left arrows from ~175-185 → 275-280; right arrows from ~494-524 → 635-645
- **Labels positioned in the gap** between boxes, offset 8-12px above arrow paths
- **Left gap labels** at approximately x=200-206
- **Right gap labels** at approximately x=548-572

### Box Styling (HUD variant)

```svg
<!-- GSA box: cyan border, dark fill, subtle glow -->
<g class="hud-box-gsa">
  <rect x="280" y="36" width="240" height="80" rx="6" fill="rgba(15,23,42,0.9)" stroke="#38bdf8" stroke-width="1.5"/>
  <rect x="280" y="36" width="240" height="80" rx="6" fill="rgba(56,189,248,0.05)"/>
  <!-- icon + text -->
</g>

<!-- Vendor box: emerald border -->
<g class="hud-box-vendor">
  <rect ... stroke="#10b981" stroke-width="1.5"/>
  <rect ... fill="rgba(16,185,129,0.05)"/>
</g>

<!-- Neutral/destination box: slate border -->
<g class="hud-box-neutral">
  <rect x="650" y="24" width="220" height="64" rx="6" fill="rgba(15,23,42,0.7)" stroke="#475569" stroke-width="1"/>
</g>
```

CSS for box glow:
```css
.hud-box-gsa { filter: drop-shadow(0 0 6px rgba(56,189,248,0.3)); }
.hud-box-vendor { filter: drop-shadow(0 0 6px rgba(16,185,129,0.3)); }
.hud-box-neutral { filter: drop-shadow(0 0 4px rgba(148,163,184,0.15)); }
```

### Icon + Text Alignment

```
Box layout: [ 36px icon zone | remaining text zone ]

- Icon (22×22px): placed at box_x + 8, vertically centered with title
- Text: centered (text-anchor="middle") at box_x + 36 + (box_width - 36) / 2
- Title: .hud-node-title (12px, font-weight 600, fill: #e2e8f0)
- Subtitle: .hud-node-sub (10px, fill: #94a3b8)
- Tertiary: .hud-node-sub2 (9.5px, fill: #64748b)
- Example: 240px box at x=280 → icon at x=288, text center at x=418
```

### Icon Embedding

- **GSA**: Inline SVG globe with 3 linearGradients (neon cyan variant)
  - Gradient IDs use `gsa-h1-{ID}`, `gsa-h2-{ID}`, `gsa-h3-{ID}` pattern
  - Icon size: 22×22px
- **Vendor**: Inline SVG circle with letter abbreviation
  - Zscaler: emerald circle (#10b981) with white "Z"
  - Cisco Umbrella: green circle (#22c55e) with white "C"
  - Palo Alto: emerald circle (#10b981) with white "PA"
  - Icon size: 22×22px

### Arrow Paths

**CRITICAL: Do NOT use SVG `filter="url(#...)"` on path elements.** This causes clipping on long/diagonal arrows. Use CSS `filter: drop-shadow()` on the class instead.

```css
.hud-gsa-flow {
  stroke-dasharray: 8 4;
  animation: hudFlowBlue 0.8s linear infinite;
  stroke-linecap: round;
  filter: drop-shadow(0 0 3px rgba(56,189,248,0.6));
}
.hud-vendor-flow {
  stroke-dasharray: 6 3;
  animation: hudFlowGreen 0.7s linear infinite;
  stroke-linecap: round;
  filter: drop-shadow(0 0 3px rgba(16,185,129,0.6));
}
```

Arrow markers in `<defs>`:
```svg
<marker id="ha{N}b" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto">
  <path d="M0,0 L8,3 L0,6 Z" fill="#38bdf8"/>
</marker>
<marker id="ha{N}g" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto">
  <path d="M0,0 L8,3 L0,6 Z" fill="#10b981"/>
</marker>
```

### Arrow Labels

Labels must be **professionally visible** against the dark background and crossing arrows:

```css
.hud-arrow-label {
  font-size: 11px;
  font-weight: 700;
  paint-order: stroke;
  stroke: rgba(10,14,26,0.92);    /* dark halo behind text */
  stroke-width: 4px;
  stroke-linecap: round;
  stroke-linejoin: round;
}
```

Key rules:
- Use `paint-order: stroke` to render stroke BEHIND fill (creates readable halo)
- Position labels **8-12px above** the nearest arrow path (not on top of it)
- Fill color matches the arrow color (#38bdf8 for GSA, #10b981 for vendor)
- Labels in the left gap (between client and proxy) go at approximately x=206
- Labels in the right gap (between proxy and destination) go at approximately x=568

### Animation Keyframes

```css
@keyframes hudFlowBlue {
  to { stroke-dashoffset: -18; }
}
@keyframes hudFlowGreen {
  to { stroke-dashoffset: -16; }
}
@keyframes particleFloat {
  0% { transform: translateY(100%) translateX(0); opacity: 0; }
  10% { opacity: 1; }
  90% { opacity: 1; }
  100% { transform: translateY(-100%) translateX(20px); opacity: 0; }
}
```

### HUD Color Palette

| Purpose | Value |
|---------|-------|
| GSA elements (arrows, borders, text) | #38bdf8 (cyan) |
| Vendor elements — Zscaler | #10b981 (emerald) |
| Vendor elements — Cisco Umbrella | #22c55e (green) |
| Vendor elements — Palo Alto | #10b981 (emerald) |
| Neutral/destination borders | #475569 (slate) |
| Node title text | #e2e8f0 |
| Node subtitle text | #94a3b8 |
| Node tertiary text | #64748b |
| Canvas background | #0a0e1a → #111827 gradient |
| Box fill (GSA) | rgba(56,189,248,0.05) |
| Box fill (Vendor — emerald) | rgba(16,185,129,0.05) |
| Box fill (Vendor — Cisco green) | rgba(34,197,94,0.05) |
| Box fill (Neutral) | rgba(15,23,42,0.7) |
| Particle color | rgba(56,189,248,0.6) |
| Corner bracket color | rgba(56,189,248,0.4) |

---

## Diagram Verification Checklist

After building or modifying any diagram, verify ALL scenarios against this checklist:

### Visual Verification (open in browser, check each scenario)

- [ ] **Scaling**: HUD canvas fills full width of the collapsible section (no centering gap)
- [ ] **No dead space**: Content extends to ~870px in a 930px viewBox (no large empty right margin)
- [ ] **Left arrows visible**: All paths from Client → Proxy boxes render with glow and arrowhead
- [ ] **Right arrows visible**: All paths from Proxy boxes → Destination boxes render fully (no clipping)
- [ ] **Arrow heads intact**: Marker triangles appear at the end of every path (not cut off)
- [ ] **Labels readable**: All arrow labels clearly visible (11px, bold, dark halo), not overlapping arrows
- [ ] **Labels offset**: Labels sit 8-12px above arrow paths, positioned in the gap between box columns
- [ ] **Particles animating**: 18 floating particles visible drifting upward in the canvas
- [ ] **Corner brackets**: All 4 cyan corners visible at edges of the canvas
- [ ] **Box glow**: GSA boxes have cyan glow, vendor boxes have emerald glow
- [ ] **Icons render**: GSA globe and vendor icon visible inside their respective boxes
- [ ] **Text legible**: All node titles, subtitles, and tertiary text readable against dark fill

### Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| Arrows clipped/half-cut | SVG `filter="url(#...)"` on `<path>` | Remove inline filter; use CSS `filter: drop-shadow()` on class |
| Diagram doesn't scale | `max-width` on `.hud-svg` or flex centering on container | Set `width: 100%` on both, use `display: block` on container |
| Labels invisible/hard to read | Small font, thin stroke, low opacity halo | Use 11px, weight 700, `paint-order: stroke` with 4px dark stroke (0.92 opacity) |
| Empty space on right side | Destination boxes too far left or too narrow | Place right boxes at x=650, width=220 (ends at 870 in 930 viewBox) |
| Labels overlapping arrows | Labels placed at same y-coordinate as arrow path | Offset labels 8-12px above (lower y value) the arrow midpoint |
| Arrow markers missing | Marker ID collision across diagrams | Use unique IDs per diagram: `ha1b`, `ha2b`, etc. |
| Gradient ID collision | Same gradient ID in multiple diagrams | Use `{ID}` template replacement per diagram instance |
| Particles not showing | Missing `overflow: hidden` on hud-canvas | Ensure `.hud-particles` is absolutely positioned inside |

---

## Data Structure (JavaScript)

```javascript
const DATA = [
  {
    id: 1,
    scenario: "Scenario 1",
    title: "Short title",
    desc: "One-line description",
    gsaOwns: ["List", "of", "GSA responsibilities"],
    vendorOwns: ["List", "of", "vendor responsibilities"],
    steps: [
      {
        t: "Step title",
        b: "gsa" | "zscaler" | "verify",  // badge/owner
        inst: [
          "Step 1 instruction text",
          "Step 2 instruction text"
        ],
        cfg: {  // optional - config block with copy button
          label: "Description of config",
          code: "line1\nline2\n<tenantid>.example.com"
        },
        chk: ["Verification item 1", "Verification item 2"]  // optional checklist
      }
    ]
  }
];
```

---

## Checklist / Progress Logic

```javascript
// Track completed steps per scenario
const doneMap = {};  // { scenarioId: Set([stepIndex, ...]) }

// "Mark Complete" button disabled until ALL checkboxes checked
function updateDoneBtn(stepIdx) {
  const cl = document.getElementById(`cl-${stepIdx}`);
  if (!cl) return;
  const boxes = cl.querySelectorAll('input[type="checkbox"]');
  const allChecked = [...boxes].every(cb => cb.checked);
  const btn = document.getElementById(`done-${stepIdx}`);
  if (btn) btn.disabled = !allChecked;
}
```

---

## How to Adapt for a New Vendor

1. **Choose vendor color**: Use a distinct color that doesn't clash with GSA cyan (#38bdf8)
   - Emerald #10b981 (Zscaler, Palo Alto), Green #22c55e (Cisco) are established
   - Avoid blue tones too close to GSA cyan
2. **Create vendor icon**: SVG circle with brand color + white letter abbreviation
   - Example: `<circle cx="10" cy="10" r="10" fill="#10b981"/><text x="10" y="14.5" text-anchor="middle" font-size="12" font-weight="700" fill="white">Z</text>`
3. **Update DATA array** with scenarios from official Microsoft docs:
   - URL pattern: `https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-{vendor}-coexistence`
4. **Build diagrams using Spatial Layout Rules**: follow the column positions (x=30/280/650) and viewBox sizes documented above
5. **Update scenario tiles**: 4-column grid, 8px radius, no transform — copy CSS from Zscaler reference
6. **Update tenant FQDNs**: Keep `<tenantid>` placeholder in any config code that has tenant-specific endpoints
7. **Update step badge labels**: Replace `b: "zscaler"` with appropriate vendor key
8. **Update CSS variables**: Set `--cp-vendor` and `--cp-vendor-soft` rgba values for the chosen color
9. **Run Diagram Verification Checklist** on all scenarios before shipping

---

## Vendor Icon Retrieval

```powershell
# Get vendor GitHub avatar as base64
$bytes = (Invoke-WebRequest -Uri "https://avatars.githubusercontent.com/u/{ORG_ID}?s=200&v=4").Content
$b64 = [Convert]::ToBase64String($bytes)
# Use in SVG: <image x="{X}" y="{Y}" width="20" height="20" href="data:image/png;base64,$b64" />
```

GitHub org IDs:
- Zscaler: 1709408
- Netskope: 14841216
- Palo Alto Networks: 3422444

---

## File Size Budget

Target: < 50KB for fast load. Current Zscaler guide: ~45KB (with base64 icon).

---

## Versioning

### Format: `X.Y`
- **X** (major): New scenarios added, structural redesign, breaking changes
- **Y** (minor): Bug fixes, wording updates, styling tweaks, config corrections

### Location in HTML
Version is displayed in the footer:
```html
<p class="version-line">Version 1.0</p>
```

### Auto-increment on publish
When pushing to the repo, the version **must** be incremented. Use this rule:

| Change type | Example | Version bump |
|-------------|---------|--------------|
| New scenario added | Scenario 5 | X+1.0 (reset minor) |
| New section/feature | Added "Export PDF" button | X+1.0 |
| Step instructions updated | Fixed typo in Step 3 | X.Y+1 |
| Config values corrected | Updated bypass IPs | X.Y+1 |
| Styling/cosmetic fix | Adjusted padding | X.Y+1 |

### Publish workflow
Before every `git push`, run:
```powershell
# Auto-increment minor version in the HTML file
$file = "Zscaler/gsa-zscaler-coexistence.html"
$content = Get-Content $file -Raw
if ($content -match 'Version (\d+)\.(\d+)') {
  $major = [int]$Matches[1]
  $minor = [int]$Matches[2] + 1
  $newVer = "Version $major.$minor"
  $content = $content -replace 'Version \d+\.\d+', $newVer
  Set-Content $file $content -NoNewline -Encoding UTF8
  Write-Host "Bumped to $newVer"
}
```

For major bumps, manually set the version or pass a flag:
```powershell
# Major version bump (resets minor to 0)
$content = $content -replace 'Version \d+\.\d+', 'Version 2.0'
```

---

## Quick Start Command

```
copilot "Create a GSA + {Vendor} coexistence guide following the pattern in agents.md. 
Source: https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-{vendor}-coexistence
Vendor icon: https://avatars.githubusercontent.com/u/{ORG_ID}?s=200&v=4"
```
