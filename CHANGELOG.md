# Changelog

## [5.13.33] - 2026-10-08

- Up-to-date model prices: haiku_5_5, sonnet_5_5.

## [5.13.32] - 2026-10-04

- Every session now keeps its own checkpoints, so running several sessions at once no longer clears out the quiet ones. Applies on every platform that saves checkpoints through Token Optimizer's engine.
- The desktop status bar only shows a checkpoint that is still there to restore.
- Passes the stricter plugin check in Claude Code 2.1.289.

## [5.13.31] - 2026-10-04

- The desktop status bar comes alive again: Clawd follows what your session is doing, and the arrow, Clean up, Start fresh and Keep warm respond to a click.

## [5.13.30] - 2026-10-03

- The desktop status bar now comes with Token Optimizer itself: install or update the plugin and it appears above your prompt, nothing extra to add.
- Hide it any time with `TOKEN_OPTIMIZER_STATUS_BAR=0` in your settings `env`.
- Older Claude Code versions keep every Token Optimizer feature and simply skip the bar.

## [5.13.29] - 2026-10-03

- New: a Token Optimizer status bar for the Claude desktop app, as the separate `token-optimizer-desktop` plugin: `/plugin install token-optimizer-desktop@alexgreensh-token-optimizer`.
- Quality, context, cache countdown and both usage limits at a glance, with a one-click Clean up, Start fresh or Keep warm when you need one.
- One more line shows your session, its last checkpoint, and the tokens Token Optimizer saved you in the last 30 days.
- Clawd acts out what your session is doing. Needs Claude Code 2.1.287 or newer; the terminal status line is unchanged.
- The status line now counts compactions the moment they happen.

## [5.13.28] - 2026-09-30

- Up-to-date model prices: gpt-6.1-sol.

## [5.13.27] - 2026-09-29

- Up-to-date model prices: sonnet_5_5.

## [5.13.26] - 2026-09-27

- Keep detached dashboard self-heals single-flight for the entire rebuild, even on large histories, and record the rebuild child's PID.
- Back off repeated Stop-hook dashboard attempts for one hour after a successful background refresh, while retrying promptly after a failed refresh or version upgrade. Fixes #204.
- Keep OpenCode and OpenClaw core dashboard labels aligned with 5.13.26; correct the OpenClaw adapter fallback to its current 2.4.24 package version.

## [5.13.25] - 2026-09-27

- Add opt-in native Pi diagnostics, active-branch usage, local archives, and review-before-use continuity checkpoints.
- Keep OpenCode and OpenClaw core version labels aligned with the release. Fixes #201.
- Protect Pi's local data from shared directories and symlinked files, reject authorization and cookie headers in archives, and keep optional checkpoint failures from interrupting compaction.
- Bound Pi archives to 1,024 files and pin OpenCode's transitive TOML parser to a patched release.
- Reject malformed Cowork telemetry requests, keep concurrent capture records intact, read capture lines with a firm memory bound, skip bad records without blocking later sessions, reject negative or imprecise token counts, and remove stale Cowork copies on the next ingest after a Claude row for the same session arrives. Restrict capture paths, keep captures private, and stop storing authentication headers. Document protected HTTPS ingress for cloud senders.
- Exit with an error when Cowork ingestion writes only some database rows, including in quiet mode.
- Keep Codex hooks working when an optional launcher is missing, validate Cowork collector URLs, and keep test child processes from inheriting host secrets.
- Update docs and VS Code development dependencies to audited versions. Install release CI dependencies without lifecycle scripts.
- Bound VS Code transcript tail reads so opening a long session cannot allocate memory for the entire file.
- Label compact-instruction dry runs as previews.

## [5.13.24] - 2026-09-25

- **Savings report matches the dashboard.** Measured savings, repeat reads avoided, estimates, and one all-in total.
- **Exact rates for repeat reads** on every model, including Opus 5.5 and Fable 5.1.
- **Every number checked against known answers** on each release.

## [5.13.23] - 2026-09-25

- **Full savings in the report.** Measured savings now include the context you no longer re-read on every later turn, often the biggest share.
- **Your own baseline, pinned.** Savings compare against your sessions from before you installed Token Optimizer, and that baseline no longer drifts as old history ages out.
- **Weekly card always shows what was measured.**

## [5.13.22] - 2026-09-25

