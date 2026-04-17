<div align="center">

# VertaaUX CLI

**UX, accessibility, and conversion audits from the terminal and CI/CD.**

[![npm version](https://img.shields.io/npm/v/@vertaaux/cli?style=flat-square)](https://www.npmjs.com/package/@vertaaux/cli)
[![npm downloads](https://img.shields.io/npm/dw/@vertaaux/cli?style=flat-square)](https://www.npmjs.com/package/@vertaaux/cli)
[![license](https://img.shields.io/npm/l/@vertaaux/cli?style=flat-square)](LICENSE)

[Website](https://vertaaux.ai) · [Docs](https://vertaaux.ai/docs) · [GitHub Action](https://github.com/marketplace/actions/vertaaux-audit)

</div>

---

## Install

Choose your package manager:

```sh
# npm
npm install -g @vertaaux/cli

# Homebrew (macOS/Linux)
brew install vertaaux/tap/vertaa

# Docker
docker run --rm ghcr.io/vertaaux/cli --help
```

## Quick start

```sh
vertaa audit https://example.com
```

First run walks you through a welcome tour, optional telemetry consent, and drops you in an interactive menu. 3 guest audits per day without an account.

Full command reference, config schema, and CI integration guides: **[vertaaux.ai/docs](https://vertaaux.ai/docs)**.

## What it does

- **UX audit** — usability, clarity, information architecture, conversion, accessibility, semantics, keyboard nav
- **Accessibility checks** — axe-core + WCAG heuristics
- **Score-based quality gates** — fail CI if scores drop below thresholds
- **Regression detection** — compare against a baseline, comment on PRs
- **Machine-readable output** — JSON, SARIF, JUnit, HTML
- **Works on localhost** — point at `http://localhost:3000` from your dev machine

## CI/CD

### GitHub Actions

```yaml
- uses: vertaaux/audit-action@v1
  with:
    url: https://example.com
    api-key: ${{ secrets.VERTAAUX_API_KEY }}
```

Marketplace: https://github.com/marketplace/actions/vertaaux-audit

### Any CI (Docker)

```yaml
- run: |
    docker run --rm \
      -e VERTAAUX_API_KEY=$VERTAAUX_API_KEY \
      ghcr.io/vertaaux/cli audit https://staging.example.com --mode standard
```

## Distribution

| Channel | Install command |
|---|---|
| **npm** | `npm install -g @vertaaux/cli` |
| **Homebrew** | `brew install vertaaux/tap/vertaa` |
| **Docker** (GHCR) | `docker pull ghcr.io/vertaaux/cli` |
| **GitHub Action** | `uses: vertaaux/audit-action@v1` |
| **MCP Server** (Claude/IDE) | `npx @vertaaux/mcp-server` |
| **JS SDK** | `npm install @vertaaux/sdk` |

## Issues & feedback

This repo is the public home for the CLI — file issues here. The implementation lives in a monorepo; we'll triage and port fixes downstream.

- **Bug reports** → [Issues](https://github.com/VertaaUX/cli/issues)
- **Feature requests** → [Issues](https://github.com/VertaaUX/cli/issues)
- **Docs corrections** → [vertaaux.ai/docs](https://vertaaux.ai/docs)

## License

MIT © VertaaUX
