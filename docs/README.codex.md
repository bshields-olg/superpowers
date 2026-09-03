# Superpowers for Codex

Complete guide for using Superpowers with the [Codex](https://developers.openai.com/codex/)
app and CLI.

## Installation

Superpowers is published in the [official Codex plugin marketplace](https://github.com/openai/plugins).

### Codex app

1. Click **Plugins** in the sidebar.
2. Find `Superpowers` in the **Coding** section.
3. Click `+` and follow the prompts.

### Codex CLI

```text
/plugins
```

Search for `superpowers` and select `Install Plugin`.

## Enable subagents

Skills that dispatch subagents — `dispatching-parallel-agents`,
`subagent-driven-development` — need Codex's multi-agent tools. Add to
`~/.codex/config.toml`:

```toml
[features]
multi_agent = true
```

Recommended alongside it, so a spawn that forgets to pin a model does not
silently inherit the session's most expensive one:

```toml
[agents]
default_subagent_model = "<a mid-tier model from your spawn allowlist>"
default_subagent_reasoning_effort = "medium"
```

Without `multi_agent`, subagent-driven skills degrade to running the work
inline; they do not invent `Task` calls.

## How It Works

The Codex manifest is `.codex-plugin/plugin.json`:

1. `"skills": "./skills/"` points Codex at the shared skills tree.
2. `"hooks": {}` — an **empty object, on purpose**. Codex auto-discovers
   `hooks/hooks.json` (the Claude Code `SessionStart` hook) whenever the manifest
   has no `hooks` field, which would re-register that hook and its install-time
   trust prompt. An absent field, `[]`, or an empty inline list all collapse back
   to the auto-discovery fallback, so the value must be exactly `{}`.
3. `interface` carries the app's storefront metadata — display name, short and
   long description, capabilities, default prompts, brand color, and the icons in
   `assets/`.

Codex surfaces skills natively and runs no session-start hook: the installed
`using-superpowers` skill is what the model sees and loads. There is no injector,
no copied skills, and no runtime dependencies.

`.agents/plugins/marketplace.json` is the development marketplace manifest
(`superpowers-dev`) used to install this checkout locally; it installs the repo
root as the plugin source.

## Tool Mapping

Codex-specific guidance lives in
[`skills/using-superpowers/references/codex-tools.md`](../skills/using-superpowers/references/codex-tools.md),
which `using-superpowers`'s Platform Adaptation section points the agent at. It
covers the parts of Codex that Superpowers skills cannot assume:

- **Spawning** — `spawn_agent {fork_turns: "none"}` for a clean child context;
  the default `"all"` copies the whole transcript into the child.
- **Fix rounds** — resume an implementer with `followup_task` rather than
  dispatching a fresh one.
- **Lifecycle** — multi-agent V2 has no `close_agent`; finished children are
  evicted automatically. Only V1 sessions close agents explicitly.
- **Waiting** — `wait_agent` is an event subscription, not a poll. Wait in
  bounded 5–10 minute stretches when idle; short polls cost a call and a context
  rebill and buy nothing.
- **Model routing** — every spawn sets `model` *and* `reasoning_effort`; setting
  `model` alone resets effort to that model's default.
- **Environment detection and Codex-app finishing** — read-only git checks for
  worktrees, and what to do when the app's sandbox blocks branch/push (commit,
  then use the app's "Create branch" / "Hand off to local" controls).

Trust your live tool list over any table, including that one, when they disagree:
which multi-agent version you get depends on the model preset.

## Verifying the install

In a clean session:

```text
Let's make a react todo list
```

A working install triggers `brainstorming` before any code is written.

## Updating

Update through the same marketplace you installed from — **Plugins** in the app,
or `/plugins` in the CLI — then start a new session.

## Distribution (maintainers)

- `scripts/sync-to-codex-plugin.sh` rsyncs the tracked plugin files into the
  marketplace fork and opens a PR. It deliberately excludes repo-internal
  directories (`docs/`, `tests/`, `scripts/`, `evals/`) and the other harnesses'
  dot-directories, and it preserves the OpenAI-owned
  `skills/*/agents/openai.yaml` metadata already in the destination.
- `scripts/package-codex-plugin.sh` builds a standalone rootless archive (zip or
  tar.gz) for portal upload, seeding that same `openai.yaml` metadata from a
  prior official package.
- `.codex-plugin/plugin.json` is registered in `.version-bump.json`, so
  `scripts/bump-version.sh` keeps its version in lockstep with the rest.

## Troubleshooting

### Skills are listed but never trigger

Codex has no session-start hook — triggering depends on the model acting on the
installed `using-superpowers` skill. Start a fresh session and confirm the plugin
is enabled. If it still doesn't trigger, include the transcript in an issue.

### An install-time trust prompt asks about hooks

That means the manifest's empty `hooks` object was lost (a fork, an edit, or a
stale install), and Codex fell back to auto-discovering `hooks/hooks.json`.
Reinstall from the marketplace.

### Subagent dispatch fails or a model name is refused

Enable `multi_agent` (above), and never copy a model name out of a skill or an
old session — V2 accepts only V2-capable presets and hard-errors on the rest.

## Tests

- `tests/codex/test-marketplace-manifest.sh` — marketplace manifest wiring and
  the empty-`hooks` invariant.
- `tests/codex/test-package-codex-plugin.sh` — the portal packaging script.
- `tests/codex-plugin-sync/test-sync-to-codex-plugin.sh` — the fork sync.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Codex docs: https://developers.openai.com/codex/