- **Accurate daily cost.** Each day on the dashboard shows exactly what that day's requests cost, subagents included. Thanks @asaarela-bw (#200).
- **Prices that stay current.** New Claude, OpenAI/Codex and Gemini models are priced automatically with each update. The plugin still makes no network calls.
- **Exact rates for Opus 5.5 and Fable 5.1** in every engine.
- **Clean dashboard launch from OpenCode.** (#199)
- Faster dashboard, tidier logs.

## [5.13.16] - 2026-09-17

- Fix: the Hermes context-fill nudge measured the session-CUMULATIVE input tally instead of the live
  prompt, so it reported a context emergency that did not exist. Every host re-sends the whole
  conversation on each turn, so that sum climbs past the model window regardless of real occupancy:
  on a 129-call Hermes session the cumulative figure reached 1,285,803 against a 1,000,000 window
  ("Context ~100% full ... Grade: F", the percentage being capped at 100) while Hermes itself
  reported 278,545 / 1,000,000 = 28% for the same session. The nudge now uses the prompt the last
  call actually sent -- the full prompt_tokens (fresh input + cache-read + cache-write), since every
  prompt token occupies the window -- and says so in the message ("last request prompt ~N tokens vs
  model window M"). The cumulative tally is unchanged for cost and usage reporting. Regression tests:
  `tests/test_hermes_context_fill_nudge.py` (3 of the 5 original cases fail on the previous code,
  plus a cache-write case so the live figure is not undercounted on a cache-creation turn).

- Fix: Bash cross-turn output dedup and the repeat-command thrash nudge were keyed only by session, so
  a subagent could be told its output was "identical to your previous output", or that a command "has
  run N times this session", for work only the main agent had done. Both are now scoped to session plus
  agent identity: a supplied agent id is treated as an opaque identity (never normalized or merged),
  while a missing, empty, or whitespace-only id falls back to the historical session-only identity so
  the main-agent case is unchanged. The thrash guard's edit-detection reads the shared session activity
  log, so a subagent that edits a file between two identical runs still suppresses the false "stuck in a
  loop" nudge. (issue #189)

- Fix: the Windows hook launcher could select an incomplete or unreadable runtime version. It now admits
  only versioned install dirs whose `run.py` actually opens, choosing the newest that does and skipping
  incomplete, unreadable, or zero-byte candidates before falling back to the baked install. This replaces
  a readability check that was a no-op on Windows and removes a case where an unreadable candidate could
  cause the wrong version to be selected on some Python versions. Argv, stdin, and environment
  forwarding are unchanged. (issue #188)

- Fix: `health` and `kill-stale` found no running sessions when Claude Code is launched by the Claude
  Desktop app under WSL2, because the desktop starts a versioned `ccd-cli` binary rather than a `claude`
  binary, so session detection never matched. Detection now also recognizes the ccd-cli launcher via an
  anchored, version-shaped path match on the process command, which excludes bundled helpers and
  unrelated processes so no false sessions are counted. (issue #192)

## [5.13.15] - 2026-09-16

- Fix: `codex_install.py` no longer emits a base64 `python -c` exec-bootstrap as the Windows hook command on versioned marketplace installs (issue #183). The encoded-exec string trips generic-loader antivirus signatures (SentinelOne flagged it as a Metasploit variant, once per shipped copy of the file), even though the payload was fixed, readable, and decode-auditable. The installer now copies a plain, auditable `windows-launcher.py` next to the versioned install dirs -- a stable path that survives marketplace upgrades -- and bakes a command that invokes it by quoted path with plainly quoted argv. Version resolution (newest semver sibling, fail-open to the baked install, TOKEN_OPTIMIZER_DEBUG-gated resolver log), stdin/argv/env passthrough, legacy-command recognition on reinstall/uninstall, and upgrade trust semantics are unchanged: the command signature now normalizes only the `--baked-root` version leaf, legacy base64 commands compare verbatim so the next install replaces them (the one-time review that ships this fix), and `--decode-launcher` still decodes legacy commands while pointing launcher-file commands at their plain source. Regression coverage: an AV-safe source-string guard (the generator and the launcher never contain or emit `exec(`/base64/bootstrap patterns), launcher install/idempotence/stability tests, cross-platform resolver execution tests, and cmd.exe execution tests on Windows CI.
- Fix: the Codex command-compression shell resolver no longer crashes on unreadable PATH entries. A directory the user cannot stat (e.g. another account's private `~/.cargo/bin` on a shared machine) raised PermissionError out of `_default_shell()` and failed the hook; unreadable entries are now skipped like any other non-match.
- Fix: the generated Windows hook command now force-quotes every path/value token. `list2cmdline` only wraps a token containing whitespace, so a space-free install path carrying a cmd.exe metacharacter (`& | < > ^`) was emitted bare and `cmd` parsed it as a command separator; every token is now quoted (cmd treats those as literal inside quotes). A `%` in the install path remains the one documented, unsupported case.

## [5.13.14] - 2026-09-14

- Fix: the Codex log-index now self-heals on Windows instead of crashing on a locked or corrupt database. The open path closes the broken connection before rebuilding (Windows refuses to unlink an open file, so the rebuild previously reconnected to the same corrupt DB), retries each unlink briefly to ride out transient share locks, and no longer misdiagnoses a merely-locked database as corruption and deletes it out from under a live process. Transient lock contention (concurrent first-opens racing journal-mode/schema setup, a writer mid-commit, or a lock-upgrade deadlock that returns BUSY without consulting the busy handler) now retries on a deadline at both the connect and write-transaction stages instead of surfacing "database is locked" to callers -- follow-up hardening on #175.
- Fix: the release installable check no longer races the signing workflow. A fresh release missing CHECKSUMS.sha256 is polled through the signing grace window (measured from published_at) instead of failing instantly -- v5.13.13's check ran at publish+2s while the asset landed ~20s later. An old unsigned release still fails immediately.
- Fix: Codex hooks on native Windows failed with "hook exited with code 1" on versioned marketplace installs. The generated cmd.exe command assigned TOKEN_OPTIMIZER_RUNTIME_ROOT inside a `for /f` loop and read it back with `!TOKEN_OPTIMIZER_RUNTIME_ROOT!` on the same line; cmd parses a /C line once and `setlocal EnableDelayedExpansion` only applies from the next line, so Python received the literal placeholder path. The runner path is now built from the FOR variable `%R` inside the do-body, and the version resolver always prints one directory (newest semver install, else the baked install) so the fallback still runs. Regression tests execute the generated command through `%COMSPEC% /D /C` with spaces in the install path. Reinstall and uninstall now also recognize those broken commands: they carry no `token-optimizer/scripts` path marker (backslash install paths and `hooks/<name>_runner.py` args), so `_is_token_optimizer_group` additionally matches the full signature of our generated command -- the quoted `TOKEN_OPTIMIZER_RUNTIME_ROOT=` assignment together with our `hooks\run.py` runner invocation under a token-optimizer path (or via our own FOR/delayed-expansion variable) -- so a user's own hook that merely references the env var is never swept up. When the version resolver falls back to the baked install it now appends a line to `token-optimizer-codex-resolver.log` next to the version dirs if `TOKEN_OPTIMIZER_DEBUG` is set, making the previously silent fallback observable.

- Fix: the token-saving hooks now reach every supported harness. Codex, Cowork, and manual installs get the same savings as Claude Code -- startup diagnostics stay out of the model's context, and the command-failure and long-output nudges reach the model through each host's supported channel.
- Fix: SessionStart no longer adds anything to the model's context. Startup diagnostics (health checks, dashboard setup, daemon status) now write to a local log file instead of stdout/stderr, both of which the host captures into the session context. Sessions begin at their true baseline, so the token savings start on the first turn.
- Add: burn nudge. When the same command fails 3 times in a row with different output, a nudge suggests changing approach instead of re-running. Catches the edit-compile-fail cycle that the existing identical-output streak guard cannot see. Tunable with `TOKEN_OPTIMIZER_FAIL_STREAK_THRESHOLD` (default `3`).
- Add: inline-script repeat nudge. When a command with a heredoc body >= 300 chars has been run 8 times in a session, a nudge suggests saving the script to a file and running that instead, so the body is not re-sent as input tokens every turn. Tunable with `TOKEN_OPTIMIZER_INLINE_SCRIPT_THRESHOLD` (default `8`).

## [5.13.13] - 2026-09-14

- Add: first-class Codex support, extracted and hardened from external PR #175 by @dormancygrace. Codex sessions get real model pricing (gpt-6-astra, gpt-5.6-sol) through a versioned model catalog, delta-based token accounting that stops the over-count on incremental log writes, canonical session-ids, a SQLite log-index for fast session discovery, native Windows process handling, a security-hardened command-compression hook, and a base64 Windows launcher that survives cmd.exe quoting.
- Add: Codex Token-Coach port with full runtime isolation. Context-window detection, model config, and savings accounting now resolve per-runtime across Claude, Codex, and the five other supported harnesses, so a foreign runtime never inherits Claude's model env vars, ~/.claude config, or the 1M default.
- Fix: coach-integrity hardening across the measurement pipeline. Safe-int/type/size guards reject malformed session records, ANSI/VT escape sequences are stripped before terminal output, log-index reads are confined to the runtime home, concurrent writers no longer corrupt the index, and the consent gate narrows its exception handling and logs unexpected errors so a corrupt config is visible, while still failing open by design.
- Docs: new benchmarks category (Overview, Terminal-Bench floor, Controlled A/B, One real month) with a refreshed real-savings.svg built from current measurements.

## [5.13.10] - 2026-09-08

- Prevent sandbox dashboard tests from replacing the real background service. Isolate hook test homes and cached modules, and keep capped transformation percentages and older marker history consistent.

- Include modeled repeat-read savings in the action card for removals made during the selected period. Retain logged setup, output, routing and unmatched-event savings without counting initial removals twice.
- Make Savings easier to scan: compact transformation and action summaries, explicit periods and estimates, matching percentage and dollar comparisons, and expandable methods that stay open during live refresh.
- Compare lifetime context savings with its matching period subtotal. Preserve previously verified history when transcripts rotate, and remove duplicate delta-read entries from the derived ledger.
- Retain other logged savings in the weekly fallback calculation and show the previously omitted concise-output estimate. Supported runtimes without repeat-read evidence keep their logged totals.

## [5.13.9] - 2026-09-08

- Fix growing session logs being skipped after their first collection. Refresh parent and child activity without duplicating totals or overwriting newer collector results.
- Show the full Savings-tab estimate for the actual subscription week, counting overlapping savings once and labeling the amount as estimated savings accrued so far. Include new sessions before background collection catches up.
- Recover the workload comparison after history backfill or rebuild changes its baseline month. Cache weekly results for up to 60 seconds and retain weekly dollars when quota readings are unavailable.
