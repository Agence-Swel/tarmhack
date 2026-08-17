---
title: "Claude Code — What's new this week (v2.1.227 → v2.1.233)"
date: 2026-08-17
period: "Week W3 · v2.1.227 → v2.1.233"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: en
---

# Claude Code — What's new this week

> Period: week W3 (August 2026) · Versions v2.1.227 → v2.1.233 (6 releases published — v2.1.230 does not exist in the changelog)
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Generated on 2026-08-17

> **Stability status — everything here is still in preparation.** The npm dist-tags read `stable 2.1.224` / `latest 2.1.233`: all six releases below sit above the stable channel. If you follow stable, none of this has reached you yet — which is exactly what the gap between channels is for. The change of default on subagent forking, in particular, benefits from another week of maturing. Check `claude --version` and your channel before deploying.

---

## What actually matters

### Your subagents fork, and they work in the background — v2.1.232 (August 13, 2026)

Two switches in a single sentence, and they change default behaviour.

> *"Subagent forking is now on by default: a `subagent_type: \"fork\"` subagent inherits the full conversation and prompt cache, and non-teammate agent spawns in interactive sessions now run in the background by default"*

**What this means in practice**: a forked subagent now inherits the *entire* conversation and the prompt cache instead of starting from a blank page. That is concrete on both latency and billing — context you already paid for isn't paid for twice. The second switch is quieter: when you launch an agent from an interactive session, it goes to the background without you asking. Your main session no longer waits for it. That's comfortable on one condition — knowing what is running. Expect more sessions in your listings without having changed any of your habits. Get into the habit of `/tasks`.

---

### Trust is no longer inherited from a parent directory — v2.1.232 (August 13, 2026)

Approving a folder no longer opens everything inside it.

> *"Fixed nested git repositories inheriting trust from a parent directory; each repository now requires its own trust confirmation"*

**What this means in practice**: if you work with nested repositories — a monorepo with submodules, worktrees, vendored dependencies — you will see trust prompts you didn't see before. A little friction in exchange for a real guarantee. The thing to watch isn't interactive use, where you simply answer the prompt: it's **automation**. A pipeline or a script that runs Claude Code with no human in front of it **has no way to answer** a confirmation request, and what it gets will look like a silent failure. Test your automations before this release reaches stable, checking which repository they actually run in.

---

### Type `@` to talk to another session — v2.1.232 (August 13, 2026)

Your sessions stop being silos.

> *"Type `@` in the prompt to mention another Claude session by name; Claude then uses `SendMessage` to reach that session directly"*
>
> *"Interactive sessions on one machine now keep unique names: starting or renaming a session to a name another live session already uses gives it a `name-word-word` variant and tells you"*

**What this means in practice**: this is the move from "I have several sessions open" to "my sessions talk to each other." You can have one prepare context while another writes code, then bring them together without going through the clipboard. Unique names are not cosmetic: without them, messaging by name is unmanageable — and shipping both together suggests a feature built to last. Name your sessions explicitly from the start (`review`, `tests`, `docs`) rather than inheriting automatic variants, and go set the `/config` row "Messages from your other sessions" straight away: it takes accept, hold, or refuse. If you don't want it, refuse it — don't just live with it.

---

### Todo tools disappear on recent models — v2.1.233 (August 14, 2026)

This is a removal, not a bug.

> *"Todo/task-tracking tools (TaskCreate/Get/Update/List, TodoWrite) are no longer available on Opus 4.8, Sonnet 5, Fable 5, Mythos 5, and newer models; set `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` to bring them back"*

**What this means in practice**: if you have instruction files, skills, or agents that explicitly tell the model to keep a task list with `TodoWrite`, they now ask for a tool that no longer exists. The symptom will be quiet — the model does something else, nobody notices. Go re-read those files now, not once this is the default everywhere. `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` brings them back if you genuinely depend on them, but treat it as a reprieve: when a platform removes a tool from its default offering, the escape hatch is rarely permanent.

---

## Notable improvements

**Nine GitLab token families are now redacted (v2.1.232)** — secret redaction covers `glrt-`, `gloas-`, `glptt-`, `glagent-`, `glimt-`, `glsoat-`, `glcbt-`, `glft-`, `glffct-`, with full redaction of routable `glpat-`/`gldt-` tokens. The `glab` CLI config store gets the same sandbox and credential-path protection as `gh`. If you're on GitLab, this is the most useful line of the week.

**Plugin marketplaces speak GitLab (v2.1.232, v2.1.233)** — bare `gitlab.com` repo URLs, nested subgroups included, clone like `github.com` URLs; the `--worktree` flag and the `claude agents` view accept merge request URLs, displayed as `!N`; and the GitHub app setup tip no longer shows in repositories whose origin is on gitlab.com or bitbucket.org. Three lines, one direction: the tool stops being GitHub-centric.

