---
title: "Claude Code - What changes this week (v2.1.178 → v2.1.195)"
date: 2026-06-29
period: "Week W5 · v2.1.178 → v2.1.195"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: en
---

# Claude Code - What changes this week

> Period: week W5 · June 15-26, 2026 · Versions v2.1.178 → v2.1.195 (11 releases)
> Sources: [code.claude.com](https://code.claude.com/docs/en/changelog) · [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
> Generated on 2026-06-29

---

## What really matters

### Deny an action by its parameters, not just by the tool - v2.1.178

Until now, a permission rule said "allow or block this tool." Now it can target a specific parameter: a model, an argument, a value.

> *"Added `Tool(param:value)` syntax for permission rules to match a tool's input parameters (with `*` wildcard), e.g. `Agent(model:opus)` to block Opus subagents"*

**What this means in practice**: `Agent(model:opus)` stops any subagent from running on Opus even while the Agent tool stays allowed. It's finer-grained, and it pays off the moment you let agents spawn each other. To use it, add the rule to your `permissions` (allow / deny / ask) like any other pattern. And v2.1.186 fixes a related gap - `Agent(type)` and `Agent(x,y)` rules were being ignored for named subagent spawns; they're now enforced.

---

### Cut sandboxed commands off from your secrets - v2.1.187

If you run commands in the sandbox, a new setting stops them from reading your credential files and sensitive environment variables.

> *"Added `sandbox.credentials` setting to block sandboxed commands from reading credential files and secret environment variables"*

**What this means in practice**: it's the setting you forget to turn on until the day you wish you had. Turn it on by default; loosen it case by case if a legitimate command genuinely needs a secret.

---

### A telemetry side effect to watch before you upgrade - v2.1.193

If you were already logging prompts through OpenTelemetry, the upgrade also starts exporting the model's response text - with no config change on your side.

> *"Added `claude_code.assistant_response` OpenTelemetry log event… when that var is unset it follows `OTEL_LOG_USER_PROMPTS`, so deployments that already log prompt content will start receiving response content on upgrade — set `OTEL_LOG_ASSISTANT_RESPONSES=0` to keep prompts-only"*

**What this means in practice**: it's not a bug, it's an inherited default. If exporting responses isn't what you want, set `OTEL_LOG_ASSISTANT_RESPONSES=0` before upgrading. Check it.

---

## Notable improvements

**Auto mode is more cautious by default (v2.1.183)** - it blocks destructive commands you didn't ask for: `git reset --hard`, `git checkout -- .`, `git clean -fd`, `git stash drop`, and `terraform destroy` / `pulumi destroy` / `cdk destroy` unless you named the specific stack. `git commit --amend` is blocked when the commit wasn't made by the agent this session. The same release warns you when a requested model is deprecated or auto-redirected.

**You now see why an action was denied (v2.1.193)** - auto-mode denial reasons are added to the transcript, the denial toast, and the "recently denied" tab in `/permissions`. No more silent refusals.

**Configure in one line (v2.1.181)** - `/config key=value` sets any setting from the prompt (e.g. `/config thinking=false`), in interactive, `-p`, and Remote Control. v2.1.183 adds `/config --help` and changes toggle behavior (Enter and Space both apply, Esc saves and closes).

---

## Important fixes

**Hook matchers finally behave the way you read them (v2.1.195 / v2.1.191)** - two fixes worth knowing if you use hooks. v2.1.195: hyphenated identifiers (`code-reviewer`, `mcp__brave-search`) no longer substring-match - they now match exactly. To cover every tool from a hyphenated MCP server, write `mcp__brave-search__.*`. If one of your hooks relied on the old behavior, it may have stopped firing: re-read your matchers. v2.1.191: comma-separated matchers (`"Bash,PowerShell"`) silently never fired - now fixed.

**Stopping a background agent is now permanent (v2.1.191 / v2.1.195)** - an agent stopped from the tasks panel no longer "resurrects" (v2.1.191). v2.1.195 also fixes background jobs that could disappear or lose data when written by a newer Claude Code version - handy if you juggle multiple installed versions.

**And a wave of stability** - partial responses preserved on mid-stream connection drops (v2.1.179), 0-byte files on network/cloud drives fixed (v2.1.181), garbled CJK/Korean paste fixed (v2.1.187/181), voice dictation on macOS and space-less languages (v2.1.195), and streaming CPU cut by roughly 37% (v2.1.191).

---

## In brief (minor / cosmetic)

- v2.1.181: bundled Bun runtime upgraded to 1.4; long paragraphs now stream line by line.
- v2.1.191: `/rewind` can resume a conversation from before a `/clear`.
- v2.1.186: `claude mcp login <name>` / `claude mcp logout <name>` to authenticate an MCP server without opening `/mcp` (`--no-browser` for SSH).
- v2.1.187: mouse support in fullscreen select menus; org-configured model restrictions shown in the model picker.
- v2.1.181: `CLAUDE_CLIENT_PRESENCE_FILE` suppresses mobile push notifications while you're at the machine.
- v2.1.190: "bug fixes and reliability improvements" (no detail).

---

**Going further** - This week moves permissions from the tool level to the parameter level (`Agent(model:opus)`, v2.1.178) and adds `sandbox.credentials` (v2.1.187) to cut sandboxed commands off from your secrets. How to turn that into a durable least-privilege posture (deny-by-default, sensitive writes, a version window, a model allowlist): our guide [Harden your own Claude Code permissions: go deny-by-default](../resources/guides/harden-claude-code-permissions.md).

---

*Compiled from the official Anthropic changelog.*
*Interpretations and use-case suggestions are explicitly marked as such.*
