# sedea-extensions/extension-template

GitHub **template repository v0.1** for Sedea **user bespoke** VS Code extensions.

Generate a new project from this template (GitHub **Use this template** or `gh repo create my-ext --template sedea-extensions/extension-template`), then:

1. Replace `your-publisher-id` in `package.json`.
2. Run `npm ci && npm run verify`.
3. Read **`AGENTS.md`** for Storybook, EDH smoke, and MCP contribution guidance.

## Development activation

- `out/extension.js` is gitignored and is not committed to the repository.
- `npm ci` compiles the extension through its `prepare` script and produces TypeScript output only; it does not build the webview.
- The **Run Extension** launch configuration runs `npm ci` before launching the extension.
- After editing TypeScript source, run `npm run compile` to refresh the extension output.

## What v0.1 includes

- TypeScript extension shell with compile / lint / typecheck / test scripts
- Vitest unit test stub
- **Storybook baseline** — React + `@storybook/react-vite` 8.x for isolated webview UI development (`webview-src/components/`)
- GitHub Actions CI (`.github/workflows/ci.yml`) — `verify` plus static `build-storybook` on every PR
- **`AGENTS.md`** — Storybook quick-start, EDH smoke, extension MCP contribution appendix, Sedea MCP Hub external-only boundary
- Example MCP contribution manifest (`sedea.mcp-contribution.example.json`)

## What v0.1 does not include

- Full webview ↔ extension host message-passing scaffold (Storybook covers isolated component dev only)
- Runtime MCP server implementation (document manifest only)
- Sedea app monorepo wiring (`@sedea/extension-shared`, native extension symlinks)
- Mandatory Chromatic visual regression (opt-in via repo secret — see `AGENTS.md`)

## Governance

Canonical template maintenance: sedea center **`maintain extension template`** mission. Post-scaffold feature delivery: R&D **`plan and deliver`** on the user extension repo.
