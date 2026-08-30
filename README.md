# gmod-claude-plugins

A cross-agent plugin marketplace for Garry's Mod GLua development. The repository
currently keeps its historical Claude-specific name, but the plugin packages are
structured so both Claude Code and Codex can consume the useful parts.

## Marketplace files

- `.claude-plugin/marketplace.json` is the Claude Code marketplace. It includes
  the `lspServers` declaration that launches each project's pinned
  `.tools/bin/glua_ls.exe`.
- `.agents/plugins/marketplace.json` is the Codex marketplace. Codex consumes the
  bundled skills via each plugin's `.codex-plugin/plugin.json`.

Codex does not consume Claude's `lspServers` field, so this plugin is
skills-only in Codex for now. Native Codex CLI LSP support is tracked in
[openai/codex#8745](https://github.com/openai/codex/issues/8745).

## Plugins

| Plugin | Description | Required binary |
| :----- | :---------- | :-------------- |
| [`glua-lsp`](plugins/glua-lsp) | GLua language server setup and tooling workflows for `.lua` files | `glua_ls` from the latest `Pollux12/gmod-glua-ls` GitHub release |

## Claude Code install

From inside Claude Code:

```
/plugin marketplace add AmyJeanes/gmod-claude-plugins
/plugin install glua-lsp@gmod-claude-plugins
```

Each plugin's README documents which external binaries it needs.

## Codex install

From Codex CLI, add this repository as a local marketplace and install the
plugin:

```powershell
codex plugin marketplace add C:\path\to\gmod-claude-plugins
codex plugin add glua-lsp@gmod-agent-plugins
```

Start a new Codex session after installing so the bundled skills are visible.

## Project-scoped install

To have collaborators on a specific project be auto-prompted to install these plugins, add to that project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "gmod-claude-plugins": {
      "source": {
        "source": "github",
        "repo": "AmyJeanes/gmod-claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "glua-lsp@gmod-claude-plugins": true
  }
}
```

## License

MIT — see [LICENSE](LICENSE).
