# VertaaUX CLI Migration Guide

## Upgrading to v0.5.0

This release adds the interactive terminal app, credential file support, and a major test infrastructure overhaul. One behavior change: running `vertaa` with no arguments now launches an interactive app instead of printing help — see below for migration.

### New: Interactive App

Running `vertaa` with no arguments now launches a full-screen interactive menu. If you have scripts that run `vertaa` with no args and expect help output, add `--help` explicitly:

```bash
# Before (printed help)
vertaa

# After (launches interactive app)
vertaa

# To get help output in scripts
vertaa --help
```

### New: Credential File Fallback

The CLI now reads API keys from `~/.vertaaux/credentials.json` when neither `VERTAAUX_API_KEY` nor `--api-key` is set. This file is created by `vertaa login`.

Auth resolution order: `VERTAAUX_API_KEY` env var → `apiKey` in config file (`.vertaaux.yml`) → `~/.vertaaux/credentials.json`.

If you relied on the CLI failing when no env var was set (e.g., to detect missing CI configuration), be aware that stored credentials on the machine may now be used as a fallback. To ensure no credentials are used, point `HOME` (or `USERPROFILE` on Windows) to an empty directory.

### New: Error Rendering

Unhandled errors in command handlers now produce branded error boxes on stderr with a `vertaa doctor` suggestion, instead of raw stack traces. Exit codes are unchanged (still 2 for errors).

### Dependency Changes

| Dependency | Before | After |
|-----------|--------|-------|
| `commander` | ^12.x | ^14.0.2 |
| `@vertaaux/tui` | — | bundled |
| `node-pty` | — | ^1.1.0 (dev) |
| `vitest` | ^3.x | ^4.0.15 (dev) |

### Security Hardening

- Credentials file is rejected if it's a symlink
- Warning emitted if credentials file permissions are too open (non-Windows)

These checks are transparent — no action needed unless you store credentials as symlinks.

## Upgrading to v0.4.0

### Added

- Localhost and private URL auditing via static HTML analysis
- `whoami` shows name, email, and plan tier

No breaking changes.

## Upgrading to v0.2.0

This release includes several breaking changes from the CLI hardening phases (36-39). Most changes improve correctness and safety -- existing scripts using standard patterns will work without modification.

### Breaking Changes

#### 1. Format Flag Scope Changed

**Before:** `--format` was a global flag applied to all commands.

**After:** `--format` is a per-command option with different allowed values per command.

| Command | Allowed Formats | Default |
|---------|----------------|---------|
| `audit` | `human`, `json`, `sarif`, `junit`, `html` | `human` |
| `comment` | `json`, `markdown` | `markdown` |
| `explain` | `human`, `json` | `human` |
| `policy show` | `json`, `yaml` | `yaml` |
| `diff` | `human`, `json` | `human` |

**Migration:**

Move `--format` from before the command to after it:

```bash
# Before
vertaa --format json audit https://example.com

# After
vertaa audit https://example.com --format json
```

Using an unsupported format for a command now exits with code 2:

```bash
vertaa comment --format sarif  # Error: Invalid format "sarif" for "comment"
```

#### 2. JSON Output Wrapped in Envelope

**Before:** JSON output was a raw data object.

**After:** JSON output is wrapped in an envelope containing metadata.

```json
// Before
{
  "scores": { "overall": 85 },
  "issues": [...]
}

// After
{
  "meta": {
    "version": "0.3.2",
    "timestamp": "2026-02-08T12:00:00.000Z",
    "command": "audit",
    "args": ["https://example.com", "--format", "json"]
  },
  "data": {
    "scores": { "overall": 85 },
    "issues": [...]
  }
}
```

**Migration:**

Update JSON parsers to access `.data` instead of the top level:

```bash
# Before
cat results.json | jq '.scores.overall'

# After
cat results.json | jq '.data.scores.overall'
```

```javascript
// Before
const result = JSON.parse(output);
console.log(result.scores.overall);

// After
const envelope = JSON.parse(output);
console.log(envelope.data.scores.overall);
// Bonus: envelope.meta.version, envelope.meta.timestamp available
```

#### 3. Diagnostic Output Moved to stderr

**Before:** Banners, progress indicators, and diagnostic messages were mixed into stdout.

**After:** Only format output goes to stdout. All diagnostics go to stderr.

**Migration:**

If you were parsing stdout and filtering out banners, you can now pipe directly:

```bash
# Before (required filtering)
vertaa audit https://example.com --format json 2>/dev/null | grep -v "VertaaUX" | jq .

# After (clean piping)
vertaa audit https://example.com --format json | jq .
```

If you were capturing stderr for errors, note that progress and banner output now also goes to stderr.

#### 4. Strict Input Validation

**Before:** Invalid flag values were silently accepted or caused confusing runtime errors.

**After:** Invalid values are rejected immediately with exit code 2 and clear error messages.

```bash
# Numeric validation
vertaa audit https://example.com --timeout abc
# error: expected a number, got "abc"
# Exit code: 2

# Enum validation with suggestions
vertaa audit https://example.com --mode depp
# error: invalid value for --mode
#   hint: Did you mean "deep"?
#   valid: basic, standard, deep
# Exit code: 2

# Range validation
vertaa audit https://example.com --threshold 150
# error: expected 0-100, got 150
# Exit code: 2
```

**Migration:**

No action needed unless your scripts intentionally pass invalid values. Scripts that were relying on invalid values being silently ignored will now fail with exit code 2.

#### 5. Branch Name Sanitization

**Before:** Any string was accepted as a branch name in `--base-branch`.

**After:** Branch names are validated against `/^[a-zA-Z0-9._\/-]+$/` with a maximum length of 255 characters. Shell metacharacters are rejected.

**Migration:**

No action needed for standard git branch names. If you have branches with unusual characters, they will be rejected.

```bash
# These work fine
vertaa audit --incremental --base-branch main
vertaa audit --incremental --base-branch feature/my-feature
vertaa audit --incremental --base-branch release/v1.2.3

# These are now rejected
vertaa audit --incremental --base-branch "main; rm -rf /"
vertaa audit --incremental --base-branch 'feat$(whoami)'
```

### New Features

| Feature | Description |
|---------|-------------|
| `vertaa doctor` | Diagnose CLI health (config, auth, network connectivity) |
| `--config <path>` | Explicit config file path (overrides auto-detection) |
| `--machine` | Strict machine-readable output mode (JSON stdout, diagnostics stderr) |
| Levenshtein suggestions | Typo corrections for enum flags (e.g., `depp` suggests `deep`) |
| Branded errors | Box-drawn error frames with flag/value context and help hints |
| `--force` on init | Overwrite existing `.vertaaux.yml` configuration |
| `--yes` on init | Skip interactive prompts, use defaults |

### Exit Code Changes

| Code | Before | After |
|------|--------|-------|
| `0` | Success | Success (unchanged) |
| `1` | Issues found | Issues found (unchanged) |
| `2` | Error | Error AND validation errors (expanded) |
| `3` | - | Threshold breach (new) |

Exit code 2 now covers all validation errors (previously some caused exit code 1 or undefined behavior). Exit code 3 is new for `--threshold` breaches (previously used exit code 1).
