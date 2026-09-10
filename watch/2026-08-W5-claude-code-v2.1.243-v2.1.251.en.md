---
title: "Claude Code: what's new this week (v2.1.243 → v2.1.251)"
date: 2026-08-31
period: "Week W5 · v2.1.243 → v2.1.251"
sources:
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://code.claude.com/docs/en/changelog
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: en
---

# Claude Code: what's new this week

> Period: week W5 (August 2026) · Versions v2.1.243 → v2.1.251, published August 24 to 28, 2026 (7 releases on npm)
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog) · [npm dist-tags](https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags)
> Generated on 2026-08-31

> **Stability status, checked on September 10, 2026: everything below is still in preparation.** The npm dist-tags read `stable 2.1.236` / `latest 2.1.267`. The lowest version covered here, v2.1.243, already sits above the stable channel: if you follow stable, none of this has reached you yet. Check `claude --version` and your channel before you deploy any of it.
>
> **Two gaps in the numbering, and one silent release.** v2.1.244 and v2.1.249 exist neither in the changelog nor on npm. v2.1.242, just below this range, exists on npm but has no public changelog entry at all. And v2.1.250 carries a single line, "Bug fixes and reliability improvements". This page covers what is documented, not what is published: the distinction matters the day you are trying to explain a behaviour change between two versions.

---

## What actually matters

### A restricted mode, in a single flag, v2.1.248 (August 27, 2026)

Four guarantees at once, without hand-assembling a single permission rule.

> *"Added `--restricted` (or `CLAUDE_CODE_RESTRICTED=1`): removes the built-in tools that run commands or code and `WebFetch` (unless named in `--tools`), keeps file tools inside the working directory, refuses `bypassPermissions`, and ignores user, project and local settings files"*

**What this means in practice**: no command or code execution, no web fetching, file tools confined to the working directory, permission bypass refused. The most interesting clause is the last one: user, project and local settings files are not read at all. Until now, locking Claude Code down meant stacking permission rules by hand and hoping you had not missed one. This mode does not harden the rules, it cuts off the source of the rules: nobody can re-enable anything from a file committed to the repo. The obvious use is reading code you do not know, an outside contributor's PR or a freshly cloned repository, without giving it anything to execute. Run `claude --restricted` and ask your questions.

---

### Two hooks to govern model switches, v2.1.251 (August 28, 2026)

A team rule stops being a document and becomes a refusal.

> *"Added `PreModelSwitch` and `PostModelSwitch` hook events (block, confirm, or annotate a model switch); `SessionStart` resume hooks now receive session staleness and the estimated re-cache cost"*

**What this means in practice**: you can now intercept a model switch, block it, ask for confirmation, or simply log it. If your team has a rule about which models are allowed, for cost, for compliance or for reproducibility, that rule used to live in a page nobody rereads. It can now live in a hook that says no. The second half of the sentence is quieter and just as useful: resume-time `SessionStart` hooks receive the session's staleness and the estimated cost of rebuilding the cache. So a script can decide, without you, whether resuming an old session is worth the price or whether starting fresh is cheaper.

---

### Paths tighten up, four times in one release, v2.1.251 (August 28, 2026)

Four fixes from the same family, and one culprit: the symbolic link.

> *"Fixed file tools (Read, Write, Edit) following a symlink swapped inside the working directory after the permission check, which could read or write outside the approved location"*
>
> *"Fixed Grep and Glob not applying `Read(...)` deny rules to files reached through a symlinked search path"*

**What this means in practice**: a link can be swapped between the moment permission is granted and the moment the file is actually opened. So your `deny` rules were bypassable by something as ordinary as a link sitting in your own working directory. Two more fixes in the same release point the same way: the Workflow tool was reading a `scriptPath` outside what the session is allowed to read, before the check ran, and commands declared by a marketplace entry could point outside their plugin's directory. If you rely on `Read(...)` deny rules to protect sensitive files, take this version. It is the kind of fix nobody notices, and it changes what your rules were actually guaranteeing.

---

### A repo file can no longer redirect your configuration, v2.1.251 (August 28, 2026)

A trust boundary that had slipped past everyone.

> *"Changed project-level `.claude/settings.json` `env` to no longer set `CLAUDE_CONFIG_DIR`, `CLAUDE_CODE_TMPDIR`, or `TMPDIR`/`TMP`/`TEMP`; set them in your shell, user, or managed settings instead"*

**What this means in practice**: `.claude/settings.json` lives in the repo, so anyone with merge rights could, in principle, move your config directory or your temp directory. Those three variables determine too much to be left to a project file. If you were setting `CLAUDE_CONFIG_DIR` that way, move it into your shell or your user settings: it still works, just not from the repo. Worth noting, the same release closes a neighbouring door: project settings could turn on detailed beta tracing or raw API body logging.

---

## Notable improvements

**Install size and memory, meaningfully down (v2.1.243, v2.1.251)** The native binary is now zstd-compressed, worth *"about 75 MB instead of 340 MB on Linux x64"*, and code is loaded on demand rather than kept fully resident, worth *"roughly 40–70 MB less memory per session"*. Add a binary about 5 MB smaller and six rarely used syntax-highlighting languages removed. If you run several sessions side by side, this is the most tangible change in the batch.

**A prompt cache that lasts an hour (v2.1.243, v2.1.248)** Two new settings, `promptCacheTtl` and `subagentPromptCacheTtl`, *"so API-key and cloud-provider users can keep a 1-hour prompt cache on the main conversation while subagents stay at 5 minutes"*. The qualifier matters: this is for API-key and cloud-provider access. v2.1.248 goes one level lower, with `experimental.cacheTtl` (`"5m"` or `"1h"`) set in an agent's frontmatter. On a long, context-heavy session, that is exactly where the bill is decided.

