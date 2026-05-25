# Changelog

All notable changes to `@vertaaux/cli` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.8.0-alpha.1] - 2026-05-25

> **Alpha release on the `next` tag.** The stable `latest` tag remains at `0.6.2`. Install this alpha via `npm install -g @vertaaux/cli@next` or pin to `@vertaaux/cli@0.8.0-alpha.1`.

### Added

- `vertaa explain <findingId> --copy` — write the recommended fix for a specific issue to stdout, pipe-clean for clipboard tools. Use `vertaa explain X --copy | pbcopy` (macOS) or `| xclip -i` (Linux) to send the fix directly to your clipboard. Evidence rendering, step-list output, and the "Fix written to stdout" hint all go to stderr so they remain visible in your terminal without polluting the pipe. Emits a `finding_fix_copied` telemetry event.
- Interactive fix-wizard (`vertaa audit … --interactive`) gains a "Copy fix to stdout" menu choice. Selecting it copies the current finding's recommended fix and stays on the same finding so you can also baseline or accept after copying.

### Fixed

- CLI builds and runs cleanly again. A stale internal alias in the copy-fix command broke `npm run build` and cascaded into 240+ downstream test failures during the previous alpha cycle. The whole cascade is gone.
- ANSI-colored error frames now render correctly in TTY mode for the `vertaa audit` "URL is required" error and for `vertaa explain --copy` missing-argument validation.
- A long-standing mock-drift in 13 internal command tests is closed out, lifting the suite to 145/145 files green (2,371 tests pass, 4 PTY-only skipped on systems without `node-pty`).

### Notes

- This alpha skips 0.7.0-alpha.3 onward on the public mirror — earlier alphas were published to npm but not synced here. The diff against the last public version (`0.6.2`) is documented above in the Added + Fixed sections.

## [0.6.2] - 2026-05-05

### Security

- **CWE-59: `vertaa upload` followed symbolic links inside `.vertaaux/artifacts/`.** Reported externally. The artifact collector used `fs.statSync` (which follows symlinks) and `fs.readFileSync` to read every file in the most-recent run directory. A malicious repo could ship `.vertaaux/artifacts/<job-id>/result.json` as a symlink pointing at `~/.ssh/id_rsa`, `~/.aws/credentials`, or any other file the user can read; running `vertaa upload` in that directory would read the symlink target and include it in the artifacts payload sent to the user's VertaaUX cloud account. Caveat on the data path: the upload target is the user's own VertaaUX account (not an attacker URL), so direct exfiltration requires a separate account compromise. This is a privacy / least-privilege violation more than a one-step credential-leak. Same general shape as the `.env` Trojan-repo issue fixed in v0.6.1, but with a narrower trigger (must run `vertaa upload`, not any command). Replaced `statSync` with `lstatSync` and rejected symlinks at both call sites: directory traversal (when picking the most-recent run) and per-file collection. Mirrors the existing `lstatSync` + `isSymbolicLink` defense in `src/auth/token-store.ts`. A stderr warning (`Warning: skipped N symlink(s) in .vertaaux/artifacts/...`) surfaces any rejected entries so the user can investigate.

### Migration

- No action required for normal users: artifacts are written by the action as regular files, and skipping symlinks is silent in the happy path.
- If you intentionally symlinked artifacts into `.vertaaux/artifacts/` for some local workflow (rare), those entries will now be skipped with a warning. Move the actual files into the directory instead.

## [0.6.1] - 2026-05-05

### Security

- **CWE-829: Trojan-repo `.env` exfiltration of API key.** Reported externally. The CLI loaded `.env` and `.env.local` from the current working directory at startup, before any command ran. A malicious repo could ship a `.env` setting `VERTAAUX_API_BASE=https://attacker.example`, and any user who cloned the repo and ran `vertaa whoami` (or any other command) inside it would silently send their API key to the attacker. Removed CWD candidates from the env loader. The new trusted candidate list is `~/.vertaaux/.env` (per-user CLI config) plus the package-relative paths inside the installed npm package (read-only after install). As a defense-in-depth backstop, `VERTAAUX_API_BASE` and `VERTAAUX_API_KEY` are now blocklisted from being injected by any `.env` file; they must come from the shell environment or `~/.vertaaux/credentials.json`. Shell-set values continue to win (`override: false` semantics preserved).

### Migration

- If you previously kept a `.env` in a project directory to point the CLI at a staging API, move that to `~/.vertaaux/.env` for non-sensitive vars (log level, theme, etc.), or set sensitive vars in your shell:
  ```bash
  VERTAAUX_API_BASE=https://staging.vertaaux.ai vertaa whoami
  ```
