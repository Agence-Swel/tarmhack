---
title: "Claude Code: what's new this week (v2.1.252 → v2.1.267)"
date: 2026-09-10
period: "Week W2 · v2.1.252 → v2.1.267"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: en
---

# Claude Code: what's new this week

> Period: week W2 (September 2026) · Published August 31 to September 9, 2026 · Versions v2.1.252 → v2.1.267 (16 numbers in the range, only 10 carry a public entry: 253, 254, 255, 256, 262 and 264 have none. You only learn v2.1.255 existed from the v2.1.258 fix that names it as the release which broke launching on macOS 12)
> Sources: [CHANGELOG.md on GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog) · [npm dist-tags](https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags)
> Generated on 2026-09-10

> **Stability status: every item in this edition is still in preparation.** The npm dist-tags read `stable 2.1.236` / `latest 2.1.267`. The stable channel has not moved since August 19, and that is the most useful fact of the week in itself: twenty-three releases have shipped above it since then, ten of which this edition covers. If you follow stable, none of this has reached you. Three weeks of gap is worth a check on where you stand: run `claude --version`, confirm your channel, then decide what to deploy.

---

## What actually matters

### Fable moves up a generation, without you doing anything, v2.1.257 (September 1, 2026)

This isn't one more model in the picker: the one that answers by default has changed.

> *"Added Claude Fable 5.1 (`claude-fable-5-1`), now the default Fable model — 1M context, $10/$50 per Mtok with $0.25/Mtok cache reads"*

**What this means in practice**: a workflow that asks for "Fable" without further detail now runs on 5.1, with a one-million-token context window, list pricing of $10/$50 per million tokens and cache reads at $0.25. Anything pinning `claude-fable-5` by its exact name stays where it is. Two caveats matter more than the announcement.

> *"Changed `fable` and `best` in Claude apps gateway sessions to keep resolving to Fable 5 for now, since gateways not yet configured for Fable 5.1 reject it; pick Fable 5.1 in `/model` to use it"*

> *"Fixed prompt caching on Claude Fable 5.1 not covering the context attached after tool results, so it was re-sent as uncached input on every tool-call turn"* (v2.1.260, September 3, 2026)

**Note**: "default" is therefore not universal, since sessions going through a Claude apps gateway stay on Fable 5. And for the model's first two days, context attached after a tool result escaped the cache and went back out as full-price input on every turn. If you switched on September 1, the September 1 to 3 line on your bill deserves a look.

---

### Organizations can now push MCP servers to everyone, v2.1.259 (September 2, 2026)

Until now, an MCP server was configured project by project, or machine by machine.

> *"Added `managedMcpServers` managed setting: organizations can provide HTTP/SSE MCP servers to every user (same entry shape as `.mcp.json`); entries that name a command to run are skipped"*

**What this means in practice**: this is the lever that was missing for standardizing access to an internal tool without depending on every machine's discipline, in the `.mcp.json` shape you already know. And the restriction is well chosen: an entry naming a command to run is simply skipped, so a setting pushed from a server cannot start a local binary. HTTP and SSE, nothing else.

> *"Changed `allowedMcpServers` to govern only servers users add: a literal `managed-mcp.json` server your allowlist used to filter out now loads on upgrade; use `deniedMcpServers` to keep it off"*

**Note**: the same release moves a boundary without saying it very loudly. An allowlist that had been filtering out a `managed-mcp.json` server stops filtering it once you upgrade. Nothing visibly breaks, and what runs on your users' machines is no longer quite what you decided. If you administer a fleet, this is the line to re-read before promoting the release.

---

### A diagnostic that tells you what your skills cost, v2.1.261 (September 4, 2026)

Every loaded skill consumes context, whether or not the current session uses it.

> *"Added `/skill-doctor` to show which loaded skills go unused and what they cost in context, so you can prune them"*

