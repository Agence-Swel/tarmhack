---
title: "Treat a cloned repo's Claude Code setup as untrusted code"
description: "When you open someone else's project in Claude Code, its hidden config can run on your machine. Here's the durable reflex - and a free checklist - to stay in control."
last_revalidated: 2026-09-10
claude_code_ref: v2.1.267
sources:
  - https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/
  - https://flatt.tech/research/posts/poisoning-claude-code-one-github-issue-to-break-the-supply-chain/
  - https://code.claude.com/docs/en/security
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/skills
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://www.npmjs.com/package/@anthropic-ai/claude-code?activeTab=versions
---

# Treat a cloned repo's Claude Code setup as untrusted code

## Why this is on your radar right now

In late May 2026, Claude Code made sharing setups even easier: plugins placed in a
project's `.claude/skills` directory are now **loaded automatically, with no marketplace
step** (v2.1.157). A single `git clone` is enough for a teammate - or a stranger - to
hand you their full environment.

That's genuinely convenient. It's also a good moment to internalize one reflex:

> **A repository's Claude Code configuration is not metadata. It is executable code you
> are about to trust.**

And the surface hasn't stopped moving. In August 2026, Anthropic had to fix nested git
repositories inheriting trust from a parent directory (v2.1.232) - which says something
useful about how far a single **trust** click used to reach. In the four weeks after, through
**v2.1.267**, two more things happened: the fence that keeps a plugin or a marketplace inside
its own folder was repaired four separate times, and folder trust acquired teeth in a place
where you are never shown the dialog. More on both below.

### First, check which "current" you are on

Before anything else, and this has nothing to do with any vendor's product. On **10 September
2026**, `npm dist-tag ls @anthropic-ai/claude-code` returns two very different answers:

| Channel | Version | Published |
|---|---|---|
| `stable` | **2.1.236** | 19 August 2026 |
| `latest` | **2.1.267** | 9 September 2026 |

The `stable` channel has not moved in three weeks. Count it exactly, because the round
number is the part people get wrong: of the fourteen releases cited on this page, **eight
landed after 19 August** - a reader who dutifully "keeps Claude Code up to date" on `stable`
has none of those eight. A slower channel is a legitimate choice and that is why it
exists. But it makes two sentences that sound identical mean different things: *"I'm on the
latest version"* and *"I have that fix"*.

**The free check:** run `claude doctor`, read the version, compare it to the version tags in
this page. Below the fix, you don't have the fix, whatever channel you follow.

## The threat, in plain terms

When you open a project, Claude Code reads its local configuration: `.claude/settings.json`,
hooks, MCP servers, environment overrides, and now auto-loaded plugins. Anyone with commit
access to a repo can change those files - and they don't get the scrutiny that application
code gets in review, because we instinctively read them as "just settings."

This isn't hypothetical. Two documented issues in 2026 made the point:

- **CVE-2025-59536** (Check Point Research, CVSS 8.7) - a hook planted in a repo's settings
  file could execute shell commands **before the trust dialog even appeared**. Open the
  folder, and code ran.
- **CVE-2026-21852** (CVSS 5.3) - a project could override `ANTHROPIC_BASE_URL` in its
  config, causing Claude Code to send an authenticated request (carrying your API key)
  to an attacker's server **before** you confirmed you trusted the project.

Separately, GMO Flatt Security showed a supply-chain path where **a single GitHub issue**
could be enough to poison a downstream setup.

**Important, and the reason this guide is not fear-mongering:** all of these are **fixed**
in current Claude Code releases. The bugs are gone. What stays useful is the *habit* they
expose - because the underlying surface (config that can act on your behalf) is permanent.

## The durable principle

Treat the trust prompt as a real security decision, not a dialog to dismiss. Before you
let Claude Code act inside an unfamiliar repository, assume its config can run commands,
reach the network, and reconfigure your tools - and decide deliberately.

**Then know what that decision covers.** A trust decision has a *scope*, and the scope is
not "everything, forever, everywhere below this folder." Two axes, both straight from the
official permissions reference:

