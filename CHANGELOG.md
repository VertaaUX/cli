# Changelog

All notable changes to `@vertaaux/cli` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.8.1] - 2026-05-25

First stable release on the 0.8 line, replacing the 0.6.2 stable line on the `latest` npm tag. Consolidates every change since the May 5 `0.6.2` release into a production-grade graduation. Skips the 0.7.0-alpha.{1,2,3} and 0.8.0-alpha.{0,1,2} pre-release tags that lived on the `next` channel during May.

### Added

#### DX foundations (was 0.7.0-alpha.1)
- **`vertaa demo`** — replay a fixture audit through the live renderer offline, no API required.
- **`vertaa completion <bash|zsh|fish>`** — shell completion scripts. Install with `vertaa completion bash > /usr/local/etc/bash_completion.d/vertaa` or equivalent.
- **`vertaa upgrade`** — self-update to the latest npm version.
- **Framework auto-detect in `vertaa init`** — `.vertaaux.yml` is now populated with sensible defaults inferred from your project (Next.js, Astro, Vite, Storybook, etc.).
- **31% smaller npm tarball** — packaging cleanup.

#### Live dashboard (was 0.7.0-alpha.2)
- **`renderDashboardV2()`** — animated 7-cell score grid, per-category sparklines (`▁▂▃▄▅▆▇█`), live-issues feed (top 5 severity-colored), proportional severity histogram, elapsed-time footer.
- **`vertaa demo` upgraded to v2** — synthesized score history and live-issues feed for a realistic replay.

#### Fuzzy palette, watch mode (was 0.7.0-alpha.3)
- **Ctrl+P opens the command palette** — fuzzy search across every command, with category tags.
- **`vertaa audit --watch`** — re-runs the audit on filesystem changes with debounced trigger.
- **Animated score reveal** — score tweening in the dashboard for a smooth final-result transition.

#### Phase 147 CLI Package Isolation (was 0.8.0-alpha.1)
- **`vertaa explain <findingId> --copy`** — writes the recommended fix for a specific finding to stdout, pipe-clean for clipboard tools. Use `vertaa explain X --copy | pbcopy` (macOS) or `| xclip -i` (Linux). Evidence rendering, step-list output, and the "Fix written to stdout" hint all route to stderr so they stay visible without polluting the pipe. Emits a `finding_fix_copied` telemetry event.
- **Interactive fix-wizard "Copy fix to stdout" choice** — sixth menu item in `vertaa audit --interactive`. Selecting it copies the current finding's recommended fix and stays on the same finding so you can also baseline or accept after copying.
- **Type mirror at `cli/src/types/audit.ts`** — canonical `Severity` + `AuditCategoryKey` unions are now mirrored into the CLI package boundary with a CI-gated drift check.

#### Production-grade boundary normalization (was 0.8.0-alpha.2 + 0.8.1 prep)
- **Synthetic `ruleId` hydration in audit output** — when the audit-engine emits issues without `ruleId` (current behavior on `lib/audit-engine/auditor-ia.ts`, `auditor-usability.ts`, `auditor-conversion.ts`), the CLI backfills stable IDs in the form `synthetic/<category>-<title-slug>-<index>`. Saved audit JSON now exposes addressable findings for `vertaa explain <id> --file file.json --copy`. Idempotent — real ruleIds (e.g. `custom/semantic-landmark` from the accessibility analyzer) pass through unchanged.
- **Severity vocab normalized to the canonical 5-tier union at the CLI boundary** — `critical | high | medium | low | info`. The production backend still emits legacy 3-tier (`error/warning/info`); the CLI maps at audit-write + explain-load boundaries so the changelog promise from 0.8.0-alpha.0 is now actually fulfilled in published output. The `--severity-legacy` flag remains available for users who need the old vocab in CI scripts.
- **Canonical `Issue` type at `cli/src/baseline/hash.ts`** now declares both snake_case AND camelCase variants for `recommendedFix`/`recommended_fix`, `wcagReference`/`wcag_reference`, `businessImpact`/`business_impact`, `impactScore`/`impact_score`, `estimatedEffort`/`estimated_effort` — matches the actual audit JSON wire shape.
- **`vertaa explain --file out.json` unwraps `{ data: { issues }, meta }`** — the envelope shape produced by `vertaa audit --format json`. Bare `{ issues: [] }` and flat-array shapes still work for backwards compat. No more `jq` reshape needed.

