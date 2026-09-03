# Superpowers for Grok Build CLI

Complete guide for using Superpowers with xAI's Grok Build CLI.

## Installation

Superpowers is published in the [official Grok plugin marketplace](https://github.com/xai-org/plugin-marketplace).

```bash
grok plugin install superpowers@xai-official --trust
```

Or open the marketplace in the TUI, search for Superpowers, and install it:

```text
/marketplace
```

Start a new session after installing.

## How It Works

This repository carries **no Grok-specific manifest, hook, or extension**. Grok
Build CLI installs Superpowers through xAI's marketplace entry, which distributes
the plugin's shared content:

- `skills/` — the harness-agnostic skills tree, identical on every harness.
- `skills/using-superpowers/SKILL.md` — the bootstrap skill that makes the agent
  check for a relevant skill before acting.

Because there is nothing Grok-specific in the repo, the exact mechanism the
marketplace entry uses to surface the skills and the bootstrap is owned by xAI's
marketplace rather than documented here. Superpowers was added to the install docs
in v6.3.0 (#1919). If you find that skills install but never auto-trigger, that is
worth an issue — it is the signal that the bootstrap is not reaching the model on
this harness.

## Tool Mapping

None shipped. Skills describe actions ("read a file", "run a shell command",
"dispatch a subagent", "create a todo") rather than naming tools, so they work
against whatever tools the harness exposes. If a skill asks for a capability Grok
does not have — subagent dispatch, todo tracking — the skill's own fallback
wording applies: do the work inline, or track tasks in a plan file, rather than
inventing tool calls.

## Verifying the install

Smoke check, in a fresh session:

```text
Tell me about your superpowers
```

Full check — in a clean session, send exactly:

```text
Let's make a react todo list
```

A working install triggers `brainstorming` before any code is written.

## Updating

Reinstall from the marketplace:

```bash
grok plugin install superpowers@xai-official --trust
```

Start a new session afterwards.

## Troubleshooting

### Skills never trigger

1. Confirm the plugin is installed, enabled, and trusted (`--trust` on install).
2. Start a fresh session.
3. Ask the model to list the skills it can see. If they are listed but never load
   on their own, the bootstrap is not being delivered — open an issue with the
   transcript and your Grok Build CLI version.

## Tests

None in this repo: there are no Grok-specific files to test. The shared skills
content this integration installs is covered by the rest of `tests/` and by the
evals in [docs/testing.md](testing.md).

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Grok plugin marketplace: https://github.com/xai-org/plugin-marketplace