**Knowing which settings actually apply (v2.1.243, v2.1.248)** `/status` now lists managed settings sources *"present but not applied because a higher-precedence managed source is active"*, and `/doctor` explains why a load failed. If you have ever spent twenty minutes wondering why a setting "isn't taking", you know precisely why this line exists.

**A warning on a deceptive class of Bash rule (v2.1.246)** A startup warning flags Bash allow rules where the wildcard precedes the subcommand, such as `Bash(git * main)`, *"since they also match options inserted before the subcommand"*. Meaning: your rule is broader than you thought. Go reread your Bash rules, this one gets written by accident.

**`/permissions` gets an Auto mode tab (v2.1.246)** for viewing and editing the auto mode classifier rules. Auto mode stops being a black box.

**Subagents get a default model, not an imposed one (v2.1.251)** `CLAUDE_CODE_SUBAGENT_MODEL` now serves *"to set the default subagent model rather than override everything"*: an agent definition's `model:`, and a model passed explicitly at spawn time, take precedence. It is the ordering you would expect, and it is finally the one that applies.

---

## Important bug fixes

**A startup crash on glibc 2.44 (v2.1.245)** The only line in this release, and it counts.

> *"Fixed a crash on startup on Linux distributions that ship glibc 2.44 (for example Arch Linux, CachyOS and Fedora Rawhide)"*

**What this means in practice**: if Claude Code refuses to start on Arch, CachyOS or Fedora Rawhide, this is the release you need, v2.1.245 at the very least. Fixed.

**Session transcripts silently overwritten (v2.1.251)** A directory change could relocate a session onto an existing transcript with the same ID, which was then overwritten without a word. Your conversation history vanished with no error message. Fixed.

**A prompt cache miss roughly once an hour (v2.1.248)** In long sessions, an OAuth token refresh caused tool definitions to be re-rendered, which meant a full cache miss and the loss of extended-thinking context. That is cost and latency you were paying without knowing. Fixed.

**Claude Desktop and Cowork sessions disappearing after 30 days (v2.1.248)** Transcript cleanup now keeps desktop-written sessions while they are still in the app, with a new `desktopSessionCleanupPeriodDays` setting to cap that exemption. Fixed.

**Git worktrees deleted by mistake (v2.1.246)** The background-session retention sweep could remove worktrees you had created yourself under `.claude/worktrees/`, whenever an old session record pointed at them. If you work across several worktrees, this is the fix not to miss. Fixed.

**Bash commands wrongly auto-approved (v2.1.251)** Permission checks were approving, with no prompt at all, commands that assign an arithmetic expression to an integer shell variable, such as `OPTIND=1/0` or `RANDOM=2+2`. They now ask for approval. Fixed.

**Credentials leaving with cloud sessions (v2.1.246, v2.1.248)** Two fixes of the same shape: telemetry requests to Anthropic were carrying the API key configured for a third-party gateway, *"a credential is now only sent to its own host"*; and `/ultrareview` was uploading uncommitted edits to `prod.env`-style and `*.tfvars` files, along with editor swap, temp and backup copies of credential files such as `key.pem.tmp` or `id_rsa.swo`. Those now stay on your machine. Fixed.

---

## Minor / cosmetic

- v2.1.246: fullscreen mode, blank transcript after a resize, erratic scrolling and keyboard focus stolen by the pointer.
- v2.1.246: severe transcript slowdown when a diff contained a very long single line, such as a base64 string. It now renders truncated with a marker.
- v2.1.246: markdown rendering wrongly disabled for a whole message when its first 500 characters contained none.
- v2.1.246: `/stats` activity heatmap off by one cell in timezones east of UTC; `/fork` from an already-forked or backgrounded session started with an empty conversation.
- v2.1.246: custom theme diff colors were ignored, and `/rename` overwrote the theme's prompt border color.
- v2.1.243: `/resume` only listed the 50 most recent sessions, the picker now loads more as you scroll; history search and up-arrow broke on a malformed entry in `~/.claude/history.jsonl`.
- v2.1.243, v2.1.247: new settings, `modelPicker` to curate the model picker list, `modelPricing` for an organization's contracted rates, `spinnerTipsOverride` and `feedbackDrafts`.
- v2.1.247: new `SendFeedback` tool, Claude can draft a report for you to review before sending it from `/feedback`. On Sonnet 5, auto-compact now uses the full 1M window, about 967K tokens instead of 934K.
- v2.1.243, v2.1.251: `/usage` gains a per-Loop breakdown and a spend limit bar; `/cost` gains a per-session prompt-cache line.
- v2.1.248: the Workflow tool's description drops from about 5.7k to 1k tokens, with the script-writing reference moved into a dedicated skill.
- v2.1.251: five commands join `claude --help`, `attach`, `logs`, `stop`, `respawn` and `rm`.


---

**Further reading**: four path containment fixes in a single release, and they all tell the same story. A `deny` rule a symlink walks around, a path whose parentheses make the rule fall silent, a command auto-approved because it assigns an arithmetic expression. Command approval is a net, not a wall, and the answer is not to write cleverer patterns: it is to start from denial, then verify what actually applies. How to set that floor, and why a rule you have never seen match protects nobody: our guide [Harden your own Claude Code permissions](../resources/guides/harden-claude-code-permissions.md).

---

*Watch generated from Anthropic's official changelog.*
*Interpretations and use cases are explicitly labeled as such.*
