# Luna -> Sol -> Luna workflow export

This package contains only the portable workflow configuration. It intentionally
excludes authentication, sessions, MCP servers, trusted-project entries, logs,
notifications, plugins, caches, host details, and credentials.

## Files

- `AGENTS.workflow.md`: global routing instructions to merge into `AGENTS.md`.
- `agents/sol-reasoner.toml`: the global read-only Sol custom agent.
- `config-snippet.toml`: the minimal Luna/max and multi-agent settings to merge.

## Windows installation

1. Close Codex. Determine `CODEX_HOME`: use the environment variable when it is
   set; otherwise Windows normally uses `%USERPROFILE%\.codex`.
2. Back up the existing `CODEX_HOME\AGENTS.md` and
   `CODEX_HOME\config.toml`.
3. Append the contents of `AGENTS.workflow.md` to the existing global
   `CODEX_HOME\AGENTS.md`. Merge; do not overwrite unrelated instructions.
4. Copy `agents\sol-reasoner.toml` to
   `CODEX_HOME\agents\sol-reasoner.toml`.
5. Merge `config-snippet.toml` into `CODEX_HOME\config.toml`:
   keep `model` and `model_reasoning_effort` at TOML root scope, and merge
   `enabled = true` into an existing `[agents]` table instead of creating a
   duplicate table. Do not overwrite the full config file.
6. If `CODEX_HOME\AGENTS.override.md` exists and is non-empty, merge the
   workflow into that active file instead; it suppresses global `AGENTS.md`.
7. Restart Codex and start a new task. Existing sessions do not reload the
   instruction chain.

## Requirements and precedence

- The account and Codex client must support custom agents and have access to
  `gpt-5.6-luna` and `gpt-5.6-sol` at `max` reasoning effort.
- Authentication is local to the destination machine and is not included.
- Project or nested `AGENTS.md`/`AGENTS.override.md` instructions load after
  global instructions and may override conflicting guidance.
- Fresh-agent, wait-before-execution, one-Sol, and fail-closed behavior are
  instruction-enforced rather than an operating-system routing gate.
- Avoid elevated parent-session permission overrides if strict read-only Sol
  behavior matters; live parent permissions can take precedence.

## Smoke test

After restarting Codex, create a new task and ask a simple question. Confirm
that a fresh `sol_reasoner` appears before Luna answers. Then request a tiny
edit and confirm Luna waits for Sol, performs the edit itself, and runs the
appropriate validation.

To bypass Sol for one request, explicitly say: `Do not use Sol for this request.`