- **Where it applies.** Trust is saved per workspace, *"keyed on the git repository root
  or, outside a repository, the directory you started Claude Code from."* That matters the
  moment a repo contains other repos - submodules, vendored dependencies, a monorepo's
  sub-repos. Until **v2.1.232**, those nested repositories inherited the parent's trust.
  Now each one asks for itself. On a current CLI, expect *more* prompts, not fewer.
- **What it covers inside that folder.** Not everything is gated by it. The docs table
  *What runs before you trust a folder* is explicit: when you've trusted only a parent
  folder, hooks in settings files, the `env` block, helper commands such as `apiKeyHelper`,
  and a project skill's hooks and `allowed-tools` are **used** - while `permissions.allow`
  rules and `additionalDirectories` are held back until you accept. On skills the docs go
  further: *"A skill can grant itself broad tool access, so review the `allowed-tools` of
  skills checked into a repository before you run Claude Code there."*

And a third thing, which is really a trap: **you are not always asked** - and since August,
"not asked" no longer means the same thing everywhere. The dialog is still interactive-only:
*"A `claude -p` run or an SDK session never shows it."* But an unanswered dialog now has
consequences of its own, and one supposedly interactive command was skipping the prompt
outright. Both cases are below.

## A free, manual checklist (no tools required)

Open these in a **plain text editor** - not by launching Claude Code inside the repo -
when the source isn't someone you already trust:

1. **`.claude/settings.json`** and **`.claude/settings.local.json`** - read them fully, and
   look first for `defaultMode`, the loudest thing a repository could ever commit. Until
   **v2.1.257**, a project settings file could set it to `"bypassPermissions"` and be obeyed;
   that release *"Changed `defaultMode: "bypassPermissions"` in `.claude/settings.json` or
   `.claude/settings.local.json` to be ignored, like `"auto"`"*. On an older CLI, one
   committed line of JSON turns off every prompt you rely on.
2. **Hooks** - for every hook, read the actual shell command it runs. Anything that pipes
   to a network call, touches credentials, or runs an opaque script is a stop sign.
3. **Environment and header overrides** - watch specifically for `ANTHROPIC_BASE_URL`,
   `apiKeyHelper`, and any var that redirects where your traffic or keys go. Two of those
   doors closed in **v2.1.251**, which tells you they were open: project-level `env` can *"no
   longer set `CLAUDE_CONFIG_DIR`, `CLAUDE_CODE_TMPDIR`, or `TMPDIR`/`TMP`/`TEMP`"*, and
   `ANTHROPIC_CUSTOM_HEADERS` from project settings now *"require[s] approval when it sets a
   credential, org/tenant, routing, or API-behavior header (e.g. `Authorization`, `Host`)."*
   What isn't on those two lists is still whatever the repo says it is.
4. **MCP servers** (`.mcp.json` / configured servers) - what command does each one launch,
   and where does that binary come from? Read a `headersHelper` with the same eye you give a
   hook: it is a command that runs to mint HTTP headers.
5. **Auto-loaded plugins / `.claude/skills`** - since v2.1.157, a folder here carrying a
   `.claude-plugin/plugin.json` manifest loads as a plugin with no marketplace and no
   install step, once you accept the workspace trust dialog. Read what's there - starting
   with any `allowed-tools`, then the **paths** the manifest declares (see *The entry surface
   is wider than it looks*, below). Since **v2.1.233** you get a free first pass:
   `claude plugin validate <path>` checks *"a bare `.claude/skills` directory,
   reporting SKILL.md files whose frontmatter fails to parse"*, and since **v2.1.259** it
   takes *"`--json` … for a machine-readable validation report"* if you run it across many
   repos. Take it for exactly what it is: it tells you a skill is well-*formed*, not that
   it's harmless.
