---
title: "Harden your own Claude Code permissions: go deny-by-default"
description: "As you spread Claude Code across more projects, its permissions quietly drift toward 'allow.' Here's how to flip to deny-by-default, write rules whose scope you actually understand, and treat command approval for what it is - free, manual, vendor-neutral."
last_revalidated: 2026-08-11
claude_code_ref: v2.1.226
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/permission-modes
  - https://code.claude.com/docs/en/hooks
  - https://code.claude.com/docs/en/security
---

# Harden your own Claude Code permissions: go deny-by-default

## Why this is on your radar right now

Between v2.1.211 and v2.1.226, Anthropic closed **more than a dozen distinct ways a shell
command could get past the permission check**. Not one bug - a dozen, across sixteen
releases, nearly all in the same code path. A representative sample, verbatim from the
official changelog:

- **Commands too long to judge** (v2.1.214): *"Fixed Bash permission checks misjudging very
  long commands - commands over 10,000 characters now always prompt instead of running
  automatically."*
- **Redirections the analyzer read differently than bash did** (v2.1.214): checks now
  *"fail closed on file-descriptor redirect forms that bash parses differently than the
  permission analyzer."*
- **zsh conditionals treated as inert text - twice.** v2.1.214 fixed checks *"treating zsh
  variable subscripts and modifiers in `[[ ]]` comparisons as inert text"*; seven releases
  later, v2.1.221 fixed *"a Bash tool permission-check bypass where zsh could execute hidden
  commands in `[[ ]]` regex conditionals."*
- **Commands hiding from the dialog you're reading** (v2.1.223): *"Fixed permission prompts
  so commands padded with tabs or invisible Unicode can no longer hide part of the command
  from the approval dialog"* - shipped alongside *"a Bash permission bypass where a crafted
  command could hide parts of itself from permission checks."* Related fixes landed in
  v2.1.211 (bidirectional-override, zero-width and look-alike quote characters in relayed
  previews) and v2.1.216 (invisible Unicode in PowerShell validation, plus non-ASCII word
  boundaries in Bash parsing).
- **Commands that look read-only and aren't** (v2.1.214): `help` and `man` invocations that
  *"could run unsafe options, command substitutions, or backslash paths"* stopped
  auto-approving. So did `docker` commands carrying daemon-redirect flags (`--url`,
  `--connection`, `--identity`), and `file` used with `-m`/`--magic-file` or
  `-f`/`--files-from`.
- **Scope bugs in the rules themselves** (v2.1.214): *"Fixed single-segment `dir/**` allow
  rules like `Edit(src/**)` auto-approving writes to nested `dir/` directories anywhere in
  the tree instead of only `<cwd>/dir`."* More on that below - it's the most actionable item
  here.

### The direction has been holding since June

This isn't a sudden scramble. The same theme has been shipping steadily.

Early June (v2.1.160 → v2.1.168) made deny-by-default cheap: glob patterns in the tool-name
position of deny rules (v2.1.166); prompts before `acceptEdits` writes build-tool config that
can run code - `.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`,
`.devcontainer/` - plus shell startup files and `~/.config/git/` (v2.1.160); and
`requiredMinimumVersion` / `requiredMaximumVersion` managed settings, which make Claude Code
refuse to start outside an allowed range (v2.1.163).

Late June (v2.1.178 → v2.1.195) pushed least privilege down a level:

- **Rules went parametric** (v2.1.178): *"Added `Tool(param:value)` syntax for permission
  rules to match a tool's input parameters (with `*` wildcard), e.g. `Agent(model:opus)` to
  block Opus subagents."* v2.1.186 closed a gap where `Agent(type)` deny rules and
  `Agent(x,y)` allowed-types restrictions *"weren't being enforced for named subagent spawns."*
- **`sandbox.credentials`** (v2.1.187) blocks sandboxed commands from reading credential files
  and secret environment variables.
- **Denials became readable** (v2.1.193): auto-mode denial reasons now appear in the
  transcript, in the denial toast, and under `/permissions` recent denials.
- **A backstop for destructive commands** (v2.1.183): auto mode blocks `git reset --hard`,
  `git clean -fd`, and `terraform`/`pulumi`/`cdk destroy` when you didn't ask to throw work
  away.

Two years of patch notes pointing one direction is not noise. It's a design position. Adopt it.

## Command approval is a net, not a wall

