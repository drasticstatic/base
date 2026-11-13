# Why drasticstatic keeps this fork

**BASE** (the `base` tool by Chris Kahler) is a companion for Claude Code: a single Rust binary that registers Claude
Code hooks and feeds each session context from a local knowledge graph, a code map of the repository and a
file-based relay between sessions. drasticstatic keeps this public fork so that any copy we might run comes from code
someone has read, at a revision that does not move by itself. On our pages it is called "our fork of `base`"; the
licence's naming line is quoted below.

## Thank you

This is Chris Kahler's work (GitHub: ChristopherKahler): <https://github.com/ChristopherKahler/base>. Thank you for
building it and for sharing it. The source writes its own trade-offs down: the updater's comment says that it does
no signature check, and the settings that change each behaviour are named in the code and its tests. Anything
good here is the author's; any change of ours is listed below and is ours to answer for. Please report problems with
the original upstream, and problems with our changes to us.

## The pin

| | |
|---|---|
| Upstream | <https://github.com/ChristopherKahler/base> |
| Reviewed revision | `a37b9a8f88fb5676ff9ed3ddd9f124d4cd6aa8ae` (2026-10-06), tag `v0.16.0` |
| Compared against | `566c75314f7e0a223791b224d151dbea0c6faab4` (2026-09-21, `v0.15.2`), the revision this fork sat at before: 166 upstream commits in between, brought in as a fast-forward |
| Reviewed on | 2026-11-12, by `Alfred · ClaudeCodeCLI · Anthropic [Opus-5.5]`, as text: nothing was installed, built or run |
| Licence | `FSL-1.1-ALv2` (Functional Source License, Version 1.1, ALv2 Future License), unchanged: [`LICENSE.md`](./LICENSE.md), [`LICENSING.md`](./LICENSING.md), [`LICENSE-APACHE-2.0`](./LICENSE-APACHE-2.0) |
| The review | [`audits/base/2026-11-12_v0.16.0_a37b9a8f.md`](https://github.com/drasticstatic/code-forked/blob/main/audits/base/2026-11-12_v0.16.0_a37b9a8f.md) in the code-forked library |
| Antivirus receipt | None recorded for this revision |
| Runs as | Nowhere. A private working copy holds the same revision; the tool is not installed on any of our machines |

## The licence, in its own words

- `Cargo.toml` line 5: `license = "FSL-1.1-ALv2"`. `LICENSE.md` line 1: *"Functional Source License, Version 1.1,
  ALv2 Future License"*; its notice reads *"Copyright 2026 Chris Kahler"*. Each version converts to the Apache
  License 2.0 *"effective on the second anniversary"* of its release (`LICENSE.md` line 90).
- `LICENSE.md` lines 26-28 grant the right to *"use, copy, modify, create derivative works, publicly perform,
  publicly display and redistribute the Software for any Permitted Purpose"*, which is any purpose other than a
  *"Competing Use"*: making it available to others in a commercial product or service that substitutes for it
  (lines 32-36).
- `LICENSING.md` line 27 lists, among what is permitted with no separate agreement: *"Modifying the source, running
  your fork internally, and redistributing your changes under these same terms."* Line 16: the engine (`src/`, the
  binary, `Cargo.toml`, `tests/`) is FSL; extensions, adapters and skills the project publishes are Apache-2.0.
  Releases up to v0.14.1 were PolyForm Noncommercial 1.0.0 (line 70).
- **The naming line**, `LICENSING.md` lines 83-85: *"A fork is welcome and permitted; calling a fork basemode is
  not."*
- `CONTRIBUTING.md` lines 47-59: anyone opening a pull request upstream grants the author a right to relicense the
  contribution.
- The three licence files are byte-identical at `566c7531` and `a37b9a8f`.

These are the licence's words and our plain reading of them; this page gives no legal advice.

## Before anyone runs this

