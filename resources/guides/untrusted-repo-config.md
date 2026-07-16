---
title: "Treat a cloned repo's Claude Code setup as untrusted code"
description: "When you open someone else's project in Claude Code, its hidden config can run on your machine. Here's the durable reflex — and a free checklist — to stay in control."
last_revalidated: 2026-07-15
claude_code_ref: v2.1.210
sources:
  - https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/
  - https://flatt.tech/research/posts/poisoning-claude-code-one-github-issue-to-break-the-supply-chain/
  - https://code.claude.com/docs/en/security
  - https://code.claude.com/docs/en/settings
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
---

# Treat a cloned repo's Claude Code setup as untrusted code

## Why this is on your radar right now

In late May 2026, Claude Code made sharing setups even easier: plugins placed in a
project's `.claude/skills` directory are now **loaded automatically, with no marketplace
step** (v2.1.157). A single `git clone` is enough for a teammate — or a stranger — to
hand you their full environment.

That's genuinely convenient. It's also a good moment to internalize one reflex:

> **A repository's Claude Code configuration is not metadata. It is executable code you
> are about to trust.**

## The threat, in plain terms

When you open a project, Claude Code reads its local configuration: `.claude/settings.json`,
hooks, MCP servers, environment overrides, and now auto-loaded plugins. Anyone with commit
access to a repo can change those files — and they don't get the scrutiny that application
code gets in review, because we instinctively read them as "just settings."

This isn't hypothetical. Two documented issues in 2026 made the point:

- **CVE-2025-59536** (Check Point Research, CVSS 8.7) — a hook planted in a repo's settings
  file could execute shell commands **before the trust dialog even appeared**. Open the
  folder, and code ran.
- **CVE-2026-21852** (CVSS 5.3) — a project could override `ANTHROPIC_BASE_URL` in its
  config, causing Claude Code to send an authenticated request (carrying your API key)
  to an attacker's server **before** you confirmed you trusted the project.

Separately, GMO Flatt Security showed a supply-chain path where **a single GitHub issue**
could be enough to poison a downstream setup.

**Important, and the reason this guide is not fear-mongering:** all of these are **fixed**
in current Claude Code releases. The bugs are gone. What stays useful is the *habit* they
expose — because the underlying surface (config that can act on your behalf) is permanent.

## The durable principle

Treat the trust prompt as a real security decision, not a dialog to dismiss. Before you
let Claude Code act inside an unfamiliar repository, assume its config can run commands,
reach the network, and reconfigure your tools — and decide deliberately.

## A free, manual checklist (no tools required)

Open these in a **plain text editor** — not by launching Claude Code inside the repo —
when the source isn't someone you already trust:

1. **`.claude/settings.json`** and **`.claude/settings.local.json`** — read them fully.
2. **Hooks** — for every hook, read the actual shell command it runs. Anything that pipes
   to a network call, touches credentials, or runs an opaque script is a stop sign.
3. **Environment overrides** — watch specifically for `ANTHROPIC_BASE_URL`, `apiKeyHelper`,
   and any var that redirects where your traffic or keys go.
4. **MCP servers** (`.mcp.json` / configured servers) — what command does each one launch,
   and where does that binary come from?
5. **Auto-loaded plugins / `.claude/skills`** — since v2.1.157 these load on their own.
   Skim what's there before you work in the folder.

If anything looks off, don't open the project in Claude Code from that directory until
you've cleaned or removed it.

## The CLI is starting to back this reflex

A small sign the direction is right. From the official changelog, **v2.1.196**:

> *"Security: `claude mcp list`/`get` no longer spawn `.mcp.json` servers that a repo
> self-approved via a committed `.claude/settings.json`; untrusted workspaces show
> `⏸ Pending approval`."*

Unpack it, because it maps straight onto item 4 above. A repo could approve *its own* MCP
servers by committing `enableAllProjectMcpServers: true` (or an `enabledMcpjsonServers` list)
into `.claude/settings.json` — and then even an innocent-looking `claude mcp list` would
*spawn* those server processes. That's code execution from a command that reads like
inspection. Now, in an untrusted folder, those two commands ignore approval that arrived in a
committed file; it only counts if it lives in a settings file *you* own and haven't committed
(`.claude/settings.local.json`). Until you say so, the server stays at `⏸ Pending approval`.

The way we read it: the CLI is implementing, natively, the exact reflex this guide is about —
**a repo doesn't get to vouch for itself.** Good. But read the scope honestly: it closes one
path for two commands. It doesn't read your hooks, your `env` overrides, or your other MCP
entry points for you. The manual checklist above is still the job; the tooling just took one
item off your plate.

## Anti-patterns to drop

- Reading `.claude/` as "just configuration" instead of as code.
- Clicking **trust** reflexively to get to work.
- Running an old CLI version "because it works." (These fixes only protect you if you
  actually have them.)
- Copy-pasting a `settings.json`, agent, or hook you found online without reading it.

## Stay current — the #1 mitigation

Every issue above was resolved by an update. Keeping the CLI current is not housekeeping;
it's your primary defense. Check your version with `claude doctor` and update regularly.

While you're at it, use the levers Claude Code now gives you to *reduce* what a setup can
do — for example, `disallowed-tools` in a skill's frontmatter (v2.1.152) removes specific
tools (like shell access) from a skill while it runs. Least privilege applies to AI setups
too.

---

<sub>Last revalidated **2026-07-15** against Claude Code **v2.1.210**. This guide is
maintained alongside our [weekly watch](../../watch/) — when the ecosystem shifts, this
page is re-checked. Sources are listed in the page header.</sub>

<sub>Doing this by hand across many projects gets tedious.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> scans your projects
locally and lays out their Claude Code config in one place — settings, every hook with the
exact command it runs, MCP servers, and any hardcoded secrets it finds — so the review
above takes seconds. Use it or not; the reflex is what matters.</sub>
