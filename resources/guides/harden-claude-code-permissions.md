---
title: "Harden your own Claude Code permissions: go deny-by-default"
description: "As you spread Claude Code across more projects, its permissions quietly drift toward 'allow.' Here's how to flip to deny-by-default and lock the writes that grant code execution — free, manual, vendor-neutral."
last_revalidated: 2026-06-08
claude_code_ref: v2.1.168
sources:
  - https://docs.claude.com/en/docs/claude-code/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://code.claude.com/docs/en/security
---

# Harden your own Claude Code permissions: go deny-by-default

## Why this is on your radar right now

The week of June 2–6, 2026 (v2.1.160 → v2.1.168), Claude Code shipped a cluster of changes
that all push the same way: make it cheap to deny by default, and harden the writes that
quietly grant code execution. A few highlights from the official changelog:

- **Glob patterns in deny rules** (v2.1.166, June 6): you can now write `"*"` in the
  tool-name position of a deny rule to **deny all tools**, then re-allow case by case.
  Allow rules reject non-MCP globs, and unknown tool names in deny rules warn at startup.
- **Prompts before code-granting writes** (v2.1.160, June 2): `acceptEdits` mode now
  prompts before writing build-tool config that can run code on your machine — `.npmrc`,
  `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/`. It
  also prompts before touching shell startup files (`.zshenv`, `.zlogin`, `.bash_login`)
  and `~/.config/git/`.
- **Version pinning for teams** (v2.1.163, June 4): `requiredMinimumVersion` and
  `requiredMaximumVersion` managed settings — Claude Code refuses to start outside the
  allowed range.

That's not three unrelated patch notes. It's a direction. Deny-by-default just got cheap.
Use it.

## The problem, in plain terms

When you run one project, you read every permission prompt. When you run ten, you stop.
You click **allow**, you set `acceptEdits` "to stop being interrupted," and over months
your posture drifts: too much is permitted by default, and you can no longer see what is
actually being *refused*. That's the failure mode — not a dramatic breach, just slow,
invisible accumulation of trust you never re-examined.

The fix is a posture, not a tool: **start from deny, and grant the minimum each project
needs.** This is the same least-privilege reflex you'd apply to a CI token or a service
account. AI tooling earned it the moment its config could run code on your machine.

## Flip to deny-by-default

This is a free, manual change to your own `settings.json`. Nothing to install.

1. **Audit what you allow today.** Open your user-level and project-level `settings.json`
   and read the `permissions.allow` list as if someone else wrote it. For each entry, ask:
   does *this* project actually need it? Most don't need most of it.
2. **Set a deny floor, then re-allow.** Set `permissions.deny` to `["*"]` (v2.1.166 makes
   the glob explicit) and re-grant tools one at a time as real work demands them. You'll be
   surprised how short the necessary list is. Unknown tool names in deny rules now warn at
   startup, so typos surface instead of silently doing nothing.
3. **Don't run `acceptEdits` everywhere.** It's convenient and it's a foot-gun. As of
   v2.1.160, `acceptEdits` now prompts before writing the config files that grant code
   execution — but the durable habit is to scope `acceptEdits` to throwaway or sandboxed
   work, not your machine-wide default.
4. **Make `WebFetch` rules explicit.** As of v2.1.162 (June 3), an explicit
   `WebFetch(domain:...)` deny/ask/allow rule now takes precedence over the preapproved-host
   auto-allow. If you care where Claude Code can reach, say so by name rather than relying
   on defaults.
5. **In a team, pin a version window.** Use `requiredMinimumVersion` /
   `requiredMaximumVersion` (v2.1.163) in managed settings so nobody on the team is silently
   running a build that predates a fix.

## Anti-patterns to drop

- **Allow-by-default you've never audited.** If you can't say what your config refuses,
  it refuses nothing.
- **`acceptEdits` as a global default.** Convenience that writes code-granting files on
  your behalf is not convenience.
- **Running an old version "because it works."** The week's hardening only protects you if
  you actually have it. A version window (v2.1.163) makes that a team setting, not a hope.
- **Copying a `settings.json` you found online without reading it.** Same reflex as treating
  [a cloned repo's config as untrusted code](untrusted-repo-config.md) — except here it's
  *your* posture you're handing over. Read every line before it becomes your default.

## One more, for multi-session setups

If you run several Claude sessions that talk to each other, note v2.1.166: messages relayed
via `SendMessage` from another session **no longer carry user authority** — the receiver
refuses relayed permission requests. You don't configure anything; it's just true now. But
it's worth knowing your deny floor can't be lifted by a message arriving from a sibling
session. That's the model working *with* deny-by-default, not around it.

---

<sub>Last revalidated **2026-06-08** against Claude Code **v2.1.168**. This guide is
maintained alongside our [weekly watch](../../watch/) — when the ecosystem shifts, this
page is re-checked. Sources are listed in the page header.</sub>

<sub>Auditing this by hand across many projects gets tedious.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> reads your projects'
Claude Code config locally and lays out what each one allows and denies in one place — so
spotting the project that's still allow-by-default takes seconds instead of ten file opens.
Use it or not; the posture is what matters.</sub>
