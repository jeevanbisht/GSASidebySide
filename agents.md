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
- Horizontal card grid (scenario tiles)
- Each tile has: **Scenario label**, **Title**, **Description**, **GSA Owns** list, **Vendor Owns** list
- Active tile gets a colored top border + subtle background tint
- Clicking a tile loads Sections 2 & 3 for that scenario

### Section 2: Traffic Flow Diagram (Collapsible)
- Inline SVG diagram per scenario
- Collapsible via chevron toggle
- Shows: End-User Device → Proxy Boxes → Destination Boxes
- Color-coded paths: Blue (#2563eb) = GSA, Green (#059669) = Vendor

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

## SVG Diagram Layout Rules

### Box Dimensions
| Element | Width | Height | rx |
|---------|-------|--------|----|
| Client device | 130px | 64px | 6 |
| Proxy box (2-line) | 180px | 64px | 6 |
| Proxy box (3-line) | 180px | 72px | 6 |
| Proxy box (4-line) | 200px | 80px | 6 |
| Destination box | 160px | 64-72px | 6 |

### Icon + Text Alignment (Fixed Icon Zone)
```
Box layout: [ 36px icon zone | remaining text zone ]

- Icon (20×20px): placed at box_x + 8, vertically centered with title
- Text: centered (text-anchor="middle") at box_x + 36 + (box_width - 36) / 2
- Title: .node-title (11px, font-weight 600)
- Subtitle: .node-sub (9px, regular)
- Example: 180px box at x=310 → icon at x=318, text center at x=418
```

### Icon Embedding
- **GSA**: Inline SVG (the official Global Secure Access globe from az-icons.com) with 3 linearGradients
  - IMPORTANT: Gradient IDs must be unique per diagram instance (use template `{ID}` replacement)
- **Vendor**: Base64 PNG `<image>` element (fetch from GitHub avatar or official source)
  - Example: `<image x="{X}" y="{Y}" width="20" height="20" href="data:image/png;base64,..." />`
  - Both icons must be same size (20×20px) for consistent alignment

### Arrows
- GSA paths: `stroke="#2563eb"`, animated with `stroke-dasharray: 6 3` + CSS keyframe
- Vendor paths: `stroke="#059669"`, solid, `stroke-width="1.8"`
- Arrow markers defined in `<defs>` with unique IDs per diagram (e.g., `a1b`, `a2b`)
- Arrow labels: `.arrow-label` class (8.5px, weight 500), positioned near arrow midpoint

### Animation CSS
```css
@keyframes flowBlue {
  to { stroke-dashoffset: -9; }
}
.gsa-flow {
  stroke-dasharray: 6 3;
  animation: flowBlue 0.6s linear infinite;
}
```

---

## Styling Conventions

### Colors
| Purpose | Value |
|---------|-------|
| GSA elements | #2563eb (blue) |
| Vendor elements | #059669 (green) |
| Neutral/destination borders | #d1d5db |
| Title text (neutral) | #374151 |
| Subtitle text | #6b7280 |
| Tertiary text | #9ca3af |
| GSA box fill | rgba(37,99,235,0.04) |
| Vendor box fill | rgba(5,150,105,0.04) |

### Typography
- Font: `"Segoe UI", sans-serif`
- Node titles: 11px, weight 600
- Node subtitles: 9px, regular
- Arrow labels: 8.5px, weight 500

### Theme Support
- Light/dark toggle via `data-theme` attribute on `<html>`
- CSS custom properties for all colors
- SVG diagrams use explicit fills (not theme vars) since they're inline

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

1. **Replace vendor color** if needed (green works for most; use vendor brand color)
2. **Replace vendor icon**: Get PNG from `https://avatars.githubusercontent.com/u/{ORG_ID}?s=200&v=4` and convert to base64
3. **Update DATA array** with scenarios from official Microsoft docs:
   - URL pattern: `https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-{vendor}-coexistence`
4. **Update diagrams**: Adjust number of proxy/destination boxes per scenario
5. **Update scenario tiles**: Change "Zscaler Owns" → "{Vendor} Owns"
6. **Update tenant FQDNs**: Keep `<tenantid>` placeholder in any config code that has tenant-specific endpoints
7. **Update step badge labels**: Replace `b: "zscaler"` with appropriate vendor key

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
