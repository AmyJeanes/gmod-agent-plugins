# glua-lsp

[`glua_ls`](https://github.com/Pollux12/gmod-glua-ls) language server setup for
Garry's Mod GLua projects.

Claude Code consumes this plugin's `lspServers` marketplace declaration and can
provide automatic diagnostics after every edit plus hover, jump-to-definition,
and find-references for `.lua` files.

Codex consumes the same package as a skills-only plugin through
`.codex-plugin/plugin.json`. Current Codex plugin support does not include a
Claude-style `lspServers` field, so Codex can use the bundled setup and porting
workflows but cannot attach `glua_ls` as a live editor/LSP tool through this
plugin. Native Codex CLI LSP support is tracked in
[openai/codex#8745](https://github.com/openai/codex/issues/8745).

`glua_ls` is a hard fork of EmmyLua Analyzer Rust, maintained specifically for Garry's Mod. It reads `.luarc.json` from your workspace, so any existing GLua API stub configuration (e.g. `luttje/glua-api-snippets`) keeps working unchanged.

## Supported Extensions
`.lua`

## How it works in Claude Code

The plugin points its LSP `command` straight at the workspace's pinned binary: `${CLAUDE_PROJECT_DIR}/.tools/bin/glua_ls.exe`. `${CLAUDE_PROJECT_DIR}` is the project root you launched Claude Code in, so each project supplies its own pinned binary and the plugin doesn't install one globally.

The `command` is an absolute path (contains slashes), so Claude Code spawns it directly — no `PATH` lookup, no interpreter. This matters on Windows: a bare command like `node` is resolved against the PATH of Claude Code's server process, which is minimal and often lacks `nodejs`, so bare-interpreter launches fail with `Command 'node' not found or is in an unsafe location`. Pointing at the `.exe` sidesteps that entirely. **Windows-only** — the path hardcodes `.exe`.

## Project setup

Your project needs three things in `.tools/`:

- `.tools/bin/glua_ls.exe` — the binary the plugin launches.
- `.tools/bin/glua_check(.exe)` *(optional)* — the CLI sibling, used by per-project lint scripts and CI.
- `.tools/glua-api/` — type stubs from [`luttje/glua-api-snippets`](https://github.com/luttje/glua-api-snippets) (referenced by `.luarc.json` under `workspace.library`).

The `install-glua-ls` skill (auto-loaded with this plugin) describes a `scripts/install-tools.ps1` template that handles all three with pinned versions. If your project doesn't have one yet, ask your agent to set it up.

In Codex, use the `install-glua-ls` and `port-glua-tooling` skills from this
plugin to install or standardize the same project-local tooling. For one-shot
checks, run the project's `scripts/glua-check.ps1`; for interactive editor LSP
features, use the VS Code Pollux extension configured to `.tools/bin/glua_ls`.

`.luarc.json` example:

```json
{
  "runtime": { "version": "LuaJIT" },
  "workspace": {
    "library": [ "./.tools/glua-api" ]
  }
}
```

## Troubleshooting

In Claude Code, if `/plugin` shows an LSP failure, use the skill's diagnose-first checklist. The most common causes are a missing `.tools/bin/glua_ls.exe`, launching Claude outside the addon root, or missing GLua API stubs.

In Codex, verify the project-local files and run `scripts/glua-check.ps1`. The absence of live LSP diagnostics is expected until Codex adds a native LSP integration.

## More information
- [`glua_ls` on GitHub](https://github.com/Pollux12/gmod-glua-ls)
- [`glua_check` CLI sibling](https://github.com/Pollux12/gmod-glua-ls/releases) — same engine as a one-shot linter
- [`luttje/glua-api-snippets`](https://github.com/luttje/glua-api-snippets) — type stubs source