**Skills synced from claude.ai are reined in (v2.1.228)** — they can no longer shadow a local command or an MCP prompt of the same name, their descriptions are sanitized and labeled, and their bodies no longer run `!` or expand `@`.

> *"Hardened skills synced from claude.ai: they no longer shadow local commands or MCP prompts, their descriptions are sanitized and labeled, and on your machine their bodies don't run `!` commands or expand `@` files"*

**Two new environment variables (v2.1.233)** — `CLAUDE_CODE_TOOL_MEMORY_LIMIT` enables, on Linux and opt-in, a memory cgroup on Bash tool commands *"so a runaway build can't stall the session"*. `CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS` sets the WebFetch URL cache TTL (default unchanged, 15 minutes).

---

## Important bug fixes

**A class of Windows path bypass is closed (v2.1.233)** — the most serious fix in the batch.

> *"Fixed Windows paths spelled with the NT `\??\` device prefix bypassing UNC path validation, closing an NTLM credential-leak vector"*

**What this means in practice**: a path spelled with the NT device prefix slipped through UNC validation, opening an NTLM credential-leak vector. If you're on Windows, this release is worth taking as soon as it reaches stable.

**Two permission bypasses closed, PowerShell and Git Bash (v2.1.232)** — on one side, variable-writing parameters that could silently overwrite `$PSDefaultParameterValues` and redirect later commands' file access; on the other, Cygwin-style symlinks that path validation saw as regular files. Writes through them now require approval.

**A round trip on Bash permissions under Windows (v2.1.232 then v2.1.233)** — v2.1.232 generalized checking of input redirections (`< file`) and tightened Cygwin symlink handling. v2.1.233 **reverts** both — *"a narrower version will return in a later release"* — and fixes a regression along the way where auto mode kept stopping for manual approval on perfectly ordinary `cd <dir> && <command> > file` commands. If you found auto mode unbearable on v2.1.232, that was it.

**Session cleanup no longer empties a project's `memory/` folder (v2.1.228)** — a terse fix with a serious consequence: a project's memory could be deleted by an automatic housekeeping cycle.

**Three crashes and three MCP irritants (v2.1.229, v2.1.231, v2.1.233)** — on crashes: a tool call with a non-string `glob`, `file_path`, or `command` value, a `RangeError` in a very narrow terminal (which could also kill `claude --continue` at startup), and on Windows an extended-length or UNC path referenced in a message. On MCP: the 30-second connect timeout ran out in full when a server answered the protocol probe badly, OAuth failed against strict authorization servers (`127.0.0.1` replaces `localhost`) and against pre-registered clients, and v2 connections no longer endlessly reopen the `subscriptions/listen` stream.

---

## Minor / cosmetic

- v2.1.233: a `[claude-code:unrecognized_model]` line is written to stderr in print mode when a request goes out for an unrecognized model ID; `modelOverrides` silences it.
- v2.1.233: `Notification` hooks weren't firing for permission prompts under Claude Desktop and VS Code — fixed. `claude plugin validate` now inspects a bare `.claude/skills` directory and reports `SKILL.md` files whose frontmatter fails to parse.
- v2.1.232: `additionalMarketplaces` and `allowedMarketplaces` become accepted aliases for `extraKnownMarketplaces` and `strictKnownMarketplaces`. `sandbox.ripgrep` is no longer honored from project settings — user, managed, and `--settings` only.
- v2.1.232: the cross-session messaging socket directory on shared `/tmp` now refuses a pre-planted symlink or another user's directory. Cowork sessions no longer inline external `@` imports from user-scope memory files.
- v2.1.229: `/commit-push-pr` no longer auto-approves git/gh commands carrying `--force`, `--amend`, `--no-verify`. The sandbox enforces ambiguous network domain spellings fail-closed, flagged by `/doctor`. A plugin marketplace can now be backed by a **local command**, re-resolved each session (`mode: "link"` uses it in place).
- v2.1.228: the Write tool can overwrite a file not read this session on recent models, matching Edit's rules; older models still require the read first.
- v2.1.227 through v2.1.232: interface comfort — a more readable slash-command menu, session groups in the VS Code sidebar, a resizable `/btw` panel, and Remote Control reconnecting for about 30 minutes after a network blip.

---

**Further reading** — This week is dense with trust boundaries: trust stops being inherited from a parent directory, two permission bypasses are closed on Windows, a third is reverted the day after it shipped, and `/commit-push-pr` stops auto-approving `--force`. The durable lesson isn't "there were holes" — it's that an approval filter which parses shell is fragile by construction, and that a narrow allowlist beats a denylist. The full reasoning, with a free checklist: our guide [Harden Claude Code permissions](../resources/guides/harden-claude-code-permissions.md).

---

*Watch generated from Anthropic's official changelog.*
*Interpretations and use cases are explicitly labeled as such.*
