<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/media/logo-light.svg">
    <img src="docs/media/logo-dark.svg" alt="Tarmhack'AI" width="300">
  </picture>
</p>

<h3 align="center">Take back control of your Claude Code environment.</h3>

<p align="center">
  <img src="https://img.shields.io/badge/status-pilot-F97316">
  <img src="https://img.shields.io/badge/platforms-Windows%20|%20Linux-2b3137">
  <img src="https://img.shields.io/badge/100%25-local-2ea44f">
  <img src="https://img.shields.io/badge/a-Sw%C4%93l%20product-7B68EE">
</p>

<p align="center">
  <a href="https://agence-swel.fr/solutions/tarmhack/"><strong>Website</strong></a> ·
  <a href="https://agence-swel.fr/solutions/tarmhack/#wishlist"><strong>Join the wishlist</strong></a>
</p>

---

**Tarmhack'AI** is a 100% local desktop app that **scans, audits and synchronizes**
your Claude Code setup — projects, hooks, skills, agents, MCP servers, `CLAUDE.md`
files and permissions. No cloud, no API key, no telemetry.

> Other tools edit files. Tarmhack'AI analyzes, protects, optimizes and syncs.

<p align="center">
  <img src="docs/media/app-dashboard.webp" alt="Tarmhack'AI dashboard" width="860">
</p>

<sub><em>"Claude Code" is a trademark of Anthropic. Tarmhack'AI is an independent product of
Swēl SAS, not affiliated with, sponsored by, or endorsed by Anthropic.</em></sub>

## The problem

The status quo holds for one project. It breaks at three. Symlinks, home-made scripts
and visual audits stop scaling the moment your Claude Code projects multiply: no overview,
copy-paste inheritance, silent drift, and security in a blind spot.

## What it does

**See everything** — every Claude Code project in one dashboard: scores, alerts, assets.
Open the terminal, the project folder or any asset in one click.

**Analyze & secure** — a Health Score out of 100 per `CLAUDE.md`, tracked over time. A
hub → projects drift detector. A Secrets Radar and a permissions audit, continuously.

**Edit & propagate** — embedded Monaco editor, drag-and-drop assets between projects, and
an AI Lab that rewrites an asset for its target project — driven by your local Claude Code
CLI. No API call from the app.

<details>
<summary><strong>The nine functions</strong></summary>

- **Total visibility** — projects, hooks, skills, agents, MCP servers and plugins in one place
- **AI Lab** — describe a hook, agent or skill; the Lab generates it via your local CLI
- **Hub → projects inheritance** — shared configs as a clear tree, with a drift detector
- **CLAUDE.md health score** — a /100 score per file, tracked over time
- **Ultra Stack** — boost a project with a tailored selection of assets
- **Security Radar** — continuous secret scanning + allow/deny permissions map
- **Dead-config detector** — missing hook scripts, unreachable MCP servers, orphan permissions
- **Built-in editor** — Monaco embedded; terminal and folder one click away
- **Daily digest** — what changed since yesterday and each project's next step
</details>

## 100% local, always

No account, no cloud, no telemetry — not even opt-in. Tarmhack'AI makes no API calls:
your local Claude does the work. Wi-Fi off, the app runs the same. → see [`SECURITY.md`](SECURITY.md)

## Also in this repo

- 📰 **[`watch/`](watch/)** — a weekly, plain-language digest of what changes in the
  Claude Code / Anthropic ecosystem, and why it matters.
- 🧰 **[`resources/`](resources/)** — curated, opinionated resources for running Claude Code
  safely and well — kept current with the ecosystem, not left to rot.

## Pricing

14-day trial, no credit card. €12/month or €99/year. **Founder price: €69/year for the
first 500** on the wishlist — locked as long as your subscription stays active. No API key,
no extra AI subscription — it uses your existing Claude Code CLI.

<p align="center">
  <a href="https://agence-swel.fr/solutions/tarmhack/#wishlist"><strong>→ Join the wishlist</strong></a>
</p>

## Status

Pilot phase — **Windows and Linux** (`.exe`, AppImage, `.deb`). macOS planned.
Access via the wishlist.

## License

Tarmhack'AI (the application) is **proprietary software**, and this repository contains
**no product source code**. The reusable configuration templates under
[`resources/templates/`](resources/templates/) are released under the **MIT License**.
All other content (documentation, watch editions, branding) is © 2026 Swēl SAS —
all rights reserved.

---

<sub>"Claude Code" and "Claude" are trademarks of Anthropic. Tarmhack'AI is an independent
product of Swēl SAS, not affiliated with, sponsored by, or endorsed by Anthropic; trademark
references are nominative and for identification only. No Anthropic API is called by the app.</sub>

<sub>Published by <strong>Swēl SAS</strong> (SIREN 104 600 374, Villetelle, France) —
<a href="https://agence-swel.fr/mentions-legales">Legal notice</a> ·
<a href="https://agence-swel.fr/confidentialite">Privacy policy</a>.</sub>
