# stringer

Tooling checked into this repo so it is available in every Claude Code session,
including ephemeral remote/web sessions that start from a fresh clone.

## What persists here

### `.claude/skills/humanizer/`

The [blader/humanizer](https://github.com/blader/humanizer) skill (v2.9.1, MIT).
A single Markdown file with no dependencies and no install step: Claude Code
discovers it automatically in any session opened on this repo.

Removes 33 patterns of AI-sounding prose, derived from Wikipedia's
["Signs of AI writing"](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing).
It never adds facts, names, dates, or citations absent from the source text.

Invoke it by asking — "humanize this", "rewrite so it doesn't sound AI-written" —
or point it at a file. Paste 2-3 paragraphs of your own writing first and it
matches your voice instead of producing generic clean output.

### `.claude/skills/impeccable/` and `.claude/agents/`

[pbakaus/impeccable](https://github.com/pbakaus/impeccable) v4.0.2 (Apache-2.0):
design fluency and anti-pattern detection for frontend work. One skill with 23
commands (`/impeccable polish`, `audit`, `critique`, …), four subagents, and a
static detector that flags the visual tells of AI-generated UI — purple/violet
gradients, neon glow shadows, overused fonts, glassmorphism, low contrast.

Self-contained Markdown plus local Node scripts, so it works from a fresh clone
with no install step. Requires Node >=22.12.

The detector also runs standalone:

```bash
node .claude/skills/impeccable/scripts/detector/detect-antipatterns.mjs <file>
```

For the fuller rule set, install the CLI as well — it catches cases the bundled
detector alone misses:

```bash
npm i -g impeccable
impeccable detect <file-or-dir>
```

The upstream plugin also ships PostToolUse and Stop hooks that re-check design
on every edit. Those are not installed here: they resolve `${CLAUDE_PLUGIN_ROOT}`,
which only exists for marketplace plugin installs, and running a design pass on
every file write is a workflow choice rather than a default. Install via
`/plugin marketplace add pbakaus/impeccable` if you want them.

## What cannot persist here

### freellmapi

[tashfeenahmed/freellmapi](https://github.com/tashfeenahmed/freellmapi) is a
stateful local service, not a skill, so checking it in would not make it
available across sessions:

- Its SQLite database holds your provider API keys, encrypted at rest. Those
  belong on a machine you control, never in git.
- A fresh container regenerates the database from scratch, which mints a new
  unified API key every session. Any client configured against the old key
  breaks immediately.
- It is single-user and localhost-bound by design.

Run it on your own machine instead. See `docs/freellmapi-setup.md`.
