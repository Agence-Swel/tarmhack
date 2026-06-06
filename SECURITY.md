# Security & Trust

Tarmhack'AI reads and edits sensitive files in your AI setup — `CLAUDE.md`,
`.claude/` settings, hooks, MCP servers. So you deserve to know exactly how it
treats your machine and your data. This page is deliberately specific, and honest
about the few things that do touch the network.

## Local-first by design

Tarmhack'AI runs entirely on your machine. Your projects, configurations, scans,
health scores and analyses **never leave your computer**. There is no Tarmhack'AI
cloud, no account to create, no server that mirrors your data.

## No AI API key, no AI calls from the app

Tarmhack'AI **never calls an AI API itself**. Its AI features (the Lab, the daily
digest, optimizations) drive the **Claude Code CLI already installed on your machine**,
as a local subprocess over stdin/stdout. That CLI uses **your own Claude
authentication** — Tarmhack'AI doesn't read, store or transmit it. No extra API key,
no extra AI subscription.

> In other words: the only thing that talks to Anthropic is your local Claude Code CLI,
> with your own credentials — exactly as it does without us.

## No telemetry

No analytics, no tracking, no usage beacons — **not even opt-in**. We ship no
telemetry SDK of any kind. The usage statistics shown in the app are read **locally**
from the transcript files Claude Code already writes on your disk; nothing is emitted.

## What actually connects to the internet

Being honest beats a slogan: Tarmhack'AI is **not fully offline**. It makes exactly
**three** outbound connections — and **none of them carry your code, your configs,
your database or your usage**:

| Destination | When | What is sent |
|---|---|---|
| **npm registry** | Periodic check "is your Claude CLI up to date?" | Anonymous request. No personal data. |
| **Keygen** (licensing) | License activation / refresh | Your license key + a machine fingerprint (a derived ID). No project data. |
| **GitHub Releases** | Update check at startup; download only if you click install | Nothing sent (versions compared locally). |

Clicking an external link opens your **system browser** (https-only). The startup
update check can be turned off in the app's Preferences.

## Filesystem safety

Tarmhack'AI only writes to **recognized AI-config locations** — `.claude/`,
`CLAUDE.md`, and standard asset folders (`skills/`, `hooks/`, `agents/`, `docs/`, …) —
and only with a **whitelist of text file types**. Path traversal (`..`), symlink
escapes and look-alike folder names are rejected. It never writes outside those
patterns, and deletion rules are even stricter.

## Secrets stay local

The **Secrets Radar** scans your local config files for hardcoded API keys and tokens
and flags them **in the app**. Detected secrets are **never sent anywhere** — the whole
point is to catch them before they leak.

## Your data stays on your machine

Scans, scores and history are stored in a local SQLite database in your app-data
folder. Nothing is synced to any server.

## Signed updates

Application updates are **cryptographically signed** (minisign). The app **verifies the
signature** before installing — so an update can't be tampered with in transit.

## Transparency & current limits

We'd rather tell you than have you find out:

- **Beta builds are not yet OS-code-signed.** Windows Authenticode and macOS
  notarization are planned for general availability. On Windows you may see a
  **SmartScreen warning** on first launch — see the installation guide. (Signed updates
  protect the *update channel*, not the operating system's first-launch trust.)
- **The local database is not encrypted at rest.** It lives on your own machine, in
  your user profile.
- **A machine fingerprint** (a derived identifier, not your content) is sent to our
  licensing provider during activation.

## Reporting a vulnerability

Found a security issue? Please report it **privately** and give us reasonable time to
fix it before any public disclosure. Contact us via
**[agence-swel.fr/contact](https://agence-swel.fr/contact)**.

We read every report and take them seriously.

---

<sub>Tarmhack'AI is a product by <a href="https://agence-swel.fr">Swēl</a>. Claude Code is
a trademark of Anthropic; Tarmhack'AI is an independent third-party tool, not affiliated
with Anthropic.</sub>
