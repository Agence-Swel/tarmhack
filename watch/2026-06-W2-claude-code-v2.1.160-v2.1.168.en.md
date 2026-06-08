---
title: "Claude Code — What's new this week (v2.1.160 → v2.1.168)"
date: 2026-06-08
period: "Week W2 · v2.1.160 → v2.1.168"
sources:
  - https://docs.claude.com/en/docs/claude-code/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: en
---

# Claude Code — What's new this week

> Period: Week W2 (2nd week of June) · June 2–6, 2026 · Versions v2.1.160 → v2.1.168 (8 releases)
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [docs.claude.com](https://docs.claude.com/en/docs/claude-code/changelog)
> Generated on 2026-06-08

---

## What actually matters

### A configurable fallback model, up to three tiers deep — v2.1.166 (June 6, 2026)

Claude Code adds a `fallbackModel` setting that configures up to three fallback models, tried in order when the primary model is overloaded or unavailable. The `--fallback-model` flag now applies to interactive sessions too, not just non-interactive runs.

> *"configure up to three fallback models tried in order when the primary model is overloaded or unavailable"*

**What this means in practice**: no more sessions dying outright when the primary model is saturated — you define a chain of backups and the work keeps going. In practice: add `fallbackModel` to your `settings.json`, or launch with `--fallback-model` even in an interactive session.

---

### Knowing what a session is blocked on — v2.1.162 (June 3, 2026)

`claude agents --json` now exposes a `waitingFor` field that shows what a waiting session is blocked on (for example, a permission prompt).

> *"`claude agents --json` now includes `waitingFor` showing what a waiting session is blocked on (e.g. permission prompt)"*

**What this means in practice**: when you're running several background agents, a session that's "waiting" with no explanation forces you to go check manually. With `waitingFor`, you immediately see which one is waiting on a permission and which one is waiting on something else. Use case: parse the `claude agents --json` output to surface the blocking reason in a dashboard.

---

### Version governance for teams — v2.1.163 (June 4, 2026)

Two new managed settings appear: `requiredMinimumVersion` and `requiredMaximumVersion`. Claude Code refuses to start if its version is outside the allowed range and directs the user to an approved version.

> *"Added `requiredMinimumVersion` and `requiredMaximumVersion` managed settings — Claude Code refuses to start if its version is outside the allowed range"*

**What this means in practice**: organizations can enforce a window of sanctioned versions — handy to keep a machine from running an outdated or unvetted build. Set these in your organization's managed settings.

---

## Notable improvements

**Glob-based tool denial (v2.1.166)** — deny rules now accept a glob in the tool-name position: `"*"` denies all tools. Allow rules reject non-MCP globs, and an unknown tool name in a deny rule warns at startup. A compact syntax for a "deny by default" posture.

**`/plugin list` (v2.1.163)** — a new command to list installed plugins, with `--enabled` / `--disabled` filters. Handy for taking stock of what's actually active.

**More flexible `Stop` and `SubagentStop` hooks (v2.1.163)** — these hooks can now return `hookSpecificOutput.additionalContext` to give Claude feedback and keep the turn going without being labeled a hook error. You can inject context without interrupting the work.

**More robust parallel tool calls (v2.1.161)** — a failed Bash command no longer cancels other calls in the same batch; each tool returns its own result independently.

**Safer sensitive writes (v2.1.160)** — Claude Code now prompts before writing to shell startup files (`.zshenv`, `.zlogin`, `.bash_login`) and `~/.config/git/`. The `acceptEdits` mode also prompts before writing build-tool config files that grant code execution (`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/`).

---

## Important bug fixes

**Windows permissions (v2.1.162)** — fixed permission rules that never matched when spelled with backslashes (`~\`, `\\server\share`) or case-variant paths; Read deny rules now also hide files from Glob/Grep results.

**Home paths and `$HOME` (v2.1.163)** — deny rules on home-directory paths (e.g. `Read(~/Desktop/**)`) now block Bash commands that reference the path via `$HOME`.

**`if: "Bash(...)"` hook condition (v2.1.163)** — this condition no longer fires on every Bash command containing `$()` or `$VAR`; the pattern now matches commands inside subshells and backticks too.

**`$TMPDIR` regression (v2.1.163)** — fixed Bash commands failing under bazel and EDR-protected Go workflows: `$TMPDIR` was overridden for all commands instead of only sandboxed ones (a regression in v2.1.154). Fixed.

**WebFetch rule precedence (v2.1.162)** — explicit `WebFetch(domain:...)` permission rules (deny/ask/allow) now take precedence over the preapproved-host auto-allow.

---

## Minor / cosmetic

- v2.1.166: hardened cross-session messaging — messages relayed via `SendMessage` from other sessions no longer carry user authority (receivers refuse relayed permission requests).
- v2.1.166: `MAX_THINKING_TOKENS=0`, `--thinking disabled`, and the per-model thinking toggle now disable thinking on models that think by default (third-party providers unchanged).
- v2.1.166: `claude update` announces the target version before downloading; fixes for JetBrains (2026.1+ flicker), Kitty (Shift+non-ASCII), and PowerShell command validation hanging on Windows.
- v2.1.162: MCP per-server timeouts below 1000 ms are now ignored (falling back to `MCP_TOOL_TIMEOUT` or the default); many `claude agents` display improvements (column width, name truncation).
- v2.1.161: `OTEL_RESOURCE_ATTRIBUTES` added as labels on metrics; `claude mcp` list/get/add no longer leaks secrets (`${VAR}` not expanded, headers redacted).
- v2.1.160: the dynamic-workflow trigger keyword is renamed from `workflow` to `ultracode`; background session teardown sends SIGTERM before SIGKILL; removed the `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` variable (now a no-op).
- v2.1.165 / v2.1.167 / v2.1.168: bug fixes and reliability improvements.

---

**Going further** — This week tightens Claude Code's permission governance: a `"*"` glob `deny` to block everything, and a prompt before writing to files that grant code execution. How to turn it into a durable posture (deny-by-default, sensitive writes, a version window): our guide [Harden your own Claude Code permissions: go deny-by-default](../resources/guides/harden-claude-code-permissions.md).

---

*Compiled from the official Anthropic changelog.*
*Interpretations and use-case suggestions are explicitly marked as such.*