What `v0.16.0` does, as read in its source at `a37b9a8f`. Each behaviour is paired with the setting that changes it,
where the review found one. Paths are at the repository root; `~/.base-gbl/` is the tool's own global folder.

1. **It updates itself at session start, by default, without verifying the download.** Every SessionStart hook
   calls `auto_update` (`src/hook/session_start.rs` lines 184-186), which asks
   `api.github.com/repos/ChristopherKahler/base/releases/latest` (`src/update/mod.rs` lines 34-35), runs the
   downloaded binary with `--version` and swaps it over the running binary (lines 364-374, 392-424, 572-582). The
   source says so itself (lines 24-27): *"There is no signature check and no checksum check in the download path."*
   A copy built from this fork is replaced by upstream's next release. **Off:** `[update] auto = false` in
   `~/.base-gbl/base.toml` (default `true`, `src/config.rs` lines 1031-1045), or the environment variable
   `BASE_NO_AUTO_UPDATE` set to any value, or `[devmode] enabled = true` (`session_start.rs` lines 1057-1067). A
   workspace's own `.base/base.toml` overrides the global value (`config.rs` lines 1930-1931). No setting pins the
   version the updater installs.
2. **It writes a section into the user's global Claude instructions.** `base install` appends `## BASE CLI` to
   `~/.claude/CLAUDE.md`, creating the file if needed (`src/install.rs` lines 1917-1950), including *"ALWAYS use
   `base ast query` BEFORE grep, find, Read, or any MCP tool for code exploration."* (lines 1739-1741). The section
   is refreshed on each new version (`session_start.rs` lines 115-131). `base scaffold` adds a "Registered
   Workspaces" block listing every registered workspace path (`src/scaffold.rs` lines 124, 218-302). No setting
   that skips this was found; `base uninstall` removes the `## BASE CLI` section (`install.rs` lines 338-387).
3. **Six global hooks, which put themselves back.** `HOOK_TABLE` (`install.rs` lines 903-911) registers
   SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop and SessionEnd in `~/.claude/settings.json`, with no
   `matcher`, so PreToolUse and PostToolUse fire on every tool call (lines 927-931). `ensure_hooks_wired`
   (`install.rs` lines 1023-1061) adds any missing hook at every session start and at the start of most `base`
   commands (`src/cli.rs` lines 2154-2199), once per version. `base install --skip-hooks` (lines 79-84) is not
   remembered.