- Most users are unaffected: credentials live in `~/.vertaaux/credentials.json`, and the default API base is correct out of the box.

## [0.6.0] - 2026-04-17

### Added

- **Anonymous demo mode.** Run `vertaa audit <url>` without an account — 3 free audits per day (IP-based quota). Demo mode uses a reserve-then-settle charging model; zero-paid-dep audits refund the reserved slot.
- **Localhost audits in demo mode.** CLI captures local pages via CDP (connects to user's Chrome) and proxies AI analysis through `POST /api/v1/audit/ai-proxy`. Free-path localhost audits are unmetered.
- **First-run welcome screen.** Bare `vertaa` invocation shows a welcome screen → telemetry consent prompt → interactive menu. Re-run anytime via `vertaa welcome`.
- **Telemetry commands.** `vertaa telemetry enable|disable|status` to manage opt-in client-side telemetry. 8 event shapes (`cli_started`, `menu_opened`, `audit_invoked`, `audit_succeeded`, `audit_failed`, `quota_hit`, `login_started`, `login_completed`) sent to `/api/analytics/events` with consent gating and exit flush.
- **429 one-keystroke login.** When quota is exceeded in TTY mode, prompts `L` to log in via device flow and automatically retries the original request. Non-TTY emits structured JSON to stderr + exit 2.
- **Interactive menu enhancements.** Left/right arrow navigation (Finder-style drill/back), blinking `▎` cursor in search mode, `›` gutter marker for selected items (visible in `--plain` mode), `?` as help shortcut alias for F1, Esc exits TUI at top-level, breadcrumb trail ("Menu → Category"), deduplicated footer hints.

### Fixed

- **SECURITY: `--dry-run` leaked `apiKey` in JSON output.** Fields matching `/key|token|secret/i` are now truncated to 8 chars + `****`.
- **`--dry-run` bypassed URL validation.** Now validates URL (scheme, hostname, port) before serialization — rejects `javascript:`, `file:///`, hostnames without dots, ports > 65535. Exit 2 on failure.
- **Global `--machine`/`--dry-run`/`--verbose` rejected in post-subcommand position.** Universal globals now propagated to all subcommands after Commander.js registration.
- **`whoami`/`doctor`/`login`/`logout` showed "Audit Complete" heading in non-TTY output.** Now uses mode-specific headings (`"Whoami Complete"`, etc.) and omits `score=`/`issues=` for non-audit modes.
- **Welcome Tour crashed TUI when selected from interactive menu.** Menu now renders welcome text inline via `writeOutput()`.
- **Unknown root flags silently accepted.** Restored `isInteractive()` + `args.length === 0` guard.
- **Duplicate "Audit Complete" heading in non-TTY audit output.** `formatHeader()` now skips heading when `process.stderr.isTTY` is false.
- **Demo mode error handling.** Corrected 429 error parsing, fixed metadata leak bypassing free-tier gating, fixed hard-coded "completed" status on failed audits, added per-mode timeout enforcement.

## [0.5.3] - 2026-04-10

_Patch release — dependency updates and build fixes._

## [0.5.1] - 2026-04-07

### Added

- **`--profile <name>` flag on `vertaa audit`** — Phase 1.5 of audit profiles is now live on npm. `vertaa audit <url> --profile wcag-aa` (and any profile with a `categories` subset) skips auditors in the cloud worker rather than running all 7 and filtering post-run. Built-in profiles: `wcag-aa`, `conversion-focus`, `quick-ux`, `ci-gate`, `compliance`. The CLI passes `profile.categories` in the POST body via `apiRequest` when a category subset is resolved; the SDK path is used when no subset applies. See `cli/tests/commands/audit-handler.test.ts` for coverage.
- **`a11y` command upgrade** — `vertaa a11y <url>` now calls the dedicated `/v1/a11y/audit` endpoint with multi-engine analysis (axe-core + AccessLint + custom analyzers). Added `--mode`, `--fail-on-score`, `--min-impact`, and `--fail-on-findings` flags.
- **Source tracking** — All CLI-triggered audits now include `source: "cli"` metadata for dashboard visibility.

### Fixed

- **Profile-filtered categories now report `null` instead of a false `100`** — Excluded categories in `scores` are `null` (not a fabricated "perfect 100" from `calculateScore([])`). `metadata.filtered_categories` lists profile exclusions; `metadata.skipped_checks` is reserved for runtime failures. `metadata.partial` stays `false` for profile-filtered audits. Server-side fix shipped in PR #375; this release gets it onto the npm-published CLI output path.

### Changed

- **SDK upgraded to `@vertaaux/sdk@2.0.0`** — Updated to match v2 API surface with snake_case parameter conventions.
- **Machine-mode hardening** — Payload output routes consistently through the data writer; enveloped JSON input is unwrapped uniformly across `baseline`, `comment`, and `diff` commands.

## [0.5.0] - 2026-03-19

### Added

- **Interactive terminal app** — full-screen menu-driven interface with keyboard navigation, live dashboard, and canvas layout (launched via `vertaa` with no args)
- **@vertaaux/tui package** — terminal UI primitives (spinner, table, progress, box, error-box, step-list, viewport) bundled into the CLI
- **PTY test infrastructure** — node-pty based terminal tests with ANSI serializers, xterm-256color emulation, and dedicated vitest config
- **Subprocess test infrastructure** — process-level E2E tests with build caching, mock audit server, and contract validation
- **Snapshot test infrastructure** — long-running compilation snapshot tests with NO_COLOR determinism
- **Package install tests** — artifact validation via `npm pack` and tarball installation
- **Cross-platform CI matrix** — Ubuntu/macOS/Windows × Node 20/22 in `cli-e2e.yml` workflow
- **1953 unit tests** across 112 test files covering commands, auth, config, output, security, caching, CI detection, monorepo support, and quality gates
- **Credential file support** — reads API key from `~/.vertaaux/credentials.json` as fallback (with symlink and permission checks)
- **Error propagation improvements** — unhandled errors in command handlers now produce exit code 2 with branded error boxes

### Changed

- **Commander upgraded to v14** — from v12, with improved help formatting and error handling
- **Vitest upgraded to v4** — from v3, with `fileParallelism: false` replacing deprecated `poolOptions.forks.singleFork`
- **Error rendering** — all command errors now render through `renderError()` with suggestion hints and proper exit codes

### Fixed

- **Flaky TTY detection** in subprocess tests
- **Fetch resource leak** — unclosed responses in API client
- **Timer cleanup** — lingering timers in polling commands
- **ESLint compliance** — replaced `require()` calls with string concatenation for no-require-imports rule

### Security

- **Symlink check on credentials** — refuses to read `~/.vertaaux/credentials.json` if it's a symlink (SECVAL-1)
- **Permission check on credentials** — warns if file permissions are overly permissive on Unix systems (SECVAL-2)

## [0.4.0] - 2026-02-15

### Added

- **Localhost/private URL auditing** via static HTML analysis
- `whoami` command shows name, email, and plan

## [0.3.3] - 2026-02-10

### Fixed

- Documentation accuracy improvements

## [0.3.0] - 2026-02-08

### Added

- **AI Intelligence commands** — `suggest`, `triage`, `fix-plan`, `patch-review`, `release-notes`, `compare`, `doc`
- **Pipeline input** — all AI commands accept stdin pipe, `--file`, or `--job`
- Per-command format validation with format registry
- JSON output envelope with metadata
- `--machine` flag for strict machine-readable output
- Branded error messages with typo suggestions
- `doctor` command for CLI health diagnostics
- Exit code 3 for threshold breaches

### Changed

- `--format` moved from global to per-command option
- Diagnostic output moved to stderr (stdout reserved for format output)
- Strict input validation with exit code 2

## [0.2.0] - 2026-01-25

### Added

- Per-command format system
- JSON output envelope
- `--machine` flag
- `doctor` command
- Levenshtein suggestions for enum flags
- Branch name validation
- Artifact path traversal protection

### Changed

- Breaking: `--format` is now per-command
- Breaking: JSON output wrapped in envelope
- Breaking: diagnostics moved to stderr
- Breaking: strict input validation (exit code 2)
- Breaking: exit code 3 for threshold breach (was 1)

[0.6.0]: https://github.com/PetriLahdelma/vertaa/compare/cli-v0.5.3...cli-v0.6.0
[0.5.3]: https://github.com/PetriLahdelma/vertaa/compare/cli-v0.5.1...cli-v0.5.3
[0.5.0]: https://github.com/PetriLahdelma/vertaa/compare/cli-v0.4.0...cli-v0.5.0
[0.4.0]: https://github.com/PetriLahdelma/vertaa/compare/cli-v0.3.3...cli-v0.4.0
[0.3.3]: https://github.com/PetriLahdelma/vertaa/compare/cli-v0.3.0...cli-v0.3.3
[0.3.0]: https://github.com/PetriLahdelma/vertaa/compare/cli-v0.2.0...cli-v0.3.0
[0.2.0]: https://github.com/PetriLahdelma/vertaa/releases/tag/cli-v0.2.0