### Fixed

- **CLI builds and runs cleanly** — a stale internal alias in the copy-fix command broke `cd cli && npm run build` and cascaded into 240+ downstream test failures during the 0.7.x alpha cycle. Cascade is fully collapsed (0 failed test files / 0 failed tests).
- **ANSI-colored error frames in TTY mode** — `vertaa audit` "URL is required" error and `vertaa explain --copy` missing-argument validation now render branded `renderError` frames in TTY mode.
- **`process.std*.write()` violations in command files** — wrapped per the renderer-enforcement contract.
- **Pre-existing test debt closeout** — 13 internal command tests' `strings.js` mock drift fixed; PTY snapshots re-recorded for current help output (`--sarif`, `--ci` flags + new `completion`/`upgrade`/`demo` commands).
- **Typo `toLeqacySeverity` → `toLegacySeverity`** in `cli/src/lib/severity-compat.ts`.

### Migration

If you're upgrading from `0.6.2` (the previous stable):

- **Severity vocab changed.** `audit --format json` now emits `critical | high | medium | low | info` instead of the legacy `error | warning | info`. CI scripts that grep for legacy values should either (a) update their patterns or (b) use `vertaa audit --severity-legacy` for one transitional cycle. The `--severity-legacy` flag is documented for removal in the next CLI minor.
- **Audit JSON now wraps in `{ data, meta }`** — most CLI commands accept both wrapped + bare shapes (`explain --file` accepts both; `baseline --from-file` accepts both). External tools that pipe `vertaa audit --format json` outputs should read from `.data.issues` (not top-level `.issues`).
- **`baseline` fallback severity** for issues without an explicit severity field is now `"medium"` (was `"warning"` pre-0.8.0).
- **`vertaa fix` and `vertaa fix-all` `--issue` flag** is now conditionally required (only when `--dry-run` is absent). Pre-0.8.0 it was strictly required, which blocked `--dry-run` workflows.

## [0.8.0-alpha.0] - 2026-05-17

### Changed

- Severity vocabulary in `audit` output is now `critical | high | medium | low | info` (Phase 137 VOICE-02).
- Baseline manager fallback severity for issues without a severity field changed from `"warning"` to `"medium"`.

### Added

- `--severity-legacy` flag for the `audit` command. Emits the legacy 3-tier vocab on stdout for one transitional cycle. CI pipelines that grep for the old values can use this flag without rewriting. Removed in the next CLI minor.

### Migration

- Replace pattern matches on `"error"` with `"high"`, `"warning"` with `"medium"` in CI scripts and machine-readable output consumers. New `"critical"` values appear when a finding blocks primary user completion.

## [0.7.0-alpha.2] - 2026-05-06

Sprint 2 of the v0.7.0 milestone: **the live dashboard**. The static frame from v0.1.x has been replaced with a 7-cell score grid, sparklines, a streaming live-issues feed, and a severity histogram in the footer. Visible during `vertaa demo`; auto-switches in for any caller that populates the new state fields. Animated score reveal and live audit-pipeline wiring land in alpha.3 alongside Sprint 3.

### Added

- **`renderDashboardV2()` in `@vertaaux/tui`** — pure render function. Header box (URL + mode + clock), phase line with bullet progress, 7-cell score grid (one cell per audit category with score + 6-glyph sparkline), live-issues feed (top 5, severity-colored), severity histogram bar (proportional widths colored by tier), elapsed-time footer.
- **Sparkline rendering** — 8-band block-glyph sparklines (`▁▂▃▄▅▆▇█`) per category, padded on the left when history is short. Exported as `renderSparkline()` for reuse.
- **Optional dashboard state fields** — `categoryScores`, `scoreHistory`, `liveIssues`, `severityCounts`, `scoreReveal`. Backwards-compatible: callers on the old shape get the legacy frame; any caller that populates one of the new fields gets v2 automatically.
- **`vertaa demo` upgraded to v2** — populates `categoryScores`, synthesizes a smooth `scoreHistory` ramp toward the final value (sparklines look like real progress), wires in `liveIssues` from the fixture and `severityCounts` from `issues_summary`. The replay now shows the full new dashboard.

