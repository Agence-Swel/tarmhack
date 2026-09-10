---
title: "Claude Code: what's new this week (v2.1.234 → v2.1.241)"
date: 2026-09-10
period: "Week W4 · v2.1.234 → v2.1.241"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: en
---

# Claude Code: what's new this week

> Period: week W4 (August 2026) · Versions v2.1.234 → v2.1.241 (8 releases, published August 17 to 22, 2026)
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog) · [npm dist-tags](https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags)
> Generated on 2026-09-10

> **Stability status, checked on September 10, 2026.** The npm dist-tags read `stable 2.1.236` / `latest 2.1.267`. Three of the eight releases described here have therefore reached the stable channel: v2.1.234, v2.1.235 and v2.1.236. The other five, v2.1.237 through v2.1.241, still sit above stable and can still change. Two of them, v2.1.240 and v2.1.241, carry a single changelog line: "Bug fixes and reliability improvements". This page covers what is documented. It does not claim to cover what shipped. Check `claude --version` and your channel before deploying.

---

## What actually matters

### Your sessions finally talk to each other on Windows, v2.1.239 (August 21, 2026)

Cross-session messaging already existed on macOS and Linux. It lands on Windows, along with the small line without which it stayed unusable.

> *"Windows: cross-session messaging is now available, so Claude Code sessions across your machines can message each other with `SendMessage` and find each other with `ListAgents`, as on macOS and Linux"*
>
> *"`ListAgents` now tells a session its own name (the one peers use to message it), and `SendMessage` to your own name says so instead of \"no agent named …\""*

**What this means in practice**: until now, running several sessions meant running several loners. Each worked in its own corner, and you were the link between them: copying a result from one terminal into another, remembering who was waiting on what. A session now knows its own name, knows who else is alive, and can write to a peer. Start small: two sessions, a long task in the first, a `SendMessage` to the second when it finishes. You will find out quickly whether your way of working gains anything from agents coordinating, or whether you would rather stay the dispatcher. Both answers are fine. What changed is that the question is now on the table.

**Note**: the companion piece, shipped three days earlier, did not follow onto Windows.

> *"Added `notify_when_idle` to cross-session `SendMessage`: ask another Claude Code session on this machine to send one notice when it next goes idle — opt-in, one-shot, no polling (macOS and Linux)"* (v2.1.236)

The original line carries the restriction itself, "(macOS and Linux)", and no entry in this window extends it to Windows. On the changelog alone: on Windows you get the messaging, the idle wake-up is not announced there.

---

### A default model that doesn't take away anyone's choice, v2.1.236 (August 19, 2026)

One more environment variable, and a difference that reads small but is paid every day.

> *"Added `ANTHROPIC_DEFAULT_MODEL` environment variable: sets the model new sessions start on, while a `/model` pick still overrides it and persists across restarts (unlike `ANTHROPIC_MODEL`)"*

**What this means in practice**: the old variable, `ANTHROPIC_MODEL`, pinned the model. This one sets a starting point. That is precisely the knob a team needed: you standardise what new sessions boot on, and everyone keeps the right to switch for a specific task, without that choice being wiped on the next launch. If you administer machines, this is the variable to set, not the other one. If you work alone, it saves you a `/model` on every new project.

---

### Context cost becomes an engineering problem, Anthropic included, v2.1.234 (August 17, 2026)

Anthropic cut the loading cost of its own skill by a factor of eight, and said exactly how.

> *"Reduced the context cost of loading the built-in `claude-api` skill from ~200k+ tokens to ~25k by loading reference docs on demand"*

**What this means in practice**: this is the clearest demonstration we have read of what a good skill looks like. A 200,000-token skill is not a rich skill, it is documentation pasted into the context that loads in full even when you need one sentence of it. Dropping to 25,000 tokens removes nothing: the reference docs are still there, they open when needed. If your own skill runs to tens of thousands of tokens, the question is not "what do I cut?" but "what didn't need to be there at load time?". Write your `SKILL.md` as an index, not a manual: it says what to do, when, and where the detail lives. The detail sits in neighbouring files the model opens itself.

---

### Folder trust becomes a prerequisite, even in non-interactive mode, v2.1.238 (August 20, 2026)

A project `.mcp.json` can cause a command to run. Cloning a repo is no longer enough to trigger it.

> *"MCP `headersHelper` in a project `.mcp.json`, and inline MCP servers in project or `--add-dir` agent files, now require that folder's trust dialog to have been accepted (also under `claude -p`)"*
>
> *"`claude plugin install/update` ask `[y/N]` (or pass `-y`)"*

**What this means in practice**: the reasoning is sound. A `headersHelper` mints HTTP headers, typically a short-lived token, by running a command on your machine. Making that conditional on explicit trust is the right call. The part to internalise is the "also under `claude -p`": non-interactive mode no longer bypasses trust. The consequence is direct and quiet. Your continuous integration scripts that run `claude -p` against repos nobody ever opened by hand will start with fewer MCP servers than you think, with no loud error. Check explicitly what loaded instead of assuming. And in a pipeline, the `-y` on `claude plugin install` is no longer a nicety: without it, the command waits on an answer nobody will give.

---

## Notable improvements

**A built-in "Concise" output style (v2.1.237)**: *"Added a built-in \"Concise\" output style: Claude leads with results and skips preamble and narration, while doing the work just as thoroughly."* Enable it under "Output style" in `/config`. If you have already written three lines of CLAUDE.md asking Claude to stop narrating what it is about to do, you can delete them.

**Keybindings can behave like Bash (v2.1.238, v2.1.239)**: the `keybindingFlavor: "readline"` key makes `Ctrl+W` delete back to the previous whitespace, as your shell does. v2.1.239 extends the treatment to `Alt+F`, `Ctrl/Option+→`, `Alt+D` and `Ctrl+Y`, and makes punctuation a word boundary. The default, `"classic"`, is unchanged. Small setting, real comfort if your fingers come from the terminal.

