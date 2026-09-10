---
title: "Claude Code - What's new this week (v2.1.169 → v2.1.176)"
date: 2026-06-15
period: "Week W3 · v2.1.169 → v2.1.176"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: en
---

# Claude Code - What's new this week

> Period: Week W3 (3rd week of June) · June 8-12, 2026 · Versions v2.1.169 → v2.1.176 (8 releases)
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Generated on 2026-06-15

---

## What actually matters

### A new model, Claude Fable 5 - v2.1.170

Claude Code announces a new model, Claude Fable 5, available from v2.1.170. The changelog doesn't hedge.

> *"Introducing Claude Fable 5: a Mythos-class model […]. Fable's capabilities exceed those of any model we've ever made generally available."*

**What this means in practice**: to get access, update to 2.1.170 or later. One handy detail - Fable 5 ships with 1M context by default, and the CLI now handles that cleanly: the `[1m]` suffix in the model name is stripped automatically (v2.1.173). Nothing to fiddle with on the model-name side.

---

### Locking down which models are allowed gets real - v2.1.175

Until now, the `availableModels` setting listed the permitted models. The new `enforceAvailableModels` managed setting makes it binding.

> *"the `availableModels` allowlist also constrains the Default model […], and user or project settings can no longer widen a managed `availableModels` list"*

**What this means in practice**: if you administer Claude Code for a team, this is the lever that guarantees an unapproved model won't run anywhere - the restriction now reaches everywhere, including sub-agent model overrides, the agent dispatch picker, and the advisor model (v2.1.172). If you're a developer on a managed team, expect your model list to be locked from the top.

---

### Sub-agents can now nest - v2.1.172

A sub-agent can now spawn its own sub-agents, five levels deep.

> *"Sub-agents can now spawn their own sub-agents (up to 5 levels deep)"*

**What this means in practice**: powerful for breaking apart a complex task. The flip side: tracking what's actually running gets harder - the same release fixes a nested sub-agent that stayed stuck as "active" after its child was stopped. Our take: use the cascade, but keep an eye on the depth.

---

### A safe mode for debugging - v2.1.169

The new `--safe-mode` flag (and the `CLAUDE_CODE_SAFE_MODE` variable) starts Claude Code with all customizations disabled.

> *"with all customizations (CLAUDE.md, plugins, skills, hooks, MCP servers) disabled"*

**What this means in practice**: when something misbehaves and you can't tell whether it's a hook, a plugin, or an MCP server, this is the move - launch in safe mode and isolate the problem in one command instead of disabling your extensions one by one.

---

## Notable improvements

**`/cd` without breaking the cache (v2.1.169)** - the `/cd` command moves a session to a new working directory without breaking the prompt cache. Useful for switching folders without starting over or losing your cache.

**Fuller session supervision (v2.1.169)** - `claude agents --json` gains an `--all` flag to include completed sessions, plus `id` and `state` fields; it no longer drops blocked or just-dispatched sessions. If you script supervision of your background sessions, you finally get the full picture.

**Bedrock region read from `~/.aws` (v2.1.172)** - on Amazon Bedrock, the CLI now reads the region from your `~/.aws` config files when `AWS_REGION` isn't set, matching AWS SDK precedence, and `/status` shows where the region came from. Less redundant configuration.

**Team memory recovered in remote sessions (v2.1.172)** - memory recall now finds mounted "team memory stores" set via `CLAUDE_MEMORY_STORES`, including in remote sessions.

---

## Important bug fixes

**Clean fallback without Opus 4.8 (v2.1.176)** - on organizations without Opus 4.8, the auto-mode classifier now falls back cleanly to the best available Opus model instead of failing. Fixed.

**Fable 5 credits banner (v2.1.174)** - the "Fable 5 is now consuming usage credits" banner no longer shows incorrectly for enterprise accounts on usage-based billing. Fixed.

**Wildcard permission rules (v2.1.172)** - `WebFetch(domain:*.example.com)` matches subdomains, and `Read(secrets-*/config.json)` (mid-pattern wildcard) is no longer rejected at startup. If you had rules that were silently ignored, check them. Fixed.

**Transcripts recovered (v2.1.170)** - sessions launched from the VS Code integrated terminal (or any shell that inherited Claude Code environment variables) save their transcripts again and reappear in `--resume`. Fixed.

**Responsiveness in long conversations (v2.1.172)** - redundant message normalization removed in long conversations, and fewer re-renders while sub-agents run in parallel. The app stays more responsive as history grows.

---

## Minor / cosmetic

- v2.1.174: `wheelScrollAccelerationEnabled` disables mouse-wheel scroll acceleration in fullscreen.
- v2.1.176: `footerLinksRegexes` - regex-matched link badges in the footer row.
- v2.1.169: `disableBundledSkills` (and `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS`) hides bundled skills, workflows, and built-in slash commands.
- v2.1.176: session titles are generated in your conversation's language; pin one via the `language` setting.
- v2.1.174: the `/model` picker shows more clearly which family "Default" resolves to (Opus, Sonnet depending on plan).
- v2.1.172: Claude in Chrome tools now load in a single batched call instead of one per tool; OTEL metric `claude_code.lines_of_code.count` gains a `model` attribute.
- v2.1.172 / v2.1.175: Bedrock GovCloud (`us-gov-*`) and credential-caching fixes.
- v2.1.169 / v2.1.172 / v2.1.174 / v2.1.176: numerous Remote Control and background-session fixes (respawn, reconnection, Windows daemon).

---

**Going further** - This week adds a managed governance lever: `enforceAvailableModels` (v2.1.175) locks down the list of allowed models for a whole team, even through a sub-agent. How to turn it into a durable least-privilege posture (deny-by-default, sensitive writes, a version window, a model allowlist): our guide [Harden your own Claude Code permissions: go deny-by-default](../resources/guides/harden-claude-code-permissions.md).

---

*Compiled from the official Anthropic changelog.*
*Interpretations and use-case suggestions are explicitly marked as such.*
