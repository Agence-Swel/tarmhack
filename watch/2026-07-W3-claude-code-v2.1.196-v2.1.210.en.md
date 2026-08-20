---
title: "Claude Code — What's changing this week (v2.1.196 → v2.1.210)"
date: 2026-07-15
period: "Week W3 · v2.1.196 → v2.1.210"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: en
---

# Claude Code — What's changing this week

> Period: week W3 (July 2026) · Versions v2.1.196 → v2.1.210 (15 releases)
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Generated 2026-07-15

> **Stability status — unconfirmed this run.** The npm dist-tags report versions (`stable 2.1.142` / `latest 2.1.150`) that are *lower* than versions already published in the official changelog — so they're out of sync this week. As a precaution we're not tagging any item below as "stable" or "in preview": check `claude --version` and your update channel before shipping to production.

---

## What actually matters

### Sonnet 5 is now the default model — v2.1.197

From the moment you update, your sessions run on Sonnet 5, with a native 1M-token context window.

> *"Introducing Claude Sonnet 5: now the default model in Claude Code, with a native 1M-token context window and promotional pricing of $2/$10 per Mtok through August 31. Update to version 2.1.197 for access."*

**What this means in practice**: once you're on 2.1.197+, your sessions use Sonnet 5 unless you've explicitly pinned another model. The native context window jumps to 1M tokens — you can hand it far more code at once without truncation, and promo pricing runs through August 31. On Bedrock, Vertex and Claude Platform on AWS, the new default is Opus 4.8 (v2.1.207). Check what `/model` shows after updating: if you were relying on an implicit default, it may have moved.

---

### The "default" permission mode is now called "Manual" — v2.1.200

Same mode as before — the one where Claude asks before each sensitive action — renamed to say plainly what it does.

> *"Changed the "default" permission mode to "Manual" across the CLI, `--help`, VS Code, and JetBrains; `--permission-mode manual` and `"defaultMode": "manual"` are accepted alongside `default`"*

**What this means in practice**: existing configs don't break, `default` is still accepted. If you script or document workflows, switch to `manual`: that's the term that will stick. A grey ⏸ badge now also shows in the footer when this mode is active (v2.1.202), so the active mode is always visible.

---

### Auto mode is now available without opt-in on cloud providers — v2.1.207

On Bedrock, Vertex and Foundry, auto mode (Claude decides which commands are safe on its own) is there by default — and it's getting stricter.

> *"Auto mode is now available without `CLAUDE_CODE_ENABLE_AUTO_MODE` opt-in on Bedrock, Vertex AI, and Foundry; disable via `disableAutoMode` in settings"*

**What this means in practice**: to turn it off, set `disableAutoMode` in your settings. Two guardrails worth noting: background task notifications now explicitly state that no human input occurred, so a fabricated approval slipped into the transcript can't be acted on (v2.1.205); and auto mode now asks before running `rm -rf` on a variable it can't resolve from context (v2.1.205). One detail that matters: auto mode no longer reads its config from the repo's `.claude/settings.local.json` — only `~/.claude/settings.json` (or managed settings) counts (v2.1.207). If you thought you were configuring it per project, you aren't anymore.

---

### Subagents run in the background by default, all the way to the PR — v2.1.198

When Claude delegates to a subagent, it no longer makes you wait — and a background agent that finishes code work now goes all the way to opening a PR.

> *"Subagents now run in the background by default, so Claude keeps working while they run and is notified when they finish"*
>
> *"Background agents launched from `claude agents` now commit, push, and open a draft PR when they finish code work in a worktree, instead of stopping to ask"*

**What this means in practice**: Claude keeps going and notifies you at the end, instead of making you wait. And a background agent no longer stops to ask: it commits, pushes and opens a draft PR. A real fluidity gain, but keep an eye on your `claude agents` list: more now happens there without you.

---

## Notable improvements

**Structured output is more reliable — and stricter (v2.1.205)** — `--json-schema` sometimes produced unstructured output silently when the schema was invalid: that's fixed, and schemas using the `format` keyword are now explicitly rejected. If you generate structured JSON via a schema, double-check it doesn't lean on `format` — otherwise you'll need to rework it.

**Much less memory and latency on long sessions (v2.1.208)** — a performance release on its own: rule matchers are compiled once and cached (no more multi-second slowdowns when you have many deny/ask rules), MCP tool-pool assembly is cached (up to 7× faster in print/SDK), several memory leaks are plugged, and edit-heavy transcript size drops up to 79×. If your long sessions were dragging, the update is felt.

**`AskUserQuestion` no longer answers for you (v2.1.200)** — question dialogs no longer auto-continue by default; you can opt back into an idle timeout via `/config`. A question now genuinely waits for your answer.

---

## Important bug fixes

**Isolated worktrees are actually isolated (v2.1.210)** — fixes `isolation: 'worktree'` subagents that could run git-mutating commands against the main checkout instead of their own worktree — a real risk of polluting the repo. The same release frees `git worktree` locks left behind by a killed session (a periodic sweep releases locks whose owning process is gone). If you were running background agents on worktrees, this one's worth taking.

**Background agents survive updates and restarts better (v2.1.200 / 205 / 208 / 210)** — an older daemon can no longer take over workers spawned by a newer version; replies typed to a background agent are no longer lost if delivery fails (they're saved and delivered on restart, v2.1.208); and `claude attach` waits for the daemon to settle instead of failing with "job not found" mid-transition (v2.1.210).

**MCP security: no more self-approval by a repo (v2.1.196)** — `claude mcp list`/`get` no longer spawn `.mcp.json` servers that a repo self-approved via a committed `.claude/settings.json`; an untrusted workspace shows `⏸ Pending approval`. If you open unfamiliar code, its MCP servers no longer start behind your back.

---

## In brief (minor / cosmetic)

- v2.1.198: **Claude in Chrome** is now generally available; new `/dataviz` skill (chart design + color-palette validator).
- v2.1.208: **screen reader mode** (`claude --ax-screen-reader` / `CLAUDE_AX_SCREEN_READER=1` / `"axScreenReader": true`); `vimInsertModeRemaps` setting (map `jj` → Escape in vim mode); `CLAUDE_CODE_PROCESS_WRAPPER` to enforce a corporate launcher.
- v2.1.196: org-wide default model configurable by the admin, shown as "Org default" in `/model`; readable session names at start.
- v2.1.202: OpenTelemetry attributes `workflow.run_id` / `workflow.name` on workflow-spawned agents; "Dynamic workflow size" setting in `/config`.
- v2.1.206: `/commit-push-pr` auto-allows `git push` to the configured push remote (`remote.pushDefault`); directory path suggestions in `/cd`.
- v2.1.210: live elapsed-time counter on long tool calls; writing a MEMORY.md index over the read limit now returns an explicit error instead of silent truncation.
- v2.1.209: fix — `/model` and other dialogs were being blocked in background `claude agents` sessions.

---

**Going further** — This week, `claude mcp list`/`get` stop starting the MCP servers a repo self-approved via a committed `.claude/settings.json` (v2.1.196): the CLI now natively enforces the reflex we've recommended from the start — treat a cloned repo's configuration as untrusted code. The durable reflex, with a free checklist: our guide [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.md).

---

*Watch generated automatically from Anthropic's official changelog.*
*Any interpretation is clearly marked as such.*
