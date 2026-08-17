---
title: "Treat a cloned repo's Claude Code setup as untrusted code"
description: "When you open someone else's project in Claude Code, its hidden config can run on your machine. Here's the durable reflex — and a free checklist — to stay in control."
last_revalidated: 2026-08-17
claude_code_ref: v2.1.233
sources:
  - https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/
  - https://flatt.tech/research/posts/poisoning-claude-code-one-github-issue-to-break-the-supply-chain/
  - https://code.claude.com/docs/en/security
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/skills
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/changelog
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

And the surface hasn't stopped moving. In August 2026, Anthropic had to fix nested git
repositories inheriting trust from a parent directory (v2.1.232) — which says something
useful about how far a single **trust** click used to reach. More on that below.

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

**Then know what that decision covers.** A trust decision has a *scope*, and the scope is
not "everything, forever, everywhere below this folder." Two axes, both straight from the
official permissions reference:

- **Where it applies.** Trust is saved per workspace, *"keyed on the git repository root
  or, outside a repository, the directory you started Claude Code from."* That matters the
  moment a repo contains other repos — submodules, vendored dependencies, a monorepo's
  sub-repos. Until **v2.1.232**, those nested repositories inherited the parent's trust.
  Now each one asks for itself. On a current CLI, expect *more* prompts, not fewer.
- **What it covers inside that folder.** Not everything is gated by it. The docs table
  *What runs before you trust a folder* is explicit: when you've trusted only a parent
  folder, hooks in settings files, the `env` block, helper commands such as `apiKeyHelper`,
  and a project skill's hooks and `allowed-tools` are **used** — while `permissions.allow`
  rules and `additionalDirectories` are held back until you accept. On skills the docs go
  further: *"A skill can grant itself broad tool access, so review the `allowed-tools` of
  skills checked into a repository before you run Claude Code there."*

And a third thing, which is really a trap: **you are not always asked.** The dialog is
interactive-only — *"A `claude -p` run or an SDK session never shows it"* — so a scripted
run walks straight past the gate. See the anti-patterns below for what to pass instead.

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
5. **Auto-loaded plugins / `.claude/skills`** — since v2.1.157, a folder here carrying a
   `.claude-plugin/plugin.json` manifest loads as a plugin with no marketplace and no
   install step, once you accept the workspace trust dialog. Read what's there — starting
   with any `allowed-tools`. Since **v2.1.233** you get a free first pass:
   `claude plugin validate <path>` now checks *"a bare `.claude/skills` directory,
   reporting SKILL.md files whose frontmatter fails to parse."* Take it for exactly what it
   is: it tells you a skill is well-*formed*, not that it's harmless.
6. **Config that never came from the repo.** Different door, and items 1-5 won't catch it:
   skills you enabled on your claude.ai account. Claude Code downloads them into
   `~/.claude/skills/synced/` when `CLAUDE_CODE_SYNC_SKILLS` is set in a non-interactive
   run, and every local session afterwards loads them from there. Audit that folder like
   any other. And know that names collide across sources: `/skills` and `/context` group
   synced skills under `claude.ai sync`, and the `/` menu labels them — use that label when
   a familiar command starts behaving in an unfamiliar way.

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
(`.claude/settings.local.json`). The permissions reference states the principle plainly:
*"The repository's own approvals don't count."* **A repo doesn't get to vouch for itself.**

### And the newer case is the stronger one

Two months later, **v2.1.232** (13 August 2026):

> *"Fixed nested git repositories inheriting trust from a parent directory; each repository
> now requires its own trust confirmation."*

Read what that implies about the behavior it replaced, because that's the durable lesson:
trusting a directory also covered every git repository nested inside it. A submodule, a
vendored dependency, a sub-repo in a monorepo — code you never opened, covered by an approval
you gave for something else. It's fixed. But the reason it was worth fixing is the reason to
keep the reflex: **a trust decision has a blast radius, and it was wider than most people
assumed it was.**

Same week, a different door — **v2.1.228** (11 August 2026):

> *"Hardened skills synced from claude.ai: they no longer shadow local commands or MCP
> prompts, their descriptions are sanitized and labeled, and on your machine their bodies
> don't run `!` commands or expand `@` files."*

That one isn't about repositories at all. It's about configuration arriving through your
*account* — which is why item 6 exists. And note what it says about name collisions: they
are real, and they don't always resolve the way you'd guess. **v2.1.233** fixed bundled skill
aliases like `/checkup` and `/review` reporting `Unknown command` in `-p` mode when a user or
project skill shadows the bundled skill. Sources shadow each other in both directions. The
`claude.ai sync` label exists because you're meant to read it.

Three releases, one direction: the CLI is making the **source** and the **scope** of a piece
of configuration visible and enforceable. That's the same job the checklist above does by
hand. Read the scope honestly, though — none of it reads your hooks or your `env` overrides
for you. The tooling took items off your plate; it didn't take the plate.

## Anti-patterns to drop

- Reading `.claude/` as "just configuration" instead of as code.
- Clicking **trust** reflexively to get to work.
- **Assuming one trust click covered the whole tree.** It didn't cover nested repositories
  before v2.1.232, and it still doesn't cover everything *inside* the folder.
- **Using `claude -p` to "take a quick look" at an unfamiliar repo.** The trust dialog never
  appears in `-p`, and the project's `.mcp.json` servers are *"connected without asking,
  approved or not."* Pass `--bare`, which *"skips auto-discovery of hooks, skills, plugins,
  MCP servers, auto memory, and CLAUDE.md"* — the docs are blunt about why it exists, since
  without it *"Claude Code runs the hooks in a project's `.claude/settings.json` even in a
  folder you've never trusted."* Narrower options for a single run: `--setting-sources user`,
  or `--settings '{"disableAllHooks": true}'`.
- **Turning hooks off in your own user settings and calling it done.** Per the docs, that
  isn't enough: the repository's project settings take precedence over yours and can set
  `disableAllHooks` back to `false`. Pass it for the run instead.
- Running an old CLI version "because it works." (These fixes only protect you if you
  actually have them.)
- Copy-pasting a `settings.json`, agent, or hook you found online without reading it.

## Stay current — the #1 mitigation

Every issue above was resolved by an update. Keeping the CLI current is not housekeeping;
it's your primary defense. Check your version with `claude doctor` and update regularly.

While you're at it, use the levers Claude Code gives you to *reduce* what a setup can do.
`disallowed-tools` in a skill's frontmatter (v2.1.152) removes specific tools (like shell
access) from Claude's pool while that skill is active. Read its limit, though: per the docs,
*"the restriction clears when you send your next message."* It's a per-invocation guard, not
a policy. For anything you never want a skill to reach, the docs are direct — *"To block
tools across all skills and prompts, add deny rules in your permission settings"* — which is
[the other half of this reflex](harden-claude-code-permissions.md). Least privilege applies
to AI setups too.

---

<sub>Last revalidated **2026-08-17** against Claude Code **v2.1.233**. This guide is
maintained alongside our [weekly watch](../../watch/) — when the ecosystem shifts, this
page is re-checked. Sources are listed in the page header.</sub>

<sub>Doing this by hand across many projects gets tedious.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> scans your projects
locally and lays out their Claude Code config in one place — settings, every hook with the
exact command it runs, MCP servers, and any hardcoded secrets it finds — so the review
above takes seconds. Use it or not; the reflex is what matters.</sub>
