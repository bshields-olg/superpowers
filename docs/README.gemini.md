# Superpowers for Gemini CLI

Complete guide for using Superpowers with [Gemini CLI](https://github.com/google-gemini/gemini-cli).

## Installation

```bash
gemini extensions install https://github.com/obra/superpowers
```

Restart Gemini CLI. The extension ships the skills and the bootstrap together;
nothing in your home directory is edited.

## How It Works

Gemini CLI is an **instructions-file** integration: there is no hook to run and
no plugin module to execute, so the bootstrap rides a context file that the
extension itself ships.

`gemini-extension.json` is the manifest:

```json
{
  "name": "superpowers",
  "description": "Core skills library: TDD, debugging, collaboration patterns, and proven techniques",
  "version": "6.3.0",
  "contextFileName": "GEMINI.md"
}
```

`contextFileName` points at the extension's own `GEMINI.md`, which Gemini CLI
loads every session. That file is two `@`-includes and nothing else:

```text
@./skills/using-superpowers/SKILL.md
@./skills/using-superpowers/references/gemini-tools.md
```

So the always-loaded context is the `using-superpowers` skill (which carries its
own `<EXTREMELY-IMPORTANT>` block internally) plus the Gemini tool mapping. There
is no injector assembling a string, no frontmatter stripping, and no
"already loaded, don't re-invoke" preamble — for an `@`-include harness the
content *is* the active instruction set.

The manifest has no `skills` field: Gemini CLI auto-discovers the `skills/`
directory bundled inside the installed extension, and the model loads a skill
with `activate_skill`.

Note that the `GEMINI.md` at the repo root is the **extension's** context file.
Your own project and global `GEMINI.md` files are untouched, and Gemini CLI keeps
loading them hierarchically as usual.

## Tool Mapping

[`skills/using-superpowers/references/gemini-tools.md`](../skills/using-superpowers/references/gemini-tools.md)
is loaded into context by `GEMINI.md`, so the agent always has it. Highlights:

| Action skills request | Gemini CLI tool |
|---|---|
| Read a file | `read_file` (`read_many_files` for several) |
| Create a file / edit a file | `write_file` / `replace` |
| Run a shell command | `run_shell_command` |
| Search file contents / find files | `grep_search` / `glob` |
| Fetch a URL / search the web | `web_fetch` / `google_web_search` |
| Invoke a skill | `activate_skill` |
| Dispatch a subagent | `invoke_agent` with `agent_name: "generalist"` (or `@generalist` in chat) |
| Task tracking | `write_todos` |

The reference also covers filling Superpowers' subagent prompt templates before
handing them to `invoke_agent`, parallel dispatch (multiple `invoke_agent` calls
in one response), the `~/.gemini/skills/` and `~/.agents/skills/` personal skill
directories, and Gemini-only tools such as `ask_user`, `enter_plan_mode`, and the
`tracker_*` task tracker.

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

```bash
gemini extensions update superpowers
```

Restart Gemini CLI afterwards.

## Troubleshooting

### Skills never trigger

1. Confirm the extension is installed: `gemini extensions list`.
2. Restart Gemini CLI — the context file is read at session start.
3. Ask the model to list its available skills. If the Superpowers skills are
   missing, extension skill discovery is not working; reinstall the extension.

### The model reads `SKILL.md` with a file tool instead of already knowing it

That means the `@`-include was treated as a hint rather than expanded inline.
This is the behavior seen on Gemini-*derived* CLIs; on Gemini CLI itself the
include is expanded. If you hit it on a fork, that fork needs its own
integration — see [docs/porting-to-a-new-harness.md](porting-to-a-new-harness.md).

## Tests

There are no Gemini-specific tests in `tests/`; the integration is two declarative
files (`gemini-extension.json` and `GEMINI.md`) with no executable parts.
`gemini-extension.json` is registered in `.version-bump.json` so its version stays
in lockstep. Skill-behavior evals drive real Gemini CLI sessions — see
[docs/testing.md](testing.md).

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Gemini CLI docs: https://google-gemini.github.io/gemini-cli/