Here's the honest read of that first list. Deciding whether a shell command is safe requires
parsing shell - and parsing shell *exactly the way the shell will* is a genuinely hard
problem. The permission analyzer is a second implementation racing bash, zsh and PowerShell.
Every gap above is the same gap: the analyzer read the command differently than the shell
would have.

Anthropic is closing these quickly and closing them the right way - v2.1.214 makes ambiguous
redirect forms *fail closed*, which is the correct default. But the structural point stands,
and Anthropic says it plainly in its own hooks documentation, about the `if` filter that gates
hooks on a command shape:

> *"The filter also fails open, running your hook regardless of pattern, when the Bash command
> can't be parsed. Because the `if` filter is best-effort, use the permission system rather
> than a hook to enforce a hard allow or deny."*

Two things follow, and both are worth acting on today.

**A narrow allowlist beats a clever denylist.** A denylist has to anticipate every spelling of
every dangerous command - every quoting trick, every shell dialect, every `$()` nesting. An
allowlist only has to enumerate the handful of things you actually do. When the parser is the
weak link, the side of the line you put "unknown" on decides your outcome. Put it on refuse.

**The approval dialog is a UI, and UIs can be lied to.** Three separate releases fixed
characters that changed what the human saw versus what would run. So when a command is long,
dense, or oddly formatted, the prompt is the worst possible moment to adjudicate. On a Bash or
PowerShell prompt, press `Ctrl+E` for an explanation of what the command does and what could
go wrong, labeled Low / Med / High risk - it only calls the model when you press it. Better
still: don't rely on in-the-moment judgment for the categories you already know you'll never
approve. Write those down as rules, once.

## The problem, in plain terms

When you run one project, you read every permission prompt. When you run ten, you stop. You
click **allow**, you set `acceptEdits` "to stop being interrupted," and over months your
posture drifts: too much is permitted by default, and you can no longer see what is actually
being *refused*. That's the failure mode - not a dramatic breach, just slow, invisible
accumulation of trust you never re-examined.

The fix is a posture, not a tool: **start from deny, and grant the minimum each project
needs.** This is the same least-privilege reflex you'd apply to a CI token or a service
account. AI tooling earned it the moment its config could run code on your machine.

## Flip to deny-by-default

This is a free, manual change to your own `settings.json`. Nothing to install.

1. **Audit what you allow today - and know where it lives.** Read `permissions.allow` in your
   user-level (`~/.claude/settings.json`) and project-level (`.claude/settings.json`) files as
   if someone else wrote them. For each entry: does *this* project actually need it? Most
   don't need most of it. Note that since **v2.1.211**, a "Yes, don't ask again" approval is
   saved to `.claude/settings.local.json` **at the root of the git repository**, resolved
   through worktrees to the main checkout - so it now applies across subdirectories and
   worktrees, where it used to be confined to the directory you started in. If you've been
   accumulating approvals from inside worktrees, they've moved. Go read that file.

2. **Don't try to "deny everything, then re-allow."** It's the intuitive move and it does not
   work. `permissions.deny` does accept `"*"` in the tool-name position, but the permissions
   reference is explicit: *"a tool matched by a bare-name glob deny rule is removed from
   Claude's context, the same as a bare tool name."* And rules are *"evaluated in order: deny,
   then ask, then allow. The first match in that order determines the outcome, and rule
   specificity doesn't change the order"* - so *"a deny rule can't carry allowlist
   exceptions."* `"deny": ["*"]` doesn't give you a floor to build on; it takes every tool
   away with no path back. If you want a true allowlist posture, use the mode built for it:
   `permissions.defaultMode: "dontAsk"` *"auto-denies every tool call that would otherwise
   prompt you,"* leaving only your `permissions.allow` rules, the built-in read-only Bash
   commands, and calls approved by a PreToolUse hook. It's designed for CI and locked-down
   environments - and it is a fast, honest way to discover how short your real allow list is.

3. **Then set a deny floor on the things that must never happen.** Targeted deny rules are the
   most durable control you have, because they're the one thing that survives every mode: per
   the modes reference, deny rules and explicit ask rules *"apply in every mode, including
   `bypassPermissions`,"* while *"allow rules have no effect in `bypassPermissions`."* If you
   write one control this year, write this one. A deny or ask rule naming a tool that doesn't
   exist produces a startup warning, so typos surface instead of silently doing nothing.

