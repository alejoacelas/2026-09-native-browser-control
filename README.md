# Restore native Codex browser control

Orca-hosted Codex sessions had lost the native Chrome tool (`cua_repl`). **Fixed and
verified on 2026-09-26:** a fresh Codex session launched from an Orca terminal
discovers `mcp__cua_repl.js`, connects to Chrome, and opened and closed a test tab.
The setup survived quitting and reopening ChatGPT.app.

Cause: Orca's managed account homes cannot load Codex's reserved `openai-bundled`
plugin marketplace ([stablyai/orca#20741](https://github.com/stablyai/orca/issues/20741)).

Setup that works: one ChatGPT login. ChatGPT.app and Orca's Codex CLI both use
`~/.codex`; Orca's Codex account is set to **System default**, and CLI-only choices
live in the `cli` profile (`~/.codex/cli.config.toml`, linked from dotfiles). Keep
`CODEX_HOME` unset, including in `launchctl`; a global value redirects ChatGPT.app
into an Orca account home.

[INVESTIGATION.md](INVESTIGATION.md) has the evidence, validation results, and
inspection locations. Uninstall/reinstall actions require Alejandro's explicit
confirmation first.
