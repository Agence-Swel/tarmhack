---
title: "Claude Code — What's new this week (v2.1.211 → v2.1.226)"
date: 2026-08-10
period: "Week W2 · v2.1.211 → v2.1.226"
sources:
  - https://docs.claude.com/en/docs/claude-code/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: en
---

# Claude Code — What's new this week

> Period: week W2 (August 2026) · Versions v2.1.211 → v2.1.226 (16 releases)
> Sources: [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [docs.claude.com](https://docs.claude.com/en/docs/claude-code/changelog)
> Generated 2026-08-10

> **Stability status — confirmed this run.** The npm dist-tags line up with the official changelog: `stable 2.1.220` / `latest 2.1.226`. In other words, anything ≤ v2.1.220 is already on the stable channel; anything ≥ v2.1.221 is still in preview and may change. Check `claude --version` and your update channel before shipping to production.

---

## What actually matters

### Claude Opus 5 becomes the default Opus model — v2.1.219

The context window jumps to one million tokens, and fast mode changes hands.

> *"Added Claude Opus 5 (`claude-opus-5`), now the default Opus model — 1M context, fast mode at $10/$50 per Mtok"*
>
> *"Removed Opus 4.7 from fast mode; `/fast` now applies to Opus 5 and Opus 4.8"*

**What this means in practice**: you can load far more code, logs, or documentation into a single session before the model loses the thread. On a large repo, that's the difference between chunking your problem and handing it over whole. If you had pinned an older Opus model, check your `/model` picker: the Opus row is now merged and displayed as "Opus (1M context)." If you stay on the default, there's nothing to do.

---

### Plan mode no longer runs commands that modify your files — v2.1.212

Plan mode is meant to think, not act. That wasn't entirely true.

> *"Fixed plan mode auto-running file-modifying Bash commands (e.g. `touch`, `rm`) without a permission prompt or SDK `canUseTool` callback"*

**What this means in practice**: an `rm` or a `touch` could slip through with no approval prompt and no `canUseTool` callback. That's closed. If you use plan mode to scope work before executing, you can trust it again on this specific point. The fix applies on its own once you update — nothing to configure.

---

### A `dir/**` pattern no longer means the same thing everywhere — v2.1.214

This is the quietest change of the week, and the one most likely to catch you out.

> *"Fixed single-segment `dir/**` allow rules like `Edit(src/**)` auto-approving writes to nested `dir/` directories anywhere in the tree instead of only `<cwd>/dir`"*
>
> *"Changed single-segment `dir/**` hook `if:` conditions to match only `<cwd>/dir`; write `**/dir/**` for any-depth matching. `deny`/`ask` permission rules keep their any-depth match."*

**What this means in practice**: two things, and you need to hold them together. First, a rule like `Edit(src/**)` was wrongly approving writes to *any* `src/` folder in the tree, not just your project's — that's fixed, and it's good news. Second, and here's the trap: inside a **hook `if:` condition**, a single-segment `dir/**` now only targets `<cwd>/dir`. To get an any-depth match back, you must write `**/dir/**`. But `deny` and `ask` permission rules **keep** their any-depth match. The same pattern, written in two places, no longer covers the same ground. If you have hooks that filter by path, re-read them: they may have quietly narrowed to your current directory.

---

### `claude agents` now asks for trust on an unknown directory — v2.1.225

The two entry points finally behave the same way.

> *"Added a workspace trust prompt to `claude agents` for untrusted directories, matching the behavior of `claude`"*

**What this means in practice**: launching a background agent in a directory you never approved meant running code without the guardrail `claude` already enforces. That's now aligned. The first time you run `claude agents` in a new folder, expect a trust prompt; approve it once and it won't come back for that folder. On substance, this is the right direction: the same risky action triggers the same check, whichever door you came through.

---

## Notable improvements

**Stronger secret masking in the sandbox (v2.1.224, v2.1.221)** — new options to hide credentials from sandboxed commands: targeted regex extraction (`extract`, `onExtractNoMatch`), JWT-aware masking (`decode: "jwt"` + `maskClaims`), AWS SigV4 re-signing (`awsPairs`/`sigv4`), and a `mode: "mask"` that lets the command read a sentinel copy while the proxy substitutes the real value on egress. If you run commands that need a token without ever exposing it in the clear, this is exactly the tool.

**An `archive` plugin source, with SHA-256 pinning (v2.1.224)** — you can now install a plugin from a zip served over HTTPS, without git or npm, with optional SHA-256 pinning. Handy in locked-down environments. The important word is *optional*: without pinning, you install whatever the server hands you on the day you call it. If you use this source, pin the hash.

**A cap on web searches and subagents (v2.1.212)** — per-session limits on WebSearch calls (200 by default) and subagent spawns (200 by default; `/clear` resets the counter), tunable via environment variable. The goal is to cut runaway loops that burn budget without producing anything. Worth noting: the 200-subagents-per-session cap was removed again in v2.1.224.

---

## Important bug fixes

**Several permission-check bypasses closed (v2.1.214, v2.1.216, v2.1.223)** — commands over 10,000 characters now always prompt instead of running automatically; file-descriptor redirect forms the analyzer misread now fail closed; zsh subscripts inside `[[ ]]` are no longer treated as inert text; and a command padded with tabs or invisible Unicode characters can no longer hide part of itself from the approval dialog.

> *"Fixed Bash permission checks misjudging very long commands — commands over 10,000 characters now always prompt instead of running automatically"*
>
> *"Fixed permission prompts so commands padded with tabs or invisible Unicode can no longer hide part of the command from the approval dialog"*

**What this means in practice**: four distinct bypasses of the same guardrail, closed across sixteen releases. That's not a run of bad luck — deciding whether to allow a command by parsing shell is a structurally hard problem. If you lean on Bash approval as a safety net, take these fixes; but keep in mind it's a net, not a wall.

---

## Minor / cosmetic

- v2.1.211: `--forward-subagent-text` (and `CLAUDE_CODE_FORWARD_SUBAGENT_TEXT`) includes subagent text and reasoning in stream-json output; v2.1.219 extends forwarding to nested subagents.
- v2.1.211: "always allow" rules now save at the repository root, so they persist across worktrees.
- v2.1.212: `/fork` copies the conversation into a background session; the Task tool's `mode` parameter is deprecated — subagents inherit the parent's permission mode.
- v2.1.214: new `modified` timestamp in memory file frontmatter; `EndConversation` tool added; new OpenTelemetry attributes (`message.uuid`, `client_request_id`, `tool_source`).
- v2.1.216, v2.1.222: workflow and scheduled-task writes no longer follow a `.claude` symlink; worktree isolation applies to edits and Bash in every session.
- v2.1.218: `/code-review` runs as a background subagent and no longer fills your conversation; Claude no longer auto-launches `/verify`, `/code-review`, or `/deep-research`.
- v2.1.223: `/review` becomes an alias of `/code-review`; `/teleport` arrives.
- v2.1.224: `claude self-hosted-runner` (Team and Enterprise plans); `crossSessionInbound` and `dialogExpiry` settings.
- v2.1.220, v2.1.226: "Bug fixes and reliability improvements."

---

**Going further** — This week is dense with permission-control hardening: four bypasses closed (v2.1.214, v2.1.216, v2.1.223) and an `Edit(src/**)` rule that stops auto-approving the whole tree. The lasting lesson isn't "there were holes" — it's that an approval filter that parses shell is fragile by nature, and that a narrow allowlist beats a denylist. The full reasoning, with a free checklist: our guide [Harden Claude Code permissions](../resources/guides/harden-claude-code-permissions.md).

---

*Watch generated from Anthropic's official changelog.*
*Interpretations and use cases are explicitly labeled as such.*