4. **It writes inside every repository a session touches.** A code map is built in `<repo>/.base-ast/` for the
   session's repository and for any repository whose files are read, edited or searched (`src/hook/automap.rs` lines
   1-45), and a first build installs `.git/hooks/post-checkout` and `post-merge` (`src/ast_repo.rs` lines 22-61; an
   existing hook that is not base's is left alone). No configuration key turns this off; the only switch found is
   the environment variable `BASE_AST_NO_SPAWN` (`automap.rs` line 56), which the source describes as a test
   setting.
5. **Prompt text is logged, by default for 90 days.** `[log] prompt_text` defaults to `"full"` and `[log]
   prompt_days` to `90` (`src/emit/match_log.rs` lines 22-29), written to `.base/match-log.jsonl`, and the hook log
   carries `prompt_text` too (`src/hook/mod.rs` line 722). A scrub removes common key and token shapes first
   (`src/scrub.rs` lines 1-25). **Less:** `prompt_text = "matched"` or `"off"`.
6. **The dashboard, when started, listens on every interface with no authentication.** `base dashboard` is opt-in;
   it binds `0.0.0.0:3741` (`src/dashboard/server.rs` lines 191-199, port from `cli.rs` line 367) with
   `CorsLayer::permissive()` (line 188), has write routes including `POST /api/domains/rule` (lines 137-189), and
   streams the hook log, prompt text included (line 186). No setting for the bind address was found.
7. **Text the tool injects into sessions**, each on by default unless marked: a *"CORRECTED:"* line and correction
   nudges (`[corrections] enabled`, `src/corrections/claude_md.rs` line 14, `src/corrections/mod.rs` line 47);
   front-matter guidance on every Markdown write (`src/hook/pre_tool_use.rs` lines 252-263); seeded engineering
   standards (`[standards] enabled`, `config.rs` lines 702-721); relay watcher nudges (`[relay] enabled`,
   `wake_nudge`, or `BASE_NO_WAKE_NUDGE`; `config.rs` lines 983-998). Opt-in only: `[flow]`, `[devmode]`, and
   `[memory] mode = "base"`, which blocks Claude Code's own memory writes (`src/hook/memory.rs` lines 98-108).
8. **On a new version it repairs graph files across workspaces in the background** (`src/upgrade/mod.rs` lines
   269-288), gated on `[graph] auto_migrate`, default `true` (`config.rs` lines 1078, 1097).

What it does not do, as far as the review found: there is no network listener in the relay (`src/relay/`, file-based),
no POST, PUT or telemetry call anywhere in `src/` (every outbound call is a `ureq::get`), and nothing that writes
permission rules or bypasses Claude Code's permission checks.

## What we do instead

We have not installed it. If it is ever installed, these are set before the first session start (install with
`--skip-hooks`, edit, and only then let the hooks be added):

```toml
# ~/.base-gbl/base.toml
[update]
auto = false          # no background swap
[log]
prompt_text = "off"   # or "matched"
prompt_days = 30
[corrections]
enabled = false
[relay]
enabled = false
wake_nudge = false
[standards]
enabled = false
[graph]
auto_migrate = false  # no background cross-workspace repair on upgrade
[memory]
enabled = false       # the default; written down so a change shows
[flow]
enabled = false       # the default; written down so a change shows
```

- In `~/.claude/settings.json` `env`: `BASE_NO_AUTO_UPDATE=1` (it survives a workspace overlay or a `base.toml`
  that fails to parse) and `BASE_NO_WAKE_NUDGE=1`.
- Install flags `--no-starter-commands --no-corrections-line`, run without a terminal so nothing prompts.
- No `curl | sh`: build from this fork, with the update address (`src/update/mod.rs` line 34) and the
  `[update] auto` default (`src/config.rs` lines 1037-1039) patched, so an update can never pull unreviewed code.
- A real `python3` first on `PATH`: `base install` runs `python3 -m pip install -r requirements.txt` with 27
  unpinned grammar packages (`install.rs` lines 1652-1670; `scripts/ast/requirements.txt`).
- `base dashboard` is not run, or run only after binding it to `127.0.0.1` (`server.rs` line 191).
- `.base/` is added to a repository's `.gitignore` before `base scaffold` runs there.

## What was not reviewed

Most of `src/cli.rs`, `src/hook/user_prompt_submit.rs`, `src/dashboard/api.rs`, the corrections, relay, domain,
graph and shadow modules beyond the lines cited, the Python extractor under `scripts/ast/` that the hooks run,
`dashboard/src/**` and whether the committed `dashboard/dist/` bundle matches it, `install.ps1`, most tests and CI,
and the separate desktop app the source refers to. The full list, by path, is in the review.

## How code moves

```text
upstream  →  this fork, at the reviewed pin  →  a private working copy (not installed)
```

- This fork moves to a new upstream revision **only after that revision is reviewed** the same way; the table above
  changes in the same commit.
- Nothing updates itself: no scheduled sync, no auto-merge, and, if it is ever installed, the tool's own updater off.

## Our changes in this fork

| Change | Commits |
|---|---|
| A short note at the top of `README.md` pointing here | the commit that adds this page |
| This page | the commit that adds it |

<sub>This page follows the fork front-door rule in [`my-template`](https://github.com/drasticstatic/my-template/blob/main/workflow-templates/FORK-FRONT-DOOR.md).
Written 2026-11-12 by Alfred (Claude Code CLI) for drasticstatic.</sub>