6. **The context files the repo ships** - `CLAUDE.md`, its `@` includes, `.claude/rules`.
   They read like documentation and they are loaded before you do anything, which also makes
   them a file-access surface: **v2.1.234** shipped a *"Security"* fix so that *"remote file
   reads, session restore, CLAUDE.md includes, workflow scripts and file uploads now reject
   Windows NT-namespace (`\??\`) paths, hardening the remaining pre-approval file accesses
   against the NTLM credential-leak vector."* Read the includes, and follow where they point.
7. **Config that never came from the repo.** Different door, and items 1-6 won't catch it:
   skills you enabled on your claude.ai account. Claude Code downloads them into
   `~/.claude/skills/synced/` when `CLAUDE_CODE_SYNC_SKILLS` is set in a non-interactive
   run, and every local session afterwards loads them from there. Audit that folder like
   any other. And know that names collide across sources: `/skills` and `/context` group
   synced skills under `claude.ai sync`, and the `/` menu labels them - use that label when
   a familiar command starts behaving in an unfamiliar way. Synced plugins got the same
   treatment in **v2.1.239**: they *"now show as `name@synced` … and never override a
   same-named plugin you installed."*

If anything looks off, don't open the project in Claude Code from that directory until
you've cleaned or removed it.

## The CLI is starting to back this reflex

A small sign the direction is right. From the official changelog, **v2.1.196**:

> *"Security: `claude mcp list`/`get` no longer spawn `.mcp.json` servers that a repo
> self-approved via a committed `.claude/settings.json`; untrusted workspaces show
> `⏸ Pending approval`."*

Unpack it, because it maps straight onto item 4 above. A repo could approve *its own* MCP
servers by committing `enableAllProjectMcpServers: true` (or an `enabledMcpjsonServers` list)
into `.claude/settings.json` - and then even an innocent-looking `claude mcp list` would
*spawn* those server processes. That's code execution from a command that reads like
inspection. Now, in an untrusted folder, those two commands ignore approval that arrived in a
committed file; it only counts if it lives in a settings file *you* own and haven't committed
(`.claude/settings.local.json`). The permissions reference states the principle plainly:
*"The repository's own approvals don't count."* **A repo doesn't get to vouch for itself.**

### And the newer case is the stronger one

A month later, **v2.1.232** (13 August 2026):

> *"Fixed nested git repositories inheriting trust from a parent directory; each repository
> now requires its own trust confirmation."*

Read what that implies about the behavior it replaced, because that's the durable lesson:
trusting a directory also covered every git repository nested inside it. A submodule, a
vendored dependency, a sub-repo in a monorepo - code you never opened, covered by an approval
you gave for something else. It's fixed. But the reason it was worth fixing is the reason to
keep the reflex: **a trust decision has a blast radius, and it was wider than most people
assumed it was.**

That radius is still being mapped. **v2.1.234** *"Fixed trust prompts omitting the
repository-wide scope warning when the directory was first seen before the repository existed
there"* - the prompt was shown, the sentence telling you how far it reached was not. And
**v2.1.265** *"Fixed Claude Code's own git status and diff probes running clean filters
configured by a nested repository inside the working tree"*: a nested repo could configure a
git filter, and Claude Code's own routine `git status` ran it. Neither is a catastrophe.
Together they say the same thing: **nested is where assumptions go to die.**

Same week as v2.1.232, a different door - **v2.1.228** (11 August 2026):

> *"Hardened skills synced from claude.ai: they no longer shadow local commands or MCP
> prompts, their descriptions are sanitized and labeled, and on your machine their bodies
> don't run `!` commands or expand `@` files."*

That one isn't about repositories at all. It's about configuration arriving through your
*account* - which is why item 7 exists. And note what it says about name collisions: they
are real, and they don't always resolve the way you'd guess. **v2.1.233** fixed bundled skill
aliases like `/checkup` and `/review` reporting `Unknown command` in `-p` mode when a user or
project skill shadows the bundled skill. Sources shadow each other in both directions. The
`claude.ai sync` label exists because you're meant to read it.

### Trust now decides things nobody asks you about

This is the change that most affects the reflex, and it lands squarely on the trap described
above. **v2.1.238** (20 August 2026):

> *"MCP `headersHelper` in a project `.mcp.json`, and inline MCP servers in project or
> `--add-dir` agent files, now require that folder's trust dialog to have been accepted
> (also under `claude -p`)."*

Nothing about the dialog changed: a `-p` run still never shows it. What changed is that its
*absence of an answer* now has a consequence. In a folder you never trusted, those two things
stop rather than start - including in exactly the scripted context where you were never going
to be asked. The gate no longer needs your eyes to be a gate.

Two readings follow. The comfortable one: if a `-p` run in an unfamiliar repo reports that an
MCP server won't come up, that is the feature, not an obstacle to route around. The other one:
it only bites where trust is genuinely absent. Trust once, for a quick look, and every later
headless run in that folder inherits that decision silently, with nothing to remind you.
**A trust click is durable state on your disk, not a one-time answer to one question.**

**The manual parade, free, one minute.** That state is written to `~/.claude.json` - the
v2.1.259 fix for concurrent sessions names the file when it says *"workspace trust no longer
resets"* there. It is plain JSON. Open it and read the project entries: that list is the only
place your past trust decisions are written down, and it is the one page nobody thinks to
re-read.

The same release closed the credential half: a `headersHelper` from a project `.mcp.json`,
plugin, or agent file *"runs without inherited credential env vars."* A repo-supplied helper
no longer starts life holding your tokens.

And the docs' own enumeration - dialog shown when interactive, never in `-p` or the SDK -
turned out to have a third case. **v2.1.248** *"Fixed `claude agents` skipping the workspace
trust prompt when the `CI` environment variable is set."* A stray `CI=true` in a shell
profile, a container image, a task runner: any of those made an interactive command behave
like a headless one. "Am I interactive?" is answered by heuristics, and heuristics have edges.

### The entry surface is wider than it looks

The checklist tells you to read a plugin's `allowed-tools`. Necessary, not sufficient: a
plugin also declares **paths**, and a path is a claim about where a file lives that something
has to verify. Between 28 August and 9 September, four releases repaired that verification.

- **v2.1.251** - *"Fixed plugin commands declared in a marketplace entry being able to point
  outside the plugin directory; such paths are now rejected with a path-traversal error."*
- **v2.1.257** - *"Fixed plugins being able to read files outside their own directory through
  a declared command, agent, skill, hooks or other component path that is a symlink; such
  paths are now refused with an error."*
- **v2.1.265** - *"Fixed a plugin path containing a backslash bypassing the symlink
  containment check on macOS and Linux."*
- **v2.1.267** - *"Fixed a case where a marketplace entry path containing a backslash could
  bypass the containment check for fetched marketplaces on macOS and Linux."*

Four entries, one sentence: **whatever loads from a marketplace, a plugin, or a skill runs
with your rights, and the fence meant to hold it inside its own folder is young code.** The
last two are the same bug found twice - same backslash, two different code paths, one day
apart (8 and 9 September) - which is what a boundary looks like while it is still hardening.

**The manual parade, free, one minute.** Before anything else in an unfamiliar repo, open
every `plugin.json` and marketplace entry and read the paths it declares. Anything that
climbs (`../`), anything that is a symlink, anything spelled with a backslash on a Unix-like
machine: those are the exact shapes all four fixes were about. `ls -lR .claude/` answers the
symlink question; `grep -rn '\.\.' .claude-plugin/` answers the other one.

### The audit isn't a gate you pass once

The checklist reads like something you do at the door. Two releases made that less true.
**v2.1.246** improved `/cd` so that *"the new directory's project settings, hooks, `.mcp.json`
servers (behind the usual approval prompt), skills, and agents now take effect right after the
move instead of on `--resume`."* And **v2.1.257** *"Fixed settings in a `.claude/` folder
created after startup not being picked up until restart"* - so a `.claude/` that appears while
you work, from a `git pull` or a branch checkout, applies immediately.

Both fix a real annoyance. Both mean the honest version of the reflex is: **re-run the
checklist when the config changes under you** - after pulling a branch that touches
`.claude/`, before `/cd`-ing somewhere you haven't read, and when you `--add-dir` a directory,
which contributes its own skills, commands and agents just as a working directory does.

Every one of these releases points the same way: the CLI is making the **source** and the
**scope** of a piece of configuration visible and enforceable. That's the same job the
checklist above does by hand. Read the scope honestly, though - none of it reads your hooks or
your `env` overrides for you. The tooling took items off your plate; it didn't take the plate.

## Anti-patterns to drop

- Reading `.claude/` as "just configuration" instead of as code.
- Clicking **trust** reflexively to get to work - and forgetting that the click persists. It
  is the answer every later headless run in that folder will inherit.
- **Assuming one trust click covered the whole tree.** It didn't cover nested repositories
  before v2.1.232, it still doesn't cover everything *inside* the folder, and as late as
  v2.1.265 a nested repo could still reach out through a git filter.
- **Using `claude -p` to "take a quick look" at an unfamiliar repo.** The trust dialog never
  appears in `-p`, and the docs describe a project's `.mcp.json` servers as *"connected
  without asking, approved or not."* Since **v2.1.238** that blanket statement no longer holds
  for all of it - a `headersHelper` and inline MCP servers are gated on the folder's recorded
  trust, even in `-p` - but the rest of the config still loads. Pass `--bare`, which *"skip
  auto-discovery of hooks, skills, custom commands, subagents, plugins, MCP servers, auto
  memory, and CLAUDE.md"* - the
  docs are blunt about why it exists, since without it *"Claude Code runs the hooks in a
  project's `.claude/settings.json` even in a folder you've never trusted."*
- **Reaching for a narrower flag without checking it does what you think.**
  `--setting-sources user` and `--settings '{"disableAllHooks": true}'` are the right
  instinct for a single run, but **v2.1.246** *"Fixed the command sandbox's filesystem
  configuration not respecting `--setting-sources`"* - below that version, the flag wasn't
  honored everywhere.
- **Turning hooks off in your own user settings and calling it done.** Per the docs, that
  isn't enough: the repository's project settings take precedence over yours and can set
  `disableAllHooks` back to `false`. Pass it for the run instead.
- Running an old CLI version "because it works." And its subtler cousin: **believing you are
  current because you follow a release channel.** On 10 September 2026 `stable` sits at
  v2.1.236 and hasn't moved since 19 August, while `latest` is v2.1.267. Eight of the
  fourteen releases cited on this page live in that gap.
- Copy-pasting a `settings.json`, agent, or hook you found online without reading it.

## Stay current - the #1 mitigation

Every issue above was resolved by an update. Keeping the CLI current is not housekeeping;
it's your primary defense. Check your version with `claude doctor` - and check *which*
version, not merely that an update ran: as the table at the top shows, the two npm channels
are three weeks and thirty-one version numbers apart. If you follow `stable` for good
reasons, keep following it, and simply know what you don't yet have.

While you're at it, use the levers Claude Code gives you to *reduce* what a setup can do.
`disallowed-tools` in a skill's frontmatter (v2.1.152) removes specific tools (like shell
access) from Claude's pool while that skill is active. Read its limit, though: per the docs,
*"the restriction clears when you send your next message."* It's a per-invocation guard, not
a policy. For anything you never want a skill to reach, the docs are direct - *"To block
tools across all skills and prompts, add deny rules in your permission settings"* - which is
[the other half of this reflex](harden-claude-code-permissions.md). Least privilege applies
to AI setups too.

For the specific job this page is about - looking at a repo you haven't read - **v2.1.248**
added a blunter instrument worth knowing:

> *"Added `--restricted` (or `CLAUDE_CODE_RESTRICTED=1`): removes the built-in tools that run
> commands or code and `WebFetch` (unless named in `--tools`), keeps file tools inside the
> working directory, refuses `bypassPermissions`, and ignores user, project and local
> settings files."*

Note the last clause: it ignores the project's settings files outright, which is exactly the
class of file this guide is about. It complements `--bare` rather than replacing it - `--bare`
skips discovery, `--restricted` removes the tools that could act on what was discovered. Neither
replaces reading the seven items above; both buy you the room to read them.

---

<sub>Last revalidated **2026-09-10** against Claude Code **v2.1.267**. This guide is
maintained alongside our [weekly watch](../../watch/) - when the ecosystem shifts, this
page is re-checked. Sources are listed in the page header.</sub>

<sub>Doing this by hand across many projects gets tedious.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> scans your projects
locally and lays out their Claude Code config in one place - settings, every hook with the
exact command it runs, MCP servers, and any hardcoded secrets it finds - so the review
above takes seconds. Use it or not; the reflex is what matters.</sub>
