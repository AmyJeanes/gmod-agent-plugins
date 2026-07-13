# glua-lsp

[`glua_ls`](https://github.com/Pollux12/gmod-glua-ls) language server for Claude Code. Provides automatic diagnostics after every edit plus hover, jump-to-definition, and find-references for `.lua` files in Garry's Mod GLua projects.

`glua_ls` is a hard fork of EmmyLua Analyzer Rust, maintained specifically for Garry's Mod. It reads `.luarc.json` from your workspace, so any existing GLua API stub configuration (e.g. `luttje/glua-api-snippets`) keeps working unchanged.

## Supported Extensions
`.lua`

## How it works

The plugin points its LSP `command` straight at the workspace's pinned binary: `${CLAUDE_PROJECT_DIR}/.tools/bin/glua_ls.exe`. `${CLAUDE_PROJECT_DIR}` is the project root you launched Claude Code in, so each project supplies its own pinned binary and the plugin doesn't install one globally.

The `command` is an absolute path (contains slashes), so Claude Code spawns it directly — no `PATH` lookup, no interpreter. This matters on Windows: a bare command like `node` is resolved against the PATH of Claude Code's server process, which is minimal and often lacks `nodejs`, so bare-interpreter launches fail with `Command 'node' not found or is in an unsafe location`. Pointing at the `.exe` sidesteps that entirely. **Windows-only** — the path hardcodes `.exe`.

## Project setup

Your project needs three things in `.tools/`:

- `.tools/bin/glua_ls.exe` — the binary the plugin launches.
- `.tools/bin/glua_check(.exe)` *(optional)* — the CLI sibling, used by per-project lint scripts and CI.
- `.tools/glua-api/` — type stubs from [`luttje/glua-api-snippets`](https://github.com/luttje/glua-api-snippets) (referenced by `.luarc.json` under `workspace.library`).

The `install-glua-ls` skill (auto-loaded with this plugin) describes a `scripts/install-tools.ps1` template that handles all three with pinned versions. If your project doesn't have one yet, ask Claude to set it up.

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

If `/plugin` Errors tab shows the LSP failing, the skill has the diagnose-first checklist. The most common causes are: no `.tools/bin/glua_ls.exe` under the project root (Claude Code can't spawn the path, so the Errors tab shows a spawn failure for it), Claude launched somewhere other than the addon root so `${CLAUDE_PROJECT_DIR}` doesn't point at `.tools/bin/`, or missing GLua API stubs (every GMod global flagged as `undefined-global`).

## More information
- [`glua_ls` on GitHub](https://github.com/Pollux12/gmod-glua-ls)
- [`glua_check` CLI sibling](https://github.com/Pollux12/gmod-glua-ls/releases) — same engine as a one-shot linter
- [`luttje/glua-api-snippets`](https://github.com/luttje/glua-api-snippets) — type stubs source
