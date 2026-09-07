# AGENTS.md

This file is intended for AI agents (Codex, Copilot, Cursor, etc.) that work on this repository. Read this first.

## What This Project Is

A white-label perpetual futures DEX frontend built on the Orderly Network SDK. Fork this repo, edit `.env`, and deploy your own branded trading UI.

Stack: React 18 + Vite + TypeScript + Tailwind CSS + Orderly JS SDK.

## Quick Start

```sh
yarn install
yarn dev        # starts at http://localhost:5173
yarn build      # production build to build/
yarn preview    # preview production build locally
```

## Configuration

**Everything is configured via `.env`.** The file is self-documenting — every variable has an inline comment explaining what it does, valid values, and defaults. Start there.

Key decisions when setting up:

### Required
- `VITE_ORDERLY_BROKER_ID` — your broker ID from Orderly. Use `"demo"` for sandbox.
- `VITE_ORDERLY_BROKER_NAME` — display name throughout the UI.

### Network
- By default users can toggle between mainnet and testnet.
- Set `VITE_DISABLE_MAINNET=true` or `VITE_DISABLE_TESTNET=true` to lock to one.
- Use `VITE_ORDERLY_MAINNET_CHAINS` / `VITE_ORDERLY_TESTNET_CHAINS` to whitelist specific chains (comma-separated chain IDs).

### Wallet Connector
- **Default**: Built-in wallet connector (MetaMask, Phantom, etc.). Set `VITE_WALLETCONNECT_PROJECT_ID` for WalletConnect support.
- **Privy**: Set `VITE_PRIVY_APP_ID` to switch to Privy-based auth. This replaces the default connector entirely.

### Branding
- Place custom logos at `public/logo.webp` and `public/logo-secondary.webp`, then set `VITE_HAS_PRIMARY_LOGO=true` / `VITE_HAS_SECONDARY_LOGO=true`.
- Theme colors are in `app/styles/theme.css` (CSS custom properties). Edit directly or export from [Storybook](https://storybook.orderly.network/?path=/story/package-trading-tradingpage--page).
- PnL share poster backgrounds go in `public/pnl/`.

### Navigation
- `VITE_ENABLED_MENUS` — which pages to show. Available: Trading, Portfolio, Markets, Swap, Leaderboard, Campaigns, Rewards, Vaults, Points.
- `VITE_CUSTOM_MENUS` — external nav links, format: `"Name,URL;Name,URL"`.

## Project Structure

```
.env                         # ALL configuration (self-documenting)
app/
  App.tsx                    # Root component
  main.tsx                   # Entry point, router, runtime config loader
  components/
    orderlyProvider/         # SDK provider setup (network, chains, wallets)
    CustomLeftNav.tsx        # Mobile hamburger nav
    WooFiWidget.tsx          # Swap widget (route /swap)
  pages/                     # Route pages (perp, portfolio, markets, etc.)
  styles/
    theme.css                # Theme colors (CSS vars)
    index.css                # Global styles + imports
  utils/
    runtime-config.ts        # Reads env vars (window.__RUNTIME_CONFIG__ or import.meta.env)
    config.tsx               # Builds full UI config (nav, logos, footer, TradingView, PnL)
    walletConfig.ts          # EVM/Solana wallet connector setup
    seo.ts                   # SEO meta tags from env vars
    symbol-filter.ts         # Whitelist specific trading pairs
    base-path.ts             # URL base path for subdirectory deployments
public/
  favicon.webp
  logo.webp / logo-secondary.webp  # Custom logos (opt-in via .env)
  pnl/                             # PnL share poster backgrounds
  tradingview/                      # TradingView charting library (vendored)
scripts/
  generate-manifest.ts       # PWA manifest generation from .env + theme.css
```

## Runtime Config

The app can read config from `public/config.js` at runtime (no rebuild needed). This is loaded in `main.tsx` before React mounts. If `public/config.js` is absent, values fall back to `.env` (baked in at build time).

Format of `public/config.js`:
```js
window.__RUNTIME_CONFIG__ = {
  VITE_ORDERLY_BROKER_ID: "your_broker_id",
  // ...any VITE_* variable
};
```

## Build & Deploy

### GitHub Pages (default)

This repo includes `.github/workflows/deploy.yml` which auto-deploys to GitHub Pages on every push to `main` or `master`.

**One-time setup after forking** (3 commands, run from your local clone):

```sh
REPO="OWNER/REPO"   # e.g. "myuser/my-dex"

# 1. Enable GitHub Actions
gh api repos/$REPO/actions/permissions -X PUT \
  -f enabled=true -f allowed_actions=all

# 2. Grant workflows write permissions (needed for Pages deploy)
gh api repos/$REPO/actions/permissions/workflow -X PUT \
  -f default_workflow_permissions=write \
  -F can_approve_pull_request_reviews=true

# 3. Enable GitHub Pages with Actions as deployment source
gh api repos/$REPO/pages -X POST -f build_type=workflow
```

<details>
<summary>Using Composio instead of <code>gh</code></summary>

If you don't have `gh` CLI available, use the equivalent Composio tools:

| Step | `gh api` command | Composio tool | Parameters |
|------|------------------|---------------|------------|
| 1 | `PUT repos/$REPO/actions/permissions` | `GITHUB_SET_GITHUB_ACTIONS_PERMISSIONS_FOR_A_REPOSITORY` | `owner`, `repo`, `enabled=true`, `allowed_actions="all"` |
| 2 | `PUT repos/$REPO/actions/permissions/workflow` | `GITHUB_SET_DEFAULT_WORKFLOW_PERMISSIONS_FOR_A_REPOSITORY` | `owner`, `repo`, `default_workflow_permissions="write"`, `can_approve_pull_request_reviews=true` |
| 3 | `POST repos/$REPO/pages` | `GITHUB_CREATE_OR_UPDATE_GITHUB_PAGES_SITE` | `owner`, `repo`, `build_type="workflow"` |

</details>

After that, every push to main/master builds and deploys automatically. The deployed URL:

```
https://<github-username>.github.io/<repo-name>/
```

**Custom domain:** Create a `CNAME` file in the repo root with your domain, then set `VITE_BASE_URL=` (empty) in `.env` so it builds with root path.

### Manual / other hosting

```sh
yarn build:spa      # outputs to build/client/
```

For subdirectory deployments, set `VITE_BASE_URL=/repo-name/` in `.env` before building.

Serve `build/client/` with any static host, with SPA fallback to `index.html` (or copy `index.html` to `404.html`).

## Conventions

- All env vars are prefixed `VITE_` (required by Vite).
- Boolean env vars use string `"true"` / `"false"`.
- Comma-separated lists are parsed at runtime (no JSON arrays in env vars).
- The theme is pure CSS custom properties — no build step for color changes.
- The codebase uses path aliases: `@/` maps to `app/`.
