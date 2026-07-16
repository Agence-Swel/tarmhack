---
title: "Harden your own Claude Code permissions: go deny-by-default"
description: "As you spread Claude Code across more projects, its permissions quietly drift toward 'allow.' Here's how to flip to deny-by-default and lock the writes that grant code execution — free, manual, vendor-neutral."
last_revalidated: 2026-06-29
claude_code_ref: v2.1.195
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

And the direction held. Over the next two weeks (v2.1.178 → v2.1.195), the same theme kept
shipping:

- **Permission rules went parametric** (v2.1.178). You can now scope a rule to a tool's
  *input parameters*, not just its name. The changelog: *"Added `Tool(param:value)` syntax
  for permission rules to match a tool's input parameters (with `*` wildcard), e.g.
  `Agent(model:opus)` to block Opus subagents."* That's least-privilege moving from "which
  tools" down to "which arguments." A follow-up closed a gap where it didn't fully bite —
  v2.1.186: *"Fixed `Agent(type)` deny rules and `Agent(x,y)` allowed-types restrictions not
  being enforced for named subagent spawns."*
- **A sandbox setting for credentials** (v2.1.187): *"Added `sandbox.credentials` setting to
  block sandboxed commands from reading credential files and secret environment variables."*
  If you run anything sandboxed, that's a turn-it-on reflex.
- **You can finally read back what's refused** (v2.1.193): *"Added auto-mode denial reasons
  to the transcript, the denial toast, and `/permissions` recent denials."* The failure mode
  described below is *not being able to see what your config refuses* — now you can.
- **A backstop for destructive commands** (v2.1.183): auto mode now blocks
  `git reset --hard`, `git clean -fd`, and `terraform`/`pulumi`/`cdk destroy` when you didn't
  ask to throw work away. Not a substitute for a deny floor — a net behind it.

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
6. **In a team, lock the models that can run.** Least privilege isn't only about tools and
   versions — it applies to *which models* execute on your behalf. Put an `availableModels`
   allowlist in managed settings and set `enforceAvailableModels: true` (v2.1.175). As the
   changelog puts it, with this on "the `availableModels` allowlist also constrains the
   Default model," and "user or project settings can no longer widen a managed
   `availableModels` list." Just as important, v2.1.172 made that allowlist apply *across the
   board* — to sub-agent model overrides, the agent dispatch model picker, and the advisor
   model. That matters because, since v2.1.172, "sub-agents can now spawn their own sub-agents
   (up to 5 levels deep)": without enforcement reaching those overrides, a nested agent could
   pick a model you never approved. With it, the approved list holds everywhere — no exception
   you have to remember.
7. **Scope rules to parameters, not just tools.** Since v2.1.178, a permission rule can target
   a tool's input parameter with `Tool(param:value)` and a `*` wildcard. So you don't have to
   allow or deny a tool wholesale: you can deny `Agent(model:opus)` and leave the rest. Write
   rules that say what you actually mean. One caveat — if you set an `Agent(type)` deny or an
   `Agent(x,y)` allowed-types rule before v2.1.186, re-check it: those weren't being applied to
   named sub-agent spawns until that fix.
8. **Turn on `sandbox.credentials`.** If you run sandboxed commands, set `sandbox.credentials`
   (v2.1.187) so they can't read your credential files or secret environment variables. It's
   cheap and it closes an obvious exfiltration path.

## Anti-patterns to drop

- **Allow-by-default you've never audited.** If you can't say what your config refuses,
  it refuses nothing.
- **`acceptEdits` as a global default.** Convenience that writes code-granting files on
  your behalf is not convenience.
- **Running an old version "because it works."** The week's hardening only protects you if
  you actually have it. A version window (v2.1.163) makes that a team setting, not a hope.
- **Assuming a managed model allowlist can be loosened locally.** Since v2.1.175, a project
  or user `settings.json` can no longer widen a managed `availableModels` list — and with
  `enforceAvailableModels: true` it binds the Default model too. If you set it as the floor,
  treat it as the floor; don't expect a local override to win.
- **Never reading what got refused.** Since v2.1.193 your denials show up in the transcript,
  in a toast, and under `/permissions` recent denials. A deny floor you never look at is a
  guess; the recent-denials list tells you whether it's matching what you intended — or just
  blocking real work and training you to widen it.
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

<sub>Last revalidated **2026-06-29** against Claude Code **v2.1.195**. This guide is
maintained alongside our [weekly watch](../../watch/) — when the ecosystem shifts, this
page is re-checked. Sources are listed in the page header.</sub>

<sub>Auditing this by hand across many projects gets tedious.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> reads your projects'
Claude Code config locally and lays out what each one allows and denies in one place — so
spotting the project that's still allow-by-default takes seconds instead of ten file opens.
Use it or not; the posture is what matters.</sub>
