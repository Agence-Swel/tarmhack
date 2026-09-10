---
title: "Harden your own Claude Code permissions: go deny-by-default"
description: "As you spread Claude Code across more projects, its permissions quietly drift toward 'allow.' Here's how to flip to deny-by-default, write rules whose scope you actually understand, and treat command approval for what it is — free, manual, vendor-neutral."
last_revalidated: 2026-09-10
claude_code_ref: v2.1.267
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/permission-modes
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/hooks
  - https://code.claude.com/docs/en/security
  - https://www.npmjs.com/package/@anthropic-ai/claude-code?activeTab=versions
---

# Harden your own Claude Code permissions: go deny-by-default

## First: which build are you actually running?

Before any of the advice below is worth anything, check what you have. As of **2026-09-10**,
the npm dist-tags for `@anthropic-ai/claude-code` read `stable` at **v2.1.236** and `latest`
at **v2.1.267** — and **`stable` hasn't moved since 19 August.** Twenty-two releases separate
the two channels.

That's a governance fact, not a complaint. Almost every hardening described on this page — the
parenthesis bug that silently dropped file rules, the symlink swapped after the check, the
`Read()` rules that Grep and Glob weren't applying — landed after v2.1.236. If your team pins
`stable`, or if your install has simply been sitting there, **you do not have them.** Run
`claude doctor`, read the version, and decide deliberately which channel you're on. A pinned
channel is a legitimate choice; an unexamined one is not.

## Why this is on your radar right now

Between v2.1.211 and v2.1.226, Anthropic closed **more than a dozen distinct ways a shell
command could get past the permission check**. Not one bug — a dozen, across fifteen
releases, nearly all in the same code path. A representative sample, verbatim from the
official changelog:

- **Commands too long to judge** (v2.1.214): *"Fixed Bash permission checks misjudging very
  long commands — commands over 10,000 characters now always prompt instead of running
  automatically."*
- **Redirections the analyzer read differently than bash did** (v2.1.214): checks now
  *"fail closed on file-descriptor redirect forms that bash parses differently than the
  permission analyzer."*
- **zsh conditionals treated as inert text — twice.** v2.1.214 fixed checks *"treating zsh
  variable subscripts and modifiers in `[[ ]]` comparisons as inert text"*; seven releases
  later, v2.1.221 fixed *"a Bash tool permission-check bypass where zsh could execute hidden
  commands in `[[ ]]` regex conditionals."*
- **Commands hiding from the dialog you're reading** (v2.1.223): *"Fixed permission prompts
  so commands padded with tabs or invisible Unicode can no longer hide part of the command
  from the approval dialog"* — shipped alongside *"a Bash permission bypass where a crafted
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
  the tree instead of only `<cwd>/dir`."* More on that below — it's the most actionable item
  here.

### It did not stop there: v2.1.227 → v2.1.267

Six weeks later the same code path is still producing the same shape of finding. The newer
ones are, if anything, more instructive, because several of them broke **the rules themselves**
rather than the command parser:

- **A rule can be silently discarded for a character in its path** (v2.1.260): *"Fixed
  `Edit`/`Write`/`Read` permission rules whose path contains parentheses being dropped as
  invalid or ignored by the Bash sandbox, which left "read-only" folders writable."* Read that
  last clause again. The rule was in the file. The folder was writable.
- **One bad rule could break every edit** (v2.1.260): *"Fixed one file permission rule with an
  uncompilable pattern (e.g. an unclosed `[`) making every file edit fail with `Invalid regular
  expression`; such a deny rule now guards the literal path it spells."*
- **A rule that never matched anything used to be accepted in silence** (v2.1.260): *"Changed
  permission rules with text after the closing parenthesis (e.g. `Bash(ls) x`), which never
  matched anything, to be reported as invalid settings instead of being silently ignored."*
- **The file could change between the check and the write** (v2.1.251): *"Fixed file tools
  (Read, Write, Edit) following a symlink swapped inside the working directory after the
  permission check, which could read or write outside the approved location."*
