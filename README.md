# Native browser control — merged

The current home is [Agent browser and computer control](https://github.com/alejoacelas/2026-09-agent-browser-rules/blob/main/README.md).
Its [report](https://github.com/alejoacelas/2026-09-agent-browser-rules/blob/main/BROWSER-CAPABILITIES.md) compares Codex, Claude Code,
Cowork and related browser/desktop surfaces.

This repository was imported, with unsquashed Git history, into
[`native-control/`](https://github.com/alejoacelas/2026-09-agent-browser-rules/tree/main/native-control) on
2026-09-28. This checkout and its public remote remain a historical backup.
Continue new work in the combined repository.

[INVESTIGATION.md](INVESTIGATION.md) preserves the September 26 repair and passing
Chrome-control test. Orca uses the standard `~/.codex` home with the System default
account; avoid a global `CODEX_HOME` pointing at an Orca managed account home.
