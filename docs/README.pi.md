# Superpowers for Pi

Complete guide for using Superpowers with [Pi](https://github.com/earendil-works/pi-coding-agent).

## Installation

```bash
pi install git:github.com/obra/superpowers
```

For local development, run Pi with this checkout loaded as a temporary package:

```bash
pi -e /path/to/superpowers
```

## How It Works

Pi is an **in-process extension** integration. The repo-root `package.json`
declares it:

```json
{
  "keywords": ["pi-package", "..."],
  "pi": {
    "extensions": ["./.pi/extensions/superpowers.ts"],
    "skills": ["./skills"]
  }
}
```

Pi runs the TypeScript extension directly — there is no build step and no runtime
dependency (the `ExtensionAPI` import is type-only and compiles away).

`.pi/extensions/superpowers.ts` registers four lifecycle handlers:

1. `resources_discover` → returns `{ skillPaths: [skillsDir] }`, registering the
   repo's `skills/` directory with Pi's native skill system.
2. `session_start` and `session_compact` → set the `injectBootstrap` flag, so the
   bootstrap is (re)injected at the start of a session **and after compaction**.
3. `agent_end` → clears the flag, so it is injected once per turn rather than on
   every step.
4. `context` → inserts the bootstrap as a **user-role message** (not a system
   message: repeated system messages bloat tokens and break some models), placed
   after any leading `compactionSummary` messages.

The bootstrap string is assembled in code: `using-superpowers/SKILL.md` is read,
its YAML frontmatter stripped, and the body wrapped in `<EXTREMELY_IMPORTANT>`
with a preamble saying the skill is already loaded and must not be re-loaded, plus
the Pi tool mapping. It is cached at module level, so `SKILL.md` is read and
parsed once, not on every callback.

A dedup guard keeps it from stacking up: before injecting, the extension scans the
message array for the marker `superpowers:using-superpowers bootstrap for pi` and
skips if it is already there.

## Skills without a `Skill` tool

Pi has native skill *discovery* but does not expose Claude Code's `Skill` tool.
The blessed way to load a skill on Pi is therefore to **read its `SKILL.md` with
`read` when the skill applies** (or for a human to invoke `/skill:name`
explicitly). That is stated in the injected mapping, so the model does not think
it is violating `using-superpowers`'s "never read skill files manually" rule —
that rule means "don't bypass your platform's skill-loading mechanism", and here
reading the file *is* the mechanism.

## Tool Mapping

Maintained in **two** places, which must stay in sync:

- `piToolMapping()` inside `.pi/extensions/superpowers.ts` — inlined into the
  injected bootstrap.
- [`skills/using-superpowers/references/pi-tools.md`](../skills/using-superpowers/references/pi-tools.md)
  — linked from `using-superpowers`'s Platform Adaptation section.

What they say:

- Pi's built-in coding tools are lowercase: `read`, `write`, `edit`, `bash`, plus
  optional `grep`, `find`, and `ls`.
- **Subagents.** Pi core ships no standard subagent tool. If one is installed —
  `subagent` from the `pi-subagents` companion package is the strong option, with
  single-agent, chain, parallel, async, forked-context, and resume/status
  workflows — use it. If not, work sequentially in the current session or say the
  capability is missing. Never fabricate `Task` calls.
- **Task lists.** Pi core ships no standard task-list tool either. Use an
  installed todo/task tool if there is one; otherwise track work in the plan file
  or a repo-local `TODO.md`. Treat older `TodoWrite` references as this action.

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

Reinstall the package:

```bash
pi install git:github.com/obra/superpowers
```

The extension's version rides the repo-root `package.json`, which is already
tracked in `.version-bump.json`, so there is no separate Pi manifest to bump.

## Troubleshooting

### The model does not know it has superpowers

The bootstrap is not injecting.

1. Confirm the package is installed and the extension loaded (or that you passed
   `pi -e /path/to/superpowers`).
2. Check that `skills/using-superpowers/SKILL.md` exists in the installed package.
   If the file cannot be read, `getBootstrapContent()` caches `null` and injects
   nothing — a missing or truncated install looks exactly like "skills never
   trigger".

### Skills are not discovered

Ask Pi to list skills. If Superpowers' skills are absent, `resources_discover`
did not run — the extension is not loaded, so fix that first.

### Skills stop triggering after a long session

Compaction re-injection is wired (`session_compact` sets the flag again). If it
still happens, capture the transcript and open an issue.

## Tests

- `tests/pi/test-pi-extension.mjs` — fakes Pi's extension API and asserts the
  handlers register, the bootstrap injects exactly once, the dedup guard works,
  compaction re-injection happens, and the tool-mapping reference documents the
  harness-specific mappings.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