### Changed

- **`AlternateScreenRenderer.update()`** — auto-detects the new state shape (any of `categoryScores` / `liveIssues` / `severityCounts`) and renders v2; otherwise falls back to the legacy frame. No flag, no opt-in. Pure additive change.
- **Phase-dot rendering** — bullet count now matches `state.phaseTotal` (audit pipeline reports 9, demo reports 9, init reports 1) instead of always being the 4-step `PHASE_ORDER`.

### Internal

- 21 new vitest cases under `packages/tui/tests/dashboard/dashboard-v2.test.ts` covering: header/phase/footer structure, 7-up category grid (canonical order, label truncation, uncategorized append), sparkline glyph mapping (range 0–100, left-pad, clamp, empty history), live-issues feed (presence/absence, max-5, severity tags, ellipsis truncation), severity histogram (proportional widths, fallback when no breakdown, "no issues" state), and color-mode stability.
- Manually rendered against a 100×50 viewport with full state to verify visual layout.
- Full CLI test suite shows zero new failures vs `main` (4 pre-existing E2E failures predate this PR; tracked for S4).

## [0.7.0-alpha.1] - 2026-05-06

First alpha of the v0.7.0 "DX & The Dashboard" milestone, building toward a Product Hunt launch in early June. This release is **DX foundations**: shell completions, self-update, `vertaa demo`, framework auto-detect in `vertaa init`, and a 31% smaller npm tarball. Sprint 2 brings the live dashboard rewrite; Sprint 3 brings the fuzzy command palette and `--watch` mode.

### Added

- **`vertaa completion <bash|zsh|fish>`** — Shell-completion script generated from the live Commander program tree. Pipe the output to your shell's completion directory or `source <(vertaa completion bash)` inline. Walks every command, sub-command, alias, and flag automatically, so future commands get tab-completion for free.
- **`vertaa upgrade`** — Self-update with semver-aware comparison. Detects the install method (global npm / npx / project-local) and runs the right install command. `--check` exits non-zero on drift (CI-friendly), `--yes` skips the confirmation prompt for unattended use.
- **`vertaa demo`** — Replay a fixture audit through the live renderer (offline, ~14 seconds). Same dashboard pipeline a real audit uses, but with guaranteed-pretty data. Use it to demo VertaaUX without burning credits or waiting for a network. `--fast` skips delays for screenshots; `--machine` emits the fixture as JSON.
- **`vertaa init` framework auto-detect** — Sniffs Next.js, Vite, Astro, Remix, Nuxt, SvelteKit, Gatsby, Eleventy, Vue (CLI), React (CRA), and Hugo from `package.json` deps and config files; reads explicit dev-server ports out of `scripts.dev` (`--port`, `-p`, `PORT=`); pre-fills `defaultUrl` so the first audit a new user runs hits their own dev server, not `https://example.com`.

### Changed

- **31% smaller npm tarball.** New `tsconfig.build.json` strips `.d.ts`/`.d.ts.map` from the published package (the CLI is consumed as a binary, not a library). Tarball: 287 kB packed / 1.2 MB unpacked / 132 files (was 416 kB / 1.7 MB / 367 files).
- **Tightened `scripts/verify-package.mjs`.** Added file-shape gates (no source maps, no declaration files in `dist/`, no test files leaked) and hard caps on tarball file count (≤200) and unpacked size (≤1.5 MB). Publish is blocked if any gate trips.

### Internal

- 70 new tests across `completion`, `upgrade`, `demo`, and `detect-framework`. Full CLI suite shows zero new failures vs `main` (the 4 pre-existing E2E failures predate this PR; tracked for S4).
- Sensitive-write hygiene: `process.stderr.write` calls in command files now go through `renderError` / `renderWarning` per the structural test in `tests/pty/renderer-enforcement.test.ts`. Fixed a pre-existing violation in `upload.ts` along the way.

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
