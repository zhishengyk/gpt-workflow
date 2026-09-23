# GPT-6 Luna and Sol coding workflow

This package contains only the portable workflow configuration. It intentionally
excludes authentication, sessions, MCP servers, trusted-project entries, logs,
notifications, plugins, caches, host details, and credentials.

## Files

- `AGENTS.workflow.md`: global routing instructions to merge into `AGENTS.md`.
- `agents/sol-coder.toml`: the GPT-6 Sol implementation agent for large code tasks.
- `agents/sol-reasoner.toml`: the read-only GPT-6 Sol agent for non-code questions.
- `config-snippet.toml`: the minimal GPT-6 Luna/max and multi-agent settings to merge.

## Windows installation

1. Close Codex. Determine `CODEX_HOME`: use the environment variable when it is
   set; otherwise Windows normally uses `%USERPROFILE%\.codex`.
2. Back up the existing `CODEX_HOME\AGENTS.md` and
   `CODEX_HOME\config.toml`.
3. Append the contents of `AGENTS.workflow.md` to the existing global
   `CODEX_HOME\AGENTS.md`. Merge; do not overwrite unrelated instructions.
4. Copy both TOML files from `agents\` to `CODEX_HOME\agents\`.
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
  `gpt-6-luna` and `gpt-6-sol` at `max` reasoning effort.
- Authentication is local to the destination machine and is not included.
- Project or nested `AGENTS.md`/`AGENTS.override.md` instructions load after
  global instructions and may override conflicting guidance.
- Small, clear code changes stay in Luna. Large or uncertain code changes use
  the writable `sol_coder`; Luna reviews and verifies its work. Non-code
  questions use the read-only `sol_reasoner`.
- Fresh-agent, one-Sol-at-a-time, and fail-closed behavior are
  instruction-enforced. The `sol_coder` inherits the active parent permission
  mode; its configured `workspace-write` mode does not grant permissions that
  the parent session lacks.

## Smoke test

After restarting Codex, verify all three routes: a non-code question starts a
fresh `sol_reasoner`; a tiny localized code change stays in Luna; and a complex
cross-module coding task starts `sol_coder`, after which Luna reviews and
verifies the diff.

For a task that would normally use Sol, explicitly say: `Do not use Sol for
this request.`
