---
title: "Claude Code - What's new this week (v2.1.152 → v2.1.159)"
date: 2026-06-01
period: "Week W4 · v2.1.152 → v2.1.159"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: en
---

# Claude Code - What's new this week

> Period: Week W4 (4th week of May) · May 27-31, 2026 · Versions v2.1.152 → v2.1.159
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Generated on 2026-06-01

---

## What actually matters

### Opus 4.8 is here - v2.1.154 (May 28, 2026)

Anthropic's new flagship model is now available in Claude Code. It becomes the default for high-effort tasks (`/effort xhigh`). A fast mode is available at 2× the standard rate for 2.5× the speed.

> *"Opus 4.8 is here! Now defaults to high effort · /effort xhigh for your hardest tasks"*
> *"Fast mode on Opus 4.8 is now available at a fraction of its previous cost: 2x the standard rate for 2.5x the speed"*

**What this means in practice**: if you're on a Max or Team subscription, Opus 4.8 is accessible via `/model`. The `/effort` slider gains a new useful position for long, complex tasks - large-scale refactoring, full codebase analysis, and the like.

**Note**: `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` is deprecated and will be removed on 06/01. If you were using it, switch to `/model claude-opus-4-6[1m]` then `/fast on`.

---

### Dynamic workflows: large-scale background orchestration - v2.1.154 (May 28, 2026)

Claude Code can now decompose a complex task and autonomously orchestrate tens to hundreds of background agents. Use `/workflows` to monitor active runs.

> *"Introducing dynamic workflows: ask Claude to create a workflow and it orchestrates work across tens to hundreds of agents in the background, so you can take on larger, more complex tasks. Run `/workflows` to view your runs"*

**What this means in practice**: you can hand Claude Code an ambitious task - migrate an API, generate tests across an entire project, audit a documentation set - and let it run without staying present. Results are consolidated once all agents complete. Related: `claude agents` now shows workflows alongside regular sessions.

**Shell sessions in the background** (same version): type `! <command>` inside `claude agents` to start an attachable/detachable background shell session, or use `claude --bg --exec '<command>'` from the command line.

---

### New `MessageDisplay` hook - control how responses appear - v2.1.152 (May 27, 2026)

A new hook event, `MessageDisplay`, lets a script transform or suppress assistant response text at display time.

> *"Added a `MessageDisplay` hook event that lets hooks transform or hide assistant message text as it is displayed"*

**What this means in practice**: typical use cases include filtering sensitive content before it appears on screen, automatically prepending context to each response, or collapsing verbose blocks. This hook only affects the displayed output - it does not modify the stored conversation history.

---

### Plugins load from `.claude/skills` without a marketplace - v2.1.157 (May 29, 2026)

Plugins placed in a project's `.claude/skills` directory are now loaded automatically, with no marketplace install step required. The new command `claude plugin init <name>` scaffolds a new plugin directly into `.claude/skills/<name>/`.

> *"Plugins in `.claude/skills` directories are now automatically loaded, no marketplace required"*
> *"Added `claude plugin init <name>` to scaffold a new plugin in `.claude/skills`"*

**What this means in practice**: you can create and share Claude Code plugins by versioning a folder in your repository. A `git clone` is all a teammate needs to get the same environment. The `/plugin` autocomplete now covers subcommands, installed plugin names, and plugins from known marketplaces.

---

## Notable improvements

**Richer `SessionStart` hooks (v2.1.152)** - hooks can now return `reloadSkills: true` to re-scan skill directories so that skills installed by the hook are available *within the same session*, without restarting. They can also set the session title via `hookSpecificOutput.sessionTitle`.

**`disallowed-tools` in skill frontmatter (v2.1.152)** - adding `disallowed-tools:` to the YAML frontmatter of a skill or slash command removes specific tools from the model while that skill is active. Useful for constraining a skill to, say, never invoke Bash.

**`/reload-skills` command (v2.1.152)** - re-scans skill directories without restarting the session. Handy during skill development.

