<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="banner-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="banner-light.svg" />
  <img src="banner-light.svg" alt="VERTAAUX" width="800" />
</picture>

[![npm](https://img.shields.io/npm/v/%40vertaaux%2Fcli?label=%40vertaaux%2Fcli)](https://www.npmjs.com/package/@vertaaux/cli)
[![npm downloads](https://img.shields.io/npm/dw/%40vertaaux%2Fcli?label=weekly%20installs)](https://www.npmjs.com/package/@vertaaux/cli)
[![Homebrew](https://img.shields.io/badge/homebrew-vertaaux%2Ftap%2Fvertaa-f08d49)](https://github.com/VertaaUX/homebrew-tap)
[![Docker](https://img.shields.io/badge/ghcr.io-vertaaux%2Fcli-2496ed?logo=docker&logoColor=white)](https://github.com/VertaaUX/cli/pkgs/container/cli)
[![Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-vertaaux--audit-24292f?logo=github)](https://github.com/marketplace/actions/vertaaux-audit)
[![License](https://img.shields.io/github/license/VertaaUX/cli)](LICENSE)

**UX, accessibility, and conversion audits from the terminal and CI/CD — with score-based quality gates, PR comments, and regression detection.**

[Website](https://vertaaux.ai) · [Docs](https://vertaaux.ai/docs) · [GitHub Action](https://github.com/marketplace/actions/vertaaux-audit) · [MCP Server](https://github.com/VertaaUX/mcp-server) · [Agent Skills](https://github.com/VertaaUX/agent-skills)

</div>

---

## Install

Pick your package manager:

```sh
# npm
npm install -g @vertaaux/cli

# Homebrew (macOS / Linux)
brew install vertaaux/tap/vertaa

# Docker (CI / no Node toolchain)
docker run --rm ghcr.io/vertaaux/cli --help
```

## Quick Start

```sh
vertaa audit https://example.com
```

First run walks you through a welcome tour, optional telemetry consent, and drops you in an interactive menu. 3 guest audits per day without an account — [get an API key](https://vertaaux.ai/settings/api) to unlock metered audits, deep mode, and baselines.

```sh
vertaa audit https://example.com --mode deep --fail-on error --baseline .vertaaux/baseline.json
```

Full command reference, config schema, and CI integration guides: **[vertaaux.ai/docs](https://vertaaux.ai/docs)**.

## What it does

- **7 audit categories** — usability · clarity · information architecture · accessibility · conversion · semantic · keyboard
- **axe-core + WCAG heuristics** for accessibility checks
- **Score-based quality gates** — fail CI when scores drop below thresholds
- **Regression detection** — compare against a baseline, comment on PRs
- **Machine-readable output** — JSON, SARIF, JUnit, HTML
- **Works on localhost** — point at `http://localhost:3000` from your dev machine (unmetered for zero-paid-dep audits)
- **AI-powered fix suggestions** — ready-to-paste HTML/CSS with impact scores and effort estimates

## CI/CD

### GitHub Actions

```yaml
- uses: vertaaux/audit-action@v1
  with:
    url: https://example.com
    api-key: ${{ secrets.VERTAAUX_API_KEY }}
    threshold: 80
    fail-on-critical: true
```

Marketplace: https://github.com/marketplace/actions/vertaaux-audit

### Any CI (Docker)

```yaml
- run: |
    docker run --rm \
      -e VERTAAUX_API_KEY=$VERTAAUX_API_KEY \
      ghcr.io/vertaaux/cli audit https://staging.example.com --mode standard
```

Works unchanged on GitLab CI, CircleCI, Buildkite, Jenkins, Azure Pipelines, and self-hosted runners.

## Distribution

| Channel | Install | Use |
|---|---|---|
| **npm** | `npm install -g @vertaaux/cli` | Local dev + any Node-capable CI |
| **Homebrew** | `brew install vertaaux/tap/vertaa` | macOS / Linux developer machines |
| **Docker (GHCR)** | `docker pull ghcr.io/vertaaux/cli` | CI without a Node toolchain |
| **GitHub Action** | `uses: vertaaux/audit-action@v1` | GitHub-native CI |
| **MCP Server** | `npx @vertaaux/mcp-server` | Claude Desktop / Cursor / IDE agents |
| **Agent Skills** | `npx skills add VertaaUX/agent-skills` | Coding-agent workflows |
| **JS SDK** | `npm install @vertaaux/sdk` | Programmatic API access |

## Feedback

This repo is the public home for the CLI — file issues, request features, and track releases here. Source lives in a downstream monorepo; we triage and port fixes through.

- **Bugs** → [Issues](https://github.com/VertaaUX/cli/issues)
- **Feature requests** → [Issues](https://github.com/VertaaUX/cli/issues)
- **Docs corrections** → [vertaaux.ai/docs](https://vertaaux.ai/docs)

## License

MIT © VertaaUX