**What this means in practice**: knowing which ones were dead weight took memory or luck. The command hands you both halves of the decision at once: what goes unused, and what it costs. It is the kind of modest tool that changes a habit, because we prune what we measure. Run it on a project you set up six months ago: the result is rarely flattering.

---

### Plugin containment gets corrected three times in nine days, v2.1.267 (September 9, 2026)

Three fixes, three releases, one family of flaw: a path declared by a plugin or a marketplace entry escaping the directory it was supposed to stay inside.

> *"Fixed plugins being able to read files outside their own directory through a declared command, agent, skill, hooks or other component path that is a symlink; such paths are now refused with an error"* (v2.1.257, September 1, 2026)

> *"Fixed a plugin path containing a backslash bypassing the symlink containment check on macOS and Linux"* (v2.1.265, September 8, 2026)

> *"Fixed a case where a marketplace entry path containing a backslash could bypass the containment check for fetched marketplaces on macOS and Linux"* (v2.1.267, September 9, 2026)

**What this means in practice**: none of the three is presented as exploited, and nothing suggests an emergency. What speaks is the cadence: a symlink on the 1st, a backslash in a plugin path on the 8th, the same backslash in a marketplace entry on the 9th. One reasoning error fixed in three places, which is the signature of a risk class being combed through, not of an isolated incident. The conclusion is one line long: a third-party plugin runs with your rights, and the barrier holding it in place is under repair. Favour marketplaces whose author you know, and take these releases as soon as they reach stable.

---

## Notable improvements

**A diff panel beside the conversation (v2.1.260).** `/diff` opens, in fullscreen mode, a panel showing your uncommitted changes as Claude edits, with no switching to a terminal or your editor.

> *"Added a diff panel that opens beside the conversation in fullscreen mode and shows your uncommitted changes as Claude edits; toggle it with `/diff`"*

**Auto mode hardens (v2.1.257, v2.1.261).** A containment-escape rule removes cloud metadata-credential fetches, egress evasion and cross-tenant reach from auto-approval, *"unless your environment marks them expected"*. A one-time prompt appears before the first file read outside your working directories (`permissions.blockReadsOutsideWorkingDirectories` blocks them outright). And a link that packs content into a public diagram renderer's URL is treated as what it is, an upload to that site.

**GitLab stops being a second-class citizen (v2.1.257, v2.1.259, v2.1.260).** `glab mr create/merge/close/reopen/note/update` is recognized and shows the merge request as `MR !N` in the collapsed tool summary; repository detection covers nested subgroups; an `owner/repo#123` reference links to the gitlab.com issue instead of github.com; and `/code-review --comment` posts its findings through `glab mr note`.

---

## Important bug fixes

**A whole campaign against prompt-cache misses (v2.1.265, v2.1.267).** The densest thread in the batch: about a dozen lines across two releases, all chasing the same symptom. Something in the conversation prefix changed mid-flight, the cache dropped, and the bill went up with nothing to flag it.

> *"Fixed switching models with /model re-sending every tool definition (a prompt-cache miss); commit and PR attribution text now arrives as a conversation note that updates on model changes"*

> *"Improved prompt-cache stability: subagents and sessions started with `--system-prompt` or `--append-system-prompt` now record the system prompt and tool definitions once instead of re-rendering them"*

**What this means in practice**: the triggers being fixed are everyday ones, switching models mid-session, resuming a conversation, an MCP server reconnecting, spawning a subagent. If you watch your costs, compare your cache-read ratio before and after these releases rather than taking either on faith.

**Permission rules that left a "read-only" folder writable (v2.1.260).** The most serious fix of the week, and it lands on settings files people write once and then forget.

> *"Fixed `Edit`/`Write`/`Read` permission rules whose path contains parentheses being dropped as invalid or ignored by the Bash sandbox, which left \"read-only\" folders writable"*

**What this means in practice**: a parenthesis in a path was enough to drop the rule, silently. Go re-read your rules now: the ones containing a parenthesis, a bracket or a brace are exactly the ones that may have been protecting nothing at all.