- **Two tools weren't applying your deny rules at all** (v2.1.251): *"Fixed Grep and Glob not
  applying `Read(...)` deny rules to files reached through a symlinked search path."*
- **Arithmetic as a bypass** (v2.1.251): *"Fixed Bash permission checks auto-approving commands
  that assign an arithmetic expression to an integer shell variable (e.g. `OPTIND=1/0`,
  `RANDOM=2+2`); these now prompt for approval."*
- **zsh, a fifth time** (v2.1.260): *"Fixed Bash permission checks auto-approving zsh commands
  that hide a command substitution in a REPORTTIME, REPORTMEMORY or DIRSTACKSIZE assignment;
  these now prompt for approval."*
- **An `ask` rule that didn't ask** (v2.1.257): *"Fixed a `permissions.ask` rule being skipped
  in auto mode when the matching command ran inside a compound command or subshell, letting it
  run without the confirmation prompt."*

### The direction has been holding since June

This isn't a sudden scramble. The same theme has been shipping steadily.

Early June (v2.1.160 → v2.1.168) made deny-by-default cheap: glob patterns in the tool-name
position of deny rules (v2.1.166); prompts before `acceptEdits` writes build-tool config that
can run code — `.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`,
`.devcontainer/` — plus shell startup files and `~/.config/git/` (v2.1.160); and
`requiredMinimumVersion` / `requiredMaximumVersion` managed settings, which make Claude Code
refuse to start outside an allowed range (v2.1.163).

Late June (v2.1.178 → v2.1.195) pushed least privilege down a level:

- **Rules went parametric** (v2.1.178): *"Added `Tool(param:value)` syntax for permission
  rules to match a tool's input parameters (with `*` wildcard), e.g. `Agent(model:opus)` to
  block Opus subagents."* v2.1.186 fixed `Agent(type)` deny rules and `Agent(x,y)`
  allowed-types restrictions *"not being enforced for named subagent spawns."*
- **`sandbox.credentials`** (v2.1.187) blocks sandboxed commands from reading credential files
  and secret environment variables.
- **Denials became readable** (v2.1.193): auto-mode denial reasons now appear in the
  transcript, in the denial toast, and under `/permissions` recent denials.
- **A backstop for destructive commands** (v2.1.183): auto mode blocks `git reset --hard`,
  `git clean -fd`, and `terraform`/`pulumi`/`cdk destroy` when you didn't ask to throw work
  away.

Through August the same line kept extending. The one to internalize: v2.1.235 fixed permission
dialogs so *"display text and "don't ask again" options now always match what a grant would
cover, and "don't ask again" is withheld when contents cannot be fully displayed."* Until then,
a "don't ask again" click could grant more than the dialog had shown you.

Two years of patch notes pointing one direction is not noise. It's a design position. Adopt it.

## Command approval is a net, not a wall

Here's the honest read of those lists. Deciding whether a shell command is safe requires
parsing shell — and parsing shell *exactly the way the shell will* is a genuinely hard
problem. The permission analyzer is a second implementation racing bash, zsh and PowerShell.
Nearly every gap above is the same gap: the analyzer read the command differently than the
shell would have. The newest findings widen the point by one turn of the screw — the analyzer
can also read **your rule file** differently than you did (parentheses, v2.1.260), and the
filesystem can change underneath a decision already taken (symlink swap, v2.1.251).

Anthropic is closing these quickly and closing them the right way — v2.1.214 makes ambiguous
redirect forms *fail closed*, v2.1.246 made checks *"always require approval for malformed
commands with a dangling `&&` or `||` operator"*, and v2.1.267 fixed managed
`allowedHttpHookUrls`, `httpHookAllowedEnvVars` and `allowedChannelPlugins` *"to admit nothing,
not everything, when unreadable."* Fail-closed on the unparseable is the correct default, and it
is being applied consistently. But the structural point stands, and Anthropic says it plainly in
its own hooks documentation, about the `if` filter that gates hooks on a command shape:

