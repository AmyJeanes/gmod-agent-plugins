---
name: install-glua-ls
description: Set up or troubleshoot project-local glua_ls, glua_check, and GLua API type stubs. Use in Claude Code when live LSP diagnostics, hover, or jump-to-definition fail; use in Claude or Codex when the pinned tools are missing, GLua checks cannot run, or built-in GMod globals such as IsValid, LocalPlayer, hook, ents, util, Color, or CreateConVar are reported as undefined.
---

# Set up GLua tooling

Each project provides its own pinned GLua tools; nothing is installed globally.

- Claude Code launches `glua_ls` from `${CLAUDE_PROJECT_DIR}/.tools/bin/glua_ls.exe` through this plugin's `lspServers` declaration.
- Codex uses this plugin's setup skill and the project's one-shot check scripts. Codex does not currently attach `glua_ls` as a live LSP tool through the plugin.
- VS Code can use the same project-local binary through the Pollux extension.

Two pieces must exist in the workspace:

1. **`.tools/bin/glua_ls.exe`** — the binary the plugin launches.
2. **`.tools/glua-api/`** — type stubs from `luttje/glua-api-snippets`, referenced from `.luarc.json` under `workspace.library`. Without these, every GMod global (`IsValid`, `hook`, `ents`, `Color`, `LocalPlayer`, `CreateConVar`, etc.) shows as `undefined-global`.

Diagnose first, then install only what's missing.

## Diagnose

Run both checks from the project root:

```powershell
Test-Path .tools/bin/glua_ls.exe
Test-Path .tools/glua-api/_globals.lua # Path may vary; check workspace.library in .luarc.json.
```

If either is missing, see **Install** below.
If both pass but diagnostics are still wrong, see **If it still doesn't work**.

## Install

Most well-set-up projects already have `scripts/install-tools.ps1` — run it:

```bash
pwsh -File scripts/install-tools.ps1
```

It is idempotent and provisions both pieces with versions pinned at the top of the script (so local matches CI).

### If the project has no `scripts/install-tools.ps1`

Create one. It needs to do two things, each idempotent:

1. **Download pinned `glua_ls` and `glua_check` releases from `Pollux12/gmod-glua-ls`** into a versioned cache under `.tools/glua-ls/<ver>/` and `.tools/glua-check/<ver>/`, then mirror the binaries to `.tools/bin/glua_ls(.exe)` and `.tools/bin/glua_check(.exe)`. Claude's plugin launches `.tools/bin/glua_ls.exe` from the project root.
2. **Download the latest `luttje/glua-api-snippets` `.lua.zip` release** into `.tools/glua-api/`, with a `.tools/glua-api/.version` marker so it only re-downloads on version change. Reference this directory from `.luarc.json` under `workspace.library`.

Pin both versions as constants at the top of the script so contributors and CI run the exact same engine. The plugin's own repo (`AmyJeanes/gmod-agent-plugins`) sources several reference projects (TARDIS, Doors) — copy `scripts/install-tools.ps1` and `scripts/glua-check.ps1` from one of those if you want a working starting point. Use Renovate's `customManagers` regex to auto-bump pinned versions:

```jsonc
// .github/renovate.json
{
  "customManagers": [{
    "customType": "regex",
    "managerFilePatterns": ["/^scripts/install-tools\\.ps1$/"],
    "matchStrings": [
      "# renovate: datasource=(?<datasource>\\S+) depName=(?<depName>\\S+)(?: versioning=(?<versioning>\\S+))?\\s+\\$\\w+\\s*=\\s*'(?<currentValue>[^']+)'"
    ]
  }]
}
```

with annotations on each pinned version in the script:

```powershell
# renovate: datasource=github-releases depName=Pollux12/gmod-glua-ls
$GluaLsVersion  = '1.0.15'
# renovate: datasource=github-releases depName=luttje/glua-api-snippets versioning=loose
$GluaApiVersion = '2026-03-31_16-30-01'
```

Don't forget to gitignore `.tools/`.

## Activate and verify

In Claude Code, reload plugins after installing:

```
/reload-plugins
```

Then trigger an edit to a `.lua` file. Diagnostics should appear automatically.

In Codex, run the repository's one-shot check instead:

```powershell
pwsh -File scripts/glua-check.ps1
```

If the repository does not have that runner yet, use the `port-glua-tooling` skill to standardize it. For interactive diagnostics, use Claude Code's plugin or VS Code's Pollux extension.

## If it still doesn't work

- In Claude Code, open `/plugin` and check the **Errors** tab. A spawn failure for `.tools/bin/glua_ls.exe` means the binary is missing or Claude Code was launched outside the addon root, so `${CLAUDE_PROJECT_DIR}/.tools/bin/glua_ls.exe` does not resolve correctly. Relaunch from the addon root or install the binary.
- In Claude Code, `Command 'node' not found or is in an unsafe location` indicates a stale pre-0.3.2 plugin install. Run `/reload-plugins` or update the plugin; 0.3.2+ launches the executable directly.
- In Codex, do not troubleshoot absent live LSP diagnostics as a plugin spawn failure. Verify the files and run `scripts/glua-check.ps1`; Codex's plugin integration is skills-only.
- Check that the project has a `.luarc.json`. `glua_ls` keys most of its analysis off it; without one, diagnostics will be sparse and globals will look undefined even when the stubs exist.
- Confirm the stubs path in `.luarc.json` is correct. The path is relative to the project root; if the project layout is unusual (e.g. nested addon directories) the stubs may need to live somewhere else.

## Related

- Upstream LSP: <https://github.com/Pollux12/gmod-glua-ls>
- CLI sibling for one-shot linting: download `glua_check-*` from the same GitHub release
- Stub source: <https://github.com/luttje/glua-api-snippets>