**Concurrent sessions overwriting each other (v2.1.259).** Working with several sessions open in parallel could silently lose changes written by another one.

> *"Fixed concurrent sessions silently reverting each other's `~/.claude.json` changes — workspace trust no longer resets and MCP/project state is no longer lost when running many sessions at once"*

**A malformed managed setting now blocks startup (v2.1.259).** If the managed settings file, a drop-in beside it, the MDM plist or the Windows registry value cannot be parsed, Claude Code refuses to start and names the source. It used to start up without applying governance rules you believed were in force.

**A round trip on `Read()` rules applied to Bash (v2.1.259 then v2.1.260).** v2.1.259 extended `Read()` deny rules to Bash command arguments. v2.1.260 **reverts** it: it denied `npm run build` under a `Read(./**/build/**)` rule in every mode, and made a plain `cd … && grep` prompt even in auto mode. If auto mode felt unusable in early September, that was it.

---

## Minor / cosmetic

- v2.1.267: `maxEffortLevel` caps the effort level across every provider, Bedrock, Vertex and Foundry included. `--system-prompt-snapshot off` renders the system prompt fresh on every request, useful while iterating on prompt text. Managed `allowedHttpHookUrls`, `httpHookAllowedEnvVars` and `allowedChannelPlugins` now admit nothing, rather than everything, when unreadable: when in doubt, close.
- v2.1.266: a regression fix for LLM-gateway and proxy setups, where the undocumented `CLAUDE_CODE_USE_GATEWAY` variable had started forcing a Cloud-gateway sign-in on its own. Shipped the same day as the release that caused it.
- v2.1.265: `--plugin-dir` accepts a folder of plugins, additions and removals while running included; tool results saved to disk are capped at 1 GB.
- v2.1.263: the entire public entry reads "Bug fixes and reliability improvements". We will not invent more.
- v2.1.261: `bashOutputMaxChars` and `taskOutputMaxChars` raise, up to 128K characters, how much command or background-task output Claude receives inline before it is saved to a file. The prompt's word-editing keys now match Bash and `keybindingFlavor` no longer has any effect. `--append-subagent-system-prompt-file` loads a subagent system prompt from a file. An "Organization policy" line in `/status` says why the organization policy failed to load.
- v2.1.260: `/reload-plugins` and a text form of `/advisor` reach headless sessions. `ctrl+l` / `cmd+k` in fullscreen clears the view like a terminal `clear`. The one-hour limit on background commands started by subagents is gone.
- v2.1.259: `--permission-prompts none` for unattended headless hosts, where anything that would prompt is denied automatically. `--json` on `claude plugin validate`.
- v2.1.258: Claude Code stopped starting on macOS 12 (Monterey), a regression introduced in v2.1.255.
- v2.1.257: `timeZone` and `timeFormat` settings. `defaultMode: "bypassPermissions"` set in project settings is now ignored, as `"auto"` already was: use user or managed settings, or `--permission-mode`.
- v2.1.252 through v2.1.267: terminal rendering and session resume, joined emoji splitting at line end, right-to-left text mixed with English, transcripts over 5 MB and parallel tool calls dropped on resume, "always allow" not saving in a project with no `.claude/settings.local.json`. On Windows, `Read`, `Write` and `Edit` were refusing every file inside an AppContainer sandbox.


---

**Further reading**: three containment fixes in three weeks on plugins and marketplaces, a component path routed through a symlink and then two backslash spellings, each closed after the fact. The lesson is not that a fix was missing, it is that a third-party repo's entry surface is wider than it looks: whatever loads from a marketplace, a plugin or a skill runs on your machine, with your rights. The free, tool-free manual checklist for knowing what a cloned repo actually brings with it: our guide [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.md).

---

*Watch generated from Anthropic's official changelog.*
*Interpretations and use cases are explicitly labeled as such.*