> *"The filter also fails open, running your hook regardless of pattern, when the Bash command
> can't be parsed. Because the `if` filter is best-effort, use the permission system rather
> than a hook to enforce a hard allow or deny."*

Two things follow, and both are worth acting on today.

**A narrow allowlist beats a clever denylist.** A denylist has to anticipate every spelling of
every dangerous command — every quoting trick, every shell dialect, every `$()` nesting. An
allowlist only has to enumerate the handful of things you actually do. When the parser is the
weak link, the side of the line you put "unknown" on decides your outcome. Put it on refuse.

**The approval dialog is a UI, and UIs can be lied to.** Three separate releases fixed
characters that changed what the human saw versus what would run. So when a command is long,
dense, or oddly formatted, the prompt is the worst possible moment to adjudicate — and you now
have one less crutch there: v2.1.257 *"Removed the Ctrl+E command explanation on Bash and
PowerShell permission prompts."* If you had built the habit of pressing `Ctrl+E` for a
Low/Med/High risk read on a command you didn't understand, that habit no longer has anything to
press. Which only sharpens the real advice: don't rely on in-the-moment judgment for the
categories you already know you'll never approve. Write those down as rules, once.

## A rule's scope can move under your feet — in both directions

This is the newest lesson on this page and the one most likely to bite a careful reader,
because it punishes exactly the person who wrote good rules and moved on.