**Read-deny rules can no longer be dodged by renaming (v2.1.236, macOS)**: *"wildcard read-deny rules (e.g. `**/.env`) now take precedence inside allowed read regions, cover matched directories' contents, and can't be bypassed by renaming the denied file"*. If you relied on `**/.env` to keep secrets out of reach, the rule now does what you thought it did.

**`/goal` stops nagging you every half hour (v2.1.236, v2.1.239)**: check-ins on long-running background work back off to 30 minutes, then 1 hour, then every 2 hours. Worth noting, a related but earlier fix: since v2.1.234 a goal *"now clears itself with a notice when a turn dies on an unrecoverable error"*, instead of staying armed over nothing.

**A spell checker in the prompt (v2.1.235)**: the `spellcheck` key underlines misspellings using whichever of `aspell`, `hunspell` or `ispell` you already have installed. Optional, off by default.

**GitLab repos finally get a badge (v2.1.234)**: a repo with a GitLab remote and an authenticated `glab` shows `MR !N` in the footer and status line, with draft, pending and green states. v2.1.233 had already opened `--worktree` to merge request URLs.

---

## Important bug fixes

**A proxy could double your API bill, silently (v2.1.239)**: the most expensive fix in this window.

> *"Fixed Bedrock streaming behind proxies that strip the response Content-Type header, which silently doubled billed API calls by re-running every turn non-streaming"*

**What this means in practice**: behind a corporate proxy that strips the `Content-Type` header, every turn was re-run non-streaming, and therefore billed twice. If you are on Bedrock and your costs looked high with no explanation, you may have just found it.

**A file starting with a UTF-8 BOM is no longer silently ignored (v2.1.239)**: *"Fixed agents, skills, and commands whose `.md` file starts with a UTF-8 BOM being silently ignored"*. The BOM is that invisible marker several Windows editors prepend to a file. A perfectly written skill could simply not exist, with no error message.

**Similarly named projects no longer share their sessions (v2.1.239)**: *"Fixed `claude -c`/resume picking up sessions from a different directory whose path differed only by characters like `_`, `-`, or `.`"*. `my_app` and `my-app` could land on the same history. If you once resumed a session that was talking about a different project, it wasn't you.

**Long sessions no longer grow in memory without bound (v2.1.238)**: *"Fixed unbounded memory growth in long interactive sessions: subagent tool results are now released once they leave the recent display window"*. Directly relevant to anyone who leaves a session open all day.

**OpenTelemetry traces no longer split in two on a hook (v2.1.239)**: a tool execution deferred by a `PreToolUse` hook resumes in the original turn's trace instead of opening a new one. If you measure Claude Code through OTEL and you use hooks, treat your earlier numbers with caution.

**Hooks no longer crash when the working directory is gone (v2.1.239)**: instead of a `posix_spawn ENOENT`, they run from the project root or your home directory. Same release, same family: the Linux sandbox no longer makes a nonexistent `.git/config.worktree` unreadable, which had been breaking every sandboxed git command in repos configured with `extensions.worktreeConfig`.

**Closing the NTLM vector continues (v2.1.234)**: remote file reads, session restore, CLAUDE.md includes, workflow scripts and file uploads now reject Windows NT-namespace (`\??\`) paths. This is the direct follow-up to last week's fix, applied this time to the file accesses that happen before any approval.

---

## Minor / cosmetic

- v2.1.234: `/permissions` and `/add-dir` can now be opened while Claude is working, and a rule change applies to the rest of the current turn.
- v2.1.234: Claude Code continues your session automatically when a claude.ai usage limit resets, switchable off in `/config`. MCP diagnostics no longer print resolved secrets: warnings show the configured `${VAR}` form.
- v2.1.234: the "Default teammate model" setting is gone from `/config`; teammates use the leader's model unless the spawn names one.
- v2.1.236: a mistyped slash command is reported instead of running the nearest fuzzy match. `/usage` shows the usage-credits row for Team and Enterprise members.
- v2.1.238: `Ctrl+L` and `Cmd+K` in fullscreen now only repaint, the double-press `/clear` shortcut is gone. `claude mcp list` and `claude mcp get` show disabled servers as `⊘ Disabled` instead of probing them. `claude self-hosted-runner` gains `--defer-shutdown-max-min` and two proxy-authorization options.
- v2.1.239: `/cost`, the status line and `--max-budget-usd` include the 1.1× US-only-inference premium for data-residency workspaces. On Alpine and musl builds, native image paste, clipboard and audio capture finally load.
- v2.1.236 to v2.1.239: the fullscreen renderer is offered on Bedrock, Vertex and Foundry, it falls back to the classic renderer after a failed start instead of exiting on every launch, and the prompt stops after three showings.
- v2.1.234 through v2.1.239: comfort and display, with vim mode, tmux and iTerm2, the VS Code screen reader, footer alignment, middle-truncation of long paths, diff wrapping with emoji, and the Clawd mascot's eyes.


---

**Further reading**: the durable fact in this window is a quiet reversal. Until now, `claude -p` never showed a trust dialog, and plenty of people concluded that non-interactive mode simply skipped the question. v2.1.238 closes that door: a `headersHelper` declared in `.mcp.json`, like inline MCP servers, now requires an approved folder even when nobody is watching the screen. A pipeline that ran yesterday can go quiet today, and a cloned repo does not become harmless just because the run is automated. What an approval actually covers, what runs before it, and a free manual checklist: our guide [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.en.md).

---

*Watch generated from Anthropic's official changelog.*
*Interpretations and use cases are explicitly labeled as such.*