4. **Know the limit of what a deny rule covers.** Read and Edit deny rules apply to Claude's
   built-in file tools and to file commands Claude Code recognizes in Bash (`cat`, `head`,
   `tail`, `sed`). Per the docs, they *"don't apply to arbitrary subprocesses that read or
   write files indirectly, like a Python or Node script that opens files itself."* For
   enforcement that holds against any process, that's what the sandbox is for. Knowing where
   a control stops is part of using it.

5. **Don't run `acceptEdits` everywhere.** It's convenient and it's a foot-gun. Since v2.1.160
   it prompts before writing the config files that grant code execution - but the durable habit
   is to scope it to throwaway or sandboxed work, not your machine-wide default.

6. **Make `WebFetch` rules explicit.** Since v2.1.162, an explicit `WebFetch(domain:...)`
   deny/ask/allow rule takes precedence over the preapproved-host auto-allow. Note the
   wildcard is deliberately conservative: `WebFetch(domain:example.*)` matches `example.org`
   but *not* `example.evil.com`, because outside a leading `*.` the wildcard won't cross a
   dot. That's a design choice that stops a trailing wildcard from matching a domain an
   attacker could register.

7. **Scope deny rules to parameters, not just tools.** Since v2.1.178 you can target a tool's
   top-level input parameter with `Tool(param:value)` - `Agent(model:opus)`,
   `Agent(isolation:worktree)`, `Bash(run_in_background:true)` - with `*` as a wildcard.
   **This works for deny and ask rules only.** The docs are explicit that *"an allow rule for
   one parameter value wouldn't establish that the call is safe overall, so allow rules
   continue to use each tool's own specifier syntax."* One caveat: if you wrote an
   `Agent(type)` deny or an `Agent(x,y)` allowed-types rule before v2.1.186, re-check it -
   those weren't applied to named sub-agent spawns until that fix.

8. **In a team, pin a version window.** `requiredMinimumVersion` / `requiredMaximumVersion`
   (v2.1.163) in managed settings, so nobody is silently running a build that predates a fix.
   Given how many of this summer's fixes were permission-check fixes, this is the setting that
   turns "we're patched" from a hope into a fact.

9. **In a team, lock the models that can run.** Put an `availableModels` allowlist in managed
   settings and set `enforceAvailableModels: true` (v2.1.175): the allowlist then *"also
   constrains the Default model,"* and *"user or project settings can no longer widen a managed
   `availableModels` list."* v2.1.172 made it apply across the board - sub-agent model
   overrides, the dispatch picker, the advisor model. That matters because sub-agents can spawn
   their own sub-agents (depth 3 by default since v2.1.219); without enforcement reaching those
   overrides, a nested agent could pick a model you never approved.

10. **Turn on `sandbox.credentials`.** If you run sandboxed commands, set it (v2.1.187) so they
    can't read your credential files or secret environment variables. Cheap, and it closes an
    obvious exfiltration path.

## Know what your patterns actually match