In **v2.1.259**, Bash argument handling was tightened: `Read()` deny rules started covering
*"files given as option values (`--ignore-revs-file=.env`, `-f.env`, `@file`), `git diff`/`git
grep` file operands, or `cd DIR && cat FILE` compounds."* Strictly more protective. Then, one
release later, **v2.1.260 took it back**:

> *"Reverted the 2.1.259 change applying `Read()` deny rules to Bash arguments; it denied `npm
> run build` under a `Read(./**/build/**)` rule in every mode and made `cd … && grep` prompt
> even in auto mode."*

Note what the revert reason tells you: a perfectly ordinary deny rule, `Read(./**/build/**)`,
suddenly stopped `npm run build`. The tightening was right in intent and wrong in blast radius,
and it shipped and un-shipped inside two releases.

It has happened before on the same surface. v2.1.232 announced *"Bash input redirections
(`< file`) are now permission-checked like their argument spellings on all platforms"*; v2.1.233
*"Reverted the 2.1.232 Bash permission changes for Cygwin-style symlinks on Windows and for
input redirections (`< file`); a narrower version will return in a later release"*; and in
v2.1.257 the narrower version duly returned — *"Fixed Bash `Read()`/`Edit()` deny rules not
applying to `< file` redirects and reader commands like `tac` and `egrep`; a deny rule on any
argument or redirect target now refuses the command."* Out, then back, across twenty-five
releases.

Three practical consequences:

1. **A rule that worked last month can be broader or narrower today**, and neither direction is
   announced before it lands. Broader means your build breaks; narrower means your floor
   quietly has a hole in it.
2. **Protective rules therefore need a periodic liveness test**, not a one-time write. Once a
   quarter, or after any upgrade you notice, provoke each one and confirm it still fires.
3. **Read release notes for reverts, not just for fixes.** A revert line is the only place a
   restriction's removal is ever recorded.

## The problem, in plain terms

When you run one project, you read every permission prompt. When you run ten, you stop. You
click **allow**, you set `acceptEdits` "to stop being interrupted," and over months your
posture drifts: too much is permitted by default, and you can no longer see what is actually
being *refused*. That's the failure mode — not a dramatic breach, just slow, invisible
accumulation of trust you never re-examined.

The fix is a posture, not a tool: **start from deny, and grant the minimum each project
needs.** This is the same least-privilege reflex you'd apply to a CI token or a service
account. AI tooling earned it the moment its config could run code on your machine.

## Flip to deny-by-default

This is a free, manual change to your own `settings.json`. Nothing to install.

1. **Audit what you allow today — and know where it lives.** Read `permissions.allow` in your
   user-level (`~/.claude/settings.json`) and project-level (`.claude/settings.json`) files as
   if someone else wrote them. For each entry: does *this* project actually need it? Most
   don't need most of it. Note that since **v2.1.211**, a "Yes, don't ask again" approval is
   saved to `.claude/settings.local.json` **at the root of the git repository**, resolved
   through worktrees to the main checkout — so it now applies across subdirectories and
   worktrees, where it used to be confined to the directory you started in. If you've been
   accumulating approvals from inside worktrees, they've moved. Go read that file.

2. **Don't try to "deny everything, then re-allow."** It's the intuitive move and it does not
   work. `permissions.deny` does accept `"*"` in the tool-name position, but the permissions
   reference is explicit: *"a tool matched by a bare-name glob deny rule is removed from
   Claude's context, the same as a bare tool name."* And rules are *"evaluated in order: deny,
   then ask, then allow. The first match in that order determines the outcome, and rule
   specificity doesn't change the order"* — so *"a deny rule can't carry allowlist
   exceptions."* `"deny": ["*"]` doesn't give you a floor to build on; it takes every tool
   away with no path back. If you want a true allowlist posture, use the mode built for it:
   `permissions.defaultMode: "dontAsk"` *"auto-denies every tool call that would otherwise
   prompt you,"* leaving only your `permissions.allow` rules, the built-in read-only Bash
   commands, and calls approved by a PreToolUse hook. It's designed for CI and locked-down
   environments — and it is a fast, honest way to discover how short your real allow list is.

3. **Then set a deny floor on the things that must never happen.** Targeted deny rules are the
   most durable control you have, because they're the one thing that survives every mode: per
   the modes reference, deny rules and explicit ask rules *"apply in every mode, including
   `bypassPermissions`,"* while *"allow rules have no effect in `bypassPermissions`."* If you
   write one control this year, write this one. A deny or ask rule naming a tool that doesn't
   exist produces a startup warning, so typos surface instead of silently doing nothing — and
   since v2.1.260 so does a rule with stray text after its closing parenthesis, which used to
   be accepted and ignored. The warning doesn't catch everything, though: see below for the two
   characters that can void a rule without a word.

4. **Know the limit of what a deny rule covers — and that the limit moves.** Read and Edit deny
   rules apply to Claude's built-in file tools and to file commands Claude Code recognizes in
   Bash. That recognized set is wider than it was: v2.1.257 extended it to `< file` redirects
   and reader commands such as `tac` and `egrep`, and v2.1.251 made Grep and Glob honor
   `Read(...)` deny rules through a symlinked search path. It is also narrower than v2.1.259
   briefly made it — see the section above. What has not changed is the outer boundary: per the
   docs, these rules *"don't apply to arbitrary subprocesses that read or write files
   indirectly, like a Python or Node script that opens files itself."* For enforcement that
   holds against any process, that's what the sandbox is for. Knowing where a control stops is
   part of using it.

5. **Consider blocking reads outside your working directories.** v2.1.257 added *"a one-time
   prompt in auto mode before the first file read outside the working directories, with the
   option to block such reads (`permissions.blockReadsOutsideWorkingDirectories`)."* This is
   the highest-leverage new setting on this page: it converts "Claude can read my whole home
   directory unless a deny rule says otherwise" into "Claude reads the project, and asks once
   before anything else." Turn it on, then expect to grant a couple of exceptions — v2.1.260
   fixed it *"on macOS hiding the user's git config from sandboxed git and hiding a
   worktree-isolated sub-agent's own checkout"*, so make sure you're past that build before
   judging it.

6. **Don't run `acceptEdits` everywhere.** It's convenient and it's a foot-gun. Since v2.1.160
   it prompts before writing the config files that grant code execution — but the durable habit
   is to scope it to throwaway or sandboxed work, not your machine-wide default. Related, and
   worth knowing if you inherit someone else's repo: since v2.1.257, `defaultMode:
   "bypassPermissions"` set *"in `.claude/settings.json` or `.claude/settings.local.json`"* is
   *"ignored, like `"auto"`; set it in user or managed settings, or pass `--permission-mode`."*
   A checked-in project file can no longer put your session in bypass mode.

7. **Make `WebFetch` rules explicit.** Since v2.1.162, an explicit `WebFetch(domain:...)`
   deny/ask/allow rule takes precedence over the preapproved-host auto-allow. Note the
   wildcard is deliberately conservative: `WebFetch(domain:example.*)` matches `example.org`
   but *not* `example.evil.com`, because outside a leading `*.` the wildcard won't cross a
   dot. That's a design choice that stops a trailing wildcard from matching a domain an
   attacker could register.

8. **Scope deny rules to parameters, not just tools.** Since v2.1.178 you can target a tool's
   top-level input parameter with `Tool(param:value)` — `Agent(model:opus)`,
   `Agent(isolation:worktree)`, `Bash(run_in_background:true)` — with `*` as a wildcard.
   **This works for deny and ask rules only.** The docs are explicit that *"an allow rule for
   one parameter value wouldn't establish that the call is safe overall, so allow rules
   continue to use each tool's own specifier syntax."* One caveat: if you wrote an
   `Agent(type)` deny or an `Agent(x,y)` allowed-types rule before v2.1.186, re-check it —
   those weren't applied to named sub-agent spawns until that fix.

9. **For unattended runs, use the flags built for them instead of loosening your settings.**
   Two arrived this summer, and they are the right answer to "the agent keeps stopping in CI":

   - **`--restricted`** (v2.1.248, or `CLAUDE_CODE_RESTRICTED=1`) *"removes the built-in tools
     that run commands or code and `WebFetch` (unless named in `--tools`), keeps file tools
     inside the working directory, refuses `bypassPermissions`, and ignores user, project and
     local settings files."* Read that last clause precisely: the settings it sets aside are
     the **user, project and local** ones. **Managed settings are not in that list and continue
     to apply** — which is what you want, since managed settings are where an organization's
     floor lives. `--restricted` is not "no configuration is read"; it is "your own three tiers
     of configuration are set aside."
   - **`--permission-prompts none`** (v2.1.259) *"for unattended headless hosts: anything that
     would prompt is denied automatically while the active permission mode (including auto
     mode) keeps deciding."* Deny-by-default as a launch flag, for the one situation where
     nobody is there to answer.

10. **In a team, pin a version window.** `requiredMinimumVersion` / `requiredMaximumVersion`
    (v2.1.163) in managed settings, so nobody is silently running a build that predates a fix.
    Given how many of this summer's fixes were permission-check fixes — and given the
    31-release gap between `stable` and `latest` noted at the top of this page — this is the
    setting that turns "we're patched" from a hope into a fact.

11. **In a team, make an unreadable policy fail loudly.** v2.1.259 fixed *"managed settings
    silently going unenforced when the managed-settings file, a drop-in, the MDM plist, or the
    HKLM value cannot be parsed: Claude Code now refuses to start and names the source."* You
    don't configure this — it's behavior you inherit by being current. But it's worth knowing
    the failure mode it replaced: a typo in your org policy used to mean *no* policy, quietly,
    on every machine. If your fleet is pinned below v2.1.259, validate that file by hand.

12. **In a team, lock the models that can run.** Put an `availableModels` allowlist in managed
    settings and set `enforceAvailableModels: true` (v2.1.175): the allowlist then *"also
    constrains the Default model,"* and *"user or project settings can no longer widen a managed
    `availableModels` list."* v2.1.172 made it apply across the board — sub-agent model
    overrides, the dispatch picker, the advisor model. That matters because sub-agents can spawn
    their own sub-agents (depth 3 by default since v2.1.219); without enforcement reaching those
    overrides, a nested agent could pick a model you never approved.

13. **Turn on `sandbox.credentials`.** If you run sandboxed commands, set it (v2.1.187) so they
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
rule, or a hook condition. If you copied a pattern from one place to another — and everyone
does — go re-read it now.

Four more anchoring rules worth internalizing, all from the permissions reference:

- **A single leading slash is not an absolute path.** `/path` anchors at the *settings source*
  — `<project root>/path` in project settings, `~/.claude/path` in user settings. For a real
  absolute path use `//path`; for home-relative use `~/path`. This bites hardest in user
  settings: a deny rule `Read(/secrets/**)` there blocks `~/.claude/secrets/**`, **not** a
  `secrets` directory in your project.
- **Bare filenames match at any depth.** `Read(.env)` and `Read(**/.env)` are equivalent.
- **Deny file writes with `Edit(...)`, not `Write(...)`.** A `Write(docs/**)` deny rule doesn't
  do what it looks like, and since v2.1.210 Claude Code says so at startup — that release
  *"Added a startup warning for `Write(path)`, `NotebookEdit(path)`, and `Glob(path)` permission
  rules — use `Edit(path)` or `Read(path)` instead."* `Edit` rules cover every file-editing tool.
- **On Windows, paths normalize to POSIX form** before matching: `C:\Users\alice` becomes
  `/c/Users/alice`. Use `//c/**/.env` for one drive, `//**/.env` across all of them.

### Two characters that can void a rule outright

Two things are worth checking for by eye, right now, in every settings file you own.

**A parenthesis anywhere in the path.** Until v2.1.260, an `Edit`/`Write`/`Read` rule whose
path contained `(` or `)` could be *"dropped as invalid or ignored by the Bash sandbox, which
left "read-only" folders writable."* On Windows this is not an exotic case —
`C:\Program Files (x86)\...` and `.../My Project (old)/...` are ordinary paths. The same release
improved the diagnostic *"for rules such as `Edit(C:\dir\(name)\**)`, where `\(` is read as an
escaped parenthesis rather than a path separator, to suggest an unambiguous spelling."* If a
protective rule of yours names a path with a parenthesis in it, treat it as unproven until you
have watched it fire on a build at v2.1.260 or later.

**An unclosed bracket in any one rule.** Before v2.1.260, a single rule with an uncompilable
pattern made *"every file edit fail with `Invalid regular expression`."* One typo, all edits
down. It now degrades gracefully — *"such a deny rule now guards the literal path it spells"* —
but the older behavior is a useful symptom to recognize: if edits start failing globally right
after you touch permissions, look for a bracket, not a bug.

### A wildcard placed before a subcommand matches more than you think

v2.1.246 *"Added a startup warning for Bash allow rules with a wildcard before the subcommand
(e.g. `Bash(git * main)`), since they also match options inserted before the subcommand."*

The failure here is not that such a rule does nothing — it's that it does **more** than its
author intended. Anything that fits between `git` and `main`, including options you never meant
to bless, satisfies the pattern. It's an allow rule with an unexamined hole in the middle, and
it reads perfectly natural. Keep wildcards after the subcommand, where the shape of the command
is already pinned down.

## Anti-patterns to drop

- **Allow-by-default you've never audited.** If you can't say what your config refuses, it
  refuses nothing.
- **`acceptEdits` as a global default.** Convenience that writes code-granting files on your
  behalf is not convenience.
- **Trusting a rule you never saw match.** A rule can be syntactically valid and semantically
  inert, and the evidence for this keeps arriving. v2.1.224 fixed sandbox filesystem deny
  entries *"written with a trailing slash (e.g. `denyRead: "~/.aws/"`) being silently
  bypassable on Linux and macOS."* v2.1.257 fixed sandbox network hosts *"written with a
  trailing dot (`example.com.`)"*, where a `deniedDomains` entry *"didn't block the host inside
  the sandbox."* v2.1.260 fixed the parenthesis case above. A trailing slash, a trailing dot, a
  parenthesis. The lesson generalizes: after writing a protective rule, provoke it once and
  confirm it fires — then provoke it again after an upgrade.
- **Assuming a rule's scope is stable.** It isn't, in either direction — see the round trip at
  v2.1.259/v2.1.260 above. Re-test protective rules on a schedule, and read revert lines in
  release notes as carefully as fix lines.
- **Trying to scope Bash by parameter.** `Bash(command:rm *)` looks reasonable and is ignored
  with a startup warning, because a compound command would bypass it. Use `Bash(rm *)`. The
  same applies to the primary content field of Read, Edit, Write, Grep, Glob, NotebookEdit and
  WebFetch.
- **Leaning on a hook `if:` condition as enforcement.** It's a best-effort filter that fails
  *open* on commands it can't parse. Anthropic's own guidance is to use the permission system
  for a hard allow or deny.
- **Running an old version "because it works."** More than a dozen permission-check fixes
  shipped in fifteen releases this summer, and the six weeks since produced another run of
  them. They only protect you if you have them. Check with `claude doctor`, and remember that
  `stable` and `latest` are 22 releases apart as of this revalidation; a version window
  (v2.1.163) makes it a team setting rather than a hope.
- **Assuming a managed model allowlist can be loosened locally.** Since v2.1.175, project or
  user settings can no longer widen a managed `availableModels` list. If you set it as the
  floor, treat it as the floor.
- **Never reading what got refused.** Since v2.1.193 denials show up in the transcript, in a
  toast, and under `/permissions` recent denials — and since v2.1.234 you can open
  `/permissions` while Claude is working, with rule changes applying to the rest of the current
  turn. A deny floor you never look at is a guess; the recent-denials list tells you whether
  it's matching what you intended — or just blocking real work and training you to widen it.
- **Waiting for the prompt to explain a command to you.** `Ctrl+E` is gone as of v2.1.257. If
  you can't read a command at the prompt, the answer is to refuse it and read it in an editor —
  never to approve it and find out.
- **Copying a `settings.json` you found online without reading it.** Same reflex as treating
  [a cloned repo's config as untrusted code](untrusted-repo-config.md) — except here it's
  *your* posture you're handing over.

## One more, for multi-session setups

If you run several Claude sessions that talk to each other, note v2.1.166: messages relayed via
`SendMessage` from another session **no longer carry user authority** — the receiver refuses
relayed permission requests. You don't configure anything; it's just true now. The surface has
kept hardening as it grew: v2.1.222 routes `SendMessage` through the permission classifier
before dispatch, v2.1.224 added `crossSessionInbound` and `dialogExpiry` settings so messages
sent to a session running with bypassed permissions are held for your approval, and v2.1.225
added a workspace trust prompt to `claude agents` for untrusted directories — matching what
`claude` already did.

Two follow-ups if you set `crossSessionInbound`. v2.1.248 fixed *"an invalid `crossSessionInbound`
value being silently ignored: it now warns and holds cross-session messages (user settings) or
refuses them (managed settings) until fixed"* — another instance of a rule that used to be inert
without telling you. And v2.1.234 fixed *"session-scoped permission answers (including denies)
being dropped when answering background subagent tool permission prompts"*, so a refusal you
gave a background agent now actually sticks for the session.

Your deny floor can't be lifted by a message arriving from a sibling session. That's the model
working *with* deny-by-default, not around it.

---

<sub>Last revalidated **2026-09-10** against Claude Code **v2.1.267**. This guide is
maintained alongside our [weekly watch](../../watch/) — when the ecosystem shifts, this
page is re-checked. Sources are listed in the page header.</sub>

<sub>Auditing this by hand across many projects gets tedious.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> reads your projects'
Claude Code config locally and lays out what each one allows and denies in one place — so
spotting the project that's still allow-by-default takes seconds instead of ten file opens.
Use it or not; the posture is what matters.</sub>