**`/model` now saves as default (v2.1.153)** - pressing Enter in the model picker sets the model for future sessions (aligning with IDE extension behavior). Press `s` to switch models for the current session only. If you customized the `modelPicker:setAsDefault` keybinding, rename it to `modelPicker:thisSessionOnly` in `keybindings.json`.

**`/code-review --fix` now applies findings (v2.1.152)** - the command now runs a review *and* applies suggestions (reuse, simplification, efficiency) directly to your working tree. `/simplify` is now an alias for `/code-review --fix`.

**Auto mode requires no opt-in (v2.1.152)** - the `auto` mode no longer requires prior consent and is directly available from the mode picker.

**`agent` field in `settings.json` (v2.1.154/v2.1.157)** - a new top-level `agent` field in `settings.json` sets the default agent for sessions dispatched via `claude agents`. Override per-dispatch with `--agent <name>`.

**Lean system prompt is now the default (v2.1.154)** - for Opus 4.8 and future models, the lean system prompt is enabled by default (Haiku, Sonnet, and Opus 4.7 and earlier keep their existing behavior).

**MCP stdio subprocesses get `CLAUDE_CODE_SESSION_ID` (v2.1.154)** - stdio MCP server subprocesses now receive `CLAUDE_CODE_SESSION_ID` and `CLAUDECODE=1` in their environment, letting MCP servers identify which Claude Code session is calling them.

**Auto mode on Bedrock, Vertex, and Foundry (v2.1.158)** - available for Opus 4.7 and 4.8, opt in with `CLAUDE_CODE_ENABLE_AUTO_MODE=1`.

---

## Important bug fixes

**Fix: CLI hang in stream-json mode (v2.1.153)** - the CLI could fail to exit when stdin was closed without EOF in `--output-format stream-json` mode, leaving a stale session marker behind. Fixed.

**Fix: Agent tool writing to an undocumented temporary worktree (v2.1.153)** - `Agent` with `subagent_type: 'claude'` could silently discard outputs written to gitignored paths by using a temporary worktree. Fixed.

**Fix: stateful MCP servers reconnect-looping (v2.1.153)** - stateful MCP servers without the optional GET SSE stream were reconnecting in a loop on `tools/list` (regression in v2.1.147). Fixed.

**Fix: subagents ignored `--strict-mcp-config` (v2.1.153)** - subagents defined via frontmatter were ignoring managed MCP policies. Fixed; blocked MCP servers now surface a visible warning.

**Fix: orphaned worktrees after 30-day retention sweep (v2.1.157)** - background agent worktrees under `.claude/worktrees/` could be orphaned after the monthly cleanup. Fixed.

**Fix: re-attached background sessions sent wrong date (v2.1.157)** - after a sleep/wake cycle, re-attached sessions didn't tell the model the correct date. Fixed.

---

## Minor / cosmetic

- v2.1.159: internal infrastructure only, no user-facing changes.
- v2.1.156: stability fix for Opus 4.8 thinking blocks.
- v2.1.157: `EnterWorktree` can now switch between Claude-managed worktrees mid-session.
- v2.1.157: "Feature of the Week" claim status moved to a notification (no longer a line above the prompt).
- v2.1.157: "bash commands will be sandboxed" startup banner removed; sandbox status still visible in `/status`.
- v2.1.153: `claude doctor` now shows the result of your last update attempt.
- v2.1.153: Windows - if an update fails, Claude Code restores the previous executable and tells you how to recover.
- v2.1.152: `pluginSuggestionMarketplaces` - new managed setting for admins (Enterprise).
- v2.1.154: `←←` to open the agents view now works on Bedrock, Vertex, and Foundry.

---

**Going further** - This week touches on how much trust you place in a project's configuration (plugins are now auto-loaded from `.claude/skills`). Our related guide covers the reflex worth keeping: [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.md).

---

*Compiled from the official Anthropic changelog.*
*Interpretations and use-case suggestions are explicitly marked as such.*