This is the part most people get wrong, and v2.1.214 is why it's worth ten minutes of your
time. **The same pattern does not have the same scope everywhere.** Read and Edit rules use
[gitignore](https://git-scm.com/docs/gitignore) syntax, and for a relative pattern with a
single directory segment, the matching depth depends on the rule type.

Against a project with a top-level `src/` and a nested copy at `vendor/pkg/src/`:

| Rule | Matches `src/app.ts` | Matches `vendor/pkg/src/lib.js` |
| :--- | :--- | :--- |
| `Edit(src/**)` as an **allow** rule | Yes | **No** |
| `Edit(src/**)` as a **deny or ask** rule | Yes | **Yes** |
| `Edit(/src/**)` in any rule type | Yes | No |
| `Edit(**/src/**)` in any rule type | Yes | Yes |

That asymmetry is deliberate and it is the right call: **allow rules were narrowed, deny rules
stayed broad.** An allow rule that matches more than you meant is a hole; a deny rule that
matches more than you meant is merely conservative. Fail-safe by construction. But it means
you cannot reason about `src/**` without knowing which list it's sitting in.

And there's a third scope for the same string. In a **hook `if:` condition**, v2.1.214 changed
single-segment patterns to match only `<cwd>/dir`; to match any depth you must write
`**/dir/**`. The changelog states the split directly:

> *"Changed single-segment `dir/**` hook `if:` conditions to match only `<cwd>/dir`; write
> `**/dir/**` for any-depth matching. `deny`/`ask` permission rules keep their any-depth
> match."*

So `Edit(src/**)` means three different things depending on whether it's an allow rule, a deny
rule, or a hook condition. If you copied a pattern from one place to another - and everyone
does - go re-read it now.

Four more anchoring rules worth internalizing, all from the permissions reference:

- **A single leading slash is not an absolute path.** `/path` anchors at the *settings source*
  - `<project root>/path` in project settings, `~/.claude/path` in user settings. For a real
  absolute path use `//path`; for home-relative use `~/path`. This bites hardest in user
  settings: a deny rule `Read(/secrets/**)` there blocks `~/.claude/secrets/**`, **not** a
  `secrets` directory in your project.
- **Bare filenames match at any depth.** `Read(.env)` and `Read(**/.env)` are equivalent.
- **Deny file writes with `Edit(...)`, not `Write(...)`.** A `Write(docs/**)` deny rule doesn't
  do what it looks like; Claude Code prints a startup warning telling you so -
  *"Write(docs/\*\*) is not matched by file permission checks - only Edit(path) rules are. Use
  Edit(docs/\*\*) instead."* `Edit` rules cover every file-editing tool.
- **On Windows, paths normalize to POSIX form** before matching: `C:\Users\alice` becomes
  `/c/Users/alice`. Use `//c/**/.env` for one drive, `//**/.env` across all of them.

## Anti-patterns to drop

- **Allow-by-default you've never audited.** If you can't say what your config refuses, it
  refuses nothing.
- **`acceptEdits` as a global default.** Convenience that writes code-granting files on your
  behalf is not convenience.
- **Trusting a rule you never saw match.** A rule can be syntactically valid and semantically
  inert. v2.1.224 fixed sandbox filesystem deny entries *"written with a trailing slash (e.g.
  `denyRead: "~/.aws/"`) being silently bypassable on Linux and macOS."* A trailing slash. The
  lesson generalizes: after writing a protective rule, provoke it once and confirm it fires.
- **Trying to scope Bash by parameter.** `Bash(command:rm *)` looks reasonable and is ignored
  with a startup warning, because a compound command would bypass it. Use `Bash(rm *)`. The
  same applies to the primary content field of Read, Edit, Write, Grep, Glob, NotebookEdit and
  WebFetch.
- **Leaning on a hook `if:` condition as enforcement.** It's a best-effort filter that fails
  *open* on commands it can't parse. Anthropic's own guidance is to use the permission system
  for a hard allow or deny.
- **Running an old version "because it works."** More than a dozen permission-check fixes
  shipped in sixteen releases this summer. They only protect you if you have them. Check with
  `claude doctor`; a version window (v2.1.163) makes it a team setting rather than a hope.
- **Assuming a managed model allowlist can be loosened locally.** Since v2.1.175, project or
  user settings can no longer widen a managed `availableModels` list. If you set it as the
  floor, treat it as the floor.
- **Never reading what got refused.** Since v2.1.193 denials show up in the transcript, in a
  toast, and under `/permissions` recent denials. A deny floor you never look at is a guess;
  the recent-denials list tells you whether it's matching what you intended - or just blocking
  real work and training you to widen it.
- **Copying a `settings.json` you found online without reading it.** Same reflex as treating
  [a cloned repo's config as untrusted code](untrusted-repo-config.md) - except here it's
  *your* posture you're handing over.

## One more, for multi-session setups

If you run several Claude sessions that talk to each other, note v2.1.166: messages relayed via
`SendMessage` from another session **no longer carry user authority** - the receiver refuses
relayed permission requests. You don't configure anything; it's just true now. The surface has
kept hardening as it grew: v2.1.222 routes `SendMessage` through the permission classifier
before dispatch, v2.1.224 added `crossSessionInbound` and `dialogExpiry` settings so messages
sent to a session running with bypassed permissions are held for your approval, and v2.1.225
added a workspace trust prompt to `claude agents` for untrusted directories - matching what
`claude` already did. Your deny floor can't be lifted by a message arriving from a sibling
session. That's the model working *with* deny-by-default, not around it.

---

<sub>Last revalidated **2026-08-11** against Claude Code **v2.1.226**. This guide is
maintained alongside our [weekly watch](../../watch/) - when the ecosystem shifts, this
page is re-checked. Sources are listed in the page header.</sub>

<sub>Auditing this by hand across many projects gets tedious.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> reads your projects'
Claude Code config locally and lays out what each one allows and denies in one place - so
spotting the project that's still allow-by-default takes seconds instead of ten file opens.
Use it or not; the posture is what matters.</sub>
