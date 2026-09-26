# Investigation — 2026-09-26

## Intended result

A fresh Codex session in Orca should discover the standard native Chrome tool,
receive its usage instructions, and control an existing signed-in Chrome tab.
Orca's embedded-browser integration should continue working independently.

## Confirmed observations

- Native Chrome extension is installed/enabled, and its host manifest passed the
  bundled diagnostic checks. Browser runtime discovery returned an extension
  browser in the personal profile. Actual tab listing failed with:
  `failed to start codex app-server: No such file or directory (os error 2)`.
- The conversation exposes `mcp__node_repl__js` and related generic tools, but no
  `cua_repl` tools. No native Chrome/browser skill appears in its injected skill list.
- Both main Codex and active Orca account configs enable `browser@openai-bundled`,
  `chrome@openai-bundled`, and `unified-computer-use@openai-bundled`. No explicit
  browser/chrome exclusion was found in their `skills.config` entries.
- Installed browser and chrome plugin skill directories are empty. Runtime scripts
  and documentation remain present.
- All three `NODE_REPL_INSTRUCTIONS_USE_CASE_BROWSER`, `_CHROME`, and
  `_COMPUTER_USE` environment values are empty. A read-only MCP startup experiment
  compared empty versus unset values: initialize instructions were identical (171
  characters), as were tool description lengths. Removing them alone did not
  restore browser guidance.
- The installed ChatGPT app's `app.asar`, module `.vite/build/main-BefHSPFJ.js`,
  contains function `Ji` that deliberately blanks those variables when unified
  browser/computer surfaces are enabled. Function `Oo` deliberately removes old
  `control-chrome`, `control-in-app-browser`, and computer-use skill directories
  when the replacement surfaces are active. Function `Ene` configures the
  replacement plugin's `cua_repl` MCP server. These are app-generated behaviors,
  not proof of a custom profile damaging the installation. Names are minified and
  version-specific; recheck the installed version before relying on them.
- The replacement plugin's `.mcp.json` enables `cua_repl`, lists tools `js`,
  `js_reset`, `turn_ended`, and sets `omit_tools_from: ["code_mode", "deferred"]`.
  This conversation uses code-mode tool discovery. Filtering is a strong lead,
  but the host's actual filtering path has NOT been verified.
- Starting the replacement MCP server directly and requesting initialize/tools/list
  succeeded. Its `js` description starts: “Control native apps or browsers on the
  user’s computer by reading or operating UI.” It supplies native entry points
  such as `await cua.getState()`. No UI operation was performed in that experiment.
- The replacement plugin config STILL references a missing
  `/Applications/ChatGPT.app/Contents/Resources/codex` executable. The installed app
  now has a `codex-cli` entry; its suitability has not been investigated.

## Prior changes already made

The main and active Orca account configs had `mcp_servers.node_repl.env.CODEX_CLI_PATH`
pointing at the missing app executable. Both were changed to:
`/Users/alejo/.codex/plugins/.plugin-appserver/codex`.
That executable reports `0.155.0-alpha.9.2`; an app-server initialize handshake
succeeded. The ordinary CLI reports `0.154.0`. Full browser control was NOT tested
in a fresh session afterward. This is a workaround, not established durable repair.

Backups are adjacent to each edited config, named
`config.toml.before-chrome-fix-20260926-172919`. The plugin-local `.mcp.json` was
not changed, so it still contains the stale executable path.

At Alejandro's request, three global instruction sentences preferring Browser Use
Cloud and constraining sub-agent UI creation were removed from dotfiles'
`agents/AGENTS.md`, committed and pushed as `f8c9d04`. Do not restore them.
The remaining custom browser-use-cloud skill still exists.

## Local inspection locations

- Main config: `~/.codex/config.toml`.
- Active account config:
  `~/Library/Application Support/orca/codex-accounts/7b28d7d0-4875-4540-8c0c-5b91ce0eb7d1/home/config.toml`.
- Bundled plugins: `~/.codex/plugins/cache/openai-bundled/`.
- Inspected browser/chrome/unified-computer-use version: `26.915.31945`.
- Replacement config: `unified-computer-use/26.915.31945/.mcp.json` under that cache.
- Native host manifest:
  `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/com.openai.codexextension.json`.
- App runtime: `/Applications/ChatGPT.app/Contents/Resources/cua_node/`.
- App code: `/Applications/ChatGPT.app/Contents/Resources/app.asar`.
- Orca app code: `/Applications/Orca.app/Contents/Resources/app.asar`.
- Dotfiles: `~/best/dotfiles`. It had unrelated untracked `skills/supervise-workers/`;
  do not stage, overwrite, or remove other work.

## Questions to resolve

1. Why is the replacement tool omitted from this Orca/Codex session? Trace plugin
   loading, tool filtering, code-mode compatibility, and app-versus-CLI support.
2. Who owns and refreshes executable paths in user/account/plugin configs? Does
   Orca account copying preserve stale app-managed state? So far no direct evidence
   establishes Orca caused the initial stale path or removed skills.
3. What supported setup restores native instructions and tools durably? Do not
   blindly remove empty variables, repopulate legacy skill caches, or strip tool
   omission settings without understanding the replacement architecture.
4. Validate after a fresh session, restart, and account switch; distinguish tool
   discovery, successful launch, and real browser operation. Confirm Orca's own
   embedded browser remains functional. Coordinate disruptive restarts with user.

## Authorization and handoff

Alejandro explicitly requested Opus 5.5 to take ownership in a new fun project.
Investigate and implement reversible fixes autonomously. Ask him before any
uninstall/reinstall or permanent deletion, explaining exactly what would change.
Prefer a supported reinstall when justified over brittle ongoing cache edits, but
wait for his confirmation. Preserve credentials and other active work. Report
verified results and limitations, and document durable findings here.
