# Investigation

Updated 2026-09-26 by Claude Opus 5.5. The earlier handoff notes are folded in below.

## Intended result

A fresh Codex session in Orca should discover the native Chrome tool (`cua_repl`),
receive its usage instructions, and control an existing signed-in Chrome tab.
Orca's embedded-browser integration should keep working independently.

## Root cause (verified)

1. **Orca account homes cannot load the `openai-bundled` marketplace.** Codex treats
   `openai-bundled` as a reserved marketplace and accepts it only when its `source`
   is exactly `$CODEX_HOME/.tmp/bundled-marketplaces/openai-bundled`
   ([`managed_local_marketplace_name`](https://github.com/openai/codex/blob/main/codex-rs/core-plugins/src/marketplace_policy.rs)).
   Orca launches Codex with `CODEX_HOME` set to a managed account home and mirrors
   `~/.codex/config.toml` into it, so the marketplace `source` still points into
   `~/.codex` and Codex silently drops it. Without that marketplace, the `browser`,
   `chrome`, and `unified-computer-use` plugins do not load, so there is no
   `cua_repl` server, no browser guidance, and no plugin cleanup hooks.
   Orca shares only `skills, hooks, plugins, plugin-state, profile-v2, themes,
   prompts, AGENTS.md` into account homes (`codex-home-paths` chunk, Orca 1.4.212).
   This is the open upstream bug
   [stablyai/orca#20741](https://github.com/stablyai/orca/issues/20741); see also
   [#18682](https://github.com/stablyai/orca/issues/18682).
   - Evidence: with the Orca account home, `codex plugin marketplace list` shows only
     `openai-primary-runtime`, and no process on the machine was running
     `cua-repl.mjs`. With `CODEX_HOME=~/.codex`, the same CLI (0.154.0) lists
     `openai-bundled`, and a fresh `codex exec --profile cli` session reports
     `mcp__cua_repl.js` and `mcp__cua_repl.js_reset` as direct tools.
2. **The code-mode lead is ruled out.** `omit_tools_from = ["code_mode", "deferred"]`
   makes `cua_repl` a direct top-level tool
   ([`apply_mcp_tool_exposure_policy`](https://github.com/openai/codex/blob/main/codex-rs/core/src/tools/spec_plan.rs)),
   which the main-home test above confirms.
3. **The empty `NODE_REPL_INSTRUCTIONS_*` values and missing legacy skills are
   intended.** When the unified surfaces are active, ChatGPT.app blanks the legacy
   `node_repl` instructions and removes `control-chrome`-style skills because the
   guidance now ships with `cua_repl`. Restoring them would not help.

## Stale executable path (verified)

ChatGPT.app is now 26.924.20706 and ships its CLI at
`Resources/codex-cli/bin/codex` (0.158.0-alpha.2). The plugin cache
(`~/.codex/plugins/cache/openai-bundled/*/26.915.31945`) is from the previous app
version, when the CLI lived at `Resources/codex`, which no longer exists. The app
rewrites the cache's `.mcp.json` from its own runtime paths whenever it
re-materializes plugins, so the stale path is left over from the update, not
corruption.

## Global `CODEX_HOME` redirects the ChatGPT app (verified, origin unknown)

`launchctl getenv CODEX_HOME` returns the Orca account home
(`~/Library/Application Support/orca/codex-accounts/7b28d7d0-…/home`). GUI apps
inherit it, so ChatGPT.app runs its app-server against Orca's account home instead
of `~/.codex`. When launched at 17:54, it materialized the 26.924 marketplace into
that home's `.tmp/`, rewrote that home's `config.toml`, and left `~/.codex` at
26.915. Orca then overwrites that config from `~/.codex` on the next Codex launch,
so the two apps fight over one file. No `launchctl setenv` call was found in
Orca 1.4.212, dotfiles, or agent transcripts.

After that launch, ChatGPT.app was quit (it had not been running before) and the
account config was restored from `config.toml.before-cua-repl-20260926-175448`.
The app-created `.tmp/bundled-marketplaces` inside the account home was left in
place; it is harmless.

## Chosen setup: one account on the standard home

Alejandro chose the native setup: a single ChatGPT login, with the ChatGPT app and
Orca's Codex CLI both using `~/.codex`. The CLI departs from the app only through
the `cli` profile (`dotfiles/codex/cli.config.toml`, linked at
`~/.codex/cli.config.toml`), which selects CLI-only skills, plugins, and connectors.
This removes the need for any `cua_repl` copy or Orca account-home workaround.

State as of 18:15:

- `launchctl getenv CODEX_HOME` is empty. It was cleared by something other than
  this investigation; the likely original cause is `launchctl setenv` run from an
  Orca terminal, since every Orca terminal exports its account home.
- ChatGPT.app is running against `~/.codex` (launched with
  `open --env CODEX_HOME=$HOME/.codex`). It refreshed `~/.codex` plugins to
  26.924.20706 and rewrote both `unified-computer-use/.mcp.json` and the legacy
  `mcp_servers.node_repl` block with the correct CLI path
  (`Resources/codex-cli/CodexCLI.app/Contents/MacOS/codex`). The earlier
  `.plugin-appserver` workaround is therefore gone.
- Orca's Codex account is set to **System default**, which is logged in as
  alejoacelas@gmail.com. With that selection, Orca 1.4.212 launches Codex without
  `CODEX_HOME` unless it sees a custom `CODEX_HOME` or fails to trust its status
  hook, in which case it silently uses `orca/codex-runtime-home/home`
  (source: `runtime-home-service-home-routing.ts`, `codex-home.ts`).
- The managed accounts 7b28d7d0 (work) and 8ef57008 remain as inactive entries.
  Removing one in Orca permanently deletes its home; ask Alejandro first.
- Orca's embedded browser is out of scope: Alejandro does not use it.

## Remaining validation

1. In a new Orca terminal, confirm `CODEX_HOME` and `ORCA_CODEX_HOME` are unset,
   run `codex --dangerously-bypass-approvals-and-sandbox --profile cli`, and confirm
   the new session file lands in `~/.codex/sessions/`.
2. In a fresh Codex session launched that way, confirm `mcp__cua_repl.js` is a
   top-level tool, read Chrome state (profile names and tab counts only), and open
   then close one `https://example.com` tab. Alejandro approved exactly this test.
   Record tool discovery, launch, and real control as separate results.
3. Restart persistence: quit ChatGPT.app, reopen it normally from Finder or the
   Dock (no `--env`), confirm its app-server has no Orca `CODEX_HOME`, and repeat
   step 2's tool discovery.
4. Write the verified outcome here and in README.md, then commit and push.

## Prior changes still in place

- `mcp_servers.node_repl.env.CODEX_CLI_PATH` in the main and account configs points to
  `~/.codex/plugins/.plugin-appserver/codex` (0.155.0-alpha.9.2). Backups:
  `config.toml.before-chrome-fix-20260926-172919`. This legacy server is not the
  native browser path; ChatGPT.app rewrites the block on its next materialization.
- Three global instruction sentences preferring Browser Use Cloud were removed from
  dotfiles `agents/AGENTS.md` (`f8c9d04`). Do not restore them.

## Local inspection locations

- Main config: `~/.codex/config.toml`; Orca account config:
  `~/Library/Application Support/orca/codex-accounts/<id>/home/config.toml`.
- Orca launches `codex --dangerously-bypass-approvals-and-sandbox --profile cli`.
  `cli.config.toml` is a dotfiles symlink in `~/.codex` only; account homes lack it.
- App plugin sources: `/Applications/ChatGPT.app/Contents/Resources/plugins/openai-bundled/`.
- Materialized marketplace: `$CODEX_HOME/.tmp/bundled-marketplaces/openai-bundled`.
- Plugin cache: `~/.codex/plugins/cache/openai-bundled/` (account homes symlink `plugins`).
- App main bundle: `app.asar` → `.vite/build/main-BefHSPFJ.js` (functions `Ene`
  writes `cua_repl` config, `Ji` blanks legacy instructions; minified names).
