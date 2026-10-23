# sedea-extensions/extension-template

GitHub **template repository v0.1** for Sedea **user bespoke** VS Code extensions.

Generate a new project from this template (GitHub **Use this template** or `gh repo create my-ext --template sedea-extensions/extension-template`), then:

1. Replace `your-publisher-id` in `package.json`.
2. Run `npm ci && npm run verify`.

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
- Example MCP contribution manifest (`sedea.mcp-contribution.example.json`)

## What v0.1 does not include

- Full webview ↔ extension host message-passing scaffold (Storybook covers isolated component dev only)
- Runtime MCP server implementation (document manifest only)
- Sedea app monorepo wiring (`@sedea/extension-shared`, native extension symlinks)
- Mandatory Chromatic visual regression (opt-in via repo secret)

## Adding a Hub launcher icon

Sedea Hub shows a toolbar of large icon buttons above the Dispatch box. Each button runs one command with one click. An extension adds its own icon through the Hub's runtime API. This section is a minimal recipe; the full contract (item fields, reject codes, lifecycle) is in the Hub UI document, [`docs/sedea-hub-ui.md`](https://github.com/sedea-ai/app/blob/main/docs/sedea-hub-ui.md) in `sedea-ai/app`.

### 1. Declare the command and the Hub dependency

Every launcher item runs a command, and the command must be declared in your own `contributes.commands`. Prefix command ids with a name your extension owns (for example `missionControl.openThing`), because command ids are global in VS Code. This is a convention and is not enforced.

An extension that activates only on its own commands never shows its icon until a command runs. So that the icon appears on every start, activate at startup with `onStartupFinished` and list the Hub in `extensionDependencies`. New projects generated from this template already activate on `onStartupFinished`; if you changed `activationEvents`, add it back.

```json
{
  "activationEvents": ["onStartupFinished"],
  "extensionDependencies": ["vscode.sedea-hub"],
  "contributes": {
    "commands": [
      { "command": "sedeaExtensionTemplate.hello", "title": "Sedea Extension Template: Hello" }
    ]
  }
}
```

A hard `extensionDependencies` entry also means your extension does not activate without the Hub. If your extension must keep working without the Hub, leave the entry out and rely on the guarded lookup below, which skips registration when the Hub is absent.

### 2. Register the item after your own activation

Register your commands first, then fetch the Hub by id, wait for its exports, and register only when `registerLauncherItems` exists, so older Hubs are tolerated.

```ts
import * as vscode from 'vscode';

interface HubLauncherApi {
  apiVersion: number;
  registerLauncherItems?: (input: {
    extensionId: string;
    items: { id: string; name: string; command: string; icon: string; tooltip: string }[];
  }) => vscode.Disposable;
}

export async function activate(context: vscode.ExtensionContext): Promise<void> {
  context.subscriptions.push(
    vscode.commands.registerCommand('sedeaExtensionTemplate.hello', () => {
      void vscode.window.showInformationMessage('Sedea extension template is active.');
    })
  );

  const hub = vscode.extensions.getExtension<HubLauncherApi>('vscode.sedea-hub');
  const api = await hub?.activate();
  if (!api?.registerLauncherItems) {
    return; // No Hub, or a Hub without the launcher API: skip quietly.
  }

  context.subscriptions.push(
    api.registerLauncherItems({
      extensionId: context.extension.id,
      items: [
        {
          id: 'hello',
          name: 'Hello',
          command: 'sedeaExtensionTemplate.hello',
          icon: 'smiley',
          tooltip: 'Say hello'
        }
      ]
    })
  );
}
```

Notes for authors:

- Each item has `id`, `name`, `command`, `icon` (a codicon id) and `tooltip`.
- The call returns a disposable that removes the items.
- `extensionId` must be an installed extension, and each item's command must be declared in that extension's own `contributes.commands`.
- If the Hub depends on your extension (Mission Control is one example), do not wait for the Hub inside your own `activate()`, because that deadlocks. Defer the registration until after your own activation completes.

### 3. Open one reusable tab from the icon

When a launcher item opens a tab, make it a single reusable tab. Keep one panel reference. When the command runs and the panel exists, reveal it instead of creating another. When the panel is disposed, clear the reference so the next click creates a fresh panel.

## Governance

Canonical template maintenance: sedea center **`maintain extension template`** mission. Post-scaffold feature delivery: R&D **`plan and deliver`** on the user extension repo.
