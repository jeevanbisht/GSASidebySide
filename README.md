# GSA Side-by-Side Coexistence Guides

Interactive configuration guides for deploying Microsoft Global Secure Access alongside third-party security vendors.

## Available Guides

| Vendor | Guide | Live Page |
|--------|-------|-----------|
| Zscaler | [Source](Zscaler/gsa-zscaler-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/Zscaler/gsa-zscaler-coexistence.html) |
| Cisco Umbrella | [Source](CiscoUmbrella/gsa-cisco-coexistence.html) | [🔗 Open Guide](https://jeevanbisht.github.io/GSASidebySide/CiscoUmbrella/gsa-cisco-coexistence.html) |

## Features

- **Scenario Selection** — Choose from multiple coexistence deployment patterns
- **Traffic Flow Diagrams** — Animated SVG diagrams showing how traffic routes through GSA and vendor proxies
- **Step-by-Step Implementation** — Expandable instructions with copy-to-clipboard config blocks
- **Tenant ID Integration** — Enter your tenant GUID to auto-populate tenant-specific bypass FQDNs (where applicable)
- **Progress Tracking** — Checkboxes and step completion tracking per scenario

## Build Playbook

See [agents.md](agents.md) for the full playbook to create guides for additional vendors.

## Official Documentation

- [Microsoft: GSA + Zscaler Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-zscaler-coexistence)
- [Microsoft: GSA + Cisco Coexistence](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-cisco-coexistence)
